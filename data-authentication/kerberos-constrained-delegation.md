# Kerberos Constrained Delegation

### Requirements

The requirements for [Enabling Kerberos Delegation](https://help.tableau.com/current/server/en-us/kerberos_delegation.htm) authentication to a database for Tableau are:

* The Tableau Server information store must be configured to use LDAP - Active Directory.
* The computer where Tableau Server is installed must be joined to Active Directory domain.
* A domain account must be configured as the Run As service account on Tableau Server.
* Delegation configured. Grant delegation rights for the Run As service account to the target database Service Principal Names \(SPNs\).

### Supported Scenarios:

* Active Directory Kerberos

### Unsupported Scenarios:

* MIT Kerberos

However, if I read this [Enabling Kerberos Delegation for SQL Server](https://community.tableau.com/s/question/0D54T00000CWcplSAD/enabling-kerberos-delegation-for-sql-server?_fsi=JnpHaLWS&_fsi=JnpHaLWS&_ga=2.182895846.1348344596.1612210017-159812869.1601602564&_fsi=JnpHaLWS) the first step it tells me to configure Kerberos authentication on the server. Does a user have to AuthN via kerberos to benefit from KCD? I am not doing those steps just to see how far I get without it.

### Setting up Kerberos for SQL

I am following the guidance in the [Configuring Kerberos authentication on Tableau server running on Windows](https://medium.com/@tableauman/configuring-kerberos-authentication-on-tableau-server-1917d127b6e3) article from my colleague Andrija Marcic to complete the setup. I'll add a few details on the configuration in this article.

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

My servers are all joined to an Active Directory domain and I am using AD as the external Identity store. 

I am testing whether I need to use a keytab or not, as there is conflicting docs out there...  

{% embed url="https://help.tableau.com/current/server/en-us/kerberos\_keytab.htm\#batch-file-set-spn-and-create-keytab-in-active-directory" %}

![](../.gitbook/assets/image%20%2854%29.png)



![](../.gitbook/assets/image%20%2850%29.png)

![](../.gitbook/assets/image%20%2851%29.png)

