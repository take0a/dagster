---
title: 'dagster.yaml リファレンス'
sidebar_position: 300
---

`dagster.yaml` ファイルは、Dagster インスタンスを構成するために使用されます。
このファイルは、ストレージ、実行、ログ記録、その他 Dagster デプロイメントのさまざまな設定を定義します。

## ファイルの場所

デフォルトでは、Dagster は `DAGSTER_HOME` 環境変数で指定されたディレクトリ内の `dagster.yaml` ファイルを検索します。

## 環境変数の使用

環境変数を使用して、`dagster.yaml` ファイル内の値を上書きできます。

```yaml
instance:
  module: dagster.core.instance
  class: DagsterInstance
  config:
    some_key:
      env: ME_ENV_VAR
```

## 完全な構成仕様

利用可能なすべてのオプションを含む包括的な `dagster.yaml` 仕様は次のとおりです:

```yaml
local_artifact_storage:
  module: dagster.core.storage.root
  class: LocalArtifactStorage
  config:
    base_dir: '/path/to/dir'

compute_logs:
  module: dagster.core.storage.local_compute_log_manager
  class: LocalComputeLogManager
  config:
    base_dir: /path/to/compute/logs

# Alternatively, logs can be written to cloud storage providers like S3, GCS, and Azure blog storage. For example:
# compute_logs:
#   module: dagster_aws.s3.compute_log_manager
#   class: S3ComputeLogManager
#   config:
#     bucket: "mycorp-dagster-compute-logs"
#    prefix: "dagster-test-"

storage:
  sqlite:
    base_dir: /path/to/sqlite/storage
  # Or use postgres:
  # postgres:
  #   postgres_db:
  #     hostname: localhost
  #     username: dagster
  #     password: dagster
  #     db_name: dagster
  #     port: 5432

run_queue:
  max_concurrent_runs: 15
  tag_concurrency_limits:
    - key: 'database'
      value: 'redshift'
      limit: 4
    - key: 'dagster/backfill'
      limit: 10

run_storage:
  module: dagster.core.storage.runs
  class: SqliteRunStorage
  config:
    base_dir: /path/to/dagster/home/history

event_log_storage:
  module: dagster.core.storage.event_log
  class: SqliteEventLogStorage
  config:
    base_dir: /path/to/dagster/home/history

schedule_storage:
  module: dagster.core.storage.schedules
  class: SqliteScheduleStorage
  config:
    base_dir: /path/to/dagster/home/schedules

scheduler:
  module: dagster.core.scheduler
  class: DagsterDaemonScheduler

run_coordinator:
  module: dagster.core.run_coordinator
  class: DefaultRunCoordinator
  # class: QueuedRunCoordinator
  # config:
  #   max_concurrent_runs: 25
  #   tag_concurrency_limits:
  #     - key: "dagster/backfill"
  #       value: "true"
  #       limit: 1

run_launcher:
  module: dagster.core.launcher
  class: DefaultRunLauncher
  # module: dagster_docker
  # class: DockerRunLauncher
  # module: dagster_k8s.launcher
  # class: K8sRunLauncher
  # config:
  #   service_account_name: pipeline_run_service_account
  #   job_image: my_project/dagster_image:latest
  #   instance_config_map: dagster-instance
  #   postgres_password_secret: dagster-postgresql-secret

telemetry:
  enabled: true

run_monitoring:
  enabled: true
  poll_interval_seconds: 60

run_retries:
  enabled: true
  max_retries: 3
  retry_on_asset_or_op_failure: true

code_servers:
  local_startup_timeout: 360

secrets:
  my_secret:
    env: MY_SECRET_ENV_VAR

retention:
  schedule:
    purge_after_days: 90
  sensor:
    purge_after_days:
      skipped: 7
      failure: 30
      success: -1

sensors:
  use_threads: true
  num_workers: 8

schedules:
  use_threads: true
  num_workers: 8

auto_materialize:
  enabled: true
  minimum_interval_seconds: 3600
  run_tags:
    key: 'value'
  respect_materialization_data_versions: true
  max_tick_retries: 3
  use_sensors: false
  use_threads: false
  num_workers: 4
```

## 設定オプション

### `storage`

ジョブとアセットの履歴を保存する方法を構成します。

```yaml
storage:
  sqlite:
    base_dir: /path/to/sqlite/storage
```

オプション:

- `sqlite`: ストレージに SQLite を使用します
- `postgres`: ストレージに PostgreSQL を使用します (`dagster-postgres` ライブラリが必要です)
- `mysql`: ストレージに MySQL を使用します (`dagster-mysql` ライブラリが必要です)

### `run_launcher`

実行される場所を決定します。

```yaml
run_launcher:
  module: dagster.core.launcher
  class: DefaultRunLauncher
```

オプション:

- `DefaultRunLauncher`: ジョブのコードと同じノードに新しいプロセスを生成します。
- `DockerRunLauncher`: 実行ごとに Docker コンテナを割り当てます。
- `K8sRunLauncher`: 実行ごとに Kubernetes ジョブを割り当てます。

### `run_coordinator`

実行の優先順位ルールと同時実行制限を設定します。

```yaml
run_coordinator:
  module: dagster.core.run_coordinator
  class: QueuedRunCoordinator
  config:
    max_concurrent_runs: 25
```

オプション:

- `DefaultRunCoordinator`: 実行を直ちに実行ランチャーに送信します
- `QueuedRunCoordinator`: 同時実行数の制限を設定できます

### `compute_logs`

stdout および stderr ログのキャプチャと永続性を制御します。

```yaml
compute_logs:
  module: dagster.core.storage.local_compute_log_manager
  class: LocalComputeLogManager
  config:
    base_dir: /path/to/compute/logs
```

オプション:

- `LocalComputeLogManager`: ログをディスクに書き込みます
- `NoOpComputeLogManager`: ログを保存しません
- `AzureBlobComputeLogManager`: ログを Azure Blob Storage に書き込みます
- `GCSComputeLogManager`: ログを Google Cloud Storage に書き込みます
- `S3ComputeLogManager`: ログを AWS S3 に書き込みます

### `local_artifact_storage`

ローカル ディスクを必要とするアーティファクト用、またはファイル システム I/O マネージャーを使用する場合のストレージを構成します。

```yaml
local_artifact_storage:
  module: dagster.core.storage.root
  class: LocalArtifactStorage
  config:
    base_dir: /path/to/artifact/storage
```

### `telemetry`

Dagster が匿名の使用統計を収集するかどうかを制御します。

```yaml
telemetry:
  enabled: false
```

### `code_servers`

Dagster がコードの場所にコードをロードする方法を構成します。

```yaml
code_servers:
  local_startup_timeout: 360
```

### `retention`

Dagster が特定の種類のデータを保持する期間を設定します。

```yaml
retention:
  schedule:
    purge_after_days: 90
  sensor:
    purge_after_days:
      skipped: 7
      failure: 30
      success: -1
```

### `sensors`

センサーの評価方法を構成します。

```yaml
sensors:
  use_threads: true
  num_workers: 8
```

### `schedules`

スケジュールの評価方法を構成します。

```yaml
schedules:
  use_threads: true
  num_workers: 8
```

### `auto_materialize`

アセットの自動実体化を構成します。

```yaml
auto_materialize:
  enabled: true
  minimum_interval_seconds: 3600
  run_tags:
    key: 'value'
  respect_materialization_data_versions: true
  max_tick_retries: 3
  use_sensors: false
  use_threads: false
  num_workers: 4
```

オプション:

- `enabled`: 自動マテリアライゼーションの有効化 (boolean)
- `minimum_interval_seconds`: マテリアライゼーション間の最小間隔 (integer)
- `run_tags`: 自動マテリアライゼーション実行に適用するタグ (dictionary)
- `respect_materialization_data_versions`: マテリアライゼーション時にデータバージョンを尊重するかどうか (boolean)
- `max_tick_retries`: エラーが発生した自動マテリアライゼーション ティックごとの最大再試行回数 (integer、default: 3)
- `use_sensors`: 自動マテリアライゼーションにセンサーを使用するかどうか (boolean)
- `use_threads`: ティック処理にスレッドを使用するかどうか (boolean、default: false)
- `num_workers`: 複数の自動化ポリシーセンサーからのティックを並列処理するために使用するスレッド数 (integer)

### `concurrency`

操作のデフォルトの同時実行制限を構成します。

```yaml
concurrency:
  default_op_concurrency_limit: 10
```

オプション:

- `default_op_concurrency_limit`: 未設定の同時実行キーにおける同時操作のデフォルトの最大数 (整数)

## 参考資料

- [Dagsterインスタンス設定ソースコード](https://github.com/dagster-io/dagster/blob/master/python_modules/dagster/dagster/_core/instance/config.py)
