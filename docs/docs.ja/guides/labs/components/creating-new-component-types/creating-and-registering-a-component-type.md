---
title: 'コンポーネントタイプの作成と登録'
sidebar_position: 100
---

import Preview from '@site/docs.ja/partials/\_Preview.md';

<Preview />

コンポーネントシステムを使用すると、プロジェクト全体で再利用できる新しいコンポーネント タイプを簡単に作成できます。

ほとんどの場合、コンポーネント タイプは特定のテクノロジーにマップされます。たとえば、Docker コンテナーでスクリプトを実行する `DockerScriptComponent` や、Snowflake でクエリを実行する `SnowflakeQueryComponent` などがあります。

:::note

コンポーネント互換プロジェクトを作成する方法については、プロジェクト構造ガイドを参照してください。

:::

## コンポーネントタイプファイルのスキャフォールディング

この例では、シェル コマンドを実行する軽量コンポーネントを作成します。

まず、`dg` コマンドライン ユーティリティを使用して、新しいコンポーネント タイプをスキャフォールディングします:

<CliInvocationExample path="docs_snippets/docs_snippets/guides/components/shell-script-component/1-dg-scaffold-shell-command.txt" />

これにより、プロジェクトの `lib` ディレクトリに新しいファイルが追加されます:

<CodeExample
  path="docs_snippets/docs_snippets/guides/components/shell-script-component/2-shell-command-empty.py"
  language="python"
  title="my_component_library/lib/shell_command.py"
/>

このファイルには、新しいコンポーネント タイプの基本構造が含まれています。私たちの目標は、`build_defs` メソッドを実装して `Definitions` を返すことです。これには、コンポーネント クラスをインスタンス化するために定義する入力が必要です。

:::note
The use of `Model` is optional if you only want a Pythonic interface to the component. If you wish to implement an `__init__` method for your class (manually or using `@dataclasses.dataclass`), you can provide the `--no-model` flag to the `dg scaffold` command.
:::

## Pythonクラスの定義

最初のステップは、このコンポーネントに必要な情報を定義することです。つまり、コンポーネントのどの側面をカスタマイズ可能にするか決定するということです。

この場合、次の項目を定義します:

- 実行するシェル スクリプトへのパス。
- このスクリプトで生成されると予想されるアセット。

クラスは、`Component` に加えて `Resolvable` からも継承します。これにより、クラスに注釈が付けられている内容に基づいて、クラスの yaml スキーマを導出します。一般的なユース ケースを簡素化するために、Dagster は、yaml から `AssetSpec` を定義するためのスキーマを公開し、コンポーネントをインスタンス化する前にそれらを解決するための `ResolvedAssetSpec` などの一般的な構成の注釈を提供します。

次のように、コンポーネントのスキーマを作成してクラスに追加できます:

<CodeExample
  path="docs_snippets/docs_snippets/guides/components/shell-script-component/with-config-schema.py"
  language="python"
  title="my_component_library/lib/shell_command.py"
  />

Additionally, it's possible to include metadata for your Component by overriding the `get_component_type_metadata` method. This allows you to set fields like `owners` and `tags` that will be visible in the generated documentation.

<CodeExample
  path="docs_snippets/docs_snippets/guides/components/shell-script-component/with-config-schema-meta.py"
  language="python"
  title="my_component_library/lib/shell_command.py"
  />

:::tip

スキーマにない、または異なるタイプのコンポーネントのフィールドを定義する場合、コンポーネント システムでは、そのフィールドに対してカスタム解決ロジックを提供できます。詳細については、[非標準タイプの解決ロジックの提供](#advanced-providing-resolution-logic-for-non-standard-types)セクションを参照してください。
:::

## 定義の構築

コンポーネントのパラメータ化方法を定義したので、それらのパラメータを `Definitions` オブジェクトに変換する方法を定義する必要があります。

そのためには、コンポーネントに関連するすべての定義を含む `Definitions` オブジェクトを返す `build_defs` メソッドをオーバーライドする必要があります。

`build_defs` メソッドは、提供されたシェル スクリプトを実行する単一の `@asset` を作成します。慣例により、このアセットを実際に実行するコードは `execute` という関数内に配置します。これにより、将来の開発者がこのコンポーネントのサブクラスを作成しやすくなります。

<CodeExample
  path="docs_snippets/docs_snippets/guides/components/shell-script-component/with-build-defs.py"
  language="python"
  title="my_component_library/lib/shell_command.py"
/>

## コンポーネントの登録

上記の手順に従うと、コンポーネント タイプが環境に自動的に登録されます。これで、以下を実行できます:

<CliInvocationExample path="docs_snippets/docs_snippets/guides/components/shell-script-component/3-dg-list-component-types.txt" />

使用可能なコンポーネント タイプのリストに新しいコンポーネント タイプが表示されます。

また、次のコマンドを実行して、新しいコンポーネント タイプを説明する自動生成されたドキュメントを表示することもできます:

<CliInvocationExample contents="dg docs serve" />

これで、このコンポーネント タイプを使用して新しいコンポーネント インスタンスを作成できるようになりました。

## カスタムスキャフォールディングの設定

コンポーネント タイプが登録されると、`dg scaffold component` コマンドを使用してコンポーネント タイプのインスタンスをスキャフォールディングできます:

<CliInvocationExample path="docs_snippets/docs_snippets/guides/components/shell-script-component/4-scaffold-instance-of-component.txt" />

デフォルトでは、未入力の `component.yaml` ファイルとともに新しいディレクトリが作成されます。ただし、component_type を `scaffoldable` で装飾することで、この動作をカスタマイズできます。

この場合、入力済みの `component.yaml` ファイルとともにテンプレート シェル スクリプトをスキャフォールディングする必要があります。これは、カスタム スキャフォールダーを使用して実現します:

<CodeExample
  path="docs_snippets/docs_snippets/guides/components/shell-script-component/with-scaffolder.py"
  language="python"
  title="my_component_library/lib/shell_command.py"
/>

ここで、`dg scaffold component` を実行すると、入力された `component.yaml` ファイルとともにテンプレート シェル スクリプトが作成されることがわかります:

<CodeExample
  path="docs_snippets/docs_snippets/guides/components/shell-script-component/5-scaffolded-component.yaml"
  language="yaml"
  title="my_component_library/components/my_shell_command/component.yaml"
/>

<CodeExample
  path="docs_snippets/docs_snippets/guides/components/shell-script-component/6-scaffolded-component-script.sh"
  language="bash"
  title="my_component_library/components/my_shell_command/script.sh"
/>

## [上級] 非標準型の解決ロジックを提供する

ほとんどの場合、コンポーネント スキーマとコンポーネント クラスで使用する型は同じか、`ResolvedAssetSpec` の場合のように、すぐに使用できる解決ロジックが備わります。

ただし、既存のスキーマに相当するものがない型を使用する必要がある場合もあります。この場合、フィールドに `Annotated[<type>, Resolver(...)]` という注釈を付けることで、値を目的の型に解決する関数を提供できます。

たとえば、YAML で API キーを使用して構成できる API クライアントや、テストでモック クライアントをコンポーネントに提供したい場合があります:

<CodeExample
  path="docs_snippets/docs_snippets/guides/components/shell-script-component/custom-schema-resolution.py"
  language="python"
/>

## [上級] YAML値のレンダリングをカスタマイズする

コンポーネント システムは、`component.yaml` ファイルに基づいて任意の Python 値をロードできる豊富なテンプレート構文をサポートしています。`Resolvable` 内のすべての文字列値は、Jinja2 テンプレート エンジンを使用してテンプレート化でき、任意の Python 型に解決できます。これにより、純粋な YAML で作業している場合でも、`PartitionsDefinition` や `AutomationCondition` などの複雑なオブジェクト型をコンポーネントのユーザーに公開できます。

コンポーネントで `get_additional_scope` クラスメソッドを定義することで、テンプレート エンジンで使用できるカスタム値を定義できます。この場合、事前定義された開始日を持つ `DailyPartitionsDefinition` オブジェクトを返す `"daily_partitions"` 関数を定義できます。

<CodeExample
  path="docs_snippets/docs_snippets/guides/components/shell-script-component/with-custom-scope.py"
  language="python"
/>

ユーザーがこのコンポーネントをインスタンス化すると、`component.yaml` ファイルでこのカスタム スコープを使用できるようになります:

```yaml
component_type: my_component

attributes:
  script_path: script.sh
  asset_specs:
    - key: a
      partitions_def: '{{ daily_partitions }}'
```

## Next steps

- [プロジェクトに新しいコンポーネントを追加する](/guides/labs/components/building-pipelines-with-components/adding-components)
