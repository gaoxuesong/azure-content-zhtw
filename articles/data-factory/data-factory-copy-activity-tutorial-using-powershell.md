<properties 
	pageTitle="教學課程：使用 Azure PowerShell 建立具有複製活動的管線 | Microsoft Azure" 
	description="在本教學課程中，您會使用 Azure PowerShell，建立具有複製活動的 Azure Data Factory 管線。" 
	services="data-factory" 
	documentationCenter="" 
	authors="spelluru" 
	manager="jhubbard" 
	editor="monicar"/>

<tags 
	ms.service="data-factory" 
	ms.workload="data-services" 
	ms.tgt_pltfrm="na" 
	ms.devlang="na" 
	ms.topic="get-started-article" 
	ms.date="09/16/2016" 
	ms.author="spelluru"/>

# 教學課程：使用 Azure PowerShell 建立具有複製活動的管線
> [AZURE.SELECTOR]
- [概觀和必要條件](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md)
- [Azure 入口網站](data-factory-copy-activity-tutorial-using-azure-portal.md)
- [Visual Studio](data-factory-copy-activity-tutorial-using-visual-studio.md)
- [PowerShell](data-factory-copy-activity-tutorial-using-powershell.md)
- [REST API](data-factory-copy-activity-tutorial-using-rest-api.md)
- [.NET API](data-factory-copy-activity-tutorial-using-dotnet-api.md)
- [複製精靈](data-factory-copy-data-wizard-tutorial.md)

在本教學課程中，您會使用 Azure PowerShell Cmdlet 來建立和監視 Azure Data Factory。在本教學課程中，您於 Data Factory 中建立的管線會使用複製活動將資料從 Azure Blob 複製到 Azure SQL Database。

複製活動會在 Azure Data Factory 中執行資料移動。此活動是由全域可用的服務所提供，可以使用安全、可靠及可調整的方式，在各種不同的資料存放區之間複製資料。如需複製活動的詳細資訊，請參閱[資料移動活動](data-factory-data-movement-activities.md)文章。

> [AZURE.IMPORTANT] 
請檢閱[教學課程概觀](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md)文章，並在執行本教學課程之前完成**必要**步驟。
>   
> 這篇文章並未涵蓋所有的 Data Factory Cmdlet。如需 Data Factory Cmdlet 的完整文件，請參閱 [Data Factory Cmdlet 參考](https://msdn.microsoft.com/library/dn820234.aspx)。
  

##必要條件
除了「教學課程概觀」主題中所列的必要條件，您還需要安裝 **Azure PowerShell**。按照[如何安裝和設定 Azure PowerShell](../powershell-install-configure.md) 一文中的指示操作，在您的電腦上安裝 Azure PowerShell。

##本教學課程內容
下表列出您在本教學課程中將執行的步驟及說明。

步驟 | 說明
-----| -----------
[建立 Azure Data Factory](#create-data-factory) | 在此步驟中，您會建立名為 **ADFTutorialDataFactoryPSH** 的 Azure Data Factory。 
[建立連結服務](#create-linked-services) | 在此步驟中，您會建立兩個連結服務：**StorageLinkedService** 和 **AzureSqlLinkedService**。StorageLinkedService 會連結 Azure 儲存體，而 AzureSqlLinkedService 會將 Azure SQL Database 連結至 ADFTutorialDataFactoryPSH。
[建立輸入和輸出資料集](#create-datasets) | 在此步驟中，您會定義兩個資料集 (**EmpTableFromBlob** 和 **EmpSQLTable**)。當您在下一個步驟中建立 ADFTutorialPipeline 時，這兩個資料集可做為**複製活動**的輸入和輸出資料表。
[建立及執行管線](#create-pipeline) | 在此步驟中，您會在 Data Factory **ADFTutorialDataFactoryPSH** 中建立名為 **ADFTutorialPipeline** 的管線。此管線的**複製活動**會將資料從 Azure Blob 複製到輸出 Azure 資料庫資料表。
[監視資料集和管線](#monitor-pipeline) | 在此步驟中，您會使用 Azure PowerShell 來監視資料集和管線。

## 建立 Data Factory
在此步驟中，您會使用 Azure PowerShell 建立名為 **ADFTutorialDataFactoryPSH** 的 Azure Data Factory。

1. 啟動 **PowerShell**。將 Azure PowerShell 維持在開啟狀態，直到本教學課程結束為止。如果您關閉並重新開啟，則需要再次執行這些命令。
	1. 執行下列命令並輸入您用來登入 Azure 入口網站的使用者名稱和密碼：
	
			Login-AzureRmAccount   
	2. 執行下列命令以檢視此帳戶的所有訂用帳戶：

			Get-AzureRmSubscription 
	3. 執行下列命令以選取您要使用的訂用帳戶。以您的 Azure 訂用帳戶名稱取代 **&lt;NameOfAzureSubscription**&gt;：

			Get-AzureRmSubscription -SubscriptionName <NameOfAzureSubscription> | Set-AzureRmContext

3. 執行以下命令，建立名為 **ADFTutorialResourceGroup** 的 Azure 資源群組：
   
		New-AzureRmResourceGroup -Name ADFTutorialResourceGroup  -Location "West US"

	本教學課程的某些步驟會假設您使用名為 **ADFTutorialResourceGroup** 的資源群組。如果使用不同的資源群組，您必須以該群組取代本教學課程中的 ADFTutorialResourceGroup。
4. 執行 **New-AzureRmDataFactory** Cmdlet，建立名為 **ADFTutorialDataFactoryPSH** 的 Data Factory：

		New-AzureRmDataFactory -ResourceGroupName ADFTutorialResourceGroup -Name ADFTutorialDataFactoryPSH –Location "West US"

	
請注意下列幾點：
 
- Azure Data Factory 的名稱在全域必須是唯一的。如果您收到錯誤，請變更名稱 (例如 yournameADFTutorialDataFactoryPSH)。執行本教學課程中的步驟時，請使用此名稱來取代 ADFTutorialFactoryPSH。請參閱 [Data Factory - 命名規則](data-factory-naming-rules.md)主題，以了解 Data Factory 成品的命名規則。
	
		Data factory name “ADFTutorialDataFactoryPSH” is not available
- 若要建立 Data Factory 執行個體，您必須是 Azure 訂用帳戶的參與者/系統管理員
- Data Factory 的名稱未來可能會註冊為 DNS 名稱，因此會變成公開可見的名稱。
- 如果您收到錯誤：「此訂用帳戶未註冊為使用命名空間 Microsoft.DataFactory」，請執行下列其中一項，然後嘗試再次發佈︰

	- 在 Azure PowerShell 中，執行下列命令以註冊 Data Factory 提供者：
		
			Register-AzureRmResourceProvider -ProviderNamespace Microsoft.DataFactory
	
		執行下列命令來確認已註冊 Data Factory 提供者：
	
			Get-AzureRmResourceProvider
	- 使用 Azure 訂用帳戶登入 [Azure 入口網站](https://portal.azure.com)並瀏覽至 [Data Factory] 刀鋒視窗 (或) 在 Azure 入口網站中建立 Data Factory。此動作會自動為您註冊提供者。

## 建立連結服務
連結服務會將資料存放區或計算服務連結至 Azure Data Factory。資料存放區可以是 Azure 儲存體、Azure SQL Database 或內部部署 SQL Server 資料庫，且內含 Data Factory 管線的輸入資料或儲存其輸出資料。計算服務是處理輸入資料並產生輸出資料的服務。

在此步驟中，您會建立兩個連結服務：**StorageLinkedService** 和 **AzureSqlLinkedService**。StorageLinkedService 連結服務會連結 Azure 儲存體帳戶，而 AzureSqlLinkedService 會將 Azure SQL Database 連結至 Data Factory：**ADFTutorialDataFactoryPSH**。您稍後會在本教學課程中建立管線，以將資料從 StorageLinkedService 中的 Blob 容器複製到 AzureSqlLinkedService 中的 SQL 資料表。

### 建立 Azure 儲存體帳戶的連結服務
1.	在 **C:\\ADFGetStartedPSH** 中，使用以下內容建立名為 **StorageLinkedService.json** 的 JSON 檔案。建立 ADFGetStartedPSH 資料夾 (如果不存在)。

			{
		  		"name": "StorageLinkedService",
		  		"properties": {
	    			"type": "AzureStorage",
		    		"typeProperties": {
		      			"connectionString": "DefaultEndpointsProtocol=https;AccountName=<accountname>;AccountKey=<accountkey>"
		    		}
		  		}
			}

	以 Azure 儲存體帳戶的名稱和金鑰取代 **accountname** 和 **accountkey**。
2.	在 **Azure PowerShell** 中，切換到 **ADFGetStartedPSH** 資料夾。
3.	您可以使用 **New-AzureRmDataFactoryLinkedService** Cmdlet 建立連結服務。此 Cmdlet 和您在本教學課程中使用的其他 Data Factory Cmdlet，皆需要您將值傳給 **ResourceGroupName** 和 **DataFactoryName** 參數。或者，您可以使用 **Get-AzureRmDataFactory** 取得 DataFactory 物件，並傳遞此物件，就不需要在每次執行 Cmdlet 時輸入 ResourceGroupName 和 DataFactoryName。執行以下命令，將 **Get-AzureRmDataFactory** Cmdlet 的輸出指派給變數 **$df**：

		$df=Get-AzureRmDataFactory -ResourceGroupName ADFTutorialResourceGroup -Name ADFTutorialDataFactoryPSH

4.	現在，執行 **New-AzureRmDataFactoryLinkedService** Cmdlet 建立連結的服務：**StorageLinkedService**。

		New-AzureRmDataFactoryLinkedService $df -File .\StorageLinkedService.json

	如果您還沒執行 **Get-AzureRmDataFactory** Cmdlet 和指派輸出給 **$df** 變數，您就必須指定 ResourceGroupName 和 DataFactoryName 參數的值，如下所示。
		
		New-AzureRmDataFactoryLinkedService -ResourceGroupName ADFTutorialResourceGroup -DataFactoryName ADFTutorialDataFactoryPSH -File .\StorageLinkedService.json

	如果您在教學課程的中途關閉 Azure PowerShell，則下次啟動 Azure PowerShell 時必須執行 Get-AzureRmDataFactory Cmdlet，才能完成教學課程。

### 建立 Azure SQL Database 的連結服務
1.	使用以下內容建立名為 AzureSqlLinkedService.json 的 JSON 檔案：

			{
				"name": "AzureSqlLinkedService",
				"properties": {
					"type": "AzureSqlDatabase",
					"typeProperties": {
				      	"connectionString": "Server=tcp:<server>.database.windows.net,1433;Database=<databasename>;User ID=<user>@<server>;Password=<password>;Trusted_Connection=False;Encrypt=True;Connection Timeout=30"
					}
		  		}
			}

	將 **servername**、**databasename**、**username@servername** 和 **password** 替換為您的 Azure SQL 伺服器名稱、資料庫名稱、使用者帳戶和密碼。

2.	執行以下命令建立連結服務：
	
		New-AzureRmDataFactoryLinkedService $df -File .\AzureSqlLinkedService.json

	確認 Azure SQL Server 已開啟 [允許存取 Azure 服務] 設定。若要確認並開啟此設定，請執行下列步驟：

	1. 按一下左邊的 [瀏覽] 中樞，然後按一下 [SQL Server]。
	2. 選取您的伺服器，然後按一下 [SQL SERVER] 刀鋒視窗上的 [**設定**]。
	3. 在 [**設定**] 刀鋒視窗中，按一下 [**防火牆**]。
	4. 在 [**防火牆設定**] 刀鋒視窗中，對 [**允許存取 Azure 服務**] 按一下 [**開啟**]。
	5. 按一下左邊的 [作用中] 中樞，切換至原先的 [Data Factory] 刀鋒視窗。
	

## 建立資料集

在上一個步驟中，您已建立連結服務 **StorageLinkedService** 和 **AzureSqlLinkedService**，將 Azure 儲存體帳戶和 Azure SQL Database 連結至 Data Factory：**ADFTutorialDataFactoryPSH**。在此步驟中，您會建立資料集，而這些資料集在下一個步驟所建立的管線中代表複製活動的輸入和輸出資料。

資料表是矩形的資料集，而且是目前唯一受支援的資料集類型。本教學課程中的輸入資料表是指 Azure 儲存體中 StorageLinkedService 所指向的 Blob 容器。輸出資料表則是指 Azure SQL Database 中 AzureSqlLinkedService 所指向的 SQL 資料表。

### 準備教學課程所需的 Azure Blob 儲存體和 Azure SQL 資料庫
如果您已完成[將資料從 Blob 儲存體複製到 SQL Database](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md)一文中的教學課程，請略過此步驟。

執行下列步驟來準備本教學課程中的 Azure Blob 儲存體和 Azure SQL Database。
 
* 在 **StorageLinkedService** 所指向的 Azure Blob 儲存體中，建立名為 **adftutorial** 的 Blob 容器。
* 建立文字檔 **emp.txt**，並以 Blob 形式上傳至 **adftutorial** 容器。
* 在 **AzureSqlLinkedService** 所指向的 Azure SQL Database中，建立名為 **emp** 的資料表。


1. 啟動 [記事本]，貼上以下文字，並命名為 **emp.txt**，然後儲存至您硬碟上的 **C:\\ADFGetStartedPSH** 資料夾。

        John, Doe
		Jane, Doe
				
2. 使用 [Azure 儲存體總管](https://azurestorageexplorer.codeplex.com/)這類的工具建立 **adftutorial** 容器，以及將 **emp.txt** 檔案上傳至該容器。

    ![Azure 儲存體總管](media/data-factory-copy-activity-tutorial-using-powershell/getstarted-storage-explorer.png)
3. 使用以下 SQL 指令碼，在您的 Azure SQL Database 中建立 **emp** 資料表。


        CREATE TABLE dbo.emp 
		(
			ID int IDENTITY(1,1) NOT NULL,
			FirstName varchar(50),
			LastName varchar(50),
		)
		GO

		CREATE CLUSTERED INDEX IX_emp_ID ON dbo.emp (ID); 

	如果您的電腦上已安裝 SQL Server 2014：請遵循[步驟 2：使用 SQL Server Management Studio 連接到管理 Azure SQL Database 的 SQL Database](../sql-database/sql-database-manage-azure-ssms.md) 文章中的指示，連接到您的 Azure SQL Server，並執行 SQL 指令碼。

	如果不允許您的用戶端存取 Azure SQL Server，則必須將 Azure SQL Server 的防火牆設定成允許從您的電腦 (IP 位址) 存取。請參閱[本文](../sql-database/sql-database-configure-firewall-settings.md)的步驟，為 Azure SQL Server 設定防火牆。
		
### 建立輸入資料集 
資料表是矩形的資料集，並具有的結構描述。在此步驟中，您會在 **StorageLinkedService** 連結服務所代表的 Azure 儲存體中，建立指向 Blob 容器的 **EmpBlobTable** 資料表。此 Blob 容器 (**adftutorial**) 包含的輸入資料位於 **emp.txt** 檔案中。

1.	在 **C:\\ADFGetStartedPSH** 資料夾中，使用下列內容建立名為 **EmpBlobTable.json** 的 JSON 檔案：

			{
			  "name": "EmpTableFromBlob",
			  "properties": {
			    "structure": [
			      {
			        "name": "FirstName",
			        "type": "String"
			      },
			      {
			        "name": "LastName",
			        "type": "String"
			      }
			    ],
			    "type": "AzureBlob",
			    "linkedServiceName": "StorageLinkedService",
			    "typeProperties": {
				  "fileName": "emp.txt",
			      "folderPath": "adftutorial/",
			      "format": {
			        "type": "TextFormat",
			        "columnDelimiter": ","
			      }
			    },
			    "external": true,
			    "availability": {
			      "frequency": "Hour",
			      "interval": 1
			    }
			  }
			}
	
	請注意下列幾點：
	
	- 資料集 **type** 設為 **AzureBlob**。
	- **linkedServiceName** 設為 **StorageLinkedService**。
	- **folderPath** 設為 **adftutorial** 容器。
	- **fileName** 設為 **emp.txt**。如果您未指定 Blob 的名稱，容器中所有 Blob 的資料都會被視為輸入資料。
	- 格式 **type** 設為 **TextFormat**
	- 文字檔中有兩個欄位 (**FirstName** 和 **LastName**)，以逗號字元分隔 (**columnDelimiter**)
	- **availability** 設定為**每小時** (**frequency** 設定為**小時**，**interval** 設定為 **1**)。因此，Data Factory 會每小時在 Blob 容器 (**adftutorial**) 的根資料夾中尋找輸入資料。

	如果您沒有指定**輸入** **資料表**的 **fileName**，則輸入資料夾 (**folderPath**) 中的所有檔案/Blob 都會視為輸入。如果您在 JSON 中指定 fileName，則只有指定的檔案/Blob 會被視為輸入。
 
	如果您未指定輸出資料表的 **fileName**，**folderPath** 中產生的檔案會依照下列格式命名：Data.<Guid>.txt (範例：Data.0a405f8a-93ff-4c6f-b3be-f69616f1df7a.txt.)。

	若要根據 **SliceStart** 時間動態設定 **folderPath** 和 **fileName**，請使用 **partitionedBy** 屬性。在下列範例中，folderPath 使用 SliceStart (所處理配量的開始時間) 中的年、月和日，fileName 使用 SliceStart 中的小時。例如，如果配量產生於 2016-10-20T08:00:00，folderName 設定為 wikidatagateway/wikisampledataout/2016/10/20，而 fileName 設定為 08.csv。

			"folderPath": "wikidatagateway/wikisampledataout/{Year}/{Month}/{Day}",
	        "fileName": "{Hour}.csv",
	        "partitionedBy": 
	        [
	        	{ "name": "Year", "value": { "type": "DateTime", "date": "SliceStart", "format": "yyyy" } },
	            { "name": "Month", "value": { "type": "DateTime", "date": "SliceStart", "format": "MM" } }, 
	            { "name": "Day", "value": { "type": "DateTime", "date": "SliceStart", "format": "dd" } }, 
	            { "name": "Hour", "value": { "type": "DateTime", "date": "SliceStart", "format": "hh" } } 
	        ],

	如需 JSON 屬性的詳細資料，請參閱 [JSON 指令碼參考](http://go.microsoft.com/fwlink/?LinkId=516971)。

2.	執行以下命令建立 Data Factory 資料集。

		New-AzureRmDataFactoryDataset $df -File .\EmpBlobTable.json

### 建立輸出資料集
在此步驟中，您會建立名為 **EmpSQLTable** 的輸出資料集。此資料集指向 Azure SQL Database 中 **AzureSqlLinkedService** 所代表的 SQL 資料表 (**emp**)。管線會將輸入 Blob 中的資料複製到 **emp** 資料表。

1.	在 **C:\\ADFGetStartedPSH** 資料夾中，使用下列內容建立名為 **EmpSQLTable.json** 的 JSON 檔案：
		
			{
			  "name": "EmpSQLTable",
			  "properties": {
			    "structure": [
			      {
			        "name": "FirstName",
			        "type": "String"
			      },
			      {
			        "name": "LastName",
			        "type": "String"
			      }
			    ],
			    "type": "AzureSqlTable",
			    "linkedServiceName": "AzureSqlLinkedService",
			    "typeProperties": {
			      "tableName": "emp"
			    },
			    "availability": {
			      "frequency": "Hour",
			      "interval": 1
			    }
			  }
			}

     請注意下列幾點：
	
	* 資料集 **type** 設為 **AzureSQLTable**。
	* **linkedServiceName** 設為 **AzureSqlLinkedService**。
	* **tablename** 設為 **emp**。
	* 資料庫的 emp 資料表中有三個資料行 – **ID**、**FirstName** 和 **LastName**。ID 是識別資料行，所以您只需在此指定 **FirstName** 和 **LastName**。
	* **availability** 設定為**每小時**，且 (**frequency** 設定為**小時**，**interval** 設定為 **1**)。Data Factory 服務會每隔一小時在 Azure SQL Database 的 **emp** 資料表中產生輸出資料配量。

2.	執行以下命令建立 Data Factory 資料集。
	
		New-AzureRmDataFactoryDataset $df -File .\EmpSQLTable.json


## 建立管線
在此步驟中，您會建立管線，其中含有使用 **EmpTableFromBlob** 做為輸入和使用 **EmpSQLTable** 做為輸出的複製活動。

1.	在 **C:\\ADFGetStartedPSH** 資料夾中，使用下列內容建立名為 **ADFTutorialPipeline.json** 的 JSON 檔案。
	
			 {
			  "name": "ADFTutorialPipeline",
			  "properties": {
			    "description": "Copy data from a blob to Azure SQL table",
			    "activities": [
			      {
			        "name": "CopyFromBlobToSQL",
			        "description": "Push Regional Effectiveness Campaign data to Azure SQL database",
			        "type": "Copy",
			        "inputs": [
			          {
			            "name": "EmpTableFromBlob"
			          }
			        ],
			        "outputs": [
			          {
			            "name": "EmpSQLTable"
			          }
			        ],
			        "typeProperties": {
			          "source": {
			            "type": "BlobSource"
			          },
			          "sink": {
			            "type": "SqlSink"
			          }
			        },
			        "Policy": {
			          "concurrency": 1,
			          "executionPriorityOrder": "NewestFirst",
			          "style": "StartOfInterval",
			          "retry": 0,
			          "timeout": "01:00:00"
			        }
			      }
			    ],
			    "start": "2016-08-09T00:00:00Z",
			    "end": "2016-08-10T00:00:00Z",
			    "isPaused": false
			  }
			}

	請注意下列幾點：

	- 在活動區段中，只會有一個 **type** 設為 **Copy** 的活動。
	- 活動的輸入設為 **EmpTableFromBlob**，活動的輸出則設為 **EmpSQLTable**。
	- 在 **transformation** 區段中，來源類型指定為 **BlobSource**，接收類型指定為 **SqlSink**。

	將 **start** 屬性的值取代為目前日期，並將 **end** 值取代為隔天的日期。開始和結束日期時間都必須是 [ISO 格式](http://en.wikipedia.org/wiki/ISO_8601)。例如：2016-10-14T16:32:41Z。**end** 時間為選擇性項目，但在本教學課程中會用到。
	
	如果您未指定 **end** 屬性的值，則會以「**start + 48 小時**」計算。若要無限期地執行管線，請指定 **9/9/9999** 做為 **end** 屬性的值。
	
	在此範例中，由於每小時即產生一個資料配量，共會有 24 個資料配量。
	
	如需 JSON 屬性的詳細資料，請參閱 [JSON 指令碼參考](http://go.microsoft.com/fwlink/?LinkId=516971)。
2.	執行以下命令建立 Data Factory 資料表。
		
		New-AzureRmDataFactoryPipeline $df -File .\ADFTutorialPipeline.json

**恭喜！** 您已成功建立 Azure Data Factory、連結服務、資料表和管線，以及排定的管線。

## 監視管線
在此步驟中，您會使用 Azure PowerShell 來監視 Azure Data Factory 的運作情形。

1.	執行 **Get-AzureRmDataFactory** 並指派輸出給變數 $df。

		$df=Get-AzureRmDataFactory -ResourceGroupName ADFTutorialResourceGroup -Name ADFTutorialDataFactoryPSH
 
2.	執行 **Get-AzureRmDataFactorySlice**，以取得 **EmpSQLTable** (管線的輸出資料表) 所有配量的詳細資料。

		Get-AzureRmDataFactorySlice $df -DatasetName EmpSQLTable -StartDateTime 2016-08-09T00:00:00

	以目前的年、月、日，取代 **StartDateTime** 參數的年、月、日部分。此設定應符合管線 JSON 中的 **Start** 值。

	您應該會看到 24 個配量，從今天 12 AM 到隔天 12 AM，每小時各一個。
	
	**範例輸出：**

		ResourceGroupName : ADFTutorialResourceGroup
		DataFactoryName   : ADFTutorialDataFactoryPSH
		TableName         : EmpSQLTable
		Start             : 8/9/2016 12:00:00 AM
		End               : 8/9/2016 1:00:00 AM
		RetryCount        : 0
		Status            : Waiting
		LatencyStatus     :
		LongRetryCount    : 0

3.	執行 **Get-AzureRmDataFactoryRun**，以取得**特定**配量的活動執行詳細資料。變更 **StartDateTime** 參數的值，以符合輸出中配量的 **Start** 時間。**StartDateTime** 的值必須是 [ISO 格式](http://en.wikipedia.org/wiki/ISO_8601)。

		Get-AzureRmDataFactoryRun $df -DatasetName EmpSQLTable -StartDateTime 2016-08-09T00:00:00

	您應該會看到類似下列範例輸出的輸出：

		Id                  : 3404c187-c889-4f88-933b-2a2f5cd84e90_635614488000000000_635614524000000000_EmpSQLTable
		ResourceGroupName   : ADFTutorialResourceGroup
		DataFactoryName     : ADFTutorialDataFactoryPSH
		TableName           : EmpSQLTable
		ProcessingStartTime : 8/9/2016 11:03:28 PM
		ProcessingEndTime   : 8/9/2016 11:04:36 PM
		PercentComplete     : 100
		DataSliceStart      : 8/9/2016 10:00:00 PM
		DataSliceEnd        : 8/9/2016 11:00:00 PM
		Status              : Succeeded
		Timestamp           : 8/9/2016 11:03:28 PM
		RetryAttempt        : 0
		Properties          : {}
		ErrorMessage        :
		ActivityName        : CopyFromBlobToSQL
		PipelineName        : ADFTutorialPipeline
		Type                : Copy

如需 Data Factory Cmdlet 的完整文件，請參閱 [Data Factory Cmdlet 參考][cmdlet-reference]。

## 摘要
在本教學課程中，您已建立要將資料從 Azure Blob 複製到 Azure SQL 資料庫的 Azure Data Factory。您已使用 PowerShell 建立 Data Factory、連結服務、資料集和管線。以下是您在本教學課程中執行的高階步驟：

1.	建立 Azure **Data Factory**。
2.	建立**連結服務**：
	1. **Azure 儲存體**連結服務可連結保留輸入資料的 Azure 儲存體帳戶。
	2. **Azure SQL** 連結服務可連結保留輸出資料的 Azure SQL Database。
3.	建立可描述管線輸入資料和輸出資料的**資料集**。
4.	建立具有**複製活動**的**管線**，以 **BlobSource** 做為來源並以 **SqlSink** 做為接收器。

## 另請參閱
| 主題 | 說明 |
| :---- | :---- |
| [資料移動活動](data-factory-data-movement-activities.md) | 本文提供您在本教學課程中使用的複製活動詳細資訊。 |
| [排程和執行](data-factory-scheduling-and-execution.md) | 本文說明 Azure Data Factory 應用程式模型的排程和執行層面。 |
| [管線](data-factory-create-pipelines.md) | 本文協助您了解 Azure Data Factory 中的管線和活動。 |
| [資料集](data-factory-create-datasets.md) | 本文協助您了解 Azure Data Factory 中的資料集。
| [使用監視應用程式來監視和管理管線](data-factory-monitor-manage-app.md) | 本文說明如何使用監視及管理應用程式，來監視、管理管線及進行偵錯。 



[use-custom-activities]: data-factory-use-custom-activities.md
[troubleshoot]: data-factory-troubleshoot.md
[developer-reference]: http://go.microsoft.com/fwlink/?LinkId=516908

[cmdlet-reference]: https://msdn.microsoft.com/library/azure/dn820234.aspx
[old-cmdlet-reference]: https://msdn.microsoft.com/library/azure/dn820234(v=azure.98).aspx
[azure-free-trial]: http://azure.microsoft.com/pricing/free-trial/

[azure-portal]: http://portal.azure.com
[download-azure-powershell]: ../powershell-install-configure.md
[data-factory-introduction]: data-factory-introduction.md

[image-data-factory-get-started-storage-explorer]: ./media/data-factory-copy-activity-tutorial-using-powershell/getstarted-storage-explorer.png

[sql-management-studio]: ../sql-database/sql-database-manage-azure-ssms.md
 

<!---HONumber=AcomDC_0921_2016-->