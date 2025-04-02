---
title: "Dagster パイプ サブプロセスリファレンス"
description: "This page shows ways to execute external code with Dagster Pipes with different entities in the Dagster system."
---

このリファレンスでは、Dagster システム内の他のエンティティでの Dagster Pipes の使用法を示します。ステップごとのチュートリアルについては、[Dagster Pipes チュートリアル](/guides/build/external-pipelines/using-dagster-pipes) を参照してください。

## 環境変数と extra の指定

サブプロセスを起動するときに、外部プロセスで環境変数または追加のパラメータを使用できるようにする必要がある場合があります。extra は、外部プロセスのコンテキスト オブジェクトで使用できる任意のユーザー定義パラメータです。

<Tabs>
<TabItem value="external_code.py の外部コード">

外部コードでは、`PipesContext` オブジェクトを介して extra にアクセスできます:

<CodeExample path="docs_snippets/docs_snippets/guides/dagster/dagster_pipes/subprocess/with_extras_env/external_code.py" lineStart="2" />


</TabItem>
<TabItem value="dagster_code.py の Dagster コード">

`PipesSubprocessClient` リソースの `run` メソッドは `env` と `extras` も受け入れます。これにより、サブプロセスの実行時に環境変数と追加の引数を指定できます:

注: この例では `os.environ` を使用していますが、Dagster では本番環境では <PyObject section="resources" module="dagster" object="EnvVar" /> を使用することを推奨しています。

<CodeExample path="docs_snippets/docs_snippets/guides/dagster/dagster_pipes/subprocess/with_extras_env/dagster_code.py" />

</TabItem>
</Tabs>

## @asset_check と連携

場合によっては、アセットをマテリアライズするのではなく、データ品質チェックの結果を報告したい場合があります。アセットに <PyObject section="asset-checks" module="dagster" object="asset_check" decorator /> で定義されたデータ品質チェックがある場合:

<Tabs>

<TabItem value="external_code.py の外部コード">

外部コードから、<PyObject section="libraries" module="dagster_pipes" object="PipesContext" method="report_asset_check" /> を介してアセット チェックが実行されたことを Dagster に報告できます。この場合、`asset_key` は必須であり、<PyObject section="asset-checks" module="dagster" object="asset_check" decorator /> で定義されたアセット キーと一致する必要があることに注意してください:

<CodeExample path="docs_snippets/docs_snippets/guides/dagster/dagster_pipes/subprocess/with_asset_check/external_code.py" />

</TabItem>
<TabItem value="dagster_code.py の Dagster コード">

Dagster 側では、`PipesSubprocessClient` から返される `PipesClientCompletedInvocation` オブジェクトに `get_asset_check_result` メソッドが含まれており、これを使用してサブプロセスによって報告された <PyObject section="asset-checks" module="dagster" object="AssetCheckResult" /> イベントにアクセスできます。

<CodeExample path="docs_snippets/docs_snippets/guides/dagster/dagster_pipes/subprocess/with_asset_check/dagster_code.py" />

</TabItem>
</Tabs>

## マルチアセットでの作業

場合によっては、API への単一の呼び出しで複数のテーブルが更新されたり、単一のスクリプトで複数のアセットが計算されたりすることがあります。このような場合は、Dagster Pipes を使用して、複数のアセットを一度にレポートすることができます。

<Tabs>

<TabItem value="external_code.py の外部コード">

**注意**: 複数のアセットを扱う場合、<PyObject section="libraries" module="dagster_pipes" object="PipesContext" method="report_asset_materialization" /> は、一意のアセット キーごとに 1 回だけ呼び出すことができます。複数回呼び出されると、次のようなエラーが表示されます:

```bash
Calling {method} with asset key {asset_key} is undefined. Asset has already been materialized, so no additional data can be reported for it
```

代わりに、<PyObject module="dagster_pipes" section="libraries" object="PipesContext" method="report_asset_materialization" /> の各インスタンスに `asset_key` パラメータを設定する必要があります:

<CodeExample path="docs_snippets/docs_snippets/guides/dagster/dagster_pipes/subprocess/with_multi_asset/external_code.py" />

</TabItem>

<TabItem value="dagster_code.py の Dagster コード">

Dagster コードでは、<PyObject section="assets" module="dagster" object="multi_asset" decorator /> を使用して、複数のアセットを表す単一のアセットを定義できます。`PipesSubprocessClient` から返される `PipesClientCompletedInvocation` オブジェクトには `get_results` メソッドが含まれており、これを使用して、サブプロセスによって報告される複数の <PyObject section="ops" module="dagster" object="AssetMaterialization" pluralize /> や <PyObject section="asset-checks" module="dagster" object="AssetCheckResult" pluralize /> などのすべてのイベントにアクセスできます:

<CodeExample path="docs_snippets/docs_snippets/guides/dagster/dagster_pipes/subprocess/with_multi_asset/dagster_code.py" />

</TabItem>
</Tabs>

## カスタムデータの受け渡し

場合によっては、出力の作成に使用するなど、Dagster に直接レポートする以外の目的で、オーケストレーション コードで使用するために外部プロセスからデータを返したい場合があります。この例では、カスタム メッセージを使用して、アセットから返される I/O 管理出力を作成します。

<Tabs>
<TabItem value="external_code.py の外部コード">

外部コードでは、`report_custom_message` を使用してメッセージを送信します。メッセージは、JSON シリアル化可能な任意のデータにすることができます。

<CodeExample path="docs_snippets/docs_snippets/guides/dagster/dagster_pipes/subprocess/custom_messages/external_code.py" />

</TabItem>
<TabItem value="dagster_code.py の Dagster コード">

Dagster コードでは、`get_custom_messages` を使用してカスタム メッセージを受信します。

<CodeExample path="docs_snippets/docs_snippets/guides/dagster/dagster_pipes/subprocess/custom_messages/dagster_code.py" />

</TabItem>
</Tabs>

## 豊富なメタデータを Dagster に渡す

Dagster は、<PyObject section="metadata" module="dagster" object="TableMetadataValue"/>、<PyObject section="metadata" module="dagster" object="UrlMetadataValue"/>、<PyObject section="metadata" module="dagster" object="JsonMetadataValue"/> などの豊富なメタデータ タイプをサポートしています。ただし、`dagster-pipes` パッケージは `dagster` に直接依存していないため、特定の形式で生の辞書を Dagster に渡す必要があります。特定のメタデータ タイプに対して、次のキーを持つ辞書を指定する必要があります:

- `type`: メタデータ値のタイプ（`table`、`url`、`json` など）。
- `raw_value`: メタデータの実際の値。

以下は、サポートされているすべてのメタデータ タイプのデータを指定する例です。float、integer、boolean、string、null のメタデータ オブジェクトは、辞書を必要とせずに直接渡すことができます。

### Examples for complex metadata types

#### URL Metadata

<CodeExample path="docs_snippets/docs_snippets/guides/dagster/dagster_pipes/subprocess/rich_metadata/url_metadata.py" />

#### Path Metadata

<CodeExample path="docs_snippets/docs_snippets/guides/dagster/dagster_pipes/subprocess/rich_metadata/path_metadata.py" />

#### Notebook Metadata

<CodeExample path="docs_snippets/docs_snippets/guides/dagster/dagster_pipes/subprocess/rich_metadata/notebook_metadata.py" />

#### JSON Metadata

<CodeExample path="docs_snippets/docs_snippets/guides/dagster/dagster_pipes/subprocess/rich_metadata/json_metadata.py" />

#### Markdown Metadata

<CodeExample path="docs_snippets/docs_snippets/guides/dagster/dagster_pipes/subprocess/rich_metadata/markdown_metadata.py" />

#### Table Metadata

<CodeExample path="docs_snippets/docs_snippets/guides/dagster/dagster_pipes/subprocess/rich_metadata/table_metadata.py" />

#### Table Schema Metadata

<CodeExample path="docs_snippets/docs_snippets/guides/dagster/dagster_pipes/subprocess/rich_metadata/table_schema_metadata.py" />

#### Table Column Lineage Metadata

<CodeExample path="docs_snippets/docs_snippets/guides/dagster/dagster_pipes/subprocess/rich_metadata/table_column_lineage.py" startAfter="start_table_column_lineage" endBefore="end_table_column_lineage" />

#### Timestamp Metadata

<CodeExample path="docs_snippets/docs_snippets/guides/dagster/dagster_pipes/subprocess/rich_metadata/timestamp_metadata.py" startAfter="start_timestamp" endBefore="end_timestamp" />
