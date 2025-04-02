---
title: Dagster ウェブサーバーと UI
description: 'The Dagster UI is a web-based interface for Dagster. You can inspect Dagster objects (ex: assets, jobs, schedules), launch runs, view launched runs, and view assets produced by those runs.'
sidebar_position: 20
---

Dagster Web サーバーは、Dagster オブジェクトを表示および操作するための Web ベースのインターフェイスである Dagster UI を提供します。また、GraphQL クエリにも応答します。

UI では、Dagster オブジェクト (アセット、ジョブ、スケジュールなど) を検査したり、実行を開始したり、開始された実行を表示したり、それらの実行によって生成されたアセットを表示したりできます。

## ウェブサーバーの起動

ローカル開発中にコマンドラインから Web サーバーを起動する最も簡単な方法は、次のコマンドを実行することです:

```shell
dagster dev
```

このコマンドは、Dagster Web サーバーと [Dagster デーモン](/guides/deploy/execution/dagster-daemon) の両方を起動し、コマンド ラインから Dagster の完全なローカル展開を開始できるようにします。

このコマンドは、ブラウザで UI にアクセスできる URL (通常はポート 3000) を出力します。

呼び出されると、Web サーバーは、Python モジュールまたはパッケージ内の <PyObject section="definitions" module="dagster" object="Definitions" /> オブジェクト、またはオープン ソース デプロイメントの [ワークスペース ファイル](/guides/deploy/code-locations/workspace-yaml) で構成されたコードの場所から、アセット、ジョブ、スケジュール、センサー、リソースなどの定義を取得します。詳細については、[コードの場所のドキュメント](/guides/deploy/code-locations/) を参照してください。

次のコマンドを実行して、コマンドラインから Web サーバーを単独で起動することもできます:

```shell
dagster-webserver
```

スケジュールやセンサーなどのいくつかの Dagster 機能が機能するには、Dagster デーモンが実行されている必要があることに注意してください。

## Dagster UI リファレンス

### Overview ページ

- **説明**: このページは「ファクトリー フロア」とも呼ばれ、すべてのコードの場所における Dagster デプロイメントのアクティビティの概要を示します。これには、実行、ジョブ、スケジュール、センサー、リソース、バックフィルに関する情報が含まれており、これらすべてにこのページのタブを使用してアクセスできます。

- **アクセス方法**: 上部のナビゲーションバーの **Overview** をクリック

![The Overview tab, also known as the Factory Floor, in the Dagster UI](/images/guides/operate/webserver/factory-floor.png)

### Assets

<Tabs>
<TabItem value="Asset catalog">

**アセットカタログ (OSS)**

- **説明**: **Asset catalog** ページには、Dagster デプロイメント内のすべての [アセット](/guides/build/assets/) が一覧表示されます。これらは、アセット キー、コンピューティングの種類、アセット グループ、[コードの場所](/guides/deploy/code-locations/)、[タグ](/guides/build/assets/metadata-and-tags/index.md#tags) でフィルター処理できます。アセットをクリックすると、そのアセットの **Asset details** ページが開きます。**Global asset lineage** ページに移動して、定義を再読み込みし、アセットを実体化することもできます。

- **アクセス方法:** 上部のナビゲーションバーで **Assets** をクリックします

![The Asset Catalog page in the Dagster UI](/images/guides/operate/webserver/asset-catalog.png)

</TabItem>
<TabItem value="Asset catalog (Dagster+ Pro)">

**アセットカタログ (Dagster+ Pro)**

:::note

この機能は Dagster+ Pro でのみご利用いただけます。

:::

- **説明**: このバージョンの **Asset catalog** ページには、コンピューティングの種類、アセット グループ、[コードの場所](/guides/deploy/code-locations/)、[タグ](/guides/build/assets/metadata-and-tags/index.md#tags)、[所有者](/guides/build/assets/metadata-and-tags/index.md#owners) などによって分類された、元のページのすべての情報と機能が含まれています。このページでは、次の操作を実行できます。

  - Dagster デプロイメント内のすべての [アセット](/guides/build/assets/) を表示します
  - 特定のアセットをクリックすると詳細が表示されます
  - アセット キー、コンピューティングの種類、アセット グループ、コードの場所、タグ、所有者などでアセットを検索します。
  - グローバルアセット系統にアクセスできます
  - 定義を再読み込み

- **アクセス方法:** 上部のナビゲーションで **Catalog** をクリックします

![The Asset Catalog page in the Dagster UI](/images/guides/operate/webserver/asset-catalog-cloud-pro.png)

</TabItem>
<TabItem value="Catalog views (Dagster+)">

**カタログビュー (Dagster+)**

- **説明**: **カタログ ビュー** は、**アセット カタログ** に対して一連のフィルターを保存し、表示したいアセットのみを表示します。これらのビューを共有することで、簡単にアクセスでき、チームの共同作業が迅速化されます。**カタログ ビュー** を使用すると、次のことができます。

  - Dagster デプロイメント内の [アセット](/guides/build/assets) のスコープセットをフィルターします
  - アセットの共有ビューを作成してチームのコラボレーションを容易にする

- **アクセス方法:**

  - 上部のナビゲーションで **Catalog** をクリックする
  - **Global asset lineage から**: **Catalog** ページの右上隅にある **View global asset lineage** をクリックします

![The Catalog views dropdown in the Dagster+ Pro Catalog UI](/images/guides/operate/webserver/catalog-views.png)

</TabItem>
<TabItem value="Global asset lineage">

**グローバルアセットの系統**

- **説明**: **Global asset lineage** ページには、すべてのコードの場所にわたる Dagster デプロイメント内のすべてのアセット間の依存関係が表示されます。このページでは、次の操作を実行できます。

  - グループ別にアセットをフィルタリング
  - [アセット選択構文](/guides/build/assets/asset-selection-syntax)を使用してアセットのサブセットをフィルタリングします。
  - 定義を再読み込み
  - 全てのアセットまたは選択したアセットを実体化する
  - あらゆるアセットの最新の実体化の実行詳細を表示します

- **アクセス方法:**

  - **Asset catalog から**: ページの右上隅にある**View global asset lineage**をクリックします。
  - **Asset details ページから**: **Lineage タブ**をクリックする

![The Global asset lineage page in the Dagster UI](/images/guides/operate/webserver/global-asset-lineage.png)

</TabItem>
<TabItem value="Asset details">

**アセットの詳細**

- **説明**: **資産の詳細** ページには、単一の資産に関する詳細が表示されます。このページのタブを使用して、資産に関する詳細情報を表示します:

  - **Overview** - アセットの説明、リソース、構成、タイプなどのアセットに関する情報。
  - **Partitions** - アセットのパーティション（そのマテリアライズステータス、メタデータ、実行情報を含む）
  - **Events** - アセットの実体化の履歴
  - **Checks** - アセットに対して定義された[アセットチェック](/guides/test/asset-checks)
  - **Lineage** - **Global asset lineage** ページのアセット系統
  - **Automation** - アセットに関連付けられた[宣言型自動化条件](/guides/automate/declarative-automation)
  - **Insights** - **Dagster+ のみ。** 障害やクレジットの使用状況など、アセットに関する履歴情報。詳細については、[Dagster+ Insights](/dagster-plus/features/insights/) のドキュメントを参照してください。

- **アクセス方法**: **Asset catalog** 内のアセットをクリックする

![The Asset Details page in the Dagster UI](/images/guides/operate/webserver/asset-details.png)

</TabItem>
</Tabs>

### 実行

<Tabs>
<TabItem value="All runs">

**すべての実行**

- **Description**: **Runs** ページにはすべてのジョブ実行が一覧表示され、ジョブ名、実行 ID、実行ステータス、またはタグでフィルタリングできます。実行 ID をクリックすると、**Run details** ページが開き、その実行の詳細が表示されます。

- **アクセス方法**: 上部のナビゲーションバーで **Runs** をクリック

![UI Runs page](/images/guides/operate/webserver/runs-page.png)

</TabItem>
<TabItem value="Run details">

**実行の詳細**

- **Description**: **Run details** ページには、タイミング情報、エラー、ログなど、1 回の実行に関する詳細が表示されます。左上のペインには、各アセットまたはオペレーションの実行にかかった時間を示すガント チャートが表示されます。下のペインには、実行中に生成されたフィルター可能なイベントとログが表示されます。

  このページでは、次のことができます:

  - **構造化されたイベント ログと生のコンピューティング ログを表示します。** 詳細については、実行ログ タブを参照してください。
  - **Re-execute** ボタンをクリックして、同じ構成を使用して **実行を再実行** します。関連する実行 (同じ以前の実行を再実行して作成された実行など) は、簡単に参照できるように右側のペインにグループ化されます。

- **アクセス方法**: **Run details** ページで実行をクリックする

![UI Run details page](/images/guides/operate/webserver/run-details.png)

</TabItem>
<TabItem value="Run logs">

**実行ログ**

- **説明**: **Run details** ページの下部にある実行ログには、実行中に発生したすべてのイベント、イベントの種類、イベント自体の詳細情報が表示されます。ログには 2 つの種類があり、次のセクションで説明します。

  - 構造化されたイベントログ
  - 生の計算ログ

- **アクセス方法**: **Run details** ページの一番下までスクロールします

**構造化されたイベントログ**

- **説明**: 構造化されたログは、メタデータによって強化され、分類されます。たとえば、ログがどの資産に関するものであるかを示すラベル、資産のメタデータへのリンク、利用可能なイベントの種類などです。この構造化により、ログのフィルタリングと検索も容易になります。

- **アクセス方法**: ログフィルターフィールドの横にあるトグルの**左側**をクリックします

![Structured event logs in the Run details page](/images/guides/operate/webserver/run-details-event-logs.png)

**生の計算ログ**

- **説明**: 生のコンピューティング ログには、[`stdout` と `stderr`](https://stackoverflow.com/questions/3385201/confused-about-stdin-stdout-and-stderr) の両方のログが含まれており、切り替えることができます。ログをダウンロードするには、ログの右上隅にある **矢印アイコン** をクリックします。

- **アクセス方法**: ログフィルターフィールドの横にあるトグルの**右側**をクリックします

![Raw compute logs in the Run details page](/images/guides/operate/webserver/run-details-compute-logs.png)

</TabItem>
</Tabs>

### スケジュール

<Tabs>
<TabItem value="All schedules">

**すべてのスケジュール**

- **説明**: **Schedules** ページには、Dagster デプロイメントで定義されているすべての [スケジュール](/guides/automate/schedules) と、予想されるスケジュール実行の今後のティックに関する情報が一覧表示されます。スケジュールをクリックすると、**Schedule details** ページが開きます。

- **アクセス方法**: **Overview (top nav) > Schedules タブ** をクリック

![UI Schedules page](/images/guides/operate/webserver/schedules-tab.png)

</TabItem>
<TabItem value="Schedule details">

**スケジュールの詳細**

- **説明**: **Schedule details** ページには、次のティック、ティック履歴、実行履歴など、単一のスケジュールに関する詳細が表示されます。ページの右上隅にある **Preview tick result** ボタンをクリックすると、スケジュールをテストできます。

- **アクセス方法**: **Schedules** ページでスケジュールをクリックします。

![UI Schedule details page](/images/guides/operate/webserver/schedule-details.png)

</TabItem>
</Tabs>

### センサー

<Tabs>
<TabItem value="All sensors">

**すべてのセンサー**

- **説明**: **Sensors** ページには、Dagster デプロイメントで定義されているすべての [センサー](/guides/automate/sensors) と、センサーの頻度および最後のティックに関する情報が一覧表示されます。センサーをクリックすると、最近のティック履歴や最近の実行など、センサーの詳細が表示されます。

- **アクセス方法**: **Overview (top nav) > Sensors タブ** をクリック

![UI Sensors page](/images/guides/operate/webserver/sensors-tab.png)

</TabItem>
<TabItem value="Sensor details">

**センサーの詳細**

- **説明**: **Sensor details** ページには、次のティック、ティック履歴、実行履歴など、単一のセンサーに関する詳細が表示されます。ページの右上隅近くにある **Preview tick result** ボタンをクリックすると、センサーをテストできます。

- **アクセス方法**: **Sensors** ページでセンサーをクリックする

![UI Sensor details page](/images/guides/operate/webserver/sensor-details.png)

</TabItem>
</Tabs>

### リソース

<Tabs>
<TabItem value="All resources">

**すべてのリソース**

- **説明**: **Resources** ページには、すべてのコードの場所にわたって、Dagster デプロイメントで定義されているすべての [リソース](/guides/build/external-resources/) が一覧表示されます。リソースをクリックすると、**Resource details** ページが開きます。

- **アクセス方法**: **Overview (top nav) > Resources タブ** をクリック

![UI Resources page](/images/guides/operate/webserver/resources-tab.png)

</TabItem>
<TabItem value="Resource details">

**リソースの詳細**

- **説明**: **Resource details** ページには、リソースの構成、説明、使用方法など、リソースに関する詳細情報が表示されます。このページのタブの詳細については、下のタブをクリックしてください。

- **アクセス方法**: **Resources** ページでリソースをクリックします。

<Tabs>
<TabItem value="Configuration tab">

**構成タブ**

- **説明**: **Configuration** タブには、各キーの名前、タイプ、各構成値の値など、リソースの構成に関する詳細情報が含まれています。キーの値が [環境変数](/guides/deploy/using-environment-variables-and-secrets) の場合、値の横に `Env var` バッジが表示されます。

- **アクセス方法**: **Resource details** ページで、**Configuration タブ** をクリックします。

![UI Resource details - Configuration tab](/images/guides/operate/webserver/resource-details-configuration-tab.png)

</TabItem>
<TabItem value="Uses tab">

**リソースを使用するもののタブ**

- **説明**: **Uses** タブには、[アセット](/guides/build/assets/)、[ジョブ](/guides/build/jobs/)、[ops](/guides/build/ops/) など、リソースを使用する他の Dagster 定義に関する情報が含まれています。これらの定義のいずれかをクリックすると、その定義タイプの詳細ページが開きます。

- **アクセス方法**: **Resource details** ページで **Uses タブ**をクリック

![UI Resource details - Uses tab](/images/guides/operate/webserver/resource-details-uses-tab.png)

</TabItem>
</Tabs>
</TabItem>
</Tabs>

### バックフィル

- **説明**: **Backfills** タブには、すべてのコードの場所にわたる Dagster デプロイメントのバックフィルに関する情報が含まれています。パーティションが作成された日時、ターゲット、ステータス、実行ステータスなどの情報が含まれます。

- **アクセス方法**: **Overview (top nav) > Backfills タブ** をクリック

![UI Backfills tab](/images/guides/operate/webserver/backfills-tab.png)

### ジョブ

<Tabs>
<TabItem value="All jobs">

**すべてのジョブ**

- **説明**: **Jobs** ページには、すべてのコードの場所にわたる Dagster デプロイメントで定義されているすべての [ジョブ](/guides/build/jobs/) が一覧表示されます。ジョブのスケジュールまたはセンサー、最新の実行時間、履歴に関する情報が含まれます。ジョブをクリックすると、**Job details** ページが開きます。

- **アクセス方法**: **Overview (top nav) > Jobs タブ** をクリック

![UI Job Definition](/images/guides/operate/webserver/jobs-tab.png)

</TabItem>
<TabItem value="Job details">

**ジョブの詳細**

- **説明**: **Job details** ページには、ジョブに関する詳細情報が表示されます。このページのタブの詳細については、下のタブをクリックしてください。

- **アクセス方法**: **Jobs** ページでジョブをクリックします。

<Tabs>
<TabItem value="Overview tab">

**概要タブ**

- **説明**: **Job details** ページの **Overview** タブには、ジョブを構成するアセットやオペレーションのグラフが表示されます。

- **アクセス方法:** **Job details** ページで **Overview** タブをクリック

![UI Job Definition](/images/guides/operate/webserver/job-definition-with-ops.png)

</TabItem>
<TabItem value="Launchpad tab">

**Launchpad タブ**

- **説明**: **Launchpad タブ** には、構成を試したり実行を開始したりするための構成エディターが用意されています。**注**: アセットの場合、このタブはジョブに構成が必要な場合にのみ表示されます。デフォルトでは、すべてのオペレーション ジョブに表示されます。

- **アクセス方法:** **Job details** ページで、**Launchpad** タブをクリックします

![UI Launchpad](/images/guides/operate/webserver/job-config-with-ops.png)

</TabItem>
<TabItem value="Runs tab">

**実行タブ**

- **説明**: **Runs** タブには、ジョブの最近の実行の一覧が表示されます。実行をクリックすると、**Run details** ページが開きます。

- **アクセス方法:** **Job details** ページで、**Runs** タブをクリックします

![UI Job runs tab](/images/guides/operate/webserver/jobs-runs-tab.png)

</TabItem>
<TabItem value="Partitions tab">

**パーティションタブ**

- **説明**: **Partitions** タブには、パーティションの合計数、不足しているパーティションの数、ジョブのバックフィル履歴など、ジョブに関連付けられている [パーティション](/guides/build/partitions-and-backfills) に関する情報が表示されます。**注**: このタブは、ジョブにパーティションが含まれている場合にのみ表示されます。

- **アクセス方法:** **Job details** ページで、**Partitions** タブをクリックします

![UI Job Partitions tab](/images/guides/operate/webserver/jobs-partitions-tab.png)

</TabItem>
</Tabs>

</TabItem>
</Tabs>

### デプロイ

**Deployment** ページには、Dagster デプロイメント内のコードの場所のステータス、デーモン (オープンソース) またはエージェント (クラウド) の健全性、スケジュール、センサー、および構成の詳細に関する情報が含まれています。

<Tabs>
<TabItem value="Code locations tab">

**コードの場所タブ**

- **説明**: **Code locations** タブには、Dagster デプロイメント内のコードの場所に関する情報 (現在のステータス、最終更新日時、そこに含まれる定義の概要など) が含まれています。次の方法で Dagster 定義を再読み込みできます:

  - **Reload all** をクリックすると、すべてのコード位置のすべての定義が再読み込みされます。
  - 特定のコード位置の横にある **Reload** をクリックすると、そのコード位置の定義のみが再読み込みされます。

- **アクセス方法:**
  - 上部のナビゲーションバーで **Deployment**をクリックする
  - **Deployment overview**ページで、**Code locations**タブをクリックします

![UI Deployment overview page](/images/guides/operate/webserver/deployment-code-locations.png)

</TabItem>
<TabItem value="Open Source (OSS)">

**オープンソース (OSS)**

**Code locations** タブに加えて、Dagster OSS デプロイメントにはいくつかの追加タブが含まれています。詳細については、以下のタブをクリックしてください。

<Tabs>
<TabItem value="Daemons tab">

**Daemons タブ**

- **説明**: **Daemons** タブには、オープン ソース Dagster デプロイメント内の [デーモン](/guides/deploy/execution/dagster-daemon) に関する情報 (現在のステータスや最後のハートビートの検出時刻など) が含まれます。
- **アクセス方法**: **Deployment overview** ページで、**Daemons** タブをクリックします

![UI Deployment - Daemons tab](/images/guides/operate/webserver/deployment-daemons-tab.png)

</TabItem>
<TabItem value="Configuration tab">

**構成タブ**

- **説明**: **Configuration** タブには、[`dagster.yaml`](/guides/deploy/dagster-yaml) ファイルを通じて管理される Dagster デプロイメントの構成に関する情報が表示されます。
- **アクセス方法**: **Deployment overview** ページで、**Configuration** タブをクリックします

![UI Deployment - Configuration tab](/images/guides/operate/webserver/deployment-configuration-tab.png)

</TabItem>
</Tabs>
</TabItem>

<TabItem value="Dagster+">

**Dagster+**

**Code locations** タブに加えて、Dagster+ デプロイメントにはいくつかの追加タブが含まれています。詳細については、以下のタブをクリックしてください。

<Tabs>
<TabItem value="Agents tab">

**エージェントタブ**

- **説明**: **Agents** タブには、Dagster+ デプロイ内のエージェントに関する情報が含まれています。
- **アクセス方法**: **Deployment overview** ページで、**Agents** タブをクリックします

![UI Dagster+ Deployment - Agents tab](/images/guides/operate/webserver/deployment-cloud-agents-tab.png)

</TabItem>
<TabItem value="Environmental variables tab">

**環境変数タブ**

- **説明**: **Agents** タブには、Dagster+ デプロイメントで構成された環境変数に関する情報が含まれています。詳細については、[Dagster+ 環境変数のドキュメント](/dagster-plus/deployment/management/environment-variables/)を参照してください。
- **アクセス方法**: **Deployment overview** ページで、**Environment variables** タブをクリックします

![UI Cloud Deployment - Environment variables tab](/images/guides/operate/webserver/deployment-cloud-environment-variables-tab.png)

</TabItem>
<TabItem value="Alerts tab">

**アラートタブ**

- **説明**: **Alerts** タブには、Dagster+ デプロイメント用に構成されたアラート ポリシーに関する情報が含まれています。詳細については、[Dagster+ アラート ガイド](/dagster-plus/features/alerts/) を参照してください。
- **アクセス方法**: **Deployment overview** ページで、**Alerts** タブをクリックします

![UI Dagster+ Deployment - Alerts tab](/images/guides/operate/webserver/deployment-cloud-alerts-tab.png)

</TabItem>
</Tabs>
</TabItem>
</Tabs>
