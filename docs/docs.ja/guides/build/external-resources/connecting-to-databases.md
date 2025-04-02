---
title: データベースへの接続
sidebar_position: 400
---

データ パイプラインを構築する場合、データベースからデータを抽出したり、データベースにデータをロードしたりする必要がある場合があります。Dagster では、リソースはデータベース クライアントのラッパーとして動作することで、データベースに接続するために使用できます。

このガイドでは、Dagster リソースを使用してデータベース接続を標準化し、その構成をカスタマイズする方法を説明します。

:::note

このガイドでは、[assets](/guides/build/assets/) に精通していることを前提としています。

:::

<details>
  <summary>前提条件</summary>

この記事のサンプル コードを実行するには、次のものが必要です:

- Snowflakeデータベースの接続情報
- 以下をインストールするには:

   ```bash
   pip install dagster dagster-snowflake pandas
   ```

</details>

## Step 1: リソースを書く \{#step-one}

この例では、Snowflake データベースを表すリソースを作成します。`SnowflakeResource` を使用して、Snowflake データベースに接続する Dagster リソースを定義します:

<CodeExample path="docs_snippets/docs_snippets/guides/external-systems/databases/snowflake-resource.py" language="python" />

## Step 2: アセット内のリソースを使用する \{#step-two}

リソースを使用するには、それをアセットのパラメータとして提供し、`Definitions` オブジェクトに含めます。

<CodeExample path="docs_snippets/docs_snippets/guides/external-systems/databases/use-in-asset.py" language="python" />

これらのアセットを具体化すると、Dagster は初期化された `SnowflakeResource` をアセットの `iris_db` パラメータに提供します。

## Step 3: 環境変数を使用したソース構成 \{#step-three}

リソースは環境変数を使用して構成できるため、環境固有のデータベースに接続したり、資格情報を交換したりすることができます。Dagster の組み込み `EnvVar` クラスを使用して、アセットの実現時に環境変数から構成値を取得できます。

この例では、`production` という名前の Snowflake リソースの 2 番目のインスタンスが追加されています:

<CodeExample path="docs_snippets/docs_snippets/guides/external-systems/databases/use-envvars.py" language="python" />

アセットが実体化されると、Dagster は `deployment_name` 環境変数を使用して、使用する Snowflake リソース (`local` または `production`) を決定します。次に、Dagster は各リソースの環境変数に設定された値 (例: `DEV_SNOWFLAKE_PASSWORD`) を読み取り、それらの値で `SnowflakeResource` を初期化します。

初期化された `SnowflakeResource` は、アセットの `iris_db` パラメータに提供されます。

:::note
`os` ライブラリを使用して環境変数を取得することもできます。Dagster は、環境変数を取得するタイミングや UI での表示方法など、環境変数を取得する各アプローチを異なる方法で処理します。詳細については、[環境変数ガイド](/guides/deploy/using-environment-variables-and-secrets)を参照してください。
:::

## 次は

- [API への接続](/guides/build/external-resources/connecting-to-apis) のリソースの使用方法を調べる
- [リソース](/guides/build/external-resources/) の理解を深める