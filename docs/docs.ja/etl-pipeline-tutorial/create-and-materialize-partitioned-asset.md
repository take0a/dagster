---
title: パーティション化されたアセットの作成と実体化
description: Partitioning Assets by datetime and categories
last_update:
  date: 2024-11-25
  author: Alex Noonan
sidebar_position: 40
---

[パーティション](/guides/build/partitions-and-backfills/partitioning-assets)は、Dagster のコア抽象化であり、大規模なデータセットの管理、増分更新の処理、パイプラインのパフォーマンスの向上を可能にします。アセットは、次の方法でパーティション分割できます:

- 時間ベース: 期間別にデータを分割します (例: 日次、月次)
- カテゴリベース: 既知のカテゴリ (国、製品タイプなど) ごとに分割します
- 2次元: 2つのパーティションタイプを組み合わせる（例：国 + 日付）
- 動的: 実行時の条件に基づいてパーティションを作成する

このステップでは、次の操作を行います:

- 月ごとに分割された時間ベースのアセットを作成する
- 製品カテゴリごとに分割されたカテゴリベースのアセットを作成する

## 1. 時間ベースで分割されたアセットを作成する

Dagster は、日時グループによるアセットのパーティション分割をネイティブにサポートしています。各営業担当者の月間パフォーマンスを計算するアセットを作成したいと考えています。月間パーティションを作成するには、次のコードを `missing_dimension_check` アセット チェックの下にコピーします。

<CodeExample
  path="docs_snippets/docs_snippets/guides/tutorials/etl_tutorial/etl_tutorial/definitions.py"
  language="python"
  lineStart="152"
  lineEnd="153"
/>

パーティション データは、コンテキストによってアセット内でアクセスされます。パーティションから特定の月に対してこの計算を実行し、その月の以前の値を削除するアセットを作成します。作成したばかりの `monthly_partition` の下に次のアセットをコピーします。

```python
@dg.asset(
    partitions_def=monthly_partition,
    compute_kind="duckdb",
    group_name="analysis",
    deps=[joined_data],
)
def monthly_sales_performance(
    context: dg.AssetExecutionContext, duckdb: DuckDBResource
):
    partition_date_str = context.partition_key
    month_to_fetch = partition_date_str[:-3]

    with duckdb.get_connection() as conn:
        conn.execute(
            f"""
            create table if not exists monthly_sales_performance (
                partition_date varchar,
                rep_name varchar,
                product varchar,
                total_dollar_amount double
            );

            delete from monthly_sales_performance where partition_date = '{month_to_fetch}';

            insert into monthly_sales_performance
            select
                '{month_to_fetch}' as partition_date,
                rep_name,
                product_name,
                sum(dollar_amount) as total_dollar_amount
            from joined_data where strftime(date, '%Y-%m') = '{month_to_fetch}'
            group by '{month_to_fetch}', rep_name, product_name;
            """
        )

        preview_query = f"select * from monthly_sales_performance where partition_date = '{month_to_fetch}';"
        preview_df = conn.execute(preview_query).fetchdf()
        row_count = conn.execute(
            f"""
            select count(*)
            from monthly_sales_performance
            where partition_date = '{month_to_fetch}'
            """
        ).fetchone()
        count = row_count[0] if row_count else 0

    return dg.MaterializeResult(
        metadata={
            "row_count": dg.MetadataValue.int(count),
            "preview": dg.MetadataValue.md(preview_df.to_markdown(index=False)),
        }
    )
```

## 2. カテゴリベースの分割アセットを作成する

既知の定義済みパーティションを使用すると、データセットをサブセット化するさまざまなグループがわかっている場合に、データセットを分割する簡単な方法になります。パイプラインでは、各製品カテゴリのパフォーマンスを表すアセットを作成します。

1. 製品カテゴリの静的に定義されたパーティションを作成するには、次のコードを `monthly_sales_performance` アセットの下にコピーします:

<CodeExample
  path="docs_snippets/docs_snippets/guides/tutorials/etl_tutorial/etl_tutorial/definitions.py"
  language="python"
  lineStart="211"
  lineEnd="214"
/>

2. パーティションが定義されたので、製品カテゴリのパフォーマンスを計算するアセットでそれを使用できます:

```python
@dg.asset(
    deps=[joined_data],
    partitions_def=product_category_partition,
    group_name="analysis",
    compute_kind="duckdb",
)
def product_performance(context: dg.AssetExecutionContext, duckdb: DuckDBResource):
    product_category_str = context.partition_key

    with duckdb.get_connection() as conn:
        conn.execute(
            f"""
            create table if not exists product_performance (
                product_category varchar, 
                product_name varchar,
                total_dollar_amount double,
                total_units_sold double
            );

            delete from product_performance where product_category = '{product_category_str}';

            insert into product_performance
            select
                '{product_category_str}' as product_category,
                product_name,
                sum(dollar_amount) as total_dollar_amount,
                sum(quantity) as total_units_sold
            from joined_data 
            where category = '{product_category_str}'
            group by '{product_category_str}', product_name;
            """
        )
        preview_query = f"select * from product_performance where product_category = '{product_category_str}';"
        preview_df = conn.execute(preview_query).fetchdf()
        row_count = conn.execute(
            f"""
            SELECT COUNT(*)
            FROM product_performance
            WHERE product_category = '{product_category_str}';
            """
        ).fetchone()
        count = row_count[0] if row_count else 0

    return dg.MaterializeResult(
        metadata={
            "row_count": dg.MetadataValue.int(count),
            "preview": dg.MetadataValue.md(preview_df.to_markdown(index=False)),
        }
    )
```

## 3. パーティション化されたアセットを実体化する

パーティション化されたアセットができたので、それを Definitions オブジェクトに追加しましょう:

定義オブジェクトは次のようになります:

```python
defs = dg.Definitions(
    assets=[products,
        sales_reps,
        sales_data,
        joined_data,
        monthly_sales_performance,
        product_performance,
    ],
    asset_checks=[missing_dimension_check],
    resources={"duckdb": DuckDBResource(database="data/mydb.duckdb")},
)
```

これらのアセットを実体化するには:

1. assets ページに移動します。
2. Reload definitions.
3. `monthly_sales_performance` アセットを選択し、**Materialize selected** します。 .
4. すべてのパーティションが選択されていることを確認してから、バックフィルを開始します。
5. `product_performance` アセットを選択し、 **Materialize selected** します。
6. すべてのパーティションが選択されていることを確認してから、バックフィルを開始します。

## 次は

ETL パイプラインに主要なアセットが揃ったので、[パイプラインに自動化](/etl-pipeline-tutorial/automate-your-pipeline)を追加します。
