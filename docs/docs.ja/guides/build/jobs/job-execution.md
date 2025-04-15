---
title: ジョブ実行
description: Dagster provides several methods to execute jobs.
sidebar_position: 300
---

:::note

このガイドは、[ops](/guides/build/ops/)と[jobs](/guides/build/jobs/)の両方に適用されます。

:::

Dagster は、[op](/guides/build/jobs/op-jobs) と [asset jobs](/guides/build/jobs/asset-jobs) を実行するための複数の方法を提供しています。
このガイドでは、Dagster UI、コマンドライン、または Python API を使用してジョブを 1 回限り実行するさまざまな方法について説明します。

ジョブは他の方法でも起動できます。

- [スケジュール](/guides/automate/schedules/)を使用すると、一定の間隔で実行を開始できます。
- [センサー](/guides/automate/sensors/)を使用すると、外部の状態変化に基づいて実行を開始できます。

## 関連API

| Name                                                            | Description                                                                        |
| --------------------------------------------------------------- | ---------------------------------------------------------------------------------- |
| <PyObject section="jobs" module="dagster" object="JobDefinition.execute_in_process" /> | 通常はスクリプトの実行やテストのために、ジョブを同期的に実行するメソッド。|

## ジョブの実行

Dagsterは、以下の方法で単発ジョブを実行できます。詳細については、タブをクリックしてください。

<Tabs>
<TabItem value="Dagster UI">

Dagster UI を使用すると、ジョブを表示、操作、実行できます。

UI でジョブを表示するには、[`dagster dev`](/api/python-api/cli#dagster-dev) コマンドを使用します:

```bash
dagster dev -f my_job.py
```

次に `http://localhost:3000` に移動します:

![Pipeline def](/images/guides/build/ops/pipeline-def.png)

**Launchpad** タブをクリックし、**Launch Run** ボタンを押してジョブを実行します:

![Job run](/images/guides/build/ops/pipeline-run.png)

デフォルトでは、Dagster は <PyObject section="execution" module="dagster" object="multiprocess_executor" /> を使用してジョブを実行します。つまり、ジョブ内の各ステップはそれぞれ独自のプロセスで実行され、相互に依存しないステップは並列実行できます。

Launchpad には、対話形式で設定を作成できる設定エディタも用意されています。
詳細については、[実行構成のドキュメント](/guides/operate/configuration/run-configuration#specifying-runtime-configuration) を参照してください。

</TabItem>
<TabItem value="コマンドライン">

dagster CLI には、ジョブ実行用の以下のコマンドが含まれています:

- [`dagster job execute`](/api/python-api/cli#dagster-job) は直接実行します。
- [`dagster job launch`](/api/python-api/cli#dagster-job) はインスタンス上の [run launcher](/guides/deploy/execution/run-launchers) を使用して非同期的に実行を開始します。

ジョブを直接実行するには、次のコマンドを実行します:

```bash
dagster job execute -f my_job.py
```

</TabItem>
<TabItem value="Python">

### Python APIs

Dagster には、テストやスクリプトの作成に役立つ実行用の Python API が含まれています。

<PyObject section="jobs" module="dagster" object="JobDefinition.execute_in_process" /> はジョブを実行し、<PyObject section="execution" module="dagster" object="ExecuteInProcessResult" /> を返します。

<CodeExample path="docs_snippets/docs_snippets/concepts/ops_jobs_graphs/job_execution.py" startAfter="start_execute_marker" endBefore="end_execute_marker" />

完全な API ドキュメントは [実行 API](/api/python-api/execution) で参照でき、テストのユースケースの詳細については [テスト ドキュメント](/guides/test/) を参照してください。

</TabItem>
</Tabs>

## ジョブのサブセットの実行

Dagster は、**op 選択** と呼ばれるジョブのサブセットを実行する方法をサポートしています。

### Op selection syntax

To specify op selection, Dagster supports a simple query syntax.

It works as follows:

- A query includes a list of clauses.
- A clause can be an op name, in which case that op is selected.
- A clause can be an op name preceded by `*`, in which case that op and all of its ancestors (upstream dependencies) are selected.
- A clause can be an op name followed by `*`, in which case that op and all of its descendants (downstream dependencies) are selected.
- A clause can be an op name followed by any number of `+`s, in which case that op and descendants up to that many hops away are selected.
- A clause can be an op name preceded by any number of `+`s, in which case that op and ancestors up to that many hops away are selected.

Let's take a look at some examples:

| Example      | Description                                                                                         |
| ------------ | --------------------------------------------------------------------------------------------------- |
| `some_op`    | Select `some_op`                                                                                    |
| `*some_op`   | Select `some_op` and all ancestors (upstream dependencies).                                         |
| `some_op*`   | Select `some_op` and all descendants (downstream dependencies).                                     |
| `*some_op*`  | Select `some_op` and all of its ancestors and descendants.                                          |
| `+some_op`   | Select `some_op` and its direct parents.                                                            |
| `some_op+++` | Select `some_op` and its children, its children's children, and its children's children's children. |

### Specifying op selection

Use this selection syntax in the `op_selection` argument to the <PyObject section="jobs" module="dagster" object="JobDefinition.execute_in_process" />:

<CodeExample path="docs_snippets/docs_snippets/concepts/ops_jobs_graphs/job_execution.py" startAfter="start_op_selection_marker" endBefore="end_op_selection_marker" />

Similarly, you can specify the same op selection in the Dagster UI Launchpad:

![Op selection](/images/guides/build/ops/solid-selection.png)

## ジョブ実行の制御

各 <PyObject section="jobs" module="dagster" object="JobDefinition" /> には、実行方法を決定する <PyObject section="internals" module="dagster" object="ExecutorDefinition" /> が含まれています。

この `executor_def` プロパティを設定することで、すべてのオペレーションを同じプロセスで実行することから、各オペレーションを独自の Kubernetes ポッドで実行することまで、さまざまなタイプの分離と並列処理が可能になります。詳細については、[エグゼキューター](/guides/operate/run-executors) を参照してください。

### デフォルトのジョブ実行プログラム

デフォルトのジョブ実行プログラム定義は、マルチプロセス実行に設定されています。
また、設定によってインプロセス実行とマルチプロセス実行を切り替えることもできます。

以下は、Dagster UI プレイグラウンドでインプロセス実行を開始するために指定できる、YAML 形式の実行設定の例です。

<CodeExample path="docs_snippets/docs_snippets/concepts/ops_jobs_graphs/job_execution.py" startAfter="start_ip_yaml" endBefore="end_ip_yaml" />

マルチプロセス実行には、パフォーマンス向上に役立つ追加の設定オプションが用意されています。
これには、同時実行可能なサブプロセスの最大数を制限したり、サブプロセスの生成方法を制御したりする機能が含まれます。

以下の例では、ジョブの実行設定を直接設定し、同時実行可能なサブプロセスの最大数を明示的に「4」に設定し、サブプロセスの開始方法をフォークサーバーを使用するように変更しています。

<CodeExample path="docs_snippets/docs_snippets/concepts/ops_jobs_graphs/job_execution.py" startAfter="start_mp_cfg" endBefore="end_mp_cfg" />

フォークサーバーの使用は、マルチプロセス実行時のプロセスごとのオーバーヘッドを削減する優れた方法ですが、特定のライブラリでは問題が発生する可能性があります。
詳細については、[Pythonドキュメント](https://docs.python.org/3/library/multiprocessing.html#contexts-and-start-methods)を参照してください。

#### オペレーションの同時実行数制限

`max_concurrent` 制限に加えて、`tag_concurrency_limits` を使用して、特定のタグを持つオペレーションを 1 回の実行で同時に実行できる数に制限を指定できます。

特定のタグキーまたはキーと値のペアを持つすべてのオペレーションに対して制限を指定できます。

オペレーションの実行によって制限を超える場合、そのオペレーションはキューに保持されます。
アセットジョブは、ジョブ内の各アセットの `op_tags` フィールドを参照して、タグの同時実行数制限を確認します。

例えば、次のジョブは、`database` タグが `redshift` であるオペレーションを最大 2 つ同時に実行しますが、同時に最大 4 つのオペレーションが実行されるようにします。

<CodeExample path="docs_snippets/docs_snippets/concepts/ops_jobs_graphs/job_execution.py" startAfter="start_tag_concurrency" endBefore="end_tag_concurrency" />

:::note

これらの制限は実行ごとにのみ適用されます。<PyObject section="libraries" module="dagster_celery" object="celery_executor" /> または <PyObject section="libraries" module="dagster_celery_k8s" object="celery_k8s_job_executor" /> を使用することで、複数の実行にわたってオペレーションの同時実行制限を適用できます。

オペレーションの同時実行と実行の同時実行を制限する方法の詳細については、[データパイプラインにおける同時実行の管理ガイド](/guides/operate/managing-concurrency) を参照してください。

:::