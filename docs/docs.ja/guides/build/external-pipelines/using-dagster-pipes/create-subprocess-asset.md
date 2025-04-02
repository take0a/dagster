---
title: "サブプロセスを呼び出すDagsterアセットを定義する"
description: "Learn how to create a Dagster asset that invokes a subprocess that executes external code."
sidebar_position: 100
---

:::note

これは、[Dagster パイプの使用](/guides/build/external-pipelines/using-dagster-pipes) チュートリアルのパート 1 です。すでに Dagster によってオーケストレーションされている既存のコードを変更する方法を探している場合は、パート 2 の [外部コードの変更](/guides/build/external-pipelines/using-dagster-pipes/modify-external-code) に進んでください。

:::note

チュートリアルのこの部分では、実行関数で Dagster パイプ セッションを開き、外部コードを実行するサブプロセスを呼び出す Dagster アセットを作成します。

:::

## Step 1: Dagsterアセットを定義する

始める前に、チュートリアルの [前提条件](/guides/build/external-pipelines/using-dagster-pipes#prerequisites) をすべて満たしていることを確認してください。次のような `external_code.py` という名前のスタンドアロン Python スクリプトが必要です:

<CodeExample path="docs_snippets/docs_snippets/guides/dagster/dagster_pipes/subprocess/part_1/external_code.py" lineStart="3" />

### Step 1.1: アセットを定義する

まず、[前提条件](/guides/build/external-pipelines/using-dagster-pipes#prerequisites) の手順で作成した `external_code.py` ファイルと同じディレクトリに `dagster_code.py` という名前の新しいファイルを作成します。

次に、アセットを定義します。次の内容をコピーしてファイルに貼り付けます:

<CodeExample path="docs_snippets/docs_snippets/guides/dagster/dagster_pipes/subprocess/part_1/dagster_code.py" startAfter="start_asset_marker" endBefore="end_asset_marker" />

この例では次の操作を実行しました:

- `subprocess_asset`という名前のアセットを作成しました
- アセットの `context` 引数として <PyObject section="execution" module="dagster" object="AssetExecutionContext" /> を指定します。このオブジェクトは、リソース、構成、ログなどのシステム情報を提供します。これについては、このセクションの後半で説明します。
- アセットが使用するリソース「PipesSubprocessClient」を指定しました。これについては後ほど説明します。
- 外部スクリプトを実行するためのコマンド リスト `cmd` を宣言しました。リスト内は:
  - まず、`shutil.which("python")` を使用して、システム上の Python 実行可能ファイルへのパスを見つけます。
  - 次に、実行するファイルへのファイル パスを指定します。この場合、それは先ほど作成した `external_code.py` ファイルです。

### Step 1.2: アセットから外部コードを呼び出す

次に、`pipes_subprocess_client` リソースを使用して、アセットから外部コードを実行するサブプロセスを呼び出します:


<CodeExample path="docs_snippets/docs_snippets/guides/dagster/dagster_pipes/subprocess/part_1/dagster_code.py" startAfter="start_asset_marker" endBefore="end_asset_marker" />

このコードが何をするのか見てみましょう:

- アセットによって使用される `PipesSubprocessClient` リソースは `run` メソッドを公開します。
- アセットが実行されると、このメソッドはパイプセッション内でサブプロセスを同期的に実行し、`PipesClientCompletedInvocation` オブジェクトを返します。
- このオブジェクトには `get_materialize_result` メソッドが含まれており、これを使用してサブプロセスによって報告された <PyObject section="assets" module="dagster" object="MaterializeResult" /> イベントにアクセスできます。次のセクションでは、サブプロセスからイベントを報告する方法について説明します。
- 最後に、サブプロセスの結果を返します。

## Step 2: 定義オブジェクトで定義する

アセットとサブプロセスのリソースを、CLI、UI、Dagster+ などの Dagster のツールで読み込み、アクセスできるようにするには、それらを含む <PyObject section="definitions" module="dagster" object="Definitions" /> オブジェクトを作成します。

次のコードをコピーして、`dagster_code.py` の下部に貼り付けます:

<CodeExample path="docs_snippets/docs_snippets/guides/dagster/dagster_pipes/subprocess/part_1/dagster_code.py" startAfter="start_definitions_marker" endBefore="end_definitions_marker" />

この時点で、`dagster_code.py` は次のようになります:

<CodeExample path="docs_snippets/docs_snippets/guides/dagster/dagster_pipes/subprocess/part_1/dagster_code_finished.py" />

## Step 3: Dagster UIからサブプロセスを実行する

このステップでは、前のステップで作成したサブプロセス アセットを Dagster UI から実行します。

1. 新しいコマンド ライン セッションで、次のコマンドを実行して UI を起動します:

   ```bash
   dagster dev -f dagster_code.py
   ```

2. [http://localhost:3000](http://localhost:3000) に移動すると、UI が表示されます:

    ![Asset in the UI](/images/guides/build/external-pipelines/subprocess/part-1-step-3-2-asset.png)

3. コードを実行するには、右上にある **Materialize** をクリックします:

    ![Materialize asset](/images/guides/build/external-pipelines/subprocess/part-1-step-3-3-materialize.png)

4. **Run details** ページに移動すると、実行のログが表示されます:

   ![Logs in the run details page](/images/guides/build/external-pipelines/subprocess/part-1-step-3-4-logs.png)

5. `external_code.py` には、`stdout` に出力する `print` ステートメントがあります。Dagster はこれらを UI の生の計算ログビューに表示します。`stdout` ログを表示するには、ログセクションを **stdout** に切り替えます:

   ![Raw compute logs in the run details page](/images/guides/build/external-pipelines/subprocess/part-1-step-3-5-stdout.png)

## 次は？

この時点で、外部 Python スクリプトを呼び出し、サブプロセスでコードを起動し、結果を Dagster UI で表示する Dagster アセットを作成しました。次に、[Dagster Pipes で動作するように外部コードを変更](/guides/build/external-pipelines/using-dagster-pipes/modify-external-code)して Dagster に情報を送り返す方法を学習します。
