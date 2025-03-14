---
title: "スケジュール"
sidebar_position: 10
---

スケジュールを使用すると、指定した間隔でジョブを自動的に実行できます。これらの間隔は、1 時間ごと、1 日ごと、1 週間ごとなどの一般的な頻度から、cron 式を使用して定義されるより複雑なパターンまでさまざまです。

<details>
<summary>前提条件</summary>

このガイドの手順を実行するには、次のものが必要です:

- [アセット](/guides/build/assets/) に関する知識
- [ジョブ](/guides/build/assets/asset-jobs/)に関する知識
</details>

## 基本的なスケジュール

基本的なスケジュールは、`ScheduleDefinition` クラスを使用して `JobDefinition` と `cron_schedule` によって定義されます。ジョブは、一緒に実行されるアセットまたは操作の選択と考えることができます。

<CodeExample path="docs_beta_snippets/docs_beta_snippets/guides/automation/simple-schedule-example.py" language="python" />

## 異なるタイムゾーンでスケジュールを実行する

デフォルトでは、タイムゾーンのないスケジュールは協定世界時 (UTC) で実行されます。別のタイムゾーンでスケジュールを実行するには、`timezone` パラメータを設定します。

```python
daily_schedule = ScheduleDefinition(
    job=daily_refresh_job,
    cron_schedule="0 0 * * *",
    # highlight-next-line
    timezone="America/Los_Angeles",
)
```

詳細については、「[スケジュールの実行タイムゾーンのカスタマイズ](customizing-execution-timezone)」を参照してください。

## パーティションからスケジュールを作成する

パーティションとジョブを使用する場合は、`build_schedule_from_partitioned_job` を使用してパーティションを使用してスケジュールを作成できます。スケジュールは、パーティション定義で指定された同じリズムで実行されます。

<Tabs>
<TabItem value="assets" label="Assets">

[パーティション化されたアセット](/guides/build/partitions-and-backfills)とジョブがある場合:

<CodeExample path="docs_beta_snippets/docs_beta_snippets/guides/automation/schedule-with-partition.py" language="python" />

</TabItem>
<TabItem value="ops" label="Ops">

パーティション化されたオペレーションジョブがある場合:

<CodeExample path="docs_beta_snippets/docs_beta_snippets/guides/automation/schedule-with-partition-ops.py" language="python" />

</TabItem>
</Tabs>

## 次は

これらの自動化の方法を理解し、効果的に使用することで、特定のニーズと制約に対応する、より効率的なデータ パイプラインを構築できます:

- スケジュールの詳細については、[自動化の理解](/guides/automate/index.md) を参照してください。
- [センサー](/guides/automate/sensors) でイベントに反応する
- スケジュールの代替として[宣言型自動化](/guides/automate/declarative-automation)を検討する
