<properties
   pageTitle="可靠的集合 | Microsoft Azure"
   description="Service Fabric 具狀態服務提供可靠的集合，可讓您撰寫高度可用、可調整且低延遲的雲端應用程式。"
   services="service-fabric"
   documentationCenter=".net"
   authors="mcoskun"
   manager="timlt"
   editor="masnider,vturecek"/>

<tags
   ms.service="service-fabric"
   ms.devlang="dotnet"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="required"
   ms.date="07/28/2016"
   ms.author="mcoskun"/>

# Azure Service Fabric 具狀態服務中可靠的集合簡介

可靠的集合可讓您撰寫高度可用、可擴充且低延遲的雲端應用程式，如同您在撰寫單一電腦應用程式一般。**Microsoft.ServiceFabric.Data.Collections** 命名空間中的類別提供一組現成的集合，可讓您自動具有高度可用的狀態。開發人員只需將程式設計為可靠的集合 API，並讓可靠的集合來管理複寫和本機狀態。

可靠的集合和其他高可用性技術 (例如 Redis、Azure 表格服務和 Azure 佇列服務) 之間的主要差異是狀態會保留在本機服務執行個體中，但同時也被設定為高可用性。這表示：

- 所有讀取都在本機，可保障低延遲及高輸送量讀取。
- 所有寫入都只會產生最少的網路 IO 數，可保障低延遲和高輸送量寫入。

![集合的演化圖。](media/service-fabric-reliable-services-reliable-collections/ReliableCollectionsEvolution.png)

我們可將可靠的集合視為 **System.Collections** 類別的自然進化：它們是一組新的集合，專為雲端和多電腦應用程式量身設計，且不會對開發人員提高複雜度。因此，可靠的集合是：

- 可複寫：進行狀態變更複寫以確保高可用性。
- 可保存：資料會保存至磁碟，可在發生大規模中斷 (例如，資料中心停電) 時保障持續性。
- 非同步：API 是非同步的，可確保在產生 IO 時不會封鎖執行緒。
- 交易式：API 會利用交易的抽象方法，讓您能夠輕鬆管理服務內多個可靠的集合。

可靠的集合具有現成的增強式一致性保證，可讓您更輕鬆地推論應用程式的狀態。增強式一致性的實現方式是藉由確保僅有在整個交易已記錄到複本多數仲裁之後 (包括主要複本)，才認可交易。若要達成較弱的一致性，應用程式可在非同步認可傳回之前，返回向用戶端/要求者進行確認。

可靠的集合 API 是並行集合 API (位於 **System.Collections.Concurrent** 命名空間) 的一種演化：

- 非同步：會傳回工作；不同於並行集合，其作業會受到複寫及保存。
- 沒有 out 參數：使用 `ConditionalValue<T>` 傳回 bool 和值，而不是 out 參數。`ConditionalValue<T>` 就像 `Nullable<T>`，但 T 可以不是結構。
- 交易：使用交易物件，讓使用者可在交易中的多個可靠的集合上群組動作。

現在，**Microsoft.ServiceFabric.Data.Collections** 包含兩個集合：

- [可靠的字典](https://msdn.microsoft.com/library/azure/dn971511.aspx)：表示可複寫、交易式和非同步索引鍵/值組的集合。類似於 **ConcurrentDictionary**，其索引鍵和值可以是任何類型。
- [可靠的佇列](https://msdn.microsoft.com/library/azure/dn971527.aspx)：表示可複寫、交易式和非同步的嚴格先進先出 (FIFO) 佇列。類似於 **ConcurrentQueue**，其值可以是任何類型。

## 隔離層級
隔離層級定義交易必須與其他交易所做的修改隔離的程度。可靠的集合支援兩種隔離等級：

- **可重複讀取**：指定陳述式無法讀取已經修改但尚未由其他交易確認的資料，以及指定在目前的交易完成之前，任何其他交易都不能修改已經由目前交易讀取的資料。如需詳細資訊，請參閱 [https://msdn.microsoft.com/library/ms173763.aspx](https://msdn.microsoft.com/library/ms173763.aspx)。
- **快照集**：指定交易中任何陳述式所讀取的資料都會與交易開始時就存在的資料版本一致。交易只能辨識交易開始之前所認可的資料修改。在目前交易中執行的陳述式無法看到在目前交易開始之後，其他交易所進行的資料修改。效果就如同交易中的陳述式會取得認可資料的快照集，因為這項資料於交易開始時就存在。可靠的集合的快照集都是一致的。如需詳細資訊，請參閱 [https://msdn.microsoft.com/library/ms173763.aspx](https://msdn.microsoft.com/library/ms173763.aspx)。

可靠的集合會依據交易建立時的作業和複本角色，自動選擇要用於指定讀取作業的隔離層級。下表說明可靠的字典和佇列作業的隔離等級預設值。

| 作業 \\ 角色 | 主要 | 次要 |
| --------------------- | :--------------- | :--------------- |
| 單一實體讀取 | 可重複讀取 | 快照 |
| 列舉 \\ 計數 | 快照 | 快照 |

>[AZURE.NOTE] 單一實體作業的常見範例包括 `IReliableDictionary.TryGetValueAsync`、`IReliableQueue.TryPeekAsync`。

可靠的字典和可靠的佇列皆支援「讀寫一致性」(Read Your Writes)。換句話說，在屬於相同交易的下列讀取作業皆可看到交易內的任何寫入作業。

## 鎖定
在可靠的集合中，所有交易都有兩個階段：在以中止或認可來終止交易之前，交易不會釋放它所取得的鎖定。

可靠的字典會針對所有單一實體作業使用資料列層級鎖定。可靠的佇列則會針對嚴格交易的 FIFO 屬性交換並行。可靠的佇列會使用作業層級的鎖定，允許一次有 `TryPeekAsync` 和/或 `TryDequeueAsync` 的一個交易，以及有 `EnqueueAsync` 的一個交易。請注意，若要保留 FIFO，如果 `TryPeekAsync` 或 `TryDequeueAsync` 曾觀察到可靠的佇列是空的，則它們也會鎖定 `EnqueueAsync`。

寫入作業一律會採取「獨佔」鎖定。讀取作業的鎖定則取決於一些因素。任何使用快照隔離所完成的讀取作業都是無鎖定的。任何可重複讀取作業預設都會採用共用鎖定。不過，針對支援可重複讀取的任何讀取作業，使用者可以要求更新鎖定，而不是共用鎖定。更新鎖定是一種非對稱式鎖定，用來避免常見的死結，這些死結通常會在多個交易鎖定資源稍後進行可能更新時發生。

下列為鎖定相容性矩陣：

| 要求 \\ 授與 | None | 共用 | 更新 | 獨佔 |
| ----------------- | :----------- | :----------- | :---------- | :----------- |
| 共用 | 無衝突 | 無衝突 | 衝突 | 衝突 |
| 更新 | 無衝突 | 無衝突 | 衝突 | 衝突 |
| 獨佔 | 無衝突 | 衝突 | 衝突 | 衝突 |

請注意，可靠的集合 API 中的逾時引數是用來進行死結偵測。例如，有兩筆交易 (T1 和 T2) 嘗試讀取和更新 K1。這樣很可能會形成死結，因為它們最後都會有共用鎖定。在這種情況下，其中一個或兩個作業將會逾時。

請注意，上述的死結案例就是更新鎖定可如何防止死結的絕佳範例。

## 持續性模型
可靠的狀態管理員和可靠的集合會遵循持續性模型，該模型稱為「記錄檔和檢查點」(Log and Checkpoint)。這個模型會將每個狀態變更記錄在磁碟上，並且只在記憶體中套用。完成狀態本身也只是偶爾保存 (也稱為檢查點)。好處是差異會轉變成磁碟上的只開頭附加循序寫入以改善效能。

為了進一步了解「記錄檔和檢查點」模型，我們來看看無限磁碟的案例。可靠的狀態管理員會在複寫作業之前，記錄每一項作業。這可讓可靠的集合只在記憶體中套用作業。由於記錄檔會保存，即使複本失敗而必須重新啟動，可靠的狀態管理員在其記錄檔中仍具有足夠的資訊，以重新執行複本已遺失的所有作業。由於磁碟是無限的，因此永遠都不需要移除記錄檔記錄，而可靠的集合就只需要管理記憶體內部的狀態。

現在讓我們看看磁碟有限的案例。隨著記錄檔一直累積記錄，可靠的狀態管理員會用完磁碟空間。在這種情況發生前，可靠的狀態管理員必須截斷其記錄檔，以騰出空間給較新的記錄。它會要求可靠的集合對磁碟進行其記憶體內部狀態的檢查點檢查。由可靠的集合負責保存到該點為止的狀態。一旦可靠的集合完成其檢查點檢查，可靠的狀態管理員即可截斷記錄檔以釋放磁碟空間。如此一來，當複本必須重新啟動時，可靠的集合將會復原其檢查點的狀態，而可靠的狀態管理員則會復原並播放在該檢查點之後發生的所有狀態變更。

>[AZURE.NOTE] 檢查點加入另一個值可改善一般情況下的復原效能。這是因為檢查點只包含最新的版本。

## 建議

- 請勿修改讀取作業 (例如 `TryPeekAsync` 或 `TryGetValueAsync`) 所傳回的自訂類型物件。可靠的集合就像並行的集合一樣，會傳回物件參考而不是複本。
- 請不要在未經修改之前，就深層複製傳回的自訂類型物件。因為結構和內建類型都是傳值，因此您不需要在其上執行深層複製。
- 請不要針對逾時使用 `TimeSpan.MaxValue`。逾時應該用來偵測死結。
- 在認可、中止或處置交易之後，請勿使用該交易。
- 請不要在其所建立的交易範圍之外使用列舉。
- 請不要在另一個交易的 `using` 陳述式內建立交易，因為它會造成死結。
- 務必確保 `IComparable<TKey>` 實作是正確的。系統會對此採取相依性以合併檢查點。
- 請不要在讀取需更新的項目時使用更新鎖定，以避免發生特定類別的死結。
- 請考慮使用備份和還原功能以擁有災害復原。
- 避免在同一個交易中因為不同的隔離等級混用單一實體作業和多實體作業 (例如 `GetCountAsync`、`CreateEnumerableAsync`)。

以下是要牢記在心的一些事項：

- 所有可靠的集合 API 的預設逾時都是 4 秒。大部分使用者應該都不會覆寫此預設值。
- 在所有可靠的集合 API 中，預設的取消權杖為 `CancellationToken.None`。
- 可靠字典的索引鍵類型參數 (*TKey*) 必須正確實作 `GetHashCode()` 和 `Equals()`。索引鍵必須是不可變的。
- 若要讓可靠的集合達到高度可用性，每個服務應至少有一個目標和大小為 3 的最小複本集。
- 次要複本上的讀取作業可能會讀取未認可的仲裁版本。這表示從單一次要複本讀取的資料版本進度可能有誤。當然，從主要複本讀取一定最穩定︰進度一定不會有誤。

## 後續步驟

- [Reliable Services 快速入門](service-fabric-reliable-services-quick-start.md)
- [使用可靠的集合](service-fabric-work-with-reliable-collections.md)
- [Reliable Services 通知](service-fabric-reliable-services-notifications.md)
- [備份與還原 Reliable Services (災害復原)](service-fabric-reliable-services-backup-restore.md)
- [Reliable State Manager 組態](service-fabric-reliable-services-configuration.md)
- [開始使用 Service Fabric Web API 服務](service-fabric-reliable-services-communication-webapi.md)
- [Reliable Services 程式設計模型進階用法](service-fabric-reliable-services-advanced-usage.md)
- [可靠的集合的開發人員參考資料](https://msdn.microsoft.com/library/azure/microsoft.servicefabric.data.collections.aspx)

<!---HONumber=AcomDC_0803_2016-->