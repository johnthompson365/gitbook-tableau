# Kerberos Constrained Delegation

### Requirements

The requirements for [Enabling Kerberos Delegation](https://help.tableau.com/current/server/en-us/kerberos_delegation.htm) authentication to a SQL database for Tableau are:

* The Tableau Server information store must be configured to use LDAP - Active Directory.
* The computer where Tableau Server is installed must be joined to Active Directory domain.
* A domain account must be configured as the Run As service account on Tableau Server.
* Delegation configured. Grant delegation rights for the Run As service account to the target database Service Principal Names \(SPNs\).

### Supported Scenarios:

* Active Directory Kerberos

### Unsupported Scenarios:

* MIT Kerberos

### Setting up Kerberos for SQL

I am following the guidance in the [Configuring Kerberos authentication on Tableau server running on Windows](https://medium.com/@tableauman/configuring-kerberos-authentication-on-tableau-server-1917d127b6e3) article from my colleague Andrija Marcic.

#### [Register a Service Principal Name for Kerberos Connections](https://docs.microsoft.com/en-us/sql/database-engine/configure-windows/register-a-service-principal-name-for-kerberos-connections?view=sql-server-ver15)

_"...A Service Principal Name \(SPN\) must be registered with Active Directory, which assumes the role of the Key Distribution Center in a Windows domain. The SPN, after it's registered, maps to the Windows account that started the SQL Server instance service. If the SPN registration hasn't been performed or fails, the Windows security layer can't determine the account associated with the SPN, and Kerberos authentication isn't used..."_

I have SQL installed and joined to an AD domain.

![](../.gitbook/assets/image%20%2844%29.png)

When SQL Database Engine starts it attempts to register a Service Principal Name  

[https://docs.microsoft.com/en-us/sql/database-engine/configure-windows/register-a-service-principal-name-for-kerberos-connections?view=sql-server-ver15](https://docs.microsoft.com/en-us/sql/database-engine/configure-windows/register-a-service-principal-name-for-kerberos-connections?view=sql-server-ver15)

