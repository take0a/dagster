---
title: "新しい Dagster プロジェクトの作成"
description: Dagster comes with a convenient CLI command for creating a new project. This guide explains the role of each generated file and directory.
sidebar_position: 100
---

Dagster プロジェクトの構築を開始する最も簡単な方法は、`dagster project` CLI を使用することです。この CLI ツールは、Dagster をすぐに使い始めることができるファイルとフォルダー構造を生成するのに役立ちます。


## Step 1: 新しいプロジェクトをブートストラップする

:::note

  Dagster がまだインストールされていない場合は、続行する前に、<a href="/getting-started/install">インストール要件</a>を満たしていることを確認してください。

:::

デフォルトのプロジェクトスケルトンを使用して新しいプロジェクトの土台を作ることも、公式の Dagster のサンプルの 1 つから開始することもできます。

Dagster プロジェクトのデフォルトファイルの詳細については、[Dagster プロジェクトファイルのリファレンス](/guides/build/projects/dagster-project-file-reference) を参照してください。

<Tabs>
<TabItem value="デフォルトのプロジェクトスケルトン">

### デフォルトのプロジェクトスケルトンを使用する

開始するには、次のコマンドを実行します:

```bash
pip install dagster
dagster project scaffold --name my-dagster-project
```

`dagster project scaffold` コマンドは、単一の Dagster コードの場所と、`pyproject.toml` や `setup.py` などのその他のファイルを含むフォルダー構造を生成します。これにより、空のプロジェクトで設定が行われ、すぐに開始できるようになります。

</TabItem>
<TabItem value="公式サンプル">

### 公式のサンプルを使用する

公式の Dagster のサンプルを使い始めるには、次のコマンドを実行します:

```bash
pip install dagster
dagster project from-example \
  --name my-dagster-project \
  --example quickstart_etl
```

コマンド `dagster project from-example` は、公式の Dagster サンプルの 1 つを現在のディレクトリにダウンロードします。このコマンドを使用すると、公式に管理されているサンプルを使用してプロジェクトをすばやくブートストラップできます。

サンプルの詳細については、[Dagster GitHub リポジトリ](https://github.com/dagster-io/dagster/tree/master/examples) にアクセスするか、`dagster project list-examples` を使用してください。

</TabItem>
</Tabs>

## Step 2: プロジェクトの依存関係をインストールする

新しく生成された `my-dagster-project` ディレクトリは完全に機能する [Python パッケージ](https://docs.python.org/3/tutorial/modules.html#packages) であり、 `pip` を使用してインストールできます。

パッケージとその Python 依存関係としてインストールするには、次を実行します:

```bash
pip install -e ".[dev]"
```

:::

  <code>--editable</code> (<code>-e</code>) フラグを使用すると、<code>pip</code>はコードの場所をPythonパッケージとしてインストールするように指示します。
  <a href="https://pip.pypa.io/en/latest/topics/local-project-installs/#editable-installs">
    "editable mode"
  </a>{" "}
  開発時にローカルコードの変更が自動的に適用されます。

:::

## Step 3: Dagster UI を起動する

[Dagster UI](/guides/operate/webserver)を起動するには、次を実行します:

```bash
dagster dev
```

**注**: このコマンドは、[Dagster デーモン](/guides/deploy/execution/dagster-daemon)も起動します。詳細については、[Dagster をローカルで実行するためのガイド](/guides/deploy/deployment-options/running-dagster-locally)を参照してください。

ブラウザを使用して [http://localhost:3000](http://localhost:3000) を開き、プロジェクトを表示します。

## Step 4: 開発

- [新しいPython依存関係の追加](#adding-new-python-dependencies)
- [環境変数とシークレット](#using-environment-variables-and-secrets)
- [ユニットテスト](#adding-and-running-unit-tests)

### 新しいPython依存関係の追加

`setup.py` で新しい Python 依存関係を指定できます。

### 環境変数とシークレットの使用

環境変数は、ソース コードの外部で設定されるキーと値のペアであり、環境に応じてアプリケーションの動作を動的に変更できます。

環境変数を使用すると、Dagster アプリケーションのさまざまな構成オプションを定義し、シークレットを安全に設定できます。たとえば、データベースの資格情報をハードコーディングする代わりに (これは悪い習慣であり、開発には面倒です)、環境変数を使用してユーザーの詳細を提供できます。これにより、コードを変更したり、機密データを安全でない状態で保存したりすることなく、パイプラインをパラメーター化できます。

詳細と例については、「[環境変数とシークレットの使用](/guides/deploy/using-environment-variables-and-secrets)」を参照してください。

### ユニットテストの追加と実行

テストは `my_dagster_project_tests` ディレクトリに追加し、`pytest` を使用して実行できます:

```bash
pytest my_dagster_project_tests
```

## 次は

プロジェクトを本番環境に移行する準備ができたら、[データ パイプラインを開発環境から本番環境に移行する](/guides/deploy/dev-to-prod) に関する推奨事項を確認してください。

展開オプションの詳細については、次のリソースを参照してください:

- [Dagster+](/dagster-plus/) - Dagsterが管理するインフラを使用してデプロイする
- [それ以外のインフラ](/guides/deploy/) - Docker、Kubernetes、Amazon Web Services などのインフラにデプロイします。
