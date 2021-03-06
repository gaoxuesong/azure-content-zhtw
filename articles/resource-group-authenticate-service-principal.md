<properties
   pageTitle="使用 PowerShell 建立 Azure 服務主體 | Microsoft Azure"
   description="描述如何使用 Azure PowerShell 建立 Active Directory 應用程式和服務主體，並透過角色型存取控制將存取權授與資源。它示範如何使用密碼或憑證來驗證應用程式。"
   services="azure-resource-manager"
   documentationCenter="na"
   authors="tfitzmac"
   manager="timlt"
   editor="tysonn"/>

<tags
   ms.service="azure-resource-manager"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="multiple"
   ms.workload="na"
   ms.date="09/12/2016"
   ms.author="tomfitz"/>

# 使用 Azure PowerShell 建立用來存取資源的服務主體

> [AZURE.SELECTOR]
- [PowerShell](resource-group-authenticate-service-principal.md)
- [Azure CLI](resource-group-authenticate-service-principal-cli.md)
- [入口網站](resource-group-create-service-principal-portal.md)

當您有需要存取資源的應用程式或指令碼時，您很可能不會想要以您擁有的認證執行此程序。您可能具有應用程式的不同權限，而如果您的職責變更，您就不希望應用程式繼續使用您的認證。相反地，您可以為應用程式建立身分識別，在其中包含驗證認證和角色指派。每次應用程式執行時，它會以這些認證進行自我驗證。本主題說明如何使用 [Azure PowerShell](powershell-install-configure.md) 來設定所需的一切項目，以便讓應用程式利用自己的認證和身分識別來執行。

在 PowerShell 中，您有兩個選項可以驗證您的 AD 應用程式︰

 - password
 - 憑證

本主題說明如何在 PowerShell 中使用這兩個選項。如果您想要從程式設計架構 (例如 Python、Ruby 或 Node.js) 登入 Azure，密碼驗證可能是您的最佳選項。在決定是要使用密碼還是憑證時，請參閱[範例應用程式](#sample-applications)一節，以查看在不同架構進行驗證的範例。

## Active Directory 概念

在本文中，您會建立 Active Directory (AD) 應用程式和服務主體這兩個物件。AD 應用程式是您的應用程式的全域表示法。其中包含認證 (應用程式識別碼以及密碼或憑證)。服務主體是您的應用程式在 Active Directory 中的本機表示法。其中包含角色指派。本主題著重在說明單一租用戶應用程式，此應用程式的目的是只在一個組織內執行。您通常會將單一租用戶應用程式用在組織內執行的企業營運系統應用程式。在單一租用戶應用程式中，您有一個 AD 應用程式和一個服務主體。

您可能想知道 - 為什麼我需要這兩個物件？ 當您考慮多租用戶應用程式時，這個方法比較合適。您通常會將多租用戶應用程式用於軟體即服務 (SaaS) 應用程式，以便您的應用程式在許多不同的訂用帳戶中執行。對於多租用戶應用程式，您有一個 AD 應用程式和多個服務主體 (每個 Active Directory 中有一個服務主體可授與應用程式的存取權)。若要設定多租用戶應用程式，請參閱[利用 Azure Resource Manager API 進行授權的開發人員指南](resource-manager-api-authentication.md)。

## 所需的權限

若要完成本主題，您必須在 Azure Active Directory 和您的 Azure 訂用帳戶中有足夠的權限。具體來說，您必須能夠在 Active Directory 中建立應用程式，並將服務主體指派給角色。

在您的 Active Directory 中，您的帳戶必須是系統管理員 (例如 [全域管理員]或 [使用者管理員])。如果您的帳戶已指派給 [使用者] 角色，您需要讓系統管理員提高您的權限。

在您的訂用帳戶中，您的帳戶必須擁有 `Microsoft.Authorization/*/Write` 存取權，這是透過[擁有者](./active-directory/role-based-access-built-in-roles.md#owner)角色或[使用者存取管理員](./active-directory/role-based-access-built-in-roles.md#user-access-administrator)角色來授與。如果您的帳戶已指派給 [參與者] 角色，您會在嘗試將服務主體指派給角色時收到錯誤。同樣地，您的訂用帳戶管理員必須授與您足夠的存取權。

現在，繼續進行[密碼](#create-service-principal-with-password)或[憑證](#create-service-principal-with-certificate)驗證的章節。

## 使用密碼建立服務主體

在本節中，您會執行下列步驟：

- 建立具有密碼的 AD 應用程式
- 建立服務主體
- 將 [讀取者角色] 指派給服務主體

若要快速執行這些步驟，請參閱下列三個 Cmdlet。

     $app = New-AzureRmADApplication -DisplayName "{app-name}" -HomePage "https://{your-domain}/{app-name}" -IdentifierUris "https://{your-domain}/{app-name}" -Password "{your-password}"
     New-AzureRmADServicePrincipal -ApplicationId $app.ApplicationId
     New-AzureRmRoleAssignment -RoleDefinitionName Reader -ServicePrincipalName $app.ApplicationId.Guid

我們會更仔細解說這些步驟，確保您了解此程序。

1. 登入您的帳戶。

        Add-AzureRmAccount

1. 建立新的 Active Directory 應用程式，方法是提供顯示名稱、描述應用程式的 URI、識別應用程式的 URI，以及應用程式身分識別的密碼。

        $app = New-AzureRmADApplication -DisplayName "exampleapp" -HomePage "https://www.contoso.org/exampleapp" -IdentifierUris "https://www.contoso.org/exampleapp" -Password "<Your_Password>"

     對於單一租用戶應用程式，不會驗證 URI。
     
     如果您的帳戶沒有 Active Directory 的[必要權限](#required-permissions)，您會看到一則錯誤訊息，指出 "Authentication\_Unauthorized" 或「在內容中找不到任何訂用帳戶」。

1. 檢查新的應用程式物件。

        $app
        
     請特別注意，需要有 **ApplicationId** 屬性，才能建立服務主體、角色指派以及取得存取權杖。

        DisplayName             : exampleapp
        ObjectId                : c95e67a3-403c-40ac-9377-115fa48f8f39
        IdentifierUris          : {https://www.contoso.org/example}
        HomePage                : https://www.contoso.org
        Type                    : Application
        ApplicationId           : 8bc80782-a916-47c8-a47e-4d76ed755275
        AvailableToOtherTenants : False
        AppPermissions          : 
        ReplyUrls               : {}

2. 傳入 Active Directory 應用程式的應用程式識別碼來建立應用程式的服務主體。

        New-AzureRmADServicePrincipal -ApplicationId $app.ApplicationId

3. 授與服務主體對您訂用帳戶的權限。在此範例中，您會將服務主體新增至 [讀取者] 角色，以授與讀取訂用帳戶中所有資源的權限。若為其他角色，請參閱 [RBAC︰內建角色](./active-directory/role-based-access-built-in-roles.md)。針對 **ServicePrincipalName** 參數，提供您在建立應用程式時所使用的 **ApplicationId**。

        New-AzureRmRoleAssignment -RoleDefinitionName Reader -ServicePrincipalName $app.ApplicationId.Guid

    如果您的帳戶沒有足夠的權限可指派角色，您會看到一則錯誤訊息。此訊息說明您的帳戶**沒有權限來對 '/subscriptions/{guid}' 範圍執行 'Microsoft.Authorization/roleAssignments/write' 動作**。

就這麼簡單！ 您的 AD 應用程式和服務主體已設定好。下一節會說明如何透過 PowerShell 以認證來登入。如果您想要使用程式碼應用程式中的認證，您可以跳到[範例應用程式](#sample-applications)。

### 透過 PowerShell 提供認證

現在，您需要以應用程式的形式登入以執行作業。

1. 執行 **Get-Credential** 命令，以建立含有您認證的 **PSCredential** 物件。您必須先擁有 **ApplicationId** 才能執行此命令，因此請確定您已有上述項目可供貼上。

        $creds = Get-Credential

2. 系統會提示您輸入認證。針對使用者名稱，使用您在建立應用程式時所使用的 **ApplicationId**。針對密碼，使用您在建立帳戶時所指定的密碼。

     ![輸入認證](./media/resource-group-authenticate-service-principal/arm-get-credential.png)

2. 每當您以服務主體的形式登入時，都需要提供 AD 應用程式目錄的租用戶識別碼。租用戶是 Active Directory 的執行個體。如果您只有一個訂用帳戶，您可以使用：

        $tenant = (Get-AzureRmSubscription).TenantId
    
     如果您有多個訂用帳戶，請指定 Active Directory 所在的訂用帳戶。如需詳細資訊，請參閱 [Azure 訂用帳戶如何與 Azure Active Directory 產生關聯](./active-directory/active-directory-how-subscriptions-associated-directory.md)。

        $tenant = (Get-AzureRmSubscription -SubscriptionName "Contoso Default").TenantId

4. 登入為服務主體，方法是指定此帳戶為服務主體，以及提供認證物件。

        Add-AzureRmAccount -Credential $creds -ServicePrincipal -TenantId $tenant
        
     您現在驗證為您所建立之 Active Directory 應用程式的服務主體。

### 儲存存取權杖以簡化登入

若要避免每次需要登入時都要提供服務主體認證，您可以儲存存取權杖。

1. 若要在稍後的工作階段中使用目前的存取權杖，請儲存設定檔。

        Save-AzureRmProfile -Path c:\Users\exampleuser\profile\exampleSP.json
        
     開啟設定檔，並檢查其內容。請注意其中包含存取權杖。
        
2. 請不要以手動方式再次登入，只要載入設定檔即可。

        Select-AzureRmProfile -Path c:\Users\exampleuser\profile\exampleSP.json
        
> [AZURE.NOTE] 存取權杖會過期，因此，只要權杖有效，就只使用可使用的已儲存設定檔。
        
## 使用憑證建立服務主體

在本節中，您會執行下列步驟：

- 建立自我簽署憑證
- 建立具有憑證的 AD 應用程式
- 建立服務主體
- 將 [讀取者角色] 指派給服務主體

若要在 Windows 10 或 Windows Server 2016 Technical Preview 上使用 Azure PowerShell 2.0 快速執行這些步驟，請參閱下列 Cmdlet。

    $cert = New-SelfSignedCertificate -CertStoreLocation "cert:\CurrentUser\My" -Subject "CN=exampleapp" -KeySpec KeyExchange
    $keyValue = [System.Convert]::ToBase64String($cert.GetRawCertData())
    $app = New-AzureRmADApplication -DisplayName "exampleapp" -HomePage "https://www.contoso.org" -IdentifierUris "https://www.contoso.org/example" -CertValue $keyValue -EndDate $cert.NotAfter -StartDate $cert.NotBefore
    New-AzureRmADServicePrincipal -ApplicationId $app.ApplicationId
    New-AzureRmRoleAssignment -RoleDefinitionName Reader -ServicePrincipalName $app.ApplicationId.Guid

我們會更仔細解說這些步驟，確保您了解此程序。本文也說明如何在使用舊版 Azure PowerShell 或作業系統時完成工作。

### 建立自我簽署憑證

Windows 10 和 Windows Server 2016 Technical Preview 提供的 PowerShell 版本已更新用來產生自我簽署憑證的 **New-SelfSignedCertificate** Cmdlet。舊版作業系統有 New-SelfSignedCertificate Cmdlet，但它不會提供本主題所需的參數。相反地，您需要匯入模組來產生憑證。根據您擁有的作業系統，本主題說明兩種用來產生憑證的方法。

- 如果您執行 **Windows 10 或 Windows Server 2016 Technical Preview**，請執行下列命令來建立自我簽署憑證：

        $cert = New-SelfSignedCertificate -CertStoreLocation "cert:\CurrentUser\My" -Subject "CN=exampleapp" -KeySpec KeyExchange
       
- 如果您**不是執行 Windows 10 或 Windows Server 2016 Technical Preview**，則必須從 Microsoft 指令碼中心下載[自我簽署憑證產生器](https://gallery.technet.microsoft.com/scriptcenter/Self-signed-certificate-5920a7c6/)。將其內容解壓縮，並匯入您需要的 Cmdlet。
     
        # Only run if you could not use New-SelfSignedCertificate
        Import-Module -Name c:\ExtractedModule\New-SelfSignedCertificateEx.ps1
    
     然後產生憑證。
    
        $cert = New-SelfSignedCertificateEx -Subject "CN=exampleapp" -KeySpec "Exchange" -FriendlyName "exampleapp"

您已擁有憑證，接下來可以繼續建立 AD 應用程式。

### 建立 Active Directory 應用程式和服務主體

1. 從憑證擷取金鑰值。

        $keyValue = [System.Convert]::ToBase64String($cert.GetRawCertData())

2. 登入您的 Azure 帳戶。

        Add-AzureRmAccount

3. 建立新的 Active Directory 應用程式，方法是提供顯示名稱、描述應用程式的 URI、識別應用程式的 URI，以及應用程式身分識別的密碼。

     如果您有 Azure PowerShell 2.0 (2016 年 8 月或更新版本)，請使用下列 Cmdlet：

        $app = New-AzureRmADApplication -DisplayName "exampleapp" -HomePage "https://www.contoso.org" -IdentifierUris "https://www.contoso.org/example" -CertValue $keyValue -EndDate $cert.NotAfter -StartDate $cert.NotBefore      
    
    如果您有 Azure PowerShell 1.0，請使用下列 Cmdlet：
    
        $app = New-AzureRmADApplication -DisplayName "exampleapp" -HomePage "https://www.contoso.org" -IdentifierUris "https://www.contoso.org/example" -KeyValue $keyValue -KeyType AsymmetricX509Cert  -EndDate $cert.NotAfter -StartDate $cert.NotBefore      
    
    對於單一租用戶應用程式，不會驗證 URI。
    
    如果您的帳戶沒有 Active Directory 的[必要權限](#required-permissions)，您會看到一則錯誤訊息，指出 "Authentication\_Unauthorized" 或「在內容中找不到任何訂用帳戶」。
        
    檢查新的應用程式物件。

        $app

    請注意，需要有 **ApplicationId** 屬性，才能建立服務主體、角色指派以及取得存取權杖。

        DisplayName             : exampleapp
        ObjectId                : c95e67a3-403c-40ac-9377-115fa48f8f39
        IdentifierUris          : {https://www.contoso.org/example}
        HomePage                : https://www.contoso.org
        Type                    : Application
        ApplicationId           : 8bc80782-a916-47c8-a47e-4d76ed755275
        AvailableToOtherTenants : False
        AppPermissions          : 
        ReplyUrls               : {}


5. 傳入 Active Directory 應用程式的應用程式識別碼來建立應用程式的服務主體。

        New-AzureRmADServicePrincipal -ApplicationId $app.ApplicationId

6. 授與服務主體對您訂用帳戶的權限。在此範例中，您會將服務主體新增至 [讀取者] 角色，以授與讀取訂用帳戶中所有資源的權限。若為其他角色，請參閱 [RBAC︰內建角色](./active-directory/role-based-access-built-in-roles.md)。針對 **ServicePrincipalName** 參數，提供您在建立應用程式時所使用的 **ApplicationId**。

        New-AzureRmRoleAssignment -RoleDefinitionName Reader -ServicePrincipalName $app.ApplicationId.Guid

    如果您的帳戶沒有足夠的權限可指派角色，您會看到一則錯誤訊息。此訊息說明您的帳戶**沒有權限來對 '/subscriptions/{guid}' 範圍執行 'Microsoft.Authorization/roleAssignments/write' 動作**。

就這麼簡單！ 您的 AD 應用程式和服務主體已設定好。下一節會說明如何透過 PowerShell 以憑證來登入。

### 透過自動化的 PowerShell 指令碼提供憑證

每當您以服務主體的形式登入時，都需要提供 AD 應用程式目錄的租用戶識別碼。租用戶是 Active Directory 的執行個體。如果您只有一個訂用帳戶，您可以使用：

    $tenant = (Get-AzureRmSubscription).TenantId
    
如果您有多個訂用帳戶，請指定 Active Directory 所在的訂用帳戶。如需詳細資訊，請參閱[管理您的 Azure AD 目錄](./active-directory/active-directory-administer.md)。

    $tenant = (Get-AzureRmSubscription -SubscriptionName "Contoso Default").TenantId

若要在指令碼中進行驗證，請指定帳戶為服務主體，並且提供憑證指紋、應用程式識別碼和租用戶識別碼。若要讓指令碼自動執行，您可以將這些值儲存為環境變數，並在執行期間擷取它們，或者您可以將它們包含在指令碼中。

    Add-AzureRmAccount -ServicePrincipal -CertificateThumbprint $cert.Thumbprint -ApplicationId $app.ApplicationId -TenantId $tenant

您現在驗證為您所建立之 Active Directory 應用程式的服務主體。

## 範例應用程式

下列範例應用程式會顯示如何登入為服務主體。

**.NET**

- [使用 .NET 以範本部署已啟用 SSH 的 VM](https://azure.microsoft.com/documentation/samples/resource-manager-dotnet-template-deployment/)
- [使用 .NET 管理 Azure 資源和資源群組](https://azure.microsoft.com/documentation/samples/resource-manager-dotnet-resources-and-groups/)

**Java**

- [開始使用資源 - 在 Java 中使用 Azure Resource Manager 範本進行部署](https://azure.microsoft.com/documentation/samples/resources-java-deploy-using-arm-template/)
- [開始使用資源 - 在 Java 中管理資源群組](https://azure.microsoft.com/documentation/samples/resources-java-manage-resource-group//)

**Python**

- [使用 Python 格式的範本部署已啟用 SSH 的 VM](https://azure.microsoft.com/documentation/samples/resource-manager-python-template-deployment/)
- [使用 Python 管理 Azure 資源和資源群組](https://azure.microsoft.com/documentation/samples/resource-manager-python-resources-and-groups/)

**Node.js**

- [使用 Node.js 格式的範本部署已啟用 SSH 的 VM](https://azure.microsoft.com/documentation/samples/resource-manager-node-template-deployment/)
- [使用 Node.js 管理 Azure 資源和資源群組](https://azure.microsoft.com/documentation/samples/resource-manager-node-resources-and-groups/)

**Ruby**

- [使用 Ruby 格式的範本部署已啟用 SSH 的 VM](https://azure.microsoft.com/documentation/samples/resource-manager-ruby-template-deployment/)
- [使用 Ruby 管理 Azure 資源和資源群組](https://azure.microsoft.com/documentation/samples/resource-manager-ruby-resources-and-groups/)

## 後續步驟
  
- 如需有關將應用程式整合至 Azure 來管理資源的詳細步驟，請參閱[利用 Azure Resource Manager API 進行授權的開發人員指南](resource-manager-api-authentication.md)。
- 如需應用程式和服務主體的詳細說明，請參閱[應用程式物件和服務主體物件](./active-directory/active-directory-application-objects.md)。
- 如需 Active Directory 驗證的詳細資訊，請參閱 [Azure AD 的驗證案例](./active-directory/active-directory-authentication-scenarios.md)。

<!---HONumber=AcomDC_0914_2016-->