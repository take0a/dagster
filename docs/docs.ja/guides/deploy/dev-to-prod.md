---
title: '開発から本番への移行'
sidebar_position: 500
---

この記事では、データパイプラインをローカル開発環境からステージング環境、そして本番環境に移行する方法について解説します。

例えば、Hacker News から最新の N 件のエントリを取得し、そのデータを 2 つのデータセットに分割するタスクがあるとします。1 つは記事に関するすべてのデータ、もう 1 つはコメントに関するすべてのデータです。
パイプラインを保守性とテスト性に優れたものにするために、さらに 2 つの要件があります。

- データパイプラインをローカル環境、ステージング環境、本番環境で実行できること。
- データが誤って上書きされないことを保証できること（例：ユーザーが設定値の変更を忘れた場合など）。

Using a few Dagster concepts, we can easily tackle this task! 
Here’s an overview of the main concepts we’ll be using in this guide:

- [アセット](/guides/build/assets/) - アセットとは、データアセットをモデル化するソフトウェアオブジェクトです。
代表的な例としては、データベース内のテーブルやクラウドストレージ内のファイルなどが挙げられます。
- [リソース](/guides/build/external-resources) - リソースとは、（通常は）外部サービスへの接続をモデル化するオブジェクトです。
リソースはアセット間で共有でき、環境に応じて異なる実装のリソースを使用できます。
例えば、リソースはSlackでメッセージを送信するメソッドを提供する場合があります。
- [I/Oマネージャー](/guides/build/io-managers/) - I/Oマネージャーは、アセットの保存と読み込みを処理する特別な種類のリソースです。
例えば、アセットをS3に保存したい場合は、Dagsterに組み込まれているS3 I/Oマネージャーを使用できます。
- [実行構成](/guides/operate/configuration/run-configuration) - アセットやリソースでは、データベースのパスワードなど、特定の値を設定するために構成が必要になる場合があります。
実行構成を使用すると、実行時にこれらの値を設定できます。
このガイドでは、APIを使用してデフォルトの実行構成を設定する方法も説明します。

これらのDagsterのコンセプトを用いて、以下のことを行います:

- 3つのアセットを作成します。Hacker Newsデータセット全体、コメントに関するデータ、ストーリーに関するデータです。
- DagsterのSnowflake I/Oマネージャーを使用して、データセットを[Snowflake](https://www.snowflake.com/)に保存します。
- コードが実行される環境に基づいて、Snowflake I/Oマネージャーの設定が自動的に提供されるように、Dagsterコードを設定します。

## 設定

<CodeReferenceLink filePath="examples/development_to_production" />

このガイドに従うには、完全なコード例をコピーし、いくつかの追加の pip ライブラリをインストールします。

```bash
dagster project from-example --name my-dagster-project --example development_to_production
cd my-dagster-project
pip install -e .
```

## パート1：ローカル開発

このセクションでは、以下の作業を行います:

- アセットの作成
- Snowflake I/O マネージャーの実行構成の追加
- Dagster UI でアセットのマテリアライズ

まずは、3つのアセットの作成から始めましょう。データの操作には Pandas DataFrame を使用します。

<CodeExample
  path="docs_snippets/docs_snippets/guides/dagster/development_to_production/assets.py"
  startAfter="start_assets"
  endBefore="end_assets"
/>

これで、これらのアセットを <PyObject section="definitions" module="dagster" object="Definitions" /> オブジェクトに追加し、ローカル開発ワークフローの一環として UI 経由でマテリアライズできるようになりました。
`SnowflakePandasIOManager` に認証情報を渡すことができます。

<CodeExample
  path="docs_snippets/docs_snippets/guides/dagster/development_to_production/repository/repository_v1.py"
  startAfter="start"
  endBefore="end"
/>

このコードスニペットでは、設定にパスワードが含まれていることに注意してください。
これは不適切な方法であり、すぐに修正する予定です。

結果として、アセットグラフは次のようになります。

![Hacker News asset graph](/images/guides/deploy/hacker_news_asset_graph.png)

UI でアセットを具体化し、データが期待どおりに Snowflake に表示されることを確認できます:

![Snowflake data](/images/guides/deploy/snowflake_data.png)

アセットをPandas DataFramesとして定義すると、Snowflake I/Oマネージャーが自動的にSnowflakeテーブルとの間でデータを変換します。
Pythonアセット名によってSnowflakeテーブル名が決まります。
この場合、`ITEMS`、`COMMENTS`、`STORIES`の3つのテーブルが作成されます。

## パート2: デプロイ

このセクションでは、以下の内容について説明します。

- ステージング環境と本番環境の両方に対応できるよう、Snowflake I/Oマネージャーの構成を変更する
- ステージング環境を管理するためのさまざまなオプションを検討する

アセットがローカルで動作するようになったので、デプロイプロセスを開始できます。
まず、本番環境向けにアセットを設定し、次にステージング環境へのデプロイのオプションを検討する。

### 本番

アセットを本番環境のSnowflakeデータベースに保存したいので、`SnowflakePandasIOManager`の設定を更新する必要があります。
しかし、ローカル開発用に設定した値を単純に更新すると、問題が発生します。開発者が次にこれらのアセットで作業する際に、設定をローカルの値に戻すことを忘れないようにする必要があるのです。
これでは、開発者がローカル開発中に誤って本番環境のアセットを上書きしてしまう可能性があります。

代わりに、環境に基づいてリソースの設定を決定することができます:

<CodeExample
  path="docs_snippets/docs_snippets/guides/dagster/development_to_production/repository/repository_v2.py"
  startAfter="start"
  endBefore="end"
/>

このコードスニペットでは、設定にパスワードがまだ残っていることに注意してください。
これは不適切な方法なので、次に解決します。

これで、デプロイメントで環境変数 `DAGSTER_DEPLOYMENT=production` を設定することができ、アセットに適切なリソースが適用されます。

この設定にはまだいくつか問題があります。

1. 開発者は、ローカルで開発する際に、`user` と `password` を自分の認証情報に、`schema` を自分の名前に変更することを忘れないようにする必要があります。
2. パスワードがコード内に保存されています。

これらの問題は、<PyObject section="resources" module="dagster" object="EnvVar"/> を使用することで簡単に解決できます。これにより、環境変数からリソースの設定を取得できます。
これにより、Snowflake の設定値を環境変数として保存し、I/O マネージャーにそれらの環境変数を参照させることができます。

<CodeExample
  path="docs_snippets/docs_snippets/guides/dagster/development_to_production/repository/repository_v3.py"
  startAfter="start"
  endBefore="end"
/>

### ステージング

組織の Dagster 設定に応じて、ステージング環境にはいくつかのオプションがあります。

- **Dagster+ ユーザーの場合** は、ステージング手順として [ブランチデプロイメント](/dagster-plus/features/ci-cd/branch-deployments/) を使用することをお勧めします。
ブランチデプロイメントは、Git ブランチごとに自動的に生成される新しい Dagster デプロイメントであり、本番環境にデプロイする前にデータパイプラインを検証するために使用できます。

- **セルフホスト型のステージングデプロイメントの場合** は、ステージング環境でアセットを実行するために必要な作業のほとんどは既に完了しています。
必要なのは、`resources` ディクショナリに別のエントリを追加し、ステージングデプロイメントで `DAGSTER_DEPLOYMENT=staging` を設定することだけです。

<CodeExample
  path="docs_snippets/docs_snippets/guides/dagster/development_to_production/repository/repository_v3.py"
  startAfter="start_staging"
  endBefore="end_staging"
/>

## 上級：スタブとモックを使ったユニットテスト

このガイドで紹介されている開発ワークフローには、ユニットテストというステップが抜けていることに気づいたかもしれません。このガイドの主な目的は、ローカル開発から本番環境へのコード移行を支援することですが、ユニットテストは開発サイクルにおいて依然として重要な部分です。
このセクションでは、独自のユニットテストを作成する際に役立つパターンを紹介します。

`items` アセットのユニットテストを作成する際に、Hacker News から受け取るデータが正確にわかっていれば、より正確なアサーションを行うことができます。
Hacker News API とのやり取りをリソースとしてリファクタリングすれば、Dagster のリソースシステムを活用してユニットテストでスタブリソースを提供できます。

実装に入る前に、いくつかのベストプラクティスを確認しましょう。

### リソースを使用するタイミング

多くの場合、外部サービスとのやり取りをリソースとしてリファクタリングするよりも、アセットやオペレーション内で直接行う方が便利です。
以下の場合には、リソースを使用するようにコードをリファクタリングすることをお勧めします。

- 複数のアセットまたはオペレーションが一貫した方法でサービスとやり取りする必要がある場合
- 特定のシナリオ（ステージング環境やユニットテストなど）で、サービスの異なる実装を使用する必要がある場合

### スタブリソースを使用する場合

ユニットテストのためにリソースをスタブ化することが適切なタイミングを判断することは、多くの議論の的となることがあります。
確かに、スタブの作成と保守が複雑すぎるリソースも存在します。
例えば、Snowflakeのようなデータベースを軽量データベースでモック化することは、SQL構文や実行時の動作が異なる可能性があるため困難です。
一般的に、リソースが比較的単純な場合、スタブを作成することは、そのリソースを使用するアセットやオペレーションのユニットテストに役立ちます。

まずは、「本物の」Hacker News APIクライアントを作成しましょう:

<CodeExample
  path="docs_snippets/docs_snippets/guides/dagster/development_to_production/resources/resources_v1.py"
  startAfter="start_resource"
  endBefore="end_resource"
/>

このクライアントをリソースとして使用するには、`items` アセットも更新する必要があります:

<CodeExample
  path="docs_snippets/docs_snippets/guides/dagster/development_to_production/assets_v2.py"
  startAfter="start_items"
  endBefore="end_items"
/>

:::note

簡潔にするために、`HNAPIClient` のプロパティ `item_field_names` の実装は省略しました。
このリソースの完全な実装は、GitHub の [完全なコード例](https://github.com/dagster-io/dagster/tree/master/examples/development_to_production) で確認できます。

:::

また、`Definitions` オブジェクトの `resources` に `HNAPIClient` のインスタンスを追加する必要があります。

<CodeExample
  path="docs_snippets/docs_snippets/guides/dagster/development_to_production/repository/repository_v3.py"
  startAfter="start_hn_resource"
  endBefore="end_hn_resource"
/>

これで、Hacker Newsリソースのスタブ版を作成できます。
スタブには、`HNAPIClient`が実装する各メソッドの実装が含まれていることを確認します。

<CodeExample
  path="docs_snippets/docs_snippets/guides/dagster/development_to_production/resources/resources_v2.py"
  startAfter="start_mock"
  endBefore="end_mock"
/>

:::note

スタブ Hacker News リソースと実際の Hacker News リソースは同じメソッドを実装する必要があるため、インターフェースを作成するのに最適なタイミングです。
このガイドでは実装は省略しますが、[完全なコード例](https://github.com/dagster-io/dagster/tree/master/examples/development_to_production) で確認できます。

:::

ここで、スタブ Hacker News リソースを使用して、`items` アセットが期待どおりにデータを変換することをテストできます:

<CodeExample
  path="docs_snippets/docs_snippets/guides/dagster/development_to_production/test_assets.py"
  startAfter="start"
  endBefore="end"
/>

:::note

この記事ではアセットに焦点を当てましたが、同じ概念と API を使用してジョブの実行構成を交換することもできます。

:::
