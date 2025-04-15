---
title: '定義によるコードの場所の管理'
description: "A code location is a collection of Dagster definitions loadable and accessible by Dagster's tools. Learn to create, load, and deploy code locations."
sidebar_position: 100
---

コードロケーションとは、CLI、UI、Dagster+などのDagsterツールから読み込み、アクセスできるDagster定義の集合です。コードロケーションは以下の要素で構成されます:

- トップレベル変数に <PyObject section="definitions" module="dagster" object="Definitions" /> のインスタンスを持つ Python モジュールへの参照
- そのモジュールを正常にロードできる Python 環境

コード内の定義は共通の名前空間を持ち、一意の名前を持つ必要があります。これにより、ツール内でコード内の定義をグループ化して整理できます。

![Code locations](/images/guides/deploy/code-locations/code-locations-diagram.png)

単一のデプロイメントには、1つまたは複数のコードロケーションを含めることができます。

コードロケーションは別のプロセスにロードされ、RPCメカニズムを介してDagsterシステムプロセスと通信します。このアーキテクチャには、いくつかの利点があります。

- ユーザーコードが更新された場合、Dagsterウェブサーバー/UIは再起動せずに変更を取得できます。
- 複数のコードロケーションを使用してジョブを整理しながら、ウェブサーバー/UIの単一のインスタンスを使用してすべてのコードロケーションを操作できます。
- Dagsterウェブサーバープロセスは、ユーザーコードとは別のPython環境で実行できるため、ジョブの依存関係をウェブサーバー環境にインストールする必要はありません。
- 各コードロケーションは別々のPython環境から取得できるため、チームは依存関係（さらにはPythonのバージョン）を個別に管理できます。

## 関連 API

| Name                                                                     | Description                                                                                                                                       |
| ------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------- |
| <PyObject section="definitions" module="dagster" object="Definitions" /> | コードロケーション内で定義されたすべての定義を含むオブジェクト。定義には、アセット、ジョブ、リソース、スケジュール、センサーが含まれます。 |

## コードの場所の定義

コードの場所を定義するには、Pythonモジュール内に <PyObject section="definitions" module="dagster" object="Definitions" /> オブジェクトを含むトップレベル変数を作成します。例:

```python
# definitions.py

defs = Definitions(
    assets=[dbt_customers_asset, dbt_orders_asset],
    schedules=[bi_weekly_schedule],
    sensors=[new_data_sensor],
    resources=[dbt_resource],
)
```

定義は、`definitions.py` という名前の Python モジュールに含めることをお勧めします。

## コードの場所のデプロイと読み込み

- [Local development](#local-development)
- [Dagster+ deployment](#dagster-deployment)
- [Open source deployment](#open-source-deployment)

### ローカル開発 {#local-development}

<Tabs>
<TabItem value="From a file" label="ファイルから">

Dagsterは、コードの場所としてファイルを直接読み込むことができます。
次の例では、`-f`引数を使用してファイル名を指定しています:

```shell
dagster dev -f my_file.py
```

このコマンドは、`my_file.py` 内の定義を現在の Python 環境のコード位置として読み込みます。

複数のファイルを一度に含めることもできます。その場合、各ファイルがコード位置として読み込まれます:

```shell
dagster dev -f my_file.py -f my_second_file.py
```

</TabItem>
<TabItem value="From a module" label="モジュールから">

DagsterはPythonモジュールを[コードの場所](/guides/deploy/code-locations/)としてロードすることもできます。この方法を使用すると、Dagsterはコマンドラインに渡されたモジュール内で定義された定義をロードします。

Python モジュール内の `definitions` というサブモジュール内に、<PyObject section="definitions" module="dagster" object="Definitions" /> オブジェクトを含む変数を定義することをお勧めします。
実際には、Python モジュールのルートレベルに `definitions.py` というファイルを追加することでサブモジュールを作成できます。

この開発スタイルは Python のインポートエラーを一挙に排除するため、本番環境にデプロイされる Dagster プロジェクトには強くお勧めします。

次の例では、`-m` 引数を使用してモジュール名と定義の場所を指定しています:

```shell
dagster dev -m your_module_name.definitions
```

このコマンドは、現在の Python 環境の `definitions` サブモジュール内の <PyObject section="definitions" module="dagster" object="Definitions" /> オブジェクトを含む変数内の定義を読み込みます。

複数のモジュールを一度に含めることもできます。その場合、各モジュールはコードの場所として読み込まれます:

```shell
dagster dev -m your_module_name.definitions -m your_second_module.definitions
```

</TabItem>
<TabItem value="Without command line arguments" label="コマンドライン引数なし">

コマンドライン引数を指定せずに定義をロードするには、`pyproject.toml` ファイルを使用できます。
すべての Dagster サンプルプロジェクトに含まれるこのファイルには、`module_name` 変数を含む `tool.dagster` セクションが含まれています。

```toml
[tool.dagster]
module_name = "your_module_name.definitions"  ## name of project's Python module and where to find the definitions
code_location_name = "your_code_location_name"  ## optional, name of code location to display in the Dagster UI
```

定義されると、`p​​yproject.toml` ファイルと同じディレクトリでこれを実行できます:

```shell
dagster dev
```

代わりに次のようになります:

```shell
dagster dev -m your_module_name.definitions
```

`pyproject.toml` ファイルを使用して、一度に複数のモジュールを含めることもできます。各モジュールはコードの場所としてロードされます:

```toml
[tool.dagster]
modules = [{ type = "module", name = "foo" }, { type = "module", name = "bar" }]
```

</TabItem>
</Tabs>

ローカルインスタンスの構成方法など、ローカル開発の詳細については、「[Dagster をローカルで実行する](/guides/deploy/deployment-options/running-dagster-locally)」を参照してください。

### Dagster+ のデプロイメント

[Dagster+ のコードロケーションに関するドキュメント](/dagster-plus/deployment/code-locations/)をご覧ください。

### オープンソースのデプロイメント

`workspace.yaml` ファイルは、オープンソース (OSS) のデプロイメントでコードの場所を読み込むために使用されます。
このファイルは、コードの場所のコレクションを読み込む方法を指定し、通常は高度なユースケースで使用されます。
詳細については、「[workspace.yaml リファレンス](/guides/deploy/code-locations/workspace-yaml)」をご覧ください。

## トラブルシューティング

| Error                                                                | Description and resolution                                                                                                                                                                                                                                    |
| -------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Cannot have more than one Definitions object defined at module scope | Dagsterは、単一のPythonモジュール内に複数の<PyObject section="definitions" module="dagster" object="Definitions" />オブジェクトを検出しました。単一のコード位置に存在できる<PyObject section="definitions" module="dagster" object="Definitions" />オブジェクトは1つだけです。 |
