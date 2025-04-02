---
title: "列レベルの系統"
description: "Column lineage enables data and analytics engineers alike to understand how a column is created and used in your data platform."
sidebar_position: 4000
---

# 列レベルの系統

データベース テーブルを生成するアセットの場合、列レベルの系統は、コラボレーションの改善や問題のデバッグに役立つ強力なツールとなります。列の系統により、データ エンジニアと分析エンジニアは、データ プラットフォームで列がどのように作成され、使用されるかを理解できます。

## 仕組み

マテリアライゼーション メタデータとして発行される列の系統は次のようになります:

- Dagster で定義されたアセットに指定
- dbt などの統合から読み込まれたアセットに対して有効

Dagster はこのメタデータを使用して、Dagster UI のアセットの詳細ページからアクセスできる列の上流および下流の依存関係を表示します。**注**: UI で列レベルの系統を表示するのは、Dagster+ の機能です。

## 列レベルの系統化を有効にする

### Dagsterで定義されたアセットの場合

データベーステーブルを生成する Dagster アセットで列レベルの系統を有効にするには、次の操作を行う必要があります:

1. `metadata`パラメータを含む <PyObject section="assets" module="dagster" object="MaterializeResult" /> オブジェクトを返します。
2. `metadata` では、`dagster/column_lineage` キーを使用して <PyObject section="metadata" module="dagster" object="TableColumnLineage" /> オブジェクトを作成します。
3. このオブジェクトでは、<PyObject section="metadata" module="dagster" object="TableColumnLineage" displayText="TableColumnLineage.deps_by_column" /> を使用して列のリストを定義します。
4. 各列について、<PyObject section="metadata" module="dagster" object="TableColumnDep" /> を使用して依存関係を定義します。このオブジェクトは `asset_key` および `column_name` 引数を受け入れ、依存関係を構成するアセットと列の名前を指定できます。

例を見てみましょう:

<CodeExample path="docs_snippets/docs_snippets/concepts/metadata-tags/asset_column_lineage.py" />

マテリアライズされると、`my_asset` アセットは `new_column_foo` と `new_column_qux` の 2 つの列を作成します。

`new_column_foo` 列は他の 2 つの列に依存しています:

1. `source_bar` アセットの `column_bar`
2. `source_baz` アセットの `column_baz`

そして、2 番目の列である `new_column_qux` は、`source_bar` アセットの `column_quuz` に依存しています。

Dagster+ を使用している場合は、Dagster UI で列レベルの系統を表示できます。

### 統合で読み込まれたアセットの場合

列レベルの系統は現在、dbt 統合でサポートされています。詳細については、[dbt ドキュメント](/integrations/libraries/dbt/reference)を参照してください。

## Dagster UI で列レベルの系統を表示する

:::note

UI で列の系統を表示するのは Dagster+ の機能です。

:::

1. Dagster UI で、列レベルの系統が有効になっているアセットの **Asset details** ページを開きます。
2. まだ開いていない場合は、**Overview** タブに移動します。
3. **Columns** セクションで、表示する列の行にある **分岐** アイコンをクリックします。アイコンは行の右端にあります。

    ![Highlighted column lineage icon in the Asset details page of the Dagster UI](/images/guides/build/assets/metadata-tags/column-lineage-icon.png)

グラフには、アセットごとにグループ化された列の列依存関係が表示されます。

![Column lineage for a credit_utilization column in the Dagster UI](/images/guides/build/assets/metadata-tags/column-level-lineage.png)

別の列の系統を表示するには、**Column** ドロップダウンをクリックして別の列を選択します。
