<properties
	pageTitle="在訂用帳戶移動之後變更金鑰保存庫租用戶識別碼 | Microsoft Azure"
	description="了解如何在訂用帳戶移到不同租用戶之後，切換金鑰保存庫的租用戶識別碼"
	services="key-vault"
	documentationCenter=""
	authors="amitbapat"
	manager="mbaldwin"
	tags="azure-resource-manager"/>

<tags
	ms.service="key-vault"
	ms.workload="identity"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="hero-article"
	ms.date="09/13/2016"
	ms.author="ambapat"/>

# 在訂用帳戶移動之後變更金鑰保存庫租用戶識別碼
### 問︰我的訂用帳戶已從租用戶 A 移到租用戶 B。如何變更現有金鑰保存庫的租用戶識別碼並且為租用戶 B 中的主體設定正確的 ACL？

當您在訂用帳戶中建立新的金鑰保存庫時，它會自動繫結到該訂用帳戶的預設 Azure Active Directory 租用戶識別碼。所有存取原則項目也會繫結到此租用戶識別碼。當您將 Azure 訂用帳戶從租用戶 A 移到租用戶 B 時，租用戶 B 中的主體 (使用者和應用程式) 無法存取現有的金鑰保存庫。若要修正這個問題，您需要

- 將此訂用帳戶中與所有現有金鑰保存庫相關聯的租用戶識別碼變更為租用戶 B
- 移除所有現有的存取原則項目
- 新增與租用戶 B 相關聯的存取原則項目。

例如，如果您在已從租用戶 A 移到租用戶 B 的訂用帳戶中有金鑰保存庫 'myvault'，以下就是您變更此金鑰保存庫的租用戶識別碼及移除舊存取原則的方式。

<pre>
$vaultResourceId = (Get-AzureRmKeyVault -VaultName myvault).ResourceId
$vault = Get-AzureRmResource –ResourceId $vaultResourceId -ExpandProperties
$vault.Properties.TenantId = (Get-AzureRmContext).Tenant.TenantId
$vault.Properties.AccessPolicies = @()
Set-AzureRmResource -ResourceId $vaultResourceId -Properties $vault.Properties
</pre>

由於此保存庫在移動前位於租用戶 A 中，所以 **$vault.Properties.TenantId** 的原始值是租用戶 A，而 **(Get-AzureRmContext).Tenant.TenantId** 則是租用戶 B。

現在，您的保存庫已與正確的租用戶識別碼相關聯，而且移除了舊的存取原則項目，請使用 [Set-AzureRmKeyVaultAccessPolicy](https://msdn.microsoft.com/library/mt603625.aspx) 來設定新的存取原則項目。

## 後續步驟

- 如果您有關於金鑰保存庫的問題，請造訪 [Azure 金鑰保存庫論壇](https://social.msdn.microsoft.com/forums/azure/home?forum=AzureKeyVault)

<!---HONumber=AcomDC_0921_2016-->