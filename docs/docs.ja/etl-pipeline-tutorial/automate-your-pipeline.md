---
title: パイプラインの自動化
description: Set schedules and utilize asset based automation
last_update:
  author: Alex Noonan
sidebar_position: 50
---

[Dagster では](/guides/automate)、パイプラインとアセットを自動化する方法がいくつかあります。

このステップでは次の操作を行います:

- 上流のアセットが実体化されるときに実行されるように、アセットに自動化を追加します。
- cron スケジュールで一連のアセットを実行するスケジュールを作成します。

## 1. スケジュールされたジョブ

データ オーケストレーションでは、Cron ベースのスケジュールが一般的です。パイプラインでは、更新された CSV が毎週特定の時間に外部プロセスによってファイルの場所にアップロードされると想定します。

`product performance` アセットの下に次のコードをコピーします:

<CodeExample path="docs_beta_snippets/docs_beta_snippets/guides/tutorials/etl_tutorial/etl_tutorial/definitions.py" language="python" lineStart="268" lineEnd="273"/>

## 2. アセットの実体化を自動化する

これで、`monthly_sales_performance` は月に 1 回実行されるはずですが、このアセットに独立した月次スケジュールを設定するのは、まさに私たちが望んでいることではありません。単純に設定すると、このアセットは、先週のデータが完了する前に、ちょうど月の境界で実行されます。月次スケジュールを数時間遅らせて上流のアセットが完了する時間を与えることもできますが、上流の計算が失敗したり、完了までに時間がかかりすぎたりする場合はどうなるでしょうか。ここで、アセットの状態とそのすべての依存関係を理解する [宣言型自動化](/guides/automate/declarative-automation) を使用できます。

`monthly_sales_performance` については、すべての依存関係が更新されたときに更新されるようにします。これを実現するには、`eager` 自動化条件を使用します。`monthly_sales_performance` アセットを更新して、自動化条件をデコレータに追加します:

<CodeExample path="docs_beta_snippets/docs_beta_snippets/guides/tutorials/etl_tutorial/etl_tutorial/definitions.py" language="python" lineStart="155" lineEnd="209"/>

`product_performance` に対しても同じことを行います:

<CodeExample path="docs_beta_snippets/docs_beta_snippets/guides/tutorials/etl_tutorial/etl_tutorial/definitions.py" language="python" lineStart="216" lineEnd="267"/>

## 3. 自動化を有効にしてテストする

スケジュールができたので、それを Definitions オブジェクトに追加しましょう。

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
      schedules=[weekly_update_schedule],
      resources={"duckdb": DuckDBResource(database="data/mydb.duckdb")},
  )
  ```

最後のステップは、UI で自動化を有効にすることです。

これを実現するには:

1. Automation ページへ遷移します。
2. すべての自動化を選択します。
3. アクションを使用して、すべての自動化を開始します。
4. `analysis_update_job` を選択します。
5. スケジュールをテストし、ドロップダウンメニューで任意の時間を評価します。
6. Launchpad で開きます。

ジョブは現在実行中です。

さらに、Runs タブに移動すると、`monthly_sales_performance` と `product_performance` のマテリアライゼーションも実行されていることがわかります。

   ![2048 resolution](/images/tutorial/etl-tutorial/automation-final.png)

## 次は

- このチュートリアルを続けて、[センサーベースのアセット](create-a-sensor-asset)を追加します。
