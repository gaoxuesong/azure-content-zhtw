<properties
	pageTitle=" 開始使用 Azure 傳統入口網站來傳遞點播內容 | Microsoft Azure"
	description="本教學課程將逐步引導您使用 Azure 傳統入口網站，藉由 Azure 媒體服務來實作點播視訊 (VoD) 內容傳遞應用程式。"
	services="media-services"
	documentationCenter=""
	authors="Juliako"
	manager="erikre"
	editor=""/>

<tags
	ms.service="media-services"
	ms.workload="media"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="get-started-article"
	ms.date="06/22/2016"
	ms.author="juliako"/>


# 開始使用 Azure 傳統入口網站來傳遞點播內容


[AZURE.INCLUDE [media-services-selector-get-started](../../includes/media-services-selector-get-started.md)]


本教學課程將逐步引導您透過 Azure 傳統入口網站實作基本的點播視訊 (VoD) 內容傳遞應用程式。

> [AZURE.NOTE] 若要完成此教學課程，您需要 Azure 帳戶。如需詳細資訊，請參閱 [Azure 免費試用](/pricing/free-trial/?WT.mc_id=A261C142F)。


本教學課程內容包括以下工作：

1.  建立 Azure 媒體服務帳戶。
2.  設定串流端點。
1.  上傳視訊檔案。
1.  將來源檔案編碼為一組自適性 MP4 檔案。
1.  發佈資產並取得串流和漸進式下載 URL。
1.  播放您的內容。


## 建立 Azure 媒體服務帳戶

1. 在 [Azure 傳統入口網站](https://manage.windowsazure.com/)中，依序按一下 [新增]、[媒體服務] 和 [快速建立]。

	![Media Services Quick Create](./media/media-services-portal-get-started/wams-QuickCreate.png)

2. 在 [名稱] 中，輸入新帳戶的名稱。媒體服務帳戶名稱為全部小寫且不含空格的數字或字母，且長度是 3 到 24 個字元。

3. 在 [區域] 中，選取將用來儲存您媒體服務帳戶之中繼資料記錄的地理區域。只有可用的媒體服務區域才會出現在下拉式清單方塊中。

4. 在 [儲存體帳戶] 中，選取儲存體帳戶以從媒體服務帳戶提供媒體內容的 Blob 儲存體。您可以選取與媒體服務帳戶相同地理區域中的現有儲存體帳戶，也可以建立新的儲存體帳戶。新的儲存體帳戶會建立於相同的區域中。

5. 如果您已建立新的儲存體帳戶，請在 [NEW STORAGE ACCOUNT NAME] 中輸入儲存體帳戶的名稱。儲存體帳戶名稱的規則會與媒體服務帳戶相同。

6. 按一下表單底部的 [快速建立]。

	您可以在視窗底端的訊息區域監視程序的狀態。

	成功建立帳戶之後，狀態會變更為作用中。

	在頁面底部會顯示 [管理金鑰] 按鈕。當您按一下此按鈕時，會顯示對話方塊，其中具有媒體服務帳戶名稱和主要和次要金鑰。您將需要帳戶名稱和主要金鑰資訊，以便以程式設計方式存取媒體服務帳戶。

	![Media Services Page](./media/media-services-portal-get-started/wams-mediaservices-page.png)

	當您按兩下帳戶名稱時，預設會顯示 [快速入門] 頁面。此頁面可讓您執行一些在入口網站的其他頁面也可以執行的管理工作。例如，您可以從此頁面上傳視訊檔案，也可以從 [內容] 頁面上傳視訊檔案。


## 使用入口網站設定串流端點

使用 Azure 媒體服務時，其中一個最常見案例是提供調適性位元速率串流給您的用戶端。使用調適性位元速率串流，用戶端可以在視訊顯示時，根據目前網路頻寬、CPU 使用率以及其他因素，切換成較高或較低的位元速率串流。媒體服務支援下列調適性位元速率串流技術：HTTP 即時串流 (HLS)、Smooth Streaming、MPEG DASH 和 HDS (僅適用於 Adobe PrimeTime/Access 授權)。

媒體服務提供動態封裝，這讓您以媒體服務支援的串流格式 (MPEG DASH, 、HLS、Smooth Streaming、HDS) 提供調適性位元速率 MP4 或 Smooth Streaming 編碼內容，而不必重新封裝成這些串流格式。

若要利用動態封裝，您需要執行下列動作：

- 將您的夾層 (來源) 檔編碼為一組調適性位元速率 MP4 檔案或調適性位元速率 Smooth Streaming 檔案 (編碼步驟稍後示範於本教學課程中)。
- 為您計畫從該處傳遞內容的*串流端點*至少取得一個串流單元。

使用動態封裝，您只需要以單一儲存格式儲存及播放檔案，媒體服務會根據來自用戶端的要求建置及傳遞適當的回應。

若要變更串流保留單元數目，請執行下列動作：

1. 在 [Azure 傳統入口網站](https://manage.windowsazure.com/)中，按一下 [媒體服務]。接著，按一下媒體服務的名稱。

2. 選取 [串流端點] 頁面。然後，按一下您要修改的串流端點。

3. 若要指定串流單元數目，請選取 [調整] 索引標籤，然後移動 [保留容量] 滑動軸。

	![Scale page](./media/media-services-portal-get-started/media-services-origin-scale.png)

4. 按一下 [儲存] 以儲存您的變更。

	配置任何新的單元需要大約 20 分鐘的時間才能完成。

	>[AZURE.NOTE] 目前，如果串流單元從任何正值到無，可能會停用串流長達一小時。
	>
	> 計算成本時會使用 24 小時內指定的最大單元數。如需定價詳細資料的相關資訊，請參閱＜[媒體服務定價詳細資料](http://go.microsoft.com/fwlink/?LinkId=275107)＞。

## 上傳內容


1. 在 [Azure 傳統入口網站](http://go.microsoft.com/fwlink/?LinkID=256666&clcid=0x409)中，按一下 [媒體服務]，然後按一下媒體服務帳戶名稱。
2. 選取 [內容] 頁面。
3. 按一下頁面或入口網站底部的 [上傳] 按鈕。
4. 在 [上傳內容] 對話方塊中，瀏覽到想要的資產檔案。按一下檔案，然後按一下 [開啟] 或按 Enter 鍵。

	![UploadContentDialog][uploadcontent]

5. 在 [上傳內容] 對話方塊中，按一下核取按鈕以接受 [檔案] 和 [內容名稱]。
6. 上傳隨即開始，而您可以在入口網站底部追蹤進度。

	![JobStatus][status]

上傳完成之後，您將會看到新資產列在 [內容] 清單中。依照慣例，名稱的結尾會附加 "**-Source**"，以協助追蹤新的內容作為編碼工作的來源內容。

![ContentPage][contentpage]

如果上傳程序停止之後檔案大小值未更新，請選取 [同步處理中繼資料] 按鈕。這樣會以儲存體中的實際檔案大小同步處理資產檔案大小，並重新整理 [內容] 頁面上的值。


## 編碼內容

### 概觀

若要透過網際網路傳遞數位視訊，您必須壓縮媒體。媒體服務提供媒體編碼程式，可讓您指定要如何為您的內容編碼 (例如，要使用的轉碼器、檔案格式、解析度及位元速率)。

使用 Azure 媒體服務時，其中一個最常見案例是提供調適性位元速率串流給您的用戶端。使用調適性位元速率串流，用戶端可以在視訊顯示時，根據目前網路頻寬、CPU 使用率以及其他因素，切換成較高或較低的位元速率串流。媒體服務支援下列調適性位元速率串流技術：HTTP 即時串流 (HLS)、Smooth Streaming、MPEG DASH 和 HDS (僅適用於 Adobe PrimeTime/Access 授權)。

媒體服務提供動態封裝，這讓您以媒體服務支援的串流格式 (MPEG DASH, 、HLS、Smooth Streaming 或 HDS) 提供調適性位元速率 MP4 或 Smooth Streaming 編碼內容，而不必重新封裝成這些串流格式。

若要利用動態封裝，您需要執行下列動作：

- 將您的夾層 (來源) 檔編碼為一組調適性位元速率 MP4 檔案或調適性位元速率 Smooth Streaming 檔案 (編碼步驟稍後示範於本教學課程中)。
- 為您計畫從該處傳遞內容的串流端點取得至少一個隨選串流單元。如需詳細資訊，請參閱[如何調整隨選串流保留單元](media-services-manage-origins.md#scale_streaming_endpoints/)。

使用動態封裝，您只需要以單一儲存格式儲存及播放檔案，媒體服務會根據來自用戶端的要求建置及傳遞適當的回應。

請注意，除了能夠使用動態封裝功能，隨選串流保留單元提供您專用傳出功能，此功能可以以 200 Mbps 為增量來購買。依預設，隨選串流是以共用執行個體模型來設定，可與其他所有使用者共用伺服器資源 (例如，運算或流出容量)。若要改進隨選串流輸送量，建議您購買隨選串流保留單元。

### 編碼

本章說明當您要使用 Azure 傳統入口網站來透過媒體編碼器標準為您的內容編碼時，可以採取的步驟。

1.  選取您想要編碼的檔案。如果支援此檔案類型的編碼，[內容] 頁面底部的 [處理] 按鈕將會啟用。
4. 在 [處理] 對話方塊中，選取 [媒體編碼器標準] 處理器。
5. 選擇其中一種**編碼組態**。

	![Process2][process2]

	[媒體編碼器標準的工作預設字串](https://msdn.microsoft.com/zh-TW/library/mt269960)主題說明每個預設格式的意義。

5. 接著，輸入想要的好記輸出內容名稱或接受預設值。然後，按一下核取按鈕以啟動編碼作業，而您可以在入口網站底部追蹤進度。
6. 選取 [確定]。

	編碼完成之後，[內容] 頁面會包含編碼的檔案。

	若要檢視編碼工作的進度，請切換到 [工作] 頁面。

	如果編碼完成之後檔案大小值未更新，請選取 [同步處理中繼資料] 按鈕。這樣會以儲存體中的實際檔案大小同步處理輸出資產檔案大小，並重新整理 [內容] 頁面上的值。


## 發佈內容

### 概觀

如要想提供 URL 給使用者，讓使用者可以利用這個 URL 來傳送或下載內容，請您先建立定位器來發佈您的資產。定位器可以存取資產中所含的檔案。媒體服務支援兩種類型的定位器：OnDemandOrigin 定位器，用於串流媒體 (例如，MPEG DASH、HLS 或 Smooth Streaming) 和存取簽章 (SAS) 定位器，用來下載媒體檔案。

當您使用 Azure 傳統入口網站發佈您的資產時，系統會為您建立定位器，並提供 OnDemand 式的 URL (如果您的資產包含 .ism 檔案) 或 SAS URL。

SAS URL 具有下列格式。

	{blob container name}/{asset name}/{file name}/{SAS signature}

資料流 URL 的格式如下，還可以用它來播放 Smooth Streaming 資產。

	{streaming endpoint name-media services account name}.streaming.mediaservices.windows.net/{locator ID}/{filename}.ism/Manifest

若要建立 HLS 資料流 URL，請在 URL 加上 (format=m3u8-aapl)。

	{streaming endpoint name-media services account name}.streaming.mediaservices.windows.net/{locator ID}/{filename}.ism/Manifest(format=m3u8-aapl)

若要建立 MPEG DASH 資料流 URL，請在 URL 加上 (format=mpd-time-csf)。

	{streaming endpoint name-media services account name}.streaming.mediaservices.windows.net/{locator ID}/{filename}.ism/Manifest(format=mpd-time-csf)


定位器有到期日。使用入口網站來發佈您的資產時，建立的定位器會在 100 年後失效。

>[AZURE.NOTE] 如果您在 2015 年 3 月前使用入口網站來建立定位器，則建立的定位器 2 年後便告失效。

若要更新定位器的到期日，請使用 [REST](http://msdn.microsoft.com/library/azure/hh974308.aspx#update_a_locator) 或 [.NET](http://go.microsoft.com/fwlink/?LinkID=533259) API。請注意，當您更新 SAS 定位器的到期日，URL 也會隨之變更。

### Publish

若要使用入口網站發佈資產，請執行下列作業：

1. 選取資產。
2. 然後，按一下 [發佈] 按鈕。

 ![PublishedContent][publishedcontent]


## 從入口網站播放內容

Azure 傳統入口網站提供內容播放程式，可讓您用來測試視訊。

按一下想要的視訊，然後按一下入口網站底部的 [播放] 按鈕。

適用一些考量事項：

- 確定已發佈視訊。
- **媒體服務內容播放程式**會從預設串流端點播放。如果您想要從非預設串流端點播放，請使用其他播放程式。例如，[Azure 媒體服務播放器](http://amsplayer.azurewebsites.net/azuremediaplayer.html)。


![AMSPlayer][AMSPlayer]



##後續步驟：媒體服務學習路徑

[AZURE.INCLUDE [media-services-learning-paths-include](../../includes/media-services-learning-paths-include.md)]

##提供意見反應

[AZURE.INCLUDE [media-services-user-voice-include](../../includes/media-services-user-voice-include.md)]

## 尋找其他內容嗎？

如果本主題未包含您預期的內容、缺少部分內容，或者提供了一些其他不符合您需求的方式，請在下方提供您使用 Disqus 執行緒的意見反應給我們。

### 其他資源
- <a href="http://channel9.msdn.com/Shows/Azure-Friday/Azure-Media-Services-101-Get-your-video-online-now-">Azure Media Services 101 - 立即在線上取得您的影片！</a>
- <a href="http://channel9.msdn.com/Shows/Azure-Friday/Azure-Media-Services-102-Dynamic-Packaging-and-Mobile-Devices">Azure Media Services 102 - 動態封裝和行動裝置</a>


<!-- Anchors. -->


<!-- URLs. -->
[Azure Classic Portal]: http://manage.windowsazure.com/


<!-- Images -->
[portaloverview]: ./media/media-services-portal-get-started/media-services-content-page.png
[publishedcontent]: ./media/media-services-portal-get-started/media-services-upload-content-published.png
[uploadcontent]: ./media/media-services-portal-get-started/UploadContent.png
[status]: ./media/media-services-portal-get-started/Status.png
[encoder]: ./media/media-services-manage-content/EncoderDialog2.png
[branding]: ./media/branding-reporting.png
[contentpage]: ./media/media-services-portal-get-started/media-services-content-page.png
[process]: ./media/media-services-manage-content/media-services-process-video.png
[process2]: ./media/media-services-portal-get-started/media-services-process-video2.png
[encrypt]: ./media/media-services-manage-content/media-services-encrypt-content.png
[AMSPlayer]: ./media/media-services-portal-get-started/media-services-portal-player.png

<!---HONumber=AcomDC_0629_2016-->