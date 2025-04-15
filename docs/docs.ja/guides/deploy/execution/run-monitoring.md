---
title: '実行監視でクラッシュしたワーカーを検出して再起動する'
sidebar_position: 500
---

Dagster は、ハングした実行を検出し、クラッシュした [実行ワーカー](/guides/deploy/oss-deployment-architecture#job-execution-flow) を再起動できます。実行モニタリングを使用するには、以下の手順が必要です:

- Dagster デーモンの実行
- Dagster インスタンスで実行モニタリングを有効にする:

<CodeExample
  path="docs_snippets/docs_snippets/deploying/dagster_instance/dagster.yaml"
  startAfter="start_run_monitoring"
  endBefore="end_run_monitoring"
/>

:::note

Dagster+では、実行監視は常に有効になっており、[デプロイメント設定](/dagster-plus/deployment/management/deployments/deployment-settings-reference)で設定できます。

:::

## 実行開始タイムアウト

Dagster が実行を開始すると、実行ワーカーが起動して実行を STARTED としてマークするまで、実行は STARTING ステータスのままになります。
何らかの障害により実行ワーカーが起動しない場合、実行は STARTING ステータスのままになる可能性があります。
`start_timeout_seconds` は、実行がこの状態で停止する時間制限を指定します。この時間制限を超えると、実行は失敗としてマークされます。

## 実行キャンセルのタイムアウト

Dagster が実行を終了すると、実行は CANCELING ステータスに移行し、実行ワーカーに終了シグナルを送信します。
実行ワーカーがリソースをクリーンアップすると、CANCELED ステータスに移行します。
何らかの障害により実行ワーカーが正常にスピンダウンしない場合、実行は CANCELING ステータスのままになる可能性があります。
`cancel_timeout_seconds` は、実行がこの状態で停止する時間制限を指定します。この時間制限を超えると、実行はキャンセルとしてマークされます。

## 一般的な実行タイムアウト

実行が「開始」とマークされた後、さまざまな理由（ユーザー API エラー、ネットワークの問題など）により、無期限にハングアップする可能性があります。
デプロイメント内のすべての実行の最大実行時間を設定するには、dagster.yaml または [Dagster+ デプロイメント設定](/dagster-plus/deployment/management/deployments/deployment-settings-reference) の `run_monitoring.max_runtime_seconds` フィールドに最大実行時間（秒）を設定します。
実行がこのタイムアウトを超え、実行モニタリングが有効になっている場合は、失敗とマークされます。
`dagster/max_runtime` タグを使用して、実行ごとにタイムアウト（秒）を設定することもできます。

たとえば、デプロイメント内のすべての実行の最大実行時間を 2 時間に設定するには、次のようにします。

```yaml
run_monitoring:
  enabled: true
  max_runtime_seconds: 7200
```

または、Dagster+ で、[デプロイメント設定](/dagster-plus/deployment/management/deployments/deployment-settings-reference) に以下を追加します:

```yaml
run_monitoring:
  max_runtime_seconds: 7200
```

以下のコード例は、ジョブごとに 10 秒の実行タイムアウトを設定する方法を示しています:

<CodeExample
  path="docs_snippets/docs_snippets/deploying/monitoring_daemon/run_timeouts.py"
  startAfter="start_timeout"
  endBefore="end_timeout"
/>

## 実行ワーカーのクラッシュの検出

:::note

実行ワーカーのクラッシュの検出は、<PyObject section="internals" module="dagster._core.launcher" object="DefaultRunLauncher" /> 以外の実行ランチャーを使用している場合にのみ機能します。

:::

実行中に実行ワーカープロセスがクラッシュする可能性があります。
これは、実行ワーカープロセスが稼働しているホストがダウンしたり、メモリ不足になったりするなど、様々な理由で発生する可能性があります。
監視デーモンがない場合、以下の2つの結果が考えられますが、どちらも望ましくありません。

- 実行ワーカーが割り込みをキャッチできた場合、実行は失敗としてマークされます。
- 実行ワーカーが猶予期間なしにダウンした場合、実行はSTARTEDステータスのままハングしたままになる可能性があります。

実行ワーカーがクラッシュすると、そのワーカーが管理している実行もハングする可能性があります。
監視デーモンは、すべてのアクティブな実行に対して実行ワーカーのヘルスチェックを実行し、これを検出できます。
失敗した実行ワーカーが検出された場合（例：K8sジョブの終了コードが0以外）、実行は失敗としてマークされるか、再開されます（以下を参照）。

## 実行ワーカーのクラッシュ後の実行の再開

この機能は現在、以下の場合にのみサポートされます。

- [`K8sRunLauncher`](/api/python-api/libraries/dagster-k8s#dagster_k8s.K8sRunLauncher) と [`k8s_job_executor`](/api/python-api/libraries/dagster-k8s#dagster_k8s.k8s_job_executor) の組み合わせ
- [`DockerRunLauncher`](/api/python-api/libraries/dagster-docker#dagster_docker.DockerRunLauncher) と [`docker_executor`](/api/python-api/libraries/dagster-docker#dagster_docker.docker_executor) の組み合わせ

監視デーモンは、実行ワーカーのヘルスチェックを実行することでこれらの処理を行います。
失敗が検出された場合、デーモンは新しい実行ワーカーを起動し、既存の実行を再開します。
実行ワーカーのクラッシュはイベントログに記録され、実行は完了まで続行されます。
実行ワーカーがクラッシュし続ける場合、デーモンは設定された試行回数後に実行を失敗としてマークします。

有効にするには、`max_resume_run_attempts` を 0 より大きい値に設定します。
