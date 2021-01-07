# Recipe: Tableau and Okta SAML

TLDR

**Scope:**  
The testing the integrations between Tableau Online and Okta for SAML authentication. This focuses on the core SAML functionality to get up and running, not advanced configuration or embedded use cases and user provisioning.

_**Okta Tenant:**  
Sign yourself up for a Okta Developer tenant. The tenant is permanent and will allow you try all the necessary features._ [_https://developer.okta.com/signup/_](https://developer.okta.com/signup/)

**Okta Applications:**  
There are two published apps in the [Okta Integration Network \(OIN\)](https://help.okta.com/en/prod/Content/Topics/Apps/Apps_Apps.htm). One for Tableau Online \(TOL\) and the other for Tableau Server. They have a different SAML feature set which you need to be aware of. The Okta Tableau server application supports the following three SAML features. 

1. SP-initiated SSO
2. IdP-initiated SSO
3. _SP-Initiated Single Logout \(SLO\)_

The Okta TOL app does **not** support _SP-initiated SLO_ \(more on that later\). Also note neither apps support _IdP-initiated SLO._

Additionally, the TOL app supports SCIM for user provisioning more on that to come...

**Documentation:**  
I don't want to duplicate setup instructions here as Okta and Tableau docs do a good job. To be honest I found the Okta instructions better than the Tableau Online ones as they had screenshots. However the Okta Tableau Server ones look out of date.  
  
**Okta Resources:** **Setup**  
[How to Configure SAML 2.0 for Tableau Online](https://saml-doc.okta.com/SAML_Docs/How-to-Configure-SAML-2.0-for-Tableau-Online.html) -&gt; _this doesn't include SP-Initated SLO as explained below_  
[How to Configure SAML 2.0 for Tableau Server](https://saml-doc.okta.com/SAML_Docs/How-to-Configure-SAML-2.0-for-Tableau-Server.html) -&gt; _this has out of date screenshots - pre-TSM_  
**Tableau Resources: Setup**  
[Configure SAML with Okta](https://help.tableau.com/current/online/en-us/saml_config_okta.htm) - Tableau Online -&gt; _this doesn't include SP-Initated SLO as explained below_  
[SAML](https://help.tableau.com/current/server/en-us/saml.htm) - Tableau Server general SAML guidance

**Configuration:**  
The key setup configuration items for SAML are described below, I tested TOL but have included some information such as the Return URL and Certificates which are only required by a Tableau Server deployment. Surprisingly, Okta does not require \(or support\) the uploading of the Service Provider Metadata to complete the configuration using metadata exchange, so you have to input details manually.

| Product | _Configuration Item_ | Description |
| :--- | :--- | :--- |
| Tableau Online | **SAML entity ID** | The entity ID uniquely identifies your Tableau Server installation to the IdP. It represents a system entity in metadata, which is a SAML service, such as an IdP or an SP as you could have multiple listed in the metadata. The value of the entityID attribute SHOULD be the canonical URL of the entity's metadata document. An example from TOL: **entityID="https://sso.online.tableau.com/public/sp/metadata?alias=4b728bd1-df88-xxxx-xxxx-xxxxxxxxxxxx"** |
| Tableau Online | **Assertion Consumer Service \(ACS\) URL** | Service Providers support SSO protocols by including one or more endpoint elements in their metadata. These are the locations to which the IdP will eventually send the user at the SP. By enumerating them in the metadata, the IdP can ensure that the user's information is sent only to authorized locations. An example from TOL: **Location="https://sso.online.tableau.com/public/sp/SSO/4b728bd1-df88-xxxx-xxxx-xxxxxxxxxxxx"** |
| Tableau Online | **IdP Metadata XML file** | SAML metadata is configuration data required to automatically negotiate agreements between system entities, comprising identifiers, binding support and endpoints, certificates, keys, cryptographic capabilities and security and privacy policies. You will download this from the Okta portal. |
| Tableau Server | **Return URL** | The URL that Tableau Server users will access, such as `https://tableau-server`. Using `https://localhost` or a URL with a trailing slash `http://tableau_server/` is not supported. |
| Tableau Server | **SAML Certificate and key files** | Tableau Server requires a certificate-key pair to encrypt the traffic, sign the request that is sent to the IdP and encrypt assertions. There are [Certificate Requirements](https://help.tableau.com/current/server/en-us/saml_requ.htm#certificate-and-identity-provider-idp-requirements) listed which are specific so need to be followed carefully. One thing to be aware of is that by default Tableau Server currently uses SHA-1 signature algorithm. Many IdP's will have SHA256 as standard. You can also change to SHA256 by running the following TSM command: `tsm configuration set -k wgserver.saml.sha256 -v true` |

