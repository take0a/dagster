---
title: UI でアセットを構成する
sidebar_position: 400
---

Dagster UI は、アセットを手動で実現したり、履歴データをバックフィルしたり、運用上の問題をデバッグしたり、その他の 1 回限りのタスクを実行したりするためによく使用されます。

アセットを具体化するときにパラメータを調整したい場合がよくありますが、これは Dagster のアセット構成システムで実現できます。

:::note

この記事では、[アセット](/guides/build/assets) と [Pydantic](https://docs.pydantic.dev/latest/) に精通していることを前提としています。

:::


## アセットを構成可能にする

アセットを構成可能にするには、まず Dagster <PyObject section="config" module="dagster" object="Config" /> クラスから継承する[実行構成スキーマ](/guides/operate/configuration/run-configuration)を定義します。

たとえば、アセットを実体化する計算のルックバック時間ウィンドウをチームが変更できるようにしたいとします:

<CodeExample path="docs_snippets/docs_snippets/guides/data-modeling/configuring-assets/config-schema.py" language="python" />

## Dagster UI を使用して構成を指定する

:::note

実行構成は、アセットに関連付けられた基礎となるコンピューティングである <PyObject section="ops" module="dagster" object="op" /> を参照します。

:::

UI の Launchpad を使用して実行を開始するときに、アセットのデフォルト構成をオーバーライドする実行構成ファイルを YAML または JSON として提供できます。

**Materialize** ボタンのあるページで、**options menu > Open launchpad** をクリックして Launchpad にアクセスします。

![Highlighted Open Launchpad option in the Materialize options menu of the Dagster UI](/images/guides/build/assets/configuring-assets-in-the-ui/open-launchpad.png)

これにより、Launchpad が開き、構成の土台を作って、その値をカスタマイズし、アセットを手動でマテリアライズできます。

![Dagster Launchpad that configures an asset to have a lookback window of 7 days](/images/guides/build/assets/configuring-assets-in-the-ui/look-back-7.png)

## 次は

- Dagster [アセット](/guides/build/assets/) について詳しく見る
- リソースを使用して外部の [API](/guides/build/external-resources/connecting-to-apis) および [データベース](/guides/build/external-resources/connecting-to-databases) に接続する
