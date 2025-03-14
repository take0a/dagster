---
title: スケジュールのトラブルシューティング
sidebar_position: 700
---

スケジュールに問題がある場合は、次の手順に従って問題を診断し、解決してください。

## Step 1: スケジュールが定義オブジェクトに含まれていることを確認します

まず、スケジュールが <PyObject section="definitions" module="dagster" object="Definitions" /> オブジェクトに含まれていることを確認します。これにより、スケジュールが Dagster UI や CLI などの Dagster ツールによって検出およびロード可能になります:

```python
defs = Definitions(
   assets=[asset_1, asset_2],
   jobs=[job_1],
   schedules=[all_assets_job_schedule],
)
```

詳細については、[コードの場所のドキュメント](/guides/deploy/code-locations/)を参照してください。

## Step 2: スケジュールが開始されたことを確認する

1. Dagster UI で、**Overview > Schedules タブ** をクリックします。
2. スケジュールを見つけます。開始されたスケジュールには、**Running** 列に有効なトグルが表示されます。

   ![Enabled toggle next to a schedule in the Schedules tab of the Overview page](/images/guides/automate/schedules/schedules-enabled-toggle.png)

## Step 3: 実行失敗を確認する

次に、スケジュールが正常に実行されたことを確認します。これは、**Schedules タブ** の **Last tick** 列を確認することで確認できます。

スケジュールされた実行に失敗した場合、この列には **Failed** バッジが表示されます。バッジをクリックすると、失敗を説明するエラーとスタック トレースが表示されます。

## Step 4:スケジュールの間隔設定を確認する

次に、スケジュールが期待どおりの時間間隔を使用していることを確認します。**Schedules** タブでスケジュールを見つけて、**Schedule** 列を確認します。

![Highlighted Next tick value for a schedule in the Dagster UI](/images/guides/automate/schedules/schedules-next-tick.png)

**Next tick** の値は、スケジュールが次にいつ実行される予定かを示します。上の画像では、次のティックは `5 月 2 日午前 12:00 UTC` です。

タイムゾーンを含め、時刻が予想どおりであることを確認します。

## Step 5: UIが最新のDagsterコードを使用していることを確認する

次のステップは、UI が最新バージョンの Dagster コードを使用していることを確認することです。タブを使用して、使用している Dagster のバージョンの手順を表示します。

<Tabs>
<TabItem value="ローカルウェブサーバーまたは Dagster OSS">

1. UI で、上部のナビゲーションにある **Settings** をクリックします。
2. **Code locations** タブで、ページの右上隅近くにある **Reload definitions** をクリックします。

</TabItem>
<TabItem value="Dagster+">

1. UI で、上部のナビゲーションにある **Deployment** をクリックします。
2. **Code locations** タブで、スケジュール定義が含まれているコードの場所を見つけます。
3. **Redeploy** をクリックします。

</TabItem>
</Tabs>

**コードの場所をロードできない場合** (たとえば、構文エラーのため)、**Status** は **Failed** になります。この列の **View error** リンクをクリックすると、エラー メッセージが表示されます。

**コードの場所が正常に読み込まれた** が、**Schedules** タブにスケジュールが存在しない場合は、スケジュールがコードの場所の `定義` オブジェクトに含まれていない可能性があります。詳細については、[Step 1](#step-1-verify-the-schedule-is-included-in-the-definitions-object) を参照してください。

## Step 6: dagster-daemon の設定を確認する

:::note

このセクションは、オープン ソース (OSS) のデプロイに適用されます。

:::

スケジュール間隔が正しく構成されているにもかかわらず実行が作成されない場合は、dagster-daemon プロセスが正しく動作していない可能性があります。まだ Dagster デーモンを設定していない場合は、詳細については [オープン ソース デプロイメント ガイド](/guides/deploy/deployment-options/) を参照してください。

### デーモンが実行中であることを確認する

1. UI で、上部のナビゲーションにある **Deployment** をクリックします。
2. **Daemons** タブをクリックします。
3. **Scheduler** 行を見つけます。

デーモン プロセスは、スケジューラから定期的にハートビートを送信します。スケジューラ デーモンのステータスが **Not running** の場合、デーモンの展開に問題があることを示しています。デーモンで例外が発生するエラーが発生した場合、このエラーがこのタブに表示されることがよくあります。

このページに明確なエラーがない場合、またはデーモンがハートビートを送信する必要があるのに送信されていない場合は、次の手順に進みます。

### デーモンプロセスのログを確認する

次に、デーモン プロセスからのログを確認します。これを行う手順は、デプロイメントによって異なります。たとえば、Kubernetes を使用している場合は、デーモンを実行しているポッドからログを取得する必要があります。スケジュールの名前 (または、スケジューラに関連付けられているすべてのログを表示するには、`SchedulerDaemon`) でこれらのログを検索して、何が問題なのかを把握できるはずです。

デーモンの出力にスケジュールが見つからなかったことを示すエラーが含まれている場合は、デーモンが Web サーバーと同じ `workspace.yaml` ファイルを使用していることを確認してください。デーモンは、`workspace.yaml` ファイルへの変更を反映するために再起動する必要はありません。詳細については、[ワークスペース ファイルのドキュメント](/guides/deploy/code-locations/workspace-yaml)を参照してください。

ログに問題の原因が示されていない場合は、次の手順に進みます。

### 実行失敗を確認する

最後のステップは、スケジュールが正常に実行されたことを確認することです。まだ実行していない場合は、詳細については [Step 3](#step-3-check-for-execution-failures) を参照してください。

## その他のヘルプ

**まだ問題が解決しない場合は?** これらの手順を実行しても問題が解決しない場合は、[Slack](https://dagster.io/slack) に連絡するか、[GitHub で問題を報告](https://github.com/dagster-io/dagster/issues)してください。

