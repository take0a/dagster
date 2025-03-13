---
title: "カスタムI/Oマネージャーの定義"
sidebar_position: 100
---

出力を保存および取得する場所と方法について特定の要件がある場合は、カスタム I/O マネージャーを定義できます。これは、出力を保存する関数と入力を読み込む関数の 2 つの関数を実装することになります。

I/O マネージャーを定義するには、<PyObject section="io-managers" module="dagster" object="IOManager" /> クラスを拡張します。多くの場合、<PyObject section="io-managers" module="dagster" object="ConfigurableIOManager"/> クラス (`IOManager` のサブクラス) を拡張して、I/O マネージャーに設定スキーマをアタッチする必要があります。

ここでは、ファイルシステムに CSV 値を読み書きする単純な I/O マネージャーを定義します。これは、config を通じてオプションのプレフィックス パスを受け取ります。

<CodeExample path="docs_snippets/docs_snippets/concepts/io_management/custom_io_manager.py" startAfter="start_io_manager_marker" endBefore="end_io_manager_marker" />

`handle_output` に指定された `context` 引数は <PyObject section="io-managers" module="dagster" object="OutputContext" /> です。 `load_input` に指定された `context` 引数は <PyObject section="io-managers" module="dagster" object="InputContext" /> です。 リンクされた API ドキュメントには、これらのオブジェクトで使用できるすべてのフィールドがリストされています。

### I/Oマネージャーファクトリーの使用

I/O マネージャーがより複雑であるか、内部状態を管理する必要がある場合は、I/O マネージャーの定義をその構成から分離することが合理的です。この場合、<PyObject section="io-managers" module="dagster" object="ConfigurableIOManagerFactory"/> を使用できます。これは、構成スキーマを指定し、構成を取得して I/O マネージャーを返すファクトリー関数を実装します。

この場合、キャッシュを維持するステートフル I/O マネージャーを実装します:

<CodeExample path="docs_snippets/docs_snippets/concepts/io_management/custom_io_manager.py" startAfter="start_io_manager_factory_marker" endBefore="end_io_manager_factory_marker" />

### Python の I/O マネージャーの定義

Pythonic I/O マネージャーは <PyObject section="io-managers" module="dagster" object="ConfigurableIOManager"/> のサブクラスとして定義され、[Pythonic リソース](/guides/build/external-resources/) と同様に、任意の構成フィールドを属性として指定します。各サブクラスは、実行時に Dagster によって呼び出され、データの保存と読み込みを処理する `handle_output` メソッドと `load_input` メソッドを実装する必要があります。

{/* TODO add dedent=4 prop to CodeExample below when implemented */}
<CodeExample path="docs_snippets/docs_snippets/concepts/resources/pythonic_resources.py" startAfter="start_new_io_manager" endBefore="end_new_io_manager" />

### パーティション化されたアセットの取り扱い

I/O マネージャーは、[パーティション化された](/guides/build/partitions-and-backfills/partitioning-assets) アセットを処理するように記述できます。パーティション化されたアセットの場合、`handle_output` の各呼び出しは単一のパーティションを (上書き) 書き込み、`load_input` の各呼び出しは 1 つ以上のパーティションをロードします。I/O マネージャーがファイルシステムまたはオブジェクト ストアによってサポートされている場合、各パーティションは通常、ファイルまたはオブジェクトに対応します。データベースによってサポートされている場合、各パーティションは通常、特定のウィンドウ内にあるテーブル内の行の範囲に対応します。

デフォルトの I/O マネージャーは、すぐに使用できる、一致するパーティションを持つ下流アセットのパーティション化された上流アセットのロードをサポートしています (複数のパーティションのロードについては、以下のセクションを参照してください)。<PyObject section="io-managers" module="dagster" object="UPathIOManager" /> を使用すると、カスタム ファイル システム ベースの I/O マネージャーでパーティションを処理できます。

カスタム I/O マネージャーでパーティションを処理するには、出力を保存するときや入力を読み込むときに、どのパーティションを処理するかを決定する必要があります。このために、<PyObject section="io-managers" module="dagster" object="OutputContext" /> と <PyObject section="io-managers" module="dagster" object="InputContext" /> には `asset_partition_key` プロパティがあります:

<CodeExample path="docs_snippets/docs_snippets/concepts/io_management/custom_io_manager.py" startAfter="start_partitioned_marker" endBefore="end_partitioned_marker" />

時間ウィンドウ パーティションを使用している場合は、`asset_partitions_time_window` プロパティも使用できます。このプロパティは、<PyObject section="partitions" module="dagster" object="TimeWindow" /> オブジェクトを返します。

#### パーティションマッピングの処理

1 つのアセットの単一のパーティションは、上流のアセットのパーティションの範囲に依存する場合があります。

デフォルトの I/O マネージャーは、複数の上流パーティションの読み込みをサポートしています。この場合、下流アセットは、上流の `DagsterType` に `Dict[str, ...]` (または空白のまま) タイプを使用する必要があります。デフォルトのパーティション マッピングを使用して複数の上流パーティションを読み込む例を次に示します:

<CodeExample path="docs_snippets/docs_snippets/concepts/io_management/loading_multiple_upstream_partitions.py" />

`upstream_asset` は、パーティション キーからパーティション値へのマッピングになります。これは、デフォルトの I/O マネージャー、または <PyObject section="io-managers" module="dagster" object="UPathIOManager" /> から継承する任意の I/O マネージャーのプロパティです。

<PyObject section="partitions" module="dagster" object="PartitionMapping" /> を <PyObject section="assets" module="dagster" object="AssetIn" /> に提供して、マップされた上流パーティションを構成できます。

複数のアップストリーム パーティションをロードするためのカスタム I/O マネージャーを作成する場合、マップされたキーには、<PyObject section="io-managers" module="dagster" object="InputContext" method="asset_partition_keys" />、<PyObject section="io-managers" module="dagster" object="InputContext" method="asset_partition_key_range" />、または <PyObject section="io-managers" module="dagster" object="InputContext" method="asset_partitions_time_window" /> を使用してアクセスできます。

### 入力ごとのI/Oマネージャの作成

場合によっては、対応する出力の I/O マネージャーの `load_input` 関数以外の方法で入力をロードする必要があることがあります。たとえば、チーム A に、出力を Pandas DataFrame として返すオペレーションがあり、Pandas DataFrame を保存およびロードする方法を知っている I/O マネージャーを指定しているとします。チームはこの出力を新しいオペレーションに使用することに興味がありますが、データを分析するには PySpark を使用する必要があります。残念ながら、このケースをサポートするためにチーム A の I/O マネージャーを変更する権限がありません。代わりに、チーム A の I/O マネージャーの動作の一部をオーバーライドする入力マネージャーをオペレーションに指定できます。

入力をロードする方法は、対応する出力の保存方法に直接影響されるため、入力マネージャーを既存の I/O マネージャーのサブクラスとして定義し、`load_input` メソッドを更新することを推奨します。この例では、次のように記述して、入力を Pandas DataFrame ではなく NumPy 配列としてロードします。

<CodeExample path="docs_snippets/docs_snippets/concepts/io_management/input_managers.py" startAfter="start_plain_input_manager" endBefore="end_plain_input_manager" />

`PandasIOManager` の所有者が出力を保存するパスを変更すると、すぐに問題が発生する可能性があります。パス定義ロジック (または `handle_output` と `load_input` で共有されるその他の計算) を、必要に応じて呼び出される新しいメソッドに分割することをお勧めします。

<CodeExample path="docs_snippets/docs_snippets/concepts/io_management/input_managers.py" startAfter="start_better_input_manager" endBefore="end_better_input_manager" />
