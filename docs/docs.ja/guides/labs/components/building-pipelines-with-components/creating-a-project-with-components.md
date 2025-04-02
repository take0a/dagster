---
title: 'コンポーネントを使用してプロジェクトを作成する'
sidebar_position: 100
---

import Preview from '@site/docs.ja/partials/\_Preview.md';

<Preview />

[依存関係をインストール](/guides/labs/components#installation)したら、コンポーネント対応プロジェクトをスキャフォールディングできます。以下の例では、`jaffle-platform` というプロジェクトをスキャフォールディングします:

<CliInvocationExample path="docs_snippets/docs_snippets/guides/components/index/2-scaffold.txt" />

このコマンドはプロジェクトをビルドし、その中に新しい Python 仮想環境を初期化します。`dg` のデフォルトの環境管理動作を使用する場合、この仮想環境を自分でアクティブ化する必要はありません。

:::note

複数のコンポーネント対応プロジェクトを作成して管理するには、「[dg を使用した複数のプロジェクトの管理](/guides/labs/dg/multiple-projects)」を参照してください。各プロジェクトには、独自の `uv` 管理 Python 環境があります。

:::

## プロジェクト構造

`dg scaffold project <project-name>` を実行すると、かなり標準的な Python プロジェクト構造が作成されます:

<CliInvocationExample path="docs_snippets/docs_snippets/guides/components/index/3-tree.txt" />

以下のファイルとディレクトリが含まれています:

- Python パッケージ `jaffle_platform` -- 名前は、プロジェクトのルート ディレクトリ (`jaffle_platform`) を下線で区切ったものです。
- (空の) `jaffle_platform_tests` テスト パッケージ。
- `uv.lock` ファイル。
- `pyproject.toml` ファイル。

:::note

pyproject.toml のセクションと設定の詳細については、「[pyproject.toml 設定](/guides/labs/components/building-pipelines-with-components/pyproject-toml)」を参照してください。

:::

## 次のステップ

コンポーネントを使用してプロジェクトをスキャフォールディングした後、[さらにコンポーネントを追加](/guides/labs/components/building-pipelines-with-components/adding-components)してパイプラインを完成させることができます。
