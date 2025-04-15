---
title: 'ワークスペースファイル (workspace.yaml) リファレンス'
sidebar_position: 200
---

:::info

このリファレンスはDagster OSSにのみ適用されます。
Dagster+については、[Dagster+のコードロケーションに関するドキュメント](/dagster-plus/deployment/code-locations)をご覧ください。

:::

ワークスペースファイルは、Dagster 内のコードの場所を設定するために使用されます。
ワークスペースファイルは、Dagster にコードの場所とロード方法を指示します。
デフォルトでは、workspace.yaml という名前の YAML ドキュメントです。
例:

```yaml
# workspace.yaml

load_from:
  - python_file: my_file.py
```

ワークスペース ファイル内の各エントリは、コードの場所とみなされます。
コードの場所には、1 つの <PyObject section="definitions" module="dagster" object="Definitions" /> オブジェクトが含まれている必要があります。

各コードロケーションは、DagsterツールがRPCプロトコルを使用して通信する独自​​のプロセスにロードされます。
このプロセス分離により、異なる環境にある複数のコードロケーションを個別にロードできるようになり、ユーザーコードのエラーがDagsterシステムコードに影響を与えないようにすることができます。

:::info `@repository` から `Definitions` への移行

`@repository` から `Definitions` への段階的な移行に対応するため、単一のワークスペース ファイル内のコードの場所は、複数の定義アプローチを混在させることができます。
たとえば、`code-location-1` はファイルまたはモジュールから単一の `Definitions` オブジェクトをロードし、`code-location-2` は複数のリポジトリをロードできます。

:::

## workspace.yaml の場所

Dagster コマンドラインツール（`dagster dev`、`dagster-webserver`、`dagster-daemon run` など）は、起動時に現在のディレクトリにあるワークスペースファイルを検索します。
これにより、コマンドライン引数を必要とせずに、そのディレクトリから起動できます。

別のフォルダからworkspace.yaml ファイルを読み込むには、-w 引数を使用します。

```bash
dagster dev -w path/to/workspace.yaml
```

## ファイル構造

`workspace.yaml` ファイルは以下の構造を使用します:

```yaml
load_from:
  - <loading_method>: <configuration_options>
```

ここで、`<loading_method>` は次のいずれかになります:

- `python_file`
- `python_module`
- `grpc_server`

## コードの場所の読み込み

ほとんどの場合、Python モジュールから読み込むことをお勧めします。

:::note

各コードの場所は、独自のプロセスで読み込まれます。

:::

### Python モジュール

ローカルまたはインストール済みの Python モジュールからコードの場所をロードするには、workspace.yaml の `python_module` キーを使用します。

**オプション:**

- `module_name`: ロードする Python モジュールの名前。
- その他のオプションは `python_file` と同じです。

**例:**

```yaml
load_from:
  - python_module:
      module_name: hello_world_module.definitions
```

### Python ファイル

Python ファイルからコードの場所を読み込むには、workspace.yaml の `python_file` キーを使用します。
`python_file` の値は、コードの場所の定義を含むファイルへの `workspace.yaml` からの相対パスを指定する必要があります。

**オプション:**

- `relative_path`: Python ファイルへのパス（`workspace.yaml` の場所からの相対パス）。
- `attribute` (オプション): 特定のリポジトリまたは `RepositoryDe​​finition` を返す関数の名前。
- `working_directory` (オプション): 相対インポート用のカスタム作業ディレクトリ。
- `executable_path` (オプション): 特定の Python 実行ファイルへのパス。
- `location_name` (オプション): コードの場所のカスタム名。

**例:**

```yaml
load_from:
  - python_file:
      relative_path: hello_world_repository.py
      attribute: hello_world_repository
      working_directory: my_working_directory/
      executable_path: venvs/path/to/python
      location_name: my_location
```

:::info @repository の使用

`@repository` を使用してコードの場所を定義する場合、`attribute` キーを使用してモジュール内の単一のリポジトリを識別できます。
このキーの値は、リポジトリの名前、または <PyObject section="repositories" module="dagster" object="RepositoryDe​​finition" /> を返す関数の名前である必要があります。
例:

```yaml
# workspace.yaml

load_from:
  - python_file:
      relative_path: hello_world_repository.py
      attribute: hello_world_repository
```

:::

### gRPC サーバー

gRPC サーバーをコードの場所として設定します。

**オプション:**

- `host`: gRPC サーバーのホストアドレス。
- `port`: gRPC サーバーのポート番号。
- `location_name`: コードの場所のカスタム名。

**例:**

```yaml
load_from:
  - grpc_server:
      host: localhost
      port: 4266
      location_name: 'my_grpc_server'
```

## 複数のコードロケーション

1 つの `workspace.yaml` ファイルで複数のコードロケーションを定義できます:

```yaml
load_from:
  - python_file:
      relative_path: path/to/dataengineering_spark_team.py
      location_name: dataengineering_spark_team_py_38_virtual_env
      executable_path: venvs/path/to/dataengineering_spark_team/bin/python
  - python_file:
      relative_path: path/to/team_code_location.py
      location_name: ml_team_py_36_virtual_env
      executable_path: venvs/path/to/ml_tensorflow/bin/python
```

## ワークスペースファイルの読み込み

デフォルトでは、Dagster コマンドラインツール（`dagster dev`、`dagster-webserver`、`dagster-daemon run` など）は、起動時に現在のディレクトリにあるワークスペースファイル（デフォルトでは `workspace.yaml`）を検索します。
これにより、コマンドライン引数を必要とせずに、そのディレクトリから起動できます。

```shell
dagster dev
```

別のフォルダーから `workspace.yaml` ファイルを読み込むには、 `-w` 引数を使用します:

```shell
dagster dev -w path/to/workspace.yaml
```

`dagster dev` を実行すると、Dagster はワークスペースファイルで定義されたすべてのコードの場所をロードします。
詳細と例については、[CLI リファレンス](/api/python-api/cli#dagster-dev) を参照してください。

構文エラーやその他の回復不可能なエラーなどによりコードの場所をロードできない場合は、Dagster UI に警告メッセージが表示されます。
Dagster がロードできなかった場所については、エラーの詳細とスタックトレースを含むステータスページが表示されます。

:::info

コードの場所の名前が変更された場合、またはワークスペース ファイル内の構成が変更された場合は、そのコードの場所にある実行中のスケジュールまたはセンサーを停止して再起動する必要があります。
UI でこれを行うには、[**Deployment overview** ページ](/guides/operate/webserver#deployment) に移動し、コードの場所をクリックして、**Schedules** タブと **Sensors** タブを使用します。

:::

## 独自の gRPC サーバーを実行する

デフォルトでは、Dagster ツールはローカルマシン上にコードの場所ごとにプロセスを自動的に作成します。
ただし、コードの場所に関する情報を提供する独自の gRPC サーバーを実行することもできます。
これは、ユーザーコードを Dagster ウェブサーバーとは別にデプロイする、より複雑なシステムアーキテクチャで役立ちます。

- [サーバーの初期化](#initializing-the-server)
- [Dockerイメージの指定](#specifying-a-docker-image)

### サーバーの初期化 {#initializing-the-server}

Dagster gRPC サーバーを初期化するには、`dagster api grpc` コマンドを実行し、以下のオプションを指定します。

- ターゲットファイルまたはモジュール。ワークスペースファイルと同様に、ターゲットは Python ファイルまたはモジュールのいずれかです。
- ホストアドレス
- ポートまたはソケット

以下のタブは、gRPC サーバーを初期化する一般的な方法を示しています。

<Tabs>
<TabItem value="Pythonファイルの使用">

Python ファイルを使用してポート上で実行します:

```shell
dagster api grpc --python-file /path/to/file.py --host 0.0.0.0 --port 4266
```

Python ファイルを使用してソケット上で実行します:

```shell
dagster api grpc --python-file /path/to/file.py --host 0.0.0.0 --socket /path/to/socket
```

</TabItem>
<TabItem value="Pythonモジュールの使用">

Pythonモジュールの使用:

```shell
dagster api grpc --module-name my_module_name.definitions --host 0.0.0.0 --port 4266
```

</TabItem>
<TabItem value="Using a specific repository">

:::note

これは、<PyObject section="repositories" module="dagster" object="repository" decorator /> で定義されたコードの場所にのみ適用されます。

:::

ターゲット内で属性を指定して、特定のリポジトリをロードします。
実行すると、サーバーは指定されたリポジトリを自動的に検出してロードします。

```shell
dagster api grpc --python-file /path/to/file.py --attribute my_repository --host 0.0.0.0 --port 4266
```

</TabItem>
<TabItem value="Local imports">

ローカルインポートのベースフォルダとして使用する作業ディレクトリを指定します:

```shell
dagster api grpc --python-file /path/to/file.py --working-directory /var/my_working_dir --host 0.0.0.0 --port 4266
```

</TabItem>
</Tabs>

新しい gRPC サーバーを実行する際に設定できるオプションの完全なリストについては、[API ドキュメント](/api/python-api/cli#dagster-api-grpc) を参照してください。

次に、ワークスペースファイルで、新しい gRPC サーバーのコードがロードされる場所を設定します:

```yaml file=/concepts/repositories_workspaces/workspace_grpc.yaml
# workspace.yaml

load_from:
  - grpc_server:
      host: localhost
      port: 4266
      location_name: 'my_grpc_server'
```

### Dockerイメージの指定 {#specifying-a-docker-image}

コンテナ内で独自の gRPC サーバーを実行する場合、ウェブサーバーに対して、コードの場所から起動されるすべての実行を、同じイメージを持つコンテナ内で起動するように指示できます。

これを行うには、サーバーを起動する前に、`DAGSTER_CURRENT_IMAGE` 環境変数にイメージ名を設定します。
サーバーでこの環境変数を設定すると、UI の **Status** ページに、コードの場所と並んでイメージが表示されます。

このイメージは、Docker イメージ (<PyObject section="libraries" module="dagster_docker" object="DockerRunLauncher" />、<PyObject section="libraries" module="dagster_k8s" object="K8sRunLauncher" />、<PyObject section="libraries" module="dagster_docker" object="docker_executor" />、<PyObject section="libraries" module="dagster_k8s" object="k8s_job_executor" /> など) の使用を想定している [ランチャーの実行](/guides/deploy/execution/run-launchers) および [エグゼキューター](/guides/operate/run-executors) によってのみ使用されます。

組み込みの [Helm チャート](/guides/deploy/deployment-options/kubernetes/deploying-to-kubernetes) を使用している場合、この環境変数は各 gRPC サーバーで自動的に設定されます。

## 例

<Tabs>
<TabItem value="相対インポートの読み込み">

### 相対インポートの読み込み

デフォルトでは、コードは `dagster-webserver` の作業ディレクトリをベースパスとして読み込まれ、コード内のローカルインポートを解決します。
`working_directory` キーを使用すると、相対インポート用のカスタム作業ディレクトリを指定できます。
例:

<CodeExample path="docs_snippets/docs_snippets/concepts/repositories_workspaces/workspace_working_directory.yaml" />

</TabItem>
<TabItem value="複数のPython環境を読み込む">

### 複数のPython環境を読み込む

デフォルトでは、ウェブサーバーやその他の Dagster ツールは、Dagster のロードに使用されたものと同じ Python 環境を使用してコードの場所をロードすることを想定しています。
ただし、コードの場所ごとに独立した環境を使用することが有用な場合がよくあります。たとえば、Spark を実行するデータエンジニアリングチームと、Tensorflow を実行する ML チームでは、依存関係が大きく異なる場合があります。

このようなユースケースに対応するため、Dagster は、場所の YAML に `executable_path` キーを追加することで、コードの場所ごとに Python 環境をカスタマイズできます。
これらの環境には、インストール済みの依存関係の異なるセットが含まれる場合もあれば、Python のバージョンが全く異なる場合もあります。
例:

<CodeExample path="docs_snippets/docs_snippets/concepts/repositories_workspaces/python_environment_example.yaml" />

上記の例は、`location_name` キーについても示しています。
ワークスペースファイル内の各コードロケーションには、UI に表示される一意の名前が付けられます。また、複数のコードロケーションで同じ名前の定義を区別するためにも使用されます。
カスタム名が指定されていない場合、Dagster はワークスペースエントリに基づいて各ロケーションにデフォルトの名前を提供します。

</TabItem>
</Tabs>

複数のコードの場所を持つ Dagster プロジェクトの動作例は、[cloud-examples/multi-location-project リポジトリ](https://github.com/dagster-io/cloud-examples/tree/main/multi-location-project) で確認できます。
