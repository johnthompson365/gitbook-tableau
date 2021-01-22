# Draft Recipe: SAML with Onelogin

My goal is to setup OneLogin as the Identity Provider for Tableau Server. I want to synchronise users from Active Directory to OneLogin and use those accounts to sign in via SAML to Tableau.

### OneLogin Developer Tenant

First off, sign up for a free OneLogin developer account. This gives me the ability to test out all the features I need.

{% embed url="https://www.onelogin.com/developer-signup" caption="Click the link to get your OneLogin developer account here! " %}

### Tableau Server Identity Store

During Tableau setup I have configured Active Directory as the External Identity store. So before I can authenticate with a user, I need to ensure there is actually a user account created within Tableau. I used the Tableau AD Users and Group Sync to import Adam.Wally from Active Directory. The Tableau username is based on the `sAMAccountName` attribute in AD:

![AD user account properties](../.gitbook/assets/image%20%2824%29.png)

So when imported:

![Tableau Users administration table](../.gitbook/assets/image%20%2819%29.png)

As described [here](https://help.tableau.com/v2020.4/server/en-us/security_auth.htm)...If you configure Tableau Server to use Active Directory during installation, then NTLM will be the default user authentication method. So my test with `Adam.Wally` proved I could authenticate with my AD username and password.

### Active Directory and OneLogin Directories

I did not want to manually populate OneLogin with user accounts, so I saw they had an [Active Directory Connector](https://onelogin.service-now.com/kb_view_customer.do?sysparm_article=KB0010442) and decided to use that.

The Active Directory Connector was straightforward to download and setup. The connections were all outbound so in my lab it just worked straight away, you know how labs do. However, in an Enterprise you will need to configure the network security for your proxy or egress. So for any OneLogin agent, domains, ip addresses and ports are listed [here](https://onelogin.service-now.com/support?id=kb_article&sys_id=e1899821db8c60103de43e0439961952&kb_category=a0b5e130db185340d5505eea4b961957)... _Use **IP whitelisting** for on-premises agents, like Active Directory Connector_

![ADC download option](../.gitbook/assets/image%20%2811%29.png)

Once downloaded the setup is simple. I chose the Create OneLogin Service Account option by inputting a password to setup the account via the wizard.

![ADC setup service account option](../.gitbook/assets/image%20%288%29.png)

![Builtin\Administrators service account](../.gitbook/assets/image%20%2833%29.png)

_...It creates a domain service account named OneLoginADC with Builtin\Administrators credentials in the local Domain..._ This was fine in my lab but I would follow the principle of least privilege in production and there is an article to guide you on how to [Create a Domain Service Account to run Active Directory Connector](https://onelogin.service-now.com/support?id=kb_article&sys_id=e73c3a35dbf0441024c780c74b961909).

The ADC Setup also configures OneLogin Desktop SSO feature, now called [Windows Domain Authentication](https://onelogin.service-now.com/kb_view_customer.do?sysparm_article=KB0010313). This allows for Integrated Windows Authentication if you require it for Windows and Mac desktops. Not a feature I required in my scenario but a default part of the ADC setup.

![Windows Domain Authentication port](../.gitbook/assets/image%20%284%29.png)

The setup is a little confusing here, as it refers to selecting the domains, but also includes the Users container. I just followed with the choice as is.

![Domain selection for sync](../.gitbook/assets/image%20%286%29.png)

Once the setup is complete you go back to the OneLogin portal and is gives you the option to select the OU's/Containers that you would like to synchronise. Always, when I am testing a directory sync I start small. I know I only have 100 test users in my OU so know this should be a pretty quick exercise. I have seen many customers try to sync their entire domain and get into issues with large directories. Start small, test early and then incrementally add if you have 1000s of objects.

![Portal user import option](../.gitbook/assets/image%20%2816%29.png)

![ADC Connector status in portal](../.gitbook/assets/image%20%2814%29.png)

Even though the ADC sync was communicating with the Onelogin service. I did not have any new users in the directory. When I check the logs in `C:\Program Data\OneLogin, Inc\logs` I was getting an error about missing attributes.

![Missing attributes in the logs](../.gitbook/assets/image%20%2813%29.png)

The attributes shown below are required and luckily I knew straight away that my lab users didn't have an email address. 

![Required sync attributes from AD to OneLogin](../.gitbook/assets/image%20%2815%29.png)

I added the email address for one of my AD users re-synced and away we went!

![awally](../.gitbook/assets/image%20%2817%29.png)

So the synchronisation is working, now I just need to setup SAML to test signing in with this user.

### Tableau Server-Wide SAML setup

The standard setup instructions for Tableau Server are below:

{% embed url="https://help.tableau.com/current/server/en-us/config\_saml.htm" %}

The obvious steps apply of defining your entity ID, uploading certificates and sharing of metadata between Tableau as the service provider and OneLogin as the Identity Provider.

![](../.gitbook/assets/image%20%287%29.png)

#### Certificates:

I chose to use the same certificates I used for the Server SSL.

In addition to [all the normal requirements](https://help.tableau.com/current/server/en-us/saml_requ.htm#Certific) for your SSL certificate you [also need to ensure that](https://help.tableau.com/current/server/en-us/saml_requ.htm#Cert_Name) your certificate for SAML only includes the certificate that applies to Tableau Server and not any other certificates or keys.

### Onelogin Apps and SAML

There are a lot of Tableau apps in the Onelogin directory. I selected the **Tableau Server SAML 2.0** app. This was my first mistake ;\)

![](../.gitbook/assets/image%20%2812%29.png)

Once installed I was looking for an option to upload metadata but there isn't \(similar to Okta\). So the only option I had was to define the Server Name and protocol, it seemed strange as normally I would have to input the Entity ID and ACS when configuring Tableau Online with an IdP, but I ploughed on regardless \(mistake \#2\).

![](../.gitbook/assets/image%20%2821%29.png)

The default settings for SAML 2.0, included a Standard Strength Certificate and SHA-1 signing algorithm. All of this information I was able to download in the metadata and then import into Tableau.

![](../.gitbook/assets/image%20%2818%29.png)

The user attribute mappings are pretty simple for Tableau.

![](../.gitbook/assets/image%20%2820%29.png)

After uploading the metadata I applied the pending changes in Tableau which required a service restart. I love 'em. I wanted to test a basic SAML configuration without adding in things like SLO or different client SAML options.

![My favourite progress bar.](../.gitbook/assets/image%20%2823%29.png)

The services restarted and I attempted my first sign in and... FAIL!  
The error I was getting was:

![](../.gitbook/assets/image%20%2826%29.png)

### Troubleshooting

I reset the password in OneLogin and checked that I could sign in. I checked the account in OneLogin and I made sure that I was 'Authenticated by' OneLogin and **not** Active Directory.

![](../.gitbook/assets/image%20%2822%29.png)

**In Tableau I wanted to check the usernames in the Identity Store. Weirdly, I couldn't find a way to do this, as I had enabled Server-Wide SAML and was automatically being redirected to the ONeLogin portal for all authentication requests.**

So I looked at using TSM or tabcmd but couldn't see an obvious cmd that just listed users! So followed these [instructions](https://kb.tableau.com/articles/howto/exporting-user-list) to access the Server Repository which is a PostgreSQL database.

I had to ensure I had network access on port 8060 and run a tsm command first as explained [here](https://help.tableau.com/current/server/en-us/perf_collect_server_repo.htm).

![](../.gitbook/assets/image%20%2828%29.png)



![Connection details](../.gitbook/assets/image%20%2834%29.png)

Once I had connected I could then browse the \_users table:

![The \_users table in all its glory](../.gitbook/assets/image%20%2837%29.png)

So I needed to ensure that I was passing the value in the Name column as the username attribute in SAML \(_adam.wally_\). This corresponds with the sAMAccountName shown at the top of the article in AD and mapped in OneLogin. I'm a big fan of [SAML-tracer for Firefox](https://addons.mozilla.org/en-US/firefox/addon/saml-tracer/) and Chrome. This showed I was passing the correct attribute:

![SAML-tracer SAML summary](../.gitbook/assets/image%20%2825%29.png)

I needed to go and look at the logs on the Tableau side to see if there was anything obvious reported. 

{% embed url="https://help.tableau.com/current/server/en-us/saml\_trouble.htm" %}

To log SAML-related events, `vizportal.log.level` must be set to `debug`. For more information, see [Change Logging Levels](https://help.tableau.com/current/server/en-us/logs_debug_level.htm).

**Don't think this is related to logging but...** _**if you are resetting logging levels for Tableau Server processes, you must stop the server before making the change, and start it applying the pending changes.**_**`tsm stop`** _**`tsm start`**_

```text
tsm configuration set -k vizportal.log.level -v debug
tsm pending-changes apply

tsm configuration set -k vizportal.log.level -d
tsm pending-changes apply
```

Check for SAML errors in the following files in the unzipped log file snapshot:`\vizportal\vizportal-<n>.log`

![Ahh, no valid audiences...?](../.gitbook/assets/image%20%2836%29.png)

So I did some searching for the error message but I wasn't clear what Audiences was referring to. I then did what I probably should have done at the start and seen if someone else had already set this up. I found this [great article](https://medium.com/@kannanmadhav/configuring-saml-for-tableau-server-with-onelogin-3e9a58cb2931) by a colleague of mine Madhav Kannan that identified my issue. At the very start I had chosen the wrong OneLogin Tableau app. I need to have chosen **Tableau Server\(Signed Response\)**.  


![](../.gitbook/assets/image%20%2835%29.png)

This allows me to configure what OneLogin refers to as the SAML [Audience](https://support.okta.com/help/s/article/Common-SAML-Terms?language=en_US), but we reference as the value  Entity ID.

![Remember this from the Tableau SAML configuration?](../.gitbook/assets/image%20%2839%29.png)

So this time the OneLogin app makes more sense.

![](../.gitbook/assets/image%20%2832%29.png)

### Single Logout

_...For the SAML sign-out redirect, if your IdP supports single logout \(SLO\), enter the page you want to redirect users to after they sign out, relative to the path you entered for the Tableau Server return URL..._  
[_https://interworks.com/blog/tkau/2016/03/30/how-configure-tableau-server-saml-onelogin-idp/_](https://interworks.com/blog/tkau/2016/03/30/how-configure-tableau-server-saml-onelogin-idp/)\_\_

### Next Steps:

Import users from OneLogin into Tableau \(maybe turn off SAML to do that if no _cli_ way of doing it\)  
Check with AuthN method as AD and ONelogin in OL  
Send questions to Robin  
Work on SLO  
  
If you have configured Server Wide SAML can you only logon to all accounts using SAML?  
What about the initial admin?  
Do we only use the ACS with TOL, because you have to be specific  
We only need the username passed as an attribute so why do we list displayName? 









