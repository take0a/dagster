---
title: アセットの分割
description: Learn how to partition your data in Dagster.
sidebar_position: 100
---

Dagster では、パーティショニングは大規模なデータセットの管理、パイプラインのパフォーマンスの向上、増分処理の有効化を実現する強力な手法です。このガイドは、Dagster プロジェクトでデータ パーティショニングを実装する方法を理解するのに役立ちます。

Dagster でデータをパーティション分割する方法はいくつかあります:

- [時間ベースのパーティショニング](#time-based)、特定の時間間隔でデータを処理します
- [静的パーティション分割](#static-partitions)、定義済みのカテゴリに基づいてデータを分割します
- [２次元分割](#two-dimensional-partitions)は、2つの異なる軸に沿って同時にデータを分割します
- [動的パーティション分割](#dynamic-partitions) は、実行時情報に基づいてパーティションを作成するためのものです。

:::note

各アセットのパーティション数を 25,000 以下に制限することをお勧めします。パーティション数がこの制限を超えるアセットでは、UI での読み込み時間が遅くなる可能性があります。

:::

## 時間ベースのパーティション \{#time-based}

パーティショニングの一般的な使用例は、日次ログや月次レポートなど、時間間隔に分割できるデータを処理することです。

<CodeExample path="docs_snippets/docs_snippets/guides/data-modeling/partitioning/time_based_partitioning.py" language="python" />

## 定義済みカテゴリを持つパーティション \{#static-partitions}

データに事前定義されたカテゴリのセットがある場合があります。たとえば、異なる地域ごとにデータを個別に処理したい場合があります。

<CodeExample path="docs_snippets/docs_snippets/guides/data-modeling/partitioning/static_partitioning.py" language="python" />

{/* TODO: Link to Backfill page to explain how to backfill regional sales data */}

## ２次元パーティション \{#two-dimensional-partitions}

2 次元パーティション分割では、2 つの異なる軸に沿って同時にデータをパーティション分割できます。これは、複数の方法で分類できるデータを処理する必要がある場合に便利です。例:

<CodeExample path="docs_snippets/docs_snippets/guides/data-modeling/partitioning/two_dimensional_partitioning.py" language="python" />

この例では:

- `MultiPartitionsDefinition` を使用すると、`two_dimensional_partitions` は `date` と `region` の 2 つのディメンションで定義されます。
- パーティションキーは次のようになります: `2024-08-01|us`
- `daily_regional_sales_data` と `daily_regional_sales_summary` アセットは、同じ2次元パーティションスキームで定義されています。
- `daily_regional_sales_schedule` は毎日午前 1 時に実行され、すべての地域の前日のデータを処理します。`MultiPartitionKey` を使用して日付と地域の両方のディメンションのパーティション キーを指定すると、1 日に 3 回 (地域ごとに 1 回) 実行されます。

## 動的カテゴリを持つパーティション \{#dynamic-partitions}

場合によっては、パーティションが事前にわからないことがあります。たとえば、システムに追加された新しい領域を処理したい場合があります。このような場合は、動的パーティション分割を使用して、ランタイム情報に基づいてパーティションを作成できます。

次の例を考えてみましょう:

<CodeExample path="docs_snippets/docs_snippets/guides/data-modeling/partitioning/dynamic_partitioning.py" language="python" title="Dynamic partitioning" />

この例では:

- パーティションの値は詳細には不明なので、`動的パーティション定義`を使用して`リージョンパーティション`を定義します。
- トリガーされると、`all_regions_sensor` はすべてのリージョンをパーティション セットに動的に追加します。実行が開始されると、すべてのリージョンの実行が動的に開始されます。この例では、リージョンごとに 1 回ずつ、合計 6 回実行されます。

## パーティション化されたアセットの実体化

パーティション化されたアセットをマテリアライズするときに、マテリアライズするパーティションを選択すると、Dagster はパーティションごとに実行を開始します。

:::note

複数のパーティションを選択する場合、複数の実行をキューに入れるために [Dagster デーモン](/guides/deploy/execution/dagster-daemon) が実行されている必要があります。

:::

次の画像は、アセットの **Details** ページの **Launch runs** ダイアログを示しています。ここでは、実体化するパーティションを選択するように求められます:

![Rematerialize partition](/images/guides/build/partitions-and-backfills/rematerialize-partition.png)

パーティションが正常にマテリアライズされると、パーティション バーに緑色で表示されます:

![Successfully materialized partition](/images/guides/build/partitions-and-backfills/materialized-partitioned-asset.png)

## パーティション別にマテリアライゼーションを表示する

特定のアセットのパーティション別のマテリアライゼーションを表示するには、アセットの **Details** ページの **Activity** タブに移動します:

![Asset activity section of asset details page](/images/guides/build/partitions-and-backfills/materialized-partitioned-asset-activity.png)