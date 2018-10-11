---
title: "How to write an API"
date: 2018-05-22T08:23:19+10:00
draft: true
tags:
  - AWS
  - API
  - Serverless
---

**Using AWS: (API Gateway, Lambda, DynamoDB)**

Recently I've been helping a data dependent organisation build resilient and scalable, decoupled services to facilitate data transit around the business.  Here are some the lessons I've learned/re-learned while building a number of API's using AWS Serverless Aplication Model (SAM).

## Before starting

### 1. Define Enpoints
    Define input/output format with examples

    This should include samples.

### 2. Define data schema
    I'm using [DynamDB](https://aws.amazon.com/dynamodb/developer-resources/).  As always, read ALL the developer docs.  There is an obvious way to do things and its slow and inefficient.

    Some of the API functions perform simple transformations, when the API is acting as a cache layer.  As such the code can be quite simple.  Decisions made at this point can potentially have a very large impact on performance.

    In these cases I prefer to do the transform on the way it.  That way, the latency sensitive requests can consist of very little code indeed.  All it should do, is conduct a query and send the data out.

### 3. Define Indexes on you tables.
    Dynamo scan operations should never be used in applications.  Create a Global or Local index to query data based on the kind of query you need to do.  These indexes should only return the required fields.  They should make use of the NoSQL feature of sparse fields to automatically filter out irrelevant data without having to explicitly provide a filter condition.


    --- In progress ---

### 4. Setup Basic Framework
    - At first put all lambdas in a single `lambdas` directory
    - Write the basic SAM template
    - Write basic handlers that print event to logs and return success

    Here is a template project with a single endpoint, a python lambda that prints the event to logs and a table.
    [Basic API](https://github.com/eL0ck/aws-APIG)

    [SAM Docs](https://github.com/awslabs/serverless-application-model/blob/develop/versions/2016-10-31.md)

    Hardcode example responses so that you can later write tests to assert output formats.

### 5. Deploy the basic framework

```

```

- Hit the enpoint with CURL or PostMan
- Go to AWS logs for the lambda and copy the received event to local `test_events` for offline testing before deployment

### 6. Setup PostMan
Use Postman to track development progress

- Write postman calls for each of you endpoints.
- Add tests to ensure the format is expected. This become your integration tests and will eventually go into your CI pipeline.

## OnGoing Itterative Development
Now you are ready to build out the functionality of API.  The ideal workflow is:

1. Write the endpoint test
2. Write API code to pass the tests

## Final Steps

Up until this point we have deliberately avoided implimenting authorisers, CORS or APIKEYs.  We have done so because these steps are best integrated once, after the endpoints are clearly defined and tested.  Their definitions in Cloudformation/SAM can greatly increase the complexity of the template and thus lead to mistakes that are difficult to debug during development.

### Add request throttling with APIKeys
Here is becomes obvious why we leave this till the end.  Even if you are highly proficient with Swagger and its integration into a SAM template you will still find it easier to generate the swagger template once the the project is complete and drop it into the SAM template.

1. Generate Swagger of existing API

Open the stage that is deployed, click on the 'export' tab and choose "Export as swagger + API Gateway extensions".


2. Modify your template
Since we are changing the configuration of the API itself, the generic one defined for us by SAM is not enough

Add a new "Resource":
```diff
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
+              responses: {}
```

Also change you lambda to use this APIG resource rather than create a new one:

```diff
         GETEndpoint:
           Type: Api
           Properties:
+            RestApiId: !Ref ApiGatewayApi
             Path: /ping
             Method: GET
```

Don't forget to change your exports :

```
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
         - '.amazonaws.com/Prod'
```

Now redeploy to ensure the same API comes up from the modifications.

### Creating API keys and a Usage Plan
Having converted to Swagger, it is easy to add a requirement for API keys.  (Now its starting to become clear why we left this till the API specs were locked in and tested!)

Add the Cloudformation for the required compenents

[AWS::ApiGateway::ApiKey](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-apigateway-apikey.html)
[AWS::ApiGateway::UsagePlan](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-apigateway-usageplan.html)
[AWS::ApiGateway::UsagePlanKey](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-apigateway-usageplankey.html)

See the example [here](github commit)

Redeploy with new components

### Using the API key
Although the api keys are attached to the API, they are not yet enabled for the method.

Make the following changes to your API specification:

```
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
```


Redeploy and test.

#### Without APIkey

```
curl https://q7rfcxk6j7.execute-api.eu-west-1.amazonaws.com/Prod/ping
{"message":"Forbidden"}
```
With code: 403


#### With APIKey

```
curl -H "x-api-key:RPW10RSPQB4hkxRSX3qYnaqk7aYAD8oD1qed5oWy" https://q7rfcxk6j7.execute-api.eu-west-1.amazonaws.com/Prod/ping
{"message": "pong", "response": {}}
```


