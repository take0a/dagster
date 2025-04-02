---
title: "Dagster プロジェクトファイルのリファレンス"
description: "A reference of the files in a Dagster project."
sidebar_position: 300
---

このリファレンスには、Dagster プロジェクトの [デフォルト](#デフォルトファイル)ファイルと[設定](#設定ファイル) ファイルに関する詳細が記載されています。

- ファイル名
- ファイルの機能
- プロジェクトディレクトリ内のファイルの配置場所
- Dagster プロジェクトを構築するためのリソース

## デフォルトファイル

以下は、`dagster project scaffold` コマンドによって生成されたデフォルトのプロジェクト スケルトンを使用した Dagster プロジェクトを示しています:

```shell
.
├── README.md
├── my_dagster_project
│   ├── __init__.py
│   ├──  assets.py
│   └──  definitions.py
├── my_dagster_project_tests
├── pyproject.toml
├── setup.cfg
├── setup.py
└── tox.ini
```

:::note

この特定の例では、スキャフォールディングによって作成されたプロジェクトを使用していますが、[公式の例を使用して](creating-a-new-project#using-an-official-example)作成されたプロジェクトにもこれらのファイルが含まれます。公式の例では、`assets.py` は Python ファイルではなくサブディレクトリになります。

:::

これらの各ファイルとディレクトリの機能を見てみましょう:

### my_dagster_project/ ディレクトリ

これは Dagster コードを含む Python モジュールです。このディレクトリには次のものも含まれています:

| File/Directory | Description |
|----------------|-------------|
| `__init__.py`  | Python パッケージに必要なファイル。詳細については、[Python ドキュメント](https://docs.python.org/3/reference/import.html#regular-packages) を参照してください。|
| `assets.py` | アセット定義を含む Python モジュール。**注:** プロジェクトが大きくなるにつれて、アセットをサブモジュールに整理することをお勧めします。たとえば、アナリティクス関連のアセットをすべて `my_dagster_project/assets/analytics/` ディレクトリに配置し、トップレベルの定義で <PyObject section="assets" module="dagster" object="load_assets_from_package_module" /> を使用してアセットをロードすると、アセットを定義するたびにトップレベルの定義にアセットを手動で追加する必要がなくなります。<br /><br /> 同様に、<PyObject section="assets" module="dagster" object="load_assets_from_modules" /> を使用して、単一の Python ファイルからアセットをロードすることもできます。 |
| `definitions.py` | `definitions.py` ファイルには、プロジェクト内で定義されているすべての定義を含む <PyObject section="definitions" module="dagster" object="Definitions" /> オブジェクトが含まれています。定義は、アセット、ジョブ、スケジュール、センサー、またはリソースのいずれかです。これにより、Dagster はモジュール内の定義を読み込むことができます。<br /><br />Dagster コードをデプロイおよび読み込むその他の方法については、[コードの場所に関するドキュメント](/guides/deploy/code-locations/) を参照してください。 |

### my_dagster_project_tests/ ディレクトリ

`my_dagster_project` のテストを含む Python モジュール。

### README.md ファイル

Dagster プロジェクトの説明とスターターガイド。

### pyproject.toml ファイル

ツールに依存しない静的な方法でパッケージのコアのメタデータを指定するファイル。

このファイルには、トップレベルで定義され検出可能な Dagster 定義を含む Python モジュールを参照する `tool.dagster` セクションが含まれています。これにより、`dagster dev` コマンドを使用して、パラメーターなしで Dagster コードをロードできます。詳細については、[コードの場所のドキュメント](/guides/deploy/code-locations/)を参照してください。

**注:** `pyproject.toml` は [PEP-518](https://peps.python.org/pep-0518/) で導入され、`setup.py` を置き換えることを目的としていますが、この仕様を使用しないツールとの互換性のために `setup.py` がまだ含まれる可能性があります。

### setup.py ファイル

新しいプロジェクトのパッケージとして Python パッケージ依存関係を含むビルド スクリプト。このファイルを使用して依存関係を指定します。

注: Dagster+ を使用する場合は、依存関係として `dagster-cloud` を追加してください。

### setup.cfg

`setup.py` コマンドのオプションのデフォルトを含む ini ファイル。

## 設定ファイル

個別のユースケースや Dagster+ を使用している場合は、プロジェクトに追加の構成ファイルを追加する必要がある場合もあります。これらのファイルがプロジェクトにどのように適合するかについては、[サンプルプロジェクト構造のセクション](#サンプルプロジェクト構造)を参照してください。

| File/Directory | Description | OSS | Dagster+ |
|----------------|-------------|-----|----------|
| dagster.yaml   | ストレージの場所、実行ランチャー、センサー、スケジュールの定義など、Dagster インスタンスを構成します。詳細については、ユースケースと使用可能なオプションのリストを含む [`dagster.yaml`](/guides/deploy/dagster-yaml) リファレンスを参照してください。<br /><br />[Dagster+ ハイブリッド デプロイメント](/dagster-plus/deployment/deployment-types/hybrid/) の場合、このファイルを使用して [ハイブリッド エージェント](/dagster-plus/deployment/management/settings/customizing-agent-settings) をカスタマイズできます。 | Optional | Optional |
| dagster_cloud.yaml | Dagster+ のコード ロケーションを定義します。詳細については、[`dagster_cloud.yaml` リファレンス](/dagster-plus/deployment/code-locations/dagster-cloud-yaml) を参照してください。 | Not applicable | Recommended |
| deployment_settings.yaml | 実行キューの優先度や同時実行の制限など、Dagster+ での完全なデプロイメントの設定を構成します。詳細については、デプロイメント設定リファレンスを参照してください。<br /><br />**注:** このファイルには任意の名前を付けることができますが、わかりやすい名前を選択することをお勧めします。 | Not applicable | Optional |
| workspace.yaml | ローカル開発またはインフラストラクチャへのデプロイ用に複数のコードの場所を定義します。詳細と使用可能なオプションについては、[`workspace.yaml` ファイル リファレンス](/guides/deploy/code-locations/workspace-yaml) を参照してください。 | Optional | Not applicable |


## サンプルプロジェクト構造

デフォルトのプロジェクト スケルトンを使用して、いくつかのサンプル Dagster プロジェクトがさまざまなシナリオでどのように構造化されるかを見てみましょう。

:::note 設定ファイルの場所

[`dagster_cloud.yaml`](/dagster-plus/deployment/code-locations/dagster-cloud-yaml) を除き、構成ファイルをプロジェクトファイルと同じ場所に配置する必要はありません。これらのファイルは通常、`DAGSTER_HOME` に配置する必要があります。たとえば、大規模なデプロイメントでは、`DAGSTER_HOME` と Dagster インフラストラクチャ構成は、サポートされるコードの場所とは別に管理できます。

:::

### ローカル開発

<Tabs>
<TabItem value="単一のコードの場所">

ローカル開発の場合、コードの場所が 1 つだけのプロジェクトは次のようになります:

```shell
.
├── README.md
├── my_dagster_project
│   ├── __init__.py
│   ├──  assets.py
│   └──  definitions.py
├── my_dagster_project_tests
├── dagster.yaml      ## optional, used for instance settings
├── pyproject.toml    ## optional, used to define the project as a module
├── setup.cfg
├── setup.py
└── tox.ini
```

</TabItem>
<TabItem value="複数のコードの場所">

ローカル開発の場合、複数のコードの場所を持つプロジェクトは次のようになります:

```shell
.
├── README.md
├── my_dagster_project
│   ├── __init__.py
│   ├──  assets.py
│   └──  definitions.py
├── my_dagster_project_tests
├── dagster.yaml      ## optional, used for instance settings
├── pyproject.toml
├── setup.cfg
├── setup.py
├── tox.ini
└── workspace.yaml    ## defines multiple code locations
```

</TabItem>
</Tabs>

### Dagster オープンソースのデプロイ

ローカルでの作業からインフラストラクチャへの Dagster のデプロイに移行する準備ができたら、[デプロイ ガイド](/guides/deploy/deployment-options/) を使用して起動して実行してください。

インフラストラクチャにデプロイされた Dagster プロジェクトは次のようになります。

```shell
.
├── README.md
├── my_dagster_project
│   ├── __init__.py
│   ├──  assets.py
│   └──  definitions.py
├── my_dagster_project_tests
├── dagster.yaml      ## optional, used for instance settings
├── pyproject.toml
├── setup.cfg
├── setup.py
├── tox.ini
└── workspace.yaml    ## defines multiple code locations
```

### Dagster+

Dagster+ で使用しているデプロイメントのタイプ (サーバーレスまたはハイブリッド) に応じて、プロジェクト構造が若干異なる場合があります。詳細については、タブをクリックしてください。

<Tabs>
<TabItem value="Serverless deployment">

#### サーバーレスのデプロイ

Dagster+ でサーバーレスのデプロイの場合、プロジェクトは次のようになります:

```shell
.
├── README.md
├── my_dagster_project
│   ├── __init__.py
│   ├──  assets.py
│   └──  definitions.py
├── my_dagster_project_tests
├── dagster_cloud.yaml         ## defines code locations
├── deployment_settings.yaml   ## optional, defines settings for full deployments
├── pyproject.toml
├── setup.cfg
├── setup.py
└── tox.ini
```

</TabItem>
<TabItem value="Hybrid deployment">

#### ハイブリッドのデプロイ

Dagster+ でハイブリッドのデプロイの場合、プロジェクトは次のようになります:

```shell
.
├── README.md
├── my_dagster_project
│   ├── __init__.py
│   ├──  assets.py
│   └──  definitions.py
├── my_dagster_project_tests
├── dagster.yaml                ## optional, hybrid agent custom configuration
├── dagster_cloud.yaml          ## defines code locations
├── deployment_settings.yaml    ## optional, defines settings for full deployments
├── pyproject.toml
├── setup.cfg
├── setup.py
└── tox.ini
```

</TabItem>
</Tabs>

## 次は

Dagster プロジェクトのデフォルトファイルとその配置場所については学習しましたが、Dagster コードを含むファイルについてはどうでしょうか。

プロジェクトを持続的に拡張するには、[プロジェクトの構築ガイド](/guides/build/projects/structuring-your-dagster-project)のベスト プラクティスと推奨事項を確認してください。
