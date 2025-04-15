---
title: 'Dagster をサービスとしてデプロイする'
sidebar_label: Dagster as a service
description: 'Learn how to deploy Dagster as a service on a single machine'
sidebar_position: 20
---

<details>
  <summary>前提条件</summary>

このガイドの手順を実行するには、以下のものが必要です。

- Dagster プロジェクトを作成済み
- コンテナ化とサービス管理に関する実用的な知識

</details>

このガイドでは、Dagster を単一のマシンにサービスとして導入する手順を説明します。
Dagster ウェブサーバーとデーモンの設定手順も含まれています。
このアプローチは、小規模な導入やテスト目的に適しています。
本番環境では、コンテナ化された導入またはクラウドベースのソリューションの使用を検討してください。

## Dagster Webサーバーの実行

Dagsterウェブサーバーは、あらゆるDagsterデプロイメントの中核コンポーネントです。Dagster UIを提供し、GraphQLクエリに応答します。

### インストール

まず、Dagster ウェブサーバーをインストールします:

```bash
pip install dagster-webserver
```

### Dagster Webサーバーの起動

Web サーバーを起動する前に、`DAGSTER_HOME` 環境変数を設定します。この変数は、Dagster に永続データとログを保存する場所を指示します。

<Tabs>
  <TabItem value="linux-mac" label="Linux/macOS (Bash)" default>

```bash
export DAGSTER_HOME="/home/yourusername/dagster_home"
```

  </TabItem>
  <TabItem value="windows" label="Windows (PowerShell)" default>

```powershell
$env:DAGSTER_HOME = "C:\Users\YourUsername\dagster_home"
```

  </TabItem>

</Tabs>

次に、Web サーバーを実行するには、次のコマンドを使用します:

```bash
dagster-webserver -h 0.0.0.0 -p 3000
```

この設定により、以下の処理が実行されます:

- `DAGSTER_HOME` 環境変数を設定します。これにより、Dagster は永続データとログを保存する場所を知ります。
- 実行ログを `$DAGSTER_HOME/logs` に書き込みます。
- `0.0.0.0:3000` を listen します。

## Dagster デーモンの実行

スケジュール、センサー、バックフィルを使用している場合、または同時に実行できる実行数に制限を設定する場合は、Dagster デーモンが必要です。

### インストール

Dagsterデーモンをインストールします:

```bash
pip install dagster
```

### デーモンの起動

`DAGSTER_HOME` 環境変数が設定されていることを確認してください。手順については、[Dagster Web サーバーの実行](#running-the-dagster-webserver) を参照してください。
次に、次のコマンドで Dagster デーモンを実行します:

```bash
dagster-daemon run
```

`dagster-daemon` プロセスは、インスタンスを定期的にチェックし、以下の情報を確認します:

- 実行キューから開始される新しい実行
- 実行スケジュールまたはセンサーによってトリガーされる実行

:::tip
`dagster-daemon` プロセスが以下のファイルにアクセスできることを確認してください。

- `dagster.yaml` ファイル
- `workspace.yaml` ファイル
- インスタンスで定義されているコンポーネント
- ワークスペースで定義されているリポジトリ
:::

### デーモンのモニタリング

Dagster UI で `dagster-daemon` プロセスのステータスを確認できます。

1. 左側のナビゲーションバーから「インスタンス」タブに移動します。
2. デーモンのステータスを表示します。

:::important
デプロイメントには `dagster-webserver` のインスタンスを複数含めることができますが、`dagster-daemon` プロセスは 1 つだけ含める必要があります。
:::

## 次のステップ

Dagster をサービスとして実行できるようになりました。以下の手順をご確認ください。

- [Dagster インスタンスの設定](/guides/deploy/dagster-instance-configuration)
- [スケジュールとセンサーの設定](/guides/automate/schedules)