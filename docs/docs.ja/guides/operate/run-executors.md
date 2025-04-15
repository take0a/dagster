---
title: 'エクゼキューターの実行'
description: Executors are responsible for executing steps within a job run.
sidebar_position: 40
---

Executor は、ジョブ実行内のステップを実行する役割を担います。
実行が開始され、実行のプロセス（[実行ワーカー](/guides/deploy/oss-deployment-architecture#job-execution-flow)）が割り当てられて開始されると、Executor が実行の責任を引き継ぎます。

Executor は、単一プロセスを連続して実行するものから、高度なコントロールプレーンを使用してステップごとに計算リソースを管理するものまで、多岐にわたります。

## 関連 API

| Name                                                                          | Description                                                                                                                       |
| ----------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------- |
| <PyObject section="internals" module="dagster" object="executor" decorator /> | Executor を定義するために使用されるデコレータ。<PyObject section="internals" module="dagster" object="ExecutorDefinition" /> を定義します。 |
| <PyObject section="internals" module="dagster" object="ExecutorDefinition" /> | Executor の定義 |

## Executor の指定

- [ジョブに対して直接指定](#directly-on-jobs)
- [コードの場所に対して指定](#for-a-code-location)

### ジョブに対して直接指定 {#directly-on-jobs}

すべてのジョブにはエグゼキューターが存在します。デフォルトのエグゼキューターは <PyObject section="execution" module="dagster" object="multi_or_in_process_executor" /> で、デフォルトでは各ステップを自身のプロセス内で実行します。
このエグゼキューターは、同じプロセス内で各ステップを実行するように設定できます。

エグゼキューターは、<PyObject section="jobs" module="dagster" object="job" decorator /> または <PyObject section="graphs" module="dagster" object="GraphDefinition" method="to_job" /> の `​​executor_def` パラメーターに <PyObject section="internals" module="dagster" object="ExecutorDefinition" /> を指定することで、ジョブに直接指定できます。

<CodeExample
  path="docs_snippets/docs_snippets/deploying/executors/executors.py"
  startAfter="start_executor_on_job"
  endBefore="end_executor_on_job"
/>

### コードの場所に対して指定 {#for-a-code-location}

コードロケーションに提供されるすべてのジョブとアセットのデフォルトのエグゼキュータを指定するには、<PyObject section="definitions" module="dagster" object="Definitions" /> オブジェクトに `executor` 引数を指定します。

ジョブでエグゼキュータが明示的に指定されている場合は、そのエグゼキュータが使用されます。
エグゼキュータを指定していないジョブでは、コードロケーションに提供されるデフォルトのエグゼキュータが使用されます。

<CodeExample
  path="docs_snippets/docs_snippets/deploying/executors/executors.py"
  startAfter="start_executor_on_repo"
  endBefore="end_executor_on_repo"
/>

:::note

<PyObject section="jobs" module="dagster" object="JobDefinition" method="execute_in_process" /> 経由でジョブを実行すると、ジョブのエグゼキュータがオーバーライドされ、代わりに <PyObject section="execution" module="dagster" object="in_process_executor" /> が使用されます。

:::

## エグゼキュータの例

| Name                                                                                            | Description                                                                                                           |
| ----------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------- |
| <PyObject section="execution" module="dagster" object="in_process_executor" />                  | 実行プランは実行ワーカー自体内で順次実行されます。        |
| <PyObject section="execution" module="dagster" object="multiprocess_executor" />                | 各ステップは、独自に生成されたプロセス内で実行されます。並列処理のレベルは設定可能です。 |
| <PyObject section="libraries" module="dagster_dask" object="dask_executor" />                   | Dask タスク内の各ステップを実行します。  |
| <PyObject section="libraries" module="dagster_celery" object="celery_executor" />               | Celery タスク内の各ステップを実行します。         |
| <PyObject section="libraries" module="dagster_docker" object="docker_executor" />               | 一時的な Docker コンテナ内で各ステップを実行します。         |
| <PyObject section="libraries" module="dagster_k8s" object="k8s_job_executor" />                 | 一時的な Kubernetes ポッド内で各ステップを実行します。        |
| <PyObject section="libraries" module="dagster_celery_k8s" object="celery_k8s_job_executor" />   | 優先順位付けとキューイングのコントロール プレーンとして Celery を使用して、一時的な Kubernetes ポッド内で各ステップを実行します。 |
| <PyObject section="libraries" module="dagster_celery_docker" object="celery_docker_executor" /> | 優先順位付けとキューイングのコントロール プレーンとして Celery を使用して、Docker コンテナ内で各ステップを実行します。      |

## カスタムエグゼキューター

エグゼキューターシステムはプラグイン可能です。つまり、異なる実行サブストレートをターゲットとする独自のエグゼキューターを作成することが可能です。
ただし、現時点では十分にドキュメント化されておらず、内部APIも流動的であることにご注意ください。
