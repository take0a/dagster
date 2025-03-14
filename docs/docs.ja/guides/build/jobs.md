---
title: "Jobs"
sidebar_position: 200
---

ジョブは、Dagster での実行と監視のメイン ユニットです。ジョブを使用すると、スケジュールまたは外部トリガーに基づいて、[アセット定義](/guides/build/assets/defining-assets) または [ops](/guides/build/ops) のグラフの一部を実行できます。

ジョブが開始されると、_実行_ が開始されます。実行とは、Dagster でのジョブの 1 回の実行です。実行は Dagster UI で起動および表示できます。

## 利点

ジョブでは、以下が可能です:

* Dagster UI でジョブの実行を表示および開始する
* [スケジュール](/guides/automate/schedules/)と[センサー](/guides/automate/sensors/)を使用して、Dagster パイプラインの実行を自動化します。
* [メタデータとタグ](/guides/build/assets/metadata-and-tags)を使用して情報を添付する
* [実行キュー](/guides/deploy/execution/run-coordinators)を使用している場合は、ジョブ実行の優先順位付けと実行方法にカスタム優先順位付けルールを適用します。
* ジョブ実行に同時実行制限を適用すると、パイプラインの効率が向上します。詳細については、「[Dagster アセット、ジョブ、および Dagster インスタンスの同時実行の管理](/guides/operate/managing-concurrency)」を参照してください。
