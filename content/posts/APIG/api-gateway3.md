---
title: "API Authorization with Cognito"
date: 2019-02-15T08:23:19+10:00
draft: false
tags:
  - AWS
  - API
  - Serverless
---

*In the [previous post]({{< relref path="api-gateway1.md" >}}) we created a basic API with API Gateway, Lambda and DynamDB. Now we'll add Authorization.*

---

# AWS Cognito

In much the same way as we implemented API keys we start by generating a swagger template from the API previously deployed with all the implicit features of the SAM transform. See the [API keys post]({{< relref path="api-gateway2.md" >}}) for the explanation of how to integrate the generated swagger template into the SAM template.

Ensure that the API deploys unchanged using the swagger template.

## Deploy a Cognito User Pool

First we add the cloudformation for the UserPool and the App client.

```yaml
  appClient:
    Type: AWS::Cognito::UserPoolClient
    Properties:
      UserPoolId: !Ref userPool
      ExplicitAuthFlows:
        - ADMIN_NO_SRP_AUTH
        - USER_PASSWORD_AUTH
      GenerateSecret: false
  userPool:
    Type: AWS::Cognito::UserPool
    Properties:
      UserPoolTags:
        String: String
      UsernameAttributes:
        - email
```

### Manually Configure the App Client Settings

Unfortunately it does not seem possible to do this via cloudformation.

Log into the console. Go to the Cognito Service, find your new user pool.  Under 'App Integrations' choose 'App client settings'

- Enable all Identity Providers
- Set your callback and signout URLs and note for later
- Choose 'Authorization code grant' and 'Implicit grant' under **'Allowed OAuth Flows'**
- Select all OAuth scopes
- Save the changes

Now manually configure the domain:

Choose a lowercase string to prefix your domain and save it.  Note it for the authentication step.

### Create a Cognito user

The hosted login for this user pool is at the following address:

```
https://your_domain/login?response_type=code&client_id=your_app_client_id&redirect_uri=your_callback_url
```
Substitute your values in here and browse to it in a browser.  Create a new user.

Finally - if you used a non-existant email- manually confirm it from the Console.  Alternatively, use the confirmation code sent to the email your registered with.

## Wire up your API methods to use the User Pool as an Authorizer

Add the following to the swagger part of the SAM template.

```diff
...
      DefinitionBody:
        swagger: "2.0"
        info:
          title:
            Ref: AWS::StackName
        paths:
          /ping:
            get:
              responses: {}
+             security:
+             - testCogAuthoriser: []
              x-amazon-apigateway-integration:
                uri:
                  Fn::Sub: arn:aws:apigateway:${AWS::Region}:lambda:path/2015-03-31/functions/${ping.Arn}/invocations
                passthroughBehavior: "when_no_match"
                httpMethod: "POST"
                type: "aws_proxy"
+       securityDefinitions:
+         testCogAuthoriser:
+           type: "apiKey"
+           name: "Authorization"
+           in: "header"
+           x-amazon-apigateway-authtype: "cognito_user_pools"
+           x-amazon-apigateway-authorizer:
+             providerARNs:
+             - "arn:aws:cognito-idp:ap-southeast-2:740435148984:userpool/ap-southeast-2_lnsfs8hZw"
+             type: "cognito_user_pools"
```

## Test Access to your API

Try curling the method:
```bash
curl $ENDPOINT_URL/ping
{"message":"Unauthorized"}
```

This is what we expect.  So now, lets use the `Authorization` header we specified in the `securityDefinitions`.

*I use an ipython shell here because I prefer boto3 and pythons JSON handling to the `aws-cli` and `jq` but the commands are almost identical should you choose that.*

```python
In [60]: ENDPOINT_URL = 'https://cxzf1rruwl.execute-api.ap-southeast-2.amazonaws.com/Prod'
In [61]: !curl $ENDPOINT_URL/ping
{"message":"Unauthorized"}
In [62]: import boto3
In [63]: cognito_idp = boto3.client('cognito-idp')
In [64]: idToken = cognito_idp.initiate_auth(
    ...:     AuthFlow='USER_PASSWORD_AUTH',
    ...:     AuthParameters={
    ...:         'USERNAME': 'testuser0@gmail.com',
    ...:         'PASSWORD': 'Password12',
    ...:     },
    ...:     ClientId='3vnkpvi180nrmd44h4gnoj8cs4',        #  App client id
    ...: )['AuthenticationResult']['IdToken']
In [65]: !curl -H "Authorization:$idToken" $ENDPOINT_URL/ping
{"message": "pong", "response": {}}
```

> If you receive the error: `NotAuthorizedException: An error occurred (NotAuthorizedException) when calling the InitiateAuth operation: Unable to verify secret hash for client xxxx` you have created your App client with an 'App Client secret'.  Create another one without it.

This is an example of how a python application would call the API but for more common front-end methods see the [developer guide](https://docs.aws.amazon.com/cognito/latest/developerguide/adding-a-web-or-mobile-app.html) for examples in Javascript, Android and iOS.

## Finally

In this post we've taken a basic API gateway deployment with a single GET method and linked AWS Cognito for user authentication.

For the complete code clone the repo/branch and deploy it yourself:

```bash
git clone https://github.com/eL0ck/aws-APIG.git -b cognito
```
