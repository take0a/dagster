---
layout: Integration
status: published
name: dlt
title: Dagster & dlt
sidebar_label: dlt
excerpt: Easily ingest and replicate data between systems with dlt through Dagster.
date: 2024-08-30
apireflink: https://docs.dagster.io/api/python-api/libraries/dagster-dlt
docslink: https://docs.dagster.io/integrations/libraries/dlt/
partnerlink: https://dlthub.com/
logo: /integrations/dlthub.jpeg
categories:
  - ETL
enabledBy:
enables:
tags: [dagster-supported, etl]
sidebar_custom_props:
  logo: images/integrations/dlthub.jpeg
---

この統合により、[dlt](https://dlthub.com/) を使用して、Dagster を介してシステム間でデータを簡単に取り込み、複製できるようになります。

### Installation

```bash
pip install dagster-dlt
```

### 例

<CodeExample path="docs_snippets/docs_snippets/integrations/dlt.py" language="python" />

:::note

[sql_database](https://dlthub.com/docs/api_reference/sources/sql_database/__init__#sql_database) ソースを使用している場合は、データベースの読み取りを減らすために `defer_table_reflect=True` を設定することを検討してください。デフォルトでは、Dagster デーモンはおよそ 1 分ごとに定義を更新し、データベースに対してリソース定義を照会します。

:::

### dlt について

[データ ロード ツール (dlt)](https://dlthub.com/) は、効率的なデータ パイプラインを作成するためのオープン ソース ライブラリです。シークレット管理、データ構造の変換、増分更新、事前構築されたソースと宛先などの機能を提供し、乱雑なデータを適切に構造化されたデータセットにロードするプロセスを簡素化します。
