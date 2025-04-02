---
title: 'コンポーネントのライブラリを作成する'
sidebar_position: 200
---

import Preview from '@site/docs.ja/partials/\_Preview.md';

<Preview />

`dg` CLI に Python パッケージにコンポーネント タイプが含まれていることを知らせるには、次の構成で `pyproject.toml` ファイルを更新します:

```toml
[tool.dg]
is_component_lib = true
```

デフォルトでは、すべてのコンポーネント タイプが `your_package.lib` で定義されると想定されています。別のディレクトリでコンポーネントを定義する場合は、`pyproject.toml` ファイルでこれを指定できます:

```toml
[tool.dg]
is_component_lib = true
component_lib_package="your_package.other_module"
```

これが完了すると、このパッケージが環境にインストールされている限り、`dg` コマンドライン ユーティリティを使用してコンポーネント タイプを操作できるようになります。
