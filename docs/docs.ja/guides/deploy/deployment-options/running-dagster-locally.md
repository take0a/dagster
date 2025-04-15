---
title: 'Dagsterをローカルで実行する'
sidebar_label: Local deployment
description: How to run Dagster on your local machine.
sidebar_position: 1
---

このガイドでは、「dagster dev」コマンドを使ってローカルマシンでDagsterを実行する方法を解説します。「dagster dev」コマンドはDagster UIとDagsterデーモンを起動し、コマンドラインからDagsterの完全なデプロイメントを開始できます。

:::warning
`dagster dev` はローカル開発のみを対象としています。本番環境で Dagster を実行したい場合は、他の [デプロイメントガイド](/guides/deploy/deployment-options/index.md) をご覧ください。
:::

## コードを見つける

ローカル開発を開始する前に、アセットとジョブを含む Python コードを見つける方法を Dagster に伝える必要があります。

Dagster プロジェクトの設定方法を再確認するには、[推奨される Dagster プロジェクト構造](/guides/build/projects/structuring-your-dagster-project) ガイドに従ってください。

<Tabs>
  <TabItem value="module" label="モジュールから">
    Dagster は、コードの場所として Python モジュールを読み込むことができます。
    <CodeExample path="docs_snippets/docs_snippets/guides/tbd/definitions.py" language="python" title="my_module/__init__.py" />
    `-m` 引数を使用してモジュール名を指定し、定義がロードされた Dagster インスタンスを起動できます:
    ```shell
    dagster dev -m my_module
    ```

  </TabItem>
  <TabItem value="without-args" label="コマンドライン引数なし">
    `-m` コマンドライン引数を指定せずにモジュールから定義をロードするには、`pyproject.toml` ファイルを使用します。このファイルはすべての Dagster サンプルプロジェクトに含まれており、`tool.dagster` セクションに `module_name` を指定できます:
    <CodeExample path="docs_snippets/docs_snippets/guides/tbd/pyproject.toml" language="toml" title="pyproject.toml" />

  </TabItem>
  <TabItem value="file" label="ファイルから">
    Dagster は、ファイルをコードの場所として直接読み込むことができます。
    <CodeExample path="docs_snippets/docs_snippets/guides/tbd/definitions.py" language="python" title="definitions.py" />
    上記のファイルでは、`-f` 引数を使用してファイル名を指定し、定義がロードされた Dagster インスタンスを起動できます:
    ```shell
    dagster dev -f defs.py
    ```

    :::note
    Python インポート エラーのクラス全体を回避するために、実稼働環境でのデプロイメントでは `-f` 引数の使用はお勧めしません。
    :::

  </TabItem>
</Tabs>

## 永続インスタンスの作成

追加設定なしで「dagster dev」を実行すると、一時ディレクトリに一時的なインスタンスが起動します。ログ出力には次のような内容が表示される場合があります:

```shell
Using temporary directory /Users/rhendricks/.tmp_dagster_home_qs_fk8_5 for storage.
```

これは、セッション中に作成された実行やマテリアライズドアセットが、セッション終了後は保存されないことを意味します。

実行やアセットのより永続的な保存場所を指定するには、`DAGSTER_HOME` 環境変数をファイルシステム上のフォルダに設定してください。これにより、Dagster は以降の `dagster dev` 実行時に、指定されたフォルダをストレージとして使用します。

```shell
mkdir -p ~/.dagster_home
export DAGSTER_HOME=~/.dagster_home
dagster dev
```

## インスタンスの設定

Dagster インスタンスを設定するには、`$DAGSTER_HOME` フォルダに `dagster.yaml` ファイルを作成します。

たとえば、ローカルインスタンスで同時実行数を制限するには、次の `dagster.yaml` を設定します:

<CodeExample
  path="docs_snippets/docs_snippets/guides/tbd/dagster.yaml"
  language="yaml"
  title="~/.dagster_home/dagster.yaml"
/>

`dagster.yaml` ファイルで設定できるオプションの完全なリストについては、[Dagster インスタンスのドキュメント](/guides/deploy/dagster-instance-configuration)を参照してください。

## `dagster dev` で実行中かどうかを検出する

ローカルで実行されているかどうかを検出したい場合があります。例えば、スケジュールやセンサーを本番環境では `RUNNING` 状態で起動したいが、ローカルのテスト環境では起動させたくない、といった場合です。

`dagster dev` は、環境変数 `DAGSTER_IS_DEV_CLI` を `1` に設定します。この環境変数の存在を確認することで、ローカル開発環境にいるかどうかを検出できます。

## プロダクションへの移行

`dagster dev` は、主にローカル開発およびテスト環境で Dagster を実行する際に役立ちます。
ほとんどの本番環境へのデプロイには適していません。
最も重要な点として、`dagster dev` には認証やウェブセキュリティ機能が含まれていないことが挙げられます。さらに、本番環境では、複数のウェブサーバーのレプリカを実行したり、ダウンタイムなしでコードを継続的にデプロイしたり、Dagster デーモンがクラッシュした場合に自動的に再起動するように設定したりする必要があるかもしれません。

本番環境での Dagster のデプロイ方法については、[Dagster オープンソース デプロイオプションのドキュメント](/guides/deploy/deployment-options/) をご覧ください。
