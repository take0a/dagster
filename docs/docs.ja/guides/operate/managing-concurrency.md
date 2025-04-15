---
title: 'Dagster アセット、ジョブ、および Dagster インスタンスの同時実行の管理'
sidebar_label: Managing concurrency
description: How to limit the number of runs a job, or assets for an instance of Dagster.
sidebar_position: 50
---

Dagster ジョブ、特定のアセット、または特定のタイプのアセットやジョブの同時実行数を制御する必要があることがよくあります。
データパイプラインの同時実行数を制限すると、パフォーマンスの問題やダウンタイムを防ぐのに役立ちます。

:::note

この記事では、[assets](/guides/build/assets/) と [jobs](/guides/build/jobs/) に関する知識があることを前提としています。

:::

## 同時に実行できる実行の総数を制限します

- Dagster Core の場合は、[dagster.yaml](/guides/deploy/dagster-yaml) に以下のコードを追加してください。
- Dagster+ の場合は、[deployment settings](/dagster-plus/deployment/management/deployments/deployment-settings-reference) に以下のコードを追加してください。

```yaml
concurrency:
  runs:
    max_concurrent_runs: 15
```

## すべての実行でアクティブに実行されるアセットまたはオペレーションの数を制限する {#limit-the-number-of-assets-or-ops-actively-executing-across-all-runs}

アセットとオペレーションを同時実行プールに割り当てることで、すべての実行における進行中のオペレーションの実行数を制限できます。まず、`pool` キーワード引数を使用して、アセットまたはオペレーションを同時実行プールに割り当てます。

<CodeExample
  path="docs_snippets/docs_snippets/guides/operate/concurrency-pool-api.py"
  language="python"
  title="Specifying pools on assets and ops"
/>

Dagster UI でアセットまたはオペレーションの詳細ペインを表示すると、プールが正しく設定されていることを確認できます。

![Viewing the pool tag](/images/guides/operate/managing-concurrency/asset-pool-tag.png)

アセットとオペレーションを同時実行プールに割り当てたら、Dagster UI または Dagster CLI を使用して、デプロイメント内のそのプールのプール制限を設定できます。

UI を使用してプール「データベース」の制限を指定するには、「デプロイメント」→「同時実行」設定ページに移動し、「プール制限を追加」ボタンをクリックします:

![Setting the pool limit](/images/guides/operate/managing-concurrency/add-pool-ui.png)

CLI を使用してプール「データベース」の制限を指定するには、次のコマンドを使用します:

```
dagster instance concurrency set database 1
```

## 一連のオペレーションで進行中の実行数を制限します。

同時実行プールを使用して、これらのアセットまたはオペレーションを含む進行中の実行数を制限することもできます。
[すべての実行でアクティブに実行されているアセットまたはオペレーションの数を制限する](#limit-the-number-of-assets-or-ops-actively-executing-across-all-runs)セクションの手順に従って、アセットとオペレーションをプールに割り当て、必要な制限を設定できます。

アセットとオペレーションをプールに割り当てたら、デプロイメント設定を変更してプール適用の粒度を設定できます。
特定のオペレーションを含む実行の総数（アクティブに実行されているオペレーションの総数ではなく）を特定の時点で制限するには、プールの粒度を「run」に設定する必要があります。

- Dagster Core の場合は、[dagster.yaml](/guides/deploy/dagster-yaml) に以下のコードを追加してください。
- Dagster+ の場合は、[deployment settings](/dagster-plus/deployment/management/deployments/deployment-settings-reference) に以下のコードを追加してください。

```yaml
concurrency:
  pools:
    granularity: 'run'
```

この粒度が設定されていない場合、デフォルトの粒度は `op` に設定されます。
つまり、制限が `1` のプール `foo` の場合、すべての実行において、特定の時点で実行される op は 1 つだけになりますが、進行中の実行の数はプールの制限の影響を受けません。

### 同時実行プールのデフォルト制限の設定

- Dagster+: [Dagster+ UI](/guides/operate/webserver) または [`dagster-cloud` CLI](/dagster-plus/deployment/management/dagster-cloud-cli/) から、デプロイメント設定の `concurrency` 構成を編集します。
- Dagster Open Source: インスタンスの [dagster.yaml](/guides/deploy/dagster-yaml) を使用します。

```yaml
concurrency:
  pools:
    default_limit: 1
```

## 実行タグごとに進行中の実行数を制限

実行タグごとに進行中の実行数を制限することもできます。
これは、実行中のアセットやオペレーションに関係なく、実行セットを制限する場合に便利です。
たとえば、特定のスケジュールで進行中の実行数を制限したい場合や、すべてのバックフィルで進行中の実行数を制限したい場合などです。

```yaml
concurrency:
  runs:
    tag_concurrency_limits:
      - key: 'dagster/sensor_name'
        value: 'my_cool_sensor'
        limit: 5
      - key: 'dagster/backfill'
        limit: 10
```

### 実行タグの一意の値ごとに、進行中の実行回数を制限する

実行タグの一意の値ごとに個別の制限を適用するには、`applyLimitPerUniqueValue` を使用して一意の値ごとに制限を設定します。
例えば、すべてのバックフィルの実行回数を制限するのではなく、進行中のバックフィルごとに実行回数を制限することができます。

```yaml
concurrency:
  runs:
    tag_concurrency_limits:
      - key: 'dagster/backfill'
        value:
          applyLimitPerUniqueValue: true
        limit: 10
```

## 1回の実行で同時に実行されるオペレーションの数を制限する

プール制限を使用すると、[すべての実行で実行されるオペレーションの数を制限する](#すべての実行でアクティブに実行されるアセットまたはオペレーションの数を制限する)ことができますが、_1回の実行で_実行されるオペレーションの数を制限するには、[実行エグゼキューター](/guides/operate/run-executors)を設定する必要があります。
実行中のオペレーションとアセットの同時実行数を制限するには、PythonまたはDagster UIのLaunchpadを使用して、実行構成で `max_concurrent` を使用します。

<CodeExample
  path="docs_snippets/docs_snippets/guides/operate/concurrency-run-scoped-op-concurrency.py"
  language="python"
  title="Limit concurrent op execution for a single run"
/>

実行中のオペレーション実行数のデフォルトの制限は、使用しているエグゼキュータによって異なります。
たとえば、<PyObject section="execution" module="dagster" object="multiprocess_executor" /> は、デフォルトで、起動された実行で実行されるオペレーションの数を `multiprocessing.cpu_count()` の値に制限します。

## 別の実行が既に実行されている場合は実行を開始しないようにする（高度な設定）

Dagster の豊富なメタデータを使用することで、スケジュールやセンサーを使用して、現在実行中のジョブがない場合にのみ実行を開始できます。

<CodeExample
  path="docs_snippets/docs_snippets/guides/operate/concurrency-no-more-than-1-job.py"
  language="python"
  title="No more than 1 running job from a schedule"
/>

## トラブルシューティング

同時実行数を制限する場合、設定が適切になるまでは問題が発生する可能性があります。

### 実行は開始状態になり、QUEUED をスキップします

:::info
これは Dagster Open Source にのみ適用されます。
:::

`1.10.0` より前のバージョンを実行している場合は、インスタンスの設定で `run_queue` キーを設定して、実行キューを有効にするようにデプロイメントを手動で構成する必要がある場合があります。
Dagster UI で、**Deployment > Configuration** に移動し、`run_queue` キーが設定されていることを確認してください。

### QUEUED ステータスのまま実行が残っている

実行が QUEUED ステータスのままになっている原因は、Dagster+ を使用しているか、Dagster Open Source を使用しているかによって異なります。

<Tabs>
  <TabItem value="Dagster+" label="Dagster+">
    Dagster+ で実行がデキューされない場合、根本原因として以下のことが考えられます:
    * **[ハイブリッドデプロイメント](/dagster-plus/deployment/deployment-types/hybrid)** を使用している場合、デプロイメントを処理するエージェントがダウンしている可能性があります。この場合、実行は一時停止されます。
    * **Dagster+ でダウンタイムが発生しています**。[ステータスページ](https://dagstercloud.statuspage.io/) で、潜在的な停止に関する最新情報をご確認ください。

  </TabItem>
  <TabItem value="Dagster Open Source" label="Dagster オープンソース">
  Dagster Open Source で実行がキューから削除されない場合は、根本原因は Dagster デーモンまたは実行キューの構成に問題がある可能性があります。

**Dagsterデーモンのトラブルシューティング**

    * **Dagster デーモンがセットアップされ、実行されていることを確認します。** Dagster UI で、**[デプロイメント] > [デーモン]** に移動し、デーモンが実行中であることを確認します。**[実行キュー]** も実行されている必要があります。[dagster dev](/guides/operate/webserver) を使用して Dagster UI を起動した場合、デーモンは自動的に起動されているはずです。デーモンが実行されていない場合は、手順 2 に進みます。
    * **Dagster デーモンが Dagster Web サーバー プロセスと同じストレージにアクセスできることを確認します。** Web サーバー プロセスと Dagster デーモンはどちらも同じストレージにアクセスする必要があります。つまり、同じ `dagster.yaml` を使用する必要があります。ローカルでは、両方のプロセスで同じ `DAGSTER_HOME` 環境変数が設定されている必要があります。dagster dev を使用して Dagster UI を起動した場合、両方のプロセスが同じストレージを使用している必要があります。詳細については、[Dagster インスタンスのドキュメント](/guides/deploy/dagster-instance-configuration)を参照してください。

**実行キューの設定に関するトラブルシューティング**

デーモンが実行中の場合、同時実行ルールにより、実行が意図的にキューに残されることがあります。調査するには、以下の手順を実行してください:
    * **デーモンプロセスから記録された出力を確認してください**。スキップされた実行も含まれます。
    * **インスタンスの dagster.yaml で max_concurrent_runs 設定を確認してください**。0 に設定されていると、キューがブロックされる可能性があります。この設定は、Dagster UI で「デプロイメント」>「構成」に移動し、concurrency.runs.max_concurrent_runs 設定を見つけることで確認できます。詳細については、[同時に進行中の実行の合計数を制限する](#limit-the-number-of-total-runs-that-c​​an-be-in-progress-at-the-same-time)セクションを参照してください。
    * **実行キューの状態を確認してください**。場合によっては、進行中の実行によってキューがブロックされることがあります。実行キューのステータスを表示するには、Dagster UI の上部ナビゲーションで  **Runs** をクリックし、**Queued** タブと  **In Progress**  タブを開きます。

キューに入っている実行や進行中の実行によってキューがブロックされている場合は、それらを終了して他の実行を続行できるようにすることができます。
  </TabItem>
</Tabs>
