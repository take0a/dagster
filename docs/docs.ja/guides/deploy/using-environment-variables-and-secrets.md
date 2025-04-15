---
title: 'Dagster コードで環境変数とシークレットを使用する'
sidebar_position: 400
---

環境変数は、ソースコード外で設定されるキーと値のペアであり、環境に応じてアプリケーションの動作を動的に変更できます。

環境変数を使用すると、Dagsterアプリケーションのさまざまな構成オプションを定義し、シークレットを安全に設定できます。
例えば、データベースの認証情報をハードコーディングする（これは開発にとって好ましくない方法で煩雑です）代わりに、環境変数を使用してユーザーの詳細を提供できます。
これにより、コードを変更したり、機密データを安全でない状態で保存したりすることなく、パイプラインをパラメータ化できます。

## 環境変数の宣言

環境変数の宣言方法は、ローカルで開発しているか、すでに Dagster プロジェクトをデプロイしているかによって異なります。

<Tabs>
<TabItem value="ローカル開発">

**ローカル開発**

Dagster 1.1.0 以降、環境変数をローカル環境に読み込むために `.env` ファイルの使用がサポートされています。
`.env` ファイルは、ローカルで使用されるキーと値のペアを含むテキストファイルで、ソース管理にはチェックインされません。
`.env` ファイルを使用すると、機密情報を危険にさらすことなく、ローカルで開発とテストを行うことができます。
例:

```shell
# .env

DATABASE_NAME=staging
DATABASE_SCHEMA=sales
DATABASE_USERNAME=salesteam
DATABASE_PASSWORD=supersecretstagingpassword
```

Dagsterは、`dagster-webserver`または`dagster-daemon`が起動されているフォルダと同じフォルダに`.env`ファイルが存在することを検出すると、そのファイル内の環境変数を自動的に読み込みます。これは、Dagster+からエクスポートされた変数にも適用されます(/dagster-plus/deployment/management/environment-variables/dagster-ui#export)。

`.env`ファイルを使用する際は、以下の点にご注意ください。

- `.env`ファイルは、`dagster-webserver`または`dagster-daemon`が起動されているフォルダと同じフォルダに配置する必要があります。
- `.env`ファイルが変更されるたびに、ワークスペースを再読み込みして、Dagsterウェブサーバー/UIに変更を反映させる必要があります。

</TabItem>
<TabItem value="Dagster+">

**Dagster+**

Dagster+では、環境変数を様々な方法で設定できます。

- UIから直接設定
- エージェント設定から設定（ハイブリッドデプロイメントのみ）

UIを使用する場合は、[ローカルスコープの変数を`.env`ファイルにエクスポート](/dagster-plus/deployment/management/environment-variables/dagster-ui#export)して、ローカル開発で使用することもできます。

詳細については、[Dagster+環境変数ガイド](/dagster-plus/deployment/management/environment-variables/)を参照してください。

</TabItem>
<TabItem value="Dagster オープンソース">

**Dagster オープンソース**

インフラにデプロイされた Dagster プロジェクトの環境変数の設定方法は、**Dagster がデプロイされている場所** によって異なります。
詳細については、ご利用のプラットフォームのデプロイメントガイドを参照してください:

- [Amazon Web Services EC2 / ECS](/guides/deploy/deployment-options/aws)
- [GCP](/guides/deploy/deployment-options/gcp)
- [Docker](/guides/deploy/deployment-options/docker)
- [Kubernetes](/guides/deploy/deployment-options/kubernetes/deploying-to-kubernetes)

</TabItem>
</Tabs>

## 環境変数へのアクセス

このセクションでは、宣言済みの環境変数にアクセスする方法を説明します。
これには2つの方法があります:

- [Pythonコード内](#in-python-code)。これはDagsterに固有のものではありません。
- [Dagster設定から](#from-dagster-configuration)。これは環境変数をDagster設定システムに組み込みます。

### Pythonコード内 {#in-python-code}

Dagster コード内の環境変数にアクセスするには、[`os.getenv`](https://docs.python.org/3/library/os.html#os.getenv) を使用できます。

```python
import os

database_name = os.getenv("DATABASE_NAME")
```

この方法は、[Dagster+ の組み込み環境変数](/dagster-plus/deployment/management/environment-variables/built-in) にアクセスする場合にも機能します:

```python
import os

deployment_name = os.getenv("DAGSTER_CLOUD_DEPLOYMENT_NAME")
```

実際の例については、[Dagster+ ブランチのデプロイメント例](#dagster-branch-deployments) をご覧ください。

`EnvVar` に対して `get_value()` メソッドを呼び出すこともできます:

```python
from dagster import EnvVar

database_name = EnvVar('DATABASE_NAME').get_value()
```

### Dagster設定から {#from-dagster-configuration}

[設定可能な Dagster オブジェクト](/guides/operate/configuration/run-configuration) (オペレーション、アセット、リソース、I/O マネージャーなど) は、環境変数から設定を受け入れることができます。
Dagster は、設定で環境変数を指定するためのネイティブな方法を提供します。
これらの環境変数は、`os.getenv` のように初期化時ではなく、起動時に取得されます。
詳細については、[次のセクション](#using-envvar-vs-osgetenv) を参照してください。

<Tabs>
<TabItem value="Pythonコード内">

**Pythonコード内**

Python コードで Dagster 構成の一部として環境変数にアクセスするには、次の特殊な構文を使用できます:

```python
"PARAMETER_NAME": EnvVar("ENVIRONMENT_VARIABLE_NAME")
```

例:

```python
"access_token": EnvVar("GITHUB_ACCESS_TOKEN")
```

整数を指定する場合:

```python
"database_port": EnvVar.int("DATABASE_PORT")
```

</TabItem>
<TabItem value="YAMLまたは設定辞書内">

**YAMLまたは設定辞書内**

YAML または config ディクショナリ内の Dagster 構成の一部として環境変数にアクセスするには、次の構文を使用します:

```python
"PARAMETER_NAME": {"env": "ENVIRONMENT_VARIABLE_NAME"}
```

例:

```python
"access_token": {"env": "GITHUB_ACCESS_TOKEN"}
```

例については、[シークレットの処理セクション](#handling-secrets)と[環境ごとの構成例](#per-environment-configuration-example)を参照してください。

</TabItem>
</Tabs>

### EnvVar と os.getenv の使用 {#using-envvar-vs-osgeten}

Dagster で環境変数にアクセスする 2 つの異なる方法について説明しました。
では、どちらを使用すべきでしょうか？
方法を選択する際には、以下の点に留意してください。

- **`os.getenv` を使用する場合**、変数の値は Dagster が [コードの場所](/guides/deploy/code-locations/) をロードしたときに取得され、UI に表示されます。
- **`EnvVar` を使用する場合**、変数の値は実行時に取得され、UI には表示**されません。**

`EnvVar` アプローチを使用すると、いくつかの独自のメリットがあります:

- **可観測性の向上。** UI に、環境変数から取得した設定値に関する情報が表示されます。
- **シークレット値は UI に表示されません。** シークレット値は、Launchpad、リソースページ、その他の設定が表示される場所では表示されません。
- **テストの簡素化。** 環境変数ではなく設定に直接文字列値を指定できるため、テストが容易になります。

## 秘密の取り扱い

環境変数を使用してシークレットを提供することで、機密情報がコードや UI の Launchpad に表示されることを防ぎます。
Dagster では、シークレットの取り扱いに関するベストプラクティスとして、[configuration](/guides/operate/configuration/run-configuration) と [resources](/guides/build/external-resources/) が使用されています。

リソースは通常、データベースなどの外部サービスまたはシステムに接続するために使用されます。
リソースはアプリの他の部分とは個別に設定できるため、一度定義しておけば必要に応じて再利用できます。

[Dagster Crash Course](https://dagster.io/blog/dagster-crash-course-oct-2022) の例を見てみましょう。この例では、GitHub リソースを作成してアセットに提供しています。
まずはリソースを見てみましょう:

```python
## resources.py

from dagster import StringSource, resource
from github import Github

class GithubClientResource(ConfigurableResource):
  access_token: str

  def get_client(self) -> Github:
    return Github(self.access_token)
```

ここで何が起こっているのか確認してみましょう。

- このコードは、`GithubClientResource` という GitHub リソースを作成します。
- <PyObject section="resources" module="dagster" object="ConfigurableResource" /> をサブクラス化し、`access_token` フィールドを指定することで、Dagster に `access_token` パラメータを使ってリソースを設定できるように指示しています。
- `access_token` は文字列値なので、この設定パラメータは次のいずれかになります。
  - 環境変数、または
  - 設定で直接指定

設定にシークレットを保存するのは好ましくないため、環境変数を使用することにします。
このコードでは、アセットにリソースを提供することでリソースを設定しています。

<CodeExample
  path="docs_snippets/docs_snippets/guides/dagster/using_environment_variables_and_secrets/repository.py"
  startAfter="start"
  endBefore="end"
/>

ここで何が起こっているか確認してみましょう:

- リソースを構築するときに、設定情報を渡します。
この例では、`GITHUB_ACCESS_TOKEN` 環境変数を `EnvVar` でラップすることで、Dagster に `access_token` を読み込むように指示しています。
- このリソースを <PyObject section="definitions" module="dagster" object="Definitions" /> オブジェクトに追加して、アセットで使用できるようにします。

## パイプラインの動作のパラメータ化

環境変数を使用することで、実行時にコードがどのように実行されるかを定義します。

- [環境ごとの設定例](#per-environment-configuration-example)

### 環境ごとの構成例 {#per-environment-configuration-example}

この例では、[configuration](/guides/operate/configuration/run-configuration)（具体的には構成済みAPI）と[resources](/guides/build/external-resources/)を使用して、`local`環境と`production`環境で異なるI/Oマネージャー構成を使用する方法を説明します。

この例は、[データパイプラインを開発環境から本番環境に移行するガイド](/guides/deploy/dev-to-prod)から抜粋したものです。

<CodeExample
  path="docs_snippets/docs_snippets/guides/dagster/using_environment_variables_and_secrets/repository_v2.py"
  startAfter="start_new"
  endBefore="end_new"
/>

ここで何が起こっているか確認してみましょう。

- リソース定義の辞書「`resources`」を作成し、`local` 環境と `production` 環境にちなんで名前を付けました。
この例では、[Pandas Snowflake I/O マネージャー](/api/python-api/libraries/dagster-snowflake-pandas) を使用しています。
- `local` と `production` の両方で、環境固有の実行構成を使用して I/O マネージャーを構築しました。
`local` と `production` の構成の違い、特に環境変数が使用されている箇所に注意してください。
- `resources` 辞書に続いて、現在の実行環境を決定する `deployment_name` 変数を定義します。
この変数のデフォルトは `local` であるため、`production` 構成を使用するには `DAGSTER_DEPLOYMENT=PRODUCTION` を設定する必要があります。

### Dagster+ ブランチデプロイメント

:::note

このセクションは Dagster+ にのみ適用されます。

:::

この例では、リソースや設定を使用せずに、実行時に現在のデプロイメントタイプ（[ブランチデプロイメント](/dagster-plus/features/ci-cd/branch-deployments/)または[フルデプロイメント](/dagster-plus/deployment/management/deployments/)）を判別する方法を示します。

`DAGSTER_CLOUD_IS_BRANCH_DEPLOYMENT` 環境変数を使用して現在のデプロイメントを判別する関数を見てみましょう:

```python

def get_current_env():
  is_branch_depl = os.getenv("DAGSTER_CLOUD_IS_BRANCH_DEPLOYMENT") == "1"
  assert is_branch_depl != None  # env var must be set
  return "branch" if is_branch_depl else "prod"

```

この関数は `DAGSTER_CLOUD_IS_BRANCH_DEPLOYMENT` の値をチェックし、`1` と等しい場合は `branch` の値を持つ変数を返します。
これは、現在のデプロイメントがブランチデプロイメントであることを示します。
それ以外の場合、デプロイメントは完全デプロイメントであり、is_branch_depl には prod という値が返されます。

この情報を使用して、ブランチデプロイメントと完全デプロイメントで異なる実行方法を実行するコードを作成できます。

## トラブルシューティング

| Error                                                                                                                                                              | Description                                                                                                                                                                                                               | Solution                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   |
| ------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **You have attempted to fetch the environment variable "[variable]" which is not set. In order for this execution to succeed it must be set in this environment.** | UI で実行が開始されたときに表示されるこのエラーは、<PyObject section="config" module="dagster" object="StringSource" /> を使用して設定された環境変数が実行環境で見つからなかったことを意味します。 | 環境変数の名前が正しく、環境内でアクセス可能であることを確認してください。<ul><li>**ローカルで開発していて `.env` ファイルを使用している場合**、UI でワークスペースを再読み込みしてみてください。UI が変更を認識するには、このファイルが変更されるたびにワークスペースを再読み込みする必要があります。</li><li>**Dagster+ を使用している場合**:</li><ul><li>組み込みのシークレット マネージャーを使用している場合は、環境変数が [環境とコードの場所にスコープされている](/dagster-plus/deployment/management/environment-variables/dagster-ui#scope) ことを確認してください。</li><li>環境変数が正しく構成され、[エージェントの構成](/dagster-plus/deployment/management/environment-variables/agent-config) に追加されていることを確認してください。</li></ul></ul> |
| **No environment variables in `.env` file.**     | Dagster は `dagster-webserver` を起動中にローカルの `.env` ファイルを見つけてロードしようとしましたが、ファイル内に環境変数が見つかりませんでした。  | これが予期しないものである場合は、`.env` が正しくフォーマットされており、`dagster-webserver` を実行しているフォルダーと同じフォルダーに配置されていることを確認してください。 |
