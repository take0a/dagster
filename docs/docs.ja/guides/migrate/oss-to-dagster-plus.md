---
title: 'Dagster OSSからDagster+へ'
sidebar_position: 30
---

## ステップ1：Dagster+を使い始める

まず、Dagster+ 組織を作成し、デプロイメントタイプ（ハイブリッドまたはサーバーレス）を選択し、ユーザーと認証を設定する必要があります。
開始するには、[Dagster+ ドキュメント](/dagster-plus/getting-started)をご覧ください。

## ステップ2: CI/CDパイプラインを更新する

次に、OSS コードをデプロイする CI/CD プロセスを、Dagster+ のデプロイメントパターンに従って変更する必要があります。
詳細については、[Dagster+ CI/CD ドキュメント](/dagster-plus/features/ci-cd/configuring-ci-cd) をご覧ください。

## ステップ3：Dagster+にメタデータを入力する

この時点で、OSSとDagster+には同じデータパイプラインが存在するはずですが、Dagster+のメタデータは空のままです。
その後、Dagster+に切り替えてメタデータの入力を開始するか、OSSから過去のメタデータを移行することができます。

### オプション1: カットオーバー後にメタデータを入力する

OSSデプロイメントからDagster+に履歴メタデータを移行する必要がない場合は、Dagster OSSデプロイメントをオフにして、Dagster+のスケジュール、センサー、その他のメタデータ追跡機能を有効にすることができます。
その時点から、アセットが具体化されると、メタデータがDagster+に表示されるようになります。

### オプション2: 履歴メタデータを移行する

OSS デプロイメントから Dagster+ に履歴メタデータを移行するには、[OSS メタデータから Plus の例](https://github.com/dagster-io/dagster/tree/master/examples/oss-metadata-to-plus) の手順に従います。
