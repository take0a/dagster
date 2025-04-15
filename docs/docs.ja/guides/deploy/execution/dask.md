---
title: 'Dask 上で Dagster を実行する'
sidebar_position: 800
---

[dagster-dask](https://github.com/dagster-io/dagster/tree/master/python_modules/libraries/dagster-dask) モジュールは、ローカル Dask クラスターまたは分散クラスターのいずれかをターゲットにできる **`dask_executor`** を提供します。
計算は実行ステップレベルでクラスター全体に分散されます。つまり、Dask はジョブ内のステップの実行を調整するために使用するものであり、ステップ内の計算を並列化するために使用するものではありません。

このエグゼキューターは、コンパイルされた実行計画を受け取り、各実行ステップを、適切なタスク依存関係が構成された [Dask Future](https://docs.dask.org/en/latest/futures.html) に変換します。これにより、タスクが適切に順序付けられます。
ジョブが実行されると、これらの Future が生成され、親 Dagster プロセスによって待機されます。

データは [IO マネージャー](/guides/build/io-managers/) を介してステップ実行間で渡されます。
そのため、永続的な共有ストレージ（すべての Dask ノードで共有されるネットワークファイルシステム、S3、GCS など）を使用する必要があります。

このエグゼキューターを使用する場合、単一のオペレーションの計算関数は、単一のマシン上の単一のプロセスで実行されることに注意してください。
単一のオペレーションのロジック内でワークロードの実行を分散することが目的の場合は、このドキュメントで説明されているエンジン レイヤーよりも、オペレーションのコンピューティング関数の本体内から直接 Dask または PySpark を呼び出す方が適している場合があります。

## 要件

[dask.distributed](https://distributed.readthedocs.io/en/latest/install.html) をインストールしてください。

## ローカル実行

ローカルのDASK上でDagsterジョブをセットアップして実行するのは比較的簡単です。
これはテストに役立ちます。

まず、`pip install dagster-dask` を実行します。

次に、DASKエグゼキュータでジョブを作成します:

<CodeExample
  path="docs_snippets/docs_snippets/deploying/dask_hello_world.py"
  startAfter="start_local_job_marker"
  endBefore="end_local_job_marker"
/>

これで、次のような構成ブロックを使用してこのジョブを実行できます:

<CodeExample path="docs_snippets/docs_snippets/deploying/dask_hello_world.yaml" />

このジョブを実行すると、ローカルの Dask 実行が開始され、ジョブが実行されて終了します。

## 分散実行

分散実行に Dask クラスターを使用する場合は、まず [Dask クラスターをセットアップ](https://distributed.readthedocs.io/en/latest/quickstart.html#setup-dask-distributed-the-hard-way) する必要があります。
Dagster 親プロセスを実行しているマシンは、Dask スケジューラが実行されているホスト/ポートに接続できる必要があります。

また、永続的な共有ストレージを使用する IO マネージャーも必要です。これは、ジョブが依存するリソースとともにジョブにアタッチする必要があります。
ここでは、<PyObject section="libraries" module="dagster_aws" object="s3.s3_pickle_io_manager"/> を使用します。

<CodeExample
  path="docs_snippets/docs_snippets/deploying/dask_hello_world_distributed.py"
  startAfter="start_distributed_job_marker"
  endBefore="end_distributed_job_marker"
/>

Dask クラスターでタスク実行を分散するには、Dask スケジューラのアドレス/ポートを含む構成ブロックを提供する必要があります:

<CodeExample path="docs_snippets/docs_snippets/deploying/dask_remote.yaml" />

Dask はクラスターワーカー上でジョブコードを呼び出すため、すべての Dask ワーカーで最新バージョンの Python コードが利用できるようにする必要があります。
理想的には、これを Python モジュールとしてパッケージ化し、`workspace.yaml` でこのモジュールをターゲットにします。

## Dask によるコンピューティングリソースの管理

Dask は、コンピューティングリソース管理のための [基本的なサポート](https://distributed.dask.org/en/latest/resources.html) を備えています。
Dask では、特定のワーカーノードに GPU を 3 基搭載するなどと指定すると、GPU 要件が指定されたタスクは、利用可能なリソースの制約を考慮してスケジュールされます。

Dask では、リソース仕様を指定してワーカーを起動することで、これを設定できます:

```shell
dask-worker scheduler:8786 --resources "GPU=2"
```

そして、Dask クラスターにタスクを送信するときに、Python API でリソース要件を指定します:

```python
client.submit(task, resources={'GPU': 1})
```

Dagster は、Dask クラスターで実行されるオペレーションのオペレーションレベルでの Dask リソース指定をシンプルにサポートしています。
オペレーション定義に、次のように _tags_ を追加するだけです:

```python
@op(
    ...
    tags={'dagster-dask/resource_requirements': {"GPU": 1}},
)
def my_op(...):
    pass
```

`dagster-dask/resource_requirements` に渡された辞書は、Dask クラスターで実行するために、Dask クライアントの **`~dask:distributed.Client.submit`** メソッドに `resources` 引数として渡されます。
Dask 以外の実行では、このキーは無視されることに注意してください。

## 注意事項

Dagster ログはまだ Dask ワーカーから取得されていません。これは後続の作業で対処される予定です。

このライブラリはまだ初期段階ですが、改善に取り組んでおり、皆様からの貢献を歓迎します。
