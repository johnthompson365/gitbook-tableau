# Draft Recipe: SAML with Onelogin

My goal is to setup OneLogin as the Identity Provider for Tableau Server. I want to synchronise users from Active Directory to OneLogin and use those accounts to sign in via SAML to Tableau.

### Tableau Server Identity Store:

During Tableau setup I have configured Active Directory as the External Identity store. So before I can authenticate with a user, I need to ensure there is actually a user account created within Tableau. I used the Tableau AD Users and Group Sync to import Adam.Wally from Active Directory. The Tableau username is based on the `sAMAccountName` attribute in AD:

![](../.gitbook/assets/image%20%2819%29.png)

So when imported:

![](../.gitbook/assets/image%20%2818%29.png)



### OneLogin Developer Tenant

First off, sign up for a free OneLogin developer account. This gives me the ability to test out all the features I need.

{% embed url="https://www.onelogin.com/developer-signup" %}

### Active Directory and OneLogin Directories

The Active Directory Connector was straightforward to download and setup. The connections were out bound so in my lab just worked straight away, however in an Enterprise you are likely to need to configure the network security for your proxy or egress. So for any OneLogin agent, domains, ip addresses and ports are listed [here](https://onelogin.service-now.com/support?id=kb_article&sys_id=e1899821db8c60103de43e0439961952&kb_category=a0b5e130db185340d5505eea4b961957)... _Use **IP whitelisting** for on-premises agents, like Active Directory Connector_

![](../.gitbook/assets/image%20%2811%29.png)

Once downloaded the setup is simple. I chose the Create OneLogin Service Account option by inputting a password to setup the account via the wizard.

![](../.gitbook/assets/image%20%288%29.png)

![](../.gitbook/assets/image%20%289%29.png)

### Question: has this any special permissions?

![](../.gitbook/assets/image%20%284%29.png)

### What is OneLogin Desktop SSO - [https://onelogin.service-now.com/kb\_view\_customer.do?sysparm\_article=KB0010313](https://onelogin.service-now.com/kb_view_customer.do?sysparm_article=KB0010313)

![](../.gitbook/assets/image%20%286%29.png)

Once the setup is complete you go back to the OneLogin portal and is gives you the option to select the OU's/Containers that you would like to synchronise. Always, when I am testing a directory sync I start small. I know I only have 100 test users in my OU so know this should be a pretty quick exercise. I have seen many customers try to sync their entire domain and get into issues with large directories. Start small, test early and then incrementally add if you have 1000s of objects.

![](../.gitbook/assets/image%20%2816%29.png)

![](../.gitbook/assets/image%20%2814%29.png)

Even though the ADC sync was communicating with the Onelogin service. I did not have any new users in the directory. When I check the logs in `C:\Program Data\OneLogin, Inc\logs` I was getting an error about missing attributes.

![](../.gitbook/assets/image%20%2813%29.png)

The attributes shown below are required and luckily I knew that my lab users straight away didn't have an email address. 

![](../.gitbook/assets/image%20%2815%29.png)

I added the email address for one of my AD users re-synced and away we went!

![](../.gitbook/assets/image%20%2817%29.png)

So the synchronisation is working, now I just need to setup SAML to test signing in with this user.

### SAML

The standard setup instructions for Tableau Server are below:

{% embed url="https://help.tableau.com/current/server/en-us/config\_saml.htm" %}



![](../.gitbook/assets/image%20%287%29.png)

### Certificates:

I chose to use the same certificates I used for the Server SSL.

In addition to [all the normal requirements](https://help.tableau.com/current/server/en-us/saml_requ.htm#Certific) for your SSL certificate you [also need to ensure that](https://help.tableau.com/current/server/en-us/saml_requ.htm#Cert_Name) your certificate for SAML only includes the certificate that applies to Tableau Server and not any other certificates or keys.

### Onelogin Apps and SAML

There are a lot of Tableau apps in the Onelogin directory. I selected the Tableau Server app.

![](../.gitbook/assets/image%20%2812%29.png)

 

