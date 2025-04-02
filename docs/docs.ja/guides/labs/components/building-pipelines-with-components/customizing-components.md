---
title: 'コンポーネントのカスタマイズ'
sidebar_position: 400
---

import Preview from '@site/docs.ja/partials/\_Preview.md';

<Preview />

`component.yaml` ファイルで利用可能なものを超えて、コンポーネントの動作をカスタマイズできます。

これを行うには、`component.yaml` ファイルと同じディレクトリにある `component.py` という名前のファイルに、目的のコンポーネントのサブクラスを作成します。

<CodeExample path="docs_snippets/docs_snippets/guides/components/custom-subclass/basic-subclass.py" language="python" />

次に、`component.yaml` ファイルの `type:` フィールドを更新して、この新しいコンポーネント タイプを参照します。これは、タイプの完全修飾名である必要があります。

```yaml
type: my_project.defs.my_def.CustomSubclass

attributes: ...
```

## 実行のカスタマイズ

たとえば、実行中にデバッグ ログ メッセージを追加する `SlingReplicationCollectionComponent` のサブクラスを作成できます。`SlingReplicationCollectionComponent` には、サブクラスによってオーバーライドできる `execute` メソッドがあります。

<CodeExample path="docs_snippets/docs_snippets/guides/components/custom-subclass/debug-mode.py" language="python" />

## コンポーネントレベルのテンプレートスコープの追加

デフォルトでは、テンプレートで使用できるスコープは次のとおりです:

- `env`: 環境変数にアクセスできる関数。
- `automation_condition`: `AutomationCondition` クラスのすべての静的コンストラクターにアクセスできるスコープ。

ただし、コンポーネント タイプに追加のスコープ オプションを追加すると便利な場合があります。たとえば、コンポーネントで使用したいカスタム自動化条件がある場合があります。

これを行うには、`AutomationCondition` を返す関数を定義し、サブクラスで `get_additional_scope` メソッドを定義します:

<CodeExample path="docs_snippets/docs_snippets/guides/components/custom-subclass/custom-scope.py" language="python" />

これを `component.yaml` ファイルで使用できます:

```yaml
type: my_project.defs.my_def.CustomSubclass

attributes:
    ...
    asset_post_processors:
        - attributes:
            automation_condition: "{{ custom_cron('@daily') }}"
```
