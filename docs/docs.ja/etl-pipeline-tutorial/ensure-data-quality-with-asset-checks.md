---
title: アセットチェックによるデータ品質の確保
description: Ensure assets are correct with asset checks
last_update:
  author: Alex Noonan
sidebar_position: 30
---

データ パイプラインではデータ品質が重要です。個々のアセットを検査することで、データ品質の問題がパイプライン全体に影響する前に検出されます。

Dagster では、アセットを定義するのと同じように [アセットチェック](/guides/test/asset-checks) を定義します。アセットチェックは、アセットが実体化されたときに実行されます。このステップでは、次の操作を行います:

- アセットチェックを定義する
- UIでアセットチェックを実行する

## 1. アセットチェックを定義する

この場合、`joined_data` 内に `rep_name` または `product_name` の値が欠落している行があるかどうかを識別するチェックを作成します。

`joined_data` アセットの下に次のコードをコピーします。

<CodeExample
  path="docs_snippets/docs_snippets/guides/tutorials/etl_tutorial/etl_tutorial/definitions.py"
  language="python"
  lineStart="134"
  lineEnd="150"
/>


## 2. アセットチェックを実行する

アセットチェックを実行する前に、それを定義オブジェクトに追加する必要があります。アセットと同様に、アセットチェックも独自のリストに追加されます。

定義オブジェクトは次のようになります:

```python
defs = dg.Definitions(
    assets=[products,
        sales_reps,
        sales_data,
        joined_data,
    ],
    asset_checks=[missing_dimension_check],
    resources={"duckdb": DuckDBResource(database="data/mydb.duckdb")},
)
```

アセットチェックはアセットが実体化されたときに実行されますが、UI でアセットチェックを手動で実行することもできます。

1. Definitions をリロードします。
2. `joined_data` アセットの Asset Details ページへ遷移します。
3. "Checks" タブを選択します。
4. `missing_dimension_check` の **Execute** ボタンをクリックします。

![2048 resolution](/images/tutorial/etl-tutorial/asset-check.png)

## 次は

- このチュートリアルを続けて、[パーティション化されたアセットの作成と実体化](/etl-pipeline-tutorial/create-and-materialize-partitioned-asset) に進みます。
