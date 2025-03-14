---
title: "テストスケジュール"
sidebar_position: 600
---

この記事では、Dagster UI と Python を使用して [スケジュール](index.md) をテストする方法を説明します。

## Dagster UI でのテストスケジュール

UI を使用すると、スケジュールのテスト評価を手動でトリガーし、結果を表示できます。これは、[スケジュールを作成する](スケジュールの定義) ときや [予期しないスケジュール動作のトラブルシューティング](スケジュールのトラブルシューティング) に役立ちます。

1. UI で **Overview > Schedules tab** をクリックする

2. テストしたいスケジュールをクリックする

3. ページの右上隅にある **Test Schedule** ボタンをクリックします。

4. 模擬スケジュールの評価時間を選択するように求められます。スケジュールは周期に基づいて定義されるため、ドロップダウンの評価時間はその周期に沿った過去と将来の時間になります。

   たとえば、`"毎日 X 時に"` という周期でスケジュールをテストしているとします。ドロップダウンには、その周期に沿った過去と将来の評価時間が表示されます。

    ![Selecting a mock evaluation time for a schedule in the Dagster UI](/images/guides/automate/schedules/testing-select-timestamp-page.png)

5. 評価時間を選択したら、**Evaluate** ボタンをクリックします。

テストが完了すると、評価結果を含むウィンドウが表示されます。評価が成功した場合は、**Open in Launchpad** をクリックして、テスト評価と同じ構成で実行を開始します。

## Python でのスケジュールのテスト

スケジュールを Python で直接テストすることもできます。このセクションでは、次のテスト方法を説明します:

- [`@schedule` で装飾された関数](#testing-schedule-decorated-functions)
- [リソースを含むスケジュール](#testing-schedules-with-resources)

### @schedule で装飾された関数のテスト

<PyObject section="schedules-sensors" module="dagster" object="schedule" decorator /> デコレータでデコレートされた関数をテストするには、スケジュール定義を通常の Python 関数のように呼び出すことができます。呼び出しにより実行構成が返され、<PyObject section="execution" module="dagster" object="validate_run_config" /> 関数を使用して検証できます。

この例では、`configurable_job_schedule` をテストしましょう:

<CodeExample path="docs_snippets/docs_snippets/concepts/partitions_schedules_sensors/schedules/schedules.py" startAfter="start_run_config_schedule" endBefore="end_run_config_schedule" />

このスケジュールをテストするために、<PyObject section="schedules-sensors" module="dagster" object="build_schedule_context" /> を使用して、`context` パラメータに提供する <PyObject section="schedules-sensors" module="dagster" object="ScheduleEvaluationContext" /> を構築しました:

<CodeExample path="docs_snippets/docs_snippets/concepts/partitions_schedules_sensors/schedules/schedule_examples.py" startAfter="start_test_cron_schedule_context" endBefore="end_test_cron_schedule_context" />

<PyObject section="schedules-sensors" module="dagster" object="schedule" decorator /> で装飾された関数にコンテキストパラメータがない場合、その関数を呼び出すときにコンテキストパラメータを指定する必要はありません。

### リソースを使用したテストスケジュール

[リソース](/guides/build/external-resources)を利用するスケジュールの場合は、スケジュール関数を呼び出すときにリソースを提供できます。

この例では、`process_data_schedule` をテストするとします:

{/* TODO add dedent=4 prop to CodeExample below when implemented */}
<CodeExample path="docs_snippets/docs_snippets/concepts/resources/pythonic_resources.py" startAfter="start_new_resource_on_schedule" endBefore="end_new_resource_on_schedule" />

このスケジュールのテストでは、関数を呼び出すときにスケジュールに `date_formatter` リソースを提供しました:

{/* TODO add dedent=4 prop to CodeExample below when implemented */}
<CodeExample path="docs_snippets/docs_snippets/concepts/resources/pythonic_resources.py" startAfter="start_test_resource_on_schedule" endBefore="end_test_resource_on_schedule" />

## このガイドのAPI

| 名前                                            | 説明  |
| ----------------------------------------------- | --------------------------------- |
| <PyObject section="schedules-sensors" module="dagster" object="schedule" decorator />        | 指定された cron スケジュールに従って実行されるスケジュールを定義するデコレータ。   |
| <PyObject section="execution" module="dagster" object="validate_run_config" />       | 提供された実行構成 BLOB をジョブに対して検証する関数。.                   |
| <PyObject section="schedules-sensors" module="dagster" object="build_schedule_context" />    | 通常はテストに使用される `ScheduleEvaluationContext` を構築する関数。 |
| <PyObject section="schedules-sensors" module="dagster" object="ScheduleEvaluationContext" /> | スケジュール定義の実行関数に渡されるコンテキスト。                    |
