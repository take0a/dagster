---
title: データのバックフィル
description: Dagster supports data backfills for each partition or subsets of partitions.
sidebar_position: 300
---

バックフィルとは、存在しないアセットのパーティションを実行するか、既存のレコードを更新するプロセスです。Dagster は、各パーティションまたはパーティションのサブセットのバックフィルをサポートしています。

[パーティション](/guides/build/partitions-and-backfills/partitioning-assets) を定義した後、複数のパーティションを同時に埋めるための実行を送信するバックフィルを起動できます。

バックフィルは、パイプラインを初めて設定するときによく行われます。マテリアライズするアセットには、アセットを最新の状態にするためにマテリアライズする必要がある履歴データが含まれている場合があります。バックフィルを実行するもう 1 つの一般的な理由は、アセットのロジックを変更し、新しいロジックで履歴データを更新する必要がある場合です。

## パーティション化されたアセットのバックフィルの開始

パーティション化されたアセットのバックフィルを開始するには、**Asset details** ページまたは **Global asset lineage** ページで **Materialize** ボタンをクリックします。バックフィル モーダルが表示されます。

最も上流のアセットが同じパーティションを共有している限り、選択したパーティション化されたアセットに対してバックフィルを起動することもできます。例: すべてのパーティションは `DailyPartitionsDefinition` を使用します。

![Backfills launch modal](/images/guides/build/partitions-and-backfills/asset-backfill-partition-selection-modal.png)

アセットのバックフィルの進行状況を確認するには、実行の **Runs details** ページに移動します。このページにアクセスするには、**Runs tab** をクリックし、実行の ID をクリックします。バックフィルによって開始された実行を含むすべての実行を表示するには、**Show runs within backfills** ボックスをオンにします:

![Asset backfill details page](/images/guides/build/partitions-and-backfills/asset-backfill-details-page.png)

## バックフィルポリシーを使用して単一実行のバックフィルを開始する

import Beta from '@site/docs/partials/\_Beta.md';

<Beta />

デフォルトでは、`N` 個のパーティションをカバーするバックフィルを開始すると、Dagster は各パーティションに対して 1 つずつ、`N` 個の個別の実行を開始します。このアプローチは、大量のデータで Dagster またはリソースが過負荷になるのを防ぐのに役立ちます。ただし、Spark や Snowflake などの並列処理エンジンを使用している場合は、並列処理に Dagster を必要としないことが多いため、バックフィルを複数の実行に分割すると、余分なオーバーヘッドが追加されるだけです。

Dagster は、単一の Snowflake クエリとしてバックフィルを実行するなど、パーティションの範囲をカバーする単一の実行として実行されるバックフィルをサポートしています。実行が完了すると、Dagster はすべてのパーティションが埋められたことを追跡します。

:::note

単一実行のバックフィルは、アセット グラフまたはアセット ページから起動された場合、またはアセットが、含まれるすべてのアセット間で同じバックフィル ポリシーを共有するアセット ジョブの一部である場合にのみ機能します。

:::

この動作を実現するには、次の操作が必要です:

- アセットの `backfill_policy` (<PyObject section="partitions" module="dagster" object="BackfillPolicy" />) を `single_run` に設定する
- **単一のパーティションではなく、パーティションの範囲で動作するコードを記述します**。つまり、コードで `partition_key` コンテキスト プロパティを使用している場合は、代わりに次のいずれかのプロパティを使用するように更新する必要があります。

  - [`partition_time_window`](/api/python-api/execution#dagster.OpExecutionContext.partition_time_window)
  - [`partition_key_range`](/api/python-api/execution#dagster.OpExecutionContext.partition_key_range)
  - [`partition_keys`](/api/python-api/execution#dagster.OpExecutionContext.partition_keys)

  どのプロパティを使用するかは、開始/終了日時オブジェクト、開始/終了パーティション キー、またはパーティション キーのリストのどれを操作するのが最も便利かによって異なります。

<CodeExample path="docs_snippets/docs_snippets/concepts/partitions_schedules_sensors/backfills/single_run_backfill_asset.py" startAfter="start_marker" endBefore="end_marker" />

