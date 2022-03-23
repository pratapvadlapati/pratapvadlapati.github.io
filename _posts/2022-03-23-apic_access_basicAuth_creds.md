---
published: true
---
##### Access Basic Auth creds in gateway script IBM APIC.

Usecase:

In any situation, if we have to access Basic auth username and password in gateway script (IBM API Connect  Api-gateway)

The below snippet can be a reference for further improvisions.

![_gateway_script ]({{ site.baseurl }}/images/basicauth.png)


With xslt we can get those details with below dp fucntion.

<xsl:value-of select="dp:auth-info('basic-auth-name')"/>


<xsl:value-of select="dp:auth-info('ssl-cipher-suite')"/>
