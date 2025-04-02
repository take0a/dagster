---
title: "種類タグ"
description: "Use kind tags to easily categorize assets within your Dagster project."
sidebar_position: 2000
---

種類タグを使用すると、Dagster UI で特定のアセットに使用されている基盤となるシステムまたはテクノロジーをすばやく識別できます。これらのタグを使用すると、Dagster プロジェクト内でアセットを整理したり、すべてのアセットをフィルタリングまたは検索したりすることができます。

## アセットに種類を追加する

<PyObject section="assets" module="dagster" object="asset" decorator /> の `kinds` 引数には最大 3 つの種類を追加できます。これは、アセットが関連付けられている複数のテクノロジーやシステムを表すのに役立ちます。たとえば、Python コードで構築され、Snowflake に保存されているアセットには、`python` と `snowflake` の両方の種類でタグ付けできます:

<CodeExample path="docs_snippets/docs_snippets/concepts/metadata-tags/asset_kinds.py" />

multi_asset で使用するために、<PyObject section="assets" module="dagster" object="AssetSpec" /> で種類を指定することもできます:

<CodeExample path="docs_snippets/docs_snippets/concepts/metadata-tags/asset_kinds_multi.py" />

バックエンドでは、これらの種類の入力はアセットのタグとして保存されます。詳細については、[タグ](/guides/build/assets/metadata-and-tags/index.md#tags) を参照してください。

系統ビューでアセットを表示すると、添付された種類がアセットの下部に表示されます。

<img
  src="/images/guides/build/assets/metadata-tags/kinds/kinds.svg"
  alt="Asset in lineage view with attached kind tags"
/>

## アセットにコンピューティングの種類を追加する

:::warning

`compute_kind` の使用は `kinds` 引数に置き換えられました。代わりに `kinds` 引数を使用することをお勧めします。

:::

`compute_kind` 引数に値を指定することで、任意のアセットに単一のコンピューティングの種類を追加できます。

```python
@asset(compute_kind="dbt")
def my_asset():
    pass
```

## サポートされているアイコン

一部の種類には、UI にブランド アイコンが与えられます。現在、約 200 個の固有のテクノロジー アイコンをサポートしています。

import KindsTags from '@site/docs/partials/\_KindsTags.md';

<KindsTags />

## 追加アイコンのリクエスト

Kinds アイコン パックはオープン ソースであり、誰でも新しいアイコンをパブリックリポジトリに[投稿したり](/about/contributing)、[問題を報告して](https://github.com/dagster-io/dagster/issues/new?assignees=&labels=type%3A+feature-request&projects=&template=request_a_feature.ym)新しいアイコンをリクエストしたりできます。デプロイメントごとのカスタム アイコンは現在サポートされていません。
