---
title: "高度な設定タイプ"
description: Dagster's config system supports a variety of more advanced config types.
sidebar_position: 200
---

場合によっては、アセットとオペレーションに対してより複雑な [config schema](run-configuration) を定義したい場合があります。たとえば、ファイルのリストや複雑なデータを取得する config schema を定義したい場合があります。このガイドでは、より複雑な config schema を定義するための一般的なパターンをいくつか説明します。

## 設定フィールドにメタデータを添付する

構成フィールドにはメタデータを注釈として付けることができ、Pydantic <PyObject section="config" module="dagster" object="Field"/> クラスを使用して、フィールドに関する追加情報を提供するのに使用できます。

たとえば、構成フィールドに説明を注釈として付けることができ、その説明は構成フィールドのドキュメントに表示されます。フィールドに値の範囲を追加することができ、構成が指定されたときに検証されます。

<CodeExample path="docs_snippets/docs_snippets/guides/dagster/pythonic_config/pythonic_config.py" startAfter="start_metadata_config" endBefore="end_metadata_config" dedent="4" />

## デフォルトとオプションの設定フィールド

構成フィールドにはデフォルト値を設定できます。デフォルト値が設定されているフィールドは必須ではありません。つまり、構成オブジェクトを構築するときに指定する必要はありません。

たとえば、`greeting_phrase` フィールドに `"hello"` というデフォルト値を添付し、フレーズを指定せずに `MyAssetConfig` を構築できます。`person_name` などの `Optional` としてマークされているフィールドには、暗黙的に `None` というデフォルト値が設定されますが、以下の例のように明示的に `None` に設定することもできます。

<CodeExample path="docs_snippets/docs_snippets/guides/dagster/pythonic_config/pythonic_config.py" startAfter="start_optional_config" endBefore="end_optional_config" dedent="4" />

### 必須の設定フィールド

デフォルトでは、`Optional` と入力されたフィールドは設定で指定する必要はなく、暗黙のデフォルト値として `None` が設定されます。設定でフィールドを指定することを必須にする場合は、省略記号 (`...`) を使用して [値を渡すことを必須にする](https://docs.pydantic.dev/usage/models/#required-fields) ことができます。

<CodeExample path="docs_snippets/docs_snippets/guides/dagster/pythonic_config/pythonic_config.py" startAfter="start_required_config" endBefore="end_required_config" dedent="4" />

## 基本的なデータ構造

基本的な Python データ構造は、これらのデータ構造のネストされたバージョンとともに、構成スキーマで使用できます。使用できるデータ構造は次のとおりです:

- `List`
- `Dict`
- `Mapping`

たとえば、ユーザー名のリストとユーザー名とユーザースコアのマッピングを取得する構成スキーマを定義できます。

<CodeExample path="docs_snippets/docs_snippets/guides/dagster/pythonic_config/pythonic_config.py" startAfter="start_basic_data_structures_config" endBefore="end_basic_data_structures_config" dedent="4" />

## ネストされたスキーマ

スキーマは、互いにネストすることも、基本的な Python データ構造内にネストすることもできます。

ここでは、ユーザー名と複雑なユーザー データ オブジェクトのマッピングを含むスキーマを定義します。

<CodeExample path="docs_snippets/docs_snippets/guides/dagster/pythonic_config/pythonic_config.py" startAfter="start_nested_schema_config" endBefore="end_nested_schema_config" dedent="4" />

## 許容スキーマ

デフォルトでは、`Config` スキーマは厳密です。つまり、スキーマで明示的に定義されているフィールドのみが受け入れられます。ユーザーが構成で任意のフィールドを指定できるようにする場合、これは面倒になる可能性があります。この目的のために、構成で任意のフィールドを指定できるようにする `PermissiveConfig` 基本クラスを使用できます。

<CodeExample path="docs_snippets/docs_snippets/guides/dagster/pythonic_config/pythonic_config.py" startAfter="start_permissive_schema_config" endBefore="end_permissive_schema_config" dedent="4" />

## ユニオン型

ユニオン型は、Pydantic [判別ユニオン](https://docs.pydantic.dev/usage/types/#discriminated-unions-aka-tagged-unions) を使用してサポートされます。各ユニオン型は、<PyObject section="config" module="dagster" object="Config"/> のサブクラスである必要があります。<PyObject section="config" module="dagster" object="Field"/> の `​​discriminator` 引数は、使用するユニオン型を決定するために使用されるフィールドを指定します。判別ユニオンは、従来の Dagster 構成 API の `Selector` 型と同等の機能を提供します。

ここでは、`pet_type` フィールドで示されるように、`Cat` または `Dog` のいずれかになる `pet` フィールドを受け取る構成スキーマを定義します。

<CodeExample path="docs_snippets/docs_snippets/guides/dagster/pythonic_config/pythonic_config.py" startAfter="start_union_schema_config" endBefore="end_union_schema_config" dedent="4" />

### ユニオン型のYAMLと設定辞書表現

判別共用体の YAML または config 辞書表現は、Python 表現とは少し異なる構造になっています。YAML 表現では、判別子キーが共用体型の辞書のキーとして使用されます。たとえば、`Cat` オブジェクトは次のように表現されます:

```yaml
pet:
  cat:
    meows: 10
```

設定辞書の表現では、同じパターンが使用されます:

```python
{
    "pet": {
        "cat": {
            "meows": 10,
        }
    }
}
```

## 列挙型

`Enum` のサブクラスである Python 列挙型は、設定フィールドとしてサポートされています。ここでは、列挙値としてロールが指定されているユーザーのリストを取得するスキーマを定義します:

<CodeExample path="docs_snippets/docs_snippets/guides/dagster/pythonic_config/pythonic_config.py" startAfter="start_enum_schema_config" endBefore="end_enum_schema_config" dedent="4" />

### 列挙型の YAML および設定辞書表現

Python 列挙型の YAML または設定辞書表現では、列挙型の名前が使用されます。たとえば、上記のユーザー リストの YAML 仕様は次のようになります:

```yaml
users_list:
  Bob: GUEST
  Alice: ADMIN
```

設定辞書の表現では、同じパターンが使用されます:

```python
{
    "users_list": {
        "Bob": "GUEST",
        "Alice": "ADMIN",
    }
}
```

## 検証された構成フィールド

構成フィールドには、[Pydantic バリデータ](https://docs.pydantic.dev/usage/validators/) を使用してカスタム検証ロジックを適用できます。Pydantic バリデータは、構成クラスのメソッドとして定義され、`@validator` デコレータで装飾されます。これらのバリデータは、構成クラスがインスタンス化されるときにトリガーされます。実行時に定義された構成の場合、バリデータが失敗しても起動ボタンが押されることは防止されませんが、例外が発生し、実行が開始されなくなります。

ここでは、構成されたユーザーの名前とユーザー名にいくつかの検証を定義します。これにより、ランチパッドまたはスケジュールやセンサーから誤った値が渡された場合に例外がスローされます。

<CodeExample path="docs_snippets/docs_snippets/guides/dagster/pythonic_config/pythonic_config.py" startAfter="start_validated_schema_config" endBefore="end_validated_schema_config" />
