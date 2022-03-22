---
published: false
---
#####
![_gateway_script ]({{ site.baseurl }}/images/formdata.png)

In realtime situation of an IT services from development phases to production the all time lifesaver would be the "logging mechanism" in place, to avoid stress in development lifecycle and chaos after moving our application or apis onto production

Even I have got this situation in my project and I have implemented this common Logging requirement for one of the reputed banks a while ago, so i thought to share this, so you can tweak as per you requirements.

As of now we have two major version in IBM API Connect like V5 and V2018, the major difference with these new version are gateways, so this policy can be applied on the V5 compatibility Gateways only.

Here we go..

If we understand the internals of the API Connect, when the request lands on the data power gateway the request is parsed and stores required information of the api transaction from source ip address to target url and all the required transformation data for the analytics purpose of that context in context variables.

So if we can access this context variables from our logging policy hence we can log as per our requirements. Fortunately IBM has provided all those details in their product documentation very clearly.
Have a glance on the official documentation "Context Variables" to have better understanding of available variables.

In API Connect we can make all the reusable logic into Policies, by this we can have leverage to drag and drop the required policy (in our case its common logging policy).

Implementation of the Policy.

Step: 1

As the initial step, we have to finalize the information of the API call which we are interested to log.

Step: 2

By default the Product provides options to write the log into multiple destinations, choose as per your requirement..

Write log to File on the data power device and upload to ftp/sftp.
Configure log target in data power and write log target to dispatch those logs using any external log servers, like Splunk.
Capture the logs as JSON message and put that in IBM MQ queue for later processing. (In our case we are using this option to log the data)
You can explore more options in IBM Data power.

Step: 3

With the above information in hand, we have to create script to run in IBM API Connect Gateway, this will ideally run in Data power which OTB supports Gateway Script (IBM proprietary) which is javascript , and XSLT.

we will be doing this in Gateway Script.

![_gateway_script ]({{ site.baseurl }}/images/s1.png)

Here we have created a function called commonlogging, and required the "apim.custom.js" module, by this we can access the gateway script functions using the key word apic. (this can be anything). we even required URL open module which we need further to put our log message in MQ queue.

To make the policy dynamic we have to provide provision to get values from user configuration, this can be accommodated using yaml definitions which the product supports otb.

In our scenario, I would like to log the request payload, response payload and log level based on the user configuration. For that I have defined commonlog Yaml definition file. which i will be showing a bit later.

For now just focus on the "props" variable where we are getting all the policy property context using the getPolicyProperty method of GWS(gateyscript). So in the next line we can see how to fetch values.

with continuation to the above function..,

![_gateway_script ]({{ site.baseurl }}/images/s2.png)

In the above snippet we are trying to get all the required information which we finalised in step 1 and assign to variables.

![_gateway_script ]({{ site.baseurl }}/images/s3.png) 

In the above block, we are assigning the request or response payloads conditionally to the variables.

Now construct the Json message as per you requirement needs, here is as per my needs.
 //construct log json message.
 ```
     var log = {

"LogMessage": {
"MetaData": {
"Level": logLevel,
"Timestamp": apicTimestamp,
"LogPointID": "OrigRequest",
"GlobalTranID": GlobalTranID,
"GwyTranID": "Source = APIC Generated Transaction ID (or) HTTP Request Header",
"LogSource": LogSource,
"TranDuration": "time diff between last logging step to now",
"APIName": apiName,
"APIVersion": apiVersion,
"InterfaceName": InterfaceName,
"StatusCode": ResponseCode,
"StatusDesc": ResponseReason,
"ProvRefNum": "Source = Final Response from Backend"
},
"APIC": {
"DevOrgName": DevOrgName,
"ClientAppName": ClientAppName,
"GwyID": "Source = DP System Variable",
"OperationID": OprId,
"Catalog": Catalog,
"InURL": InURL,
"OutURL": OutURL
},

"QueryParam": {

},
"HTTPRequestHeaders": {

},
"HTTPResponseHeaders": {

},
"OrigRequestPayload": {

},
"FinalResponsePayload": {

},
"BackendRequestPayload": {

},
"BackendResponsePayload": {

},
"ErrorResponsePayload": {

},
"Misc": {

}
}
} ```

![_gateway_script ]({{ site.baseurl }}/images/s4.png) 
![_gateway_script ]({{ site.baseurl }}/images/s5.png) 


In the below snippet, check the whether required fields are available and log to console to be aware of that, it is completely as per the requirement.
    // Check if the properties are not retrieved
    ```
    if (props === undefined) {
        console.error('The policy properties were not retrieved for policy: CommonLog.');

    }

    if (apiName === undefined || apiName === '') {
        console.error('The name of the API for this call could not be determined.');

    }

    if (ClientAppName === undefined || ClientAppName === '') {
        console.error('The name of the ClientAppName for this call could not be determined.');

    }

    if (DevOrgName === undefined || DevOrgName === '') {
        console.error('The name of the DevOrgName for this call could not be determined.');

    }

    if (RequestPayload === undefined || RequestPayload === '') {
        console.error('The RequestPayload of the API for this call could not be determined.');
    }

    if (ResponsePayload === undefined || ResponsePayload === '') {
        console.error('The ResponsePayload of the API for this call could not be determined.');
    }

    if (ResponseCode === undefined || ResponseCode === '') {
        console.error('The ResponseCode of the API for this call could not be determined.');
    }

    if (ResponseReason === undefined || ResponseReason === '') {
        console.error('The ResponseReason of the API for this call could not be determined.');
    }

    if (PlanName === undefined || PlanName === '') {
        console.error('The PlanName of the API for this call could not be determined.');
    }

    if (RequestUri === undefined || RequestUri === '') {
        console.error('The RequestUri of the API for this call could not be determined.');
    }

    if (RequestInTime === undefined || RequestInTime === '') {
        console.error('The RequestInTime of the API for this call could not be determined.');
    }

//open connection to mq, in your case the url will change.
t
  options = { target: 'dpmq://' + session.parameters.qm + '/?',
     requestQueue: session.parameters.requestQAlwaysReply,
       //replyQueue: session.parameters.replyQ,
    transactional: true,
             //sync: false,
         asyncPut: true,
          timeOut: 10000,
             data: log };

urlopen.open (options, function (error, response) {
    if(error){
        console.error("Unable to open connection to MQ" + error);
    }
} );


    return;
}

commonlogging();

```

In the above snippet by using url-open module, we have opened target to MQ target url and post message into queue.

Step: 4

As discussed we need Yaml definition file to define properties, Here is the one which i used with above policy
```
policy: 1.0.0

info:
  title: Custom Common Log
  name: commonlog
  version: 1.0.0
  description: A policy to log a custom fields for the api transaction.
  contact:
    name: someCompany
    url: https://somebank.co.in
    email: abc@somebank.com

attach:
    - rest
    - soap

properties:
    $schema: "http://json-schema.org/draft-04/schema#"
    type: object
    properties:
        logLevel:
            label: logLevel
            description: The desired level of logging. Note that the default datapower 
                         log contains messages with > error severity.
            enum:
                - info
                - notice
                - warn
                - error
                - critical
                - alert
                - emerg
                - log
                - trace
            type: string
            default: error
        LogPointID:
            lable: LogPointID
            description: This field captures the position of the log captured 
                          in the transaction-flow.
            enum:
                - OrigRequest
                - FinalResponse
                - ErrorResponse
                - ExtErrorResponse
                - BackendRequest
                - BackendResponse
            type: string    
        QMGR:
            lable: QMGR
            description: Enter the QueueManager Name to send log messages.
            type: string
            default: QMGR
        TargetQueue:
            lable: TargetQueue
            description: Enter the target MQ Request Queue name.
            type: string
            default: QueueName            
        logRequest:
            lable: logRequest
            description: By enabling this feature, Original Request will be logged. 
                         Default is disabled.
            enum:
                - enabled
                - disabled
            type: string
            default: disabled
        logResponse:
            lable: logRequest
            description: By enabling this feature, Final Response will be logged. 
                          Default is disabled.
            enum:
                - enabled
                - disabled
            type: string
            default: disabled
    required:
        - logLevel
        - logRequest
        - logResponse
        - LogPointID
        - QMGR
        - TargetQueue

gateways:
    - datapower-gateway
    
    ```
    
Step: 5

Once we are done with above steps, now we have all the artefacts to create policy.

This is the important step which we fail at initial phases util we are hands on with it.

To Create policy, we have to create Processing policy and Actions in Data power, even if you are not familiar with Data power its fine if you can walk through with this post.

Login to Data power and switch to any domain where you have access to create objects.
Create Folder in Data power file management as policy > commonlog as folder name and upload the js file which we have created in earlier steps.
Create Transformation Action and refer this file location path in the action.
Now create Processing Rule with the name same as policy name and provide extension as "-main"
it looks like this.. commonlog-main. and refer the action which you have created in the above step.
Make the input and output context as NULL since we are not manipulating request and responses.

Step: 6

Now we are done with creating of objects, time to export the configuration.

On the home screen of datapower , navigate to export and import configuration and select processing rules in the search and import the configuration of processing policy "commonlog-main" and actions "commonlog_actions" with referred file as well.
By the above step you will getting export.zip in your downloads. Keep this folder safe, we need this later.

Now it's time to Package the Policy and import into IBM API Connect.

Here you need to focus on the folder structure, if it is not in proper structure the upload of policy throws errors and fails.

Packaging Policy:

Create a folder with name as "implementation" this can be changed, and place the export.zip folder which we have downloaded in earlier step in the implementation folder.
Place the Yaml definition file outside the folder.

Select the two file (implementation folder and yaml file) and make it as zip folder and rename it as our policy name: as (commonlog.zip)
Now we have created policy and set to upload into api connect.

Uploading Policy into API Connect.

Login into API Connect and navigate to Catalogs.
Select Environments on the left panel and choose policies tab.
Click on Upload and upload policy.
Post success, navigate to assembly section and check on the left policies panel, you will be finding our commonlog policy.
Drag and drop in the flow where-ever required and configure the properties as per your configurations.
yes, its a long post,,,, thanks for being till the end.. I hope it helps to quick start things.

you can check for more details on IBM Knowledgecenter APIC #UDP