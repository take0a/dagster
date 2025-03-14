---
title: "実行構成"
description: Job run configuration allows providing parameters to jobs at the time they're executed.
sidebar_position: 100
---

実行構成を使用すると、ジョブの実行時にパラメータを提供できます。

実行時に Dagster ジョブまたはアセット定義にユーザーが選択した値を提供すると便利なことがよくあります。たとえば、データベース リソースの接続 URL を提供する必要がある場合があります。Dagster は、構成 API を通じてこの機能を公開します。

さまざまな Dagster エンティティ (アセット、オペレーション、リソース) を個別に構成できます。構成可能なエンティティを具体化 (アセット)、実行 (オペレーション)、またはインスタンス化 (リソース) するジョブを起動するときに、各エンティティの _実行構成_ を指定できます。エンティティを定義する関数内で、`config` パラメータを使用して渡された構成にアクセスできます。通常、指定された実行構成の値は、アセット/オペレーション/リソース定義に添付された _構成スキーマ_ に対応します。Dagster は、実行構成をスキーマに対して検証し、検証が成功した場合にのみ続行します。

構成の一般的な使用法は、[スケジュール](/guides/automate/schedules/) または [センサー](/guides/automate/sensors/) が起動するジョブ実行に構成を提供することです。たとえば、日次スケジュールでは、実行日を構成値としてアセットの 1 つに提供し、そのアセットではその構成値を使用して、読み取る日のデータを決定する場合があります。

## 構成の定義とアクセス

アセットまたはオペレーションによって受け入れられる構成可能なパラメータは、<PyObject section="config" module="dagster" object="Config"/> の構成モデル サブクラスと、対応するアセットまたはオペレーション関数への `config` パラメータを定義することによって指定されます。内部的には、これらの構成モデルは、データの検証とシリアル化のための一般的な Python ライブラリである [Pydantic](https://docs.pydantic.dev/) を利用しています。

実行中、指定された構成は、オペレーションまたはアセットの本体内で `config` パラメータを使用してアクセスされます。

<Tabs persistentKey="assetsorops">
<TabItem value="software-defined-assets を使用">

ここでは、ユーザー名を表す単一の文字列値を保持する <PyObject section="config" module="dagster" object="Config"/> のサブクラスを定義します。アセット本体の `config` パラメータを通じて構成にアクセスできます。

{/* TODO add dedent=4 prop when implemented */}
<CodeExample path="docs_snippets/docs_snippets/guides/dagster/pythonic_config/pythonic_config.py" startAfter="start_basic_asset_config" endBefore="end_basic_asset_config" />

</TabItem>
<TabItem value="ops とジョブを使用">

ここでは、ユーザー名を表す単一の文字列値を保持する <PyObject section="config" module="dagster" object="Config"/> のサブクラスを定義します。op 本体の `config` パラメータを通じて構成にアクセスできます。

<CodeExample path="docs_snippets/docs_snippets/guides/dagster/pythonic_config/pythonic_config.py" startAfter="start_basic_op_config" endBefore="end_basic_op_config" />

ジョブに構成を組み込むこともできます。

</TabItem>
</Tabs>

これらの例は、使用できる最も基本的な設定タイプを示しています。Dagster がサポートする設定タイプのセットの詳細については、[高度な設定タイプのドキュメント](advanced-config-types)を参照してください。

## リソースの Python 構成の定義とアクセス

リソースの設定可能なパラメータは、<PyObject section="resources" module="dagster" object="ConfigurableResource"/> のサブクラスであるリソース クラスの属性を指定することによって定義されます。以下のリソースは、リソースで定義された任意のメソッドからアクセスできる設定可能な接続 URL を定義します。

{/* TODO add dedent=4 prop when implemented */}
<CodeExample path="docs_snippets/docs_snippets/guides/dagster/pythonic_config/pythonic_config.py" startAfter="start_basic_resource_config" endBefore="end_basic_resource_config" />

リソースの使用に関する詳細については、[リソースガイド](/guides/build/external-resources/)を参照してください。

## ランタイム構成の指定

ジョブを実行したり、構成を指定するアセットをマテリアライズしたりするには、そのパラメータの値を指定する必要があります。これらの値を指定する方法は、使用しているインターフェース（Python、Dagster UI、またはコマンドライン（CLI））によって異なります。

<Tabs persistentKey="configtype">
<TabItem value="Python">

Python API から構成を指定する場合、<PyObject section="jobs" module="dagster" object="JobDefinition.execute_in_process" /> または <PyObject section="execution" module="dagster" object="materialize"/> の `​​run_config` 引数を使用できます。これは <PyObject section="config" module="dagster" object="RunConfig"/> オブジェクトを受け取り、その中でオペレーションごとまたはアセットごとに構成を指定できます。構成は辞書として指定され、キーはオペレーション/アセット名に対応し、値は構成値に対応します。

{/* TODO add dedent=4 prop when implemented */}
<CodeExample path="docs_snippets/docs_snippets/guides/dagster/pythonic_config/pythonic_config.py" startAfter="start_execute_with_config" endBefore="end_execute_with_config" />

</TabItem>
<TabItem value="Dagster UI">

UI の **Launchpad** タブから、構成エディターを使用して構成を YAML として提供できます。ここで、YAML スキーマは定義された構成クラスのレイアウトと一致します。エディターには、タイプアヘッド、スキーマ検証、およびスキーマ ドキュメントがあります。

**Scaffold Missing Config** ボタンをクリックして、構成スキーマに基づいてダミー値を生成することもできます。定義された `config` を使用してアセットをマテリアライズしようとすると、Launchpad エディターを含むモーダルがポップアップ表示されることに注意してください。

![Config in the Dagster UI](/images/guides/operate/config-ui.png)

</TabItem>
<TabItem value="コマンドライン">

### コマンドライン

Dagster の CLI から [`dagster job execute`](/api/python-api/cli#dagster-job) を使用してジョブを実行する場合、YAML ファイルに構成を配置できます:

```YAML file=/concepts/configuration/good.yaml
ops:
  op_using_config:
    config:
      person_name: Alice
```

次に、`--config` オプションを使用してファイル パスを渡します:

```bash
dagster job execute --config my_config.yaml
```

</TabItem>
</Tabs>

## 検証

Dagster は、提供された実行構成を対応する Pydantic モデルに対して検証します。検証が失敗すると、<PyObject section="errors" module="dagster" object="DagsterInvalidConfigError"/> または Pydantic `ValidationError` で実行を中止します。たとえば、次の両方とも失敗します。これは、構成スキーマに `nonexistent_config_value` がないためです。

{/* TODO add dedent=4 prop when implemented */}
<CodeExample path="docs_snippets/docs_snippets/guides/dagster/pythonic_config/pythonic_config.py" startAfter="start_execute_with_bad_config" endBefore="end_execute_with_bad_config" />

### config で環境変数を使用する

アセットとオペレーションは、設定オブジェクトの構築時に <PyObject section="resources" module="dagster" object="EnvVar" /> を渡すことで、環境変数を使用して設定できます。これは、値の機密性が高い場合や環境によって変わる可能性がある場合に便利です。Dagster+ を使用する場合、環境変数は [UI で直接設定](/guides/deploy/using-environment-variables-and-secrets) できます。

{/* TODO add dedent=4 prop when implemented */}
<CodeExample path="docs_snippets/docs_snippets/guides/dagster/pythonic_config/pythonic_config.py" startAfter="start_execute_with_config_envvar" endBefore="end_execute_with_config_envvar" />

Dagster の環境変数に関する一般的な情報については、[環境変数とシークレットのガイド](/guides/deploy/using-environment-variables-and-secrets)を参照してください。

## 次は

Config は、Dagster パイプラインをより柔軟かつ監視可能にする強力なツールです。サポートされている構成タイプの詳細については、[高度な構成タイプのドキュメント](advanced-config-types)を参照してください。再利用可能なロジックをカプセル化する強力な方法であるリソースの使用の詳細については、[リソース ガイド](/guides/build/external-resources)を参照してください。
