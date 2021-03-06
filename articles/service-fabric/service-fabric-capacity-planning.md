<properties
   pageTitle="Service Fabric 應用程式的容量規劃 | Microsoft Azure"
   description="描述如何識別 Service Fabric 應用程式所需的計算節點的數目"
   services="service-fabric"
   documentationCenter=".net"
   authors="mani-ramaswamy"
   manager="coreysa"
   editor=""/>

<tags
   ms.service="service-fabric"
   ms.devlang="dotnet"
   ms.topic="article"
   ms.tgt_pltfrm="NA"
   ms.workload="NA"
   ms.date="05/18/2016"
   ms.author="subramar"/>


# Service Fabric 應用程式的容量規劃


本文將說明您如何估計執行 Azure Service Fabric 應用程式所需的資源量 (CPU、RAM、磁碟儲存空間)。資源需求隨著時間改變是很常見的。開發/測試您的服務時通常需要少數資源，之後進入生產環境且您的應用程式受歡迎度增長時會需要更多資源。設計您的應用程式時，最好立即考慮長期需求，並選擇現在可讓您輕鬆地調整以符合高度客戶需求的服務。

 建立 Service Fabric 叢集時，您會決定組成叢集的虛擬機器 (VM) 類型。每個 VM 會以 CPU (核心和速度)、網路頻寬、RAM 和磁碟儲存空間的形式隨附有限數量的資源。當您的服務隨著時間成長時，可以升級為可提供更大量資源的 VM 和/或加入更多 VM 至您的叢集。當然，若要執行後者，您一開始必須架構您的服務，使得它可以利用動態加入至叢集的新 VM。

部分服務本身可管理 VM 上的少數資料或皆不管理。因此，這些服務的容量規劃應該主要著重在效能。這表示您應該仔細考慮執行您的演算法所需的程式碼演算法和 VM CPU (核心和速度) 的效能。此外，您應該考慮網路頻寬，包括發生網路傳輸的頻率特別以及要傳送的資料量。如果隨著服務使用量增加時，您的服務需要執行良好，您可以加入更多 VM 到叢集，並跨所有 VM 將網路要求負載平衡。

針對管理 VM 上大量資料的服務，容量規劃應該主要著重在大小。這表示您應該仔細地考量 VM 的 RAM 和磁碟儲存空間容量。Windows 中的虛擬記憶體管理系統可讓應用程式程式碼將磁碟空間做為 RAM 使用。這可讓應用程式使用較 VM 上實際可用更多的記憶體。有更多 RAM 可完全增加效能，因為 VM 可以在 RAM 中保留更多磁碟儲存空間。選擇 VM 時，請選取有足夠的磁碟空間可以包含您要放在 VM 上所有資料的 VM，並選擇可讓您以想要的速度存取該資料的 RAM 數量。如果您的服務隨著時間成長，您可以加入更多 VM 到叢集，並跨所有 VM 分割資料。

## 決定您需要多少個節點

分割您的服務可讓您將服務的資料相應放大 (如需詳細資料分割，請參閱[分割 Service Fabric](service-fabric-concepts-partitioning.md))。必須將每個資料分割納入單一 VM，但是可以將多個 (小型) 資料分割放在單一 VM 上。因此，相較於有少量的較大型磁碟分割，有大量的小型資料分割可提供您更大的彈性。缺點是有很多資料分割會增加 Service Fabric 的負擔，並且您無法跨資料分割執行交易的作業。如果您的服務程式碼經常需要存取位於不同的資料分割的資料片段，也會產生更多潛在的網路流量。設計您的服務時，應該仔細考慮這些優缺點，以達成有效的資料分割策略。

假設您的應用程式有單一可設定狀態服務，其具有您預期一年會成長到 DB\_Size GB 的存放區大小。您願意在經歷該年的成長以外加入更多應用程式 (與資料分割)。若要尋找所有複本上的 DB\_Size 總計，我們也必須採用複寫因數 RF，它可判斷您的服務的複本數目 (所有複本上的 DB\_Size 總計為複寫因數乘以 DB\_Size)。Node\_Size 表示您想要用於服務的每一節點磁碟空間/RAM。為了獲得最佳效能，您會要 DB\_Size 跨叢集納入記憶體，並且要放入 Node\_Size，其大約等於您所選擇 VM 的 RAM 容量。藉由配置大於 RAM 容量的 Node\_Size，您可依賴作業系統分頁。因此，您的效能可能不是最佳效能，但可能仍夠您的服務使用。

為了獲得最大效能所需的節點數目，可以依以下方式計算：

```
Number of Nodes = (DB_Size * RF)/Node_Size

```


## 考慮成長

除了開始使用的 DB\_Size，您可能想要根據您預期服務成長的 DB\_Size 來計算節點。然後，隨著服務的成長而讓節點數目成長，使得您不會過度佈建節點數目但是，資料分割數目應該基於以最大成長執行您的服務時所需的節點數。

而且最好隨時能有幾個備用機器 (超額容量) 可用，使得您可以處理任何未預期的尖峰或基礎結構失敗 (例如，如果幾個 VM 關閉)。雖然這是您應使用您預期的尖峰判斷的項目，但保留一些額外的 VM (額外 5-10%) 是不錯的起點。

以上假設使用單一可設定狀態服務。如果您有一個以上可設定狀態服務，您也必須將與其他服務相關聯的 DB\_Size 加入方程式，或分別為每個可設定狀態服務計算節點數目。您的服務的複本或資料分割可能不平衡。某些資料分割的資料可能多於其他資料分割，所以請參考[分割文件中的最佳做法](service-fabric-concepts-partitioning.md)以取得詳細資訊。不過，上述的方程式不受資料分割或複本影響，因為 Service Fabric 將確保複本會以最佳化方式分散在節點之間。


## 使用試算表進行成本計算

現在，讓我們在上面所列的公式中放入一些實際數字。[範例試算表](https://servicefabricsdkstorage.blob.core.windows.net/publicrelease/SF%20VM%20Cost%20calculator-NEW.xlsx)示範如何規劃包含三種資料物件類型的應用程式的容量。針對每個物件，我們會估計其大小以及我們預計具有的物件數。我們也選取對每個物件類型我們想要的複本數。試算表會計算要在叢集中儲存的記憶體數量總計。

然後，我們輸入 VM 大小和每月成本。根據 VM 大小，試算表會告訴您必須用來將資料分割的最少分割數量，以實際上納入節點。您可能想要較大量的資料分割以配合您的應用程式特定的計算和網路流量需求。試算表顯示目前管理使用者設定檔物件的資料分割數目已從 1 增加到 6。

現在，根據這所有資訊，試算表會顯示您可以實際上取得在具有 26 個節點的叢集上所需的資料分割和複本的所有資料。不過，此叢集會密集壓縮，因此您可能會想要一些其他節點來容納節點失敗和升級。試算表也顯示具有超過 57 個節點不會提供任何額外的值，因為您會有空白節點。同樣地，您可能想要有超過 57 節點以容納節點失敗和升級。您可以調整試算表以符合應用程式的特定需求。

![成本計算的試算表][Image1]



## 後續步驟

查看[分割 Service Fabric 服務][10]來深入了解分割您的服務。



<!--Image references-->
[Image1]: ./media/SF-Cost.png

<!--Link references--In actual articles, you only need a single period before the slash-->
[10]: service-fabric-concepts-partitioning.md

<!---HONumber=AcomDC_0525_2016-->