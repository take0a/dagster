---
title: アセットの作成と実体化
description: Load project data and create and materialize assets
last_update:
  author: Alex Noonan
sidebar_position: 10
---


チュートリアルの最初のステップでは、生データのファイルを使用して Dagster プロジェクトを作成しました。このステップでは、次の操作を行います:

- 最初の定義オブジェクトを作成する
- DuckDBリソースを追加する
- ソフトウェア定義アセットの構築
- アセットの実体化

## 1. 定義オブジェクトを作成する

Dagster では、 <PyObject section="definitions" module="dagster" object="Definitions" /> オブジェクトは、アセットやリソースなど、プロジェクト内のさまざまなコンポーネントを定義および整理します。

`etl_tutorial` ディレクトリの `definitions.py` ファイルを開き、次のコードをコピーします:

```python
import json
import os

from dagster_duckdb import DuckDBResource

import dagster as dg

defs = dg.Definitions(
  assets=[],
  resources={},
)
```

## 2. DuckDB リソースを定義する

Dagster では、[リソース](/api/python-api/resources)は、ジョブを実行するために必要な外部サービス、ツール、およびストレージ バックエンドです。このプロジェクトのストレージ バックエンドには、アプリケーション内で実行される高速なインプロセス SQL データベースである [DuckDB](https://duckdb.org/) を使用します。これを定義オブジェクトで 1 回定義すると、必要なすべてのアセットとオブジェクトで使用できるようになります。


```python
defs = dg.Definitions(
    assets=[],
    resources={"duckdb": DuckDBResource(database="data/mydb.duckdb")},
)
```

## 3. アセットを作成する

ソフトウェア定義 <PyObject section="assets" module="dagster" object="asset" pluralize /> は、Dagster の主要な構成要素です。資産は次の 3 つのコンポーネントで構成されます:

1. アセットキーまたは一意の識別子。
2. アセットを生成するために呼び出される関数である op。
3. アセットが依存する上流の依存関係。

[アセット中心のアプローチ](https://dagster.io/blog/software-defined-assets)の背後にある我々の哲学について詳しく読むことができます。

### 製品アセット

まず、製品 CSV からのデータを保持するための DuckDB テーブルを作成するアセットを作成します。このアセットは、前に定義した `duckdb` リソースを取得し、<PyObject section="assets" module="dagster" object="MaterializeResult" /> オブジェクトを返します。さらに、このアセットには、アセットを分類するための <PyObject section="assets" module="dagster" object="asset" decorator /> デコレータパラメータと、Dagster UI でアセットのプレビューを表示するための `return` ブロックにメタデータが含まれています。

このアセットを作成するには、`definitions.py` ファイルを開き、次のコードをコピーします:

<CodeExample
  path="docs_snippets/docs_snippets/guides/tutorials/etl_tutorial/etl_tutorial/definitions.py"
  language="python"
  lineStart="8"
  lineEnd="33"
/>

### 営業担当者アセット

営業担当者アセットのコードは、製品アセット コードと似ています。`definitions.py` ファイルで、製品アセット コードの下に次のコードをコピーします:

<CodeExample
  path="docs_snippets/docs_snippets/guides/tutorials/etl_tutorial/etl_tutorial/definitions.py"
  language="python"
  lineStart="35"
  lineEnd="61"
/>

### 販売データアセット

販売データアセットを追加するには、次のコードを `definitions.py` ファイルの営業担当者アセットの下にコピーします:

<CodeExample
  path="docs_snippets/docs_snippets/guides/tutorials/etl_tutorial/etl_tutorial/definitions.py"
  language="python"
  lineStart="62"
  lineEnd="87"
/>

## 4. 定義オブジェクトにアセットを追加する

次に、これらのアセットを Definitions オブジェクトに取り込みます。これらを Definitions オブジェクトに追加すると、Dagster プロジェクトで使用できるようになります。これらを、assets パラメータの空のリストに追加します。

```python
defs = dg.Definitions(
    assets=[products,
        sales_reps,
        sales_data,
    ],
    resources={"duckdb": DuckDBResource(database="data/mydb.duckdb")},
)
```

## 5. アセットを実体化する

アセットを実体化するには:

1. ブラウザで、先ほど起動した Dagster サーバーの URL に移動します。
2. **Deployment** に移動します。
3. Reload definitions をクリックします。
4. **Assets** をクリックして、すべてのアセットを見るため、"View global asset lineage" をクリックします。

   ![2048 resolution](/images/tutorial/etl-tutorial/etl-tutorial-first-asset-lineage.png)

5. materialize all をクリックします。
6. runs タブに移動し、 the most recent run を選択します。ここで、実行のログを確認できます。

   ![2048 resolution](/images/tutorial/etl-tutorial/first-asset-run.png)

## 次は

- [アセットの依存関係](/etl-pipeline-tutorial/create-and-materialize-a-downstream-asset)を使用してこのチュートリアルを続行します。
