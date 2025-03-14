---
title: "外部コードの変更"
description: "With Dagster Pipes, you can incorporate existing code into Dagster without huge refactors. This guide shows you how to modify existing code to work with Dagster Pipes."
sidebar_position: 200
---

:::note

これは、[Dagster パイプを使用してローカル サブプロセスを実行する](index.md) チュートリアルのパート 2 です。

:::

この時点で、次の 2 つのファイルがあるはずです:

- `external_code.py` は、Dagster でオーケストレーションするスタンドアロンの Python スクリプトです:
- Dagster アセットとその他の Dagster 定義を含む `dagster_code.py`。

このセクションでは、スタンドアロンの Python スクリプトを変更して [Dagster Pipes](/guides/build/external-pipelines/) と連携し、情報を Dagster にストリームバックする方法を学びます。これを行うには、次の操作を行います。

- [Dagster コンテキストを外部コードで利用できるようにする](#step-1-make-dagster-context-available-in-external-code)
- [ログメッセージを Dagster にストリームで戻す](#step-2-send-log-messages-to-dagster)
- [構造化メタデータを Dagster にストリームで戻す](#step-3-send-structured-metadata-to-dagster)

## Step 1: Dagster コンテキストを外部コードで利用できるようにする

外部コードから Dagster パイプ経由で Dagster に情報を送り返すには、数行のコードを追加する必要があります:

- `dagster-pipes` からインポートする

- Dagster Pipes に接続する呼び出し: <PyObject section="libraries" module="dagster_pipes" object="open_dagster_pipes"/> は、情報を Dagster にストリームするために使用できる Dagster Pipes コンテキストを初期化します。この関数は、パイプ セッションのエントリ ポイントの近くで呼び出すことをお勧めします。

  `with open_dagster_pipes()` は Python のコンテキスト マネージャーであり、特定のコード セグメントのリソースのセットアップとクリーンアップを保証します。これは、接続のオープンとクローズなど、初期セットアップと最終的なティアダウンを必要とするタスクに役立ちます。この場合、コンテキスト マネージャーは Dagster Pipes 接続の初期化とクローズに使用されます。

- <PyObject section="libraries" module="dagster_pipes" object="PipesContext.get" /> 経由の Dagster Pipes コンテキストのインスタンス。このコンテキスト オブジェクトを介して、`partition_key` や `asset_key` などの情報にアクセスできます。詳細については、[API ドキュメント](/api/python-api/libraries/dagster-pipes#dagster_pipes.PipesContext) を参照してください。

サンプルの Python スクリプトでは、変更は次のようになります:

<CodeExample path="docs_snippets/docs_snippets/guides/dagster/dagster_pipes/subprocess/part_2/step_1/external_code.py" lineStart="3" />

## Step 2: Dagster にログメッセージを送信する

Dagster Pipes コンテキストには、ログメッセージを Dagster にストリームで返すことができる組み込みのログ機能があります。標準出力に出力する代わりに、<PyObject section="libraries" module="dagster_pipes" object="PipesContext" /> の `​​context.log` メソッドを使用して、ログメッセージを Dagster に返すことができます。この場合、`info` レベルのログメッセージを送信しています:


<CodeExample path="docs_snippets/docs_snippets/guides/dagster/dagster_pipes/subprocess/part_2/step_2/external_code.py" />

そうすると、ログメッセージが Dagster UI の **Run details** ページに表示されます。ログレベルをフィルターして、`info` レベルのメッセージのみを表示できます。

1. ログフィルターフィールドの横にある **Levels** フィルターをクリックします。これにより、すべてのログレベルのドロップダウンが表示されます。
2. **info** チェックボックスをオンにし、他のチェックボックスをオフにします。これにより、**info** レベルとしてマークされたログのみが表示されます。

![Send log messages to Dagster](/images/guides/build/external-pipelines/subprocess/part-2-step-2-log-level.png)

## Step 3: 構造化メタデータをDagsterに送信する

場合によっては、外部コードからの情報を、Dagster UI に表示される構造化メタデータとしてログに記録したいことがあります。Dagster Pipes コンテキストには、構造化メタデータを Dagster にログに記録する機能も備わっています。

### アセットのマテリアライゼーションを報告する

[Dagster プロセス内でのマテリアライゼーション メタデータの報告](/guides/build/assets/metadata-and-tags/)と同様に、外部プロセスからアセットのマテリアライゼーションを Dagster に報告することもできます。

この例では、`total_orders` という名前のメタデータを <PyObject section="libraries" module="dagster_pipes" object="PipesContext" method="report_asset_materialization" /> の `​​metadata` パラメータに渡しています。このペイロードは外部プロセスから Dagster に送り返されます:


<CodeExample path="docs_snippets/docs_snippets/guides/dagster/dagster_pipes/subprocess/part_2/step_3_materialization/external_code.py" />

すると、`total_orders` が構造化メタデータとして UI に表示されます:

![Report asset materialization to Dagster](/images/guides/build/external-pipelines/subprocess/part-2-step-3-report-asset-materialization.png)

このメタデータは、UI の **Asset Details** ページの **Events** タブにも表示されます:

![View materialization events in asset details page](/images/guides/build/external-pipelines/subprocess/part-2-step-3-asset-details.png)

### アセットチェックを報告する

Dagster を使用すると、アセットのデータ品質チェックを定義して実行できます。詳細については、[アセット チェック](/guides/test/asset-checks) のドキュメントを参照してください。

アセットにデータ品質チェックが定義されている場合は、<PyObject section="libraries" module="dagster_pipes" object="PipesContext" method="report_asset_check" /> を介してアセット チェックが実行されたことを Dagster に報告できます:

<Tabs>
<TabItem value="外部コードから報告する">


<CodeExample path="docs_snippets/docs_snippets/guides/dagster/dagster_pipes/subprocess//part_2/step_3_check/external_code.py" />

</TabItem>
<TabItem value="Dagsterコードでアセットを定義する">

<CodeExample path="docs_snippets/docs_snippets/guides/dagster/dagster_pipes/subprocess/part_2/step_3_check/dagster_code.py" />

</TabItem>
</Tabs>

Dagster がコードを実行すると、UI にチェック結果を含むアセット チェック イベントが表示されます:

![Report asset checks to Dagster](/images/guides/build/external-pipelines/subprocess/part-2-step-3-report-asset-check.png)

このチェック結果は、UI の **Asset Details** ページの **Checks** タブにも表示されます:

![View checks in asset details page](/images/guides/build/external-pipelines/subprocess/part-2-step-3-check-tab.png)

## 完成したコード

この時点で、2 つのファイルは次のようになります:

<Tabs>
<TabItem value="external_code.py の外部コード">

<CodeExample path="docs_snippets/docs_snippets/guides/dagster/dagster_pipes/subprocess/part_2/step_3_check/external_code.py" />

</TabItem>
<TabItem value="dagster_code.py の Dagster コード">

<CodeExample path="docs_snippets/docs_snippets/guides/dagster/dagster_pipes/subprocess/part_2/step_3_check/dagster_code.py" />

</TabItem>
</Tabs>

## 次は？

このチュートリアルでは、Dagster Pipes コンテキストにアクセスし、外部プロセスからのログ メッセージ イベントを報告し、構造化されたイベントを Dagster に送り返す方法を学習しました。

次は何をしますか? ここから、次のことができます:

- [サブプロセスリファレンス](reference)で、Dagster Pipes を介してサブプロセスで外部コードを実行するその他の機能について学習します。
- [独自の Dagster Pipes プロトコルをカスタマイズする](/guides/build/external-pipelines/dagster-pipes-details-and-customization) 方法を学びます
- [Dagster Pipes とのその他の統合](/guides/build/external-pipelines/) について学ぶ
