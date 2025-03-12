---
title: 外部アセット
sidebar_position: 500
---

Dagster の目標の 1 つは、たとえそれらの資産が Dagster 以外のシステムによってオーケストレーションされている場合でも、組織内のすべてのデータ資産の単一の統一された系統を提示することです。

**外部アセット** を使用すると、他のシステムによってオーケストレーションされたアセットを Dagster 内でネイティブにモデル化できるため、組織のデータの包括的なカタログを確保できます。また、これらの外部アセットの下流に新しいデータ アセットを作成することもできます。

ネイティブ アセットとは異なり、Dagster は外部アセットを直接実現したり、スケジュールに組み込んだりすることはできません。このような場合、外部アセットが更新されたときに外部システムが Dagster に通知する必要があります。

たとえば、外部アセットには次のようなものがあります。

- 特注の内部ツールによって入力されたデータレイク内のファイル
- パートナーからSFTPで毎日配信されるCSVファイル
- 別のオーケストレーターによって入力されたデータ ウェアハウス内のテーブル

:::note

この記事では、[アセット](/guides/build/assets/defining-assets) と [センサー](/guides/automate/sensors) に精通していることを前提としています。

:::

## 外部アセットの定義

ほぼ毎日、SFTP で生のトランザクションデータを送信するパートナーがいるとします。このデータは後でクリーンアップされ、内部データレイクに保存されます。

生のトランザクション データは Dagster によって具体化されないため、外部アセットとしてモデル化するのが合理的です。次の例では、`AssetSpec` を使用してこれを実現します:

<CodeExample path="docs_beta_snippets/docs_beta_snippets/guides/data-modeling/external-assets/creating-external-assets.py" language="python" />

外部アセットに提供できるパラメータについては、<PyObject section="assets" module="dagster" object="AssetSpec" /> を参照してください。

## マテリアライゼーションとメタデータの記録

外部アセットが Dagster でモデル化されている場合は、外部アセットが更新されるたびに Dagster に通知する必要があります。また、最終更新時刻など、アセットに関する関連メタデータも含める必要があります。

これを行うには主に 2 つの方法があります:

- センサーによる外部アセットのイベントの取得
- Dagster の REST API を使用して外部アセット イベントをプッシュする

### センサーによるプル

Dagster [センサー](/guides/automate/sensors) を使用すると、外部システムを定期的にポーリングし、外部アセットに関する情報を Dagster に取り込むことができます。

たとえば、ファイルが変更されるたびに SFTP サーバーなどの外部システムをポーリングして外部アセットを更新する方法は次のとおりです。

<CodeExample path="docs_beta_snippets/docs_beta_snippets/guides/data-modeling/external-assets/pulling-with-sensors.py" language="python" />

センサーの詳細については、[センサーガイド](/guides/automate/sensors)を参照してください。

### REST API によるプッシュ

外部システムから REST API にイベントをプッシュすることで、外部アセットが実現されたことを Dagster に通知できます。次の例は、`raw_transactions` 外部アセットの実現が発生したことを Dagster に通知する方法を示しています。

REST API に必要なヘッダーは、Dagster+ を使用しているか OSS を使用しているかによって異なります。タブを使用して、各 Dagster タイプの API リクエストの例を表示します。

<Tabs>
<TabItem value="dagster-plus" label="Dagster+">

Dagster+ を使用する場合は、認証ヘッダーが必要です。リクエストは、Dagster+ 組織と組織内の特定のデプロイに対して行う必要があります。

```shell
curl \
  -X POST \
  -H 'Content-Type: application/json' \
  -H 'Dagster-Cloud-Api-Token: [YOUR API TOKEN]' \
  'https://[YOUR ORG NAME].dagster.cloud/[YOUR DEPLOYMENT NAME]/report_asset_materialization/' \
  -d '
{
  "asset_key": "raw_transactions",
  "metadata": {
    "file_last_modified_at_ms": 1724614700266
  }
}'
```

</TabItem>
<TabItem value="oss" label="OSS">

Dagster OSS を使用する場合、認証ヘッダーは必要ありません。リクエストはオープン ソース URL (この例では `http://localhost:3000`) を指す必要があります。

```shell
curl \
  -X POST \
  -H 'Content-Type: application/json' \
  'http://localhost:3000/report_asset_materialization/' \
  -d '
{
  "asset_key": "raw_transactions",
  "metadata": {
    "file_last_modified_at_ms": 1724614700266
  }
}'
```

</TabItem>
</Tabs>

詳細については、[外部アセット REST API ドキュメント](/api/python-api/external-assets-rest-api)を参照してください。

## 外部アセットのグラフのモデリング

通常の Dagster アセットと同様に、外部アセットにも依存関係を設定できます。これは、別のシステムによってオーケストレーションされるデータ パイプライン全体をモデル化する場合に役立ちます。

<CodeExample path="docs_beta_snippets/docs_beta_snippets/guides/data-modeling/external-assets/dag-of-external-assets.py" language="python" />
