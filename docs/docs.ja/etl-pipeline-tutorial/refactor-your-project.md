---
title: プロジェクトをリファクタリングする
description: Refactor your completed project into a structure that is more organized and scalable. 
last_update:
  author: Alex Noonan
sidebar_position: 70
---

多くのエンジニアは、期待通りに動作するようになったら、通常はそのままにしておきます。しかし、何かを初めて実行したときにユースケースが最適に実装されることはめったになく、すべてのプロジェクトは段階的な改善から恩恵を受けます。

## Splitting up project structure

現在、プロジェクトは 1 つの定義ファイルに含まれています。ただし、このファイルはかなり複雑になっており、追加すると複雑さが増すだけです。これを修正するには、コアとなる Dagster コンセプトごとに個別のファイルを作成します:

- Assets
- Schedules
- Sensors
- Partitions

最終的なプロジェクト構造は次のようになります:

```
dagster-etl-tutorial/
├── data/
│   └── products.csv
│   └── sales_data.csv
│   └── sales_reps.csv
│   └── sample_request/
│       └── request.json
├── etl_tutorial/
│   └── assets.py
│   └── definitions.py
│   └── partitions.py
│   └── schedules.py
│   └── sensors.py
├── pyproject.toml
├── setup.cfg
├── setup.py
```

### Assets

アセットはプロジェクトの大部分を占めており、これが最大のファイルになります。

<CodeExample path="docs_beta_snippets/docs_beta_snippets/guides/tutorials/etl_tutorial_completed/etl_tutorial/assets.py" language="python"/>

### Partitions

パーティション ファイルには、`monthly_partition` と `product_category_partition` が含まれます。

<CodeExample path="docs_beta_snippets/docs_beta_snippets/guides/tutorials/etl_tutorial_completed/etl_tutorial/partitions.py" language="python" />

### Schedules

スケジュール ファイルには `weekly_update_schedule` のみが含まれます。

<CodeExample path="docs_beta_snippets/docs_beta_snippets/guides/tutorials/etl_tutorial_completed/etl_tutorial/schedules.py" language="python" />

### Sensors

センサー ファイルには、`adhoc_request` アセットのジョブとセンサーが含まれます。

<CodeExample path="docs_beta_snippets/docs_beta_snippets/guides/tutorials/etl_tutorial_completed/etl_tutorial/sensors.py" language="python" />

## 定義オブジェクトのリファクタリング

個別のファイルができたので、さまざまな要素を Definitions オブジェクトに追加する方法を調整する必要があります。

:::note
Dagster プロジェクトはルート ディレクトリから実行されるため、プロジェクト内のファイルを参照するときは常に、ルートを開始点として使用する必要があります。
さらに、Dagster には、モジュールからすべてのアセットとアセット チェックをロードする関数 (それぞれ load_assets_from_modules と load_asset_checks_from_modules) があります。
:::

プロジェクトをまとめるには、次のコードを `definitions.py` ファイルにコピーします:

<CodeExample path="docs_beta_snippets/docs_beta_snippets/guides/tutorials/etl_tutorial_completed/etl_tutorial/definitions.py" language="python" />

## 迅速な検証

定義ファイルが読み込まれて検証されることを確認するには、`dagster dev` を実行するのと同じディレクトリで `dagster definitions validate` を実行します。このコマンドは CI/CD パイプラインに役立ち、Web サーバーを起動せずにプロジェクトが正しく読み込まれることを確認できます。 

## おしまい！

おめでとうございます。Dagster を使用した最初のプロジェクトが完了し、ビルディングブロックを使用して独自のデータ パイプラインを構築する方法の例がわかりました。

## 推奨される次のステップ

- [Slack コミュニティ](https://dagster.io/slack)に参加してください。
- [Dagster University](https://courses.dagster.io/) コースで学習を続けましょう。
- 独自のプロジェクトで [Dagster+ の無料トライアル](https://dagster.cloud/signup) を開始してください。
