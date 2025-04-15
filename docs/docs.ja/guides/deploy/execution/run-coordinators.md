---
title: '実行コーディネーター'
sidebar_position: 300
---

本番環境の Dagster デプロイメントでは、多くの場合、一度に複数の実行が開始されます。
_実行コーディネーター_ を使用すると、デプロイメント内の実行セットを管理するために Dagster が使用するポリシーを制御できます。

Dagster UI または Dagster コマンドラインから実行を送信すると、まず実行コーディネーターに送信され、そこで制限や優先順位付けポリシーが適用された後、最終的に [実行ランチャー](/guides/deploy/execution/run-launchers) に送信されて起動されます。

## 実行コーディネーターの種類

[Dagster インスタンス](/guides/deploy/dagster-instance-configuration)では、以下の実行コーディネーターを設定できます。

| Term                                                                                                  | Definition                                                                                                                                                                                                                                                                                                                                                                                    |
| ----------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| <PyObject section="internals" module="dagster._core.run_coordinator" object="DefaultRunCoordinator"/> | `DefaultRunCoordinator` は、インスタンスの実行ランチャーの launch_run を、制限や優先順位ルールを適用することなく、同じプロセス内で直ちに呼び出します。<br />このコーディネータが設定されている場合、Dagster UI で **Launch Run** をクリックすると、Dagster デーモンプロセスから直ちに実行が開始されます。同様に、スケジュールされた実行もスケジューラプロセスから直ちに開始されます。 |
| <PyObject section="internals" module="dagster._core.run_coordinator" object="QueuedRunCoordinator"/>  | `QueuedRunCoordinator` は、実行キューを介して Dagster デーモンに実行を送信します。デーモンはキューから実行を取得し、送信された実行に対して launch_run を呼び出します。<br/>この実行コーディネータを使用すると、インスタンスレベルでの実行同時実行の制限と、カスタム実行優先順位ルールを設定できます。 |

## 実行コーディネーターの構成

`DefaultRunCoordinator` を使用する場合、お客様側での構成は不要です。

ただし、`QueuedRunCoordinator` を使用する場合、またはカスタム実装を構築する場合は、[カスタム実行優先順位ルール](/guides/deploy/execution/customizing-run-queue-priority) と [インスタンスレベルの同時実行制限](/guides/operate/managing-concurrency) を定義できます。
