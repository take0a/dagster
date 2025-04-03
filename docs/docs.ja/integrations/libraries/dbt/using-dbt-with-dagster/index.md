---
title: 'Dagster で dbt を使用する'
description: Dagster can orchestrate dbt alongside other technologies.
---

:::note

dbt Cloud をお使いですか? [Dagster & dbt Cloud ガイド](/integrations/libraries/dbt/dbt-cloud) をご覧ください。

:::

このチュートリアルでは、dbt のサンプル [jaffle shop プロジェクト](https://github.com/dbt-labs/jaffle_shop) の小規模バージョン、[dagster-dbt ライブラリ](/api/python-api/libraries/dagster-dbt)、および [DuckDB](https://duckdb.org/) などのデータ ウェアハウスを使用して、dbt と Dagster を統合する手順を説明します。

このチュートリアルの最後には、dbt モデルが、その上流および下流にある他の [Dagster アセット定義](/integrations/libraries/dbt/reference#dbt-models-and-dagster-asset-definitions) とともに Dagster で表現されるようになります:

![Asset group with dbt models and Python asset](/images/integrations/dbt/using-dbt-with-dagster/downstream-assets/asset-graph-materialized.png)

そこに到達するには、次の操作を行います:

- [dbt プロジェクトをセットアップ](/integrations/libraries/dbt/using-dbt-with-dagster/set-up-dbt-project)
- [dbt モデルをアセットとして Dagster にロード](/integrations/libraries/dbt/using-dbt-with-dagster/load-dbt-models)
- [上流の Dagster アセットを作成して具体化](/integrations/libraries/dbt/using-dbt-with-dagster/upstream-assets)
- plotly チャートを出力する[下流のアセットを作成して具体化](/integrations/libraries/dbt/using-dbt-with-dagster/downstream-assets)

## 前提条件

このチュートリアルを完了するには、次のものが必要です:

- **[git](https://en.wikipedia.org/wiki/Git) がインストールされている**。まだインストールされていない場合 (ターミナルで `git` と入力して確認してください)、[git Web サイトの手順](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git) を使用してインストールできます。

- **dbt、Dagster、および Dagster Web サーバー/UI をインストールする**。以下を実行して、pip を使用してすべてをインストールします:

  ```shell
  pip install dagster-dbt dagster-webserver dbt-duckdb
  ```

  `dagster-dbt` ライブラリは、`dbt-core` と `dagster` の両方を依存関係としてインストールします。このチュートリアルでは [DuckDB](https://duckdb.org/) をデータベースとして使用するため、`dbt-duckdb` がインストールされます。詳細については、[dbt](https://docs.getdbt.com/dbt-cli/install/overview) および [Dagster](/getting-started/installation) のインストール ドキュメントを参照してください。

## 始める準備はできましたか？

チュートリアルの前提条件をすべて満たしたら、[dbt プロジェクトの設定](/integrations/libraries/dbt/using-dbt-with-dagster/set-up-dbt-project)から開始できます。
