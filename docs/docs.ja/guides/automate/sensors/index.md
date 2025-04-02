---
title: 'センサー'
sidebar_position: 30
---

センサーを使用すると、Dagster 内部または外部システムで発生するイベントに応じてアクションを実行できます。センサーは定期的にイベントをチェックし、アクションを実行するか、アクションがスキップされた理由の説明を提供します。

イベントの例としては次のようなものがあります:

- Dagster で実行が完了
- Dagster で実行が失敗
- ジョブが特定のアセットを実体化 
- ファイルが S3 バケットに表示される
- 外部システムがダウンしている

アクションの例は次のとおりです:

- 実行を開始
- Slack メッセージの送信
- データベースに行を挿入

:::tip

センサーによるポーリングの代わりに、[Dagster API](/guides/operate/graphql/) を使用してイベントを Dagster にプッシュできます。

:::

<details>
  <summary>前提条件</summary>

このガイドの手順を実行するには、次のことが必要です:

- [アセット](/guides/build/assets/) に関する知識
- [ジョブ](/guides/build/jobs/) に関する知識

</details>

## 基本的なセンサー

センサーは `@sensor` デコレータで定義されます。次の例には、新しいファイルの検索をシミュレートする `check_for_new_files` 関数が含まれています。実際のシナリオでは、この関数は実際のシステムまたはディレクトリをチェックします。

センサーが新しいファイルを見つけると、`my_job` の実行を開始します。そうでない場合は、実行をスキップし、Dagster UI に `No new files found` と記録します。

<CodeExample path="docs_snippets/docs_snippets/guides/automation/simple-sensor-example.py" language="python" />

:::tip
センサーの `default_status` が `DefaultSensorStatus.RUNNING` でない限り、センサーは Dagster インスタンスに最初にデプロイされたときに有効になりません。センサーを見つけて有効にするには、Dagster UI で **Automation > Sensors** をクリックします。

To explicitly disable a sensor, you can use `DefaultSensorStatus.STOPPED`.
:::

## 評価間隔のカスタマイズ

`minimum_interval_seconds` 引数を使用すると、センサー評価間の​​最小経過秒数を指定できます。つまり、センサーは指定された間隔よりも頻繁に評価されることはありません。

この間隔は、センサーの実行間隔の最小間隔を表しており、センサーが実行される正確な頻度を表しているわけではないことに注意してください。センサーの完了に指定された間隔よりも長い時間がかかる場合、次の評価はそれに応じて遅延されます。

```python
# Sensor will be evaluated at least every 30 seconds
@dg.sensor(job=my_job, minimum_interval_seconds=30)
def new_file_sensor():
  ...
```

この例では、`new_file_sensor` の評価関数の実行に 1 秒もかからない場合、センサーは 30 秒ごとに一貫して実行されることが期待できます。ただし、評価関数に時間がかかる場合、評価の間隔は長くなります。

## 重複実行の防止

重複実行を防ぐために、実行キーを使用して各 `RunRequest` を一意に識別できます。[前の例](#basic-sensor) では、`RunRequest` は `run_key` を使用して構築されました:

```
yield dg.RunRequest(run_key=filename)
```

特定のセンサーに対して、一意の `run_key` を持つ `RunRequest` ごとに 1 つの実行が作成されます。Dagster は、以前に使用された実行キーを持つリクエストの処理をスキップし、重複する実行が作成されないようにします。

## カーソルと大量のイベント

多数のイベントを処理する場合、センサーのパフォーマンスを最適化するためにカーソルを実装する必要がある場合があります。実行キーとは異なり、カーソルを使用すると、状態を管理するカスタム ロジックを実装できます。

次の例は、カーソルを使用して、センサーが最後に実行されてから更新されたディレクトリ内のファイルに対してのみ `RunRequests` を作成する方法を示しています。

<CodeExample path="docs_snippets/docs_snippets/guides/automation/sensor-cursor.py" language="python" />

複数のイベント ストリームを使用するセンサーの場合、複数のストリームにわたるセンサーの進行状況を追跡するために、カーソル文字列の内外にあるより複雑なデータ構造をシリアル化および逆シリアル化する必要がある場合があります。

:::note
前の例では、`run_key` とカーソルの両方を使用しています。つまり、カーソルがリセットされてもファイルが変更されない場合は、新しい実行は開始されません。これは、ファイルに関連付けられた実行キーが変更されないためです。

センサーのカーソルをリセットできるようにしたい場合は、`RunRequest` に `run_key` を設定しないでください。
:::

## 次のステップ

これらの自動化方法を理解し、効果的に使用することで、特定のニーズと制約に対応する、より効率的なデータ パイプラインを構築できます。

- [スケジュール](/guides/automate/schedules)に従ってパイプラインを実行する
- [アセット センサー](/guides/automate/asset-sensors) を使用してジョブ間の依存関係をトリガーする
- センサーの代替として[宣言型オートメーション](/guides/automate/declarative-automation)を検討する
