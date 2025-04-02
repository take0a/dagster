---
title: 'Pythonでプロジェクトにコンポーネントを追加する'
sidebar_position: 300
---

import Preview from '@site/docs.ja/partials/\_Preview.md';

<Preview />

場合によっては、`component.yaml` ファイルではなく、Python を使用してプロジェクトにコンポーネントを追加したい場合があります。

:::note Prerequisites

Python を使用してコンポーネントを追加する前に、[コンポーネントを含むプロジェクトを作成する](/guides/labs/components/building-pipelines-with-components/creating-a-project-with-components)か、[既存のプロジェクトを `dg` に移行する](/guides/labs/dg/incrementally-adopting-dg/migrating-project)必要があります。

:::

1. まず、`components/` ディレクトリにコンポーネント定義を格納する新しいサブディレクトリを作成します。
2. サブディレクトリに、コンポーネント インスタンスを定義する `component.py` ファイルを作成します。このファイルでは、対象のコンポーネント タイプをインスタンス化する単一の `@component` で装飾された関数を定義します:

<CodeExample path="docs_snippets/docs_snippets/guides/components/python-components/component.py" language="python" />

この関数は、必要なコンポーネント タイプのインスタンスを返す必要があります。上記の例では、この機能を使用して、`DbtProjectcomponent` クラスの `translator` 引数をカスタマイズしました。
