---
published: true
---
## JSON Web Security -- USING IBM APIC

#### Description:

Exchange information between systems securely using JSON Web Security with Digital Signature, Encrypting, Decrypting and Verifying Digital Signature in IBM API Connect. 

  **Integrity**: we make sure data you send arrives at its destination unaltered.  
  
  **Encryption**: we make your data unintelligible while in transit to keep it private.

# Usecase:  
The information typlically JSON messages/requests or responses should be protected from any interruptions while in transit. To achieve this we can opt for JSON Web Security using JOSE module in IBM API Connect with RSA Keys(Public/Private Keys).

# Implimentation: Sign Request
1. Create Crypto Keys/certificate Objects in IBM Datapower.
    ![_cryptoObjDataPower ]({{ site.baseurl }}/images/cryptObj.png)
    fig: 1
    
2. Create an Open API in IBM API Connect to receive request and do digital signature and verification.
   For simplicity, am not showing the creation of the API here.... jsut focussin on the signing and verification process.
    ![_apicAssemblyApi]({{ site.baseurl }}/images/apic_api.png)
    fig: 2
   
   In the above fig2.. I have created seperate endpoints for Sign, Verify, Sign&Encrypt, Decrypt&Verify for the requirement in APIC. 
   
  ####

APIC Paths/endpoints
  
     `**/api/v1/json/sign**          (this endpoint takes json request and signs the request...)
     **/api/v1/json/verify**	       (this endpoint takes json request and verifies digital signature...)
     **/api/v1/json/enc_sign**	     (this endpoint takes json request and encrypts message and does digital signature...)
     **/api/v1/json/enc_verify**	   (this endpoint takes json request and decrypts message and verifies digital signature...)`
     
  3. In Assembly pallete, create gateway script node to write cutsom gatewayscript.
     ![_apicAssembly]({{ site.baseurl }}/images/apic_assembly.png)
     
  Point to be noted...
 
    1. Once we have created the Crypto Objects in Datapower (to be precise, in Datapower Gateways we should'nt create objects directly instead,
    we should create using cli scripts.. am leaving here you can chek on this for more info on how to do this. )
    2. We have to reference the crypto object (Private Key) in the code to do Digital Signature.
        we have 3 option to do this.
        2.1 Refer the Private or Public Key/cert as String base64 encoded.
        2.2 Refer the Private or Public Key/cert as (DataPower)Object reference.
        2.3 Refer the Private or Public Key/cert as JWK as per standard format.
        
        I am using the 2.2 option for this implementation...
        
   4. We will use JOSE module for the JSON Web Security implementation - which is provided by IBM as inbulit moduel, where you can require and use in GW.
      ![_apicAssemblySign]({{ site.baseurl }}/images/apic_sign.png)
      
      Since the code snippet is self explanoatory Am not going through the code.
     
  #####

Point to be noted...
  
    If you notice on the line no 13 ...
    Here in this function CreateJWSHeader we should pass PrivateKey and Alogithm to sign.
    The key here provided is **jsonSignprivkey** ... the actual format to refence the Datapower Object is {name: ObjectName}
    But we should provide only ObjectName as given in line no 13.
  ###

 Sign the Request using Postman.
     ![_apicAssemblySign]({{ site.baseurl }}/images/postman_sign.png)
     
  ##

 Implimentation: Verify Signed Request
 
  1.  In Assembly pallete, create gateway script node to write cutsom gatewayscript.
     ![_apicAssembly]({{ site.baseurl }}/images/apic_verify.png)
  
  ### Verify the Signed Request using Postman.
     ![_apicAssemblySign]({{ site.baseurl }}/images/postman_verify.png)
