---
layout: Integration
status: published
name: dbt
title: Dagster & dbt
sidebar_label: dbt
excerpt: Put your dbt transformations to work, directly from within Dagster.
date: 2022-11-07
apireflink: https://docs.dagster.io/api/python-api/libraries/dagster-dbt
docslink: https://docs.dagster.io/api/python-api/libraries/dagster-dbt#dbt-core
partnerlink: https://www.getdbt.com/
categories:
  - ETL
enabledBy:
enables:
tags: [dagster-supported, etl]
sidebar_custom_props:
  logo: images/integrations/dbt/dbt.svg
---

Dagster は dbt を他のテクノロジーと連携してオーケストレーションするため、単一のデータ パイプラインで Spark、Python などを使用して dbt をスケジュールできます。

Dagster アセットは、個々の dbt モデルのレベルで dbt を理解します。つまり、次のことが可能になります。

- Dagster の UI または API を使用して、dbt モデル、シード、スナップショットのサブセットを実行します。
- 個々の dbt モデル、シード、スナップショットの障害、ログ、実行履歴を追跡します。
- 個々の dbt モデルと他のデータ アセット間の依存関係を定義します。たとえば、dbt モデルを、その読み取り元である Fivetran が取り込んだテーブルの後に配置するか、機械学習を、そのトレーニング元である dbt モデルの後に配置します。

### インストール

```bash
pip install dagster-dbt
```

### 例

<CodeExample path="docs_snippets/docs_snippets/integrations/dbt.py" language="python" />

### dbt について

**dbt** は、モジュール性、移植性、CI/CD、ドキュメントなどのソフトウェア エンジニアリングのベスト プラクティスに従って、チームが分析コードを迅速かつ共同でデプロイできるようにする SQL ファーストの変換ワークフローです。

<aside className="rounded-lg">

dbt を使用して Dagster を実行する方法について詳しく知りたいですか? [Dagster University dbt コース](https://courses.dagster.io/courses/dagster-dbt) をご覧ください。

</aside>
