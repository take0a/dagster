---
title: "アセットの監視"
description: Dagster provides functionality to record metadata about assets.
sidebar_position: 5000
---

アセットの監視は、特定のアセットに関するメタデータを記録するイベントです。アセットの実体化とは異なり、アセットの監視はアセットが変更されたことを意味するものではありません。

## 関連する API

| Name                                   | Description  |
| -------------------------------------- | ----------------------------------------- |
| <PyObject section="assets" module="dagster" object="AssetObservation" /> | アセットのメタデータが記録されたことを示す Dagster イベント。 |
| <PyObject section="assets" module="dagster" object="AssetKey" />         | 特定の外部アセットの一意の識別子。                 |

## 概要

<PyObject section="assets" module="dagster" object="AssetObservation" /> イベントは、特定のアセットに関するメタデータを Dagster に記録するために使用されます。アセット監視イベントは、実行時にオペレーションとアセット内でログに記録できます。アセットの監視を表示するには、<PyObject section="assets" module="dagster" object="asset" decorator /> デコレータを使用してアセットを定義するか、既存のマテリアライゼーションが必要です。

## オペレーションからの AssetObservation のログ記録

アセットに関するメタデータを記録したことを Dagster に認識させるには、オペレーション内から <PyObject section="assets" module="dagster" object="AssetObservation" /> イベントをログに記録します。これを行うには、コンテキストでメソッド <PyObject section="execution" module="dagster" object="OpExecutionContext.log_event" /> を使用します:

<CodeExample path="docs_snippets/docs_snippets/concepts/assets/observations.py" startAfter="start_observation_asset_marker_0" endBefore="end_observation_asset_marker_0" />

このアセットを実行すると、イベントログに監視イベントが表示されるはずです。

![asset observation](/images/guides/build/assets/asset-observations/observation.png)

### AssetObservation へのメタデータの添付

監視イベントに関連付けることができるメタデータにはさまざまな種類があり、すべて <PyObject section="metadata" module="dagster" object="MetadataValue" /> クラスを通じて行われます。各監視イベントはオプションでメタデータの辞書を取得し、イベントログと **Asset Details** ページに表示されます。使用可能なイベントメタデータの種類の詳細については、<PyObject section="metadata" module="dagster" object="MetadataValue" /> の API ドキュメントを参照してください。

<CodeExample path="docs_snippets/docs_snippets/concepts/assets/observations.py" startAfter="start_observation_asset_marker_2" endBefore="end_observation_asset_marker_2" />

**Asset Details** ページで、アセットアクティビティテーブルに観測結果が表示されます:

![asset activity observation](/images/guides/build/assets/asset-observations/asset-activity-observation.png)

### AssetObservation のパーティションを指定する

アセット全体を変更または作成するのではなく、アセットの単一のスライス（たとえば、大きなテーブル上の 1 日分のデータ）を監視している場合は、オブジェクトに `partition` 引数を含めることで、これを Dagster に示すことができます。

<CodeExample path="docs_snippets/docs_snippets/concepts/assets/observations.py" startAfter="start_partitioned_asset_observation" endBefore="end_partitioned_asset_observation" />

### 監視可能なソースアセット

<PyObject section="assets" module="dagster" object="SourceAsset" /> `DataVersion` を返すユーザー定義の監視関数が含まれる場合があります。監視関数が実行されるたびに、ソースアセットの <PyObject section="assets" module="dagster" object="AssetObservation" /> が生成され、返されたデータバージョンでタグ付けされます。アセットのデータバージョンが、下流アセットがマテリアライズされたときよりも新しいことが観察されると、下流アセットには、上流データが変更されたことを示すラベルが Dagster UI で付けられます。

<PyObject section="assets" module="dagster" object="AutomationCondition" pluralize /> を使用すると、このような状況が発生したときに下流のアセットを自動的に実体化できます。

<PyObject section="assets" module="dagster" object="observable_source_asset" /> デコレータは、監視関数を使用してソース アセットを定義する便利な方法を提供します。以下の監視可能なソースアセットは、ファイルハッシュを受け取り、それをデータバージョンとして返します。監視関数を実行するたびに、このハッシュがデータバージョンとして設定された新しい監視が生成されます。

<CodeExample path="docs_snippets/docs_snippets/concepts/assets/observable_source_assets.py" startAfter="start_plain" endBefore="end_plain" />

ファイルの内容が変更されると、ハッシュとデータバージョンが変更されます。これにより、このソースアセットの古い値 (つまり、異なるデータバージョン) から派生した下流アセットを更新する必要がある可能性があることが Dagster に通知されます。

ソースアセットの監視は、UI グラフ エクスプローラー ビューの「Observe sources」ボタンからトリガーできます。このボタンは、現在のグラフ内の少なくとも 1 つのソース アセットが監視関数を定義している場合にのみ表示されることに注意してください。

![observable source asset](/images/guides/build/assets/asset-observations/observe-sources.png)

ソース アセットの監視は、アセット ジョブの一部として実行することもできます。これにより、ソース アセットの監視をスケジュールに従って実行できます:

<CodeExample path="docs_snippets/docs_snippets/concepts/assets/observable_source_assets.py" startAfter="start_schedule" endBefore="end_schedule" />

:::note

現在、ソースアセットの監視は、アセットを具体化する標準アセットジョブの一部として実行できません。<PyObject section="assets" module="dagster" object="define_asset_job" /> の `​​selection` 引数は、監視可能なソースアセットのみをターゲットにする必要があります。通常のアセットと監視可能なソースアセットが混在して選択された場合は、エラーが送出されます。

:::
