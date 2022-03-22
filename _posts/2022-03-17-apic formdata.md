---
published: true
---
### How to proxy mimetype application/x-www-form-urlencoded in APIC

Usecase: To expose formdata apis in IBM APIC, we can do it oob by writing Gateway Script in assemby section.
 Step1: Add gateway script action before invoke and change content-type to formdata. (executed in apicv10 & 2018)
 
 The exposed api required a param client_credential in formdata. So am sending it, you can send as much data as you can as form data is a key-value pairs.


![_gateway_script]({{ site.baseurl }}/images/fordata.png)
![_config.yml]({{ site.baseurl }}/images/config.png)
