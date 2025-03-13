---
title: リソース状態の管理
sidebar_position: 900
---

リソースが一定の複雑さに達すると、そのリソースの存続期間中にその状態を管理する必要が生じる場合があります。これは、特別な初期化やクリーンアップを必要とするリソースに役立ちます。`ConfigurableResource` は、構成をカプセル化するためのデータ クラスですが、リソースの状態を管理するためのライフサイクル フックも提供します。

Pydantic の [`PrivateAttr`](https://docs.pydantic.dev/latest/usage/models/#private-model-attributes) を使用して、プライベート状態属性をマークできます。これらの属性はアンダースコアで始まる必要があり、リソースの構成には含まれません。

## ライフサイクルフック

Dagster 実行中にリソースが初期化されると、`setup_for_execution` メソッドが呼び出されます。このメソッドには、リソースの構成とその他の実行情報を含む <PyObject section="resources" module="dagster" object="InitResourceContext" /> オブジェクトが渡されます。リソースはこのコンテキストを使用して、実行中に必要な状態を初期化できます。

リソースが不要になると、`teardown_after_execution` メソッドが呼び出されます。このメソッドには、`setup_for_execution` と同じコンテキスト オブジェクトが渡されます。このメソッドは、`setup_for_execution` で初期化された状態をクリーンアップするのに役立ちます。

`setup_for_execution` と `teardown_after_execution` は、実行ごとにプロセスごとにそれぞれ 1 回呼び出されます。インプロセス エグゼキュータを使用する場合、これは実行ごとに 1 回呼び出されることを意味します。マルチプロセス エグゼキュータを使用する場合、リソースの各プロセスのインスタンスが初期化され、破棄されます。

次の例では、構成で指定されたユーザー名とパスワードに基づいて、クライアント リソースの API トークンを設定します。その後、API トークンを使用して、アセット本体の API をクエリできます。

{/* TODO add dedent=4 prop when implemented */}
<CodeExample path="docs_snippets/docs_snippets/concepts/resources/pythonic_resources.py" startAfter="start_with_state_example" endBefore="end_with_state_example" />

より複雑なユースケースでは、`yield_for_execution` をオーバーライドできます。デフォルトでは、このコンテキスト マネージャーは `setup_for_execution` を呼び出し、リソースを生成してから `teardown_after_execution` を呼び出しますが、これをオーバーライドしてカスタム動作を提供することもできます。これは、データベース接続やファイル ハンドルなど、実行中にコンテキストを開いている必要があるリソースに役立ちます。

{/* TODO add dedent=4 prop when implemented */}
<CodeExample path="docs_snippets/docs_snippets/concepts/resources/pythonic_resources.py" startAfter="start_with_complex_state_example" endBefore="end_with_complex_state_example" />
