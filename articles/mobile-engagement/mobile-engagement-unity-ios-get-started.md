<properties
	pageTitle="開始使用適用於 Unity iOS 部署的 Azure Mobile Engagement"
	description="了解如何使用 Azure Mobile Engagement 搭配分析和推播通知，以便將 Unity 應用程式部署至 iOS 裝置。"
	services="mobile-engagement"
	documentationCenter="unity"
	authors="piyushjo"
	manager=""
	editor="" />

<tags
	ms.service="mobile-engagement"
	ms.workload="mobile"
	ms.tgt_pltfrm="mobile-unity-ios"
	ms.devlang="dotnet"
	ms.topic="hero-article"
	ms.date="08/19/2016"
	ms.author="piyushjo" />

# 開始使用適用於 Unity iOS 部署的 Azure Mobile Engagement

[AZURE.INCLUDE [Hero 教學課程切換器](../../includes/mobile-engagement-hero-tutorial-switcher.md)]

本主題說明如何在部署至 iOS 裝置時，使用 Azure Mobile Engagement 來了解您應用程式的使用情形，以及如何傳送推播通知給 Unity 應用程式的區隔使用者。本教學課程使用傳統 Unity Roll a Ball 教學課程做為起點。您應該先遵循本[教學課程](mobile-engagement-unity-roll-a-ball.md)中的步驟，再繼續進行我們在下面教學課程中示範的 Mobile Engagement 整合。

本教學課程需要下列各項：

+ [Unity 編輯器](http://unity3d.com/get-unity)
+ [Mobile Engagement Unity SDK](https://aka.ms/azmeunitysdk)
+ XCode 編輯器

> [AZURE.NOTE] 若要完成此教學課程，您必須具備有效的 Azure 帳戶。如果您沒有帳戶，只需要幾分鐘的時間就可以建立免費試用帳戶。如需詳細資訊，請參閱 [Azure 免費試用](https://azure.microsoft.com/pricing/free-trial/?WT.mc_id=A0E0E5C02&amp;returnurl=http%3A%2F%2Fazure.microsoft.com%2Fzh-TW%2Fdocumentation%2Farticles%2Fmobile-engagement-unity-ios-get-started)。

##<a id="setup-azme"></a>為您的 iOS 應用程式設定 Mobile Engagement

[AZURE.INCLUDE [在入口網站中建立 Mobile Engagement 應用程式](../../includes/mobile-engagement-create-app-in-portal.md)]

##<a id="connecting-app"></a>將您的應用程式連線至 Mobile Engagement 後端

###匯入 Unity 封裝

1. 下載 [Mobile Engagement Unity 封裝](https://aka.ms/azmeunitysdk)並將它儲存到本機電腦。

2. 移至 [資產] -> [匯入封裝] -> [自訂封裝]，然後選取您在上一個步驟中下載的封裝。

	![][70]

3. 確定已選取所有檔案，然後按一下 [匯入] 按鈕。

	![][71]

4. 匯入成功後，您會在您的專案中看見匯入的 SDK 檔案。

	![][72]

###更新 EngagementConfiguration

1. 從 SDK 資料夾開啟 **EngagementConfiguration** 指令碼，並利用您先前從 Azure 入口網站取得的連接字串更新 **IOS\_CONNECTION\_STRING**。

	![][73]

2. 儲存檔案。

###設定應用程式以便進行基本追蹤

1. 開啟已附加至播放程式物件的 **PlayerController** 指令碼進行編輯。

2. 加入下列 using 陳述式：

		using Microsoft.Azure.Engagement.Unity;

3. 將下行新增至 `Start()` 方法
    
        EngagementAgent.Initialize();
        EngagementAgent.StartActivity("Home");

###部署並執行應用程式

1. 將 iOS 裝置連接到您的電腦。

2. 開啟 [檔案] -> [組建設定]

	![][40]

3. 選取 [iOS]，然後按一下 [切換平台]

	![][41]

	![][42]

4. 按一下 [播放器設定] 並提供有效的 [套件組合識別碼]。

	![][53]

5. 最後按一下 [建置並執行]

	![][54]

6. 系統可能會要求您提供用來儲存 iOS 封裝的資料夾名稱。

	![][43]

7. 如果一切正常，則會編譯專案，而您應該在 XCode 應用程式上開啟它。

8. 確定專案中的 [套件組合識別碼] 正確無誤。

	![][75]

10. 現在在 XCode 中執行應用程式，封裝就會部署到您連接的裝置，您應該會在電話上看到您的 Unity 遊戲！

##<a id="monitor"></a>將應用程式與即時監視連接

[AZURE.INCLUDE [將 App 與即時監視連接](../../includes/mobile-engagement-connect-app-with-monitor.md)]

##<a id="integrate-push"></a>啟用推播通知與 App 內傳訊

Mobile Engagement 可讓您透過「推播通知」和「應用程式內傳訊」，於活動進行時與使用者互動和觸達 (REACH)。此模組在 Mobile Engagement 入口網站中稱為 REACH。您不必在您的應用程式中做任何額外的設定來接收通知，它早已設定好了。

[AZURE.INCLUDE [mobile-engagement-ios-send-push-push](../../includes/mobile-engagement-ios-send-push.md)]

<!-- Images. -->
[40]: ./media/mobile-engagement-unity-ios-get-started/40.png
[41]: ./media/mobile-engagement-unity-ios-get-started/41.png
[42]: ./media/mobile-engagement-unity-ios-get-started/42.png
[43]: ./media/mobile-engagement-unity-ios-get-started/43.png
[53]: ./media/mobile-engagement-unity-ios-get-started/53.png
[54]: ./media/mobile-engagement-unity-ios-get-started/54.png
[70]: ./media/mobile-engagement-unity-ios-get-started/70.png
[71]: ./media/mobile-engagement-unity-ios-get-started/71.png
[72]: ./media/mobile-engagement-unity-ios-get-started/72.png
[73]: ./media/mobile-engagement-unity-ios-get-started/73.png
[74]: ./media/mobile-engagement-unity-ios-get-started/74.png
[75]: ./media/mobile-engagement-unity-ios-get-started/75.png

<!---HONumber=AcomDC_0824_2016-->