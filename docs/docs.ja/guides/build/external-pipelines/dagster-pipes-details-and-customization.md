---
title: "Dagster パイプの詳細とカスタマイズ"
description: "Learn about Dagster Pipes APIs and how to compose them to create a custom solution for your data platform."
sidebar_position: 90
---

[Dagster Pipes](/guides/build/external-pipelines) は、Dagster を任意の外部コンピューティング環境と統合するためのツールキットです。多くのユーザーは、Pipes クライアント オブジェクト (例: <PyObject section="pipes" module="dagster" object="PipesSubprocessClient" />、<PyObject section="libraries" object="PipesDatabricksClient" module="dagster_databricks"/>) が提供する簡素化されたインターフェイスで十分ですが、Pipes に対するより高度な制御を必要とするユーザーもいます。これは、既存の大規模なコードベースを Dagster に接続しようとしているユーザーに特に当てはまります。

このガイドでは、下位レベルの Pipes API と、それらを組み合わせてデータ プラットフォームにカスタム ソリューションを提供する方法について説明します。

## 概要と用語

![Detailed overview of a Dagster Pipes session](/images/guides/build/external-pipelines/pipes-overview.png)

| Term | Definition |
|------|------------|
| **External environment** | Dagster 外部の環境 (例: Databricks、Kubernetes、Docker) |
| **Orchestration process** | アセットを実体化するために Dagster コードを実行するプロセス。 |
| **External process** | 外部環境で実行されるプロセス。このプロセスから、ログ出力と Dagster イベントをオーケストレーション プロセスに報告できます。オーケストレーション プロセスは外部プロセスを起動する必要があります。 |
| **Bootstrap payload** | オーケストレーション プロセスによって外部プロセス内のグローバルにアクセス可能なキー値ストアに書き込まれる、キーと値のペアの小さなバンドル。通常、ブートストラップ ペイロードは環境変数に書き込まれますが、環境変数の設定をサポートしていない外部環境では別のメカニズムが使用される場合があります。 |
| **Context payload** | オーケストレーション プロセスの実行コンテキスト (<PyObject section="execution" module="dagster" object="AssetExecutionContext" />) から取得した情報を含む JSON オブジェクト。これには、スコープ内のアセット キー、パーティション キーなどが含まれます。コンテキスト ペイロードは、オーケストレーション プロセスによって、外部プロセスがアクセスできる場所に書き込まれます。外部プロセスは、ブートストラップ ペイロードからコンテキスト ペイロードの場所 (Amazon S3 上のオブジェクト URL など) を取得し、コンテキスト ペイロードを読み取ります。 |
| **Messages** | オーケストレーション プロセスで使用するために外部プロセスによって書き込まれる JSON オブジェクト。メッセージは、アセットの具体化を報告し、結果をチェックするだけでなく、オーケストレーション側のログ記録をトリガーすることもできます。|
| **Logs** | 外部プロセスによって生成されたログ ファイル (ログに記録された stdout/stderr ストリームを含むが、これに限定されません)。 |
| **Params loader** | グローバルにアクセス可能なキー値ストアからブートストラップ ペイロードを読み取る外部プロセス内のエンティティ。デフォルトのパラメータ ローダーは、環境変数からブートストラップ ペイロードを読み取ります。 |
| **Context injector** | コンテキスト ペイロードを外部からアクセス可能な場所に書き込み、ブートストラップ ペイロードに含めるためにこの場所をエンコードする一連のパラメーターを生成する、オーケストレーション プロセス内のエンティティ。 |
| **Context loader** | ブートストラップ ペイロードで指定された場所からコンテキスト ペイロードをロードする外部プロセス内のエンティティ。 |
| **Message reader** | 外部からアクセス可能な場所からメッセージ (およびオプションでログ ファイル) を読み取り、ブートストラップ ペイロードでこの場所をエンコードする一連のパラメーターを生成する、オーケストレーション プロセス内のエンティティ。 |
| **Message writer** | ブートストラップ ペイロードで指定された場所にメッセージを書き込む外部プロセス内のエンティティ。 |

## セッションのパイプ

**セッションのパイプ** は次の期間にわたります:

1. オーケストレーションと外部プロセス間の通信チャネルを作成する。
2. 外部プロセスを起動し終了する。
3. 外部プロセスによって報告されたすべてのメッセージを読み取り、通信チャネルを閉じる

オーケストレーション プロセスと外部プロセスでは、セッションのパイプで対話するための API が別々に用意されています。オーケストレーション プロセス API は `dagster` で定義されています。外部プロセス API は、外部プロセスでユーザー コードによってロードされる Pipes 統合ライブラリによって定義されます。このライブラリは、ブートストラップ ペイロードを解釈し、コンテキスト ローダーとメッセージ ライターを起動する方法を知っています。

現在、唯一の公式 Dagster Pipes 統合ライブラリは Python の [`dagster-pipes`](/api/python-api/libraries/dagster-pipes) であり、[PyPI](https://pypi.org/project/dagster-pipes/) で入手できます。このライブラリには依存関係がなく、[単一のファイル](https://github.com/dagster-io/dagster/blob/master/python_modules/dagster-pipes/dagster_pipes/\__init\_\_.py) に収まるため、簡単にベンダー化することもできます。

### セッションライフサイクル（オーケストレーションプロセス）

パイプ セッションは、オーケストレーション プロセスで <PyObject section="pipes" module="dagster" object="PipesSession" /> クラスによって表されます。セッションは、`PipesSession` を生成する <PyObject section="pipes" module="dagster" object="open_pipes_session" /> コンテキスト マネージャーで開始されます。`open_pipes_session` は、<PyObject section="execution" module="dagster" object="AssetExecutionContext" /> が使用可能なアセット内で呼び出す必要があります。

<CodeExample path="docs_snippets/docs_snippets/guides/dagster/dagster_pipes/dagster_pipes_details_and_customization/session_lifecycle_orchestration.py" />

上記では、<PyObject section="pipes" module="dagster" object="open_pipes_session" /> が 4 つのパラメータを取ることがわかります。

- `context`: コンテキストペイロードを派生するために使用される実行コンテキスト (<PyObject section="execution" module="dagster" object="AssetExecutionContext" />)。
- `extras`: JSON シリアル化可能な辞書形式のキーと値のペアのバンドル。これはコンテキスト ペイロードに挿入されます。ユーザーは、外部プロセスに公開する任意のデータをここに渡すことができます。
- `context_injector`: コンテキスト インジェクターは、シリアル化されたコンテキスト ペイロードをある場所に書き込み、その場所を外部プロセスが使用するためのブートストラップ パラメーターとして表現する役割を担います。上記では、組み込み (およびデフォルト) の <PyObject section="pipes" module="dagster" object="PipesTempFileContextInjector" /> を使用しました。これは、シリアル化されたコンテキスト ペイロードを自動的に作成されたローカル一時ファイルに書き込み、そのファイルへのパスをブートストラップ パラメーターとして公開します。
- `message_reader`: ある場所に書き込まれたストリーミング メッセージとログ ファイルを読み取り、その場所を外部プロセスが消費するためのブートストラップ パラメータとして表現するメッセージ リーダーです。上記では、組み込み (およびデフォルト) の <PyObject section="pipes" module="dagster" object="PipesTempFileMessageReader" /> を使用しました。これは、自動的に作成されたローカル一時ファイルを追跡し、そのファイルへのパスをブートストラップ パラメータとして公開します。

Python コンテキスト マネージャーの呼び出しには 3 つの部分があります:

1. オープニングルーチン (`__enter__`、`with` ブロックの開始時に実行されます)。
2. 本体 (`with` ブロック内にネストされたユーザー コード)。
3. 終了ルーチン (`__exit__`、`with` ブロックの最後に実行されます)。

<PyObject section="pipes" module="dagster" object="open_pipes_session" /> の場合、これら 3 つの部分は次のタスクを実行します:

- **開始ルーチン**: コンテキスト ペイロードを書き込み、メッセージ リーダーを起動します (通常は、スレッドを開始してメッセージを継続的に読み取ります)。これらの手順には、コンテキスト ペイロード用の一時ファイル (ローカルまたはリモート システム上) や、メッセージが書き込まれる一時ディレクトリなどのリソースの作成が含まれる場合があります。
- **本文**: ユーザー コードは、ここで外部プロセスの起動、ポーリング、終了を処理する必要があります。外部プロセスの実行中に、受信した中間結果は `yield from pipes_session.get_results()` を使用して Dagster に報告できます。
- **終了ルーチン**: 外部プロセスによって書き込まれたすべてのメッセージがオーケストレーション プロセスに読み込まれたことを確認し、コンテキスト インジェクターとメッセージ リーダーによって使用されたリソースをクリーンアップします。

### セッションライフサイクル（外部プロセス）

上で述べたように、現在存在する唯一の Pipes 統合ライブラリは Python の [`dagster-pipes`](/api/python-api/libraries/dagster-pipes) です。したがって、以下の例では Python と `dagster-pipes` を使用しています。将来的には、他の言語で `dagster-pipes` に相当するものをリリースする予定です。ここで説明した概念は、これらの他の統合ライブラリに直接マッピングされるはずです。

Pipes セッションは、外部プロセスでは <PyObject section="libraries" object="PipesContext" module="dagster_pipes" /> オブジェクトによって表されます。起動オーケストレーション プロセスによって作成されたセッションは、`dagster-pipes` の <PyObject section="libraries" object="open_dagster_pipes" module="dagster_pipes" /> を使用して接続できます:

<CodeExample path="docs_snippets/docs_snippets/guides/dagster/dagster_pipes/dagster_pipes_details_and_customization/session_lifecycle_external.py" />

:::tip

上記のメタデータ形式 (`{"raw_value": value, "type": type}`) は、豊富な Dagster メタデータを指定するための Dagster Pipes の特別な構文の一部です。サポートされているすべてのメタデータ タイプとその形式の完全なリファレンスについては、[Dagster Pipes メタデータ リファレンス](using-dagster-pipes/reference#passing-rich-metadata-to-dagster) を参照してください。

:::

上記では、<PyObject section="libraries" object="open_dagster_pipes" module="dagster_pipes"/> が 3 つのパラメータを取ることがわかります:

- `params_loader`: 起動時に外部プロセスに挿入されたブートストラップ ペイロードをロードするパラメーター ローダー。標準的なアプローチは、<PyObject section="libraries" object="PipesEnvVarParamsLoader" module="dagster_pipes" /> が読み取り方法を知っている所定の環境変数にブートストラップ ペイロードを挿入することです。ただし、環境変数を変更できない環境では、別のブートストラップ パラメーター ローダーを代用できます。
- `context_loader`: コンテキスト ローダーは、ブートストラップ ペイロードで指定された場所からコンテキスト ペイロードをロードする役割を担います。上記では、ブートストラップ パラメータでターゲットのファイル パスの `path` キーを検索する <PyObject section="libraries" object="PipesDefaultContextLoader" module="dagster_pipes" /> を使用しています。オーケストレーション側で先ほど使用した <PyObject section="pipes" module="dagster" object="PipesTempFileContextInjector" /> はこの `path` キーを書き込みますが、`PipesDefaultContextLoader` は特定のコンテキスト インジェクターに依存しません。
- `message_writer:` ブートストラップ ペイロードで指定された場所にストリーミング メッセージを書き込むメッセージ ライター。上記では、ブートストラップ パラメータでターゲットのファイル パスの `path` キーを検索する <PyObject section="libraries" object="PipesDefaultMessageWriter" module="dagster_pipes" /> を使用しています。オーケストレーション側で先ほど使用した <PyObject section="pipes" module="dagster" object="PipesTempFileMessageReader" /> がこの `path` キーを書き込みますが、`PipesDefaultMessageWriter` は特定のコンテキスト インジェクターに依存しません。

オーケストレーション側の <PyObject section="pipes" module="dagster" object="open_pipes_session" /> と同様に、 <PyObject section="libraries" object="open_dagster_pipes" module="dagster_pipes" /> はコンテキスト マネージャーです。その 3 つの部分は次の機能を実行します。

- **開始ルーチン**: 環境からブートストラップ ペイロードを読み取り、次にコンテキスト ペイロードを読み取ります。メッセージ ライターを起動します。これには、バッファリングされたメッセージを定期的に書き込むスレッドの開始が含まれる場合があります。
- **本文**: ビジネス ロジックはここに記述され、生成された <PyObject section="libraries" object="PipesContext" module="dagster_pipes" /> (上記の `pipes` 変数内) を使用してコンテキスト情報を読み取ったり、メッセージを書き込んだりすることができます。
- **終了ルーチン**: プロセスが終了する前に、ビジネス ロジックによって送信されたすべてのメッセージが書き込まれていることを確認します。一部のメッセージ ライターは書き込み間でメッセージをバッファリングするため、これが必要になります。

## カスタマイズ

ユーザーは、カスタム パラメータ ローダー、コンテキスト ローダー/インジェクター ペア、およびメッセージ リーダー/ライター ペアを実装できます。Dagster が現在互換性のあるコンテキスト ローダー/インジェクターまたはメッセージ リーダー/ライターを出荷していない環境で Dagster Pipes を使用する場合は、上記のいずれかが必要になることがあります。

### カスタムパラメータローダー

パラメータ ローダーは、<PyObject section="libraries" object="PipesParamsLoader" module="dagster_pipes" /> から継承する必要があります。以下は、`cloud_service` という架空のパッケージからインポートされた `METADATA` というオブジェクトからパラメータをロードする例です。"cloud service" は何らかのコンピューティング プラットフォームを表し、`cloud_service` パッケージが環境で使用可能であり、"cloud service" でプロセスを起動するための API によって、`cloud_service.METADATA` として公開されるペイロードに任意のキーと値のペアを設定できることが前提となります。

<CodeExample path="docs_snippets/docs_snippets/guides/dagster/dagster_pipes/dagster_pipes_details_and_customization/custom_bootstrap_loader.py" />

### カスタムコンテキストインジェクター/ローダー

コンテキスト インジェクターは <PyObject section="pipes" module="dagster" object="PipesContextInjector" displayText="dagster.PipesContextInjector" /> から継承し、コンテキスト ローダーは <PyObject section="libraries" object="PipesContextLoader" module="dagster_pipes" displayText="dagster_pipes.PipesContextLoader" /> から継承する必要があります。

一般的に、一方のカスタム バリアントを実装する場合は、もう一方の一致するバリアントを実装する必要があります。以下は、架空の `cloud_service` キー/値ストアを使用してコンテキストを書き込む簡単な例です。まず、コンテキスト インジェクター:

<CodeExample path="docs_snippets/docs_snippets/guides/dagster/dagster_pipes/dagster_pipes_details_and_customization/custom_context_injector.py" />

そして、コンテキストローダー:

<CodeExample path="docs_snippets/docs_snippets/guides/dagster/dagster_pipes/dagster_pipes_details_and_customization/custom_context_loader.py" />

### カスタムメッセージリーダー/ライター

:::note

メッセージ リーダー/ライターは、外部プロセスによって書き込まれたログ ファイルとメッセージの処理を担当します。ただし、ログ ファイル処理をカスタマイズするための API はまだ流動的であるため、このガイドでは説明しません。これらの質問が解決され次第、ログ処理をカスタマイズするための手順をガイドに更新します。

:::

メッセージ リーダーは <PyObject section="pipes" module="dagster" object="PipesMessageReader" displayText="dagster.PipesMessageReader" /> から継承する必要があり、メッセージ ライターは <PyObject section="libraries" module="dagster_pipes" object="PipesMessageWriter" displayText="dagster_pipes.PipesMessageWriter" /> から継承する必要があります。

一般的に、一方のカスタム バリアントを実装する場合は、もう一方の一致するバリアントを実装する必要があります。さらに、メッセージ ライターは内部的に <PyObject section="libraries" object="PipesMessageWriterChannel" module="dagster_pipes" /> サブコンポーネントを作成しますが、これにもカスタム バリアントを実装する必要がある可能性があります。詳細については、以下を参照してください。

以下は、架空の `cloud_service` キー/値ストアをメッセージ チャンクのストレージ レイヤーとして使用する簡単な例です。この例は、単純な抽象基本クラスではなく、<PyObject section="pipes" module="dagster" object="PipesBlobStoreMessageReader" /> および <PyObject section="libraries" object="PipesBlobStoreMessageWriter" module="dagster_pipes" /> を継承するため、コンテキスト インジェクター/ローダーの例よりも少し複雑です。BLOB ストア リーダー/ライターは、メッセージをチャンク化するためのインフラストラクチャを提供します。メッセージはライターでバッファリングされ、一定の間隔 (既定では 10 秒) でチャンクとしてアップロードされます。リーダーも同様に、一定の間隔 (既定では 10 秒) でメッセージ チャンクをダウンロードしようとします。これにより、すべてのメッセージに対してクラウド サービス BLOB ストアの読み取り/書き込みを行う必要がなくなります (コストが高くなる可能性があります)。

まず、メッセージリーダー:

<CodeExample path="docs_snippets/docs_snippets/guides/dagster/dagster_pipes/dagster_pipes_details_and_customization/custom_message_reader.py" />

そして、メッセージライター:

<CodeExample path="docs_snippets/docs_snippets/guides/dagster/dagster_pipes/dagster_pipes_details_and_customization/custom_message_writer.py" />
