---
title: センサーのテスト
sidebar_position: 300
---

<Tabs>
<TabItem value="Dagster UI で">

**Dagster UI で**

:::note

**テストする前に:** センサーのテスト評価では、センサーの基盤となる Python 関数が実行されます。つまり、そのセンサーの関数内に含まれる副作用が実行される可能性があるということです。

:::

UI では、センサーのテスト評価を手動でトリガーし、結果を表示できます。

1. **Overview > Sensors** をクリックする

2. テストするセンサーをクリックする。

3. ページの右上隅にある **Preview tick result** ボタンをクリックします。

    ![Test sensor button](/images/guides/automate/sensors/test-sensor-button.png)

4. カーソル値を（オプションで）入力するよう求められます。センサーの既存のカーソル (事前に入力済み) を使用することも、別の値を入力することもできます。カーソルを使用しない場合は、このフィールドを空白のままにしておきます。

    ![Cursor value field](/images/guides/automate/sensors/provide-cursor-page.png)

5. **Continue** をクリックしてセンサーを起動します。実行リクエスト、スキップ理由、Python エラーなど、評価の結果を含むウィンドウが表示されます。

    ![Evaluation result page](/images/guides/automate/sensors/eval-result-page.png)

6. If the preview was successful, then for each produced run request, you can view the run config and tags produced by that run request by clicking the **{}** button in the Actions column.

    ![Actions page in the Dagster UI](/images/guides/automate/sensors/actions-page.png)

7. Click the **Launch all & commit tick result** on the bottom right to launch all the run requests. It will launch the runs and link to the /runs page filtered to the IDs of the runs that launched.

    ![Runs page after launching all runs in the Dagster UI](/images/guides/automate/sensors/launch-all-page.png)

</TabItem>
<TabItem value="CLI で">

**CLI で**

既存のセンサーが評価時に生成するものをすばやくプレビューするには、次のコマンドを実行します:

```shell
dagster sensor preview my_sensor_name
```

</TabItem>
<TabItem value="Python で">

**Python で**

センサーを単体テストするには、センサーの Python 関数を直接呼び出すことができます。これにより、センサーによって生成されたすべての実行要求が返されます。返された実行要求から取得された構成は、<PyObject section="execution" module="dagster" object="validate_run_config" /> 関数を使用して検証できます:

<CodeExample
  path="docs_snippets/docs_snippets/concepts/partitions_schedules_sensors/sensors/sensors.py"
  startAfter="start_sensor_testing"
  endBefore="end_sensor_testing"
/>

コンテキスト引数はセンサーで使用されなかったので、コンテキスト オブジェクトを提供する必要がないことに注意してください。ただし、コンテキスト オブジェクトが **必要な** 場合は、<PyObject section="schedules-sensors" module="dagster" object="build_sensor_context" /> を介して提供できます。`my_directory_sensor_cursor` の例をもう一度考えてみましょう:

<CodeExample
  path="docs_snippets/docs_snippets/concepts/partitions_schedules_sensors/sensors/sensors.py"
  startAfter="start_cursor_sensors_marker"
  endBefore="end_cursor_sensors_marker"
/>

このセンサーは `context` 引数を使用します。これを呼び出すには、次の引数を指定する必要があります:

<CodeExample
  path="docs_snippets/docs_snippets/concepts/partitions_schedules_sensors/sensors/sensors.py"
  startAfter="start_sensor_testing_with_context"
  endBefore="end_sensor_testing_with_context"
/>

**リソースを使ったセンサーのテスト**

[リソース](/guides/build/external-resources/)を利用するセンサーの場合、センサー機能を呼び出すときに必要なリソースを提供できます。

以下は、「[センサーでのリソースの使用](/guides/automate/sensors/using-resources-in-sensors)」で定義した、`users_api` リソースを使用する `process_new_users_sensor` のテストです。

<CodeExample
  path="docs_snippets/docs_snippets/concepts/resources/pythonic_resources.py"
  startAfter="start_test_resource_on_sensor"
  endBefore="end_test_resource_on_sensor"
  dedent="4"
/>

</TabItem>
</Tabs>
