---
title: 'pyproject.toml 設定'
sidebar_position: 500
---

import Preview from '@site/docs.ja/partials/\_Preview.md';

<Preview />

`pyproject.toml` には `tool.dagster` セクションと `tool.dg` セクションが含まれています:

<CodeExample
  path="docs_snippets/docs_snippets/guides/components/index/4-pyproject.toml"
  language="TOML"
  title="jaffle-platform/pyproject.toml"
/>

### `tool.dagster` セクション

`pyproject.toml` の `tool.dagster` セクションは `dg` 固有ではありません。このセクションでは、一連の定義を `jaffle_platform.definitions` モジュールからロードできることを指定します。

### `tool.dg` セクション

`tool.dg` セクションには `is_project` と `is_component_lib` の設定が含まれています。

#### `is_project` 設定

`is_project = true` は、このプロジェクトが `dg` 管理の Dagster プロジェクトであることを指定します。コンポーネントを使用して作成されたプロジェクトは、特定の構造を持つ通常の Dagster プロジェクトです。

構造を理解するために、`jaffle_platform/definitions.py` の内容を見てみましょう:

<CodeExample
  path="docs_snippets/docs_snippets/guides/components/index/5-definitions.py"
  language="Python"
  title="jaffle-platform/jaffle_platform/definitions.py"
/>

この `load_defs` の呼び出しにより、次の処理が行われます:

- プロジェクトで定義されているコンポーネントのセットを検出します
- 各コンポーネントから `Definitions` のセットを計算します
- コンポーネント固有の定義を 1 つの `Definitions` オブジェクトにマージします

`is_project` は、プロジェクトがこの方法で構造化されているため、コンポーネント インスタンスが含まれていることを `dg` に伝えます。現在のプロジェクトでは、コンポーネント インスタンスは `jaffle_platform/components` のデフォルトの場所に配置されます。

#### `is_component_lib` 設定

`is_component_lib = true` は、プロジェクトがコンポーネント ライブラリであることを指定します。つまり、プロジェクトには、コンポーネント インスタンスを生成するときに参照できるコンポーネント タイプが含まれている可能性があります。

一般的なプロジェクトでは、ほとんどのコンポーネントは外部ライブラリ (`dagster-dbt` など) で定義されたタイプのインスタンスである可能性が高いですが、プロジェクトにスコープを設定したカスタム コンポーネント タイプを定義することもできます。そのため、`is_component_lib` はデフォルトで `true` に設定されています。`jaffle_platform` 内のスキャフォールディングされたコンポーネント タイプはすべて、`jaffle_platform/lib` のデフォルトの場所に配置されます。

また、このモジュールは `pyproject.toml` の `dagster_dg.library` エントリ ポイントに登録されていることもわかります。これにより、コンポーネントが `dg` で検出可能になります:

<CodeExample
  path="docs_snippets/docs_snippets/guides/components/index/6-pyproject.toml"
  language="TOML"
  title="jaffle-platform/pyproject.toml"
/>
