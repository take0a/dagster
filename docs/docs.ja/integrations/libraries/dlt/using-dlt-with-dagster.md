---
title: 'Dagster で dlt を使用する'
description: Ingest data with ease using Dagster and dlt
---

[データ ロード ツール (dlt)](https://dlthub.com/) オープンソース ライブラリは、乱雑になりがちなデータ ソースを構造化されたデータ セットにロードするデータ パイプラインを作成するための標準化されたアプローチを定義します。次のような高度な機能が多数用意されています。

- 接続シークレットの処理
- 宛先に必要な構造へのデータの変換
- 増分更新とマージ

dlt は、[事前に構築された検証済みのソース](https://dlthub.com/docs/dlt-ecosystem/verified-sources/) と [宛先](https://dlthub.com/docs/dlt-ecosystem/destinations/) の大規模なコレクションも提供しており、dlt コミュニティの成果を活用することで、記述するコード (まったく記述しなくてもかまいません!) を減らすことができます。

このガイドでは、dlt 統合の仕組み、dlt 用の Dagster プロジェクトの設定方法、定義済みの dlt ソースの使用方法について説明します。

## 仕組み

Dagster dlt 統合では、[multi-assets](/guides/build/assets/defining-assets#multi-asset) を使用します。これは、複数のアセットを生成する単一の定義です。これらのアセットは、`DltSource` から派生します。

以下は、ソースが 2 つのリソースで構成されている dlt ソース定義の例です。

```python
@dlt.source
def example(api_key: str = dlt.secrets.value):
    @dlt.resource(primary_key="id", write_disposition="merge")
    def courses():
        response = requests.get(url=BASE_URL + "courses")
        response.raise_for_status()
        yield response.json().get("items")

    @dlt.resource(primary_key="id", write_disposition="merge")
    def users():
        for page in _paginate(BASE_URL + "users"):
            yield page

    return courses, users
```

各リソースは API エンドポイントをクエリし、データ ウェアハウスにロードするデータを生成します。ソースで定義された 2 つのリソースは、Dagster アセットにマップされます。

次に、データのロード方法を指定する dlt パイプラインを定義しました:

```python
pipeline = dlt.pipeline(
    pipeline_name="example_pipeline",
    destination="snowflake",
    dataset_name="example_data",
    progress="log",
)
```

dlt ソースとパイプラインは、dlt を使用してデータをロードするために必要な 2 つのコンポーネントです。これらは、dlt と Dagster を統合するマルチアセットのパラメーターになります。

## 前提条件

このガイドの手順を実行するには、次のものが必要です:

- **dlt をこれまで使用したことがない場合は、[dlt の概要](https://dlthub.com/docs/intro)** をお読みください。
- **次のライブラリを[インストールする](/getting-started/installation)**:

  ```bash
  pip install dagster dagster-dlt
  ```

  `dagster-dlt` をインストールすると、`dlt` パッケージもインストールされます。

## Step 1: dltをサポートするようにDagsterプロジェクトを構成する

最初のステップは、データの取り込みに使用する `dlt` コードの場所を定義することです。Dagster プロジェクトのルートに `dlt_sources` ディレクトリを作成することをお勧めしますが、このコードは Python プロジェクト内のどこにでも配置できます。

`dlt_sources` ディレクトリを作成するには、以下を実行します:

```bash
cd $DAGSTER_HOME && mkdir dlt_sources
```

## Step 2: dlt 取り込みコードを初期化する

`dlt_sources` ディレクトリでは、[dlt チュートリアル](https://dlthub.com/docs/tutorial/load-data-from-an-api) に従って取り込みコードを記述するか、検証済みのソースを使用できます。

この例では、dlt が提供する [GitHub ソース](https://dlthub.com/docs/dlt-ecosystem/verified-sources/github) を使用します。

1. 次のコマンドを実行して、dlt ソース コードの場所を作成し、GitHub ソースを初期化します:

   ```bash
   cd dlt_sources

   dlt init github snowflake
   ```

   この時点で、コマンド ラインに次の内容が表示されます:

   ```bash
   Looking up the init scripts in https://github.com/dlt-hub/verified-sources.git...
   Cloning and configuring a verified source github (Source that load github issues, pull requests and reactions for a specific repository via customizable graphql query. Loads events incrementally.)
   ```

2. 続行するように求められたら、「y」と入力します。GitHub ソースがプロジェクトに追加されたことを確認する次の画面が表示されます:

   ```bash
   Verified source github was added to your project!
   * See the usage examples and code snippets to copy from github_pipeline.py
   * Add credentials for snowflake and other secrets in ./.dlt/secrets.toml
   * requirements.txt was created. Install it with:
   pip3 install -r requirements.txt
   * Read https://dlthub.com/docs/walkthroughs/create-a-pipeline for more information
   ```

これにより、GitHub API からデータを収集するために必要なコードがダウンロードされました。また、`requirements.txt` と `.dlt/` 構成ディレクトリも作成されました。パイプラインは Dagster を通じて構成するため、これらのファイルは削除できますが、参照すると役立つ場合があります。

```bash
$ tree -a
.
├── .dlt               # can be removed
│   ├── .sources
│   ├── config.toml
│   └── secrets.toml
├── .gitignore
├── github
│   ├── README.md
│   ├── __init__.py
│   ├── helpers.py
│   ├── queries.py
│   └── settings.py
├── github_pipeline.py
└── requirements.txt   # can be removed
```

## Step 3: dlt 環境変数を定義する

この統合では、`dlt` として環境変数を使用して接続とシークレットを管理します。`dlt` ライブラリは、ソースとリソースで使用される必要な環境変数を推測できます。詳細については、[dlt のシークレットと構成](https://dlthub.com/docs/general-usage/credentials/configuration) ドキュメントを参照してください。

これまで使用してきた例では、次のようになります:

- `github_reactions` ソースには GitHub アクセス トークンが必要です
- Snowflake の宛先にはデータベース接続の詳細が必要です

これにより、次の環境変数が必要になります:

```bash
SOURCES__GITHUB__ACCESS_TOKEN=""
DESTINATION__SNOWFLAKE__CREDENTIALS__DATABASE=""
DESTINATION__SNOWFLAKE__CREDENTIALS__PASSWORD=""
DESTINATION__SNOWFLAKE__CREDENTIALS__USERNAME=""
DESTINATION__SNOWFLAKE__CREDENTIALS__HOST=""
DESTINATION__SNOWFLAKE__CREDENTIALS__WAREHOUSE=""
DESTINATION__SNOWFLAKE__CREDENTIALS__ROLE=""
```

これらの変数が、ローカルで実行している場合は `.env` ファイルで、または [Dagster デプロイメントの環境変数](/guides/deploy/using-environment-variables-and-secrets) で環境で定義されていることを確認します。

## Step 4: DagsterDltResourceを定義する

次に、dlt パイプライン ランナーのラッパーを提供する <PyObject section="libraries" module="dagster_dlt" object="DagsterDltResource" /> を定義します。以下を使用して、すべての dlt パイプラインで共有できるリソースを定義します:

```python
from dagster_dlt import DagsterDltResource

dlt_resource = DagsterDltResource()
```

後の手順で、 <PyObject section="definitions" module="dagster" object="Definitions" /> にリソースを追加します。

## Step 5: GitHub 用の dlt_assets 定義を作成する

<PyObject section="libraries" object="dlt_assets" module="dagster_dlt" decorator /> デコレータは、`dlt_source` および `dlt_pipeline` パラメータを受け取ります。この例では、`github_reactions` ソースを使用し、`dlt_pipeline` を作成して Github から Snowflake にデータを取り込みました。

Dagster アセットを含む同じファイルで、次のようにして <PyObject section="libraries" object="dlt_assets" module="dagster_dlt" decorator /> のインスタンスを作成できます。

:::

[sql_database](https://dlthub.com/docs/api_reference/sources/sql_database/__init__#sql_database) ソースを使用している場合は、データベースの読み取りを減らすために `defer_table_reflect=True` を設定することを検討してください。デフォルトでは、Dagster デーモンはおよそ 1 分ごとに定義を更新し、データベースに対してリソース定義を照会します。

:::

```python
from dagster import AssetExecutionContext, Definitions
from dagster_dlt import DagsterDltResource, dlt_assets
from dlt import pipeline
from dlt_sources.github import github_reactions


@dlt_assets(
    dlt_source=github_reactions(
        "dagster-io", "dagster", max_items=250
    ),
    dlt_pipeline=pipeline(
        pipeline_name="github_issues",
        dataset_name="github",
        destination="snowflake",
        progress="log",
    ),
    name="github",
    group_name="github",
)
def dagster_github_assets(context: AssetExecutionContext, dlt: DagsterDltResource):
    yield from dlt.run(context=context)
```

## Step 6: 定義オブジェクトを作成する

最後のステップは、アセットとリソースを <PyObject section="definitions" module="dagster" object="Definitions" /> オブジェクトに含めることです。これにより、Dagster ツールは定義したすべてのものを読み込むことができます:

```python
defs = Definitions(
    assets=[
        dagster_github_assets,
    ],
    resources={
        "dlt": dlt_resource,
    },
)
```

これで完了です。これで、対応する Snowflake テーブルにデータをロードする 2 つのアセットが作成されました。1 つは問題用、もう 1 つはプル リクエスト用です。

## 高度な使い方

### トランスレータをオーバーライドして dlt アセットをカスタマイズする

<PyObject section="libraries" module="dagster_dlt" object="DagsterDltTranslator" /> オブジェクトを使用すると、dlt プロパティを Dagster コンセプトにマップする方法をカスタマイズできます。

たとえば、アセットの名前の派生方法を変更する場合や、上流のソース アセットのキーを変更する場合は、<PyObject section="libraries" module="dagster_dlt" object="DagsterDltTranslator" method="get_asset_spec" /> メソッドをオーバーライドできます。

<CodeExample path="docs_snippets/docs_snippets/integrations/dlt/dlt_dagster_translator.py" />

この例では、トランスレータをカスタマイズして、dlt アセットの名前の定義方法を変更しました。また、アセットの上流にアセット依存関係をハードコードして、単一の依存関係から dlt アセットへのファンアウト モデルを提供しました。

### 上流の外部アセットにメタデータを割り当てる

よくある質問は、dlt アセットの上流にある外部アセットのメタデータをどのように定義するかということです。

これは、<PyObject section="assets" module="dagster" object="AssetSpec" /> メソッドで定義されたキーと一致するキーを持つ <PyObject section="libraries" module="dagster_dlt" object="DagsterDltTranslator" method="get_asset_spec" /> を定義することで実現できます。

たとえば、`thinkific_assets` という名前の dlt アセットのセットを定義したとします。これらのアセットを反復処理して、`group_name` などの属性を持つ <PyObject section="assets" module="dagster" object="AssetSpec" /> を導出できます。

<CodeExample path="docs_snippets/docs_snippets/integrations/dlt/dlt_source_assets.py" />

### dlt アセットでのパーティションの使用

dlt アセット内でパーティションを使用することは可能です。ただし、状態は dlt によって管理されるため、同時実行性に関する問題が発生する可能性があることに注意してください。このため、パーティション化された dlt アセットには同時実行性の制限を設定することをお勧めします。詳細については、[データ パイプラインでの同時実行性の制限](/guides/operate/managing-concurrency) ガイドを参照してください。

とはいえ、DLT ソースからの静的な名前付きパーティションを使用する例を次に示します。

<CodeExample path="docs_snippets/docs_snippets/integrations/dlt/dlt_partitions.py" />

## 次は？

実際の運用環境での dlt の例をご覧になりたいですか? Dagster の [Dagster Open Platform](https://github.com/dagster-io/dagster-open-platform) プロジェクトで、Dagster 社内でどのように dlt が使用されているかを確認してください。
