---
title: "アセットメタデータ"
---

[アセット](/guides/build/assets/) は Dagster UI で重要な位置を占めています。アセットに情報を添付すると、アセットがどこに保存されているか、何が含まれているか、どのように整理するかを理解できます。

Dagsterのメタデータを使用すると:

- 所有者情報を付与する
- タグでアセットを整理する
- Markdownの説明、テーブルスキーマ、時系列などのリッチで複雑な情報を添付する
- アセットをソースコードにリンクする

## アセットへの所有者の追加 \{#owners}

大規模な組織では、特定のデータ資産に対してどの個人とチームが責任を負っているかを把握することが重要です。

<CodeExample path="docs_beta_snippets/docs_beta_snippets/guides/data-modeling/metadata/owners.py" language="python" />

:::note

`owners` は、電子メール アドレスか、`team:` で始まるチーム名 (例: `team:data-eng`) のいずれかである必要があります。

:::

:::tip
Dagster+ Pro を使用すると、トリガーされるとアセットの所有者に自動的に通知するアセットベースのアラートを作成できます。詳細については、[Dagster+ アラートのドキュメント](/dagster-plus/features/alerts)を参照してください。
:::

## タグによるアセットの整理 \{#tags}

[**タグ**](tags) は、Dagster でアセットを整理するための主な方法です。アセットを定義するときに複数のタグを添付することができ、それらは UI に表示されます。また、タグを使用してアセットカタログ内のアセットを検索およびフィルタリングすることもできます。タグは、文字列のキーと値のペアとして構造化されています。

アセットに適用できるタグの例を次に示します:

```python
{"domain": "marketing", "pii": "true"}
```

`owners` と同様に、アセットを定義するときに、`tags` 引数にタグの辞書を渡すことができます:

<CodeExample path="docs_beta_snippets/docs_beta_snippets/guides/data-modeling/metadata/tags.py" language="python" />

タグにはキーと値として文字列のみを含める必要があることに注意してください。また、Dagster UI は、空の文字列を含むタグをキーと値のペアではなく「ラベル」としてレンダリングします。

## アセットにメタデータを添付する \{#attaching-metadata}

**メタデータ** を使用すると、Markdown の説明、テーブル スキーマ、時系列などの豊富な情報をアセットに添付できます。メタデータはタグよりも柔軟性が高く、より複雑な情報を保存できます。

メタデータは、定義時、コードが最初にインポートされたとき、または実行時にアセットが実体化されたときにアセットに添付できます。

### 定義時 \{#definition-time-metadata}

定義メタデータを使用してアセットを記述すると、自分やチームにコンテキストを簡単に提供できます。このメタデータには、アセットの説明、アセットの種類、または関連するドキュメントへのリンクなどがあります。

<CodeExample path="docs_beta_snippets/docs_beta_snippets/guides/data-modeling/metadata/definition-metadata.py" language="python" />

添付できるさまざまな種類のメタデータの詳細については、<PyObject section="metadata" module="dagster" object="MetadataValue" /> API ドキュメントを参照してください。

一部のメタデータキーは、Dagster UI で特別な扱いを受けます。詳細については、[標準メタデータ タイプ](#standard-metadata-types) セクションを参照してください。

### 実行時 \{#runtime-metadata}

ランタイム メタデータを使用すると、処理されたレコードの数やマテリアライズが発生した時期など、アセットのマテリアライズに関する情報を表示できます。これにより、アセットの情報が変更されたときに更新し、履歴メタデータを時系列として追跡できます。

アセットにマテリアライゼーション メタデータを添付するには、メタデータ パラメータを含む <PyObject section="assets" module="dagster" object="MaterializeResult" /> オブジェクトを返します。このパラメータは、キーと値のペアの辞書を受け入れます。キーは文字列である必要があります。

値を指定するときは、<PyObject section="metadata" module="dagster" object="MetadataValue" /> ユーティリティ クラスを使用してデータをラップし、UI に正しく表示されるようにします。値は Python のプリミティブ型にすることもでき、Dagster はそれを適切な `MetadataValue` に変換します。

<CodeExample path="docs_beta_snippets/docs_beta_snippets/guides/data-modeling/metadata/runtime-metadata.py" language="python" />

:::note

数値メタデータは、Dagster UI では時系列として扱われます。

:::

## 標準メタデータ型 \{#standard-metadata-types}

次のメタデータ キーは、Dagster UI で特別な処理が行われます。

| Key                           | Description                                |
| ----------------------------- | ------------------------------------------ |
| `dagster/uri`                 | **Type:** `str` <br/><br/> アセットの URI。例: "s3://my_bucket/my_object"|
| `dagster/column_schema`       | **Type:** <PyObject section="metadata" module="dagster" object="TableSchema" /> <br/><br/> テーブルであるアセットの場合、テーブル内の列のスキーマ。詳細については、[テーブルと列のメタデータ](#table-column)セクションを参照してください。             |
| `dagster/column_lineage`      | **Type:** <PyObject section="metadata" module="dagster" object="TableColumnLineage" /><br/><br/> テーブルであるアセットの場合、テーブルの列入力から列出力までの系統。詳細については、[テーブルと列のメタデータ](#table-column)セクションを参照してください。  |
| `dagster/row_count`           | **Type:** `int` <br/><br/> テーブルであるアセットの場合、テーブル内の行数。詳細については、テーブル メタデータのドキュメントを参照してください。   |
| `dagster/partition_row_count` | **Type:** `int` <br/><br/> テーブルであるアセットのパーティションの場合、パーティション内の行数。   |
| `dagster/table_name`          | **Type:** `str` <br/><br/>テーブル/ビューの一意の識別子。通常は完全修飾です。例: my_database.my_schema.my_table    |
| `dagster/code_references`     | **Type:** <PyObject section="metadata" module="dagster" object="CodeReferencesMetadataValue" /><br/><br/> ファイルの場所や GitHub URL への参照など、アセットの [コード参照](#source-code) のリスト。マテリアライゼーション メタデータではなく、定義レベルのメタデータでのみ提供する必要があります。 |

## テーブルと列のメタデータ \{#table-column}

最も強力なメタデータ タイプのうち 2 つは、<PyObject section="metadata" module="dagster" object="TableSchema" /> と <PyObject section="metadata" module="dagster" object="TableColumnLineage" /> です。これらのメタデータ タイプを使用すると、関係者は Dagster 内でテーブルのスキーマを表示し、Dagster+ で列の系統を使用して [Asset カタログ](/dagster-plus/features/asset-catalog/)に移動できます。

### テーブルスキーマメタデータ \{#table-schema}

次の例では、定義時と実行時の両方で [テーブルと列のスキーマ メタデータ](table-metadata) をアタッチします。

<CodeExample path="docs_beta_snippets/docs_beta_snippets/guides/data-modeling/metadata/table-schema-metadata.py" language="python" />

<PyObject section="metadata" module="dagster" object="TableColumn" /> オブジェクトでは、いくつかのデータ型と制約を使用できます。詳細については、API ドキュメントを参照してください。

### 列系統メタデータ \{#column-lineage}

:::tip
[dbt](/integrations/libraries/dbt/) などの多くの統合では、列系統メタデータがすぐに使用できる状態で自動的に添付されます。
:::

[列系統メタデータ](column-level-lineage)は、テーブル内の列が他の列からどのように派生しているかを追跡するための強力な方法です:

<CodeExample path="docs_beta_snippets/docs_beta_snippets/guides/data-modeling/metadata/table-column-lineage-metadata.py" language="python" title="Table column lineage metadata" />

:::tip
Dagster+ は、アセット カタログ内の列の系統の豊富な視覚化とナビゲーションを提供します。詳細については、[Dagster+ のドキュメント](/dagster-plus/features/asset-catalog/)を参照してください。
:::

## アセットをソースコードにリンクする \{#source-code}

import Beta from '../../../../partials/\_Beta.md';

<Beta />


アセットをソース コードにリンクするには、コード参照を添付します。コード参照は、ローカル開発と本番環境の両方で、Dagster UI からアセットのソース コードを簡単に表示できるようにするメタデータの一種です。

:::tip

[dbt](/integrations/libraries/dbt/reference#attaching-code-reference-metadata) などの多くの統合がこの機能をサポートしています。

:::

### ローカル開発用の Python コード参照を添付する \{#python-references}

Dagster は、ローカル開発中にアセットにコード参照を自動的に添付できます:

<CodeExample path="docs_beta_snippets/docs_beta_snippets/guides/data-modeling/metadata/python-local-references.py" language="python" />

### コード参照のカスタマイズ \{#custom-references}

コード参照の添付方法をカスタマイズしたい場合（[アセットファクトリを使用したドメイン固有言語](/guides/build/assets/creating-asset-factories)を構築する場合など）は、アセット定義に `dagster/code_references` メタデータを手動で追加できます:

<CodeExample path="docs_beta_snippets/docs_beta_snippets/guides/data-modeling/metadata/custom-local-references.py" language="python" />

### 本番環境でのコード参照の添付 \{#production-references}

<Tabs>
  <TabItem value="dagster-plus" label="Dagster+">

Dagster+ は、GitHub や GitLab などのソース管理へのコード参照を使用してアセットに自動的に注釈を付けることができます。

<CodeExample path="docs_beta_snippets/docs_beta_snippets/guides/data-modeling/metadata/plus-references.py" language="python" />

</TabItem>
<TabItem value="dagster-open-source" label="OSS">

Dagster+ を使用していない場合は、ソースコントロールへのコード参照を使用してアセットに注釈を付けることができますが、手動でのマッピングが必要です。

<CodeExample path="docs_beta_snippets/docs_beta_snippets/guides/data-modeling/metadata/oss-references.py" language="python" />

`link_code_references_to_git` は現在、GitHub および GitLab リポジトリをサポートしています。また、ファイル パスのマッピング方法のカスタマイズもサポートしています。詳細については、`AnchorBasedFilePathMapping` API ドキュメントを参照してください。

</TabItem>
</Tabs>
