<properties
   pageTitle="使用傳統部署模型和 PowerShell 建立和修改 ExpressRoute 線路| Microsoft Azure"
   description="本文將逐步引導您建立和佈建 ExpressRoute 線路。本文也會示範如何檢查狀態、更新或刪除和取消佈建線路。"
   documentationCenter="na"
   services="expressroute"
   authors="ganesr"
   manager="carmonm"
   editor=""
   tags="azure-service-management"/>
<tags
   ms.service="expressroute"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="infrastructure-services"
   ms.date="09/14/2016"
   ms.author="ganesr;cherylmc"/>

# 建立和修改 ExpressRoute 線路

> [AZURE.SELECTOR]
[Azure Portal - Resource Manager](expressroute-howto-circuit-portal-resource-manager.md)
[PowerShell - Resource Manager](expressroute-howto-circuit-arm.md)
[PowerShell - Classic](expressroute-howto-circuit-classic.md)

本文將逐步引導您使用 PowerShell Cmdlet 和傳統部署模型建立 Azure ExpressRoute 線路。本文也會示範如何檢查狀態、更新或刪除和取消佈建 ExpressRoute 循環。

**關於 Azure 部署模型**

[AZURE.INCLUDE [vpn-gateway-clasic-rm](../../includes/vpn-gateway-classic-rm-include.md)]


## 開始之前

### 1\.檢閱必要條件和工作流程文章

開始設定之前，請確定您已經檢閱過[必要條件](expressroute-prerequisites.md)和[工作流程](expressroute-workflows.md)。


### 2\.安裝最新版的 Azure PowerShell 模組 

請遵循[如何安裝和設定 Azure PowerShell](../powershell-install-configure.md) 中的指示，取得如何設定您的電腦以使用 Azure PowerShell 模組的逐步指引。

### 3\.登入您的 Azure 帳戶並選取訂用帳戶

1. 使用提升權限的 Windows PowerShell 命令提示字元，執行下列 Cmdlet：

		Add-AzureAccount
2. 在出現的登入畫面中，登入您的帳戶。

3. 取得您的訂用帳戶清單。

		Get-AzureSubscription
4. 選取您要使用的訂用帳戶。
	
		Select-AzureSubscription -SubscriptionName "mysubscriptionname"

## 建立和佈建 ExpressRoute 線路

### 1\.匯入 ExpressRoute 的 PowerShell 模組

 如果您尚未執行此動作，您必須將 Azure 和 ExpressRoute 模組匯入 PowerShell 工作階段，才能開始使用 ExpressRoute Cmdlet。您要將模組從其安裝所在的位置匯入您的本機電腦。根據您用來安裝模組的方法而定，位置可能與下列範例所示的不同。如有必要，請修改範例。

	Import-Module 'C:\Program Files (x86)\Microsoft SDKs\Azure\PowerShell\ServiceManagement\Azure\Azure.psd1'
	Import-Module 'C:\Program Files (x86)\Microsoft SDKs\Azure\PowerShell\ServiceManagement\Azure\ExpressRoute\ExpressRoute.psd1'

### 2\.取得支援的提供者、位置和頻寬清單

建立 ExpressRoute 線路之前，您需要有支援的連線提供者、位置和頻寬選項的清單。

PowerShell Cmdlet `Get-AzureDedicatedCircuitServiceProvider` 會傳回此資訊，在稍後的步驟中將會用到：

	Get-AzureDedicatedCircuitServiceProvider

請檢查是否列出您的連線服務提供者。記下下列資訊，因為您稍後在建立線路時將會用到：

- Name

- PeeringLocations

- BandwidthsOffered

您現在已經準備好建立 ExpressRoute 線路。

### 3\.建立 ExpressRoute 線路

以下示範如何透過矽谷的 Equinix 建立 200-Mbps ExpressRoute 線路。如果您使用不同的提供者和不同的設定，請在您提出要求時替換成該資訊。

>[AZURE.IMPORTANT] ExpressRoute 循環將會從發出服務金鑰時開始收費。請確定在連線提供者準備好佈建線路之後，再執行這項作業。


下列是新服務金鑰的要求範例：

	$Bandwidth = 200
	$CircuitName = "MyTestCircuit"
	$ServiceProvider = "Equinix"
	$Location = "Silicon Valley"

	New-AzureDedicatedCircuit -CircuitName $CircuitName -ServiceProviderName $ServiceProvider -Bandwidth $Bandwidth -Location $Location -sku Standard -BillingType MeteredData

或者，如果您想要使用進階附加元件建立 ExpressRoute 線路，請使用下列範例。如需進階附加元件的詳細資訊，請參閱 [ExpressRoute 常見問題集](expressroute-faqs.md)。

	New-AzureDedicatedCircuit -CircuitName $CircuitName -ServiceProviderName $ServiceProvider -Bandwidth $Bandwidth -Location $Location -sku Premium - BillingType MeteredData


回應會包含服務金鑰。您可以執行下列命令來取得所有參數的詳細描述：

	get-help new-azurededicatedcircuit -detailed

### 4\.列出所有 ExpressRoute 線路

您可以執行 `Get-AzureDedicatedCircuit` 命令，以取得您建立的所有 ExpressRoute 線路的清單︰


	Get-AzureDedicatedCircuit

回應將如以下範例所示：

	Bandwidth                        : 200
	CircuitName                      : MyTestCircuit
	Location                         : Silicon Valley
	ServiceKey                       : *********************************
	ServiceProviderName              : equinix
	ServiceProviderProvisioningState : NotProvisioned
	Sku                              : Standard
	Status                           : Enabled

您隨時可以使用 `Get-AzureDedicatedCircuit` Cmdlet 擷取此資訊。執行呼叫時，若未指定任何參數，將會列出所有線路。[服務金鑰] 欄位會列出您的服務金鑰。

	Get-AzureDedicatedCircuit

	Bandwidth                        : 200
	CircuitName                      : MyTestCircuit
	Location                         : Silicon Valley
	ServiceKey                       : *********************************
	ServiceProviderName              : equinix
	ServiceProviderProvisioningState : NotProvisioned
	Sku                              : Standard
	Status                           : Enabled

您可以執行下列命令來取得所有參數的詳細描述：

	get-help get-azurededicatedcircuit -detailed

### 5\.將服務金鑰傳送給連線提供者以進行佈建


[服務提供者佈建狀態] 提供服務提供者端目前的佈建狀態相關資訊。[狀態] 提供 Microsoft 端的狀態。如需線路佈建狀態的詳細資訊，請參閱[工作流程](expressroute-workflows.md#expressroute-circuit-provisioning-states)文章。

當您建立新的 ExpressRoute 線路時，線路會是下列狀態：

	ServiceProviderProvisioningState : NotProvisioned
	Status                           : Enabled


當連線提供者正在為您啟用線路時，線路會處於下列狀態：

	ServiceProviderProvisioningState : Provisioning
	Status                           : Enabled

ExpressRoute 線路必須處於下列狀態，才可供您使用。

	ServiceProviderProvisioningState : Provisioned
	Status                           : Enabled


### 6\.定期檢查線路金鑰的情況和狀態

這樣可讓您知道提供者何時已啟用您的線路。設定線路之後，[服務提供者佈建狀態] 會顯示為 [已佈建]，如下列範例所示：

	Get-AzureDedicatedCircuit

	Bandwidth                        : 200
	CircuitName                      : MyTestCircuit
	Location                         : Silicon Valley
	ServiceKey                       : *********************************
	ServiceProviderName              : equinix
	ServiceProviderProvisioningState : Provisioned
	Sku                              : Standard
	Status                           : Enabled

### 7\.建立路由組態

請參閱 [ExpressRoute 線路路由設定 (建立和修改線路對等)](expressroute-howto-routing-classic.md) 一文，取得逐步指示。

>[AZURE.IMPORTANT] 這些指示只適用於由提供第 2 層連線服務的服務提供者所建立的線路。如果您使用的服務提供者是提供受管理的第 3 層服務 (通常是 IP VPN，如 MPLS)，您的連線提供者會為您設定和管理路由。

### 8\.將虛擬網路連結到 ExpressRoute 電路

接下來，將虛擬網路連結到 ExpressRoute 線路。如需逐步指示，請參閱[將 ExpressRoute 線路連結至虛擬網路](expressroute-howto-linkvnet-classic.md)。如果您需要使用傳統部署模型為 ExpressRoute 建立虛擬網路，請參閱[建立 ExpressRoute 的虛擬網路](expressroute-howto-vnet-portal-classic.md)中的相關指示。

## 取得 ExpressRoute 線路的狀態

您隨時可以使用 `Get-AzureCircuit` Cmdlet 擷取此資訊。執行呼叫時，若未指定任何參數，將會列出所有線路。

	Get-AzureDedicatedCircuit

	Bandwidth                        : 200
	CircuitName                      : MyTestCircuit
	Location                         : Silicon Valley
	ServiceKey                       : *********************************
	ServiceProviderName              : equinix
	ServiceProviderProvisioningState : Provisioned
	Sku                              : Standard
	Status                           : Enabled

	Bandwidth                        : 1000
	CircuitName                      : MyAsiaCircuit
	Location                         : Singapore
	ServiceKey                       : #################################
	ServiceProviderName              : equinix
	ServiceProviderProvisioningState : Provisioned
	Sku                              : Standard
	Status                           : Enabled

您可以將服務金鑰當作參數傳遞給呼叫，以取得特定 ExpressRoute 線路的詳細資訊。

	Get-AzureDedicatedCircuit -ServiceKey "*********************************"

	Bandwidth                        : 200
	CircuitName                      : MyTestCircuit
	Location                         : Silicon Valley
	ServiceKey                       : *********************************
	ServiceProviderName              : equinix
	ServiceProviderProvisioningState : Provisioned
	Sku                              : Standard
	Status                           : Enabled


您可以執行下列命令來取得所有參數的詳細描述：

	get-help get-azurededicatedcircuit -detailed

## 修改 ExpressRoute 線路

您可以修改 ExpressRoute 線路的某些屬性，而不會影響連線。

您可以執行下列工作，而無需中途停機：

- 啟用或停用 ExpressRoute 線路的 ExpressRoute 進階附加元件。
- 增加 ExpressRoute 線路的頻寬。請注意，不支援將循環的頻寬降級。
- 將計量方案從 [已計量資料] 變更為 [無限制資料]。請注意，不支援將計量方案從 [無限制資料] 變更為 [已計量資料]。
- 您可以啟用和停用 [允許傳統作業]。

如需限制的詳細資訊，請參閱 [ExpressRoute 常見問題集](expressroute-faqs.md)。

### 啟用 ExpressRoute 進階附加元件

您可以使用下列 PowerShell Cmdlet，為現有的線路啟用 ExpressRoute Premium 附加元件：

	Set-AzureDedicatedCircuitProperties -ServiceKey "*********************************" -Sku Premium

	Bandwidth                        : 1000
	CircuitName                      : TestCircuit
	Location                         : Silicon Valley
	ServiceKey                       : *********************************
	ServiceProviderName              : equinix
	ServiceProviderProvisioningState : Provisioned
	Sku                              : Premium
	Status                           : Enabled

您的線路現在將啟用 ExpressRoute Premium 附加功能。請注意，順利執行命令後，我們會開始向您收取進階附加元件功能的費用。

### 停用 ExpressRoute 進階附加元件

>[AZURE.IMPORTANT] 如果您使用的資源超出標準線路所允許的數量，這項作業可能會失敗。

請注意：

- 從高階降級為標準之前，您必須確定連結至線路的虛擬網路數目小於 10。如果您沒有這麼做，則更新要求將會失敗，將會以高階費率向您收費。

- 您必須取消連結其他地理政治區域中的所有虛擬網路。如果您沒有這麼做，則更新要求將會失敗，將會以高階費率向您收費。

- 就私用對等設定而言，路由表必須少於 4000 個路由。如果路由表大小超過 4000 個路由，BGP 工作階段將會中斷，而且在通告的首碼數目降到 4000 以下之前不會重新啟用。

您可以使用下列 PowerShell Cmdlet，為現有的線路停用 ExpressRoute Premium 附加元件：

	Set-AzureDedicatedCircuitProperties -ServiceKey "*********************************" -Sku Standard

	Bandwidth                        : 1000
	CircuitName                      : TestCircuit
	Location                         : Silicon Valley
	ServiceKey                       : *********************************
	ServiceProviderName              : equinix
	ServiceProviderProvisioningState : Provisioned
	Sku                              : Standard
	Status                           : Enabled



### 更新 ExpressRoute 線路頻寬

請查閱 [ExpressRoute 常見問題集](expressroute-faqs.md)，以取得提供者支援的頻寬選項。您可以挑選任何比現有線路規模還大的大小。

>[AZURE.IMPORTANT] 降低 ExpressRoute 線路的頻寬時必須中斷運作。頻寬降級需要取消佈建 ExpressRoute 線路，然後重新佈建新的 ExpressRoute 線路。

一旦決定需要的大小後，您可以使用下列命令來調整線路大小。

	Set-AzureDedicatedCircuitProperties -ServiceKey ********************************* -Bandwidth 1000

	Bandwidth                        : 1000
	CircuitName                      : TestCircuit
	Location                         : Silicon Valley
	ServiceKey                       : *********************************
	ServiceProviderName              : equinix
	ServiceProviderProvisioningState : Provisioned
	Sku                              : Standard
	Status                           : Enabled

我們會在 Microsoft 端調整電路的大小。您必須連絡連線提供者，將他們那邊的設定更新為符合這項變更。請注意，從這個階段起，我們將開始計算更新頻寬選項的費用。

## 取消佈建和刪除 ExpressRoute 循環

請注意：

- 您必須取消連結 ExpressRoute 線路的所有虛擬網路，此作業才會成功。如果此作業失敗，請檢查您是否有任何虛擬網路連結至線路。

- 如果 ExpressRoute 循環服務提供者佈建狀態為 [Provisioning] 或 [Provisioned]，您就必須與服務提供者一起合作，取消佈建他們那邊的循環。我們將繼續保留資源並向您收取費用，直到線路服務提供者完成取消佈建並通知我們。

- 若服務提供者已取消佈建循環 (服務提供者佈建狀態設定為 [NotProvisioned])，則您可以刪除循環。這樣將會停止針對循環計費。

您可以執行下列命令來刪除 ExpressRoute 線路：

	Remove-AzureDedicatedCircuit -ServiceKey "*********************************"



## 後續步驟

建立好線路後，請務必執行下列作業：

- [建立和修改 ExpressRoute 線路的路由](expressroute-howto-routing-classic.md)
- [將虛擬網路連結至 ExpressRoute 線路](expressroute-howto-linkvnet-classic.md)

<!---HONumber=AcomDC_0914_2016-->