---
title: 'dg で複数のプロジェクトを管理する'
sidebar_position: 300
---

import Preview from '@site/docs.ja/partials/\_Preview.md';

<Preview />

:::note

始めたばかりの場合は、複数のプロジェクトを含むワークスペースではなく、[単一のプロジェクトのスキャフォールディング](/guides/labs/dg/scaffolding-a-project)をお勧めします。

:::

複数のチームと共同作業する必要がある場合、または相互に分離する必要がある競合する依存関係を扱う場合は、それぞれ独自の Python 環境を持つ複数のプロジェクトを含むワークスペース ディレクトリをスキャフォールディングできます。

ワークスペース ディレクトリには、ワークスペース レベルの設定を含むルート `pyproject.toml` と、1 つ以上のプロジェクトを含む `projects` ディレクトリが含まれます。

:::note

ワークスペースでは、デフォルトでは Python 環境は定義されません。代わりに、Python 環境はプロジェクトごとに定義されます。

:::

## 新しいワークスペースと最初のプロジェクトを構築する

`project-1` という初期プロジェクトで新しいワークスペースをスキャフォールディングするには、`--workspace-name` オプションを指定して `dg init` を実行します。

<CliInvocationExample path="docs_snippets/docs_snippets/guides/dg/workspace/1-dg-init.txt" />

これにより、`project-1` を含む `projects` サブディレクトリを持つ `dagster-workspace` という新しいディレクトリが作成されます。また、このプロジェクト用に新しい `uv` 管理 Python 環境もセットアップされます。

### ワークスペース構造を確認する

新しいワークスペースの構造は次のとおりです:

<CliInvocationExample path="docs_snippets/docs_snippets/guides/dg/workspace/2-tree.txt" />

`workspace` フォルダの `pyproject.toml` ファイルには、このディレクトリをワークスペースとしてマークする `is_workspace` 設定が含まれています。

<CodeExample
  path="docs_snippets/docs_snippets/guides/dg/workspace/3-pyproject.toml"
  language="TOML"
  title="workspace/pyproject.toml"
/>

:::note

`project-1` には、上記には示されていない `.venv` という仮想環境ディレクトリも含まれています。この環境は `uv` によって管理され、その内容は `uv.lock` ファイルで指定されます。

:::

`project-1` ディレクトリには、それを Dagster プロジェクトとして定義する `pyproject.toml` ファイルが含まれています。

<CodeExample
  path="docs_snippets/docs_snippets/guides/dg/workspace/4-project-pyproject.toml"
  language="TOML"
  title="workspace/projects/project-1/pyproject.toml"
/>

## ワークスペースに2番目のプロジェクトを追加する

上で述べたように、環境はプロジェクトごとにスコープが設定されます。`dg` コマンドは、`project-1` ディレクトリ内にいる場合にのみ `project-1` の環境を使用します。

別のプロジェクトを作成しましょう:

<CliInvocationExample path="docs_snippets/docs_snippets/guides/dg/workspace/5-scaffold-project.txt" />

これで 2 つのプロジェクトができました。次のようにリストできます:

<CliInvocationExample path="docs_snippets/docs_snippets/guides/dg/workspace/6-project-list.txt" />

## `dg` でワークスペースをロードする

最後に、`dg dev` で 2 つのプロジェクトをロードしましょう。`dg dev` はワークスペース内のプロジェクトを自動的に認識し、それぞれの環境で起動します。ワークスペースのルート ディレクトリで `dg dev` を実行し、ブラウザーで Dagster UI をロードしましょう:

<CliInvocationExample contents="cd ../.. && dg dev" />

![](/images/guides/build/projects-and-components/setting-up-a-workspace/two-projects.png)
