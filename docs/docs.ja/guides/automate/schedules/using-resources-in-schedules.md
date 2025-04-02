---
title: スケジュールでのリソースの使用
sidebar_position: 500
---

この例では、スケジュールでリソースを使用する方法を示します。リソースの依存関係を指定するには、スケジュールの関数のパラメーターとしてリソースに注釈を付けます。

:::note

この記事では、[リソース](/guides/build/external-resources/)、[コードの場所と定義](/guides/deploy/code-locations/)、[テストのスケジュール](/guides/automate/schedules/testing-schedules)に関する知識があることを前提としています。

スケジュールやリソースを含むすべての Dagster 定義は、<PyObject section="definitions" module="dagster" object="Definitions" /> 呼び出しに添付する必要があります。

:::

<CodeExample
  path="docs_snippets/docs_snippets/concepts/resources/pythonic_resources.py"
  startAfter="start_new_resource_on_schedule"
  endBefore="end_new_resource_on_schedule"
  dedent="4"
/>

## このガイドのAPI

| 名前 | 説明 |
|------|-------------|
| <PyObject section="schedules-sensors" module="dagster" object="schedule" decorator /> | 指定された cron スケジュールに従って実行されるスケジュールを定義するデコレータ。 |
| <PyObject section="resources" module="dagster" object="ConfigurableResource" /> | |
| <PyObject section="jobs" module="dagster" object="job" decorator /> | ジョブを定義するために使用されるデコレータ。 |
| <PyObject section="schedules-sensors" module="dagster" object="RunRequest" />                          | 1 回の実行を開始するために必要なすべての情報を表すクラス。 |
| <PyObject section="config" module="dagster" object="RunConfig" /> | |
| <PyObject section="definitions" module="dagster" object="Definitions" /> | |
