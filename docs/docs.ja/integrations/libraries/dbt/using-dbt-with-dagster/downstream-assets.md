---
title: '下流のアセットを追加する'
description: Dagster can orchestrate dbt alongside other technologies.
sidebar_position: 400
---

ここまでで、[dbt プロジェクトをセットアップ](/integrations/libraries/dbt/using-dbt-with-dagster/set-up-dbt-project)、[dbt モデルをアセットとして Dagster にロード](/integrations/libraries/dbt/using-dbt-with-dagster/load-dbt-models)、[dbt モデルのアップストリーム アセットを定義](/integrations/libraries/dbt/using-dbt-with-dagster/upstream-assets)しました。

このステップでは、次の操作を行います。

- [plotly ライブラリをインストールする](#step-1-install-the-plotly-library)
- [plotly を使用してチャートを計算するダウンストリーム アセットを定義する](#step-2-define-the-order_count_chart-asset)
- [`order_count_chart` アセットをマテリアライズする](#step-3-materialize-the-order_count_chart-asset)

## Step 1: plotly ライブラリをインストールする {#step-1-install-the-plotly-library}

```shell
pip install plotly
```

## Step 2: order_count_chartアセットを定義する {#step-2-define-the-order_count_chart-asset}

これまで、データ パイプラインに上流アセットを追加しましたが、下流アセットは追加していませんでした。このステップでは、`customers` dbt モデルのデータを使用して、顧客ごとの注文数のプロット チャートを計算する `order_count_chart` という Dagster アセットを定義します。

[前のセクション](/integrations/libraries/dbt/using-dbt-with-dagster/upstream-assets#step-2-define-an-upstream-dagster-asset) で追加した `raw_customers` アセットと同様に、このアセットを `assets.py` ファイルの `jaffle_dagster` ディレクトリ内に配置します。

`order_count_chart` アセットを追加するには:

1. imports セクションを次の内容に置き換えます:

   <CodeExample
     path="docs_snippets/docs_snippets/integrations/dbt/tutorial/downstream_assets/assets.py"
     startAfter="start_imports"
     endBefore="end_imports"
   />

   これにより、plotly のインポートと、アセットで使用する <PyObject section="libraries" module="dagster_dbt" object="get_asset_key_for_model" /> および <PyObject section="metadata" module="dagster" object="MetadataValue" /> が追加されます。

2. `jaffle_shop_dbt_assets` の定義の後に、`order_count_chart` アセットの定義を追加します:

   <CodeExample
     path="docs_snippets/docs_snippets/integrations/dbt/tutorial/downstream_assets/assets.py"
     startAfter="start_downstream_asset"
     endBefore="end_downstream_asset"
   />

   このアセット定義は、前のセクションで定義したアセットに似ています。この場合、外部ソースからデータを取得して DuckDB に書き込むのではなく、DuckDB からデータを読み取り、それを使用してプロットを作成します。

   `deps=[get_asset_key_for_model([jaffle_shop_dbt_assets], "customers")]` という行は、このアセットが `customers` dbt モデルの下流にあることを Dagster に伝えます。この依存関係は、Dagster の UI にそのように表示されます。両方を具体化するために実行を開始すると、Dagster は `customers` が完了するまで `order_count_chart` を実行しません。

3. `order_count_chart` を `Definitions` に追加します:

   <CodeExample
     path="docs_snippets/docs_snippets/integrations/dbt/tutorial/downstream_assets/definitions.py"
     startAfter="start_defs"
     endBefore="end_defs"
   />

## Step 3: `order_count_chart` アセットをマテリアライズする {#step-3-materialize-the-order_count_chart-asse}

前のセクションから Dagster UI がまだ実行されている場合は、右上隅の「定義の再読み込み」ボタンをクリックします。シャットダウンした場合は、前のセクションと同じコマンドで再度起動できます:

```shell
dagster dev
```

UI は次のようになります:

![Asset group with dbt models and Python asset](/images/integrations/dbt/using-dbt-with-dagster/downstream-assets/asset-graph.png)

`order_count_chart` という名前の新しいアセットが、`customers` アセットの下流の一番下にあります。`order_count_chart` をクリックし、**Materialize selected** をクリックします。

これで完了です。実行が正常に完了すると、次のチャートがブラウザで自動的に開きます。

![plotly chart asset displayed in Chrome](/images/integrations/dbt/using-dbt-with-dagster/downstream-assets/order-count-chart.png)

## 次は？

これでこのチュートリアルは終了です。おめでとうございます。これで、dbt と Dagster の統合が機能し、Dagster アセットがいくつかマテリアライズされているはずです。

次は何をするのでしょうか。ここから、次のことができます。

- [アセット定義](/guides/build/assets/) の詳細を学ぶ
- [dbt アセットをマテリアライズするジョブを作成する](/integrations/libraries/dbt/reference#scheduling-dbt-jobs) 方法を学ぶ
- [Dagster の dbt 統合についてさらに理解する](/integrations/libraries/dbt/reference)
- [`dagster-dbt` API ドキュメント](/api/python-api/libraries/dagster-dbt) を確認する
