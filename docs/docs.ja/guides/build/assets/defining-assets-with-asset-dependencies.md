---
title: 他のアセットに依存するアセットの定義
sidebar_position: 200
---

アセット定義は、他のアセット定義に依存できます。依存するアセットは **ダウンストリームアセット** と呼ばれ、依存するアセットは **アップストリームアセット** と呼ばれます。

## 基本的な依存関係の定義

上流アセットを下流アセットの `@asset` デコレータの `deps` パラメータに渡すことで、2 つのアセット間の依存関係を定義できます。

この例では、アセット `sugary_cereals` は `cereals` テーブルからレコードを選択して新しいテーブル (`sugary_cereals`) を作成します。次に、アセット `shopping_list` は `sugary_cereals` からレコードを選択して新しいテーブル (`shopping_list`) を作成します:

<CodeExample path="docs_snippets/docs_snippets/guides/data-modeling/asset-dependencies/asset-dependencies.py" language="python" lineStart="6" lineEnd="20"/>

## コードの場所をまたがるアセットの依存関係の定義

アセットは、異なる [コードの場所](/guides/deploy/code-locations/) にあるアセットに依存する場合があります。次の例では、`code_location_1_asset` アセットは `code_location_1` にあるファイルから JSON 文字列を生成します。

<CodeExample path="docs_snippets/docs_snippets/guides/data-modeling/asset-dependencies/asset-dependencies.py" language="python" lineStart="21" lineEnd="34"/>

`code_location_2` では、アセット キーを介して `code_location_1_asset` を参照できます:

<CodeExample path="docs_snippets/docs_snippets/guides/data-modeling/asset-dependencies/asset-dependencies.py" language="python" lineStart="34" lineEnd="46"/>

