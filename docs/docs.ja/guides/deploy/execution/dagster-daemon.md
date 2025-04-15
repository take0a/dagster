---
title: 'Dagster デーモン'
sidebar_position: 100
---

[スケジュール](/guides/automate/schedules/)、[センサー](/guides/automate/sensors/)、[実行キューイング](/guides/deploy/execution/customizing-run-queue-priority)などのいくつかの Dagster 機能では、デプロイメントに長時間実行される `dagster-daemon` プロセスを含める必要があります。

## デーモンの起動

- [ローカルで実行](#running-locally)
- [デーモンをデプロイ](#deploying-the-daemon)

### ローカルで実行 {#running-locally}

<Tabs>
  <TabItem value="Running the daemon and webserver" label="デーモンとウェブサーバーの実行">

Dagster デーモンをローカルで実行する最も簡単な方法は、`dagster dev` コマンドを実行することです:

```shell
dagster dev
```

このコマンドは、DagsterウェブサーバーとDagsterデーモンの両方を起動し、コマンドラインからDagsterの完全なローカルデプロイメントを開始できるようにします。
`dagster dev`の詳細については、「[Dagsterをローカルで実行する](/guides/deploy/deployment-options/running-dagster-locally)」を参照してください。

  </TabItem>
  <TabItem value="Running only the daemon" label="デーモンのみ実行">

Dagster デーモンを単独で実行するには:

```shell
dagster-daemon run
```

このコマンドは、コードの場所を指定するために `dagster dev` と同じ引数を取ります。

  </TabItem>
</Tabs>

### デーモンをデプロイ {#deploying-the-daemon}

Docker や Kubernetes などの環境にデーモンをデプロイする方法については、[デプロイ オプションのドキュメント](/guides/deploy/deployment-options/)を参照してください。

## 利用可能なデーモン

`dagster-daemon` プロセスは、[Dagster インスタンス](/guides/deploy/dagster-instance-configuration) ファイルを読み取って、どのデーモンを含めるかを決定します。
含められた各デーモンは、それぞれのスレッドで定期的に実行されます。

現在利用可能なデーモンは次のとおりです:

| Name                  | Description                                                                                        | Enabled by                                                                                                                                                                                        |
| --------------------- | -------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Scheduler daemon      | アクティブなスケジュールから実行を作成します     | デフォルトの <PyObject section="schedules-sensors" module="dagster._core.scheduler" object="DagsterDaemonScheduler"/> がインスタンスのスケジューラとして上書きされない限り、有効/実行されます。 |
| Run queue daemon      | インスタンスに設定されている制限と優先順位ルールを考慮して、キューに入れられた実行を開始します。 | インスタンスに[実行コーディネーター](/guides/deploy/execution/run-coordinators)を設定します <PyObject section="internals" module="dagster._core.run_coordinator" object="QueuedRunCoordinator" />。  |
| Sensor daemon         | オンになっているアクティブな[センサー](/guides/automate/sensors/)から実行を作成します  | 常に有効です。                                                                   |
| Run monitoring daemon | [ワーカーの実行](/guides/deploy/oss-deployment-architecture#job-execution-flow)の失敗を処理します  | インスタンスの「run_monitoring」フィールドを使用します。詳細については、「[実行モニタリング](/guides/deploy/execution/run-monitoring)」をご覧ください。   |

デーモンが [ワークスペースファイル](/guides/deploy/code-locations/workspace-yaml) を使用して [コードの場所](/guides/deploy/code-locations/) をロードするように設定されている場合、ファイルは定期的に再ロードされることに注意してください。
つまり、ワークスペースファイルが変更されても `dagster-daemon` プロセスを再起動する必要はありません。

## Dagster UI でデーモンのステータスを確認する

UI で `dagster-daemon` プロセスのステータスを確認するには:

1. 上部のナビゲーションで、**Deployment** をクリックします。
2. **Daemons** タブをクリックします。

このタブには、インスタンスで現在設定されているすべてのデーモンに関する情報が表示されます。

各デーモンは、定期的にインスタンスのストレージにハートビートを書き込みます。
デーモンに最近のハートビートが表示されない場合は、`dagster-daemon` プロセスのログでエラーを確認してください。
