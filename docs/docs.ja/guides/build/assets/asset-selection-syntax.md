---
title: 'アセット選択構文'
sidebar_position: 1000
---

# アセット選択構文

このリファレンスには、アセットの選択の構文に関する情報が含まれており、アセットとそのダウンストリームおよびアップストリームの依存関係を選択するためのさまざまな例が含まれています。

アセット選択は次の目的で使用できます:

- 選択したアセットを対象とするジョブを定義する
- Dagster UIで表示するアセットのセットを選択します
- アドホック実行のアセットセットを選択する

## 構文の使用法

クエリには句のリストが含まれます。句は、次のメソッドの `selection` パラメータの場合を除き、コンマで区切られます。これらの場合、各句はリスト内の個別の要素になります。

- `define_asset_job`
- `materialize`
- `materialize_to_memory`

| Clause syntax         | Description               |
| --------------------- | ----------------------------- |
| `ASSET_KEY`           | アセット キーで単一のアセットを選択します。[例を参照](#single-asset)。 |
| `COMPONENT/COMPONENT` | プレフィックスなどの複数のコンポーネントを持つアセット キーを選択します。コンポーネント間にはスラッシュ (`/`) が挿入されます。[例を参照](#multiple-key-components)。  |
| `*ASSET_KEY`          | アセットとそのすべての上流依存関係を選択します。[例を参照](#all-upstream)。  |
| `ASSET_KEY*`          | アセットとその下流の依存関係すべてを選択します。[例を参照](#all-downstream)。 |
| `+ASSET_KEY`          | アセットとそのアセットの上流レイヤーを 1 つ選択します。複数の `+` を含めると、その数の上流レイヤーがアセットから選択されます。任意の数の `+` がサポートされています。[例を参照](#specific-upstream)。 |
| `ASSET_KEY+`          | アセットとそのアセットの下流の 1 つのレイヤーを選択します。複数の `+` を含めると、その数の下流レイヤーがアセットから選択されます。任意の数の `+` がサポートされています。[例を参照](#specific-downstream)。 |

## 例

このセクションの例では、[Dagster University Essentials プロジェクト](https://github.com/dagster-io/project-dagster-university) の次のアセットグラフを使用して、選択構文の使用方法を示します:

![Screenshot of Daggy U project graph](/images/guides/build/assets/asset-selection-syntax/asset-selection-syntax-dag.png)

### 単一のアセットを選択する \{#single-asset}

単一のアセットを選択するには、アセットのアセット キーを使用します。この例では、`taxi_zones_file` アセットを選択します:

<Tabs>
<TabItem value="python" label="Python">

```python
raw_data_job = define_asset_job(name="raw_data_job", selection="taxi_zones_file")
```

</TabItem>
<TabItem value="cli" label="CLI">

```shell
dagster asset list --select taxi_zones_file
dagster asset materialize --select taxi_zones_file
```

</TabItem>
<TabItem value="dagster-ui" label="Dagster UI">

```shell
taxi_zones_file
```

結果として、次のアセットグラフが作成されます:

![Screenshot of Daggy U project graph](/images/guides/build/assets/asset-selection-syntax/select-single-asset.png)

</TabItem>
</Tabs>

---

### 複数のキーコンポーネントを持つアセットの選択 \{#multiple-key-components}

プレフィックスなどの複数のコンポーネントを含むキーを持つアセットを選択するには、コンポーネント間にスラッシュ (`/`) を挿入します。

この例では、以下のように定義されている `manhattan/manhattan_stats` アセットを選択します:

```python
@asset(
    deps=[AssetKey(["taxi_trips"]), AssetKey(["taxi_zones"])], key_prefix="manhattan"
)
def manhattan_stats(database: DuckDBResource):
 ...
```

<Tabs>
<TabItem value="python" label="Python">

```python
manhattan_job = define_asset_job(name="manhattan_job", selection="manhattan/manhattan_stats")
```

</TabItem>
<TabItem value="cli" label="CLI">

```shell
dagster asset list --select manhattan/manhattan_stats
dagster asset materialize --select manhattan/manhattan_stats
```

</TabItem>
<TabItem value="dagster-ui" label="Dagster UI">

```shell
manhattan/manhattan_stats
```

結果として、次のアセットグラフが作成されます:

![Screenshot of Daggy U project graph](/images/guides/build/assets/asset-selection-syntax/select-multiple-components.png)

</TabItem>
</Tabs>

---

### 複数のアセットを選択する \{#multiple-assets}

複数のアセットを選択するには、アセットのアセット キーのリストを使用します。アセットは相互に依存している必要はありません。

この例では、以下のように定義されている `taxi_zones_file` アセットと `taxi_trips_file` アセットを選択します:

<Tabs>
<TabItem value="python" label="Python">

```python
raw_data_job = define_asset_job(
    name="taxi_zones_job", selection=["taxi_zones_file", "taxi_trips_file"]
)
```

</TabItem>
<TabItem value="cli" label="CLI">

複数のアセットを選択する場合は、アセット キーのリストを二重引用符 (`"`) で囲み、各アセット キーをコンマで区切ります:

```shell
dagster asset list --select "taxi_zones_file,taxi_trips_file"
dagster asset materialize --select "taxi_zones_file,taxi_trips_file"
```

</TabItem>
<TabItem value="dagster-ui" label="Dagster UI">

```shell
taxi_zones_file taxi_trips_file
```

結果として、次のアセットグラフが作成されます:

![Screenshot of Daggy U project graph](/images/guides/build/assets/asset-selection-syntax/select-disjointed-lineages.png)

</TabItem>
</Tabs>

---

### アセットの系統全体を選択する \{#full-lineage}

資産の系統全体を選択するには、クエリ内の資産キーの前後にアスタリスク (`*`) を追加します。

この例では、`taxi_zones` アセットの系統全体を選択します。

<Tabs>
<TabItem value="python" label="Python">

```python
taxi_zones_job = define_asset_job(name="taxi_zones_job", selection="*taxi_zones*")
```

</TabItem>
<TabItem value="cli" label="CLI">

CLI を使用してアセットの系統全体を選択する場合は、アスタリスク (`*`) とアセット キーを二重引用符 (`"`) で囲みます:

```shell
dagster asset list --select "*taxi_zones*"
dagster asset materialize --select "*taxi_zones*"
```

</TabItem>
<TabItem value="dagster-ui" label="Dagster UI">

```shell
*taxi_zones*
```

結果として、次のアセットグラフが作成されます。

![Screenshot of Daggy U project graph](/images/guides/build/assets/asset-selection-syntax/select-entire-lineage.png)

</TabItem>
</Tabs>

---

### 上流依存関係の選択

#### すべての上流依存関係を選択 \{#all-upstream}

アセットとそのすべての上流依存関係を選択するには、クエリ内のアセット キーの前にアスタリスク (`*`) を追加します。

この例では、`manhattan_map` アセットとそのすべての上流依存関係を選択します。

<Tabs>
<TabItem value="python" label="Python">

```python
manhattan_job = define_asset_job(name="manhattan_job", selection="*manhattan_map")
```

</TabItem>
<TabItem value="cli" label="CLI">

CLI を使用してアセットの依存関係を選択する場合は、アスタリスク (`*`) とアセット キーを二重引用符 (`"`) で囲みます:

```shell
dagster asset list --select "*manhattan_map"
dagster asset materialize --select "*manhattan_map"
```

</TabItem>
<TabItem value="dagster-ui" label="Dagster UI">

```shell
*manhattan_map
```

結果として、次の資産グラフが作成されます:

![Screenshot of Daggy U project graph](/images/guides/build/assets/asset-selection-syntax/select-upstream-dependencies.png)

</TabItem>
</Tabs>

#### 特定の数の上流レイヤーを選択する \{#specific-upstream}

アセットと複数の上流レイヤーを選択するには、クエリ内のアセット キーの前に、選択するレイヤーごとにプラス記号 (`+`) を追加します。

この例では、`manhattan_map` アセットと 2 つの上流レイヤーを選択します。

<Tabs>
<TabItem value="python" label="Python">

```python
manhattan_job = define_asset_job(name="manhattan_job", selection="++manhattan_map")
```

</TabItem>
<TabItem value="cli" label="CLI">

CLI を使用してアセットの依存関係を選択する場合は、プラス記号 (`+`) とアセット キーを二重引用符 (`"`) で囲みます:

```shell
dagster asset list --select "++manhattan_map"
dagster asset materialize --select "++manhattan_map"
```

</TabItem>
<TabItem value="dagster-ui" label="Dagster UI">

```shell
++manhattan_map
```

結果として、次のアセットグラフが作成されます。

![Screenshot of Daggy U project graph](/images/guides/build/assets/asset-selection-syntax/select-two-upstream-layers.png)

</TabItem>
</Tabs>

---

### 下流の依存関係の選択

#### すべての下流依存関係を選択 \{#all-downstream}

アセットとその下流の依存関係をすべて選択するには、クエリ内のアセット キーの後にアスタリスク (`*`) を追加します。

この例では、`taxi_zones_file` アセットとそのすべての下流依存関係を選択します。

<Tabs>
<TabItem value="python" label="Python">

```python
taxi_zones_job = define_asset_job(name="taxi_zones_job", selection="taxi_zones_file*")
```

</TabItem>
<TabItem value="cli" label="CLI">

CLI を使用してアセットの依存関係を選択する場合は、アスタリスク (`*`) とアセット キーを二重引用符 (`"`) で囲みます:

```shell
dagster asset list --select "taxi_zones_file*"
dagster asset materialize --select "taxi_zones_file*"
```

</TabItem>
<TabItem value="dagster-ui" label="Dagster UI">

```shell
taxi_zones_file*
```

結果として、次のアセットグラフが作成されます:

![Screenshot of Daggy U project graph](/images/guides/build/assets/asset-selection-syntax/select-downstream-dependencies.png)

</TabItem>
</Tabs>

#### 特定の数の下流レイヤーを選択する \{#specific-downstream}

アセットと複数の下流レイヤーを選択するには、クエリ内のアセット キーの後に、選択するレイヤーごとにプラス記号 (`+`) を追加します。

この例では、`taxi_trips_file` アセットと 2 つの下流レイヤーを選択します。

<Tabs>
<TabItem value="python" label="Python">

```python
taxi_zones_job = define_asset_job(name="taxi_zones_job", selection="taxi_zones_file++")
```

</TabItem>
<TabItem value="cli" label="CLI">

CLI を使用してアセットの依存関係を選択する場合は、プラス記号 (`+`) とアセット キーを二重引用符 (`"`) で囲みます:

```shell
dagster asset list --select "taxi_zones_file++"
dagster asset materialize --select "taxi_zones_file++"
```

</TabItem>
<TabItem value="dagster-ui" label="Dagster UI">

```shell
taxi_zones_file++
```

結果として、次のアセットグラフが作成されます:

![Screenshot of Daggy U project graph](/images/guides/build/assets/asset-selection-syntax/select-two-downstream-layers.png)

</TabItem>
</Tabs>
