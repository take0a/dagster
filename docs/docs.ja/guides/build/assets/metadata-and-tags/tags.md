---
title: "タグ"
sidebar_position: 1000
---

**タグ** は、Dagster でアセットを整理するための主な方法です。アセットを定義するときに複数のタグを添付することができ、それらは UI に表示されます。また、タグを使用して、Dagster+ の [アセットカタログ](/dagster-plus/features/asset-catalog/) 内のアセットを検索およびフィルタリングすることもできます。タグは、文字列のキーと値のペアとして構造化されています。

アセットに適用できるタグの例を次に示します:

```python
{"domain": "marketing", "pii": "true"}
```

`owners` と同様に、アセットを定義するときに、`tags` 引数にタグの辞書を渡すだけです。

<CodeExample path="docs_snippets/docs_snippets/guides/data-modeling/metadata/tags.py" language="python" />

タグにはキーと値として文字列のみを含める必要があることに注意してください。また、Dagster UI は、空の文字列を含むタグをキーと値のペアではなく「ラベル」としてレンダリングします。

### タグのキー

有効なタグキーには、オプションのプレフィックスと名前の 2 つのセグメントがあり、スラッシュ (`/`) で区切られます。プレフィックスは通常、Python パッケージ名であることが期待されます。例: `dagster/priority`

プレフィックスと名前セグメントはそれぞれ次の条件を満たす必要があります:

- 63文字以内
- 英数字、ダッシュ (`-`)、アンダースコア (`_`)、ドット (`.`) のみを含めます。

### タグの値

タグの値は、次の条件を満たす必要があります:

- 63文字以内
- 英数字、ダッシュ (`-`)、アンダースコア (`_`)、ドット (`.`) のみを含めます。
- 文字列または文字列にシリアル化可能なJSONであること

### Labels

A label is a tag that only contains a key. To create a label, set the tag value to an empty string:

```python
@dg.asset(
    tags={"private":""}
)
def my_asset() -> None:
    ...
```

A label will look like the following in the UI:

![Label in UI](/images/guides/build/assets/metadata-tags/label-ui.png)

### タグを使用して実行をカスタマイズする

タグは主にラベル付けと整理に使用されますが、実行機能の一部は実行タグを使用して制御されます。

- [Customizing Kubernetes config](/guides/deploy/deployment-options/kubernetes/customizing-your-deployment)
- [Specifying Celery config](/guides/deploy/deployment-options/kubernetes/kubernetes-and-celery)
- [Setting concurrency limits when using the `QueuedRunCoordinator`](/guides/operate/managing-concurrency)
- [Setting the priority of different runs](/guides/deploy/execution/customizing-run-queue-priority)

### システムタグ

#### アセットタグ

次の表は、Dagster がアセットに自動的に追加する可能性のあるタグの一覧です。

| Tag                   | Description             |
| --------------------- | ---------------------- |
| `dagster/kind/{kind}` | アセットが特定の種類であることを識別するタグ。詳細については、[種類タグ](kind-tags) を参照してください。 |

### 実行タグ

次の表は、Dagster が実行時に自動的に追加するタグの一覧です。

| Tag                     | Description                         |
| ----------------------- | ----------------------------------- |
| `dagster/op_selection`  | 実行時のオペレーション選択        |
| `dagster/partition`     | 実行のパーティション           |
| `dagster/schedule_name` | 実行のきっかけとなったスケジュール |
| `dagster/sensor_name`   | 走行を開始したセンサー   |
| `dagster/backfill`      | バックフィルID                     |
| `dagster/parent_run_id` | 再実行された実行の親実行 |
| `dagster/image`         | Dockerイメージタグ               |
