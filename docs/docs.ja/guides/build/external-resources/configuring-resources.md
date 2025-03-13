---
title: リソースの構成
sidebar_position: 200
---

リソースは、環境変数で、または、起動時に構成できます。さらに、他のリソースに依存するリソースを定義して、共通の構成を管理することもできます。

## リソースでの環境変数の使用

リソースは環境変数を使用して設定できます。これは、シークレットやその他の環境固有の設定に役立ちます。[Dagster+](/dagster-plus/) を使用している場合は、環境変数を [UI で直接設定](/dagster-plus/deployment/management/environment-variables) できます。

環境変数を使用するには、リソースの構築時に <PyObject section="resources" module="dagster" object="EnvVar" /> を渡します。`EnvVar` は `str` から継承され、リソースの任意の文字列構成フィールドに入力するために使用できます。環境変数の値は、実行が開始されたときに評価されます。

{/* TODO add dedent=4 prop to CodeExample below when implemented */}
<CodeExample path="docs_snippets/docs_snippets/concepts/resources/pythonic_resources.py" startAfter="start_new_resources_env_vars" endBefore="end_new_resources_env_vars" />

:::note

**`os.getenv()` はどうでしょうか?** `os.getenv()` を使用すると、Dagster がコードの場所をロードするときに変数の値が取得されます。`EnvVar` を使用すると、実行時に値を取得するように Dagster に指示するだけでなく、UI に値を表示しないようにも指示します。

<!-- Lives in /next/components/includes/EnvVarsBenefits.mdx -->

Dagster で環境変数を使用する方法の詳細については、[環境変数ガイド](/guides/deploy/using-environment-variables-and-secrets)を参照してください。

:::

## 起動時にリソースを構成する

場合によっては、起動時に、Launchpad または [スケジュール](/guides/automate/schedules/) または [センサー](/guides/automate/sensors/) の <PyObject section="schedules-sensors" module="dagster" object="RunRequest" /> でリソースの構成を指定する必要があることがあります。たとえば、センサーによってトリガーされる実行で、実行ごとにデータベース リソース内の異なるターゲット テーブルを指定する必要がある場合があります。

`configure_at_launch()` メソッドを使用すると、構成可能なリソースの構築を起動時まで延期できます:

{/* TODO add dedent=4 prop to CodeExample below when implemented */}
<CodeExample path="docs_snippets/docs_snippets/concepts/resources/pythonic_resources.py" startAfter="start_new_resource_runtime" endBefore="end_new_resource_runtime" />

### Pythonコードでリソース起動時間の設定を提供する

次に、Launchpad での起動時にリソースの構成を指定するか、<PyObject section="schedules-sensors" module="dagster" object="RunRequest" /> の `​​config` パラメータを使用して Python コードで指定できます。

{/* TODO add dedent=4 prop to CodeExample below when implemented */}
<CodeExample path="docs_snippets/docs_snippets/concepts/resources/pythonic_resources.py" startAfter="start_new_resource_runtime_launch" endBefore="end_new_resource_runtime_launch" />

## 他のリソースに依存するリソース

状況によっては、他のリソースに依存するリソースを定義したい場合があります。これは、共通の構成に役立ちます。たとえば、データベースとファイルストアの個別のリソースは、どちらも特定のクラウド プロバイダーの資格情報に依存している可能性があります。これらの資格情報を個別のネストされたリソースとして定義すると、1 か所で構成を指定できます。また、ネストされたリソースをモックできるため、リソースのテストも簡単になります。

この場合、ネストされたリソースをリソース クラスの属性としてリストできます:

{/* TODO add dedent=4 prop to CodeExample below when implemented */}
<CodeExample path="docs_snippets/docs_snippets/concepts/resources/pythonic_resources.py" startAfter="start_new_resources_nesting" endBefore="end_new_resources_nesting" />

起動時に資格情報の構成を提供する場合は、`configure_at_launch()` メソッドを使用して、`CredentialsResource` の構築を起動時まで延期します。

`credentials` は、Launchpad を介した起動時の構成を必要とするため、起動時に構成を提供できるように、<PyObject section="definitions" module="dagster" object="Definitions" /> オブジェクトにも渡す必要があります。ネストされたリソースは、起動時の構成が必要な場合のみ、<PyObject section="definitions" module="dagster" object="Definitions" /> オブジェクトに渡す必要があります。

<CodeExample path="docs_snippets/docs_snippets/concepts/resources/pythonic_resources.py" startAfter="start_new_resource_dep_job_runtime" endBefore="end_new_resource_dep_job_runtime" />

## 次は

リソースは、アセットとオペレーションに再利用可能なロジックをカプセル化する強力な方法です。リソースでサポートされている構成タイプの詳細については、[高度な構成タイプのドキュメント](/guides/operate/configuration/advanced-config-types)を参照してください。アセットとオペレーションをパラメータ化するために使用できる Dagster 構成システムの詳細については、[実行構成のドキュメント](/guides/operate/configuration/run-configuration)を参照してください。