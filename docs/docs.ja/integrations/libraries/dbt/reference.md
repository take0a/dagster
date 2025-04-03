---
title: 'dagster-dbt 統合リファレンス'
description: Dagster can orchestrate dbt alongside other technologies.
---

:::note

dbt Cloud をお使いですか? [dbt Cloud with Dagster ガイド](/integrations/libraries/dbt/dbt-cloud) をご覧ください。

:::

このリファレンスでは、[`dagster-dbt` 統合ライブラリ](/api/python-api/libraries/dagster-dbt) を使用して、Dagster の [ソフトウェア定義アセット](/guides/build/assets/) フレームワークを通じて dbt モデルを操作する方法について概要を説明します。

実装の詳細な手順については、[dbt を Dagster アセット定義と共に使用するチュートリアル](/integrations/libraries/dbt/using-dbt-with-dagster) を参照してください。

## 関連API

| Name                                                                                                    | Description                                                                                                                                                                     |
| ------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| [`dagster-dbt project scaffold`](/api/python-api/libraries/dagster-dbt#scaffold)                        | 既存の dbt プロジェクト用に新しい Dagster プロジェクトを初期化する CLI コマンド。                                                         |
| <PyObject section="libraries" module="dagster_dbt" object="dbt_assets" decorator />                     | dbt マニフェストで定義された dbt モデルの Dagster アセットを定義するために使用されるデコレータ。                                                        |
| <PyObject section="libraries" module="dagster_dbt" object="DbtCliResource" />                           | dbt CLI コマンドを実行するために使用される Dagster リソースを定義するクラス。                                          |
| <PyObject section="libraries" module="dagster_dbt" object="DbtCliInvocation" />                         |呼び出された dbt コマンドの表現を定義するクラス。          |
| <PyObject section="libraries" module="dagster_dbt" object="DbtProject" />                               | 依存関係の管理と `manifest.json` の準備を支援する dbt プロジェクトの表現と関連設定を定義するクラス。|
| <PyObject section="libraries" module="dagster_dbt" object="DagsterDbtTranslator" />                     | Dagster アセット メタデータを dbt マニフェストから取得する方法をカスタマイズするためにオーバーライドできるクラス。              |
| <PyObject section="libraries" module="dagster_dbt" object="DagsterDbtTranslatorSettings" />             | dbt プロジェクトで Dagster 機能を有効にする設定を持つクラス。  |
| <PyObject section="libraries" module="dagster_dbt" object="DbtManifestAssetSelection" />                |dbt マニフェストと dbt 選択文字列からのアセットの選択を定義するクラス。          |
| <PyObject section="libraries" module="dagster_dbt" object="build_dbt_asset_selection" />                | dbt マニフェストと dbt 選択文字列から <PyObject section="libraries" module="dagster_dbt" object="DbtManifestAssetSelection" /> を構築するヘルパー メソッド。  |
| <PyObject section="libraries" module="dagster_dbt" object="build_schedule_from_dbt_selection" />        | dbt マニフェスト、dbt 選択文字列、および cron 文字列から <PyObject section="schedules-sensors" module="dagster" object="ScheduleDefinition" /> を構築するヘルパー メソッド。|
| <PyObject section="libraries" module="dagster_dbt" object="get_asset_key_for_model" />                  |dbt モデルの <PyObject section="assets" module="dagster" object="AssetKey" /> を取得するヘルパー メソッド。   |
| <PyObject section="libraries" module="dagster_dbt" object="get_asset_key_for_source" />                 | 単一のテーブルを持つ dbt ソースの <PyObject section="assets" module="dagster" object="AssetKey" /> を取得するヘルパー メソッド。  |
| <PyObject section="libraries" module="dagster_dbt" object="get_asset_keys_by_output_name_for_source" /> | 複数のテーブルを持つ dbt ソースの <PyObject section="assets" module="dagster" object="AssetKey" pluralize /> を取得するヘルパー メソッド。    |

## dbt モデルと Dagster アセット定義

Dagster の [アセット定義](/guides/build/assets/) には、dbt モデルとの類似点がいくつかあります。アセット定義には、アセット キー、一連のアップストリーム アセット キー、およびアップストリーム依存関係からアセットを計算する操作が含まれます。dbt プロジェクトで定義されたモデルは、Dagster アセット定義として解釈できます。

- dbt モデルのアセット キーは (デフォルトでは) モデルの名前です。
- dbt モデルのアップストリーム依存関係は、モデルの定義内で `ref` または `source` 呼び出しを使用して定義されます。
- アップストリーム依存関係からアセットを計算するために必要な計算は、モデルの定義内の SQL です。

これらの類似点により、dbt モデルをアセット定義として操作することが自然になります。コードで dbt モデルとアセット定義を見てみましょう:

![Comparison of a dbt model and Dagster asset in code](/images/integrations/dbt/using-dbt-with-dagster/asset-dbt-model-comparison.png)

この例では、次の処理が行われます:

- 最初のコード ブロックは **dbt モデル** です
- dbt モデルはファイル名を使用して命名されるため、このモデルの名前は `orders` です
- このモデルのデータは `raw_orders` という名前の依存関係から取得されます
- 2 番目のコード ブロックは **Dagster アセット** です
- アセット キーは dbt モデルの名前 `orders` に対応します
- `raw_orders` はアセットへの引数として提供され、依存関係として定義されます

## dbt プロジェクトから Dagster プロジェクトを構築する

:::note

この概念を実際の状況で確認するには、[dbt と Dagster のチュートリアルのパート 2](/integrations/libraries/dbt/using-dbt-with-dagster/load-dbt-models) を参照してください。

:::

[`dagster-dbt プロジェクト スキャフォールド`](/api/python-api/libraries/dagster-dbt#scaffold) コマンドライン インターフェースを使用して、dbt プロジェクトをラップする Dagster プロジェクトを作成できます。

```shell
dagster-dbt project scaffold --project-name project_dagster --dbt-project-dir path/to/dbt/project
```

これにより、現在のディレクトリ内に `project_dagster/` というディレクトリが作成されます。`project_dagster/` ディレクトリには、`--dbt-project-dir` で定義されたパスで dbt プロジェクトをロードする Dagster プロジェクトを定義する一連のファイルが含まれます。dbt プロジェクトへのパスには `dbt_project.yml` が含まれている必要があります。

## dbt プロジェクトから dbt モデルをロードする

:::note

この概念を実際の状況で確認するには、[dbt と Dagster のチュートリアルのパート 2](/integrations/libraries/dbt/using-dbt-with-dagster/load-dbt-models) を参照してください。

:::

`dagster-dbt` ライブラリは、dbt モデルの Dagster アセットを定義するための <PyObject section="libraries" module="dagster_dbt" object="dbt_assets" decorator /> を提供します。dbt プロジェクトの表現を解析するには、dbt プロジェクトから [dbt マニフェスト](https://docs.getdbt.com/reference/artifacts/manifest-json)、つまり `manifest.json` を作成する必要があります。

マニフェストは 2 つの方法で作成できます:

1. **実行時**: Dagster 定義がロードされるときに dbt マニフェストが生成されます。または

2. **ビルド時**: Dagster 定義がロードされる前に dbt マニフェストが生成され、Python パッケージの一部として含まれます。

Dagster プロジェクトを本番環境にデプロイする場合、**ビルド時にマニフェストを生成することを推奨します**。これにより、Dagster コードが実行されるたびに dbt プロジェクトを再コンパイルするオーバーヘッドを回避できます。`manifest.json` は、Dagster コードの Python パッケージにプリコンパイルして含める必要があります。

マニフェスト ファイルの作成を処理する最も簡単な方法は、<PyObject section="libraries" object="DbtProject" module="dagster_dbt" /> を使用することです。

[`dagster-dbt project scaffold`](/api/python-api/libraries/dagster-dbt#scaffold) コマンドによって作成された Dagster プロジェクトでは、マニフェストの作成は開発中の実行時に処理されます:

<CodeExample
  path="docs_snippets/docs_snippets/integrations/dbt/dbt.py"
  startAfter="start_compile_dbt_manifest_with_dbt_project"
  endBefore="end_compile_dbt_manifest_with_dbt_project"
/>

マニフェスト パスには、`my_dbt_project.manifest_path` でアクセスできます。

ローカルで開発する場合は、次のコマンドを実行して、dbt および Dagster プロジェクトの実行時にマニフェストを生成できます:

```shell
dagster dev
```

運用環境では、プリコンパイルされたマニフェストを使用する必要があります。<PyObject section="libraries" object="DbtProject" module="dagster_dbt" /> を使用すると、CI/CD ワークフローで [`dagster-dbt project prepare-and-package`](/api/python-api/libraries/dagster-dbt#prepare-and-package) コマンドを実行して、ビルド時にマニフェストを作成できます。詳細については、[dbt プロジェクトを使用した Dagster プロジェクトのデプロイ](#deploying-a-dagster-project-with-a-dbt-project) セクションを参照してください。

## dbt プロジェクトを使用して Dagster プロジェクトをデプロイする

:::note

**推奨事項について質問がある場合や、追加したい内容がある場合は、**

[GitHub ディスカッション](https://github.com/dagster-io/dagster/discussions/13767) に参加して、dbt プロジェクトで Dagster コードをデプロイする方法を共有してください。

:::

Dagster プロジェクトを本番環境にデプロイする場合、dbt コマンドを実行できるように、dbt プロジェクトが Dagster プロジェクトと一緒に存在している必要があります。そのため、継続的インテグレーションと継続的デプロイメント (CI/CD) ワークフローを設定して、dbt プロジェクトを Dagster プロジェクトとパッケージ化することをお勧めします。

### 別の Git リポジトリから dbt プロジェクトをデプロイする

Dagster プロジェクトを dbt プロジェクトとは別の Git リポジトリで管理している場合は、CI/CD ワークフローに次の手順を含める必要があります。

Dagster プロジェクトの CI/CD ワークフロー:

1. CI/CD 環境に dbt プロジェクトに必要なシークレットをすべて含めます。

2. dbt プロジェクト リポジトリを Dagster プロジェクトのサブディレクトリとしてクローンします。

3. `dagster-dbt project prepare-and-package --file path/to/project.py` を実行して、次の操作を実行します。
- dbt プロジェクトの依存関係をビルドします。
- Dagster プロジェクト用の dbt マニフェストを作成します。
- dbt プロジェクトをパッケージ化します。

dbt プロジェクトの CI/CD ワークフローで、dbt プロジェクトが変更されたときに Dagster プロジェクトのデプロイメントをトリガーするディスパッチ アクションを設定します。

### モノレポから dbt プロジェクトをデプロイする

:::note

[Dagster+](https://dagster.io/cloud) を使用すると、このオプションが効率化されます。dbt ユーザー向けの Dagster+ オンボーディングの一環として、既存の dbt プロジェクト リポジトリに Dagster プロジェクトを自動的に作成できます。

:::

Dagster プロジェクトを dbt プロジェクトと同じ Git リポジトリで管理している場合は、CI/CD ワークフローに次の手順を含める必要があります。

Dagster および dbt プロジェクトの CI/CD ワークフロー:

1. CI/CD 環境に dbt プロジェクトに必要なシークレットをすべて含めます。

2. `dagster-dbt project prepare-and-package --file path/to/project.py` を実行して、次の操作を行います。
- dbt プロジェクトの依存関係をビルドします。
- Dagster プロジェクトの dbt マニフェストを作成します。
- dbt プロジェクトをパッケージ化します。

## ブランチ展開で dbt defer を活用する

:::note

この機能を使用するには、CI/CD で `DAGSTER_BUILD_STATEDIR` 環境変数を設定する必要があります。Dagster+ の CI/CD で必要な環境変数の詳細については、[こちら](/dagster-plus/features/ci-cd/configuring-ci-cd) を参照してください。

:::

`state_path` を <PyObject section="libraries" object="DbtProject" module="dagster_dbt" /> に渡すことで、[dbt defer](https://docs.getdbt.com/reference/node-selection/defer) を活用できます。これは、開発で行われた最近の変更を、本番環境の dbt プロジェクトの状態に対してテストする場合に便利です。`dbt defer` を使用すると、上流の親を最初にビルドしなくても、開発と本番環境の間で変更されたモデルまたはテストのサブセットを実行できます。

実際には、これは Dagster+ のブランチ デプロイメントと組み合わせて、ブランチで行われた変更をテストする場合に最も便利です。これは、CI/CD ファイルと Dagster コードを更新することで実行できます。

まず、CI/CD ファイルを確認しましょう。本番環境とブランチ デプロイメントを管理するために、1 つまたは 2 つの CI/CD ファイルがある場合があります。これらのファイルで、dbt プロジェクトに関連する手順を見つけます。詳細については、[dbt プロジェクトを使用した Dagster プロジェクトのデプロイ](#deploying-a-dagster-project-with-a-dbt-project) セクションを参照してください。

dbt 手順が見つかったら、次の手順を追加して、dbt プロジェクトの状態を管理します。

```bash
dagster-cloud ci dagster-dbt project manage-state --file path/to/project.py
```

`dagster-cloud ci dagster-dbt project manage-state` CLI コマンドは、`manifest.json` ファイルを本番ブランチから取得し、それを状態ディレクトリに保存して、`dbt defer` コマンドを実行します。

実際には、このコマンドは `manifest.json` ファイルを本番ブランチから取得し、`path/to/project.py` にある DbtProject の `state_path` に設定された状態ディレクトリに追加します。 その後、本番の `manifest.json` ファイルを遅延 dbt アーティファクトとして使用できます。

dagster-cloud CLI を使用して dbt プロジェクトの状態を管理するために CI/CD ファイルが更新されたので、状態ディレクトリを DbtProject に渡すように Dagster コードを更新する必要があります。

`state_path` を `DbtProject` オブジェクトに渡すように Dagster コードを更新します。 `state_path` に渡される値は、dbt プロジェクト ディレクトリからの相対パスで、dbt アーティファクトの状態ディレクトリへのパスである必要があります。以下のコードでは、`state_path` を 'state/' に設定しています。このディレクトリがプロジェクト構造に存在しない場合は、Dagster によって作成されます。

また、`@dbt_assets` 定義の dbt コマンドを更新して、`get_defer_args` を使用して defer 引数を渡します。

<CodeExample
  startAfter="start_use_dbt_defer_with_dbt_project"
  endBefore="end_use_dbt_defer_with_dbt_project"
  path="docs_snippets/docs_snippets/integrations/dbt/dbt.py"
/>

## `@dbt_assets` で設定を使用する

Dagster ソフトウェア定義アセットと同様に、`@dbt_assets` は構成システムを使用して [実行構成](/guides/operate/configuration/run-configuration) を有効にできます。これにより、ジョブの実行時にパラメータを提供できます。

dbt のコンテキストでは、特定のユースケースでコマンドまたはフラグを実行する場合に役立ちます。たとえば、場合によっては、dbt コマンドに [--full-refresh フラグ](https://docs.getdbt.com/reference/resource-configs/full_refresh) を追加する必要があります。構成システムを使用すると、`@dbt_assets` オブジェクトを簡単に変更して、このユースケースをサポートできます。

<CodeExample
  startAfter="start_config_dbt_assets"
  endBefore="end_config_dbt_assets"
  path="docs_snippets/docs_snippets/integrations/dbt/dbt.py"
/>

`@dbt_assets` オブジェクトが更新されたので、実行構成をジョブに渡すことができます。

<CodeExample
  startAfter="start_config_dbt_job"
  endBefore="end_config_dbt_job"
  path="docs_snippets/docs_snippets/integrations/dbt/dbt.py"
/>

上記の例では、アセットをマテリアライズするときに、dbt build コマンドで `--full-refresh` フラグを使用するようにジョブが構成されています。

## dbt ジョブのスケジュール設定

dbt アセットを取得したら、スケジュールに従ってこれらのアセットの選択を具体化するジョブを定義できます。

### dbt アセットのみを含むジョブのスケジュール

この例では、<PyObject section="libraries" module="dagster_dbt" object="build_schedule_from_dbt_selection" /> 関数を使用して、ジョブ `daily_dbt_models` と、このジョブを 1 日に 1 回実行するスケジュールを作成します。[dbt の選択構文](https://docs.getdbt.com/reference/node-selection/syntax#how-does-selection-work) を使用して、実行するモデルのセットを定義します。この場合は、タグ `daily` が付いたモデルのみを選択します。

<CodeExample
  startAfter="start_schedule_assets_dbt_only"
  endBefore="end_schedule_assets_dbt_only"
  path="docs_snippets/docs_snippets/integrations/dbt/dbt.py"
/>

### dbt アセットと非 dbt アセットを含むジョブのスケジュール設定

多くの場合、dbt アセットを非 dbt アセットと一緒にスケジュールできると便利です。この例では、<PyObject section="assets" module="dagster" object="AssetSelection" /> を使用して dbt アセットの <PyObject section="libraries" module="dagster_dbt" object="build_dbt_asset_selection" /> を構築し、これらの dbt モデルのダウンストリームにあるすべてのアセット (dbt 関連かどうかに関係なく) を選択します。そこから、そのアセットの選択をターゲットとするジョブを作成し、毎日実行するようにスケジュールします。

<CodeExample
  startAfter="start_schedule_assets_dbt_downstream"
  endBefore="end_schedule_assets_dbt_downstream"
  path="docs_snippets/docs_snippets/integrations/dbt/dbt.py"
/>

スケジュールに従ってジョブを実行する方法の詳細については、[スケジュールのドキュメント](/guides/automate/schedules/)を参照してください。

## アセット定義属性の理解

Dagster では、各アセット定義に属性があります。Dagster は、dbt プロジェクトからロードされた各アセット定義に対してこれらの属性を自動的に生成します。これらの属性は、ユーザーがオプションで上書きできます。

- [アセットキーのカスタマイズ](#customizing-asset-keys)
- [グループ名のカスタマイズ](#customizing-group-names)
- [所有者のカスタマイズ](#customizing-owners)
- [説明のカスタマイズ](#customizing-descriptions)
- [メタデータのカスタマイズ](#customizing-metadata)
- [タグのカスタマイズ](#customizing-tags)
- [自動化条件のカスタマイズ](#customizing-automation-conditions)

### アセットキーのカスタマイズ {#customizing-asset-keys}

dbt モデル、シード、スナップショットの場合、デフォルトのアセット キーは、そのノードに構成されたスキーマとノードの名前を連結したものになります。

| dbt node type         | Schema    | Model name   | Resulting asset key |
| --------------------- | --------- | ------------ | ------------------- |
| model, seed, snapshot | `None`    | `MODEL_NAME` | `MODEL_NAME`        |
|                       | `SCHEMA`  | `MODEL_NAME` | `SCHEMA/MODEL_NAME` |
|                       | `None`    | my_model     | some_model          |
|                       | marketing | my_model     | marketing/my_model  |

dbt ソースの場合、デフォルトのアセット キーは、ソースの名前とソース テーブルの名前を連結したものになります。

| dbt node type | Source name   | Table name   | Resulting asset key      |
| ------------- | ------------- | ------------ | ------------------------ |
| source        | `SOURCE_NAME` | `TABLE_NAME` | `SOURCE_NAME/TABLE_NAME` |
|               | jaffle_shop   | orders       | jaffle_shop/orders       |

Dagster によって dbt アセット用に生成されたアセット キーをカスタマイズする方法は 2 つあります:

1. dbt ノードで [meta config](https://docs.getdbt.com/reference/resource-configs/meta) を定義する、または

2. カスタム <PyObject section="libraries" module="dagster_dbt" object="DagsterDbtTranslator" /> を実装して Dagster のアセット キー生成をオーバーライドする。

Dagster によって dbt ノード用に生成されたアセット キーをオーバーライドするには、dbt ノードの `.yml` ファイルで `meta` キーを定義します。次の例では、ソースとテーブルのアセット キーを `snowflake/jaffle_shop/orders` としてオーバーライドします。

```yaml
sources:
  - name: jaffle_shop
    tables:
      - name: orders
        meta:
          dagster:
            asset_key: ['snowflake', 'jaffle_shop', 'orders']
```

あるいは、dbt プロジェクト内のすべての dbt ノードのアセット キー生成をオーバーライドするには、カスタム <PyObject section="libraries" module="dagster_dbt" object="DagsterDbtTranslator" /> を作成し、<PyObject section="libraries" module="dagster_dbt" object="DagsterDbtTranslator.get_asset_key" /> を実装します。次の例では、デフォルトで生成されたアセット キーに `snowflake` プレフィックスを追加します:

<CodeExample
  startAfter="start_custom_asset_key_dagster_dbt_translator"
  endBefore="end_custom_asset_key_dagster_dbt_translator"
  path="docs_snippets/docs_snippets/integrations/dbt/dbt.py"
/>

### グループ名のカスタマイズ {#customizing-group-names}

dbt モデル、シード、スナップショットの場合、デフォルトの Dagster グループ名は、そのノードに対して定義された [dbt グループ](https://docs.getdbt.com/docs/build/groups) になります。

| dbt node type         | dbt group name | Resulting Dagster group name |
| --------------------- | -------------- | ---------------------------- |
| model, seed, snapshot | `GROUP_NAME`   | `GROUP_NAME`                 |
|                       | `None`         | `None`                       |
|                       | finance        | finance                      |

dbt アセット用に Dagster によって生成されたグループ名をカスタマイズする方法は 2 つあります:

1. dbt ノードで [meta config](https://docs.getdbt.com/reference/resource-configs/meta) を定義する、または

2. カスタム <PyObject section="libraries" module="dagster_dbt" object="DagsterDbtTranslator" /> を実装して Dagster のグループ名生成をオーバーライドする

dbt ノード用に Dagster によって生成されたグループ名をオーバーライドするには、dbt プロジェクト ファイル、dbt ノードのプロパティ ファイル、またはノードのファイル内構成ブロックで `meta` キーを定義します。次の例では、次のモデルの Dagster グループ名を `marketing` としてオーバーライドします。

```yaml
models:
  - name: customers
    config:
      meta:
        dagster:
          group: marketing
```

あるいは、dbt プロジェクト内のすべての dbt ノードの Dagster グループ名生成をオーバーライドするには、カスタム <PyObject section="libraries" module="dagster_dbt" object="DagsterDbtTranslator" /> を作成し、<PyObject section="libraries" module="dagster_dbt" object="DagsterDbtTranslator.get_group_name" /> を実装します。次の例では、すべての dbt ノードのグループ名として `snowflake` を定義します:

<CodeExample
  startAfter="start_custom_group_name_dagster_dbt_translator"
  endBefore="end_custom_group_name_dagster_dbt_translator"
  path="docs_snippets/docs_snippets/integrations/dbt/dbt.py"
/>

### 所有者のカスタマイズ {#customizing-owners}

dbt モデル、シード、スナップショットの場合、デフォルトの Dagster 所有者は、そのノードに定義されている [dbt グループ](https://docs.getdbt.com/docs/build/groups) に関連付けられた電子メールになります。

| dbt node type         | dbt group name | dbt group's email   | Resulting Dagster owner |
| --------------------- | -------------- | ------------------- | ----------------------- |
| model, seed, snapshot | `GROUP_NAME`   | `OWNER@DOMAIN.COM`  | `OWNER@DOMAIN.COM`      |
|                       | `GROUP_NAME`   | `None`              | `None`                  |
|                       | `None`         | `None`              | `None`                  |
|                       | finance        | `owner@company.com` | `owner@company.com`     |
|                       | finance        | `None`              | `None`                  |

Dagster によって dbt アセット用に生成されたアセット キーをカスタマイズする方法は 2 つあります:

1. dbt ノードで [meta config](https://docs.getdbt.com/reference/resource-configs/meta) を定義する、または

2. カスタム <PyObject section="libraries" module="dagster_dbt" object="DagsterDbtTranslator" /> を実装して、Dagster による所有者の生成をオーバーライドする

Dagster によって dbt ノード用に生成された所有者をオーバーライドするには、dbt プロジェクト ファイル、dbt ノードのプロパティ ファイル、またはノードのファイル内構成ブロックで `meta` キーを定義します。次の例では、次のモデルの Dagster 所有者を `owner@company.com` および `team:data@company.com` としてオーバーライドします:

```yaml
models:
  - name: customers
    config:
      meta:
        dagster:
          owners: ['owner@company.com', 'team:data@company.com']
```

あるいは、dbt プロジェクト内のすべての dbt ノードの所有者の Dagster 生成をオーバーライドするには、カスタム <PyObject section="libraries" module="dagster_dbt" object="DagsterDbtTranslator" /> を作成し、<PyObject section="libraries" module="dagster_dbt" object="DagsterDbtTranslator.get_group_name" /> を実装します。次の例では、`owner@company.com` と `team:data@company.com` をすべての dbt ノードの所有者として定義しています:

<CodeExample
  startAfter="start_custom_owners_dagster_dbt_translator"
  endBefore="end_custom_owners_dagster_dbt_translator"
  path="docs_snippets/docs_snippets/integrations/dbt/dbt.py"
/>

### 説明のカスタマイズ {#customizing-descriptions}

dbt モデル、シード、スナップショットの場合、デフォルトの Dagster の説明は dbt ノードの説明になります。

dbt プロジェクト内のすべての dbt ノードの Dagster の説明をオーバーライドするには、カスタム <PyObject section="libraries" module="dagster_dbt" object="DagsterDbtTranslator" /> を作成し、<PyObject section="libraries" module="dagster_dbt" object="DagsterDbtTranslator.get_description" /> を実装します。次の例では、dbt ノードの生の SQL を Dagster の説明として定義しています。

<CodeExample
  startAfter="start_custom_description_dagster_dbt_translator"
  endBefore="end_custom_description_dagster_dbt_translator"
  path="docs_snippets/docs_snippets/integrations/dbt/dbt.py"
/>

### メタデータのカスタマイズ {#customizing-metadata}

dbt モデル、シード、スナップショットの場合、デフォルトの Dagster 定義メタデータは、dbt ノードの宣言された列スキーマになります。

dbt プロジェクト内のすべての dbt ノードの Dagster 定義メタデータをオーバーライドするには、カスタム <PyObject section="libraries" module="dagster_dbt" object="DagsterDbtTranslator" /> を作成し、<PyObject section="libraries" module="dagster_dbt" object="DagsterDbtTranslator.get_metadata"/> を実装します。次の例では、<PyObject section="metadata" module="dagster" object="MetadataValue"/> を使用して、dbt ノードのメタデータを Dagster メタデータとして定義しています:

<CodeExample
  startAfter="start_custom_metadata_dagster_dbt_translator"
  endBefore="end_custom_metadata_dagster_dbt_translator"
  path="docs_snippets/docs_snippets/integrations/dbt/dbt.py"
/>

Dagster は、dbt 実行時に追加のメタデータを取得して、アセットのマテリアライゼーションに添付することもサポートしています。詳細については、[アセットのマテリアライゼーション メタデータのカスタマイズ](#customizing-asset-materialization-metadata) セクションを参照してください。

#### コード参照メタデータの添付

Dagster の dbt 統合では、dbt アセットをバックアップする SQL ファイルに [コード参照](/guides/build/assets/metadata-and-tags/index.md#source-code) メタデータを自動的に添付できます。この機能を有効にするには、<PyObject section="libraries" module="dagster_dbt" object="DagsterDbtTranslator" /> に渡される <PyObject section="libraries" module="dagster_dbt" object="DagsterDbtTranslatorSettings" /> で `enable_code_references` パラメータを `True` に設定します:

<CodeExample path="docs_snippets/docs_snippets/guides/dagster/code_references/with_dbt_code_references.py" />

### タグのカスタマイズ {#customizing-tags}

:::note

Dagster では、タグはキーと値のペアです。ただし、dbt では、タグは文字列です。このギャップを埋めるために、dbt タグ文字列が Dagster タグ キーとして使用され、Dagster タグ値は空の文字列 `""` に設定されます。Dagster でサポートされているタグ キー形式と一致しない dbt タグ (サポートされていない文字が含まれているなど) は、デフォルトで無視されます。

:::

dbt モデル、シード、スナップショットの場合、デフォルトの Dagster タグは dbt ノードの構成済みタグになります。

Dagster がサポートするタグ キー形式と一致しない dbt タグ (サポートされていない文字が含まれているなど) は無視されます。

dbt プロジェクト内のすべての dbt ノードの Dagster タグをオーバーライドするには、カスタム <PyObject section="libraries" module="dagster_dbt" object="DagsterDbtTranslator" /> を作成し、<PyObject section="libraries" module="dagster_dbt" object="DagsterDbtTranslator.get_tags" /> を実装します。次のコードでは、`foo=bar` 形式の dbt タグをキー/値のペアに変換します:

<CodeExample
  startAfter="start_custom_tags_dagster_dbt_translator"
  endBefore="end_custom_tags_dagster_dbt_translator"
  path="docs_snippets/docs_snippets/integrations/dbt/dbt.py"
/>

### 自動化条件のカスタマイズ {#customizing-automation-conditions}

dbt プロジェクト内の各 dbt ノードに対して生成された <PyObject section="assets" module="dagster" object="AutomationCondition" /> をオーバーライドするには、カスタム <PyObject section="libraries" module="dagster_dbt" object="DagsterDbtTranslator" /> を作成し、 <PyObject section="libraries" module="dagster_dbt" object="DagsterDbtTranslator.get_automation_condition" /> を実装します。次の例では、すべての dbt ノードの条件として <PyObject section="assets" object="AutomationCondition.eager" /> を定義します:

<CodeExample
  startAfter="start_custom_automation_condition_dagster_dbt_translator"
  endBefore="end_custom_automation_condition_dagster_dbt_translator"
  path="docs_snippets/docs_snippets/integrations/dbt/dbt.py"
/>

:::note

自動化条件を評価するには、[`default_automation_condition_sensor` が有効になっている](/guides/automate/declarative-automation/#using-declarative-automation)ことを確認します。

:::

## dbt モデル、コード バージョン、および「非同期」

Dagster では、各アセット定義に対して [`code_version`](/guides/build/assets/defining-assets#asset-code-versions) をオプションで指定できます。これは変更を追跡するために使用されます。dbt モデルから発生するアセットの `code_version` は、DBT モデルを定義する SQL のハッシュとして自動的に定義されます。これにより、UI のアセット グラフは「同期されていない」ステータスを使用して、最後にマテリアライズされてからどの dbt モデルに新しい SQL があるかを示すことができます。

## アセットのチェックとして dbt テストをロードする

:::note

**dbt のアセット チェックは、`dagster-dbt` 0.23.0 以降でデフォルトで有効になっています。**

完全な機能を使用するには、`dbt-core` 1.6 以降が必要です。

:::

Dagster は dbt テストを [アセット チェック](/guides/test/asset-checks) として読み込みます。

### 間接選択

Dagster は [dbt 間接選択](https://docs.getdbt.com/reference/global-configs/indirect-selection) を使用して dbt テストを選択します。デフォルトでは、Dagster は `DBT_INDIRECT_SELECTION` を設定しないため、Dagster によって選択されたテストのセットは dbt によって選択されたものと同じになります。必要に応じて、Dagster は `DBT_INDIRECT_SELECTION` を `empty` にオーバーライドして、dbt テストを明示的に選択します。例:

- dbt アセットをマテリアライズし、そのアセット チェックを除外する
- アセットをマテリアライズせずに dbt アセット チェックを実行する

### 特異テスト

Dagster は、汎用テストと特異テストの両方をアセット チェックとして読み込みます。特異テストが複数の dbt モデルに依存する場合は、dbt メタデータを使用して、ターゲットとする Dagster アセットを指定できます。これらのフィールドは、dbt [ref 関数](https://docs.getdbt.com/reference/dbt-jinja-functions/ref) の場合と同じように入力できます。設定は、特異テストの [config ブロック](https://docs.getdbt.com/reference/data-test-configs) で指定できます。

```sql
{{
    config(
        meta={
            'dagster': {
                'ref': {
                    'name': 'customers',
                    'package_name': 'my_dbt_assets'
                    'version': 1,
                },
            }
        }
    )
}}
```

Dagster がこのメタデータを読み取るには、`dbt-core` バージョン 1.6 以降が必要です。

このメタデータが提供されない場合、Dagster はテストをアセット チェックとして取り込みません。それでもテストは実行され、テスト結果とともに <PyObject section="assets" module="dagster" object="AssetObservation" /> イベントが発行されます。

### アセットチェックを無効にする

dbt テストをアセット チェックとしてモデリングすることを無効にできます。テストは引き続き実行され、<PyObject section="assets" module="dagster" object="AssetObservation"/> イベントとして発行されます。これを行うには、アセット チェックが無効になっている <PyObject section="libraries" module="dagster_dbt" object="DagsterDbtTranslator" /> を <PyObject section="libraries" module="dagster_dbt" object="DagsterDbtTranslatorSettings" /> で定義する必要があります。次の例では、<PyObject section="libraries" module="dagster_dbt" object="dbt_assets" decorator/> を使用するときにアセット チェックを無効にします:

<CodeExample
  startAfter="start_disable_asset_check_dagster_dbt_translator"
  endBefore="end_disable_asset_check_dagster_dbt_translator"
  path="docs_snippets/docs_snippets/integrations/dbt/dbt.py"
/>

## アセットマテリアライゼーションメタデータのカスタマイズ

Dagster は、dbt 実行時に追加のメタデータを取得して [マテリアライゼーション メタデータ](/guides/build/assets/metadata-and-tags/) として添付することをサポートしています。これは、モデルが再構築されて Dagster UI に表示されるたびに記録されます。

### 行数データの取得

:::note

この機能を使用するには、少なくとも `dagster>=0.17.6` および `dagster-dbt>=0.23.6` を使用する必要があります。

:::

Dagster は、dbt で生成されたテーブルの [行数](/guides/build/assets/metadata-and-tags/) を自動的に取得し、それらを [マテリアライゼーション メタデータ](/guides/build/assets/metadata-and-tags/) として出力して、Dagster UI に表示することができます。

行数は、dbt モデルの実行と並行して取得されます。この機能を有効にするには、`stream()` dbt CLI 呼び出しによって返された <PyObject section="libraries" object="core.dbt_cli_invocation.DbtEventIterator" module="dagster_dbt" /> で <PyObject section="libraries" object="core.dbt_cli_invocation.DbtEventIterator" module="dagster_dbt" /> を呼び出します:

<CodeExample
  startAfter="start_fetch_row_count"
  endBefore="end_fetch_row_count"
  path="docs_snippets/docs_snippets/integrations/dbt/dbt.py"
/>

dbt モデルがマテリアライズされると、メタデータ テーブルで行数データを表示できます。

### 列レベルのメタデータの取得

:::note

この機能を使用するには、少なくとも `dagster>=1.8.0` および `dagster-dbt>=0.24.0` を使用する必要があります。

:::

Dagster を使用すると、[列スキーマ](/guides/build/assets/metadata-and-tags/) や [列系統](/guides/build/assets/metadata-and-tags/) などの列レベルのメタデータを [マテリアライゼーション メタデータ](/guides/build/assets/metadata-and-tags/) として出力できます。

このメタデータを使用すると、dbt プロジェクトで記述されている列だけでなく、すべての列のドキュメントを Dagster で表示できます。

列レベルのメタデータは、dbt モデルの実行と並行して取得されます。この機能を有効にするには、`stream()` dbt CLI 呼び出しによって返された <PyObject section="libraries" object="core.dbt_cli_invocation.DbtEventIterator" module="dagster_dbt" /> で <PyObject section="libraries" object="core.dbt_cli_invocation.DbtEventIterator.fetch_column_metadata" module="dagster_dbt" displayText="fetch_column_metadata()" /> を呼び出します。

<CodeExample
  startAfter="start_fetch_column_metadata"
  endBefore="end_fetch_column_metadata"
  path="docs_snippets/docs_snippets/integrations/dbt/dbt.py"
/>

### メタデータ取得メソッドの作成

<PyObject section="libraries" object="core.dbt_cli_invocation.DbtEventIterator.fetch_column_metadata" module="dagster_dbt" displayText="fetch_column_metadata()" /> などのメタデータ取得メソッドは、<PyObject section="libraries" object="core.dbt_cli_invocation.DbtEventIterator.fetch_row_counts" module="dagster_dbt" displayText="fetch_row_counts()" /> などの他のメタデータ取得メソッドと連鎖させることができます。

<CodeExample
  startAfter="start_fetch_column_metadata_chain"
  endBefore="end_fetch_column_metadata_chain"
  path="docs_snippets/docs_snippets/integrations/dbt/dbt.py"
/>

## 依存関係の定義

- [上流の依存関係](#upstream-dependencies)
- [下流の依存関係](#downstream-dependencies)

### 上流の依存関係 {#upstream-dependencies}

#### アセットを dbt モデルの上流依存関係として定義する

Dagster を使用すると、既存のアセットを dbt モデルのアップストリーム依存関係として定義できます。たとえば、アセット キーが `upstream` である次のアセットがあるとします:

<CodeExample
  startAfter="start_upstream_dagster_asset"
  endBefore="end_upstream_dagster_asset"
  path="docs_snippets/docs_snippets/integrations/dbt/dbt.py"
/>

次に、ダウンストリーム モデルで、このソース データから選択できます。これにより、アップストリーム アセットと dbt モデル間の依存関係が定義されます:

```sql
select *
  from {{ source("dagster", "upstream") }}
 where foo=1
```

#### dbt ソースを Dagster アセットとして定義する

Dagster は、dbt プロジェクト自体から特定の dbt モデルの上流にあるアセットに関する情報を解析します。モデルが [dbt ソース](https://docs.getdbt.com/docs/building-a-dbt-project/using-sources) の下流にある場合は常に、そのソースは上流アセットとして解析されます。

たとえば、`sources.yml` ファイルで次のようにソースを定義したとします:

```yaml
sources:
  - name: jaffle_shop
    tables:
      - name: orders
```

それをモデルで使用します:

```sql
select *
  from {{ source("jaffle_shop", "orders") }}
 where foo=1
```

次に、このモデルには、`jaffle_shop/orders` アセット キーを持つアップストリーム ソースがあります。

このアップストリーム アセットを Dagster で管理するには、<PyObject section="libraries" module="dagster_dbt" object="get_asset_key_for_source"/> を介してアセット定義にキーを渡すことで定義できます:

<CodeExample
  startAfter="start_upstream_asset"
  endBefore="end_upstream_asset"
  path="docs_snippets/docs_snippets/integrations/dbt/dbt.py"
/>

これにより、対応する Dagster 定義を更新せずに、dbt プロジェクト内のアセット キーを変更できます。

<PyObject section="libraries" module="dagster_dbt" object="get_asset_key_for_source" /> メソッドは、ソースにテーブルが 1 つしかない場合に使用されます。ただし、次の例のように、ソースに複数のテーブルが含まれている場合は、次のようになります:

```yaml
sources:
  - name: clients_data
    tables:
      - name: names
      - name: history
```

代わりに、<PyObject section="assets" module="dagster" object="multi_asset" decorator/> を <PyObject section="libraries" module="dagster_dbt" object="get_asset_keys_by_output_name_for_source"/> のキーを使用して定義することもできます:

<CodeExample
  startAfter="start_upstream_multi_asset"
  endBefore="end_upstream_multi_asset"
  path="docs_snippets/docs_snippets/integrations/dbt/dbt.py"
/>

### 下流の依存関係 {#downstream-dependencies}

Dagster では、<PyObject section="libraries" module="dagster_dbt" object="get_asset_key_for_model"/> を介して、特定の dbt モデルのダウンストリームにあるアセットを定義できます。以下の例では、`my_downstream_asset` を `my_dbt_model` のダウンストリーム依存関係として定義しています:

<CodeExample
  startAfter="start_downstream_asset"
  endBefore="end_downstream_asset"
  path="docs_snippets/docs_snippets/integrations/dbt/dbt.py"
/>

下流アセットでは、dbt モデルの内容に直接アクセスしたい場合があります。そのためには、アップストリーム データをロードするために、`@asset` で装飾された関数内のコードをカスタマイズできます。

Dagster では、データのロードを I/O マネージャーに委任することもできます。たとえば、アセット キー `my_dbt_model` を持つ dbt モデルを Pandas データフレームとして使用する場合、次のようになります:

<CodeExample
  startAfter="start_downstream_asset_pandas_df_manager"
  endBefore="end_downstream_asset_pandas_df_manager"
  path="docs_snippets/docs_snippets/integrations/dbt/dbt.py"
/>

## パーティションを使用した増分モデルの構築

増分モデルを構築するために、dbt と一緒に Dagster <PyObject section="partitions" module="dagster" object="PartitionsDefinition"/> を定義できます。

パーティション化されたアセットは、<PyObject section="partitions" module="dagster" object="TimeWindow"/> の開始日と終了日にアクセスでき、これらは増分モデルのフィルタリングに使用できる変数として dbt の CLI に渡すことができます。

パーティション定義が <PyObject section="libraries" module="dagster_dbt" object="dbt_assets" decorator/> デコレータに渡されると、すべてのアセットは同じパーティションで動作するように定義されます。これを念頭に置いて、現在の開始パーティションと終了パーティションを取得するために、<PyObject section="execution" module="dagster" object="AssetExecutionContext.partition_time_window" /> プロパティから任意の時間ウィンドウを取得できます。

<CodeExample
  startAfter="start_build_incremental_model"
  endBefore="end_build_incremental_model"
  path="docs_snippets/docs_snippets/integrations/dbt/dbt.py"
/>

変数が定義されたので、SQL で `min_date` と `max_date` を参照し、dbt モデルを増分として構成できるようになりました。ここでは、`min_date` と `max_date` の間にある `order_date` を持つ行を操作する増分実行を定義します。

```sql
-- Configure the model as incremental, use a unique_key and the delete+insert strategy to ensure the pipeline is idempotent.
{{ config(materialized='incremental', unique_key='order_date', incremental_strategy="delete+insert") }}

select * from {{ ref('my_model') }}

-- Use the Dagster partition variables to filter rows on an incremental run
{% if is_incremental() %}
where order_date >= '{{ var('min_date') }}' and order_date <= '{{ var('max_date') }}'
{% endif %}
```
