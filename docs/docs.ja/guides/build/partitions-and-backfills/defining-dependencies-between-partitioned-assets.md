---
title: パーティション化されたアセット間の依存関係の定義
description: Learn how to define dependencies between partitioned and unpartitioned assets in Dagster.
sidebar_label: Partitioning dependencies
sidebar_position: 200
---

パーティション化されたアセットをさまざまな方法でモデル化する方法を確認しました。パーティション化されたアセット間、またはパーティション化されていないアセット間の依存関係を定義する必要があるかもしれません。

Dagster のパーティション化されたアセットは、他のパーティション化されたアセットと依存関係を持つことができるため、1 つのパーティション化されたアセットの出力が別のパーティション化されたアセットに送られる複雑なデータ パイプラインを作成できます。仕組みは次のとおりです:

- 下流のアセットは上流のアセットの１つ以上のパーティションに依存できる
- パーティションスキームは同一である必要はありませんが、互換性が必要である

## デフォルトのパーティション依存関係ルール

デフォルトのパーティション間の依存関係には、いくつかのルールが適用されます:

- 上流アセットと下流アセットに同じ <PyObject section="partitions" module="dagster" object="PartitionsDefinition" /> がある場合、下流アセットの各パーティションは上流アセットの同じパーティションに依存します。
- 上流アセットと下流アセットの両方が [時間ウィンドウ パーティション分割](partitioning-assets#time-based) されている場合、下流アセット内の各パーティションは、その時間ウィンドウと交差する上流アセット内のすべてのパーティションに依存します。

たとえば、<PyObject section="partitions" module="dagster" object="DailyPartitionsDefinition" /> を持つアセットが <PyObject section="partitions" module="dagster" object="HourlyPartitionsDefinition" /> を持つアセットに依存している場合、日次アセットのパーティション `2024-04-12` は、時間別アセットの 24 個のパーティション (`2024-04-12-00:00` から `2024-04-12-23:00`) に依存します。

## デフォルトの依存関係ルールを上書きする

アセットへの依存関係を指定するときに <PyObject section="partitions" module="dagster" object="PartitionMapping" /> を指定すると、デフォルトのパーティション依存関係ルールを上書きできます。これがどのように実現されるかは、アセットが持つ依存関係のタイプによって異なります。

### 基本的なアセットの依存関係

基本的なアセット依存関係のパーティション依存関係ルールをオーバーライドするには、<PyObject section="assets" module="dagster" object="AssetDep" /> を使用して、上流アセットのパーティション依存関係を指定します:

<CodeExample path="docs_snippets/docs_snippets/concepts/partitions_schedules_sensors/partitioned_asset_mappings.py" />

### 管理されたロードアセットの依存関係

管理読み込みアセットの依存関係のパーティション依存関係ルールをオーバーライドするには、<PyObject section="partitions" module="dagster" object="PartitionMapping" /> を使用して、アセットの各パーティションがアップストリーム アセットのパーティションに依存するように指定します。

次のコードでは、<PyObject section="partitions" module="dagster" object="TimeWindowPartitionMapping" /> を使用して、毎日パーティション分割されたアセットの各パーティションが、上流のアセット内の前日のパーティションに依存するように指定します:

<CodeExample path="docs_snippets/docs_snippets/concepts/partitions_schedules_sensors/partition_mapping.py" />

利用可能な `PartitionMappings` のリストについては、[API ドキュメント](/api/python-api/partitions#dagster.PartitionMapping) を参照してください。

## 例

### 異なる時間ベースのパーティション間の依存関係 \{#different-time-dependencies}

次の例では、単一のスケジュールで同時に実行できる `daily_sales_data` と `daily_sales_summary` の 2 つのパーティションを作成します。

<details>
<summary>例を表示</summary>

<CodeExample path="docs_beta_snippets/docs_beta_snippets/guides/data-modeling/partitioning/time_based_partitioning.py" language="python" />

</details>

ただし、異なる時間ベースのパーティション間の依存関係を定義したい場合もあります。たとえば、毎日のデータを週次レポートに集計したい場合などです。

次の例を考えてみましょう:

<CodeExample path="docs_beta_snippets/docs_beta_snippets/guides/data-modeling/partitioning/time_based_partition_dependencies.py" language="python" />

この例では:

- 日ごとに分割された `daily_sales_data` アセットがあり、毎日実行されます。
- `weekly_sales_summary` アセットは、毎週実行され、`daily_sales_data` アセットに依存します。

  - このアセットでは、週ごとのパーティションはすべての親パーティション (週 7 日間すべて) に依存します。週の開始と終了を含むパーティション キーの範囲を取得するには、`context.asset_partition_key_range_for_input("daily_sales_data")` を使用します。

- これらのアセットの実行を自動化するには:

  - まず、`weekly_sales_summary` アセットに `automation_condition=AutomationCondition.eager()` を指定します。これにより、`daily_sales_data` の 7 つの日次パーティションがすべて最新になった後に毎週実行されるようになります。
  - 次に、`daily_sales_data` アセットに `automation_condition=AutomationCondition.cron(cron_schedule="0 1 * * *")` を指定します。これにより、毎日実行されるようになります。


:::tip

上記の例のように、複雑な依存関係ロジックを持つコードでは、[スケジュール](/guides/automate/schedules)ではなく[自動化条件](/guides/automate/declarative-automation/)を使用することをお勧めします。自動化条件は、アセットを実行するタイミングを指定するため、カスタム スケジュール ロジックを追加しなくても実行条件を定義できます。

:::


{/* ## Dependencies between time-based partitions and un-partitioned assets */}

{/* TODO */}

{/* ## Dependencies between time-based and static partitions */}

{/* Combining time-based and static partitions allows you to analyze data across both temporal and categorical dimensions. This is particularly useful for scenarios like regional time series analysis. */}

{/* TODO */}

{/* ## Dependencies between time-based and dynamic partitions */}

{/* TODO */}

{/* ## Dependencies between time-based partitions and un-partitioned assets */}

{/* TODO */}

{/* ## Integrating Dagster partitions with external systems: incremental models and dbt */}

{/* TODO */}
