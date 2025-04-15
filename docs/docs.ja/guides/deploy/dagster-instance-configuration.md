---
title: 'Dagsterインスタンス'
description: 'Define configuration options for your Dagster instance.'
sidebar_position: 200
---

:::note

この記事は、Dagster Open Source (OSS) デプロイメントに適用されます。
Dagster+ の詳細については、[Dagster+ ドキュメント](/dagster-plus/deployment/management/settings/customizing-agent-settings) を参照してください。

:::

Dagster インスタンスは、Dagster が単一のデプロイメントに必要な構成を定義します。例えば、過去の実行履歴と関連ログの保存場所、op コンピューティング関数からの生のログのストリーミング場所、新しい実行の開始方法などです。

Dagster デプロイメントを構成するすべてのプロセスとサービスは、情報を効率的に共有できるように、単一のインスタンス構成ファイル（「dagster.yaml」）を共有する必要があります。

:::warning

[実行の並列処理](/guides/operate/run-executors)などの重要な構成は、インスタンスではなくジョブごとに設定されます。

:::

## デフォルトのローカル動作

Dagster ウェブサーバーや Dagster CLI コマンドなどの Dagster プロセスが起動されると、Dagster はインスタンスのロードを試みます。
環境変数 `DAGSTER_HOME` が設定されている場合、Dagster は `$DAGSTER_HOME/dagster.yaml` にあるインスタンス設定ファイルを検索します。
このファイルには、インスタンスを構成する設定が含まれています。

`DAGSTER_HOME` が設定されていない場合、Dagster ツールは、プロセス終了時にクリーンアップされる一時ディレクトリをストレージとして使用します。
これは、Dagster を一時的なローカル開発やテストに使用する場合、結果の永続化を気にしない場合に便利です。

`DAGSTER_HOME` は設定されているが、`dagster.yaml` が存在しないか空の場合、Dagster は次のような構造のデータをローカルファイルシステムに永続化します:

```
$DAGSTER_HOME
├── dagster.yaml
├── history
│   ├── runs
│   │   ├── 00636713-98a9-461c-a9ac-d049407059cd.db
│   │   └── ...
│   └── runs.db
└── storage
    ├── 00636713-98a9-461c-a9ac-d049407059cd
    │   └── compute_logs
    │       └── ...
    └── ...
```

生成されるファイルとディレクトリの内訳は次のとおりです:

| File or directory             | Description                                                                                                                          |
| ----------------------------- | ------------------------------------------------------------------------------------------------------------------------------------ |
| history/                      |実行の履歴情報を含むディレクトリ。|
| history/runs.db               | 実行に関する情報が含まれる SQLite データベース ファイル。 |
| history/[run_id].db           | 実行ごとのイベント ログが含まれる SQLite データベース ファイル。 |
| storage/                      | 実行ごとに 1 つずつサブディレクトリのディレクトリ。 |
| storage/[run_id]/compute_logs | 各オペレーションの計算関数の実行からの `stdout` および `stderr` ログが含まれる実行固有のディレクトリ。 |

## 構成リファレンス

永続的な Dagster デプロイメントでは、通常、インスタンス上の多くのコンポーネントを設定する必要があります。
例えば、Postgres インスタンスを使用して実行とそれに対応するイベントログを保存し、コンピューティングログを Amazon S3 バケットにストリーミングしたいとします。

これを行うには、ウェブサーバーとその他のすべての Dagster ツールが起動時に参照する `$DAGSTER_HOME/dagster.yaml` ファイルを用意します。
このファイルでは、Dagster インスタンスのさまざまな側面を設定できます。具体的には、次のようなものがあります。

| Name                   | Key                      | Description                                                                                                                                     |
| ---------------------- | ------------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------- |
| Dagster storage        | `storage`                | ジョブとアセットの履歴をどのように保存するかを制御します。これには、実行、イベントログ、スケジュール/センサーティックのメタデータ、その他の有用なデータが含まれます。 |
| Run launcher           | `run_launcher`           | 実行される場所を決定します。                      |
| Run coordinator        | `run_coordinator`        | 実行の優先順位付けルールと同時実行制限を設定するために使用されるポリシーを決定します。         |
| Compute log storage    | `compute_logs`           |生の stdout および {" "} stderr ext ログのキャプチャと永続化を制御します。              |
| Local artifact storage | `local_artifact_storage` | ローカル ディスクを必要とするアーティファクトのストレージ、またはファイル システム I/O マネージャー ( ) を使用する場合のストレージを構成します。        |
| Telemetry              | `telemetry`              | Dagster による匿名の使用統計の収集をオプトイン/オプトアウトするために使用されます。                  |
| gRPC servers           | `code_servers`           | Dagster がコードの場所にコードをロードする方法を構成します。                                        |
| Data retention         | `data_retention`         | スケジュール/センサー ティック データなど、時間の経過とともに価値が減少する特定の種類のデータを Dagster が保持する期間を制御します。       |
| Sensor evaluation      | `sensors`                | センサーの評価方法を制御します。        |
| Schedule evaluation    | `schedules`              | スケジュールの評価方法を制御します。 |
| Auto-materialize       | `auto_materialize`       | アセットが自動的に具体化される方法を制御します。 |

:::note

YAML設定における環境変数は、リテラル文字列値の代わりに`env:`キーを使用することでサポートされます。
このリファレンスのサンプル設定には、環境変数を使用した例が含まれています。

:::

### Dagster ストレージ

`storage` キーを使用すると、ジョブとアセットの履歴の保存方法を設定できます。
これには、実行に関するメタデータ、イベントログ、スケジュール/センサーのティック、その他の有用なデータが含まれます。

利用可能なオプションとサンプル構成については、次のタブを参照してください。

<Tabs>
  <TabItem value="Sqlite storage (default)" label="Sqlite ストレージ (デフォルト)">

**Sqlite ストレージ (デフォルト)**

ストレージに SQLite データベースを使用するには、`dagster.yaml` で `storage.sqlite` を設定します:

<CodeExample
  path="docs_snippets/docs_snippets/deploying/dagster_instance/dagster.yaml"
  startAfter="start_marker_storage_sqlite"
  endBefore="end_marker_storage_sqlite"
/>

</TabItem>
<TabItem value="Postgres storage" label="Postgres ストレージ">

**Postgres ストレージ**

:::note

Postgres ストレージを使用するには、[dagster-postgres](/api/python-api/libraries/dagster-postgres) ライブラリをインストールする必要があります。

:::

ストレージに [PostgreSQL データベース](/api/python-api/libraries/dagster-postgres) を使用するには、`dagster.yaml` で `storage.postgres` を設定します。

<CodeExample
  path="docs_snippets/docs_snippets/deploying/dagster_instance/dagster.yaml"
  startAfter="start_marker_storage_postgres"
  endBefore="end_marker_storage_postgres"
/>

</TabItem>
<TabItem value="MySQL storage" label="MySQLストレージ">

**MySQLストレージ**

:::note

MySQL ストレージを使用するには、[dagster-mysql](/api/python-api/libraries/dagster-mysql) ライブラリをインストールする必要があります。

:::

ストレージに [MySQL データベース](/api/python-api/libraries/dagster-mysql) を使用するには、`dagster.yaml` で `storage.mysql` を設定します。

<CodeExample
  path="docs_snippets/docs_snippets/deploying/dagster_instance/dagster.yaml"
  startAfter="start_marker_storage_mysql"
  endBefore="end_marker_storage_mysql"
/>

</TabItem>
</Tabs>

### 実行ランチャー {#run-launcher}

`run_launcher` キーを使用すると、インスタンスの実行ランチャーを設定できます。
実行ランチャーは、実行場所を決定します。
Dagster が提供するオプションのいずれかを使用することも、独自のカスタム実行ランチャーを作成することもできます。
詳細については、「[実行ランチャー](/guides/deploy/execution/run-launchers)」をご覧ください。

使用可能なオプションとサンプル設定については、次のタブを参照してください。
データベースは UTC タイムゾーンを使用するように設定する必要があることに注意してください。

<Tabs>
<TabItem value="DefaultRunLauncher" label="DefaultRunLauncher (デフォルト)">

**DefaultRunLauncher**

<PyObject section="internals" module="dagster._core.launcher" object="DefaultRunLauncher" /> は、ジョブのコードの場所と同じノードに新しいプロセスを生成します。

<CodeExample
  path="docs_snippets/docs_snippets/deploying/dagster_instance/dagster.yaml"
  startAfter="start_marker_run_launcher_default"
  endBefore="end_marker_run_launcher_default"
/>

</TabItem>
<TabItem value="DockerRunLauncher" label="DockerRunLauncher">

**DockerRunLauncher**

<PyObject section="libraries" module="dagster_docker" object="DockerRunLauncher" /> は実行ごとに Docker コンテナを割り当てます。

<CodeExample
  path="docs_snippets/docs_snippets/deploying/dagster_instance/dagster.yaml"
  startAfter="start_marker_run_launcher_docker"
  endBefore="end_marker_run_launcher_docker"
/>

</TabItem>
<TabItem value="K8sRunLauncher" label="K8sRunLauncher">

**K8sRunLauncher**

<PyObject section="libraries" module="dagster_k8s" object="K8sRunLauncher" /> は、実行ごとに Kubernetes ジョブを割り当てます。

<CodeExample
  path="docs_snippets/docs_snippets/deploying/dagster_instance/dagster.yaml"
  startAfter="start_marker_run_launcher_k8s"
  endBefore="end_marker_run_launcher_k8s"
/>

</TabItem>
</Tabs>

### 実行コーディネーター

`run_coordinator` キーを使用すると、インスタンスの実行コーディネーターを設定できます。
実行コーディネーターは、実行の優先順位付けルールと同時実行制限を設定するために使用されるポリシーを決定します。
詳細とトラブルシューティングのヘルプについては、「[実行コーディネーター](/guides/deploy/execution/run-coordinators)」をご覧ください。

利用可能なオプションとサンプル設定については、次のタブを参照してください。

<Tabs>
<TabItem value="DefaultRunCoordinator (default)" label="DefaultRunCoordinator (デフォルト)">

**DefaultRunCoordinator (デフォルト)**

デフォルトの実行コーディネーターである <PyObject section="internals" module="dagster._core.run_coordinator" object="DefaultRunCoordinator" /> は、実行を直ちに [実行ランチャー](#run-launcher) に送信します。
`Queued` 実行という概念はありません。

<CodeExample
  path="docs_snippets/docs_snippets/deploying/dagster_instance/dagster.yaml"
  startAfter="start_marker_run_coordinator_default"
  endBefore="end_marker_run_coordinator_default"
/>

</TabItem>
<TabItem value="QueuedRunCoordinator" label="QueuedRunCoordinator">

**QueuedRunCoordinator**

<PyObject section="internals" module="dagster._core.run_coordinator" object="QueuedRunCoordinator" /> を使用すると、一度に実行できる実行の数に制限を設定できます。
**注** 実行を開始するには、アクティブな [dagster-daemon プロセス](/guides/deploy/execution/dagster-daemon) が必要です。

この実行コーディネータは、同時実行の総数を制限することと、実行タグに基づいて特定の制限を設定することの両方をサポートしています。
たとえば、スロットルを回避するために、特定のクラウド サービスとやり取りする実行に対して同時実行数の制限を指定できます。

<CodeExample
  path="docs_snippets/docs_snippets/deploying/dagster_instance/dagster.yaml"
  startAfter="start_marker_run_coordinator_queued"
  endBefore="end_marker_run_coordinator_queued"
/>

</TabItem>
</Tabs>

### コンピューティングログストレージ

`compute_logs` キーを使用すると、コンピューティングログのストレージを設定できます。
コンピューティングログのストレージは、生の `stdout` および `stderr` テキストログのキャプチャと保存を制御します。

利用可能なオプションとサンプル設定については、次のタブを参照してください。

<Tabs>
<TabItem value="LocalComputeLogManager (default)" label="LocalComputeLogManager (default)">

**LocalComputeLogManager**

デフォルトで使用される <PyObject section="internals" module="dagster._core.storage.local_compute_log_manager" object="LocalComputeLogManager" /> は、`stdout` および `stderr` ログをディスクに書き込みます。

<CodeExample
  path="docs_snippets/docs_snippets/deploying/dagster_instance/dagster.yaml"
  startAfter="start_marker_compute_log_storage_local"
  endBefore="end_marker_compute_log_storage_local"
/>

</TabItem>
<TabItem value="NoOpComputeLogManager" label="NoOpComputeLogManager">

**NoOpComputeLogManager**

<PyObject section="internals" module="dagster._core.storage.noop_compute_log_manager" object="NoOpComputeLogManager" /> は、どのステップでも `stdout` および `stderr` ログを保存しません。

<CodeExample
  path="docs_snippets/docs_snippets/deploying/dagster_instance/dagster.yaml"
  startAfter="start_marker_compute_log_storage_noop"
  endBefore="end_marker_compute_log_storage_noop"
/>

</TabItem>
<TabItem value="AzureBlobComputeLogManager" label="AzureBlobComputeLogManager">

**AzureBlobComputeLogManager**

<PyObject section="libraries" module="dagster_azure" object="blob.AzureBlobComputeLogManager" /> は、`stdout` と `stderr` を Azure Blob Storage に書き込みます。

<CodeExample
  path="docs_snippets/docs_snippets/deploying/dagster_instance/dagster.yaml"
  startAfter="start_marker_compute_log_storage_blob"
  endBefore="end_marker_compute_log_storage_blob"
/>

</TabItem>
<TabItem value="GCSComputeLogManager" label="GCSComputeLogManager">

**GCSComputeLogManager**

<PyObject section="libraries" module="dagster_gcp" object="gcs.GCSComputeLogManager" /> は、`stdout` と `stderr` を Google Cloud Storage に書き込みます。

<CodeExample
  path="docs_snippets/docs_snippets/deploying/dagster_instance/dagster.yaml"
  startAfter="start_marker_compute_log_storage_gcs"
  endBefore="end_marker_compute_log_storage_gcs"
/>

</TabItem>
<TabItem value="S3ComputeLogManager" label="S3ComputeLogManager">

**S3ComputeLogManager**

<PyObject section="libraries" module="dagster_aws" object="s3.S3ComputeLogManager" /> は、`stdout` と `stderr` を Amazon Web Services S3 バケットに書き込みます。

<CodeExample
  path="docs_snippets/docs_snippets/deploying/dagster_instance/dagster.yaml"
  startAfter="start_marker_compute_log_storage_s3"
  endBefore="end_marker_compute_log_storage_s3"
/>

</TabItem>
</Tabs>

### ローカルアーティファクトストレージ

`local_artifact_storage` キーを使用すると、ローカルアーティファクトストレージを設定できます。
ローカルアーティファクトストレージは、次の目的で使用されます。

- ローカルディスクを必要とするアーティファクト用のストレージを設定する。
- ファイルシステム I/O マネージャー (<PyObject section="io-managers" module="dagster" object="FilesystemIOManager" />) を使用する際に、入力と出力を保存する。他の I/O マネージャーがアーティファクトを保存する方法の詳細については、[I/O マネージャーのドキュメント](/guides/build/io-managers/) を参照してください。

:::note

<PyObject section="internals" module="dagster._core.storage.root" object="LocalArtifactStorage" />
は、現在、ローカルアーティファクトストレージの唯一のオプションです。
このオプションは、デフォルトのファイルシステムI/Oマネージャーが使用するディレクトリと、ローカルディスクを必要とするアーティファクトを構成します。

:::

<CodeExample
  path="docs_snippets/docs_snippets/deploying/dagster_instance/dagster.yaml"
  startAfter="start_marker_local_artifact_storage"
  endBefore="end_marker_local_artifact_storage"
/>

### テレメトリー

`telemetry` キーを使用すると、Dagster による匿名の使用状況統計の収集をオプトインまたはオプトアウトできます。
デフォルトでは `true` に設定されています。

<CodeExample
  path="docs_snippets/docs_snippets/deploying/dagster_instance/dagster.yaml"
  startAfter="start_marker_telemetry"
  endBefore="end_marker_telemetry"
/>

詳細については、[テレメトリのドキュメント](/about/telemetry)を参照してください。

### gRPC サーバー

`code_servers` キーを使用すると、Dagster が [コードの場所](/guides/deploy/code-locations/) にコードをロードする方法を設定できます。

[独自の gRPC サーバーを実行](/guides/deploy/code-locations/workspace-yaml#grpc-server) していない場合、ウェブサーバーと Dagster デーモンは、サブプロセスで実行されている gRPC サーバーからコードをロードします。

デフォルトでは、コードのロードに 180 秒以上かかる場合、Dagster はコードがハングしていると判断し、ロードの待機を停止します。

コードのロードに 180 秒以上かかると予想される場合は、`code_servers.local_startup_timeout` キーを設定してください。
この値は、最大タイムアウトを秒単位で示す整数である必要があります。

<CodeExample
  path="docs_snippets/docs_snippets/deploying/dagster_instance/dagster.yaml"
  startAfter="start_marker_code_servers"
  endBefore="end_marker_code_servers"
/>

### データ保持

`retention` キーを使用すると、Dagster が特定の種類のデータを保持する期間を設定できます。
具体的には、スケジュール/センサーティックデータなど、時間の経過とともに価値が減少するデータです。
古いティックをクリーンアップすることで、ストレージに関する懸念を最小限に抑え、クエリのパフォーマンスを向上させることができます。

デフォルトでは、Dagster はスキップされたセンサーティックを 7 日間、その他のすべてのティックタイプを無期限に保持します。
スケジュールとセンサーティックの保持ポリシーをカスタマイズするには、`purge_after_days` キーを使用します:

<CodeExample
  path="docs_snippets/docs_snippets/deploying/dagster_instance/dagster.yaml"
  startAfter="start_marker_retention"
  endBefore="end_marker_retention"
/>

`purge_after_days` キーには、以下のいずれかを指定できます:

- すべての種類のティックを保持する期間（日数）を示す単一の整数。
**注**: 値 `-1` は、ティックを無期限に保持します。
- ティックの種類（`skipped`、`failure`、`success`）と整数のマッピング。
これらの整数は、ティックの種類を保持する期間（日数）を示します。

### センサー評価

`sensors` キーを使用すると、センサーの評価方法を設定できます。
複数のセンサーを同時に並列に評価するには、`use_threads` キーと `num_workers` キーを設定します:

<CodeExample
  path="docs_snippets/docs_snippets/deploying/dagster_instance/dagster.yaml"
  startAfter="start_marker_sensors"
  endBefore="end_marker_sensors"
/>

オプションの `num_submit_workers` キーを設定して、同じセンサー ティックからの複数の実行リクエストを並行して評価することもできます。これにより、単一のセンサー ティックが多数の実行リクエストを返す場合のレイテンシを短縮できます。

### スケジュール評価

`schedules` キーを使用すると、スケジュールの評価方法を設定できます。
デフォルトでは、Dagster はスケジュールを 1 つずつ評価します。

複数のスケジュールを同時に並列に評価するには、`use_threads` キーと `num_workers` キーを設定します:

<CodeExample
  path="docs_snippets/docs_snippets/deploying/dagster_instance/dagster.yaml"
  startAfter="start_marker_schedules"
  endBefore="end_marker_schedules"
/>

オプションの `num_submit_workers` キーを設定して、同じスケジュール ティックからの複数の実行リクエストを並行して評価することもできます。これにより、単一のスケジュール ティックで多数の実行リクエストが返される場合のレイテンシを削減できます。
