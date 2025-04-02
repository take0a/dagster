---
title: 'Scaffolding Dagster definitions'
sidebar_position: 200
---

import Preview from '@site/docs.ja/partials/\_Preview.md';

<Preview />

`dg` は、[assets](/guides/build/assets/)、[schedules](/guides/automate/schedules/)、[sensors](/guides/automate/sensors/) などの Dagster 定義のスキャフォールディングに役立ちます。`dg` を使用してスキャフォールディングされたプロジェクトを使用すると、`defs` ディレクトリの下に追加された新しい定義は、トップレベルの `Definitions` オブジェクトに自動的にロードされます。これにより、トップレベルの定義ファイルにこれらの定義を明示的にインポートしなくても、プロジェクトに新しい定義を簡単に追加できます。

このガイドでは、`dg` を使用して新しいアセットをスキャフォールディングする方法について説明します。

## アセットの足場

`dg scaffold` コマンドを使用して、`defs` フォルダの下に新しいアセットをスキャフォールディングできます。この例では、`my_asset.py` という名前のアセットをスキャフォールディングし、それを `defs/assets` ディレクトリに書き込みます:

<CliInvocationExample path="docs_snippets/docs_snippets/guides/dg/dagster-definitions/1-scaffold.txt" />

これが完了すると、この場所に新しいファイルが追加され、その内容を表示できるようになります:

<CliInvocationExample path="docs_snippets/docs_snippets/guides/dg/dagster-definitions/2-tree.txt" />
<CliInvocationExample path="docs_snippets/docs_snippets/guides/dg/dagster-definitions/3-cat.txt" />

### 定義を書く

上記の例でわかるように、スキャフォールディングされたアセットには基本的なコメントアウトされた定義が含まれています。この定義を、必要なアセット コードに置き換えることができます:

<CliInvocationExample path="docs_snippets/docs_snippets/guides/dg/dagster-definitions/4-written-asset.py" />

### 作業を確認する

最後に、`dg list defs` を実行して、新しいアセットが定義リストに表示されていることを確認できます:

<CliInvocationExample path="docs_snippets/docs_snippets/guides/dg/dagster-definitions/5-list-defs.txt" />
