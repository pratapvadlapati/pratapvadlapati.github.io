---
published: false
---
## A Prototype Design to Expose & verify signature of Webhook-events in IBM 

Enter text in [Markdown](http://daringfireball.net/projects/markdown/). Use the toolbar above, or click the **?** button for formatting help.


### What to Expect
Once the Webhook Provider application sends the event to APIC API, APIC will verify the message is sent by the authorized sender. To verify that, custom code in APIC will generate a signature (using HMAC SHA1) and compare it with the value of ‘signature’ from the HTTP header of the message sent by the Provider app.

### Acronyms
APIC – API Connect Enterprise
Gateway Script – IBM’s proprietary custom language/functions based on JS.

Webhook Event – Data sent to the registered consumer app/url when there is any change.

Assembly Section-Node editor for APIC to build custom use cases 



### OVERVIEW WORKFLOW APPROACH FOR GENERATING, VERIFYING SIGNATURE AND SEND EVENT TO KAFKA.
![webhook1.jpg]({{site.baseurl}}/_posts/webhook1.jpg)

 

#### Notes:
Webhook Provider App:
1.	Once the event triggers the event is sent to APIC with Signature in HTTP-header as X-<custom-name> and payload (event data).
  
****APIC:
1.	APIC receives event as request, and it process the generation of signature and verification of signature received from the http headers for every request.
2.	Once validation is success then, the data is sent to configured Kafka topic.

****Kafka:
1.	Kafka topic receives the event data and do further process to registered applications.



### NODE LEVEL IMPLEMENTATION LAYOUT OF THE API.
Prototype Implementation of an API in assembly section in IBM APIC.
![p1.PNG]({{site.baseurl}}/_posts/p1.PNG)

 

A brief note about each node function in this use case as per the design.
Node Name	Description	Technical Details
Gateway Script	Custom Code	This node receives the original Event request as JSON input message, here we fetch required values to validate or for later use. 
1.	Encode API-Key to utf8 (which is shared by the partner confidentially.)
2.	Encode received payload to utf8
3.	Get the partner-signature from HTTP headers.
4.	Create signature using HMAC SHA1 algorithm and encoded payload.
5.	Verify the Partner-signature and Generated signature.
6.	Send event data to Kafka topic.
  
  
  ### VERIFICATION EXAMPLE IN APIC
   
	//secret message Api key shared by partner
    var partnerApiKey = '2bh7Ptx0pdYn9NWv5a9S1WPaer2BQ3U4R40pygAqnQMy6X2q0q';   
//will be loaded from env variables
	
	//convert api key and received payload into bytes
      var encodedKey = Buffer.from(partnerApiKey);
      var encodedPayload = Buffer.from(strPayload);
	
    //generate signature	(compute hash result and convert to hex)
      var GeneratedSignature = crypto.createHmac('hmac-sha1',  encodedKey).update(encodedPayload).digest('hex');
      
     if(signature !== GeneratedSignature) {
     context.reject('CustomError', 'You are not authorized to make this API call');
     context.message.statusCode = '401 Unauthorized';
    } else {
	//send data to kafka
    const data = {
        responses: [
            {
                pusherEmail: req.pusher.email,
                commitMsg: req.head_commit.message,
                modified: req.head_commit.modified[0]
            }
        ]
    };



