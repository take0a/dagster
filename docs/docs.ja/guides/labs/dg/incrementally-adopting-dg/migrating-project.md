---
title: '既存のプロジェクトを dg を使用するように変換する'
sidebar_position: 200
---

import Preview from '@site/docs.ja/partials/\_Preview.md';

<Preview />

:::note

このガイドは、_既存の_ Dagster プロジェクトから開始する場合にのみ関連します。`dg scaffold project` を使用して [新しいプロジェクトをスキャフォールディング](/guides/labs/dg/scaffolding-a-project) した場合、このセットアップは不要です。

:::

基本的な既存の Dagster プロジェクトがあり、`pip` を使用して Python 環境を管理しています。プロジェクトでは、`setup.py` と単一の Dagster アセットを含む Python パッケージを定義します。アセットは、`my_existing_project/definitions.py` の最上位の `Definitions` オブジェクトで公開されます。

<CliInvocationExample path="docs_snippets/docs_snippets/guides/dg/migrating-project/1-tree.txt" />

## 依存関係をインストールする

### `dg`コマンドラインツールをインストールする

新しい Python 仮想環境から始めましょう (すでに存在している場合もあります)。

<CliInvocationExample contents="python -m venv .venv && source .venv/bin/activate" />

`dg` コマンドライン ツールをインストールする必要があります。`pip` を使用して仮想環境にインストールできます:

<CliInvocationExample contents="pip install dagster-dg" />

:::note
このチュートリアルでは、わかりやすくするために、`dg` 実行ファイルをプロジェクトと同じ仮想環境にインストールします。ただし、通常は、[`uv`](https://docs.astral.sh/uv/getting-started/installation/) パッケージ マネージャーを使用して、`uv tool install dagster-dg` で `dg` をグローバルにインストールすることをお勧めします。これにより、`dagster-dg` が分離された環境にインストールされ、グローバルに使用可能な実行ファイルになります。これにより、`dg` を使用して複数のプロジェクトで作業しやすくなります。
:::

<!-- ### `dagster-components` をインストールする

次に、プロジェクトの依存関係として `dagster-components` を追加する必要があります。`setup.py` の `install_requires` に追加します:

<CodeExample path="docs_snippets/docs_snippets/guides/dg/migrating-project/2-setup.py" language="python" title="setup.py" />

次に、プロジェクトをアクティブな仮想環境にインストール (または再インストール) します:

<CliInvocationExample contents="pip install -e ." /> -->

## プロジェクト構造の更新

### `pyproject.toml` を追加する

`dg` コマンドは、`tool.dg` セクションを含む `pyproject.toml` ファイルの存在によって Dagster プロジェクトを認識します。プロジェクトですでに `pyproject.toml` を使用している場合は、必要な `tool.dg` をファイルに追加するだけです。サンプル プロジェクトには `setup.py` があり、`pyproject.toml` がないため、新しい `pyproject.toml` ファイルを作成する必要があります。これは `setup.py` と共存できます。これは純粋に `dg` の設定ソースです。`pyproject.toml` に追加する必要がある設定は次のとおりです:

<CodeExample path="docs_snippets/docs_snippets/guides/dg/migrating-project/3-pyproject.toml" language="toml" title="pyproject.toml" />

設定は 3 つあります:

- `tool.dg.directory_type = "project"`: これは `dg` がパッケージを Dagster プロジェクトとして識別する方法です。これは必須です。
- `tool.dg.project.root_module = "my_existing_project"`: これはプロジェクトのルート モジュールを指します。これも必須です。
- `tool.dg.project.code_location_target_module = "my_existing_project.definitions"`: これは `dg` にプロジェクト内の最上位の `Definitions` オブジェクトの場所を伝えます。これは実際にはデフォルトで `[root_module].definitions` になっているため、ここで設定する必要は厳密にはありませんが、明示するためにこの設定を含めています。既存のプロジェクトでは最上位の `Definitions` オブジェクトが別のモジュールで定義されている可能性があり、その場合はこの設定が必要です。

これらの設定が完了したら、`dg` を使用してプロジェクトを操作できます。`dg list defs` を実行すると、プロジェクトに存在する唯一のアセットが表示されます:

<CliInvocationExample path="docs_snippets/docs_snippets/guides/dg/migrating-project/4-list-defs.txt"  />

### `defs`ディレクトリを作成する

`dg` エクスペリエンスの一部は、定義の _自動読み込み_ です。これは、特定のモジュール内に存在する定義を自動的に取得することを意味します。`my_existing_project.defs` (`defs` は、`dg` 内で定義が存在するモジュールの慣例的な名前です) という名前の新しいサブモジュールを作成し、そこから定義を自動読み込みします。

<CliInvocationExample path="docs_snippets/docs_snippets/guides/dg/migrating-project/5-mkdir-defs.txt" />

## トップレベルの定義を変更する

自動ロードは、`Definitions` オブジェクトを返す関数によって提供されます。プロジェクトにはすでに他の定義がいくつかあるため、それらを `my_existing_project.defs` から自動ロードされた定義と組み合わせます。

これを行うには、`definitions.py` ファイル、または最上位の `Definitions` オブジェクトを含むファイルを変更する必要があります。

`load_defs` を使用して定義を自動ロードし、`Definitions.merge` を使用して既存の定義とマージします。`load_defs` に、作成した `defs` モジュールを渡します:

<Tabs>
  <TabItem value="before" label="Before">
    <CodeExample
      path="docs_snippets/docs_snippets/guides/dg/migrating-project/6-initial-definitions.py"
      language="python"
    />
  </TabItem>
  <TabItem value="after" label="After">
    <CodeExample
      path="docs_snippets/docs_snippets/guides/dg/migrating-project/7-updated-definitions.py"
      language="python"
    />
  </TabItem>
</Tabs>

次に、新しい `defs` モジュールにアセットを追加しましょう。次の内容で `my_existing_project/defs/autoloaded_asset.py` を作成します:

<CodeExample path="docs_snippets/docs_snippets/guides/dg/migrating-project/8-autoloaded-asset.py" />

最後に、新しいアセットが自動ロードされていることを確認しましょう。もう一度 `dg list defs` を実行すると、新しい `autoloaded_asset` と古い `my_asset` の両方が表示されます:

<CliInvocationExample path="docs_snippets/docs_snippets/guides/dg/migrating-project/9-list-defs.txt"  />

これで、プロジェクトは `dg` と完全に互換性を持つようになりました！

## Next steps

- [既存の定義を再構築する](/guides/labs/dg/incrementally-adopting-dg/migrating-definitions)
- [プロジェクトに新しい定義を追加する](/guides/labs/dg/dagster-definitions)
