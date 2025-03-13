---
title: リソースの定義
sidebar_position: 100
---

通常、リソースは <PyObject section="resources" module="dagster" object="ConfigurableResource"/> をサブクラス化することで定義されます。リソースには通常、一連の [構成値](/guides/operate/configuration/run-configuration) があり、外部ツールやサービスとインターフェイスするときにアカウント識別子、API キー、データベース名などの情報を指定するために使用されます。この構成スキーマは、クラスの属性によって指定されます。

設定システムには、単純な Python パラメータ渡しに比べていくつかの利点があります。設定可能な値は次の通りです:

- Dagster UI の Launchpad、GraphQL API、または Python API で実行開始時に置き換えられる
- Dagster UIに表示される
- 環境変数を使用して動的に設定し、実行時に解決する

## アセット定義で

次の例は、外部サービスへの接続を表す <PyObject section="resources" module="dagster" object="ConfigurableResource"/> のサブクラスを定義する方法を示しています。リソースは、<PyObject section="definitions" module="dagster" object="Definitions" /> 呼び出し内で構築することで構成できます。

構成値に依存するリソース クラスにメソッドを定義できます。

{/* TODO add dedent=4 prop when implemented */}
<CodeExample path="docs_snippets/docs_snippets/concepts/resources/pythonic_resources.py" startAfter="start_new_resources_configurable_defs" endBefore="end_new_resources_configurable_defs" />

アセットは、アセット関数のパラメータとしてリソースに注釈を付けることで、リソースの依存関係を指定します。

アセットにリソース値を提供するには、それらを <PyObject section="definitions" module="dagster" object="Definitions" /> オブジェクトに添付します。これらのリソースは、実行時に関数に自動的に渡されます。

## センサーで

[センサー](/guides/automate/sensors/) は、アセットと同じ方法でリソースを使用するため、外部サービスにデータを照会する場合に役立ちます。

センサーのリソース依存関係を指定するには、リソース タイプをセンサーの機能のパラメーターとして注釈付けします。詳細と例については、[センサーのドキュメント](/guides/automate/sensors/using-resources-in-sensors) を参照してください。

## スケジュールで

[スケジュール](/guides/automate/schedules) では、スケジュール ロジックを外部ツールと連携させる必要がある場合や、スケジュール ロジックをよりテスト可能にする必要がある場合に、リソースを使用できます。

スケジュールのリソース依存関係を指定するには、スケジュールの関数のパラメータとしてリソース タイプに注釈を付けます。詳細については、「[スケジュールでのリソースの使用](/guides/automate/schedules/using-resources-in-schedules)」を参照してください。

## ジョブで

次の例では、外部サービスへの接続を表す <PyObject section="resources" module="dagster" object="ConfigurableResource"/> のサブクラスを定義します。リソースは、<PyObject section="definitions" module="dagster" object="Definitions" /> 呼び出し内で構築することで構成できます。

構成値に依存するリソース クラスにメソッドを定義できます。

{/* TODO add dedent=4 prop when implemented */}
<CodeExample path="docs_snippets/docs_snippets/concepts/resources/pythonic_resources.py" startAfter="start_new_resources_configurable_defs_ops" endBefore="end_new_resources_configurable_defs_ops" />

リソースを定義するときに使用できる、サポートされている構成タイプが多数あります。使用可能な構成タイプのより包括的な概要については、[高度な構成タイプのドキュメント](/guides/operate/configuration/advanced-config-types)を参照してください。
