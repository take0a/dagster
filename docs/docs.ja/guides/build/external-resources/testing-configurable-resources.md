---
title: 構成可能なリソースのテスト
sidebar_position: 700
---

<PyObject section="resources" module="dagster" object="ConfigurableResource"/> の初期化は、手動で構築することでテストできます。ほとんどの場合、リソースは直接構築できます:

{/* TODO add dedent=4 prop when implemented */}
<CodeExample path="docs_snippets/docs_snippets/concepts/resources/pythonic_resources.py" startAfter="start_new_resource_testing" endBefore="end_new_resource_testing" />

リソースに他のリソースが必要な場合は、それらをコンストラクター引数として渡すことができます:

{/* TODO add dedent=4 prop when implemented */}
<CodeExample path="docs_snippets/docs_snippets/concepts/resources/pythonic_resources.py" startAfter="start_new_resource_testing_with_nesting" endBefore="end_new_resource_testing_with_nesting" />

## リソースコンテキストによるテスト

リソースがリソース初期化コンテキストを使用する場合は、リソース クラスの `with_init_resource_context` ヘルパーと一緒に <PyObject section="resources" module="dagster" object="build_init_resource_context"/> ユーティリティを使用できます:

{/* TODO add dedent=4 prop when implemented */}
<CodeExample path="docs_snippets/docs_snippets/concepts/resources/pythonic_resources.py" startAfter="start_new_resource_testing_with_context" endBefore="end_new_resource_testing_with_context" />
