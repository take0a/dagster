---
title: APIへの接続
sidebar_position: 500
---

データ パイプラインを構築する場合、それぞれ独自の構成と動作を持つ複数の外部 API に接続する必要がある可能性があります。このガイドでは、Dagster リソースを使用して API 接続を標準化し、その構成をカスタマイズする方法を説明します。

:::note

このガイドでは、[アセット](/guides/build/assets/)と[リソース](/guides/build/external-resources/)に精通していることを前提としています。

:::

<details>
  <summary>前提条件</summary>

この記事のサンプル コードを実行するには、`requests` ライブラリをインストールする必要があります:

    ```bash
    pip install requests
    ```

</details>

## Step 1: APIに接続するリソースを作成する

この例では、REST API から指定された場所の日の出時刻を取得します。

`ConfigurableResource` を使用して、場所の日の出時刻を返すメソッドを持つ Dagster リソースを定義します。このリソースの最初のバージョンでは、場所はサンフランシスコ国際空港にハードコードされています。

<CodeExample path="docs_snippets/docs_snippets/guides/external-systems/apis/minimal_resource.py" language="python" />

## Step 2: アセット内のリソースを使用する

リソースを使用するには、それをアセットのパラメータとして提供し、`Definitions` オブジェクトに含めます。

<CodeExample path="docs_snippets/docs_snippets/guides/external-systems/apis/use_minimal_resource_in_asset.py" language="python" />

`sfo_sunrise` を具体化すると、Dagster は初期化された `SunResource` を `sun_resource` パラメータに提供します。

## Step 3: リソースを構成する

多くの API には、使用方法をカスタマイズするために設定できる構成があります。次の例では、クエリの場所を設定できるように構成でリソースを更新します:

<CodeExample path="docs_snippets/docs_snippets/guides/external-systems/apis/use_configurable_resource_in_asset.py" language="python" />

構成可能なリソースは、これまでとまったく同じようにアセットに提供できます。リソースが初期化されると、各構成オプションの値を渡すことができます。

`sfo_sunrise` を具体化すると、Dagster は `sun_resource` パラメータの設定値で初期化された `SunResource` を提供します。

## Step 4: 環境変数を使用したソース構成

リソースは環境変数を使用して設定することもできます。Dagster の組み込み `EnvVar` クラスを使用して、マテリアライズ時に環境変数から設定値を取得できます。

この例では、新しい `home_sunrise` アセットがあります。自宅の場所をハードコーディングするのではなく、環境変数に設定し、その値を読み取って `SunResource` を構成することができます。

<CodeExample path="docs_snippets/docs_snippets/guides/external-systems/apis/env_var_configuration.py" language="python" />

`home_sunrise` を具体化すると、Dagster は `HOME_LATITUDE`、`HOME_LONGITUDE`、および `HOME_TIMZONE` 環境変数に設定された値を読み取り、それらの値で `SunResource` を初期化します。

初期化された `SunResource` は `sun_resource` パラメータに提供されます。

:::note
`os` ライブラリを使用して環境変数を取得することもできます。Dagster は、環境変数を取得するタイミングや UI での表示方法など、環境変数を取得する各アプローチを異なる方法で処理します。詳細については、[環境変数ガイド](/guides/deploy/using-environment-variables-and-secrets)を参照してください。
:::

