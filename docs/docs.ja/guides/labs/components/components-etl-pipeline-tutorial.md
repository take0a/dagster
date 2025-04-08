---
title: 'コンポーネント ETL パイプライン チュートリアル'
sidebar_position: 10
---

import Preview from '@site/docs.ja/partials/\_Preview.md';

<Preview />

## 設定

### 1. プロジェクトの依存関係をインストールする

:::info Prerequisites

このチュートリアルを完了するには、[`uv` と `dg` をインストール](/guides/labs/components/index.md#installation)する必要があります。

:::

まず、ローカル データベース用の [`duckdb`](https://duckdb.org/docs/installation/?version=stable&environment=cli&platform=macos&download_method=package_manager) と、プロジェクト構造を視覚化するための [`tree`](https://oldmanprogrammer.net/source.php?dir=projects/tree/INSTALL) をインストールします:

<Tabs>

<TabItem value="mac" label="Mac">

<CliInvocationExample contents="brew install duckdb tree" />

</TabItem>

<TabItem value="windows" label="Windows">

[`duckdb`](https://duckdb.org/docs/installation/?version=stable&environment=cli&platform=win&download_method=package_manager) Windows インストール手順と [`tree`](https://oldmanprogrammer.net/source.php?dir=projects/tree/INSTALL) インストール手順を参照してください。

</TabItem>

<TabItem value="linux" label="Linux">

[`duckdb`](https://duckdb.org/docs/installation/?version=stable&environment=cli&platform=linux&download_method=direct&architecture=x86_64) および [`tree`](https://oldmanprogrammer.net/source.php?dir=projects/tree/INSTALL) Linux インストール手順を参照してください。

</TabItem>

</Tabs>

:::note

`tree` はオプションであり、コマンドラインでプロジェクト構造の適切にフォーマットされた表現を生成するためにのみ使用されます。`find`、`ls`、`dir`、またはその他のディレクトリ一覧コマンドを使用することもできます。

:::

### 2. 新しいプロジェクトを準備する

依存関係をインストールしたら、コンポーネント対応プロジェクトをスキャフォールディングします:

<CliInvocationExample path="docs_snippets/docs_snippets/guides/components/index/2-scaffold.txt" />

`dg scaffold project` コマンドは、`jaffle-platform` でプロジェクトをビルドし、その中に新しい Python 仮想環境を初期化します。`dg` のデフォルトの環境管理動作を使用すると、この仮想環境を自分でアクティブ化する必要はありません。

`dg scaffold project` でスキャフォールディングされたプロジェクトのファイル、ディレクトリ、およびデフォルト設定の詳細については、「[コンポーネントを使用したプロジェクトの作成](/guides/labs/components/building-pipelines-with-components/creating-a-project-with-components#project-structure)」を参照してください。

## データの取り込み

### 1. Slingコンポーネントタイプを環境に追加する

To ingest data, you must set up [Sling](https://slingdata.io/). However, if you list the available component types in your environment at this point, the Sling component won't appear, since the `dagster` package doesn't contain components for specific integrations (like Sling):

<CliInvocationExample path="docs_snippets/docs_snippets/guides/components/index/7-dg-list-component-types.txt" />

Sling コンポーネントを環境で使用できるようにするには、`dagster-slingents` の `sling` エクストラをインストールします。

<CliInvocationExample contents="uv add 'dagster-components[sling]'" />

:::note

`dg` は常に分離された環境で動作しますが、実行されるたびにプロジェクト ルートを解決しようとするため、プロジェクト環境で使用可能なコンポーネント タイプのセットにアクセスできます。`dg` が `tool.dg.is_project = true` 設定の `pyproject.toml` ファイルを見つけた場合、`uv` 管理の仮想環境が同じディレクトリに存在すると想定されます。(これは `uv.lock` ファイルの存在によって確認できます。)

`dg list component-type` のようなコマンドを実行すると、`dg` はスコープ内のプロジェクト環境を識別してクエリを実行し、結果を取得します。この場合、プロジェクト環境は `dg scaffold project` コマンドの一部として設定されています。

:::

### 2. Slingコンポーネントタイプの可用性を確認する

`dagster_sling.SlingReplicationCollectionComponent` コンポーネントタイプが使用可能になったことを確認するには、`dg list component-type` コマンドを再度実行します:

<CliInvocationExample path="docs_snippets/docs_snippets/guides/components/index/8-dg-list-component-types.txt" />

### 3. Slingコンポーネントの新しいインスタンスを作成する

次に、このコンポーネント タイプの新しいインスタンスを作成します:

<CliInvocationExample path="docs_snippets/docs_snippets/guides/components/index/9-dg-scaffold-sling-replication.txt" />

これにより、プロジェクトの `jaffle_platform/defs/ingest_files` にコンポーネント インスタンスが追加されます:

<CliInvocationExample path="docs_snippets/docs_snippets/guides/components/index/10-tree-jaffle-platform.txt" />

コンポーネント フォルダーに `component.yaml` という単一のファイルが作成されました。`component.yaml` ファイルはすべての Dagster コンポーネントに共通であり、実行時にコンポーネントから定義をスキャフォールディングするために使用されるコンポーネント タイプとパラメーターを指定します。

<CodeExample
  path="docs_snippets/docs_snippets/guides/components/index/11-component.yaml"
  language="YAML"
  title="jaffle-platform/jaffle_platform/defs/ingest_files/component.yaml"
/>

現在、パラメータは単一の「レプリケーション」を定義します。これは、ソースからターゲットにデータをレプリケートする方法を指定する Sling の概念です。詳細は、Sling によって読み取られる `replication.yaml` ファイルで指定されます。このファイルはまだ存在していませんが、すぐに作成する予定です。

:::note
レプリケーションの `path` パラメータは、component.yaml を含む同じフォルダを基準とします。これはコンポーネントの規則です。
:::

<!-- ### 4. DuckDBをセットアップする

DuckDB をセットアップしてテストします:

<CliInvocationExample path="docs_snippets/docs_snippets/guides/components/index/12-sling-setup-duckdb.txt" />

<CliInvocationExample path="docs_snippets/docs_snippets/guides/components/index/13-sling-test-duckdb.txt" /> -->

### 4. Sling ソースのファイルをダウンロード

次に、Sling はパブリック インターネットからの読み取りをサポートしていないため、Sling ソースを使用するにはいくつかのファイルをローカルにダウンロードする必要があります:

<CliInvocationExample path="docs_snippets/docs_snippets/guides/components/index/12-curl.txt" />

### 5. Set up the Sling to DuckDB replication

ダウンロードしたファイルを参照する `replication.yaml` ファイルを作成します。

<CodeExample
  path="docs_snippets/docs_snippets/guides/components/index/13-replication.yaml"
  language="YAML"
  title="jaffle-platform/jaffle_platform/defs/ingest_files/replication.yaml"
/>

Finally, modify the `component.yaml` file to tell the Sling component where replicated data with the `DUCKDB` target should be written:

<CodeExample
  path="docs_snippets/docs_snippets/guides/components/index/14-component-connections.yaml"
  language="YAML"
  title="jaffle-platform/jaffle_platform/defs/ingest_files/component.yaml"
/>

### 6. Dagster UI でアセットを表示およびマテリアライズする

Dagster UI にプロジェクトをロードして、これまでに構築した内容を確認します。アセットをマテリアライズして DuckDB インスタンスにテーブルをロードするには、**Materialize All** をクリックします。

<CliInvocationExample contents="dg dev" />

![](/images/guides/build/projects-and-components/components/sling.png)

コマンドラインで DuckDB テーブルを確認します:

<CliInvocationExample path="docs_snippets/docs_snippets/guides/components/index/15-duckdb-select.txt" />

## データの変換

データを変換するには、GitHub からサンプル dbt プロジェクトをダウンロードし、Sling で取り込んだデータを dbt プロジェクトの入力として使用する必要があります。

### 1. GitHub からサンプル dbt プロジェクトをクローンする

まず、プロジェクトをクローンし、埋め込まれた git リポジトリを削除します:

<CliInvocationExample path="docs_snippets/docs_snippets/guides/components/index/16-jaffle-clone.txt" />

### 2. dbtプロジェクトコンポーネントタイプをインストールする

dbt プロジェクトとインターフェースするには、Dagster dbt プロジェクト コンポーネントをインスタンス化する必要があります。dbt プロジェクト コンポーネント タイプにアクセスするには、`dagster-dbt` と `dbt-duckdb` をインストールします:

<CliInvocationExample contents="uv add 'dagster-components[dbt]' dbt-duckdb" />

`dagster_dbt.DbtProjectComponent` コンポーネント タイプが使用可能になったことを確認するには、`dg list component-type` を実行します:

<CliInvocationExample path="docs_snippets/docs_snippets/guides/components/index/17-dg-list-component-types.txt" />

### 3. dbt プロジェクト コンポーネントの新しいインスタンスをスキャフォールディングする

次に、`dagster_dbt.DbtProjectComponent` コンポーネントの新しいインスタンスをスキャフォールディングし、先ほどクローンした dbt プロジェクトへのパスを `project_path` スキャフォールディング パラメータとして指定します:

<CliInvocationExample path="docs_snippets/docs_snippets/guides/components/index/18-dg-scaffold-jdbt.txt" />

これにより、プロジェクトの `jaffle_platform/defs/jdbt` に新しいコンポーネント インスタンスが作成されます。コンポーネント構成を確認するには、そのディレクトリの `component.yaml` を開きます:

<CodeExample
  path="docs_snippets/docs_snippets/guides/components/index/19-component-jdbt.yaml"
  language="YAML"
  title="jaffle-platform/jaffle_platform/defs/jdbt/component.yaml"
/>

### 4. dbtプロジェクトコンポーネント構成を更新する

Dagster UI でプロジェクトを見てみましょう:

<CliInvocationExample contents="dg dev" />

![](/images/guides/build/projects-and-components/components/dbt-1.png)

`raw_customers`、`raw_orders`、および `raw_payments` テーブルのコピーが 2 つあることがわかります。アセットをクリックすると、アセットの完全なキーが表示されます。dbt コンポーネントによって生成されるキーは `main/*` の形式ですが、Sling コンポーネントによって生成されるキーは `target/main/*` の形式です。

Sling コンポーネントによって生成されたキーと一致するように、`dagster_dbt.DbtProjectComponent` コンポーネントの設定を更新する必要があります。以下の設定で `components/jdbt/component.yaml` を更新します:

<CodeExample
  path="docs_snippets/docs_snippets/guides/components/index/20-project-jdbt-incorrect.yaml"
  language="YAML"
  title="jaffle-platform/jaffle_platform/defs/jdbt/component.yaml"
/>

上記のファイルにはタイプミスがあることに気付いたかもしれません。コンポーネント ファイルを更新した後は、変更がコンポーネントのスキーマと一致しているかどうかを検証すると便利です。これは、`dg check yaml` を実行することで実行できます:

<CliInvocationExample path="docs_snippets/docs_snippets/guides/components/index/21-dg-component-check-error.txt" />

エラー メッセージには、ファイル名、行番号、およびエラーの正確な内容を示すコード スニペットが含まれていることがわかります。タイプミスを修正しましょう:

<CodeExample
  path="docs_snippets/docs_snippets/guides/components/index/22-project-jdbt.yaml"
  language="YAML"
  title="jaffle-platform/jaffle_platform/defs/jdbt/component.yaml"
/>

最後に、`dg check yaml` を再度実行して修正を検証します:

<CliInvocationExample path="docs_snippets/docs_snippets/guides/components/index/23-dg-component-check.txt" />

Dagster UI でプロジェクトを再ロードして、キーが正しくロードされていることを確認します:

![](/images/guides/build/projects-and-components/components/dbt-2.png)

これで、Sling と dbt プロジェクト コンポーネントによって生成されたキーが一致し、アセット グラフが正しくなりました。dbt プロジェクト コンポーネントを介して定義された新しいアセットをマテリアライズするには、**Materialize All** をクリックします。

修正を確認するには、コマンド ラインから DuckDB に新しくマテリアライズされたアセットのサンプルを表示します:

<CliInvocationExample path="docs_snippets/docs_snippets/guides/components/index/24-duckdb-select-orders.txt" />

## パイプラインを自動化する

いくつかの資産を定義したので、それらをスケジュールしましょう。

スケジュールの最初のスキャフォールド:

<CliInvocationExample path="docs_snippets/docs_snippets/guides/components/index/56-scaffold-daily-jaffle.txt" />

そして、`*` をターゲットにして `@daily` をスケジュールします:

<CodeExample
  path="docs_snippets/docs_snippets/guides/components/index/26-daily-jaffle.py"
  language="Python"
  title="jaffle-platform/jaffle_platform/defs/daily_jaffle.py"
/>

## Next steps

コンポーネントの使用を続けるには、[プロジェクトにコンポーネントを追加する](/guides/labs/components/building-pipelines-with-components/adding-components)か、[`dg` を使用して複数のコンポーネント対応プロジェクトを管理する](/guides/labs/dg/multiple-projects)方法を学習します。
