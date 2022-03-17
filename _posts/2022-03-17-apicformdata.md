---
published: true
---
### How to proxy mimetype application/x-www-form-urlencoded in APIC

Usecase: To expose formdata apis in IBM APIC, we can do it oob by writing Gateway Script in assemby section.
 Step1: Add gateway script action before invoke and change content-type to formdata. (executed in apicv10 & 2018)
 ```context.message.header.set('Content-Type',  'application/x-www-form-urlencoded') 
//sent req data if needed  
//assign actual request data     
var reqBody = 'grant_type=client_credentials'
context.message.body.write(reqBody)```
![gatewayscript]({{site.baseurl}}/_posts/Screenshot 2022-03-17 at 12.36.15 PM.png)
