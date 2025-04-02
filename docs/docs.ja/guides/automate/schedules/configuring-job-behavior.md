---
title: スケジュールされた実行時間に基づいてジョブの動作を構成する
sidebar_position: 200
---

この例では、実行構成を使用して、スケジュールされた実行時間に基づいてジョブの動作を変更する方法を示します。

<CodeExample
  path="docs_snippets/docs_snippets/concepts/partitions_schedules_sensors/schedules/schedules.py"
  startAfter="start_run_config_schedule"
  endBefore="end_run_config_schedule"
/>

## APIs in this example

- <PyObject section="ops" module="dagster" object="op" decorator />
- <PyObject section="jobs" module="dagster" object="job" decorator />
- <PyObject section="execution" module="dagster" object="OpExecutionContext" />
- <PyObject section="schedules-sensors" object="ScheduleEvaluationContext" />
- <PyObject section="schedules-sensors" module="dagster" object="RunRequest" />
