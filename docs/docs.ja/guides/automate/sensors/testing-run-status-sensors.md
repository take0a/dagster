---
title: 実行ステータスセンサーのテスト
sidebar_position: 600
---

他のセンサーと同様に、実行ステータス センサーを直接呼び出すことができます。ただし、<PyObject section="schedules-sensors" module="dagster" object="run_status_sensor" /> および <PyObject section="schedules-sensors" module="dagster" object="run_failure_sensor" /> を介して提供される `context` には、通常は実行時にのみ使用できるオブジェクトが含まれています。以下に、ユニット テストで関数を直接呼び出すことができるようにコンテキストを構築する方法を示すコード スニペットを示します。

次のようなステータス センサーを記述した場合 (関数 `email_alert` を他の場所に実装したと仮定):


<CodeExample path="docs_snippets/docs_snippets/concepts/partitions_schedules_sensors/sensors/sensor_alert.py" startAfter="start_simple_success_sensor" endBefore="end_simple_success_sensor" />

まず、成功する簡単なジョブを記述します:

<CodeExample path="docs_snippets/docs_snippets/concepts/partitions_schedules_sensors/sensors/sensor_alert.py" startAfter="start_run_status_sensor_testing_with_context_setup" endBefore="end_run_status_sensor_testing_with_context_setup" />

次に、このジョブを実行して、`context` を構築するために必要な属性を取得します。正しいコンテキスト オブジェクトを返す関数 <PyObject section="schedules-sensors" module="dagster" object="build_run_status_sensor_context" /> を提供します。

<CodeExample path="docs_snippets/docs_snippets/concepts/partitions_schedules_sensors/sensors/sensor_alert.py" startAfter="start_run_status_sensor_testing_marker" endBefore="end_run_status_sensor_testing_marker" />

{/* TODO the methods and statuses below do not exist in API docs
We have provided convenience functions <PyObject section="execution" module="dagster" object="ExecuteInProcessResult" method="get_job_success_event" /> and <PyObject section="execution" module="dagster" object="ExecuteInProcessResult" method="get_job_failure_event" /> for retrieving `DagsterRunStatus.SUCCESS` and `DagsterRunStatus.FAILURE` events, respectively. If you have a run status sensor triggered on another status, you can retrieve all events from `result` and filter based on your event type.
*/}

同じパターンを使用して、<PyObject section="schedules-sensors" module="dagster" object="run_failure_sensor" /> のコンテキストを構築できます。この実行失敗センサーをテストする場合は、次のようにします:

<CodeExample path="docs_snippets/docs_snippets/concepts/partitions_schedules_sensors/sensors/sensor_alert.py" startAfter="start_simple_fail_sensor" endBefore="end_simple_fail_sensor" />

まず、失敗する単純なジョブを作成する必要があります:

<CodeExample path="docs_snippets/docs_snippets/concepts/partitions_schedules_sensors/sensors/sensor_alert.py" startAfter="start_failure_sensor_testing_with_context_setup" endBefore="end_failure_sensor_testing_with_context_setup" />

次に、ジョブを実行してコンテキストを作成します:

<CodeExample path="docs_snippets/docs_snippets/concepts/partitions_schedules_sensors/sensors/sensor_alert.py" startAfter="start_alert_sensor_testing_with_context_marker" endBefore="end_alert_sensor_testing_with_context_marker" />

`context` を作成した後の追加の関数呼び出し <PyObject section="schedules-sensors" module="dagster" object="RunStatusSensorContext" method="for_run_failure" /> に注意してください。 <PyObject section="schedules-sensors" module="dagster" object="run_failure_sensor" /> によって提供される `context` は、 <PyObject section="schedules-sensors" module="dagster" object="run_status_sensor" /> によって提供されるコンテキストのサブクラスであり、この追加の呼び出しを使用して構築できます。
