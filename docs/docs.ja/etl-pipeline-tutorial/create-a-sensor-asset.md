---
title: センサーアセットを作成する
description: Use sensors to create event driven pipelines
last_update:
  author: Alex Noonan
sidebar_position: 60
---

[センサー](/guides/automate/sensors) を使用すると、外部のイベントや条件に基づいてワークフローを自動化できるため、特にジョブが不規則な頻度で、または連続して発生する状況でのイベント駆動型の自動化に役立ちます。

次の状況ではセンサーの使用を検討してください。

- **イベント駆動型ワークフロー**: ワークフローが、新しいデータファイルの到着や API 応答の変更などの外部イベントに依存する場合。
- **条件付き実行**: 特定の条件が満たされた場合にのみジョブを実行し、不要な計算を削減します。
- **リアルタイム処理**: スケジュールされた時間を待つのではなく、データが利用可能になったらすぐに処理する必要がある場合。

このステップでは次の操作を行います:

- イベント駆動型ワークフローに基づいて実行されるアセットを作成する
- アセットを実体化するための条件を監視するセンサーを作成する

## 1. イベント駆動型アセットを作成する

私たちのパイプラインでは、幹部が部門別および製品別の売上結果のピボット テーブル レポートを希望する状況をモデル化します。幹部は、要求に応じてリアルタイムで処理されることを望んでいます。

このアセットについては、実体化のコンテキストで期待されるリクエストの構造を定義する必要があります。

それ以外は、このアセットの定義は以前のアセットと同じです。次のコードを `product_performance` の下にコピーします。

<CodeExample
  path="docs_snippets/docs_snippets/guides/tutorials/etl_tutorial/etl_tutorial/definitions.py"
  language="python"
  lineStart="275"
  lineEnd="312"
/>

## 2. センサーを構築する

Dagster でセンサーを定義するには、`@sensor` デコレータを使用します。このデコレータは、ジョブをトリガーするための条件が満たされているかどうかを評価する関数に適用されます。

センサーには次の要素が含まれます:

- **Job**: 条件が満たされたときにセンサーがトリガーするジョブ。
- **RunRequest**: ジョブ実行の構成を指定するオブジェクト。これには、冪等性を保証する `run_key` と、ジョブ固有の設定用の `run_config` が含まれます。

<CodeExample
  path="docs_snippets/docs_snippets/guides/tutorials/etl_tutorial/etl_tutorial/definitions.py"
  language="python"
  lineStart="314"
  lineEnd="355"
/>

## 3. センサーアセットの実体化

1. 定義オブジェクトを次のように更新します:

<CodeExample
  path="docs_snippets/docs_snippets/guides/tutorials/etl_tutorial/etl_tutorial/definitions.py"
  language="python"
  lineStart="357"
  lineEnd="373"
/>

2. 定義を再読み込みします。

3. Automation ページへ遷移します。

4. `adhoc_request_sensor` をオンにします。

5. `adhoc_request_sensor` の詳細をクリックします。

   ![2048 resolution](/images/tutorial/etl-tutorial/sensor-evaluation.png)

6. `data/sample_request` フォルダから `request.json` を `data/requests` フォルダに追加します。

7. このリクエストの実行を確認するには、緑色のチェックマークをクリックします。

   ![2048 resolution](/images/tutorial/etl-tutorial/sensor-asset-run.png)


## 次は

これでプロジェクトが完成したので、次のステップでは、必要に応じて追加できるように、プロジェクトをより管理しやすい構造にリファクタリングします。

[プロジェクトをリファクタリング](/etl-pipeline-tutorial/refactor-your-project)してチュートリアルを終了します。
