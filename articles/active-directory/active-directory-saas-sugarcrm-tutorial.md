<properties 
    pageTitle="教學課程：Azure Active Directory 與 SugarCRM 整合 | Microsoft Azure" 
    description="了解如何使用 SugarCRM 搭配 Azure Active Directory 來啟用單一登入、自動佈建和更多功能！" 
    services="active-directory" 
    authors="jeevansd"  
    documentationCenter="na" 
    manager="femila"/>
<tags 
    ms.service="active-directory" 
    ms.devlang="na" 
    ms.topic="article" 
    ms.tgt_pltfrm="na" 
    ms.workload="identity" 
    ms.date="09/11/2016" 
    ms.author="jeedes" />

#教學課程：Azure Active Directory 與 SugarCRM 整合
  
本教學課程的目的是要示範 Azure 與 SugarCRM 的整合。本教學課程中說明的案例假設您已經具有下列項目：

-   有效的 Azure 訂用帳戶
-   已啟用 Sugar CRM 單一登入功能的訂用帳戶
  
完成本教學課程之後，您指派給 Sugar CRM 的 Azure AD 使用者就能夠從您的 Sugar CRM 公司網站 (服務提供者起始登入)，或使用[存取面板](active-directory-saas-access-panel-introduction.md)來單一登入應用程式。
  
本教學課程中說明的案例由下列建置組塊組成：

1.  啟用 Sugar CRM 的應用程式整合
2.  設定單一登入
3.  設定使用者佈建
4.  指派使用者

![案例](./media/active-directory-saas-sugarcrm-tutorial/IC795881.png "案例")

##啟用 Sugar CRM 的應用程式整合
  
本節的目的是概述如何啟用 Sugar CRM 的應用程式整合。

###若要啟用 Sugar CRM 的應用程式整合，請執行下列步驟：

1.  在 Azure 傳統入口網站中，按一下左方瀏覽窗格的 [Active Directory]。

    ![Active Directory](./media/active-directory-saas-sugarcrm-tutorial/IC700993.png "Active Directory")

2.  從 [目錄] 清單中，選取要啟用目錄整合的目錄。

3.  若要開啟應用程式檢視，請在目錄檢視中，按一下頂端功能表中的 [應用程式]。

    ![應用程式](./media/active-directory-saas-sugarcrm-tutorial/IC700994.png "應用程式")

4.  按一下頁面底部的 [新增]。

    ![新增應用程式](./media/active-directory-saas-sugarcrm-tutorial/IC749321.png "新增應用程式")

5.  在 [欲執行動作] 對話方塊中，按一下 [從資源庫加入應用程式]。

    ![從資源庫新增應用程式](./media/active-directory-saas-sugarcrm-tutorial/IC749322.png "從資源庫新增應用程式")

6.  在 [搜尋方塊] 中，輸入 **Sugar CRM**。

    ![應用程式庫](./media/active-directory-saas-sugarcrm-tutorial/IC795882.png "應用程式庫")

7.  在結果窗格中，選取 [Sugar CRM]，然後按一下 [完成] 加入應用程式。

    ![Sugar CRM](./media/active-directory-saas-sugarcrm-tutorial/IC795883.png "Sugar CRM")

##設定單一登入
  
本節的目的是概述如何依據 SAML 通訊協定來使用同盟，讓使用者能夠以自己的 Azure AD 帳戶在 Sugar CRM 中進行驗證。在此程序中，您需要上傳 base-64 編碼憑證到您的 Sugar CRM 租用戶。如果您不熟悉這個程序，請參閱[如何將二進位憑證轉換成文字檔](http://youtu.be/PlgrzUZ-Y1o)。

###若要設定單一登入，請執行下列步驟：

1.  在 Azure 傳統入口網站的 [Sugar CRM] 應用程式整合頁面上，按一下 [設定單一登入] 以開啟 [設定單一登入] 對話方塊。

    ![設定單一登入](./media/active-directory-saas-sugarcrm-tutorial/IC795884.png "設定單一登入")

2.  在 [要如何讓使用者登入 Sugar CRM] 頁面上，選取 [Microsoft Azure AD 單一登入]，然後按 [下一步]。

    ![設定單一登入](./media/active-directory-saas-sugarcrm-tutorial/IC795885.png "設定單一登入")

3.  在 [設定應用程式 URL] 頁面的 [Sugar CRM 登入 URL] 文字方塊中，輸入使用者用來登入 Sugar CRM 應用程式的 URL (例如："*http://company.sugarondemand.com*"*)，然後按 [下一步]*。

    ![設定應用程式 URL](./media/active-directory-saas-sugarcrm-tutorial/IC795886.png "設定應用程式 URL")

4.  在 [設定在 Sugar CRM 單一登入] 頁面上，若要下載您的憑證，請按一下 [下載憑證]，然後將憑證檔案儲存在您的電腦上。

    ![設定單一登入](./media/active-directory-saas-sugarcrm-tutorial/IC796918.png "設定單一登入")

5.  在不同的 Web 瀏覽器視窗中，以系統管理員身分登入您的 Sugar CRM 公司網站。

6.  移至 [管理]。

    ![Admin](./media/active-directory-saas-sugarcrm-tutorial/IC795888.png "Admin")

7.  在 [管理] 區段中，按一下 [密碼管理]。

    ![系統管理](./media/active-directory-saas-sugarcrm-tutorial/IC795889.png "系統管理")

8.  按一下 [啟用 SAML 驗證]。

    ![系統管理](./media/active-directory-saas-sugarcrm-tutorial/IC795890.png "系統管理")

9.  在 [SAML 驗證] 區段中，執行下列步驟：

    ![SAML 驗證](./media/active-directory-saas-sugarcrm-tutorial/IC795891.png "SAML 驗證")

    1.  在 Azure 傳統入口網站的 [設定在 Sugar CRM 單一登入] 對話方塊頁面上，複製 [遠端登入 URL] 值，然後將它貼到 [登入 URL] 文字方塊中。
    2.  在 Azure 傳統入口網站的 [設定在 Sugar CRM 單一登入] 對話方塊頁面上，複製 [遠端登入 URL] 值，然後將它貼到 [SLO URL] 文字方塊中。
    3.  從您下載的憑證建立 **Base-64 編碼**檔案。

        >[AZURE.TIP] 如需詳細資料，請參閱[如何將二進位憑證轉換成文字檔](http://youtu.be/PlgrzUZ-Y1o)

    4.  在記事本中開啟您的 base-64 編碼的憑證，將它的內容複製到您的剪貼簿，然後將整個憑證貼到 [X.509 憑證] 文字方塊中。
    5.  按一下 [儲存]。

10. 在 Azure 傳統入口網站的 [設定在 Sugar CRM 單一登入] 對話方塊上，選取 [單一登入設定確認]，然後按一下 [完成]。

    ![設定單一登入](./media/active-directory-saas-sugarcrm-tutorial/IC796919.png "設定單一登入")

##設定使用者佈建
  
若要讓 Azure AD 使用者能夠登入 Sugar CRM，必須將他們佈建到 Sugar CRM。Sugar CRM 需以手動方式佈建。

###若要佈建使用者帳戶，請執行下列步驟：

1.  以系統管理員身分登入您的 **Sugar CRM** 公司網站。

2.  移至 [管理]。

    ![Admin](./media/active-directory-saas-sugarcrm-tutorial/IC795888.png "Admin")

3.  在 [管理] 區段中，按一下 [使用者管理]。

    ![系統管理](./media/active-directory-saas-sugarcrm-tutorial/IC795893.png "系統管理")

4.  移至 [使用者] > [建立新的使用者]。

    ![建立新的使用者](./media/active-directory-saas-sugarcrm-tutorial/IC795894.png "建立新的使用者")

5.  在 [使用者設定檔] 索引標籤上，執行下列步驟：

    ![新使用者](./media/active-directory-saas-sugarcrm-tutorial/IC795895.png "新使用者")

    1.  在相關的文字方塊中，輸入有效 Azure Active Directory 使用者的使用者名稱、姓氏和電子郵件地址。

6.  在 [狀態] 選取 [作用中]。

7.  在 [密碼] 索引標籤上，執行下列步驟：

    ![新使用者](./media/active-directory-saas-sugarcrm-tutorial/IC795896.png "新使用者")

    1.  在相關的文字方塊中輸入密碼。
    2.  按一下 [儲存]。

>[AZURE.NOTE] 您可以使用任何其他的 Sugar CRM 使用者帳戶建立工具或 Sugar CRM 提供的 API 來佈建 AAD 使用者帳戶。

##指派使用者
  
若要測試您的組態，則需指派您所允許使用您應用程式的 Azure AD 使用者，藉此授予其存取組態的權限。

###若要將使用者指派到 Sugar CRM，請執行下列步驟：

1.  在 Azure 傳統入口網站中建立測試帳戶。

2.  在 [Sugar CRM] 應用程式整合頁面上，按一下 [指派使用者]。

    ![指派使用者](./media/active-directory-saas-sugarcrm-tutorial/IC795897.png "指派使用者")

3.  選取測試使用者，按一下 [指派]，然後按一下 [是] 以確認指派。

    ![是](./media/active-directory-saas-sugarcrm-tutorial/IC767830.png "是")
  
如果要測試您的單一登入設定，請開啟存取面板。如需存取面板的詳細資料，請參閱[存取面板簡介](active-directory-saas-access-panel-introduction.md)。

<!---HONumber=AcomDC_0914_2016-->