---
title: Build an ETL Pipeline
description: Learn how to build an ETL pipeline with Dagster
last_update:
   author: Alex Noonan
sidebar_class_name: hidden
---

# 最初のETLパイプラインを構築する

このチュートリアルでは、Dagsterを使用してETLパイプラインを構築します:

- 販売データをDuckDBにインポートする
- データをレポートに変換する
- スケジュールされたレポートを自動的に実行する
- オンデマンドで1回限りのレポートを生成

## 以下の内容を学びます:

- 推奨プロジェクト構造でDagsterプロジェクトを設定する
- アセットの作成と実体化
- 依存アセットの作成と実体化
- アセットチェックでデータ品質を確保
- パーティション化したアセットの作成と実体化
- パイプラインを自動化する
- センサーアセットの作成と実体化
- プロジェクトが複雑になったらリファクタリングする

<details>
  <summary>前提条件</summary>

このガイドの手順を実行するには:

- Pythonの基礎知識
- システムに Python 3.9 以降がインストールされていること。詳細については、[インストール ガイド](/getting-started/installation)を参照してください。
- SQL および Pandas などの Python データ操作ライブラリに精通していること。
- データ パイプラインと抽出、変換、ロード プロセスに関する理解。
</details>


## Step 1: Dagster環境を設定する

まず、新しい Dagster プロジェクトを設定します。

1. ターミナルを開き、プロジェクト用の新しいディレクトリを作成します:

   ```bash
   mkdir dagster-etl-tutorial
   cd dagster-etl-tutorial
   ```

2. 仮想環境を作成してアクティブ化します:

   <Tabs>
   <TabItem value="macos" label="MacOS">
   ```bash
   python -m venv dagster_tutorial
   source dagster_tutorial/bin/activate
   ```
   </TabItem>
   <TabItem value="windows" label="Windows">
   ```bash
   python -m venv dagster_tutorial
   dagster_tutorial\Scripts\activate
   ```
   </TabItem>
   </Tabs>

3. Dagster と必要な依存関係をインストールします:

   ```bash
   pip install dagster dagster-webserver pandas dagster-duckdb
   ```

## Step 2: Dagsterプロジェクト構造を作成する

次のコマンドを実行して、このチュートリアルのプロジェクトディレクトリとファイルを作成します:

   ```bash 
      dagster project from-example --example getting_started_etl_tutorial
   ```

プロジェクトはこのような構造にする必要があります:

{/* vale off */}
```
dagster-etl-tutorial/
├── data/
│   └── products.csv
│   └── sales_data.csv
│   └── sales_reps.csv
│   └── sample_request/
│       └── request.json
├── etl_tutorial/
│   └── definitions.py
├── pyproject.toml
├── setup.cfg
├── setup.py
```
{/* vale on */}

:::info
Dagster には、ユースケースに応じてインストールできるサンプル プロジェクトがいくつかあります。完全なリストを表示するには、`dagster project list-examples` を実行します。`dagster project` コマンドの詳細については、[API ドキュメント](https://docs-preview.dagster.io/api/cli#dagster-project) を参照してください。
::: 

### Dagster プロジェクトの構造

#### dagster-etl-tutorial ルートディレクトリ

`dagster-etl-tutorial` ルート ディレクトリには、Python パッケージ管理でよく使用される 3 つの構成ファイルがあります。これらのファイルは依存関係を管理し、プロジェクト内の Dagster モジュールを識別します。

| File | Purpose |
|------|---------|
| pyproject.toml | このファイルは、Python プロジェクトのビルド システム要件とパッケージ メタデータを指定するために使用されます。これは、Python パッケージ エコシステムの一部です。 |
| setup.cfg | このファイルは、Python パッケージの構成に使用されます。パッケージ、依存関係、その他の構成オプションに関するメタデータを含めることができます。 |
| setup.py | このスクリプトは、Python パッケージをビルドして配布するために使用されます。これは、パッケージの詳細を指定するための Python プロジェクトの標準ファイルです。 |

#### etl_tutorial ディレクトリ

これは、アセット、ジョブ、スケジュール、センサー、リソースを定義するメインのディレクトリです。

| File | Purpose |
|------|---------|
| definitions.py | このファイルは通常、ジョブ、スケジュール、センサーを定義するために使用されます。このファイルは、Dagster プロジェクトのさまざまなコンポーネントを整理します。これにより、Dagster はモジュール内の定義を読み込むことができます。 |

#### data ディレクトリ

データ ディレクトリには、プロジェクトの生データ ファイルが含まれています。チュートリアルの次の手順で、ソフトウェア定義アセット内のこれらのファイルを参照します。

## Step 3: Dagster ウェブサーバーを起動する

Dagsterとその依存関係が正しくインストールされていることを確認するには、プロジェクトのルートディレクトリに移動し、Dagsterウェブサーバーを起動します:

   ```bash
   dagster dev
   ```

## 次は

- このチュートリアルを続けるには、[アセットの作成と実体化](create-and-materialize-assets)へ進んでください。
