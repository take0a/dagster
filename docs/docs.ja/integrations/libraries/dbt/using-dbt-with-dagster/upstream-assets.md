---
title: 'dbtモデルの上流のアセットを定義する'
description: Dagster can orchestrate dbt alongside other technologies.
sidebar_position: 300
---

この時点で、[dbt プロジェクトをセットアップ](/integrations/libraries/dbt/using-dbt-with-dagster/set-up-dbt-project)し、[dbt モデルをアセットとして Dagster にロード](/integrations/libraries/dbt/using-dbt-with-dagster/load-dbt-models)しました。

ただし、パイプラインのルートにあるテーブルは静的です。これらは [dbt シード](https://docs.getdbt.com/docs/build/seeds) であり、dbt プロジェクトにハードコードされている CSV です。より現実的なデータ パイプラインでは、これらのテーブルは通常、Airbyte や Fivetran などのツールや Python コードを使用して、外部データ ソースから取り込まれます。

パイプラインのこれらの取り込み手順は、dbt 内で定義しても意味がないことがよくありますが、Dagster アセットとして定義すれば意味があることはよくあります。 Dagster アセット定義は、dbt モデルのより一般的なバージョンと考えることができます。dbt モデルはアセットの一種ですが、別の種類は、Dagster の Python API を使用して Python で定義されるものです。dbt 統合リファレンス ページには、dbt モデルと Dagster アセット定義の類似点を概説した [セクション](/integrations/libraries/dbt/reference#dbt-models-and-dagster-asset-definitions) が含まれています。

このセクションでは、`raw_customers` dbt シードを、それを表す Dagster アセットに置き換えます。Web からデータを取得してこのテーブルにデータを入力する Python コードを記述します。これにより、最初に Python コードを実行して `raw_customers` テーブルにデータを入力し、次に dbt を呼び出してダウンストリーム テーブルにデータを入力する実行を開始できます。

次の操作を行います:

- [Pandas および DuckDB Python ライブラリをインストールします](#step-1-install-the-pandas-and-duckdb-python-libraries)
- [上流の Dagster アセットを定義します](#step-2-define-an-upstream-dagster-asset)
- [dbt プロジェクトで、シードをソースに置き換えます](#step-3-in-the-dbt-project-replace-a-seed-with-a-source)
- [Dagster UI を使用してアセットをマテリアライズします](#step-4-materialize-the-assets-using-the-dagster-ui)

## Step 1: Pandas および DuckDB Python ライブラリをインストールします {#step-1-install-the-pandas-and-duckdb-python-libraries}

書き込む Dagster アセットは、[Pandas](https://pandas.pydata.org/) を使用してデータを取得し、[DuckDB の Python API](https://duckdb.org/docs/api/python/overview.html) を使用して DuckDB ウェアハウスに書き出します。これらを使用するには、以下をインストールする必要があります:

```shell
pip install pandas duckdb pyarrow
```

## Step 2: 上流の Dagster アセットを定義します {#step-2-define-an-upstream-dagster-asset}

dbt モデルに必要なデータを取得するには、`raw_customers` 用の Dagster アセットを作成します。このアセットを `assets.py` ファイルの `jaffle_dagster` ディレクトリ内に配置します。これは、[最後のセクション](/integrations/libraries/dbt/using-dbt-with-dagster/load-dbt-models#step-4-understand-the-python-code-in-your-dagster-project) の最後で確認した dbt モデルを定義するコードを含むファイルです。このコードをコピーして貼り付け、そのファイルの既存の内容を上書きします。

<CodeExample
  path="docs_snippets/docs_snippets/integrations/dbt/tutorial/upstream_assets/assets.py"
  startAfter="start_python_assets"
  endBefore="end_python_assets"
/>

行った変更を確認しましょう:

1. 上部に、`pandas` と `duckdb` のインポートを追加しました。これらは、`DataFrame` にデータを取得して DuckDB に書き込むために使用します。

2. DuckDB データベースの場所を保持する `duckdb_database_path` 変数を追加しました。DuckDB データベースは、ローカル ファイル システム上の通常のファイルであることに注意してください。パスは、`profiles.yml` ファイルを構成するときに使用したパスと同じです。この変数は、`raw_customers` アセットの実装で使用されます。

3. `raw_customers` という名前の関数を記述し、<PyObject section="assets" module="dagster" object="asset" decorator /> デコレータで装飾して、`raw_customers` テーブルの定義を追加しました。 Dagster UI でこれが Python で定義されたアセットであることを示すために、`compute_kind="python"` というラベルを付けました。関数内の実装は、インターネットからデータを取得し、それを DuckDB データベースのテーブルに書き込みます。dbt モデルを実行すると select ステートメントが実行されるのと同様に、このアセットを具体化すると、この Python コードが実行されます。

最後に、`definitions.py` の `Definitions` オブジェクトの `assets` 引数を更新して、定義したばかりの新しいアセットを含めます:

<CodeExample
  path="docs_snippets/docs_snippets/integrations/dbt/tutorial/upstream_assets/definitions.py"
  startAfter="start_defs"
  endBefore="end_defs"
/>

## Step 3: dbt プロジェクトで、シードをソースに置き換えます {#step-3-in-the-dbt-project-replace-a-seed-with-a-source}

1. これを Dagster アセットに置き換えるため、`raw_customers` の dbt シードは不要になり、削除できます:

   ```shell
   cd ..
   rm seeds/raw_customers.csv
   ```

2. 代わりに、`raw_customers` は dbt プロジェクトの外部で定義されたテーブルであることを dbt に伝えます。これを行うには、[dbt ソース](https://docs.getdbt.com/docs/build/sources) 内で定義します。

   `models/` ディレクトリ内に `sources.yml` というファイルを作成し、その中に次のコードを配置します:

   ```yaml
   version: 2

   sources:
     - name: jaffle_shop
       tables:
         - name: raw_customers
           meta:
             dagster:
               asset_key: ['raw_customers'] # This metadata specifies the corresponding Dagster asset for this dbt source.
   ```

   これは標準の dbt ソース定義ですが、1 つ追加されています。`meta` プロパティの下に、対応する Dagster アセットを指定するメタデータが含まれています。Dagster は dbt プロジェクトのコンテンツを読み取るときに、このメタデータを読み取って対応を推測します。この dbt ソースに依存するすべての dbt モデルについて、Dagster は、dbt モデルに対応する Dagster アセットがソースに対応する Dagster アセットに依存する必要があることを認識します。

3. 次に、`raw_customers` シードに依存するモデルを更新して、ソースに依存するようにします。`model/staging/stg_customers.sql` の内容を次のように置き換えます:

   ```sql
   with source as (

       {#-
       Use source instead of seed:
       #}
       select * from {{ source('jaffle_shop', 'raw_customers') }}

   ),

   renamed as (

       select
           id as customer_id,
           first_name,
           last_name

       from source

   )

   select * from renamed
   ```

## Step 4: Dagster UI を使用してアセットをマテリアライズします {#step-4-materialize-the-assets-using-the-dagster-ui}

前のセクションから Dagster UI がまだ実行されている場合は、右上隅の「定義の再読み込み」ボタンをクリックします。シャットダウンした場合は、前のセクションと同じコマンドで再度起動できます:

```shell
dagster dev
```

これで、`raw_customers` モデルが Python アセットとして定義されました。また、`raw_customers` のコード定義が変更されたため、`stg_customers` や `customers` など、この新しい Python アセットの下流のアセットが古くなったとマークされていることもわかります。

![Asset group with dbt models and Python asset](/images/integrations/dbt/using-dbt-with-dagster/upstream-assets/asset-graph.png)

**Materialize all** ボタンをクリックします。これにより、次の 2 つの手順で実行が開始されます。

- `raw_customers` Python 関数を実行してデータを取得し、`raw_customers` テーブルを DuckDB に書き込みます。
- 前のセクションと同様に、`dbt build` を使用してすべての dbt モデルを実行します。

クリックして実行を表示すると、これらの手順のグラフィカルな表現とログが表示されます。

![Run page for run with dbt models and Python asset](/images/integrations/dbt/using-dbt-with-dagster/upstream-assets/run-page.png)

## 次は？

この時点で、上流の Dagster アセットを構築してマテリアライズし、dbt モデルにソース データを提供しました。チュートリアルの最後のセクションでは、[パイプラインに下流のアセットを追加する](/integrations/libraries/dbt/using-dbt-with-dagster/downstream-assets) 方法を説明します。
