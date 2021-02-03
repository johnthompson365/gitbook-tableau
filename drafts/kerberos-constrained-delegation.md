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
  
Additionally, some guidance in the [Configuring Kerberos authentication on Tableau server running on Windows](https://medium.com/@tableauman/configuring-kerberos-authentication-on-tableau-server-1917d127b6e3) article from my colleague Andrija Marcic. 



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
setspn -Q */tableau-win2016

setspn -s HTTP/tableau-win2016.thompson365.com thompson365\tableausvc
setspn -s HTTP/tableau-win2016 thompson365\tableausvc

setspn -D HTTP/tableau-win2016.thompson365.com thompson365\tableausvc
setspn -D HTTP/tableau-win2016. thompson365\tableausvc
```

![More SetSPN results this time for Tableau](../.gitbook/assets/image%20%2848%29.png)

### 

\*\*\*\*

**Put something about the security implications of this on the runas account:**

**Without the Tableau HTTP SPN configured you do not get the option to configure delegation.**

![](../.gitbook/assets/image%20%2854%29.png)



![](../.gitbook/assets/image%20%2850%29.png)

### Testing \(breaking it...\)

This great article provides a useful SQL script to confirm your client is using kerberos : [https://www.red-gate.com/simple-talk/sql/database-administration/questions-about-kerberos-and-sql-server-that-you-were-too-shy-to-ask/](https://www.red-gate.com/simple-talk/sql/database-administration/questions-about-kerberos-and-sql-server-that-you-were-too-shy-to-ask/)

<table>
  <thead>
    <tr>
      <th style="text-align:left">Configuration</th>
      <th style="text-align:left">Test</th>
      <th style="text-align:left">Result</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left">Delete keytab</td>
      <td style="text-align:left">Add SQL as a datasource in Desktop and check klist/sql script</td>
      <td
      style="text-align:left">No issues. Keytab not relevant.</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left"></td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">Add SQL as a datasource in Desktop and check klist/sql script</td>
      <td
      style="text-align:left">
        <p>Signs in on the Windows workstation computer.</p>
        <p>Creates a cached kerberos ticket (below)</p>
        </td>
    </tr>
    <tr>
      <td style="text-align:left">Delete Tableau SPN</td>
      <td style="text-align:left">
        <p>Delete SPN whilst SQL service is still running.</p>
        <p>Add SQL as a datasource in Desktop and check klist/sql script</p>
      </td>
      <td style="text-align:left">This had no impact on desktop as it is making a SQL connection not an
        HTTP connection.</td>
    </tr>
    <tr>
      <td style="text-align:left">Remove MSSQL SPN from Runas account</td>
      <td style="text-align:left">Go users and computers and deselect MSSQLSvc from the delegation tab</td>
      <td
      style="text-align:left">
        <p>With the Tableau HTTP SPN deleted, there is not the delegation option
          available to delete anything.</p>
        <p>(However, even without that you can still sign in)</p>
        </td>
    </tr>
    <tr>
      <td style="text-align:left">Delete SQL SPN (<code>setspn -D MSSQLSvc/sql-win2019.thompson365.com sql-win2019)</code>
      </td>
      <td style="text-align:left">Add SQL as a datasource in Desktop and check klist/sql script</td>
      <td
      style="text-align:left">I am able to sign in via Desktop. It doesn&apos;t pick up a kerberos ticket
        and it connects via NTLM</td>
    </tr>
  </tbody>
</table>

When I signed in using Tableau Desktop to the SQL server I received this cached ticket when running klist. 

![A ](../.gitbook/assets/image%20%2857%29.png)

Running the SQL Script shows that the connection from the tableau-desktop workstation is using kerberos.

![](../.gitbook/assets/image%20%2859%29.png)

After I'd removed the Tableau SPN's \(which removes the delegation config too\) the user connection fell back to NTLM. 

![](../.gitbook/assets/image%20%2860%29.png)

After recreating the SPN's and delegation configuration ont he runas account, I needed to reboot the workstation to revert back to Kerberos from NTLM.



### Keytab in my lab

Useful notes in the [article](https://social.technet.microsoft.com/wiki/contents/articles/36470.active-directory-using-kerberos-keytabs-to-integrate-non-windows-systems.aspx) talk about how:

> Kerberos keytabs, also known as key table files, are only employed on non-Windows servers. In a homogenous Windows-only environment, keytabs will not ever be used, as the AD service account in conjunction with the Windows Registry and Windows security DLLs provide the Kerberos SSO foundation.

We have a batch file that provides guidance on how to configure your keytab.

{% embed url="https://help.tableau.com/current/server/en-us/kerberos\_keytab.htm\#batch-file-set-spn-and-create-keytab-in-active-directory" %}

```text
ktpass /princ http/tableau-win2016.thompson365.com@thompson365.com -SetUPN /mapuser thompson365\<tableau runas account> /pass <user password> /crypto AES256-SHA1 /ptype KRB5_NT_PRINCIPAL /out tableau.keytab 
```

![](../.gitbook/assets/image%20%2855%29.png)

\*\*\*\*[https://docs.microsoft.com/en-us/archive/blogs/pie/all-you-need-to-know-about-keytab-files?\_fsi=JnpHaLWS](https://docs.microsoft.com/en-us/archive/blogs/pie/all-you-need-to-know-about-keytab-files?_fsi=JnpHaLWS)  
[https://social.technet.microsoft.com/wiki/contents/articles/36470.active-directory-using-kerberos-keytabs-to-integrate-non-windows-systems.aspx](https://social.technet.microsoft.com/wiki/contents/articles/36470.active-directory-using-kerberos-keytabs-to-integrate-non-windows-systems.aspx)

### Browser IWA

Logging in to browser on Tableau Server and the workstation. Likely IWA issue with browser below. _\(Can’t connect to Microsoft SQL Server Detailed Error Message \[Microsoft\]\[ODBC SQL Server Driver\]\[SQL Server\]Login failed for user 'THOMPSON365\tableau-runas'. Integrated authentication failed. 2021-02-02 **15:02:39.841,** \(YBlpjyzPedSdTWKp5GyAsQAAAuo,1:0\)_

![](../.gitbook/assets/image%20%2856%29.png)





