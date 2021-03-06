<properties 
pageTitle="クラウド サービスのライフ サイクル イベントの処理 |Microsoft Azure" 
description=".NET でクラウド サービスのロールのライフサイクル メソッドを使用する方法について説明します。" 
services="cloud-services" 
documentationCenter=".net" 
authors="Thraka" 
manager="timlt" 
editor=""/>
<tags 
ms.service="cloud-services" 
ms.workload="tbd" 
ms.tgt_pltfrm="na" 
ms.devlang="na" 
ms.topic="article" 
ms.date="06/07/2016" 
ms.author="adegeo"/>

# .NET で Web または Worker ロールのライフサイクルをカスタマイズする

worker ロールを作成する際に、[RoleEntryPoint](https://msdn.microsoft.com/library/azure/microsoft.windowsazure.serviceruntime.roleentrypoint.aspx) クラスを拡張します。このクラスは、ライフサイクル イベントに応答できるようオーバーライドするメソッドを提供します。Web ロールの場合、このクラスは任意であり、必要に応じてライフサイクル イベントへの応答に使用する必要があります。

## RoleEntryPoint クラスを拡張する

[RoleEntryPoint](https://msdn.microsoft.com/library/azure/microsoft.windowsazure.serviceruntime.roleentrypoint.aspx) クラスには、Web ロールやworker ロールを**開始**、**実行**、**停止**するときに Azure が呼び出すメソッドが含まれています。必要に応じてこれらのメソッドをオーバーライドし、ロールの初期化、ロールのシャットダウン シーケンス、ロールの実行スレッドを管理できます。

**RoleEntryPoint** を拡張した場合、メソッドの次のような動作に注意する必要があります。

-   [OnStart](https://msdn.microsoft.com/library/azure/microsoft.windowsazure.serviceruntime.roleentrypoint.onstart.aspx) メソッドと [OnStop](https://msdn.microsoft.com/library/azure/microsoft.windowsazure.serviceruntime.roleentrypoint.onstop.aspx) メソッドは、ブール値を返します。したがって、これらのメソッドから **false** を返すこともできます。

     コードが **false** を返した場合、ロール プロセスは指定したシャットダウン シーケンスを実行せずに突然終了します。一般的には、**OnStart** メソッドから **false** は返さないようにしてください。
     
-   **RoleEntryPoint** メソッドのオーバーロード内でキャッチされなかった例外は、未処理の例外として扱われます。

     例外がライフサイクル メソッドの内部で発生すると、Azure で [UnhandledException](https://msdn.microsoft.com/library/system.appdomain.unhandledexception.aspx) イベントが発生し、プロセスが終了します。ロールは、オフラインになった後、Azure によって再開されます。未処理の例外が発生した場合、[Stopping](https://msdn.microsoft.com/library/azure/microsoft.windowsazure.serviceruntime.roleenvironment.stopping.aspx) イベントは発生せず、**OnStop** メソッドは呼び出されません。

ロールが開始しない場合や、ロールが初期化、ビジー状態、停止中の状態で再利用されている場合、コードが再開されるたびに、ライフサイクル イベントの内部で未処理の例外がスローされる場合があります。この場合は、[UnhandledException](https://msdn.microsoft.com/library/system.appdomain.unhandledexception.aspx) イベント使用して、例外の原因を特定し適切に処理します。ロールは、[Run](https://msdn.microsoft.com/library/azure/microsoft.windowsazure.serviceruntime.roleentrypoint.run.aspx) メソッドから戻ることもあります。その場合、ロールは再開されます。デプロイ状態の詳細については、「[ロールがリサイクルされる一般的な問題](https://msdn.microsoft.com/library/azure/gg465402.aspx)」をご覧ください。

> [AZURE.NOTE] [Azure Tools for Microsoft Visual Studio](https://msdn.microsoft.com/library/azure/ee405484.aspx) を使用してアプリケーションを開発している場合は、ロール プロジェクトのテンプレートによって WebRole.cs ファイルと WorkerRole.cs ファイルで **RoleEntryPoint** クラスが自動的に拡張されます。

## OnStart メソッド

**OnStart** メソッドは、ロール インスタンスが Azure によってオンラインに切り替えられたときに呼び出されます。OnStart コードの実行中、ロール インスタンスは "**ビジー**" とマークされ、外部トラフィックはロード バランサーからこのロールに送信されません。このメソッドをオーバーライドして、イベント ハンドラーの実装や [Azure 診断](cloud-services-how-to-monitor.md)の開始などの初期化作業を実行できます。

**OnStart** が **true** を返す場合、インスタンスは正常に初期化されており、Azure は **RoleEntryPoint.Run** メソッドを呼び出します。**OnStart** が **false** を返す場合、ロールは予定されたシャットダウン シーケンスを実行せず、すぐに終了します。

**OnStart** メソッドをオーバーライドするコード例を次に示します。このメソッドは、ロール インスタンスが開始されたときに診断モニターを構成して開始し、ストレージ アカウントへのログ データの転送をセットアップします。

```csharp
public override bool OnStart()
{
    var config = DiagnosticMonitor.GetDefaultInitialConfiguration();

    config.DiagnosticInfrastructureLogs.ScheduledTransferLogLevelFilter = LogLevel.Error;
    config.DiagnosticInfrastructureLogs.ScheduledTransferPeriod = TimeSpan.FromMinutes(5);

    DiagnosticMonitor.Start("DiagnosticsConnectionString", config);

    return true;
}
```

## OnStop メソッド

**OnStop** メソッドは、Azure によってロール インスタンスがオフラインになった後、プロセスが終了する前に呼び出されます。このメソッドをオーバーライドして、ロール インスタンスが完全にシャットダウンするのに必要なコードを呼び出すことができます。

> [AZURE.IMPORTANT] **OnStop** メソッドで実行されるコードは、ユーザーが開始したシャットダウン以外の理由で呼び出された場合、限られた時間内で処理を完了する必要があります。この時間が経過するとプロセスは終了するため、**OnStop** メソッドのコードが短時間で実行できるか、最後まで実行されなくても許容できることを確認する必要があります。**OnStop** メソッドは、**Stopping** イベントが発生した後に呼び出されます。


## Run メソッド

**Run** メソッドをオーバーライドして、ロール インスタンスの実行時間の長いスレッドを実装できます。

**Run** メソッドのオーバーライドは必須ではありません。既定の実装では、スリープ状態を永遠に続けるスレッドが開始されます。**Run** メソッドをオーバーライドした場合、コードは無期限にブロックされます。**Run** メソッドから制御が戻った場合、ロールは自動的に適切に再利用されます。言い換えれば、Azure で **Stopping** イベントが発生し、**OnStop** メソッドが呼び出されて、ロールがオフラインになる前にシャットダウン シーケンスを実行できます。


### Web ロール用の ASP.NET ライフサイクル メソッドの実装

Web ロールの初期化とシャットダウン シーケンスを管理するには、**RoleEntryPoint** クラスで提供されるメソッドのほかに、ASP.NET のライフサイクル メソッドも使用できます。これは既存の ASP.NET アプリケーションを Azure に移植する場合に、互換性の面で役立つことがあります。ASP.NET のライフサイクル メソッドは、**RoleEntryPoint** メソッドの内部から呼び出されます。**Application\_Start** メソッドは、**RoleEntryPoint.OnStart** メソッドが終了した後に呼び出されます。**Application\_End** メソッドは、**RoleEntryPoint.OnStop** メソッドが呼び出される前に呼び出されます。

## 次のステップ
[クラウド サービス パッケージを作成する方法](cloud-services-model-and-package.md)について説明します。

<!---HONumber=AcomDC_0608_2016-->