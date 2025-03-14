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

3. ページの右上隅にある **Test Sensor** ボタンをクリックします。

    ![Test sensor button](/images/guides/automate/sensors/test-sensor-button.png)

4. カーソル値を入力するよう求められます。センサーの既存のカーソル (事前に入力済み) を使用することも、別の値を入力することもできます。カーソルを使用しない場合は、このフィールドを空白のままにしておきます。

    ![Cursor value field](/images/guides/automate/sensors/provide-cursor-page.png)

5. **Evaluate** をクリックしてセンサーを起動します。実行リクエスト、スキップ理由、Python エラーなど、評価の結果を含むウィンドウが表示されます。

    ![Evaluation result page](/images/guides/automate/sensors/eval-result-page.png)

   実行が成功した場合は、生成された実行リクエストごとに、その実行リクエストによって生成された構成で事前スキャフォールディングされた Launchpad を開くことができます。また、評価から計算された新しいカーソル値が表示され、その値を保持するオプションも表示されます。

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


<CodeExample path="docs_snippets/docs_snippets/concepts/partitions_schedules_sensors/sensors/sensors.py" startAfter="start_sensor_testing" endBefore="end_sensor_testing" />

コンテキスト引数はセンサーで使用されなかったので、コンテキスト オブジェクトを提供する必要がないことに注意してください。ただし、コンテキスト オブジェクトが **必要な** 場合は、<PyObject section="schedules-sensors" module="dagster" object="build_sensor_context" /> を介して提供できます。`my_directory_sensor_cursor` の例をもう一度考えてみましょう:

<CodeExample path="docs_snippets/docs_snippets/concepts/partitions_schedules_sensors/sensors/sensors.py" startAfter="start_cursor_sensors_marker" endBefore="end_cursor_sensors_marker" />

このセンサーは `context` 引数を使用します。これを呼び出すには、次の引数を指定する必要があります:

<CodeExample path="docs_snippets/docs_snippets/concepts/partitions_schedules_sensors/sensors/sensors.py" startAfter="start_sensor_testing_with_context" endBefore="end_sensor_testing_with_context" />

**リソースを使ったセンサーのテスト**

[リソース](/guides/build/external-resources/)を利用するセンサーの場合、センサー機能を呼び出すときに必要なリソースを提供できます。

以下は、「[センサーでのリソースの使用](using-resources-in-sensors)」で定義した、`users_api` リソースを使用する `process_new_users_sensor` のテストです。

{/* TODO add dedent=4 prop to CodeExample below when implemented */}
<CodeExample path="docs_snippets/docs_snippets/concepts/resources/pythonic_resources.py" startAfter="start_test_resource_on_sensor" endBefore="end_test_resource_on_sensor" />

</TabItem>
</Tabs>