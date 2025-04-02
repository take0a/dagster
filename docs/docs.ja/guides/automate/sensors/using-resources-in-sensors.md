---
title: センサーのリソースの使用
sidebar_position: 100
---

Dagster の [リソース](/guides/build/external-resources/) システムをセンサーで使用すると、外部システムへの呼び出しが容易になり、テスト目的でセンサーのコンポーネントを簡単にプラグインできるようになります。

リソースの依存関係を指定するには、リソースをセンサーの関数のパラメータとして注釈付けします。リソースは、<PyObject section="definitions" module="dagster" object="Definitions" /> 呼び出しに添付することで提供されます。

ここでは、外部 API へのアクセスを提供するリソースが提供されています。同じリソースは、センサーがトリガーするジョブまたはアセットでも使用できます。

<CodeExample
  path="docs_snippets/docs_snippets/concepts/resources/pythonic_resources.py"
  startAfter="start_new_resource_on_sensor"
  endBefore="end_new_resource_on_sensor"
  dedent="4"
/>

リソースの詳細については、[リソースのドキュメント](/guides/build/external-resources)を参照してください。リソースを使用してスケジュールをテストする方法については、「[センサーのテスト](/guides/automate/sensors/testing-sensors)」のリソースを使用したセンサーのテストに関するセクションを参照してください。
