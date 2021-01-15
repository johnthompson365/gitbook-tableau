# Draft Recipe: SAML with Onelogin

### Configuring Server-wide SAML with OneLogin

First off, sign up for a free OneLogin developer account.

{% embed url="https://www.onelogin.com/developer-signup" %}

### Tableau Server Configuration:

{% embed url="https://help.tableau.com/current/server/en-us/config\_saml.htm" %}



![](../.gitbook/assets/image%20%287%29.png)

### Certificates:

I chose to use the same certificates I used for the Server SSL

In addition to [all the normal requirements](https://help.tableau.com/current/server/en-us/saml_requ.htm#Certific) for your SSL certificate you [also need to ensure that](https://help.tableau.com/current/server/en-us/saml_requ.htm#Cert_Name) your certificate for SAML only includes the certificate that applies to Tableau Server and not any other certificates or keys.

### Onelogin Directories

Onelogin domains, ip addresses and ports are listed [here](https://onelogin.service-now.com/support?id=kb_article&sys_id=e1899821db8c60103de43e0439961952&kb_category=a0b5e130db185340d5505eea4b961957)... _Use **IP whitelisting** for on-premises agents, like Active Directory Connector_

![](../.gitbook/assets/image%20%2811%29.png)

I inputted a password and it created the account in AD:

![](../.gitbook/assets/image%20%288%29.png)

![](../.gitbook/assets/image%20%289%29.png)

Question: has this any special permissions?

![](../.gitbook/assets/image%20%284%29.png)

What is OneLogin Desktop SSO - [https://onelogin.service-now.com/kb\_view\_customer.do?sysparm\_article=KB0010313](https://onelogin.service-now.com/kb_view_customer.do?sysparm_article=KB0010313)

![](../.gitbook/assets/image%20%286%29.png)

![](../.gitbook/assets/image%20%2816%29.png)

![](../.gitbook/assets/image%20%2814%29.png)

Even though the ADC sync was communicating with the Onelogin service. I did not have any new users in the directory. When I check the logs in C:\Program Data I was getting an error about missing attributes.

![](../.gitbook/assets/image%20%2813%29.png)

These attributes are required and luckily I knew that my lab users straight away didn't have an email address. 

![](../.gitbook/assets/image%20%2815%29.png)

I added the email address for one of my AD users re-synced and away we went!

![](../.gitbook/assets/image%20%285%29.png)

### Onelogin Apps and SAML

There are a lot of Tableau apps in the Onelogin directory. I selected the Tableau Server app.

![](../.gitbook/assets/image%20%2812%29.png)

 

