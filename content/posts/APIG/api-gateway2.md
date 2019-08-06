---
title: "Control API Usage with API keys"
date: 2018-12-03T08:23:19+10:00
draft: false
tags:
  - AWS
  - API
  - Serverless
---

*In the [previous post]({{< relref path="api-gateway1.md" >}}) we created a basic API with API Gateway, Lambda and DynamDB*

---

Here is becomes obvious why we leave this till the end.  Even if you are highly proficient with Swagger and its integration into a SAM template you will still find it easier to generate the swagger template once the project is complete and drop it into the SAM template.

## Preparation

### 1. Generate Swagger of existing API

Open the API stage that is deployed, click on the *'export'* tab and choose *"Export as swagger + API Gateway extensions"*.

### 2. Modify your stack template
Since we are changing the configuration of the API resource, the generic one provided implicitly for us by SAM is not enough.  We use the Swagger specification to control the API in a more fine-grained way.

Copy your Swagger document into the SAM template as shown below.

Add a new *"Resource"*:
```diff
   ...
+  ApiGatewayApi:
+    Type: AWS::Serverless::Api
+    Properties:
+      StageName: Prod
+      DefinitionBody:  # Paste generated swagger in here
+        swagger: 2.0
+        info:
+          title:
+            Ref: AWS::StackName
+        paths:
+          /ping:
+            get:
+              responses: {}
+              x-amazon-apigateway-integration:
+                httpMethod: POST
+                type: aws_proxy
+                uri:
+                  Fn::Sub: arn:aws:apigateway:${AWS::Region}:lambda:path/2015-03-31/functions/${ping.Arn}/invocations
   ...
```
Adapt the Swagger document buy:

- a) removing the keys: `schemes`, `host` and `basePath`
- b) Replacing the `title` and `uri` values with `!Ref` values

Notice that Cloudformation intrinsic functions - in their expanded form - are available for use here since we have embedded it within the SAM template.

Now change your lambda to use this API resource rather than the implicit one.

```diff
         ...
         GETEndpoint:
           Type: Api
           Properties:
+            RestApiId: !Ref ApiGatewayApi
             Path: /ping
             Method: GET
         ...
```

Don't forget to change your exports :

```diff
...
 Outputs:
   ApiURL:
     Description: API endpoint URL
     Export:
       Name: !Sub "${AWS::StackName}-Endpoint"
     Value: !Join
       - ''
       - - https://
-        - !Sub "${ServerlessRestApi}"
+        - !Ref ApiGatewayApi
         - '.execute-api.'
         - !Ref 'AWS::Region'
         - '.amazonaws.com/'
+        - !Ref ApiGatewayApi.Stage
    ...
```

Now redeploy to ensure the API is unchanged.

> If you receive the error ` Invalid permissions on Lambda function`. You may have incorrectly referenced the lambda in your Swagger template.  If you are using `AutoPublishAlias` then you should refer to that alias by `${ping.Alias}` whereas if you do not publish an alias, simple use the Arn: `${ping.Arn}`

### Creating API keys and a Usage Plan
Having converted to Swagger, it is easy to add a requirement for API keys.  (I hope its starting to become clear why we left this until the API specs were locked in!)

Add the Cloudformation for the required components

- [AWS::ApiGateway::ApiKey](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-apigateway-apikey.html)
- [AWS::ApiGateway::UsagePlan](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-apigateway-usageplan.html)
- [AWS::ApiGateway::UsagePlanKey](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-apigateway-usageplankey.html)

See the the commit [here](https://github.com/eL0ck/aws-APIG/commit/5c0d153e8428a8aa1f4dc505b481400bc95d2848).

### Using the API key
Although the API keys are attached to the API, they are not yet enabled for the method.

Make the following changes to your API specification:

```diff
    ...
    ApiGatewayApi:
      Type: AWS::Serverless::Api
      Properties:
        StageName: Prod
        DefinitionBody:
          swagger: 2.0
          info:
            title:
              Ref: AWS::StackName
          paths:
            /ping:
              get:
                responses: {}
+               security:
+                 - api_key: []
                x-amazon-apigateway-integration:
                  uri:
                    Fn::Sub: arn:aws:apigateway:${AWS::Region}:lambda:path/2015-03-31/functions/${ping.Alias}/invocations
                  passthroughBehavior: "when_no_match"
                  httpMethod: "POST"
                  type: "aws_proxy"
+         securityDefinitions:
+           api_key:
+             type: "apiKey"
+             name: "x-api-key"
+             in: "header"
    ...
```


Redeploy.

### Test API Access

Without an API key:

```bash
curl ${ENDPOINT_URL}/ping
{"message":"Forbidden"}
```
With code: 403


With API Key:

```bash
curl -H "x-api-key:RPW10RSPQB4hkxRSX3qYnaqk7aYAD8oD1qed5oWy" ${ENDPOINT_URL}/ping
{"message": "pong", "response": {}}
```


