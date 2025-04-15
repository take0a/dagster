---
title: 'Celery 上で Dagster を実行する'
sidebar_position: 700
---

[Celery](https://docs.celeryq.dev/) は、オープンソースの Python 分散タスクキューシステムであり、さまざまなキュー（ブローカー）と結果永続化戦略（バックエンド）をサポートしています。

`dagster-celery` エグゼキューターは、Celery を使用して、本番環境でジョブを実行する際に共通する 3 つの要件を満たします。

- 複数のコンピューティングノードに水平方向にスケールする並列実行能力。
- 実行を分離し、オペレーションレベルで外部リソースの使用を制御するための個別のキュー。
- オペレーションレベルでの優先度ベースの実行。

dagster-celery エグゼキューターは、ジョブとそれに関連する設定を具体的な実行プランにコンパイルし、各実行ステップを個別の Celery タスクとしてブローカーに送信します。
dagster-celery ワーカーは、各タスクに割り当てられた優先度に従って、サブスクライブされているキューからタスクを取得し、タスクに対応するステップを実行します。

## 前提条件

このガイドの手順を完了するには、「dagster」と「dagster-celery」をインストールする必要があります:

```shell
pip install dagster dagster-celery
```

- Celery Executor を実行するには、**実行中のブローカー** も必要です。
ブローカーの選択に関する詳細は、[Celery のドキュメント](https://docs.celeryq.dev/en/stable/getting-started/first-steps-with-celery.html#choosing-a-broker) を参照してください。

## パート1：ジョブの作成と実行

まずは、Celery Executor を使用した並列トイジョブを作成します。

Dagster プロジェクトで、「celery_job.py」という新しいファイルを作成し、以下のコードを貼り付けます:

<CodeExample path="docs_snippets/docs_snippets/deploying/celery_job.py" />

Celery Executor を実行します。今回のケースでは、ブローカーとして RabbitMQ を実行しています。Docker の場合は、以下のようになります:

```shell
docker run -p 5672:5672 rabbitmq:3.8.2
```

タスクを実行するには、Celeryワーカーを実行する必要があります。
`celery_job.py`ファイルを保存したのと同じディレクトリから、次のコマンドを実行します:

```shell
dagster-celery worker start -A dagster_celery.app
```

次に、次のコマンドを実行して、Celery でこのジョブを実行します:

```shell
dagster dev -f celery_job.py
```

これで、[Dagster UI](/guides/operate/webserver)から並列ジョブを実行できます。

## パート2：ワーカーの同期の確保

パート1では、いくつかのショートカットを使用しました。

- **Dagsterウェブサーバーと同じノードで単一のCeleryワーカーを実行しました。**
これにより、ローカルの一時的な実行ストレージとイベントログストレージを両者で共有し、ファイルシステムI/Oマネージャーを使用してワーカーのタスク実行間で値を交換できるようになりました。
- **Celeryワーカーを`celery_job.py`と同じディレクトリで実行しました。**
これにより、Dagsterコードはウェブサーバーとワーカーの両方で利用可能になり、具体的には、両方が同じファイル（`-f celery_job.py`）でジョブ定義を見つけることができるようになりました。

本番環境では、さらに設定が必要です。

### ステップ 1: 永続的な実行ログとイベントログのストレージを構成する

まず、[Dagster インスタンス](/guides/deploy/dagster-instance-configuration) で、適切な永続的な実行ログとイベントログのストレージ（例: `PostgresRunStorage` と `PostgresEventLogStorage`）を構成します（[`dagster.yaml`](/guides/deploy/dagster-yaml) を使用）。
これにより、ウェブサーバーとワーカーが実行とイベントに関する情報を相互に通信できるようになります。
設定方法については、[Dagster インスタンスのドキュメントの Dagster ストレージのセクション](/guides/deploy/dagster-instance-configuration#dagster-storage) を参照してください。

:::note

ウェブサーバーの環境とワーカーの環境で同じインスタンス設定が存在している必要があります。詳細については、[Dagsterインスタンス](/guides/deploy/dagster-instance-configuration)のドキュメントを参照してください。

:::

### ステップ 2: 永続的な I/O マネージャーを構成する

ジョブ実行に Celery Executor を使用する場合、Celery ワーカーが実行されているすべてのノードからアクセス可能なストレージを使用する必要があります。
これは、異なるワーカープロセス（場合によっては異なるノード）にあるオペレーション間でデータが交換されるためです。
このようなアクセス可能なストレージの一般的な選択肢としては、Amazon S3 または Google Cloud Storage (GCS) バケット、あるいは NFS マウントなどがあります。

これを行うには、ジョブのリソースに適切な I/O マネージャーを含めます。
例えば、以下のいずれかの I/O マネージャーが適しています。

- <PyObject section="libraries" module="dagster_aws" object="s3.s3_pickle_io_manager" />
- <PyObject section="libraries" module="dagster_azure" object="adls2.adls2_pickle_io_manager" />
- <PyObject section="libraries" module="dagster_gcp" object="gcs_pickle_io_manager" />

### ステップ 3: エグゼキューターとワーカーの設定を指定する

実行時にカスタム設定を使用する場合（異なる Celery ブローカー URL やバックエンドを使用する場合など）、ワーカーがその設定で起動するようにする必要があります。

手順:

1. エンジン設定が、ワーカーからアクセス可能な YAML ファイルにあることを確認します。
2. 以下のように、`-y` パラメータを指定してワーカーを起動します。

   ```shell
   dagster-celery worker start -y /path/to/celery_config.yaml
   ```

### ステップ 4: Dagster コードへのアクセスを確認する

最後に、ワーカーに実行させる Dagster コードが以下の条件を満たしていることを確認する必要があります。

1. ワーカーの環境に存在すること。
2. コードが、ウェブサーバーを実行しているノード上のコードと同期していること。

通常、これを最も簡単に行う方法は、コードを Python モジュールにパッケージ化し、プロジェクトの [`workspace.yaml`](/guides/deploy/code-locations/workspace-yaml) を設定して、ウェブサーバーがそのモジュールから読み込むようにすることです。

パート 1 では、`-f` パラメータを指定してウェブサーバーを起動することで、この設定を実現しました。

```shell
dagster dev -f celery_job.py
```

これにより、Web サーバーにジョブを含むファイル (`celery_job.py`) が通知され、ファイル システム内の同じポイントから Celery ワーカーが開始され、ジョブが同じ場所で利用できるようになります。

## 追加情報

###dagster-celery CLI の使用

このチュートリアルでは、Celery を直接起動するのではなく、`dagster-celery` CLI を使用してワーカーを起動しました。
この CLI は、Celery の完全な設定の複雑さを軽減する便利なラッパーとして設計されています。
**注**: Celery ワーカーを直接起動することも可能です。ユースケースで必要な場合はお知らせください。

これらのコマンドはすべて、ブローカーが実行中であることが必須です。

```shell
## Start new workers
dagster-celery worker start

## View running workers
dagster-celery worker list

## Terminate workers
dagster-celery worker terminate
```

:::note

Celery をカスタム構成で実行する場合は、ワーカーが正しい構成で起動するように、これらのコマンドに構成ファイルのパスを含めてください。
詳細については、ウォークスルーの [ステップ 3](#step-3-supply-executor-and-worker-config) を参照してください。

:::

`dagster-celery` は、必要に応じて Celery の全設定を利用できるように設計されていますが、設定の組み合わせによっては互換性がない場合があることに注意してください。
ただし、Celery の調整に慣れている場合は、設定の一部を変更すると、ユースケースに合わせてより適切に機能する可能性があります。

### モニタリングとデバッグ

キューとワーカーのモニタリングとデバッグには、いくつかのツールが利用可能です。まずDagster UIは、実行時のイベントログと`stdout`/`stderr`を表示します。また、ブローカーとワーカープロセスによって生成されたログも表示できます。

ブローカー/キューレベルの問題をデバッグするには、実行しているブローカーが提供するモニタリングツールを使用してください。RabbitMQには[モニタリングAPI](https://www.rabbitmq.com/monitoring.html)が含まれており、本番環境でのPrometheusとGrafanaの統合を第一級のサポートで提供しています。

Celeryのワーカーとキューをモニタリングするには、Celeryの[Flower](https://flower.readthedocs.io/en/latest/)ツールを使用できます。これは、ワーカーがキューとどのようにやり取りするかを理解するのに役立ちます。

### ブローカーとバックエンド

`dagster-celery` は、RabbitMQ ブローカーとデフォルトの RPC バックエンドを使用してテストされています。
