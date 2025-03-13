---
title: "Dagster パイプの使用"
description: "Learn how to use Dagster Pipes's built-in subprocess implementation to invoke a subprocess with a given command and environment"
sidebar_position: 10
---

このガイドでは、[Dagster Pipes](/guides/build/external-pipelines/) を Dagster の組み込みサブプロセス <PyObject section="pipes" module="dagster" object="PipesSubprocessClient" /> とともに使用して、指定されたコマンドと環境でローカル サブプロセスを実行する方法を説明します。その後、構造化メタデータやログなどの情報をサブプロセスから Dagster に送信し、Dagster UI で表示できるようになります。

そうするには、次のことが必要です:

- [サブプロセスを呼び出すDagsterアセットを作成する](create-subprocess-asset)
- [既存のコードを修正して、Dagster パイプと連携し、情報を Dagster に送り返す](modify-external-code)
- DagsterパイプをDagsterシステム内の他のエンティティと併用する方法については、[リファレンス](reference)セクションを参照してください。

:::note

このガイドでは、すぐに使用できる <PyObject section="pipes" module="dagster" object="PipesSubprocessClient" /> リソースの使用に焦点を当てています。サブプロセス呼び出しをさらにカスタマイズするには、代わりに <PyObject section="libraries" module="dagster_pipes" object="open_dagster_pipes"/> アプローチを使用します。詳細については、[Dagster Pipes プロトコルのカスタマイズ](/guides/build/external-pipelines/dagster-pipes-details-and-customization)を参照してください。

:::

## 前提条件

Dagster パイプを使用してサブプロセスを実行するには、Dagster (`dagster`) と Dagster UI (`dagster-webserver`) がインストールされている必要があります。詳細については、[インストール ガイド](/getting-started/installation)を参照してください。

また、**既存の Python スクリプト** も必要です。次の Python スクリプトを使用して説明します。このファイルは、このチュートリアルの後半で作成する Dagster アセットによって呼び出されます。

`external_code.py` という名前のファイルを作成し、次の内容を貼り付けます:

<CodeExample path="docs_snippets/docs_snippets/guides/dagster/dagster_pipes/subprocess/part_1/external_code.py" lineStart="3" />

## 始める準備はできましたか？

チュートリアルの前提条件をすべて満たしたら、[サブプロセスを実行する Dagster アセットを作成](create-subprocess-asset)して開始できます。
