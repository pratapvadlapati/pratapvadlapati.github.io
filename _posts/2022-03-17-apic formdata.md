---
published: true
---
##### How to proxy mimetype application/x-www-form-urlencoded in APIC

**Usecase**: To expose formdata apis in IBM APIC, we can do it oob by writing Gateway Script in assemby section.

 1. Add gateway script policy before invoke and set content-type to formdata. (executed in apicv10 & 2018)
 
 In this example, the exposed api required a param client_credential in formdata. So am sending it, you can send as much data as you can as form data is a key-value pairs.


![_gateway_script ]({{ site.baseurl }}/images/formdata.png)

2. Drag invoke policy and configure your target url (which the service accepts formdata)

![_assembly_pallet ]({{ site.baseurl }}/images/assembly.png)

This is a generic use case, based on this we can improvise as per the requirement.
