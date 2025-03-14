---
title: センサーのログ
sidebar_position: 200
---

どのセンサーも、評価関数の実行中にログ メッセージを出力できます:

<CodeExample path="docs_snippets/docs_snippets/concepts/partitions_schedules_sensors/sensors/sensors.py" startAfter="start_sensor_logging" endBefore="end_sensor_logging" />

これらのログは、対応するセンサー ページのティック履歴ビューでティックを検査するときに表示できます。

:::note

センサー ログは、Dagster インスタンスのコンピューティング ログ ストレージに保存されます。コンピューティング ログ ストレージがセンサー ログを表示するように構成されていることを確認する必要があります。詳細については、「[Dagster インスタンスの構成](/guides/deploy/dagster-instance-configuration#compute-log-storage)」を参照してください。

:::
