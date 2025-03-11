---
title: 最初のDagsterプロジェクトを構築する
description: Learn how to quickly get up and running with Dagster
sidebar_position: 30
sidebar_label: "Quickstart"
---

Dagster へようこそ! このガイドでは、Dagster を使用して、次の基本的なパイプラインを作成します:

- CSVファイルからデータを抽出します
- データを変換する
- 変換されたデータを新しいCSVファイルに読み込みます

## 学ぶ内容

- 基本的なDagsterプロジェクトの設定方法
- 抽出、変換、ロード (ETL) プロセスの各ステップで Dagster アセットを作成する方法
- Dagster の UI を使用してパイプラインを監視および実行する方法

## 前提条件

<details>
  <summary>前提条件</summary>

このガイドの手順を実行するには、次のことが必要です:

- Pythonの基礎知識
- システムに Python 3.9 以降がインストールされていること。詳細については、[インストール ガイド](/getting-started/installation)を参照してください。
</details>

## Step 1: Dagster環境を設定する

1. ターミナルを開き、プロジェクト用の新しいディレクトリを作成します:

   ```bash
   mkdir dagster-quickstart
   cd dagster-quickstart
   ```

2. 仮想環境を作成してアクティブ化します:

   <Tabs>
   <TabItem value="macos" label="MacOS">
   ```bash
   python -m venv venv
   source venv/bin/activate
   ```
   </TabItem>
   <TabItem value="windows" label="Windows">
   ```bash
   python -m venv venv
   source venv\Scripts\activate
   ```
   </TabItem>
   </Tabs>

3. Dagster と必要な依存関係をインストールします:

   ```bash
   pip install dagster dagster-webserver pandas
   ```

## Step 2: Dagsterプロジェクト構造を作成する

:::info
このガイドのプロジェクト構造は、すぐに開始できるように簡略化されています。新しいプロジェクトを作成するときは、`dagster project scaffold` を使用して完全な Dagster プロジェクトを生成します。
:::

次に、以下のような基本的な Dagster プロジェクトを作成します。

```
dagster-quickstart/
├── quickstart/
│   ├── __init__.py
│   └── assets.py
├── data/
    └── sample_data.csv
```

1. 上記のファイルとディレクトリを作成するには、次のコマンドを実行します:

   ```bash
   mkdir quickstart data
   touch quickstart/__init__.py quickstart/assets.py
   touch data/sample_data.csv
   ```

2. `data/sample_data.csv` ファイルに次の内容を追加します:

   ```csv
   id,name,age,city
   1,Alice,28,New York
   2,Bob,35,San Francisco
   3,Charlie,42,Chicago
   4,Diana,31,Los Angeles
   ```

   この CSV は、Dagster パイプラインのデータ ソースとして機能します。

## Step 3: アセットを定義する

次に、ETL パイプラインのアセットを作成します。`quickstart/assets.py` を開き、次のコードを追加します:

<CodeExample path="docs_beta_snippets/docs_beta_snippets/getting-started/quickstart.py" language="python" />

タスクベースのオーケストレーションに慣れている場合、これは異常に思えるかもしれません。その場合、抽出、変換、読み込みの 3 つの個別のステップが必要になります。

ただし、Dagster では、タスクではなくアセットを基本的な構成要素として使用してパイプラインをモデル化します。

## Step 4: パイプラインを実行する

1. ターミナルで、プロジェクトのルート ディレクトリに移動し、次のコマンドを実行します:

   ```bash
   dagster dev -f quickstart/assets.py
   ```

2. Web ブラウザを開いて `http://localhost:3000` に移動すると、Dagster UI が表示されます。

   ![2048 resolution](/images/getting-started/quickstart/dagster-ui-start.png)

3. 上部のナビゲーションで、 **Assets > View global asset lineage** をクリックします。

4. **Materialize** をクリックしてパイプラインを実行します。

5. 表示されるポップアップで、**View** をクリックします。これにより、**Run details** ページが開き、実行中に実行内容を表示できるようになります。

   ![2048 resolution](/images/getting-started/quickstart/run-details.png)

   ページの左上隅近くにある **view buttons** を使用して、実行の表示方法を変更します。アセットをクリックして、ログとメタデータを表示することもできます。

## Step 5: 結果を確認する

ターミナルで以下を実行します:

```bash
cat data/processed_data.csv
```

新しい `age_group` 列を含む変換されたデータが表示されます:

```bash
id,name,age,city,age_group
1,Alice,28,New York,Young
2,Bob,35,San Francisco,Middle
3,Charlie,42,Chicago,Senior
4,Diana,31,Los Angeles,Middle
```

## 次のステップ

おめでとうございます。これで、Dagster で最初のパイプラインを構築して実行できました。次に、以下の操作を実行できます。

- より複雑なETLパイプラインの構築方法については、[ETLパイプラインチュートリアル](/etl-pipeline-tutorial/)を続けて学習してください。
- [アセットで考える](/guides/build/assets/)方法を学ぶ
