---
title: "テーブルメタデータ"
description: "Table metadata can be used to provide additional context about a tabular asset, such as its schema, row count, and more."
sidebar_position: 3000
---

テーブルメタデータは、スキーマ、行数など、表形式のアセットに関する追加のコンテキストを提供します。このメタデータを使用すると、データプラットフォームでのコラボレーション、デバッグ、データ品質を向上させることができます。

Dagster は、次のようなさまざまな種類のテーブルメタデータをアセットに添付することをサポートしています:

- [**Column schema**](#attaching-column-schema)、列名や型など、テーブルの構造を記述します。
- [**Row count**](#attaching-row-count)、マテリアライズされたテーブル内の行数を表します。
- [**Column-level lineage**](#attaching-column-level-lineage)、列が他のアセットによってどのように作成され、使用されるかを説明します。

## 列スキーマを添付する

### Dagsterで定義されたアセットの場合

列スキーマ メタデータは、[定義メタデータ](index.md#definition-time-metadata) または [ランタイム メタデータ](index.md#runtime-metadata) として Dagster アセットに添付することができ、Dagster UI に表示されます。例:

![Column schema for an asset in the Dagster UI](/images/guides/build/assets/metadata-tags/metadata-table-schema.png)

アセットのスキーマが事前に定義されている場合は、それを定義メタデータとして添付できます。アセットが実体化されたときにのみスキーマがわかる場合は、それを実体化した際にメタデータとして添付できます。

アセットにスキーマ メタデータを添付するには、次の手順を実行する必要があります:

1. テーブル内の各列を記述する <PyObject section="metadata" module="dagster" object="TableColumn"  /> エントリを持つ <PyObject section="metadata" module="dagster" object="TableSchema"/> オブジェクトを構築します。
2. `TableSchema` オブジェクトを、`dagster/column_schema` キーの下の `metadata` パラメータの一部としてアセットに添付します。これは、アセット定義に添付することも、アセット関数によって返される <PyObject section="assets" module="dagster" object="MaterializeResult" /> オブジェクトに添付することもできます。

以下に、列スキーマメタデータをアセットに添付する方法の 2 つの例を示します。1 つは定義メタデータとして、もう 1 つはランタイムメタデータとしてです:

<CodeExample path="docs_snippets/docs_snippets/concepts/metadata-tags/asset_column_schema.py" />

`my_asset` のスキーマは Dagster UI に表示されます。

### 統合で読み込まれたアセットの場合

Dagster の dbt 統合により、dbt モデルから読み込まれたアセットに列スキーマ メタデータを自動的に添付できるようになります。詳細については、[dbt ドキュメント](/integrations/libraries/dbt/reference#fetching-column-level-metadata) を参照してください。

## 行数を添付する

行数メタデータは、[ランタイム メタデータ](index.md#runtime-metadata) として Dagster アセットに添付され、マテリアライズド テーブル内の行数に関する追加のコンテキストを提供できます。これは Dagster UI で強調表示されます。たとえば:

![Row count for an asset in the Dagster UI](/images/guides/build/assets/metadata-tags/metadata-row-count.png)

Dagster では、最新の行数を表示するだけでなく、時間の経過に伴う行数の変化を追跡し、この情報を使用してデータの品質を監視することができます。

アセットに行数メタデータを添付するには、アセット関数によって返される <PyObject section="assets" module="dagster" object="MaterializeResult" /> オブジェクトのメタデータ パラメータの `dagster/row_count` キーに数値を添付する必要があります。たとえば:

<CodeExample path="docs_snippets/docs_snippets/concepts/metadata-tags/asset_row_count.py" />

## 列レベルの系統の添付

列の系統により、データ エンジニアと分析エンジニアは、データ プラットフォームで列がどのように作成され、使用されるかを理解できます。詳細については、[列レベルの系統のドキュメント](/guides/build/assets/metadata-and-tags/column-level-lineage) を参照してください。

## テーブルスキーマの一貫性の確保

実行時に実行時メタデータを通じて列スキーマが定義される場合、マテリアライゼーション間のスキーマ変更を検出して警告することが役立ちます。Dagster は、これらの変更を検出するために <PyObject section="asset-checks" module="dagster" object="build_column_schema_change_checks"/> API を提供します。

この関数は、現在のマテリアライゼーションのスキーマを以前のマテリアライゼーションのスキーマと比較するアセット チェックを作成します。これらのチェックでは、次のことを検出できます。

- 追加された列
- 削除された列
- 列の型の変更

実行時にテーブル スキーマを定義する上記の例で、`my_other_asset` アセットの列スキーマ変更チェックを定義しましょう。

<CodeExample path="docs_snippets/docs_snippets/concepts/metadata-tags/schema_change_checks.py" startAfter="start_check" endBefore="end_check" />
