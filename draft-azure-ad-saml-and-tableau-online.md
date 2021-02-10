# Draft: Azure AD and Tableau

### Scope

Microsoft provides Azure AD apps that can be used to simplify the integration between Tableau Server and Tableau Online and Azure AD. The goal is to create a good onboarding and user experience to the Tableau services.

### Features

The two apps have a different feature sets. The Tableau Online application supports the following three features, whereas Tableau Server does not support user provisioning.

1. SP-initiated SSO
2. SP-Initiated Single Logout \(SLO\)
3. **REST API user provisioning**

Neither apps support [IdP-initiated sign-on](https://duo.com/blog/the-beer-drinkers-guide-to-saml). This means that if you publish the app in the Azure MyApps portal it will still do an SP-initiated Authentication request and therefore have the usual redirections in the browser for that flow.

### Documentation

There are articles to configure the authentication steps. The Microsoft ones are little simpler to follow as they have screenshots.

**Microsoft Docs:**  
[Tutorial: Azure Active Directory single sign-on \(SSO\) integration with Tableau Online](https://docs.microsoft.com/en-us/azure/active-directory/saas-apps/tableauonline-tutorial)  
[Tutorial: Azure Active Directory single sign-on \(SSO\) integration with Tableau Server](https://docs.microsoft.com/en-us/azure/active-directory/saas-apps/tableauserver-tutorial)  
**Tableau Docs:**   
[Configure SAML with Azure Active Directory](https://help.tableau.com/current/online/en-us/saml_config_azure_ad.htm) \(Tableau Online\)  
[Configure Server-Wide SAML](https://help.tableau.com/current/server/en-us/config_saml.htm) \(Tableau Server\)

### SAML Attributes

As part of the SAML authentication flow attributes are passed as assertions between the IdP \(Azure\) and the Service Provider \(Tableau\). Getting them right is key to a successful SSO. 

#### Tableau Online

There are two main properties that TOL is interested in, your **Email** and **Display Name**. The Email attribute is mapped to the username in Tableau Online and must match a licensed user stored in the Tableau Server Repository. ****The Display Name maps to the Full Name field in Tableau Online, it is populated with the assertions for **First name** and **Last name** or **Full name**. ****If they are not provided in the AuthN flow then the email address is used.

![](.gitbook/assets/image%20%2868%29.png)

![TOL Attribute Configuration](.gitbook/assets/image%20%2861%29.png)

#### Claims

[How to: customize claims issued in the SAML token for enterprise applications](https://docs.microsoft.com/en-us/azure/active-directory/develop/active-directory-saml-claims-customization)

![](.gitbook/assets/image%20%2863%29.png)



#### 

#### Email  __

The guidance we give on the attributes to use has some nuance.

* If all accounts you’re giving access to are sourced from **Microsoft accounts** \(e.g. outlook.com/gmail.com, or any consumer email domain that has signed up for an account\) this will be: `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/emailaddress`
* If all accounts are sourced from **Microsoft Azure Active Directory**, use the following value:

  `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/name`.

**Display Name:** Enter an assertion name for either the first name and last name, or for the full name, depending on how the IdP stores this information. Tableau Online uses these attributes to set the display name.

* `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/givenname`
* `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/surname`
* `http://schemas.microsoft.com/identity/claims/displayname`

So you can select either `givenname` + `surname` OR `displayname`

[https://docs.microsoft.com/en-us/windows-server/identity/ad-fs/technical-reference/the-role-of-claims](https://docs.microsoft.com/en-us/windows-server/identity/ad-fs/technical-reference/the-role-of-claims) 

![](.gitbook/assets/image%20%2862%29.png)

[Troubleshoot SAML](https://help.tableau.com/current/online/en-us/saml_trouble.htm)

### User Experience - Tableau Online

The Azure AD Tableau Online app always uses a SP-initiated flow. This means that the user experience involves a number of redirects. There are some configuration options that can smooth this experience.

#### Remember me

You should advise your users to always check the **Remember me** option when signing in to TOL. Without this checked the SAML flow will take the user back to the initial TOL SSO page to input the username before redirecting to the IdP.   
  


![](.gitbook/assets/image%20%2867%29.png)

If you select 'Remember me'  
  
Login URL workaround

[https://kb.tableau.com/articles/issue/prompted-to-enter-credentials-when-accessing-tableau-online-configured-with-saml](https://kb.tableau.com/articles/issue/prompted-to-enter-credentials-when-accessing-tableau-online-configured-with-saml)

SP-initiated -   
  
Message: AADSTS50011: The reply URL specified in the request does not match the reply URLs configured for the application: '[https://sso.online.tableau.com/public/sp/metadata?alias=4e828bd1-](https://sso.online.tableau.com/public/sp/metadata?alias=4e828bd1-df88-4648-ab4e-743eadabed2a)xxx'.



