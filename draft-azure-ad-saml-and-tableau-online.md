# Draft: Azure AD SAML and Tableau Online

Microsoft provides an Azure AD app that can be used to simplify the integration between Tableau Server and Tableau Online and Azure AD. 

### Features

The two apps have a different feature set which you need to be aware of. The Tableau Online application supports the following three SAML features, whereas Tableau Server does not support user provisioning.

1. SP-initiated SSO
2. SP-Initiated Single Logout \(SLO\)
3. **REST API user provisioning**

Neither apps support **idp-initiated sign-on** which means even if you publish the app in the Azure MyApps portal it will still do an SP-initiated Authentication request and therefore not provide seamless sign on from the portal.

### TOL SAML Authentication

There are articles to configure the authentication steps. The Microsoft ones are little simpler to follow as they have screenshots.

**Tableau Docs:**   
[https://help.tableau.com/current/online/en-us/saml\_config\_azure\_ad.htm](https://help.tableau.com/current/online/en-us/saml_config_azure_ad.htm)  
**Microsoft Docs:**  
[Tutorial: Azure Active Directory single sign-on \(SSO\) integration with Tableau Online](https://docs.microsoft.com/en-us/azure/active-directory/saas-apps/tableauonline-tutorial)  
[Tutorial: Azure Active Directory single sign-on \(SSO\) integration with Tableau Server](https://docs.microsoft.com/en-us/azure/active-directory/saas-apps/tableauserver-tutorial)

### 

### Login URL workaround

[https://kb.tableau.com/articles/issue/prompted-to-enter-credentials-when-accessing-tableau-online-configured-with-saml](https://kb.tableau.com/articles/issue/prompted-to-enter-credentials-when-accessing-tableau-online-configured-with-saml)

SP-initiated -   
  
Message: AADSTS50011: The reply URL specified in the request does not match the reply URLs configured for the application: '[https://sso.online.tableau.com/public/sp/metadata?alias=4e828bd1-](https://sso.online.tableau.com/public/sp/metadata?alias=4e828bd1-df88-4648-ab4e-743eadabed2a)xxx'.

### Metadata

Unlike Okta and OneLogin you can actually upload the metadata file to Azure AD.

![](.gitbook/assets/image%20%2865%29.png)



![](.gitbook/assets/image%20%2866%29.png)

### Attributes

There are two main properties that TOL is interested in, your **Email** and **Display Name**:

![TOL Attribute Configuration](.gitbook/assets/image%20%2861%29.png)

#### Email

_Enter the name of the IdP assertion that contains the email address sent from the IdP to Tableau Online during the authentication process. The user is authenticated if the IdP email address is an exact match for the user's email address as stored in Tableau._

The guidance we give on the attributes to use has some nuance.

* If all accounts you’re giving access to are sourced from **Microsoft accounts** \(e.g. outlook.com/gmail.com, or any consumer email domain that has signed up for an account\) this will be: `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/emailaddress`
* If all accounts are sourced from **Microsoft Azure Active Directory**, use the following value:

  `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/name`.

**Display Name:** Enter an assertion name for either the first name and last name, or for the full name, depending on how the IdP stores this information. Tableau Online uses these attributes to set the display name.

* `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/givenname`
* `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/surname`
* `http://schemas.microsoft.com/identity/claims/displayname`

So you can select either `givenname` + `surname` OR `displayname`

  
  


#### Azure AD Claims

![](.gitbook/assets/image%20%2863%29.png)

[https://docs.microsoft.com/en-us/windows-server/identity/ad-fs/technical-reference/the-role-of-claims](https://docs.microsoft.com/en-us/windows-server/identity/ad-fs/technical-reference/the-role-of-claims) 

![](.gitbook/assets/image%20%2862%29.png)

### User Experience

The Azure AD Tableau Online app uses a SP-initiated flow. 



