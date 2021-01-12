---
description: My adventures with kerberos delegation to a Presto data source
---

# Notes: Adventures in Kerberos Constrained Delegation

The challenge if you wish to accept it is to enable Kerberos authentication to a Presto data source. 

The environment is a Active Directory joined, Windows Tableau server with users authenticating using SAML via OneLogin. 

### The first thing I learned

Kerberos User Authentication is separate from Kerberos Data authentication. Authenticating to data sources is actually Kerberos Constrained Delegation which I am more familiar with being a remote proxy access scenario. Review the Tableau content [here](https://help.tableau.com/current/server/en-us/kerberos_keytab.htm#datasource-delegation)

_You can also use Kerberos delegation to access data sources in an Active Directory. In this scenario, users can be authenticated to Tableau Server with any supported authentication mechanism \(SAML, local authentication, Kerberos, etc\), but can access datasources that are enabled by Kerberos._

[Enable Kerberos Delegation](https://help.tableau.com/current/server/en-us/kerberos_delegation.htm) provides more detailed guidance.

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



