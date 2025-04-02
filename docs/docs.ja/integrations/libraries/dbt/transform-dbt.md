---
title: はじめる
sidebar_position: 200
---

Dagster は dbt を他のテクノロジーと一緒にオーケストレーションするため、単一のデータ パイプラインで Spark、Python などを使用して dbt をスケジュールできます。Dagster のアセット指向のアプローチにより、Dagster は個々の dbt モデルのレベルで dbt を理解できます。

<details>
  <summary>前提条件</summary>

このガイドの手順を実行するには、次のものが必要です:

- [assets](/guides/build/assets/) や [resources](/guides/build/external-resources/) などの dbt、DuckDB、および Dagster の概念に関する基本的な理解
- [dbt](https://docs.getdbt.com/docs/core/installation-overview) および [DuckDB CLI](https://duckdb.org/docs/api/cli/overview.html) をインストールする
- 次のパッケージをインストールする:

```shell
pip install dagster duckdb plotly pandas dagster-dbt dbt-duckdb
```

</details>

## 基本的な dbt プロジェクトの設定

まず、いくつかのモデルと DuckDB バックエンドを含むこの基本的な dbt プロジェクトをダウンロードします:

```bash
git clone https://github.com/dagster-io/basic-dbt-project
```

プロジェクト構造は次のようになります:

```
├── README.md
├── dbt_project.yml
├── profiles.yml
├── models
│   └── example
│       ├── my_first_dbt_model.sql
│       ├── my_second_dbt_model.sql
│       └── schema.yml
```

まず、Dagster を dbt プロジェクトに向け、Dagster にアセット グラフの構築に必要なものがあることを確認する必要があります。dbt プロジェクトと同じディレクトリに `definitions.py` を作成します:

<CodeExample
  path="docs_snippets/docs_snippets/guides/etl/transform-dbt/dbt_definitions.py"
  language="python"
  title="definitions.py"
/>

## 上流依存関係の追加

多くの場合、下流の dbt モデルで使用されるデータを Dagster で生成する必要があります。これを行うには、dbt プロジェクトがソースとして使用する上流アセットを追加します:

<CodeExample
  path="docs_snippets/docs_snippets/guides/etl/transform-dbt/dbt_definitions_with_upstream.py"
  language="python"
  title="definitions.py"
/>

次に、`raw_customers` アセットをソースし、Dagster の依存関係を定義する dbt モデルを追加します。dbt モデルを作成します:

<CodeExample
  path="docs_snippets/docs_snippets/guides/etl/transform-dbt/basic-dbt-project/models/example/customers.sql"
  language="sql"
  title="customers.sql"
/>

次に、dbt をアップストリームの `raw_customers` アセットにポイントする `_source.yml` ファイルを作成します:

<CodeExample
  path="docs_snippets/docs_snippets/guides/etl/transform-dbt/basic-dbt-project/models/example/_source.yml"
  language="yaml"
  title="_source.yml_"
/>

![Screenshot of dbt lineage](/images/integrations/dbt/dbt-lineage.png)

## 下流の依存関係の追加

dbt モデルの出力に依存するアセットもあるかもしれません。次に、新しい `customers` モデルの結果に依存するアセットを作成します。このアセットは、顧客のファーストネームのヒストグラムを作成します:

<CodeExample
  path="docs_snippets/docs_snippets/guides/etl/transform-dbt/dbt_definitions_with_downstream.py"
  language="python"
  title="definitions.py"
/>

## スケジュール dbt モデル

`dagster-dbt` の `build_schedule_from_dbt_selection` 関数を使用して、dbt モデルをスケジュールできます:

<CodeExample
  path="docs_snippets/docs_snippets/guides/etl/transform-dbt/dbt_definitions_with_schedule.py"
  language="python"
  title="Scheduling our dbt models"
/>
