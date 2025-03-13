---
title: "Dagster プロジェクトの構造"
sidebar_position: 200
---

:::note
新しい Dagster プロジェクトを作成する方法については、プロジェクト スキャフォールディング チュートリアルを参照してください。
:::

Dagster プロジェクトを構成する方法は多数あり、どこから始めればよいか判断が難しい場合があります。このガイドでは、Dagster プロジェクトを整理するための推奨事項について説明します。プロジェクトが拡大するにつれて、これらの推奨事項から逸脱してもかまいません。

## 初期のプロジェクト構造

Dagster コマンドラインツールを使用して最初にプロジェクトの枠組みを作ると、プロジェクトのルートに `assets.py` と `definitions.py` が作成されます。

```sh
$ dagster project scaffold --name example-dagster-project
```

```
.
├── README.md
├── example_dagster_project
│   ├── __init__.py
│   ├── assets.py
│   └── definitions.py
├── example_dagster_project_tests
│   ├── __init__.py
│   └── test_assets.py
├── pyproject.toml
├── setup.cfg
└── setup.py
```

これは、最初に始めるときには最適な構造ですが、アセット、ジョブ、リソース、センサー、ユーティリティコードをさらに導入し始めると、Python ファイルが大きくなりすぎて管理できなくなる場合があります。

## プロジェクトを再構築する

プロジェクトを構造化できるパラダイムはいくつかあります。これらの構造の 1 つを選択することは、多くの場合、個人の好みであり、あなたとチーム メンバーの作業方法によって左右されます。このガイドでは、考えられる 3 つのプロジェクト構造について概説します。

1. [Option 1: テクノロジーによって構造化](#option-1-テクノロジーによって構造化)
2. [Option 2: コンセプト別に構成](#option-2-コンセプト別に構成)


### Option 1: テクノロジーによって構造化

データエンジニアは、データパイプラインで使用される基盤となるテクノロジーを深く理解していることが多いです。そのため、プロジェクトをテクノロジー別に整理すると効果的です。これにより、エンジニアはコードベースを簡単にナビゲートし、特定のテクノロジーに関連するファイルを見つけることができます。

テクノロジーモジュール内でサブモジュールを作成して、コードをさらに整理することができます。

```
.
└── example_dagster_project/
    ├── dbt/
    │   ├── __init__.py
    │   ├── assets.py
    │   ├── resources.py
    │   └── definitions.py
    ├── dlt/
    │   ├── __init__.py
    │   ├── pipelines/
    │   │   ├── __init__.py
    │   │   ├── github.py
    │   │   └── hubspot.py
    │   ├── assets.py
    │   ├── resources.py
    │   └── definitions.py
    └── definitions.py
```

### Option 2: コンセプト別に構成

包括的なデータ処理の概念によって分類のレイヤーを導入することもできます。たとえば、ジョブが何らかの変換、データの取り込み、または処理操作を実行しているかどうかなどです。

これにより、使用されている基盤となるテクノロジーにそれほど精通していないエンジニアにも追加のコンテキストが提供されます。

```
.
└── example_dagster_project/
    ├── ingestion/
    │   └── dlt/
    │       ├── assets.py
    │       ├── resources.py
    │       └── definitions.py
    ├── transformation/
    │   ├── dbt/
    │   │   ├── assets.py
    │   │   ├── resources.py
    │   │   └── partitions.py
    │   │   └── definitions.py
    │   └── adhoc/
    │       ├── assets.py
    │       ├── resources.py
    │       └── definitions.py
    └── definitions.py
```

## 定義オブジェクトのマージ

複数の `Definitions` オブジェクトを定義することも可能で、多くの場合、プロジェクト内のサブモジュールごとに 1 つずつ定義します。これらの定義は、`Definitions.merge` メソッドを使用してプロジェクトのルートでマージできます。

このような構造の利点は、リソースやパーティションなどの依存関係を、対応する定義にスコープ設定できることです。

```py title="example-merge-definitions.py"
from dbt.definitions import dbt_definitions
from dlt.definitions import dlt_definitions


defs = Definitions.merge(
    dbt_definitions,
    dlt_definitions,
)
```

## 複数のコードの場所の設定

このガイドでは、単一のコードの場所内でプロジェクトを構成する方法について概説しましたが、Dagster では複数の場所にまたがるプロジェクトを構成することもできます。

ほとんどの場合、コードの場所は 1 つで十分です。便利なパターンでは、複数のコード場所を使用して競合する依存関係を分離し、各定義に独自のパッケージ要件とデプロイメント仕様を設定します。

1 つのプロジェクトに複数のコード ロケーションを含めるには、プロジェクトに構成ファイルを追加する必要があります:

- **Dagster+ を使用する場合**、プロジェクトのルートに `dagster_cloud.yaml` ファイルを追加します。
- **ローカルで開発する場合、またはインフラストラクチャにデプロイする場合**、プロジェクトのルートにworkspace.yaml ファイルを追加します。

## 外部プロジェクト

データ プラットフォームが進化するにつれて、Dagster を使用すると、dbt、Sling、Jupyter ノートブックなどの他のデータ ツールをオーケストレーションできるようになります。

これらのプロジェクトについては、Dagster プロジェクトの外部に保存することをお勧めします。以下の `dbt_project` の例を参照してください。

```
.
├── dbt_project/
│   ├── config/
│   │   └── profiles.yml
│   ├── dbt_project.yml
│   ├── macros/
│   │   ├── aggregate_actions.sql
│   │   └── generate_schema_name.sql
│   ├── models/
│   │   ├── activity_analytics/
│   │   │   ├── activity_daily_stats.sql
│   │   │   ├── comment_daily_stats.sql
│   │   │   └── story_daily_stats.sql
│   │   ├── schema.yml
│   │   └── sources.yml
│   └── tests/
│       └── assert_true.sql
└── example_dagster_project/
```

## 次は

- <PyObject section="definitions" module="dagster" object="Definitions.merge" /> API ドキュメントを調べる
