---
title: マルチアセット統合の作成
description: Create a decorator based multi-asset integration
sidebar_position: 200
---

Dagster エコシステムで作業しているときに、デコレータが頻繁に使用されていることに気付いたかもしれません。たとえば、アセット、ジョブ、オペレーションはデコレータを使用します。多数のアセットを生成するサービスがある場合は、それをマルチアセット デコレータとして定義して、既存の Dagster API に一貫性のある直感的な開発者エクスペリエンスを提供することができます。

Dagster のコンテキストでは、デコレータは何らかの処理をラップすることが多いため便利です。たとえば、アセットを作成するときは、処理コードを定義し、関数に `asset` decorator /> デコレータで注釈を付けます。その後、内部の Dagster コードでアセットを登録したり、メタデータを割り当てたり、コンテキスト データを渡したり、アセット コードを Dagster プラットフォームに統合するために必要なその他のさまざまな操作を実行したりできます。

このガイドでは、架空のレプリケーション ツールのマルチアセット統合を開発する方法を学習します。

:::note
このガイドでは、Dagster と Python デコレータに関する基本的な知識があることを前提としています。
:::

## Step 1: インプット

このガイドでは、2 つのデータベース間でデータを複製するツールを想像してみましょう。このツールは `replication.yaml` 構成ファイルを使用して構成され、ユーザーはソース データベースと宛先データベース、およびこれらのシステム間で複製するテーブルを定義できます。

```yml
connections:
  source:
    type: duckdb
    connection: example.duckdb
  destination:
    type: postgres
    connection: postgresql://postgres:postgres@localhost/postgres

tables:
  - name: users
    primary_key: id
  - name: products
    primary_key: id
  - name: activity
    primary_key: id
```

私たちが構築している統合では、このレプリケーション プロセスを網羅し、レプリケートされるテーブルごとにアセットを生成するマルチアセットを提供したいと考えています。

レプリケーション プロセスをモックし、各テーブルのレプリケーション ステータスを含む辞書を返す `replicate` というダミー関数を定義します。実際の環境では、これはライブラリ内の関数、またはコマンドライン ツールの呼び出しになる可能性があります。

```python
import yaml

from pathlib import Path
from typing import Mapping, Iterator, Any


def replicate(replication_configuration_yaml: Path) -> Iterator[Mapping[str, Any]]:
    data = yaml.safe_load(replication_configuration_yaml.read_text())
    for table in data.get("tables"):
        # < perform replication here, and get status >
        yield {"table": table.get("name"), "status": "success"}
```

## Step 2: 実装 {#step-2-implementation}

まず、構成 YAML ファイルのパスを受け取る `Project` オブジェクトを定義しましょう。これにより、プロジェクト構成からメタデータとテーブル情報を取得するロジックをカプセル化できるようになります。

```python
import yaml
from pathlib import Path


class ReplicationProject():
    def __init__(self, replication_configuration_yaml: str):
        self.replication_configuration_yaml = replication_configuration_yaml

    def load(self):
        return yaml.safe_load(Path(self.replication_configuration_yaml).read_text())
```

次に、`multi_asset` 関数を返す関数を定義します。`multi_asset` 関数自体がデコレータなので、`multi_asset` の動作をカスタマイズして、独自の新しいデコレータを作成できます:

```python
def custom_replication_assets(
    *,
    replication_project: ReplicationProject,
    name: Optional[str] = None,
    group_name: Optional[str] = None,
) -> Callable[[Callable[..., Any]], AssetsDefinition]:
    project = replication_project.load()

    return multi_asset(
        name=name,
        group_name=group_name,
        specs=[
            AssetSpec(
                key=table.get("name"),
            )
            for table in project.get("tables")
        ],
    )
```

このコードが何をするのか確認してみましょう:

- `multi_asset` 関数を返す関数を定義します
- レプリケーション プロジェクトをロードし、入力 YAML ファイルで定義されているテーブルを反復処理します
- テーブルを使用して `AssetSpec` オブジェクトのリストを作成し、それらを `specs` パラメータに渡して、Dagster UI に表示されるアセットを定義します

次に、レプリケーション関数の実行方法を説明します。

デコレータを使用すると、何らかの操作を実行する関数をラップできることを思い出してください。`multi_asset` の場合、テーブルに `AssetSpec` オブジェクトを定義し、実際に実行される処理はデコレートされた関数の本体で行われます。

この関数では、レプリケーションを実行し、特定のテーブルでレプリケーションが成功したことを示す `AssetMaterialization` オブジェクトを生成します。

```python
from dagster import AssetExecutionContext


replication_project_path = "replication.yaml"
replication_project = ReplicationProject(replication_project_path)


@custom_replication_assets(
    replication_project=replication_project,
    name="my_custom_replication_assets",
    group_name="replication",
)
def my_assets(context: AssetExecutionContext):
    results = replicate(Path(replication_project_path))
    for table in results:
        if table.get("status") == "SUCCESS":
            yield AssetMaterialization(asset_key=str(table.get("name")), metadata=table)
```

このアプローチにはいくつかの制限があります:

- **テーブルをレプリケートするためのロジックをカプセル化していません。** つまり、`custom_replication_assets` デコレータを使用するユーザーは、アセットの実体化を自分で行う必要があります。
- **ユーザーはアセットの属性をカスタマイズできません。**

最初の制限については、アセット関数の本体のコードを Dagster リソースにリファクタリングすることで解決できます。

## Step 3: レプリケーションロジックをリソースに移動する

レプリケーション ロジックをリソースにリファクタリングすると、ロジックのより適切な構成と再利用をサポートできます。

これを実現するには、`ConfigurableResource` オブジェクトを拡張してカスタム リソースを作成します。次に、レプリケーション操作を実行する `run` メソッドを定義します:

```python
from dagster import ConfigurableResource
from dagster._annotations import public


class ReplicationResource(ConfigurableResource):
    @public
    def run(
        self, replication_project: ReplicationProject
    ) -> Iterator[AssetMaterialization]:
        results = replicate(Path(replication_project.replication_configuration_yaml))
        for table in results:
            if table.get("status") == "SUCCESS":
                # NOTE: this assumes that the table name is the same as the asset key
                yield AssetMaterialization(
                    asset_key=str(table.get("name")), metadata=table
                )
```

これで、このリソースを使用するように `custom_replication_assets` インスタンスをリファクタリングできます:

```python
@custom_replication_assets(
    replication_project=replication_project,
    name="my_custom_replication_assets",
    group_name="replication",
)
def my_assets(replication_resource: ReplicationProject):
    replication_resource.run(replication_project)
```

## Step 4: トランスレーターの使用

[ステップ 2](#step-2-implementation) の最後で、エンド ユーザーはデコレータによって生成されたアセット キーなどのアセット属性をカスタマイズできないことを説明しました。トランスレータ クラスは、このロジックを定義するための推奨される方法であり、ツールの概念 (テーブル名など) を Dagster の対応する概念 (アセット キーなど) に変換するために使用される既定のメソッドをオーバーライドするオプションをユーザーに提供します。

まず、テーブル仕様を Dagster アセット キーにマップするトランスレータ メソッドを定義します。

:::note
実際の統合では、依存関係、グループ名、メタデータなどのすべての共通属性に対してメソッドを定義する必要があります。
:::

```python
from dagster import AssetKey, _check as check

from dataclasses import dataclass


@dataclass
class ReplicationTranslator:
    @public
    def get_asset_key(self, table_definition: Mapping[str, str]) -> AssetKey:
        return AssetKey(str(table_definition.get("name")))
```

次に、`AssetSpec` で `key` を定義するときにトランスレータを使用するように `custom_replication_assets` を更新します。

:::note
この機会を利用して、レプリケーション プロジェクトとトランスレータ インスタンスも `AssetSpec` メタデータに含めたことに注意してください。これは、これらのオブジェクトを一度定義して、アセットのコンテキストでアクセスできるようにするため、このアプローチでよく使用される回避策です。
:::

```python
def custom_replication_assets(
    *,
    replication_project: ReplicationProject,
    name: Optional[str] = None,
    group_name: Optional[str] = None,
    translator: Optional[ReplicationTranslator] = None,
) -> Callable[[Callable[..., Any]], AssetsDefinition]:
    project = replication_project.load()

    translator = (
        check.opt_inst_param(translator, "translator", ReplicationTranslator)
        or ReplicationTranslator()
    )

    return multi_asset(
        name=name,
        group_name=group_name,
        specs=[
            AssetSpec(
                key=translator.get_asset_key(table),
                metadata={
                    "replication_project": project,
                    "replication_translator": translator,
                },
            )
            for table in project.get("tables")
        ],
    )
```

最後に、メタデータで提供されるトランスレータとプロジェクトを使用するようにリソースを更新する必要があります。メタデータからオブジェクトを取得するときにオブジェクトのタイプが適切であることを確認するために、`dagster._check` によって提供される `check` メソッドを使用しています。

これで、アセットのマテリアライゼーションを生成するときに同じ `translator.get_asset_key` を使用できるようになりました。これにより、アセットの宣言がアセットのマテリアライゼーションと一致することが保証されます:

```python
class ReplicationResource(ConfigurableResource):
    @public
    def run(self, context: AssetExecutionContext) -> Iterator[AssetMaterialization]:
        metadata_by_key = context.assets_def.metadata_by_key
        first_asset_metadata = next(iter(metadata_by_key.values()))

        project = check.inst(
            first_asset_metadata.get("replication_project"),
            ReplicationProject,
        )

        translator = check.inst(
            first_asset_metadata.get("replication_translator"),
            ReplicationTranslator,
        )

        results = replicate(Path(project.replication_configuration_yaml))
        for table in results:
            if table.get("status") == "SUCCESS":
                yield AssetMaterialization(
                    asset_key=translator.get_asset_key(table), metadata=table
                )
```

## 結論

このガイドでは、カスタム マルチアセット デコレータ、ツール ロジックをカプセル化するリソース、仕様を Dagster コンセプトに変換するロジックを定義するトランスレータを定義する方法について説明しました。

このアプローチで統合を定義することは、Dagster の全体的な開発パラダイムとうまく一致しており、多くのアセットを生成するツールに適しています。

コード全体を以下に示します:

<CodeExample
  path="docs_snippets/docs_snippets/guides/tutorials/multi-asset-integration/integration.py"
  language="python"
/>
