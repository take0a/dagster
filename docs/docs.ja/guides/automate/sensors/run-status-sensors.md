---
title: 実行ステータスに反応するセンサーの作成
sidebar_position: 500
---

実行のステータスに応じて操作する場合、Dagster は実行ステータスに反応するセンサーを作成する方法を提供します。<PyObject section="schedules-sensors" module="dagster" object="run_status_sensor" /> を指定された <PyObject section="internals" module="dagster" object="DagsterRunStatus" /> とともに使用して、特定のステータスが発生したときに実行される関数を装飾できます。これを使用して、他の実行を開始したり、実行の失敗時に監視サービスにアラートを送信したり、実行の成功を報告したりできます。

以下は、実行が成功した場合に `status_reporting_job` の実行を開始する実行ステータス センサーの例です:

<CodeExample path="docs_snippets/docs_snippets/concepts/partitions_schedules_sensors/sensors/run_status_run_requests.py" startAfter="start" endBefore="end" />

`request_job` は、`RunRequest` が返されたときに実行されるジョブです:

`report_status_sensor` では、条件付きで `RunRequest` を返すことに注意してください。これにより、`report_status_sensor` が `status_reporting_job` を実行するときに、`status_reporting_job` の成功によって `status_reporting_job` の別の実行がトリガーされ、それが別の実行をトリガーする、という無限ループに陥らないことが保証されます。

以下は、Slack メッセージでジョブの成功を報告するセンサーの例です:

<CodeExample path="docs_snippets/docs_snippets/concepts/partitions_schedules_sensors/sensors/sensor_alert.py" startAfter="start_success_sensor_marker" endBefore="end_success_sensor_marker" />

実行ステータス センサーが実行によってトリガーされたが何も返されない場合、Dagster はセンサーが実行されたことを示すイベントを実行に報告します。

センサーを記述したら、そのセンサーを <PyObject section="definitions" module="dagster" object="Definitions" /> オブジェクトに追加して、他のセンサーと同じように有効化して使用できるようになります。

<CodeExample path="docs_snippets/docs_snippets/concepts/partitions_schedules_sensors/sensors/sensor_alert.py" startAfter="start_definitions_marker" endBefore="end_definitions_marker" />
