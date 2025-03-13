---
title: "I/O マネージャー"
sidebar_position: 50
---

Dagster の I/O マネージャーを使用すると、データ処理用のコードとデータの読み取りおよび書き込み用のコードを分離できます。これにより、繰り返しコードが削減され、データの保存場所の変更が容易になります。

多くの Dagster パイプラインでは、アセットは次の手順で分類できます:

1. データストアからメモリにデータを読み込む
2. メモリ内の変換の適用
3. 変換されたデータをデータストアに書き込む

このパターンに従うアセットの場合、I/O マネージャーはソースとの間のデータの読み取りと書き込みを処理するコードを合理化できます。

:::note

この記事は、[アセット](/guides/build/assets/)と[リソース](/guides/build/external-resources/)を理解していることを前提としています。

:::

## 始める前に

**I/O マネージャーは Dagster を使用するために必須ではなく、すべてのシナリオで最適なオプションでもありません。** 各アセットの最初と最後に同じコードを記述してデータをロードおよび保存する場合は、I/O マネージャーが役立つ場合があります。例:

- 同じ場所に保管されているアセットがあり、保管パスを決定するために一貫した一連のルールに従っている
- ローカル、ステージング、本番環境で異なる方法で保存されているアセットがある
- 計算を行うために上流の依存関係をメモリにロードするアセットがある

**I/O マネージャーは、次の場合には最適ではない可能性があります:**

- データベース内のテーブルを作成または更新するSQLクエリを実行したい
- パイプラインが、ストレージに書き込む他のライブラリ/ツールを使用して、独自にI/Oを管理する
- 数十億行のデータベーステーブルなど、アセットがメモリに収まらない

一般的なルールとして、I/O マネージャーを使用するためにパイプラインが複雑になる場合は、I/O マネージャーは適していない可能性があります。このような場合は、`deps` を使用して [依存関係を定義](/guides/build/assets/passing-data-between-assets) する必要があります。

## アセットでの I/O マネージャーの使用 \{#io-in-assets}

次の例を検討してください。この例には、DuckDB 接続オブジェクトを構築し、上流テーブルからデータを読み取り、メモリ内変換を適用し、その結果を DuckDB の新しいテーブルに書き込むアセットが含まれています:

<CodeExample path="docs_beta_snippets/docs_beta_snippets/guides/external-systems/assets-without-io-managers.py" language="python" />

I/O マネージャーを使用すると、アセット自体からデータを読み書きするコードが削除され、代わりに I/O マネージャーに委任されます。アセットには、変換を適用するコードや初期 CSV ファイルを取得するコードのみが残ります。

<CodeExample path="docs_beta_snippets/docs_beta_snippets/guides/external-systems/assets-with-io-managers.py" language="python" />

I/O マネージャーを使用して上流アセットをロードするには、アセット関数への入力パラメーターとしてアセットを指定します。この例では、`DuckDBPandasIOManager` I/O マネージャーは、上流アセット (`raw_sales_data`) と同じ名前の DuckDB テーブルを読み取り、そのデータを Pandas DataFrame として `clean_sales_data` に渡します。

I/O マネージャーを使用してデータを保存するには、アセット関数でデータを返します。返されるデータは有効な型である必要があります。この例では Pandas DataFrames を使用し、`DuckDBPandasIOManager` はアセットと同じ名前で DuckDB テーブルに書き込みます。

有効なタイプとデータの保存方法の詳細については、個々の I/O マネージャーのドキュメントを参照してください。

## データストアのスワップ \{#swap-data-stores}

I/O マネージャーでは、データストアのスワッピングは、I/O マネージャーの実装の変更で構成されます。変換ロジックのみを含むアセット定義を変更する必要はありません。

次の例では、Snowflake I/O マネージャーが DuckDB I/O マネージャーに置き換えられました。

<CodeExample path="docs_beta_snippets/docs_beta_snippets/guides/external-systems/assets-with-snowflake-io-manager.py" language="python" />

## 組み込み I/O マネージャー \{#built-in}

Dagster は、一般的なデータ ストアとメモリ内形式用の I/O マネージャー用の組み込みライブラリ実装を提供します。

| Name                                                                                       | Description                                                                   |
| ------------------------------------------------------------------------------------------ | ----------------------------------------------------------------------------- |
| <PyObject section="io-managers" module="dagster" object="FilesystemIOManager" />                                 | デフォルトの I/O マネージャー。出力をローカル ファイル システムに pickle ファイルとして保存します。|
| <PyObject section="io-managers" module="dagster" object="InMemoryIOManager" />                                   | 出力をメモリに保存します。主にユニット テストに役立ちます。                  |
| <PyObject section="libraries" module="dagster_aws" object="s3.S3PickleIOManager" />                            | 出力をピクルファイルとして Amazon Web Services S3 に保存します。                    |
| <PyObject section="libraries" module="dagster_azure" object="adls2.ConfigurablePickledObjectADLS2IOManager" /> | 出力を Azure ADLS2 に pickle ファイルとして保存します。                                |
| <PyObject section="libraries" module="dagster_gcp" object="GCSPickleIOManager" />                              | 出力を Google Cloud Platform GCS に pickle ファイルとして保存します。              |
| <PyObject section="libraries" module="dagster_gcp_pandas" object="BigQueryPandasIOManager" />                  | Pandas DataFrame 出力を Google Cloud Platform BigQuery に保存します。          |
| <PyObject section="libraries" module="dagster_gcp_pyspark" object="BigQueryPySparkIOManager" />                | PySpark DataFrame 出力を Google Cloud Platform BigQuery に保存します。           |
| <PyObject section="libraries" module="dagster_snowflake_pandas" object="SnowflakePandasIOManager" />           | Pandas DataFrame 出力を Snowflake に保存します。                             |
| <PyObject section="libraries" module="dagster_snowflake_pyspark" object="SnowflakePySparkIOManager" />         | PySpark DataFrame 出力を Snowflake に保存します。                              |
| <PyObject section="libraries" module="dagster_duckdb_pandas" object="DuckDBPandasIOManager" />                 | SPandas DataFrame 出力を DuckDB に保存します。                                  |
| <PyObject section="libraries" module="dagster_duckdb_pyspark" object="DuckDBPySparkIOManager" />               | PySpark DataFrame 出力を DuckDB に保存します。                                 |
| <PyObject section="libraries" module="dagster_duckdb_polars" object="DuckDBPolarsIOManager" />                 | Polars DataFrame 出力を DuckDB に保存します。                                   |                                       |

## 次は

- リソースを使用して[データベースに接続](/guides/build/external-resources/connecting-to-databases)する方法を学ぶ
- リソースを使用して [API を接続する](/guides/build/external-resources/connecting-to-apis) 方法を学ぶ
