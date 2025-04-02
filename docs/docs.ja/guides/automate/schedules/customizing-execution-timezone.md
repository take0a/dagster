---
title: スケジュールの実行タイムゾーンのカスタマイズ
sidebar_position: 300
---

タイムゾーンが設定されていないスケジュールは、デフォルトでは [UTC](https://en.wikipedia.org/wiki/Coordinated_Universal_Time) で実行されます。このガイドを読み終えると、次のことが分かるようになります:

- スケジュール定義にカスタムタイムゾーンを設定する
- パーティション化されたジョブにカスタムタイムゾーンを設定する
- 夏時間がスケジュール実行時間に与える影響を考慮する

:::note

このガイドでは、以下の知識があることを前提としています:

- スケジュール
- ジョブで[アセット](/guides/build/jobs/asset-jobs)ベースまたはopベースのいずれか
- [パーティション](/guides/build/partitions-and-backfills/partitioning-assets)

:::

## スケジュール定義にタイムゾーンを設定する

`execution_timezone` パラメータを使用すると、次のオブジェクトのスケジュールのタイムゾーンを指定できます:

- <PyObject section="schedules-sensors" module="dagster" object="schedule" decorator />
- <PyObject section="schedules-sensors" module="dagster" object="ScheduleDefinition" />
- <PyObject section="libraries" object="build_schedule_from_dbt_selection" module="dagster_dbt" />

このパラメータは、任意の [`tz` タイムゾーン](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones) を受け入れます。たとえば、次のスケジュールは、**米国太平洋時間 (America/Los_Angeles) で毎日午前 9 時に実行されます**。

<CodeExample
  path="docs_snippets/docs_snippets/concepts/partitions_schedules_sensors/schedules/schedules.py"
  startAfter="start_timezone"
  endBefore="end_timezone"
/>

## パーティションジョブのタイムゾーンの設定

パーティション化されたジョブから構築されたスケジュールは、パーティションの設定で定義されたタイムゾーンで実行されます。パーティション定義には `timezone` パラメータがあり、任意の [`tz` タイムゾーン](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones) を受け入れます。

たとえば、次のパーティションは **米国太平洋 (America/Los_Angeles)** タイムゾーンを使用します:

<CodeExample path="docs_snippets/docs_snippets/concepts/partitions_schedules_sensors/partition_with_timezone.py" />

## 実行時間と夏時間

夏時間 (DST) の開始と終了時には、スケジュールの実行時間に何らかの影響が出る可能性があります。

### 日次のスケジュールへの影響

DST 移行のため、スケジュールされた間隔ごとに存在しない実行時間を指定する可能性があります。

**毎日午前 2:30 に実行されるスケジュール** があるとします。DST が開始する日には、時刻が午前 2:00 から午前 3:00 にジャンプするため、午前 2:30 という時刻は存在しなくなります。

Dagster は代わりに、次の存在する時刻、つまり午前 3 時にスケジュールを実行します。

```markdown
# 夏時間開始: 午前2時に時間が1時間進みます

- 12:30 AM
- 1:00 AM
- 1:30 AM
- 3:00 AM ## time transition; schedule executes
- 3:30 AM
- 4:00 AM
```

毎年 1 日に 2 回存在する実行時間を指定することもできます。

**毎日午前 1:30 に実行されるスケジュール** があるとします。DST が終了する日には、午前 1:00 から午前 2:00 までの時間が繰り返されるため、午前 1:30 の時間が 2 回存在することになります。つまり、スケジュールが実行される可能性のある時間は 2 回あることになります。

この場合、Dagster は 2 回目の反復である午前 1:30 にスケジュールを実行します。

```markdown
# 夏時間終了: 午前2時に時間が1時間戻ります

- 12:30 AM
- 1:00 AM
- 1:30 AM
- 1:00 AM ## time transition
- 1:30 AM ## schedule executes
- 2:00 AM
```

### 時間ごとのスケジュールへの影響

時間ごとのスケジュールは、夏時間への移行の影響を受けません。DST が終了し、午前 1 時から午前 2 時までの時間が繰り返されても、スケジュールは 1 時間に 1 回だけ実行され続けます。

**毎時 30 分に実行されるスケジュール** があるとします。DST が終了する日には、スケジュールは午前 12:30 に実行され、午前 1:30 の両方のインスタンスが実行され、午前 2:30 に通常どおり続行されます:

```markdown
# 夏時間終了: 午前2時に時間が1時間戻ります

- 12:30 AM ## schedule executes
- 1:00 AM
- 1:30 AM ## schedule executes
- 1:00 AM ## time transition
- 1:30 AM ## schedule executes
- 2:00 AM
- 2:30 AM ## schedule executes
```

## このガイドのAPI

| Name                                                                         | Description                                                                                         |
| ---------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------- |
| <PyObject section="schedules-sensors" module="dagster" object="schedule" decorator />         | 指定された cron スケジュールに従って実行されるスケジュールを定義するデコレータ。     |
| <PyObject section="schedules-sensors" module="dagster" object="ScheduleDefinition" />                                     | スケジュールのクラス。    |
| <PyObject section="schedules-sensors" module="dagster"  object="build_schedule_from_partitioned_job" />  | パーティション化されたジョブのパーティション分割と一致する間隔を持つスケジュールを構築する関数。 |
| <PyObject section="libraries" object="build_schedule_from_dbt_selection" module="dagster_dbt" /> | 指定された dbt リソースのセットを実体化するスケジュールを構築する関数。           |
