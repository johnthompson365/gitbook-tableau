---
description: My adventures with kerberos delegation to a Presto data source
---

# Notes: Adventures in Kerberos Delegation

The challenge if you wish to accept it is to enable Kerberos authentication to a Presto data source. 

The environment is a Active Directory joined, Windows Tableau server with users authenticating using SAML via OneLogin. 

### The first thing I learned

Kerberos User Authentication is separate from Kerberos Data authentication. Authenticating to data sources is actually Kerberos Constrained Delegation which I am more familiar with being a remote proxy access scenario. Review the Tableau content [here](https://help.tableau.com/current/server/en-us/kerberos_keytab.htm#datasource-delegation)

_You can also use Kerberos delegation to access data sources in an Active Directory. In this scenario, users can be authenticated to Tableau Server with any supported authentication mechanism \(SAML, local authentication, Kerberos, etc\), but can access datasources that are enabled by Kerberos._

[Enable Kerberos Delegation](https://help.tableau.com/current/server/en-us/kerberos_delegation.htm) provides more detailed guidance.



