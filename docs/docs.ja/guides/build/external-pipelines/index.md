---
title: 外部パイプライン (Dagster パイプ)
sidebar_position: 60
---

Dagster パイプは、ネイティブ Dagster パイプラインのスケジュール設定、レポート、および監視機能の利点をすべて提供しながら、Dagster 外部のコードを呼び出すための強力なメカニズムを提供します。Dagster は Python で記述されていますが、他の言語でコードを実行し、情報を Dagster に送り返すことができます。

このガイドでは、パイプを通じて Dagster 以外のコードを呼び出す方法について説明します。

:::note

このドキュメントは、[Dagster アセット](/guides/build/assets) に精通していることを前提としています。

:::

## 外部コードを呼び出すアセットの設定

Dagster 外部でコードを呼び出すように設定するには、まずアセットを設定する必要があります。Dagster Pipes クライアント リソースを使用して、アセット関数内で外部コードを呼び出すことができます。

この外部コードは、Dagster について何も知る必要はありません。リモート マシン上で別の言語を実行するプロセスであってもかまいません。唯一の要件は、Python からトリガーできることです。

次の例では、外部コードは Dagster アセット内で呼び出す Python スクリプト内にあります。

<CodeExample path="docs_beta_snippets/docs_beta_snippets/guides/external-systems/pipes/external_code_opaque.py" language="python" title="/usr/bin/external_code.py" />
<CodeExample path="docs_beta_snippets/docs_beta_snippets/guides/external-systems/pipes/asset_wrapper.py" language="python" title="Asset invoking external compute using Dagster Pipes" />

UI またはセンサー/スケジュールから Dagster でこのアセットを実体化すると、その外部コードの実行が開始されます。

## 外部コードからログとメタデータを Dagster に送り返す

Dagster Pipes は、オプションでログとメタデータを Dagster に送り返すための外部コード用のプロトコルも確立します。このプロトコルの Python クライアントは、[`dagster-pipes`](/api/python-api/libraries/dagster-pipes) パッケージの一部として利用できます。ログとメタデータを Dagster に送り返すには、外部コード内に `PipesContext` オブジェクトを作成します:

<CodeExample path="docs_beta_snippets/docs_beta_snippets/guides/external-systems/pipes/external_code_data_passing.py" language="python" title="/usr/bin/external_code.py" />

`PipesContext` を使用して送り返されたログは、そのアセットのマテリアライゼーションの実行の構造化されたログに表示され、マテリアライゼーションのメタデータはアセット履歴に反映されます。
