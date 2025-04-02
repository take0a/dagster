<!-- ---
title: アセットジョブ
sidebar_position: 900
---

ジョブは、Dagster における [アセット定義](/guides/build/assets/defining-assets) の実行と監視の主要単位です。アセットジョブは、選択したアセットをターゲットとし、起動できるジョブの一種です:

- Dagster UIから手動で
- 一定の間隔で、[スケジュール](/guides/automate/schedules)によって
- 外部の変化が発生した場合、[センサー](/guides/automate/sensors)を使用して

## アセットジョブの作成

このセクションでは、次のアセットを対象とするいくつかのアセット ジョブを作成する方法を説明します:

<CodeExample path="docs_beta_snippets/docs_beta_snippets/guides/data-modeling/asset-jobs/asset-jobs.py" language="python" startAfter="start_marker_assets" endBefore="end_marker_assets" />

アセット ジョブを作成するには、[`define_asset_job`](/api/python-api/assets#dagster.define_asset_job) メソッドを使用します。アセット ベースのジョブは、ジョブが対象とするアセットとその依存関係に基づいています。

1 つまたは複数のアセットをターゲットにすることも、重複するアセット セットをターゲットとする複数のジョブを作成することもできます。次の例では、2 つのジョブがあります:

- `all_assets_job`はすべてのアセットをターゲットにします
- `sugary_cereals_job` は `sugary_cereals` アセットのみを対象とします

<CodeExample path="docs_beta_snippets/docs_beta_snippets/guides/data-modeling/asset-jobs/asset-jobs.py" language="python" startAfter="start_marker_jobs" endBefore="end_marker_jobs" />

## アセットジョブを Dagster ツールで利用できるようにする

Python モジュールまたはファイルの最上位レベルにある [`Definitions`](/api/python-api/definitions) オブジェクトにジョブを含めると、アセット ジョブを UI、GraphQL、およびコマンド ラインで使用できるようになります。Dagster ツールは、そのモジュールをコードの場所として読み込みます。スケジュールまたはセンサーを含めると、[コードの場所](/guides/deploy/code-locations) には、それらのスケジュールまたはセンサーが対象とするジョブが自動的に含まれます。

<CodeExample path="docs_snippets/docs_snippets/concepts/assets/jobs_to_definitions.py" />

## アセットジョブのテスト

Dagster には、ビジネス ロジックを環境から分離したり、制御できない入力に対する明示的な期待値を設定するなど、テストのサポートが組み込まれています。詳細については、[テストのドキュメント](/guides/test) を参照してください。

## アセットジョブの実行

アセットジョブはさまざまな方法で実行できます:

- ジョブが定義された Python プロセス内で
- コマンドライン経由
- GraphQL API経由
- UI内で

## 例

[Hacker News の例](https://github.com/dagster-io/dagster/tree/master/examples/project_fully_featured) [アセットグループをターゲットとするアセットジョブを構築します](https://github.com/dagster-io/dagster/blob/master/examples/project_fully_featured/project_fully_featured/jobs.py)。 -->
