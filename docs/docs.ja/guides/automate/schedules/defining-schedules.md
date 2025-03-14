---
title: スケジュールの定義
sidebar_position: 100
---

## 基本的なスケジュールの定義

次の例は、いくつかの基本的なスケジュールを定義する方法を示しています。

<Tabs>
  <TabItem value="ScheduleDefinition の使用">

この例では、<PyObject section="schedules-sensors" module="dagster" object="ScheduleDefinition" /> を使用して、毎日深夜にジョブを実行するスケジュールを定義する方法を示します。この例では op ジョブを使用していますが、[アセットジョブ](/guides/build/assets/asset-jobs) でも同じアプローチが機能します。

<CodeExample path="docs_snippets/docs_snippets/concepts/partitions_schedules_sensors/schedules/schedules.py" startAfter="start_basic_schedule" endBefore="end_basic_schedule" />

:::note

`cron_schedule` 引数は標準の [cron 式](https://en.wikipedia.org/wiki/Cron) を受け入れます。`croniter` 依存関係のバージョンが `>= 1.0.12` の場合、引数は次のものも受け入れます。
<ul><li>`@daily`</li><li>`@hourly`</li><li>`@monthly`</li></ul>

:::

</TabItem>
<TabItem value="@schedule の使用">

この例では、<PyObject section="schedules-sensors" module="dagster" object="schedule" decorator /> を使用してスケジュールを定義する方法を示します。これは、<PyObject section="schedules-sensors" module="dagster" object="ScheduleDefinition" /> よりも柔軟性が高くなります。たとえば、[スケジュールされた実行時間に基づいてジョブの動作を構成する](configuring-job-behavior) または [ログメッセージを出力](#emitting-log-messages-from-schedule-evaluation) することができます。

```python
@schedule(job=my_job, cron_schedule="0 0 * * *")
def basic_schedule(): ...
  # things the schedule does, like returning a RunRequest or SkipReason
```

:::note

`cron_schedule` 引数は標準の [cron 式](https://en.wikipedia.org/wiki/Cron) を受け入れます。`croniter` 依存関係のバージョンが `>= 1.0.12` の場合、引数は次のものも受け入れます。
<ul><li>`@daily`</li><li>`@hourly`</li><li>`@monthly`</li></ul>

:::

</TabItem>
</Tabs>

## スケジュール評価からのログメッセージの出力

この例では、評価関数の実行中にスケジュールからログ メッセージを出力する方法を示します。これらのログは、スケジュールのティック履歴でティックを検査するときに UI に表示されます。

<CodeExample path="docs_snippets/docs_snippets/concepts/partitions_schedules_sensors/schedules/schedules.py" startAfter="start_schedule_logging" endBefore="end_schedule_logging" />

:::note

スケジュール ログは、[Dagster インスタンスのコンピューティング ログ ストレージ](/guides/deploy/dagster-instance-configuration#compute-log-storage) に保存されます。コンピューティング ログ ストレージがスケジュール ログを表示するように構成されていることを確認する必要があります。

:::

