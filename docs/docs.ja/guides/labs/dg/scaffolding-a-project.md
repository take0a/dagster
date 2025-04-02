---
title: 'プロジェクトの足場作り'
sidebar_position: 100
---

import Preview from '@site/docs.ja/partials/\_Preview.md';

<Preview />

`dg` は、[Dagster コードの場所](https://docs.dagster.io/guides/deploy/code-locations/managing-code-locations-with-definitions)を定義する、_プロジェクト_と呼ばれる特別なタイプの Python パッケージのスキャフォールディングをサポートします。

新しいプロジェクトをスキャフォールディングするには、`dg scaffold project` コマンドを使用します:

<CliInvocationExample path="docs_snippets/docs_snippets/guides/dg/scaffolding-project/1-scaffolding-project.txt" />

## プロジェクト構造

`dg scaffold project` コマンドは、いくつかの追加機能を備えた標準の Python パッケージ構造を持つディレクトリを作成します:

<CliInvocationExample path="docs_snippets/docs_snippets/guides/dg/scaffolding-project/2-tree.txt" />

- 最上位パッケージ `my_project` には、Dagster パイプラインを定義するデプロイ可能なコードが含まれています。
- `my_project/defs` には、Dagster 定義が含まれます。
- `my_project/lib` では、カスタム コンポーネント タイプと、オプションで Dagster 定義間で共有するその他のコードを定義します。
- `my_project/definitions.py` は、コードの場所をデプロイするときに Dagster が読み込むエントリ ポイントです。`my_project/defs` からすべての定義を読み込むように構成されています。このファイルを変更する必要はありません。
- `my_project_tests` には、`my_project` のコードに対するテストが含まれています。
- `pyproject.toml` は、標準の Python パッケージ構成ファイルです。通常の Python パッケージ メタデータに加えて、`dg` 固有の設定用の `tool.dg` セクションが含まれています。
- `uv.lock` は、Python パッケージ マネージャー [`uv`](https://docs.astral.sh/uv/concepts/projects/layout/#the-lockfile) の [ロックファイル](https://docs.astral.sh/uv/concepts/projects/layout/#the-lockfile) です。`dg` プロジェクトはデフォルトで `uv` を使用します。詳細については、[`uv` 統合](/guides/labs/dg/uv-integration) を参照してください。
