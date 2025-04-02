---
title: '自動化'
description: Learn how to automate your data pipelines.
sidebar_class_name: hidden
---

自動化は、信頼性が高く効率的なデータ パイプラインを構築する上で重要です。Dagster は、さまざまなニーズに合わせてパイプラインの実行を自動化するさまざまな方法を提供します。

自動化方法を選択するときは、次の要素を考慮してください:

* **パイプライン構造**: 主に [アセット](/guides/build/assets/)、[ops](/guides/build/ops/)、またはその両方を使って作業していますか？
* **タイミング要件**: 定期的な更新やイベント駆動型処理が必要ですか？
* **データ特性**: データはパーティション分割されていますか? 履歴データを更新する必要がありますか？
* **システム統合**: 外部のイベントやシステムに反応する必要がありますか？

## 自動化の方法

| 方法                       | 説明                                | 最適な用途                     | 対応するオブジェクト                               |
| ---------------------------- | ------------------------------------------ | ---------------------------- | ---------------------------------------- |
| [Schedules](schedules/) | cron式を使用して、指定した時間に[選択したアセット](/guides/build/assets/asset-selection-syntax)を実行します。 | 定期的な時間ベースのジョブ実行と基本的な時間ベースの自動化 | Assets, Ops, Graphs |
| [Declarative automation](declarative-automation/) |  アセットとアセットチェックの自動化条件を設定できるフレームワーク | アセット中心、状態ベースの更新 | Assets only         |
| [Sensors](sensors/)     | トリガーは、新しいファイルの到着や外部システムの変更など、定義したイベントや条件に基づいて実行されます。 | イベント駆動型自動化                | Assets, Ops, Graphs |
| [Asset sensors](/guides/automate/asset-sensors) | 指定されたアセットが実現されたときにジョブをトリガーし、ジョブまたはコードの場所間の依存関係を作成できます。 | ジョブ/ロケーション間のアセット依存関係  | Assets only         |
| [GraphQL triggers](/guides/operate/graphql/) | GraphQLエンドポイントからマテリアライゼーションとジョブをトリガーする      | 外部システムからのイベントトリガー   | Assets, Ops, Jobs   |
