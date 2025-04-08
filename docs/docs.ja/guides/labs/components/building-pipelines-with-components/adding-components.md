---
title: 'プロジェクトにコンポーネントを追加する'
sidebar_position: 200
---

import Preview from '@site/docs.ja/partials/\_Preview.md';

<Preview />

プロジェクトにコンポーネントを追加するには、コマンドラインからインスタンス化します。これにより、`components/` フォルダー内に `component.yaml` ファイルを含む新しいディレクトリが作成されます。

代わりに Python を使用してプロジェクトにコンポーネントを追加する場合は、「[Python を使用してプロジェクトにコンポーネントを追加する](/guides/labs/components/building-pipelines-with-components/adding-components-python)」を参照してください。

:::note Prerequisites

コンポーネントを追加する前に、[コンポーネントを使用してプロジェクトを作成する](/guides/labs/components/building-pipelines-with-components/creating-a-project-with-components)か、[既存のプロジェクトを `dg` に移行する](/guides/labs/dg/incrementally-adopting-dg/migrating-project)必要があります。

:::

## コンポーネントの検索

次のコマンドを実行すると、環境で使用可能なコンポーネント タイプを表示できます。

```bash
dg list component-type
```

これにより、プロジェクトで使用可能なすべてのコンポーネント タイプのリストが表示されます。特定のコンポーネント タイプに関する詳細情報を表示するには、次のコマンドを実行します:

```bash
dg docs component-type <component-name>
```

これにより、指定されたコンポーネント タイプのドキュメントを含む Web ページが表示されます。

## コンポーネントのインスタンス化

使用するコンポーネント タイプを決定したら、次のコマンドを実行してインスタンス化できます:

```bash
dg scaffold component <component-type> <component-name>
```

これにより、`components/` フォルダ内に `component.yaml` ファイルを含む新しいディレクトリが作成されます。一部のコンポーネント タイプでは、必要に応じて追加のファイルも生成される場合があります。

## 構成

### 基本構成

`component.yaml` はコンポーネントの主要な設定ファイルです。これには 2 つの最上位フィールドが含まれます。

- `type`: このディレクトリで定義されているコンポーネントのタイプ
- `attributes`: このコンポーネント タイプに固有の属性の辞書。これらの属性のスキーマは `Component` の属性によって定義され、コンポーネント クラスの `get_model_cls` メソッドをオーバーライドすることで完全にカスタマイズされます。

特定のコンポーネントのサンプル `component.yaml` ファイルを表示するには、次のコマンドを実行します:

```bash
dg docs component-type <component-name>
```

### コンポーネントテンプレート

各 `component.yaml` ファイルは、`jinja2` を利用した豊富なテンプレート構文をサポートしています。

#### テンプレート環境変数

テンプレートの一般的な使用例は、YAML ファイルで環境変数 (特にシークレット) を公開しないようにすることです。`component.yaml` ファイルの Jinja スコープには、テンプレートに環境変数を挿入するために使用できる `env` 関数が含まれています:

```yaml
component_type: my_snowflake_component

attributes:
  account: "{{ env('SNOWFLAKE_ACCOUNT') }}"
  password: "{{ env('SNOWFLAKE_PASSWORD') }}"
```
