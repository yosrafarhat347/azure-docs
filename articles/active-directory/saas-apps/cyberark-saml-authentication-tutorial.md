---
title: 'Tutorial: Azure Active Directory single sign-on (SSO) integration with CyberArk SAML Authentication | Microsoft Docs'
description: Learn how to configure single sign-on between Azure Active Directory and CyberArk SAML Authentication.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 02/09/2021
ms.author: jeedes
---

# Tutorial: Azure Active Directory single sign-on (SSO) integration with CyberArk SAML Authentication

In this tutorial, you'll learn how to integrate CyberArk SAML Authentication with Azure Active Directory (Azure AD). When you integrate CyberArk SAML Authentication with Azure AD, you can:

* Control in Azure AD who has access to CyberArk SAML Authentication.
* Enable your users to be automatically signed-in to CyberArk SAML Authentication with their Azure AD accounts.
* Manage your accounts in one central location - the Azure portal.

## Prerequisites

To get started, you need the following items:

* An Azure AD subscription. If you don't have a subscription, you can get a [free account](https://azure.microsoft.com/free/).
* CyberArk SAML Authentication single sign-on (SSO) enabled subscription.

## Scenario description

In this tutorial, you configure and test Azure AD SSO in a test environment.

* CyberArk SAML Authentication supports **SP and IDP** initiated SSO.

## Add CyberArk SAML Authentication from the gallery

To configure the integration of CyberArk SAML Authentication into Azure AD, you need to add CyberArk SAML Authentication from the gallery to your list of managed SaaS apps.

1. Sign in to the Azure portal using either a work or school account, or a personal Microsoft account.
1. On the left navigation pane, select the **Azure Active Directory** service.
1. Navigate to **Enterprise Applications** and then select **All Applications**.
1. To add new application, select **New application**.
1. In the **Add from the gallery** section, type **CyberArk SAML Authentication** in the search box.
1. Select **CyberArk SAML Authentication** from results panel and then add the app. Wait a few seconds while the app is added to your tenant.

## Configure and test Azure AD SSO for CyberArk SAML Authentication

Configure and test Azure AD SSO with CyberArk SAML Authentication using a test user called **B.Simon**. For SSO to work, you need to establish a link relationship between an Azure AD user and the related user in CyberArk SAML Authentication.

To configure and test Azure AD SSO with CyberArk SAML Authentication, perform the following steps:

1. **[Configure Azure AD SSO](#configure-azure-ad-sso)** - to enable your users to use this feature.
    * **[Create an Azure AD test user](#create-an-azure-ad-test-user)** - to test Azure AD single sign-on with B.Simon.
    * **[Assign the Azure AD test user](#assign-the-azure-ad-test-user)** - to enable B.Simon to use Azure AD single sign-on.
1. **[Configure CyberArk SAML Authentication SSO](#configure-cyberark-saml-authentication-sso)** - to configure the single sign-on settings on application side.
    * **[Create CyberArk SAML Authentication test user](#create-cyberark-saml-authentication-test-user)** - to have a counterpart of B.Simon in CyberArk SAML Authentication that is linked to the Azure AD representation of user.
1. **[Test SSO](#test-sso)** - to verify whether the configuration works.

## Configure Azure AD SSO

Follow these steps to enable Azure AD SSO in the Azure portal.

1. In the Azure portal, on the **CyberArk SAML Authentication** application integration page, find the **Manage** section and select **single sign-on**.
1. On the **Select a single sign-on method** page, select **SAML**.
1. On the **Set up single sign-on with SAML** page, click the pencil icon for **Basic SAML Configuration** to edit the settings.

   ![Edit Basic SAML Configuration](common/edit-urls.png)

1. On the **Basic SAML Configuration** section, if you wish to configure the application in **IDP** initiated mode, enter the values for the following fields:

    In the **Reply URL** text box, type a URL using the following pattern:
    `https://<PVWA DNS or IP>/passwordvault/api/auth/saml/logon`

1. Click **Set additional URLs** and perform the following step if you wish to configure the application in **SP** initiated mode:

    In the **Sign-on URL** text box, type a URL using the following pattern:
    `https://<PVWA DNS or IP>/PasswordVault/v10/logon/saml`

	> [!NOTE]
	> These values are not real. Update these values with the actual Reply URL and Sign-On URL. Contact [CyberArk SAML Authentication Client support team](mailto:bizdevtech@cyberark.com) to get these values. You can also refer to the patterns shown in the **Basic SAML Configuration** section in the Azure portal.

1. On the **Set up single sign-on with SAML** page, in the **SAML Signing Certificate** section,  find **Certificate (Base64)** and select **Download** to download the certificate and save it on your computer.

	![The Certificate download link](common/certificatebase64.png)

1. On the **Set up CyberArk SAML Authentication** section, copy the appropriate URL(s) based on your requirement.

	![Copy configuration URLs](common/copy-configuration-urls.png)

### Create an Azure AD test user

In this section, you'll create a test user in the Azure portal called B.Simon.

1. From the left pane in the Azure portal, select **Azure Active Directory**, select **Users**, and then select **All users**.
1. Select **New user** at the top of the screen.
1. In the **User** properties, follow these steps:
   1. In the **Name** field, enter `B.Simon`.  
   1. In the **User name** field, enter the username@companydomain.extension. For example, `B.Simon@contoso.com`.
   1. Select the **Show password** check box, and then write down the value that's displayed in the **Password** box.
   1. Click **Create**.

### Assign the Azure AD test user

In this section, you'll enable B.Simon to use Azure single sign-on by granting access to CyberArk SAML Authentication.

1. In the Azure portal, select **Enterprise Applications**, and then select **All applications**.
1. In the applications list, select **CyberArk SAML Authentication**.
1. In the app's overview page, find the **Manage** section and select **Users and groups**.
1. Select **Add user**, then select **Users and groups** in the **Add Assignment** dialog.
1. In the **Users and groups** dialog, select **B.Simon** from the Users list, then click the **Select** button at the bottom of the screen.
1. If you are expecting a role to be assigned to the users, you can select it from the **Select a role** dropdown. If no role has been set up for this app, you see "Default Access" role selected.
1. In the **Add Assignment** dialog, click the **Assign** button.

## Configure CyberArk SAML Authentication SSO

To configure single sign-on on **CyberArk SAML Authentication** side, you need to send the downloaded **Certificate (Base64)** and appropriate copied URLs from Azure portal to [CyberArk SAML Authentication support team](mailto:bizdevtech@cyberark.com). They set this setting to have the SAML SSO connection set properly on both sides.

### Create CyberArk SAML Authentication test user

In this section, you create a user called B.Simon in CyberArk SAML Authentication. Work with [CyberArk SAML Authentication support team](mailto:bizdevtech@cyberark.com) to add the users in the CyberArk SAML Authentication platform. Users must be created and activated before you use single sign-on.

## Test SSO

In this section, you test your Azure AD single sign-on configuration with following options. 

#### SP initiated:

* Click on **Test this application** in Azure portal. This will redirect to CyberArk SAML Authentication Sign on URL where you can initiate the login flow.  

* Go to CyberArk SAML Authentication Sign-on URL directly and initiate the login flow from there.

#### IDP initiated:

* Click on **Test this application** in Azure portal and you should be automatically signed in to the CyberArk SAML Authentication for which you set up the SSO. 

You can also use Microsoft My Apps to test the application in any mode. When you click the CyberArk SAML Authentication tile in the My Apps, if configured in SP mode you would be redirected to the application sign on page for initiating the login flow and if configured in IDP mode, you should be automatically signed in to the CyberArk SAML Authentication for which you set up the SSO. For more information about the My Apps, see [Introduction to the My Apps](https://support.microsoft.com/account-billing/sign-in-and-start-apps-from-the-my-apps-portal-2f3b1bae-0e5a-4a86-a33e-876fbd2a4510).

## Next steps

Once you configure CyberArk SAML Authentication you can enforce session control, which protects exfiltration and infiltration of your organization’s sensitive data in real time. Session control extends from Conditional Access. [Learn how to enforce session control with Microsoft Defender for Cloud Apps](/cloud-app-security/proxy-deployment-any-app).
