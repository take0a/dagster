---
title: アセットファクトリーの作成
sidebar_position: 700
---

データ エンジニアリングでは、多くの場合、類似したアセットを大量に作成する必要があります。例:

- データベーステーブルのセットはすべて同じスキーマを持つ
- ディレクトリ内のファイルのセットはすべて同じ形式である

Python や Dagster に慣れていない利害関係者にサービスを提供している可能性もあります。彼らは、YAML などの構成言語上に構築されたドメイン固有言語 (DSL) を使用してアセットを操作することを好むかもしれません。

アセットファクトリパターンは、これらの問題の両方を解決できます。

:::note

この記事は、以下の知識があることを前提としています:

  - [Assets](/guides/build/assets/defining-assets)
  - [Resources](/guides/build/external-resources/)
  - SQL, YAML と Amazon Web Services (AWS) S3
  - [Pydantic](https://docs.pydantic.dev/latest/) と [Jinja2](https://jinja.palletsprojects.com/en/3.1.x/)

:::

<details>
  <summary>前提条件</summary>

この記事のコードを実行するには、Python 仮想環境を作成してアクティブ化し、次の依存関係をインストールする必要があります:

   ```bash
   pip install dagster dagster-aws duckdb pyyaml pydantic
   ```
</details>

## Python でアセットファクトリーを構築する

同じ繰り返しの ETL タスクを頻繁に実行する必要があるチームを想像してみましょう。S3 から CSV ファイルをダウンロードし、それに対して基本的な SQL クエリを実行し、その結果を新しいファイルとして S3 にアップロードします。

このプロセスを自動化するには、次のように Python でアセットファクトリを定義します。

<CodeExample path="docs_beta_snippets/docs_beta_snippets/guides/data-modeling/asset-factories/python-asset-factory.py" language="python" />

アセットファクトリパターンは、基本的に、何らかの構成を受け取り、`dg.Definitions` を返す関数です。

## YAML を使用したアセットファクトリーの設定

現在、チームは次のようなファイルを使用して、Python ではなく YAML を使用してアセット ファクトリーを構成できるようにしたいと考えています。

<CodeExample path="docs_beta_snippets/docs_beta_snippets/guides/data-modeling/asset-factories/etl_jobs.yaml" language="yaml" title="etl_jobs.yaml" />

これを実装するには、YAML ファイルを解析し、それを使用して S3 リソースと ETL ジョブを作成します:

<CodeExample path="docs_beta_snippets/docs_beta_snippets/guides/data-modeling/asset-factories/simple-yaml-asset-factory.py" language="python" />

## PydanticとJinjaでユーザビリティを向上

現在のアプローチにはいくつか問題があります:

1. **YAMLファイルは型チェックされていない**ため、不可解な`KeyError`を引き起こす間違いを犯しやすい。
2. **YAML ファイルにはシークレットが含まれています**。代わりに、環境変数を参照する必要があります。

これらの問題を解決するには、Pydantic を使用して YAML ファイルのスキーマを定義し、Jinja を使用して環境変数を使用して YAML ファイルをテンプレート化します。

新しい YAML ファイルは次のようになります。環境変数を参照するために Jinja テンプレートがどのように使用されているかに注意してください。

<CodeExample path="docs_beta_snippets/docs_beta_snippets/guides/data-modeling/asset-factories/etl_jobs_with_jinja.yaml" language="yaml" title="etl_jobs.yaml" />

Python 実装は次のとおりです:

<CodeExample path="docs_beta_snippets/docs_beta_snippets/guides/data-modeling/asset-factories/advanced-yaml-asset-factory.py" language="python" />

## 次は

TODO
