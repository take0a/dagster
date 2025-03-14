---
title: "JavaScript でパイプラインを構築する"
sidebar_position: 20
---

このガイドでは、パイプを使用して Dagster で JavaScript を実行する方法について説明しますが、同じ原則が他の言語にも適用されます。

<details>
<summary>前提条件</summary>

このガイドに従うには、次のものが必要です:

- [アセット](/guides/build/assets/) に関する知識
- JavaScript と Node.js の基本的な理解

例を実行するには、以下をインストールする必要があります:

- [Node.js](https://nodejs.org/en/download/package-manager/)
- 次の Python パッケージ:

   ```bash
   pip install dagster dagster-webserver
   ```

- 次の Node パッケージ:

   ```bash
   npm install @tensorflow/tfjs
   ```
</details>

## Step 1: JavaScript で Tensorflow を使用してスクリプトを作成する

まず、CSV ファイルを読み取り、Tensorflow を使用して順次モデルをトレーニングする JavaScript スクリプトを作成します。

次の内容を含む `tensorflow/main.js` という名前のファイルを作成します:

<CodeExample path="docs_beta_snippets/docs_beta_snippets/guides/non-python/pipes-contrived-javascript.js" language="javascript" title="tensorflow/main.js" />

## Step 2: スクリプトを実行するDagsterアセットを作成する

Dagster で、次のアセットを作成します:

- `PipesSubprocessClient` リソースを使用して、`node` でスクリプトを実行します。
- `compute_kind` を `javascript` に設定します。これにより、代替コンピューティングが実体化に使用されることを簡単に識別できます。

<CodeExample path="docs_beta_snippets/docs_beta_snippets/guides/non-python/pipes-asset.py" language="python" />

アセットがマテリアライズされると、stdout と stderr が自動的にキャプチャされ、アセット ログに表示されます。Pipes に渡されたコマンドが正常終了コードを返すと、Dagster はアセットのマテリアライズ結果を生成します。

## Step 3: スクリプトからデータを送受信する

スクリプトにコンテキストを送信したり、イベントを Dagster に送り返したりするには、`PipesSubprocessClient` によって提供される環境変数を使用できます。

- `DAGSTER_PIPES_CONTEXT` - 入力コンテキスト
- `DAGSTER_PIPES_MESSAGES` - 出力コンテキスト

環境変数を読み取り、データをデコードし、メッセージを Dagster に書き戻す次のヘルパー関数を含む新しいファイルを作成します:

<CodeExample path="docs_beta_snippets/docs_beta_snippets/guides/non-python/pipes-javascript-utility.js" language="javascript" />

両方の環境変数は、base64 でエンコードされ、zip で圧縮された JSON オブジェクトです。各 JSON オブジェクトには、データの読み取りまたは書き込み先を示すパスが含まれています。

## Step 4: 外部プロセスからイベントを発行し、マテリアライズをレポートする

ユーティリティ関数を使用して Dagster Pipes 環境変数をデコードすると、JavaScript プロセスに追加のパラメータを送信できます。また、アセットのマテリアライゼーションに詳細情報を出力することもできます。

`tensorflow/main.js` スクリプトを次のように更新します:

- Dagsterコンテキストからモデル構成を取得し、
- モデル メタデータを使用してアセットの実体化を Dagster に報告する

<CodeExample path="docs_beta_snippets/docs_beta_snippets/guides/non-python/pipes-full-featured-javascript.js" language="javascript" />

:::tip

上記のメタデータ形式 (`{"raw_value": value, "type": type}`) は、豊富な Dagster メタデータを指定するための Dagster Pipes の特別な構文の一部です。サポートされているすべてのメタデータ タイプとその形式の完全なリファレンスについては、[Dagster Pipes メタデータ リファレンス](using-dagster-pipes/reference#passing-rich-metadata-to-dagster) を参照してください。

:::

## Step 5: アセットを更新して追加のパラメータを提供する

最後に、スクリプトで使用されるモデル情報を渡せるように Dagster アセットを更新します:

<CodeExample path="docs_beta_snippets/docs_beta_snippets/guides/non-python/pipes-asset-with-context.py" language="python" />

## 次は？

- [パイプラインの自動化](/guides/automate/index.md)を使用して、パイプラインを定期的に実行するようにスケジュールします。
- [アセット チェックの理解](/guides/test/asset-checks) で、スクリプトを検証するためのアセット チェックの追加について学習します。
