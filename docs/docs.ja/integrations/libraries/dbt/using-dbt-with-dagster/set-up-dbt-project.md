---
title: 'dbtプロジェクトを設定する'
description: Dagster can orchestrate dbt alongside other technologies.
sidebar_position: 100
---

チュートリアルのこの部分では、次の操作を行います:

- [dbt プロジェクトをダウンロード](#step-1-download-the-sample-dbt-project)
- [dbt プロジェクトを DuckDB で実行するように構成](#step-2-configure-your-dbt-project-to-run-with-duckdb)
- [dbt プロジェクトをビルド](#step-3-build-your-dbt-project)

## Step 1: dbt プロジェクトをダウンロード {#step-1-download-the-sample-dbt-project}

まず、サンプルの dbt プロジェクトをダウンロードしましょう。標準の dbt [Jaffle Shop](https://github.com/dbt-labs/jaffle_shop) の例を使用します。

1. まず、最終的に dbt プロジェクトと Dagster コードの両方が含まれるフォルダーを作成します。

   ```shell
   mkdir tutorial-dbt-dagster
   ```

2. 次に、そのフォルダに移動します:

   ```shell
   cd tutorial-dbt-dagster
   ```

3. 最後に、サンプル dbt プロジェクトをそのフォルダーにダウンロードします。

   ```shell
   git clone https://github.com/dbt-labs/jaffle_shop.git
   ```

## Step 2: dbt プロジェクトを DuckDB で実行するように構成 {#step-2-configure-your-dbt-project-to-run-with-duckdb}

dbt を実行するには、dbt モデルから作成されたテーブルを保存するためのデータ ウェアハウスが必要です。データ ウェアハウスには DuckDB を使用します。これは、セットアップに長時間実行されるサービスや外部インフラストラクチャが不要なためです。

dbt [プロファイル](https://docs.getdbt.com/docs/core/connect-data-platform/connection-profiles) を構成して、dbt を DuckDB と連携するように設定します:

1. プロジェクトをダウンロードしたときに作成された、`tutorial-dbt-dagster` フォルダ内の `jaffle_shop` フォルダに移動します:

   ```shell
   cd jaffle_shop
   ```

2. このフォルダーで、任意のテキスト エディターを使用して `profiles.yml` という名前のファイルを作成し、次のコードを追加します:

   ```yaml
   jaffle_shop:
     target: dev
     outputs:
       dev:
         type: duckdb
         path: tutorial.duckdb
         threads: 24
   ```

## Step 3: dbt プロジェクトをビルド {#step-3-build-your-dbt-project}

上記のようにプロファイルを設定すると、dbt プロジェクトが使用可能になります。テストするには、次のコマンドを実行します:

```shell
dbt build
```

これにより、プロジェクト内のすべてのモデル、シード、スナップショットが実行され、DuckDB データベースに一連のテーブルが保存されます。

:::note

その他の dbt プロジェクトでは、プロジェクトをビルドする前に追加のコマンドを実行する必要があります。たとえば、[依存関係](https://docs.getdbt.com/docs/collaborate/govern/project-dependencies) のあるプロジェクトでは、プロジェクトをビルドする前に [`dbt deps`](https://docs.getdbt.com/reference/commands/deps) を実行する必要があります。詳細については、[公式の dbt コマンド リファレンス ページ](https://docs.getdbt.com/reference/dbt-commands) を参照してください。

:::note

## 次は？

この時点で、Dagster で使用できるように完全に構​​成された dbt プロジェクトが完成しているはずです。次の手順は、[dbt モデルをアセットとして Dagster にロード](/integrations/libraries/dbt/using-dbt-with-dagster/load-dbt-models) することです。
