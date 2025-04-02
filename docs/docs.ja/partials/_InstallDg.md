次に、`uv` を使用して `dg` をインストールします:

<CliInvocationExample contents="uv tool install dagster-dg" />

:::tip

`uv tool install` は、PyPI から分離された環境に Python パッケージをインストールし、その実行ファイルをシェル パスに公開します。つまり、`dg` コマンドは常にプロジェクト環境とは別の分離された環境で実行されます。

:::

:::note

`dagster` リポジトリのローカル クローンがある場合は、リポジトリのルート フォルダーで `make install_editable_uv_tools` を実行して、`dg` のローカル バージョンをインストールできます。これにより、標準の `uv tool install` と同様に `dg` の分離された環境が作成されますが、その環境には `dagster-dg` の編集可能なインストールが含まれます。

:::
