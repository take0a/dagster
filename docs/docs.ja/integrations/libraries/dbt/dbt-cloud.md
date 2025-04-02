---
layout: Integration
status: published
name: dbt Cloud
title: Dagster & dbt Cloud
sidebar_label: dbt Cloud
excerpt: Run dbt Cloud™ jobs as part of your data pipeline.
date: 2022-11-07
apireflink: https://docs.dagster.io/api/python-api/libraries/dagster-dbt#assets-dbt-cloud
docslink: https://docs.dagster.io/integration/libraries/dbt/dbt_cloud
partnerlink:
categories:
  - ETL
enabledBy:
enables:
tags: [dagster-supported, etl]
sidebar_custom_props:
  logo: images/integrations/dbt/dbt.svg
---

import Beta from '@site/docs.ja/partials/\_Beta.md';

<Beta />

Dagster を使用すると、dbt Cloud ジョブを他のテクノロジーと並行して実行できます。ジョブを大規模なパイプラインのステップとして実行するようにスケジュールし、データ アセットとして管理できます。

### インストール

```bash
pip install dagster-dbt
```

### 例

<CodeExample path="docs_snippets/docs_snippets/integrations/dbt_cloud.py" language="python" />

### dbt Cloud について

**dbt Cloud** は、dbt ジョブを実行するためのホスト型サービスです。データ アナリストやエンジニアが dbt デプロイメントを本番環境に移行するのに役立ちます。dbt オープンソース以外にも、dbt Cloud はスケジュール、CI/CD、ドキュメントの提供、監視とアラート機能を提供します。

現在 dbt Cloud™ を使用している場合は、代わりに Dagster を使用して `dbt-core` を実行することもできます。[その方法](https://dagster.io/blog/migrate-off-dbt-cloud) の詳細については、こちらをご覧ください。
