<properties
   pageTitle="Azure Batch PowerShell の使用 | Microsoft Azure"
   description="Azure Batch サービスの管理に使用できる Azure PowerShell のコマンドレットについて簡単に説明します。"
   services="batch"
   documentationCenter=""
   authors="dlepow"
   manager="timlt"
   editor=""/>

<tags
   ms.service="batch"
   ms.devlang="NA"
   ms.topic="get-started-article"
   ms.tgt_pltfrm="powershell"
   ms.workload="big-compute"
   ms.date="04/21/2016"
   ms.author="danlep"/>

# Azure Batch PowerShell コマンドレットの使用
この記事では、Batch アカウントの管理、およびプール、ジョブ、タスクなどの Batch リソースの操作に使用できる Azure PowerShell コマンドレットについて説明します。Batch API、Azure ポータル、Azure コマンド ライン インターフェイス (CLI) で実行するタスクの多くは、Batch コマンドレットでも実行できます。この記事は、Azure PowerShell Version 1.3.2 以降のコマンドレットを基にしています。

すべての Batch コマンドレットの一覧およびコマンドレットの詳細な構文については、[Azure Batch コマンドレットのリファレンス](https://msdn.microsoft.com/library/azure/mt125957.aspx)を参照してください。


## 前提条件

* **Azure PowerShell** - Azure PowerShell をダウンロードしてインストールする手順については、「[Azure PowerShell のインストールと構成の方法](../powershell-install-configure.md)」をご覧ください。 
   
    * Azure Batch コマンドレットは Azure Resource Manager モジュールに付属しているので、**Login-AzureRmAccount** コマンドレットを使用してサブスクリプションに接続する必要があります。 
    
    * 最新のサービスや機能強化を活かすためにも、Azure PowerShell は定期的に更新することをお勧めします。
    
* **Batch プロバイダーの名前空間に登録する (1 回限りの操作)** - Batch アカウントを利用するには、Batch プロバイダーの名前空間に登録する必要があります。この操作は、サブスクリプションごとに 1 回のみ実行する必要があります。次のコマンドレットを実行します。

        Register-AzureRMResourceProvider -ProviderNamespace Microsoft.Batch


## Batch アカウントとキーを管理する

### Batch アカウントを作成する

**New-AzureRmBatchAccount** は、指定したリソース グループに新しい Batch アカウントを作成します。リソース グループがない場合は、[New-AzureRmResourceGroup](https://msdn.microsoft.com/library/azure/mt603739.aspx) コマンドレットを実行して作成し、**Location** パラメーターでいずれかの Azure リージョン ("米国中央部" など) を指定します 次に例を示します。


    New-AzureRmResourceGroup –Name MyBatchResourceGroup –location "Central US"


次に、リソース グループに新しい Batch アカウントを作成し、<*account\_name*> にアカウント名と、Batch サービスが使用可能な場所を指定します。アカウントの作成は完了までに数分かかる場合があります。次に例を示します。


    New-AzureRmBatchAccount –AccountName <account_name> –Location "Central US" –ResourceGroupName MyBatchResourceGroup

> [AZURE.NOTE] Batch アカウント名は、リソース グループの Azure リージョン内で重複しないこと、文字数が 3 ～ 24 文字であること、小文字と数字のみで構成されていることが必要になります。

### アカウントのアクセス キーを取得する
**Get-AzureRmBatchAccountKeys** は、Azure Batch アカウントに関連付けられているアクセス キーを示します。たとえば、作成したアカウントのプライマリ キーとセカンダリ キーを取得するには、次のように実行します。

    $Account = Get-AzureRmBatchAccountKeys –AccountName <accountname>

    $Account.PrimaryAccountKey

    $Account.SecondaryAccountKey


### 新しいアクセス キーを生成する
**New-AzureRmBatchAccountKey** は、Azure Batch アカウントの新しいプライマリ アカウント キーまたはセカンダリ アカウント キーを生成します。たとえば、Batch アカウントの新しいプライマリ キーを生成するには、次のように入力します。


    New-AzureRmBatchAccountKey -AccountName <account_name> -KeyType Primary


> [AZURE.NOTE] 新しいセカンダリ キーを生成するには、**KeyType** パラメーターの "Secondary" を指定します。プライマリ キーとセカンダリ キーは個別に再生成する必要があります。

### Batch アカウントを削除する
**Remove-AzureRmBatchAccount** は、Batch アカウントを削除します。次に例を示します。


    Remove-AzureRmBatchAccount -AccountName <account_name>

メッセージが表示されたら、削除するアカウントを確認します。アカウントの削除は完了するまでに時間がかかる場合があります。

## BatchAccountContext オブジェクトを作成する

Batch PowerShell コマンドレットを使用して Batch のリソース (プール、ジョブ、タスクなど) を認証するにはまず、BatchAccountContext オブジェクトを作成して、アカウントの名前とキーを格納します。

    $context = Get-AzureRmBatchAccountKeys -AccountName <account_name>

BatchAccountContext オブジェクトは、**BatchContext** パラメーターを引数として受け取るコマンドレットに渡します。

> [AZURE.NOTE] 既定では、アカウントのプライマリ キーは認証に使用されますが、BatchAccountContext オブジェクトの **KeyInUse** プロパティ `$context.KeyInUse = "Secondary"` を変更することで使用するキーを明示的に選択できます。



## Batch リソースを作成および変更する
Batch アカウントでリソースを作成するには、**New-AzureBatchPool**、**New-AzureBatchJob**、**New-AzureBatchTask** などのコマンドレットを使用します。既存のリソースのプロパティを更新するための対応する **Get-** コマンドレットと **Set-** コマンドレットがあり、Batch アカウントのリソースを削除するための **Remove-** コマンドレットがあります。

### Create a Batch pool

たとえば、次のコマンドレットは、ファミリ 3 (Windows Server 2012) の最新のオペレーティング システム バージョンと、自動スケール式によって決定される目標のコンピューティング ノード数でイメージを作成された Small サイズの仮想マシンを使用するように構成された新しい Batch プールを作成します。この例では、式は **$TargetDedicated=3** という簡単なものであり、プールのコンピューティング ノードの数が最大 3 であることを示します。**BatchContext** パラメーターは、BatchAccountContext オブジェクトとして先に定義した変数 *$context* を指定します。


    New-AzureBatchPool -Id "MyAutoScalePool" -VirtualMachineSize "Small" -OSFamily "3" -TargetOSVersion "*" -AutoScaleFormula '$TargetDedicated=3;' -BatchContext $Context

>[AZURE.NOTE]現在、Batch PowerShell コマンドレットでサポートされるのは、計算ノードに対するクラウド サービスの構成だけです。たとえば、計算ノードで実行する Windows Server オペレーティング システムとして、Azure ゲスト OS の中からいずれかのリリースを選択できます。Batch プールの計算ノードに対するその他の構成オプションについては、Batch SDK または Azure CLI をご利用ください。

## プール、ジョブ、タスク、およびその他の詳細のクエリ

**Get-AzureBatchPool**、**Get-AzureBatchJob**、**Get-AzureBatchTask** などのコマンドレットを使用して、Batch アカウントで作成されたエンティティを照会します。


### データのクエリ

たとえば、**Get-AzureBatchPools** を使用してプールを検索します。既定では、これは、既に BatchAccountContext オブジェクトが *$context* に格納されていると仮定して、自分のアカウントのすべてのプールを照会します。


    Get-AzureBatchPool -BatchContext $context

### OData フィルターを使用する

**Filter** パラメーターを使用して OData フィルターを指定すると、関心のあるオブジェクトのみを検索できます。たとえば、ID が "myPool" で始まるすべてのプールを検索できます。


    $filter = "startswith(id,'myPool')"

    Get-AzureBatchPool -Filter $filter -BatchContext $context


このメソッドは、ローカルのパイプラインで “Where-Object” を使用するほど柔軟ではありません。ただし、クエリは Batch サービスに直接送信されるため、すべてのフィルター処理がサーバー側で行われ、インターネットの帯域幅を節約できます。

### ID パラメーターの使用

OData フィルターに代わる方法として、**ID** パラメーターを使用します。"myPool" という ID の特定のプールを照会するには:


    Get-AzureBatchPool -Id "myPool" -BatchContext $context


**ID** パラメーターは、完全 ID の検索のみをサポートし、ワイルドカードや OData 形式のフィルターはサポートしません。



### MaxCount パラメーターを使用する

既定では、各コマンドレットは最大で 1000 のオブジェクトを返します。この制限に達した場合は、オブジェクトが少なくなるようにフィルターで絞り込むか、**MaxCount** パラメーターを使用して明示的に最大値を設定してください。次に例を示します。


    Get-AzureBatchTask -MaxCount 2500 -BatchContext $context

上限を削除するには、**MaxCount** を 0 以下に設定します。

### パイプラインを作成する

Batch コマンドレットは、コマンドレット間でデータを送信するために PowerShell パイプラインを活用できます。これは、パラメーターを指定するのと同じ効果がありますが、複数のエンティティを簡単に一覧表示できます。たとえば、次のコマンドでは、自分のアカウントのすべてのタスクを検索しています。


    Get-AzureBatchJob -BatchContext $context | Get-AzureBatchTask -BatchContext $context


## 次のステップ
* コマンドレットの詳しい構文と例については、[Azure Batch コマンドレットのリファレンス](https://msdn.microsoft.com/library/azure/mt125957.aspx)を参照してください。

* Batch に対するクエリから返される情報の項目数と種類を制限する方法について詳しくは、「[効率的な Azure Batch サービスのクエリ](batch-efficient-list-queries.md)」を参照してください。

<!---HONumber=AcomDC_0427_2016-->