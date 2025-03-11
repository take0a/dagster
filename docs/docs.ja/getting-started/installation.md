---
title: Dagsterのインストール
description: Learn how to install Dagster
sidebar_position: 20
sidebar_label: Installation
---

このガイドの手順を実行するには、次のことが必要です。

- Python 3.9 以降をインストールします。**Python 3.12 が推奨されます**。
- Pythonパッケージインストーラーpipをインストールする

## 仮想環境の設定

Python をインストールした後は、仮想環境を設定することをお勧めします。これにより、Dagster プロジェクトがシステムの他の部分から分離され、依存関係の管理が容易になります。

これを行う方法はたくさんありますが、このガイドでは追加の依存関係を必要としないため、`venv` を使用します。

<Tabs>
<TabItem value="macos" label="MacOS">
```bash
python -m venv venv
source venv/bin/activate
```
</TabItem>
<TabItem value="windows" label="Windows">
```bash
python -m venv venv
source venv\Scripts\activate
```
</TabItem>
</Tabs>

:::tip
**`venv` よりも強力なものをお探しですか?** `pyenv` または `pyenv-virtualenv` をお試しください。これらは、1 台のマシンで複数のバージョンの Python を管理するのに役立ちます。詳細については、[pyenv GitHub リポジトリ](https://github.com/pyenv/pyenv) を参照してください。
:::

## Dagsterのインストール

仮想環境に Dagster をインストールするには、ターミナルを開いて次のコマンドを実行します。

```bash
pip install dagster dagster-webserver
```

このコマンドは、Dagster UI を提供するために使用されるコア Dagster ライブラリと Web サーバーをインストールします。

## インストールの確認

Dagster が正しくインストールされていることを確認するには、次のコマンドを実行します:

```bash
dagster --version
```

Dagster のバージョン番号がターミナルに表示されるはずです:

```bash
> dagster --version
dagster, version 1.8.4
```

## トラブルシューティング

インストールプロセス中に問題が発生した場合:

- トラブルシューティングについては、[Dagster GitHubリポジトリ](https://github.com/dagster-io/dagster)を参照してください。もしくは
- [Dagster コミュニティ](/about/community) に連絡してください

## 次のステップ

- [クイックスタート](/getting-started/quickstart)で最初のDagsterプロジェクトを立ち上げて実行しましょう
- [Dagster でデータアセットを作成する](/guides/build/assets/defining-assets) 方法を学ぶ
