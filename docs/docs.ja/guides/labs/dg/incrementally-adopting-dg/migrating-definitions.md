---
title: '既存の Dagster 定義の自動読み込み'
sidebar_position: 100
---

import Preview from '@site/docs.ja/partials/\_Preview.md';

<Preview />

:::note

このガイドでは、`dg` 互換プロジェクトで既存の Dagster 定義を使用する方法について説明します。既存のプロジェクトを `dg` を使用するように変換するには、「[既存のプロジェクトを `dg` を使用するように変換する](/guides/labs/dg/incrementally-adopting-dg/migrating-project)」を参照してください。

:::

`dg` を多用するプロジェクトでは、通常、すべての定義を `defs/` ディレクトリに保存します。ただし、既存のプロジェクトを `dg` を使用するように変換した場合、さまざまな他のモジュールに定義が配置されている可能性があります。このガイドでは、これらの既存の定義を `defs` ディレクトリに移動して、自動的に読み込まれるようにする方法を説明します。

## プロジェクト構造の例

次の構造を持つプロジェクトを使用して、既存の定義を移行する例を見てみましょう:

<CliInvocationExample path="docs_snippets/docs_snippets/guides/dg/migrating-definitions/1-tree.txt" />

最上位レベルでは、さまざまなモジュールから定義を読み込みます:

<CodeExample
  path="docs_snippets/docs_snippets/guides/dg/migrating-definitions/2-definitions-before.py"
  startAfter="start"
  title="my_existing_project/definitions.py"
/>

これらの各モジュールには、アセット、ジョブ、スケジュールなど、さまざまな Dagster 定義が含まれています。

`elt` モジュールをコンポーネントに移行してみましょう。

## 定義を `defs` に移動する

まず、トップレベルの `elt` モジュールを `defs/elt` に移動します:

<CliInvocationExample path="docs_snippets/docs_snippets/guides/dg/migrating-definitions/3-mv.txt" />

定義が `defs` ディレクトリにあるので、ルート `definitions.py` ファイルを更新して、`elt` モジュールの `Definitions` を明示的にロードしないようにすることができます:

<CodeExample
  path="docs_snippets/docs_snippets/guides/dg/migrating-definitions/4-definitions-after.py"
  title="my_existing_project/definitions.py"
/>

プロジェクト構造は次のようになります:

<CliInvocationExample path="docs_snippets/docs_snippets/guides/dg/migrating-definitions/5-tree-after.txt" />

`definitions.py` ファイルの `load_defs` コマンドは、`defs` モジュール内にある定義を自動的にロードします。つまり、すべての定義が自動的にロードされるようになり、トップレベルの組織スキームにインポートする必要がなくなりました。

他のモジュールについても同じプロセスを繰り返すことができます。

## 完全に移行されたプロジェクト構造

各定義モジュールが移行されると、プロジェクトには標準化された構造が残ります:

<CliInvocationExample path="docs_snippets/docs_snippets/guides/dg/migrating-definitions/6-tree-after-all.txt" />

プロジェクト ルートは、`defs` モジュールからの定義のみを構築するようになりました:

<CodeExample
  path="docs_snippets/docs_snippets/guides/dg/migrating-definitions/7-definitions-after-all.py"
  title="my_existing_project/definitions.py"
/>

すべての定義が正しくロードされていることを確認するには、`dg list defs` を実行します:

<CodeExample
  path="docs_snippets/docs_snippets/guides/dg/migrating-definitions/8-list-defs-after-all.txt"
  title="my_existing_project/definitions.py"
/>

## Next steps

- [プロジェクトに新しい定義を追加する](/guides/labs/dg/dagster-definitions)
