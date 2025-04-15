---
title: '実行再試行の設定'
sidebar_position: 600
---

実行の再試行を設定すると、何らかの理由で実行が失敗するたびに新しい実行が開始されます。
[op retries](/guides/build/ops/op-retries) と比較すると、実行の再試行の最大再試行回数制限は、個々の op ではなく実行全体に適用されます。
実行の再試行は、実行プロセスがクラッシュしたり、予期せず終了したりした場合にも対処します。

## 設定

実行の再試行の設定方法は、Dagster+ と Dagster Open Source のどちらを使用しているかによって異なります:

- **Dagster+**: [Dagster+ UI または dagster-cloud CLI](/dagster-plus/deployment/management/deployments/deployment-settings-reference) を使用して、デフォルトの最大再試行回数を設定します。実行の再試行を明示的に有効にする必要はありません。
- **Dagster Open Source**: インスタンスの `dagster.yaml` を使用して、実行の再試行を有効にします。

例えば、以下のコマンドは、すべての実行に対してデフォルトの最大再試行回数を `3` に設定します:

```yaml
run_retries:
  enabled: true # Omit this key if using Dagster+, since run retries are enabled by default
  max_retries: 3
```

Dagster+ と Dagster Open Source の両方で、ジョブ定義または Dagster UI [Launchpad](/guides/operate/webserver) でタグを使用して再試行を構成することもできます。

<CodeExample path="docs_snippets/docs_snippets/deploying/job_retries.py" />

### 再試行戦略

`dagster/retry_strategy` タグは、再試行時に実行するオペレーションを制御します。

デフォルトでは、再試行は失敗から再実行されます（タグ値が `FROM_FAILURE`）。
つまり、成功したオペレーションはスキップされますが、その出力は下流のオペレーションに使用されます。
`dagster/retry_strategy` タグが `ALL_STEPS` に設定されている場合、すべてのオペレーションが再実行されます。

:::note

`FROM_FAILURE` には、他の実行からの出力にアクセスできる I/O マネージャーが必要です。
たとえば、Kubernetes では <PyObject section="libraries" object="s3.s3_pickle_io_manager" module="dagster_aws" /> は機能しますが、 <PyObject section="io-managers" object="FilesystemIOManager" module="dagster" /> は機能しません。これは、新しい実行が別のファイルシステムを持つ新しい Kubernetes ジョブ内にあるためです。

:::

### オペレーションと実行の再試行の組み合わせ

デフォルトでは、オペレーションの失敗により実行が失敗し、オペレーションと実行の両方の再試行が有効になっている場合、再試行が重複することで、オペレーションが想定よりも多く再試行される可能性があります。
これは、オペレーションの再試行回数が再試行されるたびにリセットされるためです。

これを防ぐには、クラッシュや実行ワーカーの予期せぬ終了など、オペレーションの失敗以外の理由で失敗した場合にのみ実行の再試行を行うように設定できます。
この動作は `run_retries.retry_on_asset_or_op_failure` 設定によって制御されます。この設定はデフォルトで `true` ですが、 `false` にオーバーライドできます。

例えば、以下の設定では、ステップの失敗により失敗した実行を無視するように実行の再試行を設定します:

```yaml
run_retries:
  enabled: true # Omit this key if using Dagster+, since run retries are enabled by default
  max_retries: 3
  retry_on_asset_or_op_failure: false
```

また、タグを使用して特定のジョブに `dagster/retry_on_asset_or_op_failure` タグを適用し、そのジョブの実行のデフォルト値を上書きすることもできます:

```python
from dagster import job


@job(tags={"dagster/max_retries": 3, "dagster/retry_on_asset_or_op_failure": False})
def sample_job():
    pass
```

:::note

`retry_on_asset_or_op_failure` を `false` に設定すると、Dagster バージョン 1.6.7 以降での実行の再試行動作のみが変更されます。

:::
