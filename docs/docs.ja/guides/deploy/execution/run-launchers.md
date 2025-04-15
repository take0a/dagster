---
title: '実行ランチャー'
sidebar_position: 200
---

:::note

この記事は、Dagsterオープンソース（OSS）デプロイメントに適用されます。
Dagster+の詳細については、[Dagster+ドキュメント](/dagster-plus/)をご覧ください。

:::

Dagster UI、スケジューラ、または `dagster job launch` CLI コマンドから開始された実行は、Dagster で起動されます。
これは、`execute_job` Python API または `execute` CLI コマンドを使用してジョブを実行する操作とは異なります。
起動操作は、実行を実行するための計算リソース（プロセス、コンテナ、Kubernetes ポッドなど）を割り当て、実行を開始します。

起動プロセスの中核となる抽象化は、_実行ランチャー_です。これは、[Dagsterインスタンス](/guides/deploy/dagster-instance-configuration) の一部として構成されます。
実行ランチャーは、Dagster の実行を実際に行うために使用される計算リソースへのインターフェースです。
作成された実行のIDと、実行が開始されるパイプラインの表現を受け取ります。

## 関連API

| Name                                                                                  | Description                   |
| ------------------------------------------------------------------------------------- | ----------------------------- |
| <PyObject section="internals" module="dagster._core.launcher" object="RunLauncher" /> | 実行ランチャーの基本クラス。 |

## 組み込み実行ランチャー

最もシンプルな実行ランチャーは、組み込みの実行ランチャー <PyObject section="internals" module="dagster._core.launcher" object="DefaultRunLauncher" /> です。
この実行ランチャーは、ジョブのコードと同じノード上で、実行ごとに新しいプロセスを生成します。

その他の実行ランチャーには、以下のものがあります:

| Name                                                                                       | Description                                                                                                                    | Documentation                                                                                           |
| ------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------- |
| <PyObject section="libraries" module="dagster_k8s" object="K8sRunLauncher" />              |実行ごとに Kubernetes ジョブを割り当てる実行ランチャー。     | [Deploying Dagster to Kubernetes](/guides/deploy/deployment-options/kubernetes/deploying-to-kubernetes) |
| <PyObject section="libraries" module="dagster_aws" object="ecs.EcsRunLauncher" />          |実行ごとに Amazon ECS タスクを起動する実行ランチャー。      | [Deploying Dagster to Amazon Web Services](/guides/deploy/deployment-options/aws)                       |
| <PyObject section="libraries" module="dagster_docker" object="DockerRunLauncher" />        |Docker コンテナ内で実行を起動する実行ランチャー。     | [Deploying Dagster using Docker Compose](/guides/deploy/deployment-options/)                            |
| <PyObject section="libraries" module="dagster_celery_k8s" object="CeleryK8sRunLauncher" /> |`celery_k8s_job_executor` をサポートするための追加構成を備えた単一の Kubernetes ジョブとして実行を開始する実行ランチャー。 | [Using Celery with Kubernetes](/guides/deploy/deployment-options/kubernetes/kubernetes-and-celery)      |

## カスタム実行ランチャー

カスタム実行ランチャーが必要となる例をいくつか挙げます。

- 実行用のノードを割り当てるためのカスタムインフラストラクチャまたはカスタムAPIがある場合。
- 異なるクラスター、プラットフォームなどで実行を開始するためのカスタムロジックがある場合。

実行ランチャーによって作成されるプロセスまたは計算リソースを[実行ワーカー](/guides/deploy/oss-deployment-architecture#job-execution-flow)と呼びます。
実行ランチャーは、実行ワーカーの動作を決定するだけです。
実行ワーカー内で実行が開始されると、実行ワーカープロセス内のメモリ内抽象化であるエグゼキューターが計算リソースの管理を引き継ぎます。
