<properties
   pageTitle="流量管理員端點類型 | Microsoft Azure"
   description="本文說明可搭配 Azure 流量管理員使用的各類型端點"
   services="traffic-manager"
   documentationCenter=""
   authors="sdwheeler"
   manager="carmonm"
   editor="tysonn" />
<tags
   ms.service="traffic-manager"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="infrastructure-services"
   ms.date="06/08/2016"
   ms.author="sewhee" />

# 流量管理員端點

Microsoft Azure 流量管理員可讓您控制使用者流量如何分散到在世界各地不同的資料中心或其他位置執行的應用程式部署。

每個應用程式部署都必須設定為流量管理員中的 [端點]。當流量管理員收到 DNS 要求時，它會根據目前的端點可用性和所選的流量路由方法，選擇在 DNS 回應中傳回其中一個端點。如需詳細資訊，請參閱[流量管理員的運作方式](traffic-manager-how-traffic-manager-works.md)。

此頁面描述流量管理員如何支援不同類型的端點。

## 流量管理員端點類型

流量管理員支援三種類型的端點：

- **Azure 端點**用於在 Azure 中裝載的服務。
- **外部端點**用於 Azure 外部裝載的服務 (在內部部署或使用不同的主機服務提供者)。
- **巢狀端點**用於合併流量管理員端點，以建立更有彈性的流量路由配置，進而支援更大型且更複雜部署的需求。

在單一流量管理員設定檔中結合不同類型的端點的方式不受限。每個設定檔可以包含任意混合的端點類型。

下列各節會更深入說明每個端點類型。

### Azure 端點

Azure 端點在流量管理員中用來設定以 Azure 為基礎的服務。Azure 端點目前支援下列 Azure 資源類型：

- 「傳統」IaaS VM 和 PaaS 雲端服務。
- Web 應用程式
- PublicIPAddress 資源 (可以直接或透過 Azure Load Balancer 連接至 VM)。要注意的是，publicIpAddress 必須已獲指派 DNS 名稱，才能在流量管理員中使用。

PublicIPAddress 資源是 Azure Resource Manager 資源，它們不存在於 Azure 服務管理 API 中。因此，僅在流量管理員的 Azure Resource Manager 經驗中支援。其他端點類型會透過流量管理員中的 Resource Manager 和服務管理體驗來支援。

使用 Azure 端點時，流量管理員將會偵測「傳統」IaaS VM 或 PaaS 雲端服務或 Web 應用程式何時停止和啟動。這會反映於端點狀態中 (如需詳細資料，請參閱[流量管理員端點監視](traffic-manager-monitoring.md#endpoint-and-profile-status))。 基礎服務停止時，流量管理員不會再收取端點健康情況檢查的費用，而且不會將流量導向端點；重新啟動服務時，會繼續計費，而且端點便能夠接收流量。這並不適用於 PublicIpAddress 端點。

### 外部端點

外部端點用來設定流量管理員以將流量直接導向外部 Azure 服務，例如內部部署裝載的服務或透過不同提供者裝載的服務。

外部端點可用於其本身，或與相同流量管理員設定檔中的 Azure 端點結合。結合 Azure 端點與外部端點可造就不同的案例：

- 使用 Azure 為現有的內部部署應用程式提供提升的備援性 (以主動-主動或主動-被動容錯移轉模式)。
- 使用 Azure 將現有的內部部署應用程式擴充至其他地理位置，並搭配[流量管理員「效能」流量路由](traffic-manager-routing-methods.md#performance-traffic-routing-method)，以減少應用程式延遲並改善使用者的效能。
- 使用 Azure 為現有的內部部署應用程式提供額外容量，以持續不斷或「高載至雲端」的方式來處理突然增加的需求。

在某些情況下，使用外部端點參考 Azure 服務很實用 (如需範例，請參閱[常見問題集](#faq))。在此情況下，健康情況檢查是以 Azure 端點費率 (而不是外部端點費率) 計費。但是，不同於 Azure 端點，如果您停止或刪除基礎服務，您將繼續支付對應健康情況檢查的費用，直到您明確停用或刪除流量管理員中的端點為止。

### 巢狀端點

巢狀端點用於合併流量管理員端點，以建立更有彈性的流量路由配置，進而支援更大型且更複雜部署的需求。

使用巢狀端點，「子系」設定檔會當作端點加入至「父系」設定檔，以建立階層。子系和父系設定檔都可以包含任何類型的其他端點，包括其他巢狀設定檔。

如需進一步詳細資訊，請參閱[巢狀流量管理員設定檔](traffic-manager-nested-profiles.md)。

## Web Apps 做為端點

將 Web Apps 設定為流量管理員中的端點時，適用其他一些考量：

1. 只有「標準」SKU 或更高的 Web Apps 能夠搭配流量管理員使用。呼叫流量管理員來新增較低 SKU 的 Web 應用程式將會失敗。將現有 Web 應用程式的 SKU 降級，會導致流量管理員不再將流量傳送到該 Web 應用程式，而端點可能會從流量管理員中移除。

2. 當 Web Apps 服務收到的 HTTP 要求時，它會使用要求中的「主機」標頭來判斷服務應該要求哪個 Web 應用程式。主機標頭包含用來起始要求的 DNS 名稱，例如 'contosoapp.azurewebsites.net'。若要將不同的 DNS 名稱用於 Web 應用程式，DNS 名稱必須註冊為該應用程式的自訂網域。將 Web 應用程式端點當作 Azure 端點加入至流量管理員設定檔時，流量管理員設定檔 DNS 名稱會自動註冊為該應用程式的自訂網域。刪除端點時，會自動移除此註冊。

3. 一般而言，Web 應用程式會設定為 Azure 端點。不過，在某些情況下，使用外部端點來設定 Web 應用程式很有用 (如需範例，請參閱下一個項目)。使用 Resource Manager 的流量管理員體驗時，Web Apps 只能設定為流量管理員中的外部端點，而透過服務管理體驗不支援此做法。

4. 每個流量管理員設定檔最多可以有來自每個 Azure 區域的一個 Web 應用程式端點。此條件約束的因應措施已列入[常見問題集](#faq)中。

## 啟用和停用端點

在流量管理員中停用端點，對於暫時從處於維護模式或被重新部署的端點移除流量而言非常有用。端點一旦再次啟動並執行，即可予以重新啟用。

透過流量管理員入口網站、PowerShell、CLI 或 REST API 可以啟用及停用端點，而 Resource Manager 和服務管理體驗中均可支援上述各項。

>[AZURE.NOTE] 停用 Azure 端點會與其在 Azure 中的部署狀態無關。如果流量直接送至該服務，而不是透過流量管理員設定檔 DNS 名稱，則Azure 服務 (例如 VM 或 Web 應用程式) 會繼續執行，而即使在流量管理員中停用，也能夠接收流量。如需詳細資訊，請參閱[流量管理員的運作方式](traffic-manager-how-traffic-manager-works.md)。

停用端點後，便不再於 DNS 回應中傳回，因此沒有流量導向至端點。此外，端點的健康情況檢查會停止，而且不再計費。停用端點等同於刪除端點，不同之處在於再次重新啟用較為容易。

每個端點目前接收流量的適用性取決於設定檔狀態 (啟用/停用)、端點狀態 (啟用/停用)，以及該端點的健康情況檢查結果。如需詳細資訊，請參閱[流量管理員端點監視](traffic-manager-monitoring.md#endpoint-and-profile-status)。

>[AZURE.NOTE] 由於流量管理員是在 DNS 層級上運作，所以無法影響任何端點的現有連線。在引導端點之間的流量時 (透過變更設定檔設定，或是在容錯移轉或容錯回復期間)，流量管理員會將新的連接導向至可用的端點。不過，其他端點可透過現有的連接來繼續接收流量，直到這些工作階段終止為止。若要將現有連接的流量排出，應用程式應該限制每個端點使用的工作階段持續時間。

如果設定檔中的所有端點都已停用，或設定檔本身已停用，則 DNS 查詢會符合 'NXDOMAIN' 回應。這就如同設定檔已遭刪除。

## 常見問題集

### 可以將流量管理員用於來自多個訂用帳戶的端點嗎？
就 Azure Web Apps 而言，無法這麼做。這是因為 Web Apps 要求任何搭配 Web Apps 使用的自訂網域名稱都只能在單一訂用帳戶內使用。無法將來自多個訂用帳戶的 Web Apps 與相同的網域名稱搭配使用，因此無法將它們與「流量管理員」搭配使用。

針對其他端點類型，則可以將「流量管理員」與來自多個訂用帳戶的端點搭配使用。做法取決於您將服務管理 API 或 Resource Manager API 用於流量管理員。[Azure 入口網站](https://portal.azure.com)使用 Resource Manager，[「傳統」入口網站](https://manage.windowsazure.com)則使用「服務管理」。

在 Resource Manager 中，來自任何訂用帳戶的端點都可以新增至流量管理員，只要設定流量管理員設定檔的人員具有端點的讀取權限即可。使用 [Azure Resource Manager 角色型存取控制 (RBAC)](../active-directory/role-based-access-control-configure.md) 可以授與這些權限。

在服務管理中，流量管理員要求任何設為 Azure 端點的雲端服務或 Web 應用程式，需位於與流量管理員設定檔相同的訂用帳戶中。其他訂用帳戶中的雲端服務端點可以當作「外部」端點新增至流量管理員 (但仍以「內部」端點費率計費)。

### 可以使用流量管理員搭配雲端服務「預備」位置嗎？
是。雲端服務「預備」位置可在流量管理員中設定為外部端點。

因為此服務由 Azure 中的端點參考，健康情況檢查仍會以用於 Azure 端點的費率收費。

因為外部端點類型正在使用中，所以不會自動挑選基礎服務的變更。因此，如果雲端服務已停止或刪除，則流量管理員端點將會繼續支付健康情況檢查的費用，直到流量管理員內的端點停用或刪除為止。

### 流量管理員是否支援 IPv6 端點？

是。雖然流量管理員目前不提供 IPv6 可定址的名稱伺服器，但 IPv6 用戶端仍可用來連線到 IPv6 端點。

僅支援 IPv6 的用戶端仍可使用流量管理員，因為用戶端不會直接對流量管理員提出 DNS 要求，而是使用遞迴 DNS 服務。它可以透過 IPv6 對此服務提出要求，而遞迴服務應該就能夠連絡使用 IPv4 的流量管理員名稱伺服器。

收到 DNS 查詢後，流量管理員會以用戶端應連接端點的 DNS 名稱回應。只要設定一筆 DNS AAAA 記錄來將端點 DNS 名稱指向 IPv6 位址，即可支援 IPv6 端點。

請注意，為了讓流量管理員健康情況檢查運作正常，服務也需要公開 IPv4 端點。這需使用 DNS A 記錄，從相同的端點 DNS 名稱進行對應。

### 可以使用流量管理員搭配相同區域中的多個 Web 應用程式嗎？

通常，流量管理員用於將流量導向不同區域中部署的應用程式。不過，也可用於應用程式在相同區域中有多個部署時。

若為 Web Apps，Azure 流量管理員端點不允許將相同 Azure 區域中的多個 Web 應用程式端點新增至相同的流量管理員設定檔。下列步驟提供此條件約束的因應措施：

1.	檢查相同區域內的 Web Apps 是否位於不同的 Web 應用程式「縮放單位」，也就是不同的 Web 應用程式服務執行個體。若要這樣做，請檢查 <...>.azurewebsites.net DNS 項目的 DNS 路徑，縮放單位看起來應該像 ‘waws-prod-xyz-123.vip.azurewebsites.net’。指定的網域名稱必須對應到指定的縮放單位中的單一網站，因此相同縮放單位中的兩個 Web Apps 無法共用一個流量管理員設定檔。
2.	假設每個 Web 應用程式位於不同的縮放單位中，將虛名網域名稱新增為每個 Web 應用程式的自訂主機名稱。所有 Web Apps 必須屬於相同的訂用帳戶。
3.	如往常一樣，將一個 (且唯一一個) Web 應用程式端點新增至流量管理員設定檔，成為 Azure 端點。
4.	將每個額外的 Web 應用程式端點新增至流量管理員設定檔，成為外部端點。您需要將 Resource Manager 經驗 (而非服務管理經驗) 用於流量管理員。
5.	建立從虛名網域 (如上述步驟 2 所用) 至流量管理員設定檔 DNS 名稱 (<…>.trafficmanager.net) 的 DNS CNAME 記錄。
6.	透過虛名網域名稱 (而不是流量管理員設定檔 DNS 名稱) 存取您的網站。

## 後續步驟

- 了解[流量管理員的運作方式](traffic-manager-how-traffic-manager-works.md)。

- 了解「流量管理員」的[端點監視和自動容錯移轉](traffic-manager-monitoring.md)。

- 了解「流量管理員」的[流量路由方法](traffic-manager-routing-methods.md)。

<!---HONumber=AcomDC_0824_2016-->