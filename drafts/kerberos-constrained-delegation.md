# Draft: Kerberos Constrained Delegation

### Goal

My goal is to configure KCD to a SQL server data source, with the user authentication being [SAML with OneLogin](https://app.gitbook.com/@johnthompson365/s/tableau/~/drafts/-MSUgSon0V7lwJisPCjI/authentication/draft-recipe-saml-with-onelogin). My servers and workstations are all joined to an Active Directory domain and I am using AD as the external Identity store.

### Requirements

The requirements for [Enabling Kerberos Delegation](https://help.tableau.com/current/server/en-us/kerberos_delegation.htm) authentication to a database for Tableau are:

* The Tableau Server information store must be configured to use LDAP - Active Directory.
* The computer where Tableau Server is installed must be joined to Active Directory domain.
* A domain account must be configured as the Run As service account on Tableau Server.
* Delegation configured. Grant delegation rights for the Run As service account to the target database Service Principal Names \(SPNs\).

If I read this [Enabling Kerberos Delegation for SQL Server](https://community.tableau.com/s/question/0D54T00000CWcplSAD/enabling-kerberos-delegation-for-sql-server?_fsi=JnpHaLWS&_fsi=JnpHaLWS&_ga=2.182895846.1348344596.1612210017-159812869.1601602564&_fsi=JnpHaLWS) the first step it tells me to configure Kerberos authentication on the server. However I assumed you wouldn't require this to be configured for KCD to a data source. If I then review the guidance on keytab requirements it refers to [Data Source Delegation](https://help.tableau.com/current/server/en-us/kerberos_keytab.htm#datasource-delegation) four key points

1. The computer account for Tableau Server \(Windows or Linux\) must be in Active Directory domain.
2. The keytab file that you use for Kerberos delegation can be the same keytab that you use for Kerberos user authentication \(SSO\).
3. The keytab must be mapped to the service principal for Kerberos delegation in Active Directory.
4. You may use the same keytab for multiple data sources.

We also have the article [Use SAML SSO with Kerberos Database Delegation](https://help.tableau.com/current/server/en-us/saml_with_kerberos.htm), which doesn't refer to enabling kerberos as a user authentication scheme. However it does refer to using _...the Tableau Server keytab_ to access the database.   
  
the guidance in the [Configuring Kerberos authentication on Tableau server running on Windows](https://medium.com/@tableauman/configuring-kerberos-authentication-on-tableau-server-1917d127b6e3) article from my colleague Andrija Marcic 

So, my conclusion so far is I just require the keytab configuration for the delegation as I am using SAML SSO...

#### Supported Scenarios:

* Active Directory Kerberos

#### Unsupported Scenarios:

* MIT Kerberos

### SQL and SPN's

#### [Register a Service Principal Name for Kerberos Connections](https://docs.microsoft.com/en-us/sql/database-engine/configure-windows/register-a-service-principal-name-for-kerberos-connections?view=sql-server-ver15)

> ...A Service Principal Name \(SPN\) must be registered with Active Directory, which assumes the role of the Key Distribution Center in a Windows domain. The SPN, after it's registered, maps to the Windows account that started the SQL Server instance service. If the SPN registration hasn't been performed or fails, the Windows security layer can't determine the account associated with the SPN, and Kerberos authentication isn't used...

I have SQL installed and joined to an AD domain, and currently set to the defaults. The accounts that are running the services are local [Virtual Accounts](https://docs.microsoft.com/en-us/sql/database-engine/configure-windows/configure-windows-service-accounts-and-permissions?view=sql-server-ver15#New_Accounts) which are auto-managed and can logon to the network and register SPN's.

![](../.gitbook/assets/image%20%2844%29.png)

> **Security Note:** Always run SQL Server services by using the lowest possible user rights. Use a [MSA](https://docs.microsoft.com/en-us/sql/database-engine/configure-windows/configure-windows-service-accounts-and-permissions?view=sql-server-ver15#MSA), [gMSA](https://docs.microsoft.com/en-us/sql/database-engine/configure-windows/configure-windows-service-accounts-and-permissions?view=sql-server-ver15#GMSA) or [virtual account](https://docs.microsoft.com/en-us/sql/database-engine/configure-windows/configure-windows-service-accounts-and-permissions?view=sql-server-ver15#VA_Desc) when possible. When MSA, gMSA and virtual accounts are not possible, use a specific low-privilege user account or domain account instead of a shared account for SQL Server services. Use separate accounts for different SQL Server services. Do not grant additional permissions to the SQL Server service account or the service groups. Permissions will be granted through group membership or granted directly to a service SID, where a service SID is supported...

I wanted to check the registering of SPN's worked with virtual accounts, so I queried AD for the SPN's related to the SQL server \(screenshot below\), I then deleted those SPN's using:`setspn -D MSSQLSvc/sql-win2019.thompson365.com sql-win2019`and `setspn -D MSSQLSvc/sql-win2019.thompson365.com:1433 sql-win2019`and restarted the SQL Database Engine and found they were recreated. 

We query for the SPN based on the computer name because "_...Services that run as virtual accounts access network resources by using the credentials of the computer account in the format &lt;domain\_name&gt;**\**&lt;computer\_name&gt;**$**..."_

![Using SetSPN](../.gitbook/assets/image%20%2852%29.png)

Tableau SPN config

```text
setspn -s HTTP/tableau-win2016.thompson365.com thompson365\tableausvc
setspn -s HTTP/tableau-win2016 thompson365\tableausvc
```

 

![More SetSPN results this time for Tableau](../.gitbook/assets/image%20%2848%29.png)

### Keytab in my lab

We have a batch file that provides guidance on how to configure your keytab.

{% embed url="https://help.tableau.com/current/server/en-us/kerberos\_keytab.htm\#batch-file-set-spn-and-create-keytab-in-active-directory" %}

```text
ktpass /princ http/tableau-win2016.thompson365.com@thompson365.com -SetUPN /mapuser thompson365\<tableau runas account> /pass <user password> /crypto AES256-SHA1 /ptype KRB5_NT_PRINCIPAL /out tableau.keytab 
```

![](../.gitbook/assets/image%20%2855%29.png)

\*\*\*\*[https://docs.microsoft.com/en-us/archive/blogs/pie/all-you-need-to-know-about-keytab-files?\_fsi=JnpHaLWS](https://docs.microsoft.com/en-us/archive/blogs/pie/all-you-need-to-know-about-keytab-files?_fsi=JnpHaLWS)  
[https://social.technet.microsoft.com/wiki/contents/articles/36470.active-directory-using-kerberos-keytabs-to-integrate-non-windows-systems.aspx](https://social.technet.microsoft.com/wiki/contents/articles/36470.active-directory-using-kerberos-keytabs-to-integrate-non-windows-systems.aspx)

\*\*\*\*

**Put something about the security implications of this....**

![](../.gitbook/assets/image%20%2854%29.png)



![](../.gitbook/assets/image%20%2850%29.png)

![](../.gitbook/assets/image%20%2851%29.png)

