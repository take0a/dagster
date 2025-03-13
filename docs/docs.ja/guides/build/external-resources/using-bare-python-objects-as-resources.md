---
title: 裸のPythonオブジェクトをリソースとして使用する
sidebar_position: 800
---

アセットまたはジョブのセットの構築を開始するときに、サードパーティの API クライアントなどのリソースとして、構成なしの Python オブジェクトを使用する必要がある場合があります。

Dagster は、プレーンな Python オブジェクトをリソースとして渡すことをサポートしています。これは、<PyObject section="resources" module="dagster" object="ConfigurableResource"/> サブクラスを使用する場合と同様のパターンに従います。ただし、これらのリソースを使用するアセットは、`ResourceParam` を使用して [注釈](https://docs.python.org/3/library/typing.html#typing.Annotated) する必要があります。この注釈により、Dagster は、パラメーターがリソースであり、アップストリーム入力ではないことを認識します。

{/* TODO replace `ResourceParam` with <PyObject section="resources" module="dagster" object="ResourceParam"/>  */}

{/* TODO add dedent=4 prop when implemented */}
<CodeExample path="docs_snippets/docs_snippets/concepts/resources/pythonic_resources.py" startAfter="start_raw_github_resource" endBefore="end_raw_github_resource" />
