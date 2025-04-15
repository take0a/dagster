---
title: アセットジョブ
sidebar_position: 100
---

ジョブは、Dagsterにおける[アセット定義](/guides/build/assets/defining-assets)の実行と監視の主要単位です。
アセットジョブは、[選択したアセット](/guides/build/assets/asset-selection-syntax)を対象とするジョブの一種で、以下の方法で起動できます。

- Dagster UIから手動で起動
- [スケジュール](/guides/automate/schedules)に基づいて一定間隔で起動
- [センサー](/guides/automate/sensors)を使用して外部の変化が発生したときに起動

## アセットジョブの作成

このセクションでは、次のアセットを対象とするいくつかのアセット ジョブを作成する方法を説明します:

<CodeExample path="docs_snippets/docs_snippets/guides/data-modeling/asset-jobs/asset-jobs.py" language="python" startAfter="start_marker_assets" endBefore="end_marker_assets" />

アセットジョブを作成するには、[`define_asset_job`](/api/python-api/assets#dagster.define_asset_job) メソッドを使用します。アセットベースのジョブは、ジョブの対象となるアセットとその依存関係に基づいて作成されます。

1つまたは複数のアセットをターゲットにすることも、重複するアセットセットをターゲットとする複数のジョブを作成することもできます。
次の例では、2つのジョブがあります。

- `all_assets_job` はすべてのアセットをターゲットにします。
- `sugary_cereals_job` は `sugary_cereals` アセットのみをターゲットにします。

<CodeExample path="docs_snippets/docs_snippets/guides/data-modeling/asset-jobs/asset-jobs.py" language="python" startAfter="start_marker_jobs" endBefore="end_marker_jobs" />

## アセットジョブを Dagster ツールで利用できるようにする

Python モジュールまたはファイルの最上位レベルにある [`Definitions`](/api/python-api/definitions) オブジェクトにジョブを含めると、UI、GraphQL、コマンドラインでアセットジョブを利用できるようになります。
Dagster ツールは、そのモジュールをコードの場所として読み込みます。スケジュールまたはセンサーを含めると、[コードの場所](/guides/deploy/code-locations) には、それらのスケジュールまたはセンサーが対象とするジョブが自動的に含まれます。

<CodeExample path="docs_snippets/docs_snippets/concepts/assets/jobs_to_definitions.py" />

## アセットジョブのテスト

Dagster には、ビジネスロジックを環境から分離したり、制御不能な入力に対する明確な期待値の設定など、テストのためのサポートが組み込まれています。
詳細については、[テストに関するドキュメント](/guides/test) をご覧ください。

## アセットジョブの実行

アセットジョブは、以下の様々な方法で実行できます。

- 定義された Python プロセス内
- コマンドライン経由
- GraphQL API 経由
- UI 経由

## 例

[Hacker News のサンプル](https://github.com/dagster-io/dagster/tree/master/examples/project_fully_featured)は、[アセットグループをターゲットとするアセットジョブを構築します](https://github.com/dagster-io/dagster/blob/master/examples/project_fully_featured/project_fully_featured/jobs.py)。
