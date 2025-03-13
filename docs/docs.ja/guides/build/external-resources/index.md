---
title: 外部リソース
sidebar_position: 40
---

Dagster リソースは、外部システム、データベース、またはサービスへのアクセスを提供するオブジェクトです。リソースは外部システムへの接続を管理するために使用され、Dagster アセットとオペレーションによって使用されます。たとえば、単純な ETL (抽出、変換、ロード) パイプラインは、API からデータを取得し、それをデータベースに取り込み、ダッシュボードを更新します。このパイプラインが使用する外部ツールとサービスには、次のようなものがあります:

- データが取得されるAPI
- APIのレスポンスが保存されるAWS S3バケット
- データが取り込まれるSnowflake/Databricks/BigQueryアカウント
- ダッシュボードが作成されたBIツール

Dagster リソースを使用すると、[アセット定義](/guides/build/assets)、[スケジュール](/guides/automate/schedules)、[センサー](/guides/automate/sensors)、[ジョブ](/guides/build/assets/asset-jobs) などの Dagster 定義全体でこれらのツールへの接続と統合を標準化できます。

リソースを使用すると次のことが可能になります:

- **異なる環境で異なる実装をプラグインする** - 本番環境では使用したいがテスト環境では使用したくない外部依存関係がある場合は、環境ごとに異なるリソースを提供することでこれを実現できます。
- **Dagster UI での構成の表示** - リソースとその構成が UI に表示されるため、リソースがどこで使用されているか、どのように構成されているかを簡単に確認できます。
- **複数のアセットまたはオペレーション間で構成を共有** - リソースは構成可能で共有されるため、構成を個別に提供するのではなく、1 か所で提供できます。
- **複数のアセットまたはオペレーション間で実装を共有する** - 複数のアセットが同じ外部サービスにアクセスする場合、リソースは実装を共有するためにコードを構造化する標準的な方法を提供します。

## 関連 API

{/* TODO replace `ResourceParam` with <PyObject section="resources" module="dagster" object="ResourceParam"/> in table below  */}

| 名前                                             | 説明                  |
| ------------------------------------------------ | ----------------------------- |
| <PyObject section="resources" module="dagster" object="ConfigurableResource"/>        | リソースを定義するために拡張された基本クラス。内部的には、<PyObject section="resources" object="ResourceDefinition" /> を実装します。|
| `ResourceParam`               |アセットまたはオペレーションのプレーン Python オブジェクト パラメータがリソースであることを指定するために使用されるアノテーション。|
| <PyObject section="resources" module="dagster" object="ResourceDefinition" />         | リソース定義のクラス。このクラスを直接初期化することはほとんどありません。代わりに、<PyObject section="resources" object="ResourceDefinition" /> を実装する <PyObject section="resources" object="ConfigurableResource" /> クラスを拡張する必要があります。 |
| <PyObject section="resources" module="dagster" object="InitResourceContext"/>         | 初期化中にリソースに提供されるコンテキスト オブジェクト。このオブジェクトには、必要なリソース、構成、およびその他の実行情報が含まれます。|
| <PyObject section="resources" module="dagster" object="build_init_resource_context"/> | 実行外で <PyObject section="resources" object="InitResourceContext"/> を構築するための関数。リソースをテストするときに使用することを目的としています。 |
| <PyObject section="resources" module="dagster" object="build_resources"/>             | ジョブ実行のコンテキスト外でリソース セットを初期化する関数。        |
| <PyObject section="resources" module="dagster" object="with_resources"/>              | 特定のアセット定義セットにリソースを提供し、<PyObject section="definitions" object="Definitions"/> に提供されたリソースをオーバーライドするための高度な API。 |
