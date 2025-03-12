---
title: 下流のアセットの作成と実体化
description: Reference Assets as dependencies to other assets
last_update:
  author: Alex Noonan
sidebar_position: 20
---

DuckDB に生のデータが読み込まれたので、上流のアセットを結合する [下流のアセット](/guides/build/assets/defining-assets-with-asset-dependencies) を作成する必要があります。このステップでは、次の操作を行います:

- 下流のアセットを作成する
- そのアセットを実体化する

## 1. 下流のアセットを作成する

すべての生データが DuckDB に読み込まれたので、次のステップでは、3 つのソース テーブルすべてのデータで構成されるビューにデータを結合します。

これを SQL で実現するには、`sales_data` テーブルを取り込み、`sales_reps` と `products` のそれぞれの ID 列を左結合します。さらに、このビューは簡潔に保ち、分析に関連する列のみにします。

ご覧のとおり、新しい `joined_data` アセットは、いくつかの小さな変更を除いて、以前のものとよく似ています。このアセットを別のグループに配置します。このアセットを生のテーブルに依存させるには、アセット定義の `deps` パラメータにアセット キーを追加します。

<CodeExample path="docs_beta_snippets/docs_beta_snippets/guides/tutorials/etl_tutorial/etl_tutorial/definitions.py" language="python" lineStart="89" lineEnd="132"/>

## 2. そのアセットを実体化する

1. joined_data アセットを定義オブジェクトに追加する

  ```python
  defs = dg.Definitions(
    assets=[products,
        sales_reps,
        sales_data,
        joined_data,
    ],
    resources={"duckdb": DuckDBResource(database="data/mydb.duckdb")},
  )
  ```

2. Dagster UI で定義を再読み込みし、`joined_data` アセットを実体化します。

## 次は

- このチュートリアルを続けて、[アセット チェックによるデータ品質の確保](ensure-data-quality-with-asset-checks) に進みます。