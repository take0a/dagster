---
title: 宣言型オートメーション
sidebar_position: 20
---

宣言型オートメーションは、アセットの状態やアセット間の依存関係に影響を与えるイベントに関する情報にアクセスして、次のことを可能にするフレームワークです:

- 最新のデータを使用して作業していることを確認します。
- 必要な場合にのみアセットを実体化したりチェックを実行したりすることで、リソースの使用を最適化します。
- 他のアセットの状態に基づいて、特定のアセットをいつ更新するかを正確に定義します。

宣言型オートメーションには 2 つのコンポーネントがあります:

- アセットまたはアセット チェックに設定される **[自動化条件](#automation-conditions)** は、個々のアセットまたはチェックをいつ実行する必要があるかを表します。
- **[自動化条件センサー](#automation-condition-sensors)** は、自動化条件を評価し、そのステータスに応じて実行を開始します。

## 宣言型オートメーションの使用

宣言型オートメーションを使用するには、次のことが必要です:

* [コード内のアセットまたはアセットチェックに自動化条件を設定する](#setting-automation-conditions-on-assets-and-asset-checks).
* Dagster UI で自動化状態センサーを有効にします:
    1. **Automation** に移動します。
    2. 目的のコードの場所を見つけます。
    3. **default_automation_condition_sensor** センサーをオンに切り替えます。

## 自動化条件

アセットまたはアセット チェックの <PyObject section="assets" module="dagster" object="AutomationCondition" /> は、作業を実行する条件を記述します。

Dagster は、一般的なユースケースを処理するために、いくつかの事前構築された自動化条件を提供します:

| 名前 | 条件 | 有用なケース |
|------|-----------|------------|
| `AutomationCondition.on_cron(cron_schedule)` | この条件は、すべての依存関係が更新された後、指定された `cron_schedule` でアセットを実現します。| 依存関係の更新方法の詳細を気にせずに、アセットを定期的に更新します。 |
| `AutomationCondition.on_missing()` | この条件では、すべての依存関係が更新されているが、アセット自体は更新されていない場合にアセットが実体化されます。 | 上流データが利用可能になるとすぐに、パーティション化されたアセットを入力します。 |
| `AutomationCondition.eager()` | この条件では、アセットがマテリアライズされます: <ul><li>アセットがこれまでにマテリアライズされたことがない場合、または</li><li>アセットの依存関係が更新されたとき (依存関係が現在欠落していないか、更新が進行中でない限り)。</li></ul> | アセット グラフを通じて変更を自動的に伝播します。<br /><br />アセットが最新の状態に保たれるようにします。|

### アセットとアセットチェックの自動化条件の設定

<PyObject section="assets" module="dagster" object="asset" decorator /> デコレータまたは <PyObject section="assets" module="dagster" object="AssetSpec" /> オブジェクトに自動化条件を設定できます:

```python
import dagster as dg

@dg.asset(automation_condition=dg.AutomationCondition.eager())
def my_eager_asset(): ...

AssetSpec("my_cron_asset", automation_condition=AutomationCondition.on_cron("@daily"))
```

<PyObject section="asset-checks" module="dagster" object="asset_check" decorator /> デコレータまたは <PyObject section="asset-checks" module="dagster" object="AssetCheckSpec" /> オブジェクトに自動化条件を設定することもできます:

```python
@dg.asset_check(asset=dg.AssetKey("orders"), automation_condition=dg.AutomationCondition.on_cron("@daily"))
def my_eager_check() -> dg.AssetCheckResult:
    return dg.AssetCheckResult(passed=True)


dg.AssetCheckSpec(
    "my_cron_check",
    asset=dg.AssetKey("orders"),
    automation_condition=dg.AutomationCondition.on_cron("@daily"),
)
```

### 自動化条件のカスタマイズ

[あらかじめ構築された自動化条件](#automation-conditions)がニーズに合わない場合は、独自の条件を構築できます。詳細については、「[自動化条件のカスタマイズ](customizing-automation-conditions/)」を参照してください。

## 自動化状態センサー

**default_automation_conditition_sensor** は、それが [有効](#using-declarative-automation) になっているコードの場所にあるすべての資産と資産チェックを監視します。そのコードの場所にある資産または資産チェックの自動化条件が満たされると、センサーはそれに応じて実行を実行します。

センサーの評価履歴は UI に表示されます:

![Default automation sensor evaluations in the Dagster UI](/images/guides/automate/declarative-automation/default-automation-sensor.png)

また、各アセットの評価の詳細な履歴は、アセットの「Asset Details」ページで確認することもできます。これにより、さまざまな時点でアセットが実体化された、または実体化されなかった理由を確認できます。

![Automation condition evaluations in the Asset Details page](/images/guides/automate/declarative-automation/evaluations-asset-details.png)

複数のセンサーを使用する場合、またはデフォルトのセンサーのプロパティを変更する場合は、<PyObject section="assets" module="dagster" object="AutomationConditionSensorDefinition" /> API ドキュメントを参照してください。
