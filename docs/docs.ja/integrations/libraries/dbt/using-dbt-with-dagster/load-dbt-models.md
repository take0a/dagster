---
title: 'dbt モデルを Dagster アセットとしてロードする'
description: Dagster can orchestrate dbt alongside other technologies.
sidebar_position: 200
---

この時点で、Dagster で使用できる [完全に構成された dbt プロジェクト](/integrations/libraries/dbt/using-dbt-with-dagster/set-up-dbt-project) が完成しているはずです。

このセクションでは、最終的に dbt と Dagster の統合を開始します。これを行うには、次の操作を行います。

- [dbt プロジェクトをラップする Dagster プロジェクトを作成する](#step-1-create-a-dagster-project-that-wraps-your-dbt-project)
- [Dagster の UI で Dagster プロジェクトを検査する](#step-2-inspect-your-dagster-project-in-dagsters-ui)
- [Dagster で dbt モデルを構築する](#step-3-build-your-dbt-models-in-dagster)
- [Dagster プロジェクトの Python コードを理解する](#step-4-understand-the-python-code-in-your-dagster-project)

## Step 1: dbt プロジェクトをラップする Dagster プロジェクトを作成する {#step-1-create-a-dagster-project-that-wraps-your-dbt-project}

`dagster-dbt` コマンドライン インターフェイスを使用して、dbt プロジェクトをラップする Dagster プロジェクトを作成できます。`dbt_project.yml` があるディレクトリにいることを確認してください。前のセクションから続行している場合は、すでにこのディレクトリにいるはずです。次に、以下を実行します:

```shell
dagster-dbt project scaffold --project-name jaffle_dagster
```

これにより、現在のディレクトリ内に `jaffle_dagster/` というディレクトリが作成されます。`jaffle_dagster/` ディレクトリには、Dagster プロジェクトを定義する一連のファイルが含まれています。

一般的に、Dagster プロジェクトをどこに置くかはあなた次第です。Dagster プロジェクトを Git リポジトリのルートに置くのが最も一般的です。したがって、この場合、`dbt_project.yml` が `jaffle_shop` Git リポジトリのルートにあったため、そこに Dagster プロジェクトを作成しました。

**注**: `dagster-dbt project scaffold` コマンドは、実行したディレクトリに Dagster プロジェクトを作成します。それが `dbt_project.yml` があるディレクトリと異なる場合は、Dagster が dbt プロジェクトを探す場所を認識できるように、`--dbt-project-dir` オプションの値を指定する必要があります。

## Step 2: Dagster の UI で Dagster プロジェクトを検査する {#step-2-inspect-your-dagster-project-in-dagsters-ui}

Dagster プロジェクトが作成されたので、Dagster の UI を実行して確認することができます。

1. ディレクトリを Dagster プロジェクト ディレクトリに変更します:

   ```shell
   cd jaffle_dagster/
   ```

2. Dagster の UI を起動するには、次のコマンドを実行します:

   ```shell
   dagster dev
   ```

   次のような出力が得られます:

   ```shell
   Serving dagster-webserver on http://127.0.0.1:3000 in process 70635
   ```

3. ブラウザで [http://127.0.0.1:3000](http://127.0.0.1:3000) に移動します。ページに次のアセットが表示されます:

![Asset graph in Dagster's UI, containing dbt models loaded as Dagster assets](/images/integrations/dbt/using-dbt-with-dagster/load-dbt-models/asset-graph.png)

## Step 3: Dagster で dbt モデルを構築する {#step-3-build-your-dbt-models-in-dagster}

Dagster では、dbt モデルを表示するだけでなく、実行することもできます。Dagster では、dbt モデルを実行することは、アセットを _具体化_ することに対応します。アセットを具体化するということは、永続ストレージ内のコンテンツを更新するために何らかの計算を実行することを意味します。このチュートリアルでは、その永続ストレージはローカルの DuckDB データベースです。

dbt プロジェクトをビルドする、つまりアセットを具体化するには、ページの右上隅にある [**Materialize all**] ボタンをクリックします。これにより、アセットを具体化する実行が開始されます。完了すると、アセットの [**Materialized**] 属性と [**Latest Run**] 属性が入力されます。

![Asset graph in Dagster's UI, showing materialized assets](/images/integrations/dbt/using-dbt-with-dagster/load-dbt-models/asset-graph-materialized.png)

実行が完了したら、次の操作を実行できます。

- **asset** をクリックすると、アセットに関する情報 (最後のマテリアライズ統計や **Asset details** ページを表示するためのリンクなど) を含むサイドバーが開きます。
- アセット内の **Latest Run** の ID をクリックすると、**Run details** ページが表示されます。このページには、タイミング情報、エラー、ログなど、実行に関する詳細情報が含まれています。

## Step 4: Dagster プロジェクトの Python コードを理解する {#step-4-understand-the-python-code-in-your-dagster-project}

dbt プロジェクトをロードする Dagster プロジェクトを作成する方法を説明しました。これはどのように機能しますか? Dagster が dbt プロジェクトをロードする方法を理解することで、Dagster が dbt プロジェクトを実行する方法をカスタマイズしたり、dbt 以外の他のデータ アセットに接続したりするための基礎が得られます。

最も重要なファイルは、Dagster がロードする定義のセットを含む Python ファイル `jaffle_shop/definitions.py` です。Dagster はこのファイル内のコードを実行して、認識する必要があるアセットとそれらのアセットの詳細を調べます。たとえば、前の手順で `dagster dev` を実行したとき、Dagster はこのファイル内のコードを実行して、UI に表示するアセットを決定しました。

`definitions.py` Python ファイルでは、dbt モデルを Dagster アセットとしてモデル化するコードを含む `assets.py` からインポートします。各 dbt モデルの Dagster アセットを返すには、この `assets.py` ファイルのコードで、所有している dbt モデルを把握する必要があります。`manifest.json` というファイルを読み取ることで、所有しているモデルを特定します。このファイルは、dbt が任意の dbt プロジェクトに対して生成できるファイルで、プロジェクト内のすべてのモデル、シード、スナップショット、テストなどの情報が含まれています。

`manifest.json` を取得するために、`assets.py` は `project.py` からインポートします。これは、dbt プロジェクトの内部表現を定義します。次に、`assets.py` で、`jaffle_shop_project.manifest_path` を使用して、`manifest.json` ファイルへのパスにアクセスできます:

<CodeExample
  path="docs_snippets/docs_snippets/integrations/dbt/tutorial/load_dbt_models/project.py"
  startAfter="start_load_project"
  endBefore="end_load_project"
/>

dbt プロジェクトの `manifest.json` ファイルの生成には時間がかかるため、この Python モジュールをインポートするたびに生成するのは避けた方がよいでしょう。したがって、Dagster の運用環境では通常、コードをパッケージ化する CI/CD システムで `manifest.json` を生成します。

ただし、開発環境では通常、dbt プロジェクトのファイルに加えた変更を、マニフェストを再生成せずに Dagster UI にすぐに反映させたいものです。

`jaffle_shop_project.prepare_if_dev()` はこれに役立ちます。Dagster がコードをインポートするときに `manifest.json` を再生成しますが、_ただし_ `dagster dev` コマンドによってインポートされている場合のみです。

`manifest.json` ファイルを作成したら、それを使用して Dagster アセットを定義します。プロジェクトの `assets.py` にある次のコードは、次のことを実行します:

<CodeExample
  path="docs_snippets/docs_snippets/integrations/dbt/tutorial/load_dbt_models/assets.py"
  startAfter="start_dbt_assets"
  endBefore="end_dbt_assets"
/>

このコードはデコレータを使用しているため、少し凝ったものに見えるかもしれません。何が起こっているかの内訳は次のとおりです。

- Dagster アセットのセットを表すオブジェクトを保持する `jaffle_shop_dbt_assets` という変数を作成します。
- これらの Dagster アセットは、マニフェスト ファイルに記述されている dbt モデルを反映します。マニフェスト ファイルは、`manifest` 引数を使用して渡されます。
- デコレートされた関数は、これらの Dagster アセットの 1 つをマテリアライズするときに何が起こるかを定義します。たとえば、UI の **Materialize** ボタンをクリックするか、スケジュールに入れて自動的にマテリアライズします。この場合、選択されたアセットに対して `dbt build` コマンドが呼び出されます。`dbt build` とともに提供される `context` パラメータは、選択内容を保持します。

後で dbt モデルを Dagster アセットに変換する方法をカスタマイズする場合は、`assets.py` でその定義を編集します。

## 次は？

この時点で、dbt モデルをアセットとして Dagster に読み込み、Dagster のアセット グラフ UI で表示し、マテリアライズしました。次に、[上流の Dagster アセットを追加する](/integrations/libraries/dbt/using-dbt-with-dagster/upstream-assets) 方法を学習します。
