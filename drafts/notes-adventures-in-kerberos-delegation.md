---
description: My adventures with kerberos delegation to a Presto data source
---

# Draft Recipe: Tableau and Presto with Kerberos sauce

The challenge if you wish to accept it is to enable Kerberos authentication to a Presto data source. 

### The environment

Tableau Server - Windows Server 2016 VM

Tableau Desktop - Windows 10 VM

AD-VM - Windows Server 2016 DC VM

Presto Container 

SQL on VM

The environment is a Active Directory joined, Windows Tableau server with users authenticating using SAML via OneLogin. 

### The first thing I learned

I'd bundled everything Kerberos'y into 1 bucket but Kerberos **User** Authentication is separate from Kerberos **Data** authentication. Authenticating to data sources is actually Kerberos Constrained Delegation which personally I am more familiar with being a reverse proxy access scenario. Review the Tableau content [here](https://help.tableau.com/current/server/en-us/kerberos_keytab.htm#datasource-delegation)

_You can also use Kerberos delegation to access data sources in an Active Directory. In this scenario, users can be authenticated to Tableau Server with any supported authentication mechanism \(SAML, local authentication, Kerberos, etc\), but can access datasources that are enabled by Kerberos._

[Enable Kerberos Delegation](https://help.tableau.com/current/server/en-us/kerberos_delegation.htm) provides more detailed guidance.

### 

I'll layer on the ingredients in this recipe

### Deploying Tableau on Azure

Much to my frustration I did the usual and didn't RTFM, as obviously I know how to install Tableau on Azure. Comfortable with both of them, naturally. Unfortunately, I **repeatedly** made the mistake of not signing in correctly to the Tableau Services Manager \(TSM\) UI. I signed in as the local administrator, and not with my domain account \(domain\admin\) and therefore the installation failed. The error kept on pointing towards the runas account, however it took me a while to figure it out!

### Setup SSL on the Tableau Server

I am a big fan of the Github project Posh-ACME which leverages Lets Encrypt to allow me to generate certificate request using PowerShell on a Windows Server.

{% embed url="https://github.com/rmbolger/Posh-ACME" %}

  
[https://abouconde.com/2020/06/28/warning-unable-to-download-from-uri-https-go-microsoft-com-fwlink-linkid627338clcid0x409-to-warning-unable-to-download-the-list-of-available-providers-check-your-internet-connect/](https://abouconde.com/2020/06/28/warning-unable-to-download-from-uri-https-go-microsoft-com-fwlink-linkid627338clcid0x409-to-warning-unable-to-download-the-list-of-available-providers-check-your-internet-connect/)

Because you didn't specify a plugin, it will default to using the `Manual` DNS plugin. That manual plugin will also be prompting you to create a DNS TXT record to answer the ACME server's validation challenge for the domain.

![](../.gitbook/assets/image%20%282%29.png)

![](../.gitbook/assets/image%20%283%29.png)

The password on the PFX files is `poshacme` because we didn't override the default with `-PfxPass` or `-PfxPassSecure`.

{% embed url="https://help.tableau.com/current/server/en-us/saml\_requ.htm\#Cert\_Name" %}





{% embed url="https://www.onelogin.com/developer-signup" %}

The instructions to follow for setting up Server-wide SAML are here:

{% embed url="https://help.tableau.com/current/server/en-us/config\_saml.htm" %}







### 

### Presto DB

Setup the container from Docker Hub:

{% embed url="https://hub.docker.com/r/ahanaio/prestodb-sandbox" %}

**Tableau Desktop: 2020.3**

\*\*\*\*[https://help.tableau.com/current/pro/desktop/en-us/examples\_presto.htm](https://help.tableau.com/current/pro/desktop/en-us/examples_presto.htm)

1. See your server vendor documentation to identify the latest compatible driver for your server.
2. If your  vendor doesn't supply such a driver, download the JDBC driver from the Presto site.  - If you are connecting to PrestoDB, download the appropriate driver from the [PrestoDB ](https://prestodb.io/download.html)page. - If you are connecting to PrestoSQL, download the appropriate driver from the [PrestoSQL ](https://prestosql.io/download.html)page.
3. Install the file in this folder \(you may have to create it manually\): ~/Library/Tableau/Drivers

**Resources**

* [Connect Tableau to Presto](http://onlinehelp.tableau.com/current/pro/desktop/en-us/examples_presto.htm)

Simba Presto Kerberos [https://www.simba.com/products/Presto/doc/JDBC\_InstallGuide/content/jdbc/pr/authenticating/kerberos-intro.htm](https://www.simba.com/products/Presto/doc/JDBC_InstallGuide/content/jdbc/pr/authenticating/kerberos-intro.htm)

My Container:

[http://tableau-presto.westus2.azurecontainer.io:8080](http://tableau-presto.westus2.azurecontainer.io:8080/ui/)

JDBC Driver - [https://vertica.com/docs/9.2.x/HTML/Content/Authoring/ConnectingToVertica/InstallingDrivers/MacOSX/InstallingTheJDBCDriverOnMacOSX.htm](https://vertica.com/docs/9.2.x/HTML/Content/Authoring/ConnectingToVertica/InstallingDrivers/MacOSX/InstallingTheJDBCDriverOnMacOSX.htm)

Presto Web Connector for Tableau: [https://prestodb.io/docs/current/installation/tableau.html](https://prestodb.io/docs/current/installation/tableau.html)

Install Presto on Linux: [https://optimalbi.com/setting-up-prestodb-on-linux/](https://optimalbi.com/setting-up-prestodb-on-linux/)

