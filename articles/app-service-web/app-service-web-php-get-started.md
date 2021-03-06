<properties
	pageTitle="建立、設定和部署 PHP Web 應用應式至 Azure"
	description="示範如何在 Azure App Service 中執行 PHP (Laravel) Web 應用程式的教學課程。了解如何設定 Azure App Service，以符合您所選擇的 PHP 架構的需求。"
	services="app-service\web"
	documentationCenter="php"
	authors="cephalin"
	manager="wpickett"
	editor=""
	tags="mysql"/>

<tags
	ms.service="app-service-web"
	ms.workload="web"
	ms.tgt_pltfrm="na"
	ms.devlang="PHP"
	ms.topic="article"
	ms.date="06/03/2016" 
	ms.author="cephalin"/>

# 建立、設定和部署 PHP Web 應用應式至 Azure

[AZURE.INCLUDE [索引標籤](../../includes/app-service-web-get-started-nav-tabs.md)]

本教學課程說明如何建立、設定和部署 Azure 的 PHP Web 應用程式，以及如何設定 Azure App Service，以符合 PHP Web 應用程式的需求。本教學課程結束時，您將擁有一個實際在 [Azure App Service](../app-service/app-service-value-prop-what-is.md) 中運作的 [Laravel](https://www.laravel.com/) Web 應用程式。

身為 PHP 開發人員，您可以將喜歡的 PHP 架構帶進 Azure 中。本教學課程直接將 Laravel 當做具體的應用程式範例。您將了解：

- 使用 Git 部署
- 設定 PHP 版本
- 使用不在應用程式根目錄中的啟動檔案
- 存取環境特有的變數
- 在 Azure 中更新應用程式

您可以將這裡的所學運用到您部署至 Azure 的其他 PHP Web 應用程式。

## 必要條件

- 安裝 [PHP 5.6.x](http://php.net/downloads.php) (PHP 7 支援是 Beta 版)
- 安裝[編輯器](https://getcomposer.org/download/)
- 安裝 [Azure CLI](../xplat-cli-install.md)
- 安裝 [Git](http://www.git-scm.com/downloads)
- 取得 Microsoft Azure 帳戶。如果您沒有這類帳戶，可以[申請免費試用](/pricing/free-trial/?WT.mc_id=A261C142F)，或是[啟用自己的 Visual Studio 訂閱者權益](/pricing/member-offers/msdn-benefits-details/?WT.mc_id=A261C142F)。

>[AZURE.NOTE] 請看看作用中的 Web 應用程式。立即[試用 App Service](http://go.microsoft.com/fwlink/?LinkId=523751) 並建立短期的入門應用程式 — 不需信用卡，不需任何承諾。

## 在開發電腦上建立 PHP (Laravel) 應用程式

1. 開啟新的 Windows 命令提示字元、PowerShell 視窗、Linux Shell 或 OS X 終端機。執行下列命令來確認您的電腦上正確安裝所需的工具。

        php --version
        composer --version
        azure --version
        git --version

    ![Test tools installation before creating your PHP (Laravel) app for Azure](./media/app-service-web-php-get-started/test-tools.png)

    如果您尚未安裝工具，請參閱[必要條件](#Prerequisites)以取得下載連結。
    
2. 安裝 Laravel，如下所示︰

        composer global require "laravel/installer

3. `CD` 安裝到工作目錄，並建立新的 Laravel 應用程式，如下所示︰

        cd <working_directory>
        laravel new <app_name>

4. `CD` 安裝到新建立的 `<app_name>` 目錄，並測試應用程式，如下所示︰

        cd <app_name>
        php artisan serve
        
    您現在應該能夠在瀏覽器中瀏覽至 http://localhost:8000，並看到 Laravel 啟動顯示畫面。
    
    ![Test your PHP (Laravel) app locally before deploying it to Azure](./media/app-service-web-php-get-started/laravel-splash-screen.png)
    
目前為止只是一般的 Laravel 工作流程，還沒有開始<a href="https://laravel.com/docs/5.2" rel="nofollow">學習 Laravel</a>。我們繼續下去。

## 建立 Azure Web 應用程式並設定 Git 部署

>[AZURE.NOTE] 「等一下！ 如果我想要使用 FTP 來部署，該怎麼做？」 有一個 [FTP 教學課程](web-sites-php-mysql-deploy-use-ftp.md)可符合您的需求。

透過 Azure CLI，只要一行程式碼就可以在 Azure App Service 中建立 Web 應用程式並設定來進行 Git 部署。讓我們開始吧！

1. 變更為 ASM 模式並登入 Azure：

        azure config mode asm
        azure login
    
    依照說明訊息進行來繼續登入程序。
    
    ![Log in to Azure to deploy your PHP (Laravel) app to Azure](./media/app-service-web-php-get-started/log-in-to-azure-cli.png)

4. 執行命令建立 Azure Web 應用程式和 Git 部署。出現提示時，指定所需的區域數目。

        azure site create --git <app_name>
    
    ![Create the Azure resource for your PHP (Laravel) app in Azure](./media/app-service-web-php-get-started/create-site-cli.png)
    
    >[AZURE.NOTE] 如果您從未設定 Azure 訂用帳戶的部署認證，系統將會提示您加以建立。App Service 只會將這些認證 (而不是您的 Azure 帳戶認證) 使用於 Git 部署與 FTP 登入。
    
    此命令會在目前的目錄中建立新的 Git 儲存機制 (使用 `git init`)，並將它連線至 Azure 中的儲存機制當做 Git 遠端 (使用 `git remote add`)。

<a name="configure"/>
## 設定 Azure Web 應用程式

為了讓 Laravel 應用程式在 Azure 中運作，您必須注意幾件事。您將對您所選擇的 PHP 架構進行此類似的練習。

- 設定 PHP 5.5.9 或更高版本。如需得完整的伺服器需求清單，請參閱[最新的 Laravel 5.2 伺服器需求](https://laravel.com/docs/5.2#server-requirements)。清單的其餘部分是 Azure 的 PHP 安裝已啟用的擴充功能。
- 設定應用程式需要的環境變數。Laravel 使用 `.env` 檔案來輕鬆設定環境變數。不過，因為不會認可到原始檔控制 (請參閱 [Laravel 環境組態](https://laravel.com/docs/5.2/configuration#environment-configuration)，您將改以設定 Azure Web 應用程式的應用程式設定。
- 請確定最先載入 Laravel 應用程式的進入點：`public/index.php`。請參閱 [Laravel 生命週期概觀](https://laravel.com/docs/5.2/lifecycle#lifecycle-overview)。換句話說，您必須設定 Web 應用程式的根目錄 URL 以指向 `public` 目錄。
- 由於您有 composer.json，請在 Azure 中啟用編輯器擴充。如此一來，當您使用 `git push` 部署時，編輯器會替您取得所需的封裝。這樣比較方便。如果您未啟用編輯器自動化，則只需要從 `.gitignore` 檔案中移除 `/vendor`，在認可及部署程式碼時，Git 就會包含 ("un-ignores") `vendor` 目錄中的所有項目。

我們將依序設定這些工作。

4. 設定 Laravel 應用程式需要的 PHP 版本。

        azure site set --php-version 5.6

    您已完成設定 PHP 版本！
    
4. 為您的 Azure Web 應用程式產生新的 `APP_KEY`，並將它設為 Azure Web 應用程式的應用程式設定。

        php artisan key:generate --show
        azure site appsetting add APP_KEY="<output_of_php_artisan_key:generate_--show>"

4. 此外，開啟 Laravel 偵錯以攔截任何怪異的 `Whoops, looks like something went wrong.` 頁面。

        azure site appsetting add APP_DEBUG=true

    您已完成設定環境變數！
    
    >[AZURE.NOTE] 等一下，我們稍微放慢腳步，說明 Laravel 和 Azure 在這裡的運作情形。Laravel 使用 `.env` 根目錄來提供環境變數給應用程式，您可以在其中找到 `APP_DEBUG=true` (以及 `APP_KEY=...`) 這一行。`config/app.php` 中以程式碼 `'debug' => env('APP_DEBUG', false),` 存取這個變數。[env()](https://laravel.com/docs/5.2/helpers#method-env) 是 Laravel 協助程式方法，在幕後使用 PHP [getenv()](http://php.net/manual/en/function.getenv.php)。
    >
    >不過，Git 會忽略 `.env`，因為根目錄中的 `.gitignore` 檔案會呼叫它。簡單地說，本機 Git 儲存機制中的 `.env` 不會隨著其餘檔案一起推送至 Azure。當然，您可以從 `.gitignore` 中移除該行，但我們已聲明不建議將此檔案認可到原始檔控制。您還是需要設法在 Azure 中指定這些環境變數。
    >
    >好消息是，Azure App Service 中的應用程式設定在 PHP 中支援 [getenv()](http://php.net/manual/en/function.getenv.php)。因此，雖然您可以使用 FTP 或其他方式手動將 `.env` 檔案上傳到 Azure，但您也可以直接將您想要的變數指定為 Azure 應用程式設定，而不需要 Azure 中的 `.env`，如同您剛才的做法一樣。此外，如果變數同時存在於 `.env` 檔案和 Azure 應用程式設定中，則以 Azure 應用程式設定為優先。

4. 最後兩項工作 (設定虛擬目錄及啟用編輯器) 需要 [Azure 入口網站](https://portal.azure.com)，請以您的 Azure 帳戶登入 [Azure 入口網站](https://portal.azure.com)。

4. 從左窗格開始，按一下 [應用程式服務] > **&lt;app\_name>** > [工具]。

    ![Enable Composer for your PHP (Laravel) app in Azure](./media/app-service-web-php-get-started/configure-composer-tools.png)
    
    >[AZURE.TIP] 如果您按一下 [設定] 而不是 [工具]，您將能夠存取 [應用程式設定] 刀鋒視窗，在其中設定 PHP 版本、應用程式設定和虛擬目錄，如同您剛才的做法一樣。
    
4. 按一下 [擴充功能] > [新增]，新增擴充功能。

4. 在 [選擇擴充功能] [刀鋒視窗](../azure-portal-overview.md)中，選取 [編輯器] \(刀鋒視窗︰水平開啟的入口網站頁面)。

4. 在 [接受法律條款] 刀鋒視窗中按一下 [確定]。

5. 在 [新增擴充功能] 刀鋒視窗中按一下 [確定]。

    當 Azure 完成新增擴充功能後，您應該會在角落看到友善的快顯訊息，也會看到 [擴充功能] 刀鋒視窗中列出 [編輯器]。

    ![Extensions blade after enabling Composer for your PHP (Laravel) app in Azure](./media/app-service-web-php-get-started/configure-composer-end.png)

    您已完成啟用編輯器！
    
4. 回到 Web 應用程式的刀鋒視窗，按一下 [設定] > [應用程式設定]。

    ![Access Settings blade to set virtual directory for your PHP (Laravel) app in Azure](./media/app-service-web-php-get-started/configure-virtual-dir-settings.png)

    在 [應用程式設定] 刀鋒視窗中，注意您先前設定的 PHP 版本︰

    ![PHP version in Settings blade for your PHP (Laravel) app in Azure](./media/app-service-web-php-get-started/configure-virtual-dir-settings-a.png)

    以及您新增的應用程式設定︰
    
    ![App settings in Settings blade for your PHP (Laravel) app in Azure](./media/app-service-web-php-get-started/configure-virtual-dir-settings-b.png)

4. 捲動至刀鋒視窗的底部，變更根虛擬目錄來指向 **site\\wwwroot\\public** 而不是 **site\\wwwroot**。

    ![Set virtual directory for your PHP (Laravel) app in Azure](./media/app-service-web-php-get-started/configure-virtual-dir-public.png)

4. 按一下刀鋒視窗頂端的 [儲存]。

    您已完成設定虛擬目錄！

## 使用 Git (和設定環境變數) 部署 Web 應用程式

您現在可以開始部署程式碼。您將回到命令提示字元或終端機來執行此工作。

4. 認可所有變更，並將程式碼部署至 Azure Web 應用程式，如同在任何 Git 儲存機制中的做法一樣︰

        git add .
        git commit -m "Hurray! My first commit for my Azure app!"
        git push azure master 

    當執行 `git push`，系統會要求您提供 Git 部署密碼。如果您稍早要求在 `azure site create` 中建立部署認證，請輸入您所使用的密碼。
    
5. 讓我們執行此命令來看它在瀏覽器中執行的情形︰

        azure site browse

    瀏覽器應該會顯示 Laravel 啟動顯示畫面。
    
    ![Laravel splash screen after deploying web app to Azure](./media/app-service-web-php-get-started/laravel-azure-splash-screen.png)
    
    恭喜，您現在已在 Azure 中執行 Laravel Web 應用程式。
             
## 常見問題疑難排解

以下是您遵循本教學課程時可能遇到的一些錯誤︰

- [Azure CLI 顯示「'site' 不是 azure 命令」](#clierror)
- [Web 應用程式顯示 HTTP 403 錯誤](#http403)
- [Web 應用程式顯示「糟糕！似乎發生錯誤。」](#whoops)
- [Web 應用程式顯示「找不到支援的加密程式。」](#encryptor)

<a name="clierror"></a>
### Azure CLI 顯示「'site' 不是 azure 命令」

在命令列終端機中執行 `azure site *` 時，您看到錯誤：`error:   'site' is not an azure command. See 'azure help'.`

這通常是因為切換到 "ARM" (Azure Resource Manager) 模式。若要解決此問題，請執行 `azure config mode asm` 以切回到 "ASM" (Azure 服務管理) 模式。

<a name="http403"></a>
### Web 應用程式顯示 HTTP 403 錯誤

您已成功將 Web 應用程式部署至 Azure，但在瀏覽至 Azure Web 應用程式時，卻出現 `HTTP 403` 或 `You do not have permission to view this directory or page.`

這很可能是因為 Web 應用程式找不到 Laravel 應用程式的進入點。請確定您已變更虛擬根目錄來指向 `site\wwwroot\public`，Laravel 的 `index.php` 位於此處 (請參閱[設定 Azure Web 應用程式](#configure))。

<a name="whoops"></a>
### Web 應用程式顯示「糟糕！似乎發生錯誤。」

您已成功將 Web 應用程式部署至 Azure，但在瀏覽至 Azure Web 應用程式時，卻出現怪異的訊息：`Whoops, looks like something went wrong.`

若要取得更具描述性的錯誤，請將 `APP_DEBUG` 環境變數設為 `true`，以啟用 Laravel 偵錯 (請參閱[設定 Azure Web 應用程式](#configure))。

<a name="encryptor"></a>
### Web 應用程式顯示「找不到支援的加密程式。」

您已成功將 Web 應用程式部署至 Azure，但在瀏覽至 Azure Web 應用程式時，卻出現下列錯誤訊息：

![APP\_KEY missing in your PHP (Laravel) app in Azure](./media/app-service-web-php-get-started/laravel-error-APP_KEY.png)
    
這是很討厭的錯誤，但至少不會很奇怪，因為您已開啟 Laravel 偵錯。在 Laravel 論壇上大概搜尋一下錯誤字串，就知道這是因為沒有在 `.env` 中設定 APP\_KEY，或者，以您的情況而言，就是 Azure 中完全沒有 `.env`。您可以將 `APP_KEY` 設定為 Azure 應用程式設定，即可修正此問題 (請參閱[設定 Azure Web 應用程式](#configure))。
    
## 後續步驟

了解如何[在 Azure 中建立 MySQL 資料庫](../store-php-create-mysql-database.md)，將資料加入至您的應用程式。此外，關於 Azure 中的 PHP，請查看以下更有用的連結︰

- [PHP 開發人員中心](/develop/php/)。
- [從 Azure Marketplace 建立 Web 應用程式](app-service-web-create-web-app-from-marketplace.md)
- [在 Azure App Service Web Apps 中設定 PHP](web-sites-php-configure.md)
- [在 Azure App Service 中將 WordPress 轉換成多站台](web-sites-php-convert-wordpress-multisite.md)
- [Azure App Service 上的企業級 WordPress](web-sites-php-enterprise-wordpress.md)

<!---HONumber=AcomDC_0914_2016-->