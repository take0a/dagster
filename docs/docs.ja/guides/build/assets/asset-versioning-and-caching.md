---
title: "アセットのバージョン管理とキャッシュ"
sidebar_position: 1300
---

import Beta from '@site/docs/partials/\_Beta.md';

<Beta />

このガイドでは、アセットのメモ化可能なグラフを構築する方法を説明します。メモ化可能なアセットは、不要な再計算を回避し、開発者のワークフローを高速化し、計算リソースを節約するのに役立ちます。

## コンテクスト

結果が前回の資産の実現結果と同じになる場合は、資産の実現に時間を費やす理由はありません。

Dagster のバージョン管理システムは、アセットを実体化することで異なる結果が生成されるかどうかを事前に判断するのに役立ちます。このシステムは、次の条件を満たす限り、アセットの実体化の結果は変わらないという考えに基づいています。

- 使用されるコードは、アセットが最後に実現されたときと同じコードである
- 入力データは、アセットが最後に生成されたときと同じ入力データである

Dagster には、各マテリアライゼーションに使用されるコードと入力データを表す 2 つのバージョン管理概念があります:

- **Code version.** アセットを計算するコードのバージョンを表す文字列。これは、<PyObject section="assets" module="dagster" object="asset" decorator /> の `code_version` 引数です
- **Data version.** アセットによって表されるデータのバージョンを表す文字列。これは、`DataVersion` オブジェクトとして表されます。
{/* TODO link `DataVersion` to API docs */}

コードとデータのバージョンを追跡することで、Dagster はマテリアライゼーションによって基になる値が変更されるかどうかを予測できます。これにより、Dagster は冗長なマテリアライゼーションをスキップし、代わりに以前に計算された値を返すことができます。より技術的な用語で言えば、Dagster はアセットに対して限定的な形式の [メモ化](https://en.wikipedia.org/wiki/Memoization) を提供します。つまり、最後に計算されたアセット値は常にキャッシュされます。

計算コストの高いデータパイプラインでは、このアプローチによって大きなメリットが得られます。

## Step one: データのバージョンを理解する

デフォルトでは、Dagster はアセットの各マテリアライゼーションのデータ バージョンを自動的に計算します。これは、コード バージョンと入力アセットのデータ バージョンをハッシュすることによって行われます。

ハードコードされた数値を返す簡単なアセットから始めましょう:


<CodeExample path="docs_snippets/docs_snippets/guides/dagster/asset_versioning_and_caching/vanilla_asset.py" />

次に、Dagster UI を起動します：

```shell
dagster dev
```

**Asset catalog** に移動し、**Materialize** をクリックしてアセットをマテリアライズします。

次に、Asset Catalog の "Events" タブにあるマテリアライゼーションのエントリを確認します。マテリアライゼーションの詳細の **システム タグ** セクションにある 2 つのハッシュ (`code_version` と `data_version`) に注意してください:

![Simple asset data version](/images/guides/build/assets/asset-versioning-and-caching/simple-asset-in-catalog.png)

表示されるコードバージョンは、このマテリアライゼーションを生成した実行の実行 ID のコピーです。`a_number` にはユーザー定義の `code_version` がないため、Dagster は実行ごとに異なるコードバージョンを想定し、それを実行 ID で表します。

`data_version` も Dagster によって生成されます。これは、コードバージョンと入力のデータ バージョンを組み合わせたハッシュです。`a_number` には入力がないため、この場合、データバージョンはコードバージョンのみのハッシュになります。

アセットを再度マテリアライズすると、コードバージョンとデータバージョンの両方が変更されていることがわかります。コードバージョンは新しい実行の ID になり、データバージョンは新しいコードバージョンのハッシュになります。

明示的なコード バージョンを設定してこの状況を改善しましょう。アセットに `code_version` を追加します:

<CodeExample path="docs_snippets/docs_snippets/guides/dagster/asset_versioning_and_caching/vanilla_asset_with_code_version.py" />

次に、アセットをマテリアライズします。ユーザー定義のコード バージョン `v1` は、最新のマテリアライズに関連付けられます:

![Simple asset data version with code version](/images/guides/build/assets/asset-versioning-and-caching/simple-asset-with-code-version-in-catalog.png)

次に、コードを更新して、コードが変更されたことを Dagster に通知します。これを行うには、`code_version` 引数を変更します:

<CodeExample path="docs_snippets/docs_snippets/guides/dagster/asset_versioning_and_caching/vanilla_asset_with_code_version_v2.py" />

変更を有効にするには、**Reload definitions** をクリックします。

![Simple asset data version with code version](/images/guides/build/assets/asset-versioning-and-caching/simple-asset-with-code-version-in-asset-graph.png)

アセットには、最後にマテリアライズされてからコード バージョンが変更されたことを示すラベルが付きました。これは、アセット グラフとサイドバーの両方で確認できます。サイドバーでは、選択したノードの最後のマテリアライズに関する詳細が表示されます。`versioned_number` の最後のマテリアライズに関連付けられたコード バージョンは `v1` ですが、現在のコード バージョンは `v2` であることがわかります。これは、インジケーター タグの `(i)` アイコンにマウスを移動したときに表示されるツールヒントでも説明されています。

:::note
The "Unsynced" label can appear for three reasons:

- The code version of the asset is changed.
- The dependencies of the asset have changed (a dependency was added or removed).
- The data version of a parent asset has changed due to a new materialization. Note that if you are not using code versions, all new materialization of a dependency will change the data version. The UI will in this case report a "new materialization" rather than a "new data version".
:::

`versioned_number` アセットを最新の状態にするには、再度マテリアライズする必要があります。**Materialize** ボタンの右側にあるトグルをクリックして、**Materialize unsynced** オプションを表示します。これをクリックすると、`versioned_number` をマテリアライズして、変更されたコード バージョンが伝播されます。これにより、サイドバーに表示されている最新のマテリアライズ `code_version` が `v2` に更新され、アセットが最新の状態になります。

## Step two: 依存関係のあるデータバージョン

依存関係がある場合、変更の追跡はより強力になります。最初のアセットの下流にアセットを追加してみましょう:

<CodeExample path="docs_snippets/docs_snippets/guides/dagster/asset_versioning_and_caching/dependencies_code_version_only.py" />

Dagster UI で、**Reload definitions** をクリックします。`multiplied_number` アセットは、**Never materialized** としてマークされます。

Once again, click the toggle to the right side of the **Materialize** button to display the **Materialize unsynced** option. This will also provide the option to materialize "Never materialized" assets. This time, you will not see `versioned_number` as an option, because the system knows that `versioned_number` is up to date. Confirm the materialization of `multiplied_number`.

![Run event log](/images/guides/build/assets/asset-versioning-and-caching/materialize-stale-event-log.png)

それでは、`versioned_number` アセットを更新しましょう。具体的には、戻り値とコード バージョンを変更します:

<CodeExample path="docs_snippets/docs_snippets/guides/dagster/asset_versioning_and_caching/dependencies_code_version_only_v2.py" />

以前と同様に、これにより `versioned_number` は、最新のマテリアライズ以降にコード バージョンが変更されたことを示すラベルを取得します。 You might think that, since `multiplied_number` depends on `versioned_number`, it would also appear to be "Unsynced". However, "Unsynced" status is _not_ transitive in Dagster. `multiplied_number` will only appear to be "Unsynced" if its last materialization is against an outdated version of `versioned_number`. Materialize `versioned_number` and you will see that `multiplied_number` then becomes "Unsynced", with a reported reason of "Upstream data version change".

![Dependencies code version only](/images/guides/build/assets/asset-versioning-and-caching/dependencies-code-version-only.png)

両方のアセットを最新の状態にするには、`multiplied_number` を実体化します。

## Step three: 独自のデータバージョンを計算する

データ バージョンは、アセットが表す値、つまりそのマテリアライゼーション関数の出力の指紋のようなものです。したがって、データ バージョンは、マテリアライゼーション関数の可能な戻り値に 1 対 1 で対応する必要があります。Dagster は、コード バージョンと入力データ バージョンをハッシュすることで、データ バージョンを自動生成します。多くの場合、これは上記の基準を満たしますが、別のアプローチが必要になる場合もあります。

たとえば、マテリアライズ関数にランダム要素が含まれている場合、同じコードで同じ入力を持つアセットを複数回マテリアライズすると、異なる出力に対して同じデータ バージョンが生成されます。逆に、ソースハッシュなどの自動化されたアプローチでコード バージョンを生成する場合、外観上のリファクタリング後にアセットをマテリアライズすると、異なるデータ バージョン (コード バージョンから派生) が生成されますが、出力は同じになります。

Dagster は、ユーザー コードが独自のデータ バージョンを提供できるようにすることで、このようなシナリオや同様のシナリオに対応しています。これを行うには、<PyObject section="ops" module="dagster" object="Output" /> オブジェクトで返されたアセット値とともにデータ バージョンを含めます。これを行うには、`versioned_number` を更新します。簡単にするために、文字列化された戻り値をデータ バージョンとして使用します:

<CodeExample path="docs_snippets/docs_snippets/guides/dagster/asset_versioning_and_caching/manual_data_versions_1.py" />

If you reload definitions, as before, you will see `versioned_number` gets an "Unsynced" label to indicate the latest materialization is out of sync with its code version. We also know that if we materialize `versioned_number`, `multiplied_number` will become unsynced. Let's re-materialize them both in one run to avoid that intermediate state. Notice the `DataVersion` of `versioned_number` is now `20`:

![Manual data versions 1](/images/guides/build/assets/asset-versioning-and-caching/manual-data-versions-1.png)

`versioned_number` を再度更新して、返される値を変更せずに表面的なリファクタリングをシミュレートしてみましょう。コード バージョンを `v5` に上げ、`20` を `10 + 10` に変更します:

<CodeExample path="docs_snippets/docs_snippets/guides/dagster/asset_versioning_and_caching/manual_data_versions_2.py" />

Once again, `versioned_asset` will have an "Unsynced" label to indicate the change in the code version.

`versioned_number` のみがマテリアライズされた場合に何が起こるかを見てみましょう。アセットグラフでそれを選択し、**Materialize selected** をクリックします。サイドバーには、最新のマテリアライズの code_version が `v5` になり、データバージョンが再び `20` になっていることが示されています。

![Manual data versions 2](/images/guides/build/assets/asset-versioning-and-caching/manual-data-versions-2.png)

Notice that, unlike the last time we materialized `versioned_number`, `multiplied_number` does not have an "Unsynced" label! Here's what happened: the new materialization of `versioned_number` with the explicitly supplied data version supersedes the code version of `versioned_number`. Dagster then compared the data version of `versioned_number` last used to materialize `multiplied_number` to the current data version of `versioned_number`. Since this comparison shows that the data version of `versioned_number` hasn't changed, Dagster knows that the change to the code version of `versioned_number` doesn't affect `multiplied_number`.

`versioned_number` が Dagster によって生成されたデータ バージョンを使用していた場合、戻り値は変更されなかったにもかかわらず、更新されたコード バージョンにより `versioned_number` のデータ バージョンが変更されます。`multiplied_number` には、上流のデータ バージョンが変更されたことを示すラベルが付きます。

## Step four: ソースアセットを含むデータバージョン

現実の世界では、データ パイプラインは外部の上流データに依存します。これまでのガイドでは、外部データは使用していません。グラフのルートにあるアセット内のハードコードされたデータを置き換え、そのデータのバージョンの代わりとしてコード バージョンを使用しています。これよりも優れた方法があります。

Dagster の外部データ ソースは、<PyObject section="assets" module="dagster" object="SourceAsset" pluralize /> によってモデル化されます。`SourceAsset` を監視可能にすることで、バージョン管理を追加できます。監視可能なソース アセットには、データ バージョンを計算して返すユーザー定義関数があります。

`input_number` という <PyObject section="assets" module="dagster" object="observable_source_asset" decorator="true" /> を追加してみましょう。これは、パイプラインの上流の外部プロセスによって書き込まれたファイルを表します:

<CodeExample path="docs_snippets/docs_snippets/guides/dagster/asset_versioning_and_caching/input_number.txt" />

`input_number` 関数の本体は、ファイルの内容のハッシュを計算し、それを `DataVersion` として返します。`input_number` を `versioned_number` の上流依存関係として設定し、`versioned_number` がファイルから読み取った値を返すようにします:

<CodeExample path="docs_snippets/docs_snippets/guides/dagster/asset_versioning_and_caching/observable_source_asset_path_with_non_argument_deps.py" />

監視可能なソース アセットをアセット グラフに追加すると、新しいボタン **Observe sources** が表示されます:

![Source asset in graph](/images/guides/build/assets/asset-versioning-and-caching/source-asset-in-graph.png)

このボタンをクリックすると、`input_number` を観測する関数の実行が開始されます。`input_number` のアセットカタログのエントリを見てみましょう:

![Source asset in catalog](/images/guides/build/assets/asset-versioning-and-caching/source-asset-in-catalog.png)

ここで計算した `data_version` をメモしてください。

また、`versioned_number` と `multiplied_number` には、新しい上流依存関係があることを示すラベルが付いています (監視可能なソースアセットをグラフに追加したため)。**Materialize all** をクリックして、最新の状態にします。

最後に、外部プロセスのアクティビティをシミュレートするためにファイルを手動で変更してみましょう。`input_number.txt` の内容を変更します:

<CodeExample path="docs_snippets/docs_snippets/guides/dagster/asset_versioning_and_caching/input_number_v2.txt" />

**Observe Sources** ボタンをもう一度クリックすると、下流のアセットに上流のデータが変更されたことを示すラベルが再び表示されます。観察の実行により、コンテンツが変更されたため、`input_number` の新しいデータ バージョンが生成されました。
