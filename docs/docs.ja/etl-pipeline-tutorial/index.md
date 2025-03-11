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
- アセットの作成と実装
- 依存アセットの作成と実装
- アセットチェックでデータ品質を確保
- パーティション化したアセットの作成と実装
- パイプラインを自動化する
- センサーアセットの作成と実装
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

Run the following command to create the project directories and files for this tutorial:

   ```bash 
      dagster project from-example --example getting_started_etl_tutorial
   ```

Your project should have this structure:
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
Dagster has several example projects you can install depending on your use case. To see the full list, run `dagster project list-examples`. For more information on the `dagster project` command, see the [API documentation](https://docs-preview.dagster.io/api/cli#dagster-project).
::: 

### Dagster project structure

#### dagster-etl-tutorial root directory

In the `dagster-etl-tutorial` root directory, there are three configuration files that are common in Python package management. These files manage dependencies and identify the Dagster modules in the project.
| File | Purpose |
|------|---------|
| pyproject.toml | This file is used to specify build system requirements and package metadata for Python projects. It is part of the Python packaging ecosystem. |
| setup.cfg | This file is used for configuration of your Python package. It can include metadata about the package, dependencies, and other configuration options. |
| setup.py | This script is used to build and distribute your Python package. It is a standard file in Python projects for specifying package details. |

#### etl_tutorial directory

This is the main directory where you will define your assets, jobs, schedules, sensors, and resources.
| File | Purpose |
|------|---------|
| definitions.py | This file is typically used to define jobs, schedules, and sensors. It organizes the various components of your Dagster project. This allows Dagster to load the definitions in a module. |

#### data directory

The data directory contains the raw data files for the project. We will reference these files in our software-defined assets in the next step of the tutorial.

## Step 3: Launch the Dagster webserver

To make sure Dagster and its dependencies were installed correctly, navigate to the project root directory and start the Dagster webserver:"

   ```bash
   dagster dev
   ```

## Next steps

- Continue this tutorial by [creating and materializing assets](create-and-materialize-assets)
