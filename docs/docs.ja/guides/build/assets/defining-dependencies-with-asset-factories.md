---
title: 'アセットファクトリーによる依存関係の定義'
sidebar_position: 600
---

データエンジニアリングでは、コードを再利用して類似のアセットを定義すると便利なことがよくあります。たとえば、ディレクトリ内のすべてのファイルをアセットとして表したい場合があります。

さらに、Python や Dagster に慣れていない利害関係者にサービスを提供する場合もあります。そのような利害関係者は、YAML などの構成言語上に構築されたドメイン固有言語 (DSL) を使用してアセットを操作することを好む場合があります。

アセット ファクトリを使用すると複雑さが軽減され、追加のアセットを定義するためのプラグ可能なエントリ ポイントが作成されます。

:::note

このガイドでは、[アセットファクトリ](/guides/build/assets/creating-asset-factories) に精通していることを前提としています。
:::

---

## Python でアセットファクトリーを構築する

多数のテーブルを管理するデータ分析チームを想像してください。分析のニーズをサポートするために、チームはクエリを実行し、結果から新しいテーブルを構築します。

各テーブルは、名前、アップストリーム アセットの依存関係、およびクエリによって YAML で表すことができます。

<CodeExample path="docs_snippets/docs_snippets/guides/data-modeling/asset-factories-with-deps/table_definitions.yaml" language="yaml" title="YAML Definition for ETL tables" />

Dagster でこれらのアセットを定義するために Python ロジックを追加する方法は次のとおりです。

<CodeExample path="docs_snippets/docs_snippets/guides/data-modeling/asset-factories-with-deps/asset-factory-with-deps.py" language="python" title="Programmatically defining asset dependencies" />

## ファクトリーアセットと通常のアセット間の依存関係を定義する

ファクトリアセットの下流に Dagster アセットを定義する Python ロジックを追加する方法は次のとおりです。

<CodeExample path="docs_snippets/docs_snippets/guides/data-modeling/asset-factories-with-deps/asset_downstream_of_factory_assets.py" language="python" title="Defining dependencies between factory assets and regular assets" />
