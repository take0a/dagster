---
title: "パーティション分割されたアセットとジョブからスケジュールを構築する"
description: "Learn to construct schedules for your partitioned jobs."
sidebar_position: 400
---

このガイドでは、パーティション化された [アセット](/guides/build/assets/) とジョブからスケジュールを構築する方法について説明します。最後に、次のことができるようになります:

- 時間分割ジョブのスケジュールを作成する
- パーティションジョブの開始時間をカスタマイズする
- セット内の最新のパーティションをカスタマイズする
- 静的にパーティション分割されたジョブのスケジュールを構築する

:::note

この記事は、以下の知識があることを前提としています。

- [スケジュール](index.md)
- [パーティション](/guides/build/partitions-and-backfills/partitioning-assets)
- [アセット定義](/guides/build/assets/defining-assets)
- [アセットジョブ](/guides/build/assets/asset-jobs) and op jobs

:::

## 時間ベースのパーティションの操作

時間でパーティション化されたジョブの場合、<PyObject section="schedules-sensors" module="dagster" object="build_schedule_from_partitioned_job"/> を使用してジョブのスケジュールを作成できます。スケジュールの間隔は、ジョブ内のパーティションの間隔と一致します。たとえば、実行されるたびにテーブルの日付パーティションを入力する毎日のパーティション化されたジョブがある場合、そのジョブを毎日実行することが考えられます。

<PyObject section="schedules-sensors" module="dagster" object="build_schedule_from_partitioned_job"/> を使用してスケジュールを構築するアセットおよびオペレーションベースのジョブの例については、次のタブを参照してください:

<Tabs>
<TabItem value="アセットジョブ">

**アセットジョブ**

アセット ジョブは、<PyObject section="assets" module="dagster" object="define_asset_job" /> を使用して定義されます。この例では、`partitioned_job` という名前のアセット ジョブを作成し、次に <PyObject section="schedules-sensors" module="dagster" object="build_schedule_from_partitioned_job"/> を使用して `asset_partitioned_schedule` を構築しました:

<CodeExample path="docs_snippets/docs_snippets/concepts/partitions_schedules_sensors/schedule_from_partitions.py" startAfter="start_partitioned_asset_schedule" endBefore="end_partitioned_asset_schedule" />

</TabItem>
<TabItem value="Op ジョブ">

**Op ジョブ**

Op ジョブは、<PyObject section="jobs" module="dagster" object="job" decorator /> を使用して定義されます。この例では、`partitioned_op_job` という名前のパーティション化されたジョブを作成し、次に <PyObject section="schedules-sensors" module="dagster" object="build_schedule_from_partitioned_job"/> を使用して `partitioned_op_schedule` を構築しました:

<CodeExample path="docs_snippets/docs_snippets/concepts/partitions_schedules_sensors/schedule_from_partitions.py" startAfter="start_marker" endBefore="end_marker" />

</TabItem>
</Tabs>

### スケジュールタイミングのカスタマイズ

`build_schedule_from_partitioned_job` の `minute_of_hour`、`hour_of_day`、`day_of_week`、および `day_of_month` パラメータを使用して、スケジュールのタイミングを制御できます。

次のジョブを考えてみましょう:

<CodeExample path="docs_snippets/docs_snippets/concepts/partitions_schedules_sensors/schedule_from_partitions.py" startAfter="start_partitioned_schedule_with_offset" endBefore="end_partitioned_schedule_with_offset" />

2024 年 5 月 20 日、スケジュールは午前 1 時 30 分 (UTC) に評価され、前日のパーティション キー `2024-05-19` の実行が開始されます。

### セット内の終了パーティションのカスタマイズ

:::tip

このセクションの例では日次パーティションを使用していますが、同じロジックは時間別、週別、月別パーティションなどの他の時間ベースのパーティションにも適用されます。

:::

パーティション化されたジョブの各スケジュール ティックは、ティック時間の時点で存在するパーティション セット内の最新のパーティションをターゲットにします。たとえば、毎日パーティション化されたジョブを実行するスケジュールを考えてみましょう。スケジュールが `2024-05-20` に実行されると、最新のパーティションがターゲットになり、これは前日の `2024-05-19` に相当します。

| この日にジョブが実行されると... | このパーティションをターゲットにします |
| ----------------------------- | ----------------------------- |
| 2024-05-20                    | 2024-05-19                    |
| 2024-05-21                    | 2024-05-20                    |
| 2024-05-22                    | 2024-05-21                    |

これは、各パーティションが **時間ウィンドウ** であるため発生します。時間ウィンドウは、開始時刻と終了時刻を持つ設定された期間です。パーティションのキーは時間ウィンドウの開始ですが、パーティションは時間ウィンドウが完了するまでパーティション セットに含まれません。時間ウィンドウが完了した後に実行を開始すると、実行は時間ウィンドウ全体のデータを処理できます。

日次パーティションの例を続けると、`2024-05-20` パーティションの開始時刻と終了時刻は次のようになります:

- **開始時刻** - `2024-05-20 00:00:00`
- **終了時刻** - `2024-05-20 23:59:59`

`2024-05-20 23:59:59` が経過すると、時間枠が完了し、Dagster は新しい `2024-05-20` パーティションをパーティション セットに追加します。この時点で、プロセスは次の時間枠 `2024-05-21` で繰り返されます。

セット内の最後のパーティションまたは最新のパーティションをカスタマイズする必要がある場合は、パーティションの設定で `end_offset` パラメータを使用します:

<CodeExample path="docs_snippets/docs_snippets/concepts/partitions_schedules_sensors/schedule_from_partitions.py" startAfter="start_offset_partition" endBefore="end_offset_partition" />

このパラメータを設定すると、各スケジュール ティックで埋められるパーティションが変更されます。正と負の整数が受け入れられ、次のような効果があります:

- `1` のような**正の数** を指定すると、スケジュールは**現在の** 時間/日/週/月のパーティションを埋めます。
- **負の数**、たとえば「-1」を指定すると、スケジュールは**早い**時間/日/週/月のパーティションを埋めます。

一般的に、`end_offset` の計算は次のように表すことができます:

```shell
current_date - 1 type_of_partition + end_offset
```

日ごとに分割されたスケジュールの例と、異なる `end_offset` 値がセット内の最新のパーティションにどのように影響するかを見てみましょう。この例では、開始日として `2024-05-20` を使用しています:

| End offset   | Calculated as                 | Ending (most recent) partition          |
| ------------ | ----------------------------- | --------------------------------------- |
| Offset of -1 | `2024-05-20 - 1 day + -1 day` | 2024-05-18 (2 days prior to start date) |
| No offset    | `2024-05-20 - 1 day + 0 days` | 2024-05-19 (1 day prior to start date)  |
| Offset of 1  | `2024-05-20 - 1 day + 1 day`  | 2024-05-20 (start date)                 |

## 静的パーティションの操作

次に、静的パーティションを持つジョブのスケジュールを作成する方法を説明します。これを行うには、<PyObject section="schedules-sensors" module="dagster" object="build_schedule_from_partitioned_job"/> などのヘルパー関数を使用するのではなく、<PyObject section="schedules-sensors" module="dagster" object="schedule" decorator /> デコレータを使用して、最初からスケジュールを構築します。これにより、スケジュールによって実行されるパーティションをより柔軟に決定できるようになります。

この例では、ジョブは大陸ごとに分割されています:

<CodeExample path="docs_snippets/docs_snippets/concepts/partitions_schedules_sensors/static_partitioned_asset_job.py" startAfter="start_job" endBefore="end_job" />

<PyObject section="schedules-sensors" module="dagster" object="schedule" decorator /> デコレータを使用して、各パーティション、つまり `continent` を対象とするスケジュールを記述します:

<CodeExample path="docs_snippets/docs_snippets/concepts/partitions_schedules_sensors/static_partitioned_asset_job.py" startAfter="start_schedule_all_partitions" endBefore="end_schedule_all_partitions" />

`Antarctica` パーティションのみをターゲットにしたい場合は、次のようなスケジュールを作成できます:

<CodeExample path="docs_snippets/docs_snippets/concepts/partitions_schedules_sensors/static_partitioned_asset_job.py" startAfter="start_single_partition" endBefore="end_single_partition" />

## このガイドのAPI

| 名前                                                      | 説明                                                                                         |
| --------------------------------------------------------- | --------------------------------------------------------------------------------------------------- |
| <PyObject section="schedules-sensors" module="dagster" object="schedule" decorator />                  | 指定された cron スケジュールに従って実行されるスケジュールを定義するデコレータ。                 |
| <PyObject section="schedules-sensors" module="dagster" object="build_schedule_from_partitioned_job" /> | パーティション化されたジョブのパーティション分割と一致する間隔を持つスケジュールを構築する関数。 |
| <PyObject section="schedules-sensors" module="dagster" object="RunRequest" />                          | 1 回の実行を開始するために必要なすべての情報を表すクラス。                        |
| <PyObject section="assets" module="dagster" object="define_asset_job" />                    |選択したアセットからジョブを定義する機能。            |
| <PyObject section="jobs" module="dagster" object="job" decorator />                       | ジョブを定義するために使用されるデコレータ。               |
