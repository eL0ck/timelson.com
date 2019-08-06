---
title: "How to write an auto-scaling API in 3 hours"
date: 2018-12-02T08:23:19+10:00
draft: false
tags:
  - AWS
  - API
  - Serverless
---


API Gateway is a service from AWS that allows a first time user to create an API with little-to-no knowledge of .... API's.  Yep, its great.  I'm a big fan.

An alternative would be to use a web application framework of you choosing and then go host it, configure load balancing, configure auto-scaling, maintain container services etc.  Yuk.

I've been writing a lot of these lately and recently wrote a simple one to perform some data caching and transformations in 2 and a half hours.  It passed tests the next day at a loads of over 5000 concurrent requests per second.

Here are some the lessons I've learned/re-learned while building a number of API's using AWS Serverless Application Model (SAM).

## Before starting

### 1. Define Endpoints
Clearly define input/output format.  This should include a number of detailed samples.

### 2. Define data schema
Efficient design of the schema is probably the most important thing in the whole process.  If you've absolutely intent on maintaining you ignorance about API's you should still know everything there is to know about the database you're using.  If you don't, you should not have promised a product for verification in 3 hours time.  I'm using [DynamoDB](https://aws.amazon.com/dynamodb/developer-resources/). With Dynamo there is an obvious way to do things and its slow and inefficient.

Some of your API functions may perform simple transformations when the API is acting as a only as a high-throughput cache layer.  As such the code can be quite simple.  Decisions made at this point can potentially have a very large impact on performance.

In these cases I prefer to do the transformations on the way in.  That way, the GET requests can consist of very little code indeed.  All they should do, is conduct a highly specific query of the database and return the JSON object they received from it.

### 3. Define Indexes on you tables.
Dynamo scan operations should never be used in applications.  If you are using them you didn't take me seriously in step 2 or alternatively, you took me too seriously at the title.  Create a Global or Local index to query data based on the kind of query you need to do.  These indexes should only return the required fields.  They should make use of sparse fields to automatically filter out irrelevant data without having to explicitly provide a filter condition.

### 4. Set-up Basic Framework
- At first put all lambdas in a single `lambdas` directory
- Write the basic SAM template
- Write basic handlers that print events to logs and return success

Here is a template project with a single endpoint, a python lambda that prints the event to logs and a table.

- [Basic API](https://github.com/eL0ck/aws-APIG)

Hard-code example responses so that you can later write tests to assert output formats.

### 5. Deploy the basic framework

```bash
ARTIFACT_BUCKET=<your_existing_bucket>
aws cloudformation package \
    --template-file template.yaml \
    --s3-bucket ${ARTIFACT_BUCKET} \
    --s3-prefix <subfolderpath>  \
    --output-template /tmp/packaged.yaml
aws cloudformation deploy \
    --capabilities CAPABILITY_IAM \
    --template-file /tmp/packaged.yaml \
    --stack-name my-API-dev
```

Hit the endpoint with CURL or Postman.

```bash
curl ${ENDPOINT_URL}/ping
{"message": "pong", "response": {}}
```

Go to AWS logs for the lambda and copy the received event to local `test_events` for off-line testing before deployment.

### 6. Set-up Postman
Use Postman to track development progress

- Write postman calls for each of you endpoints.
- Add tests to ensure the format is expected. These will become your integration tests and will eventually go into your I pipeline.

## Ongoing Iterative Development
Now you are ready to build out the functionality of API.  The ideal work flow is:

1. Write the endpoint test
2. Write API code to pass the tests

## Finally

I deliberately avoid implementing authorisers, COORS or API keys until the application itself is clearly defined, tested and functional.  Setting these up using Cloudformation/SAM can greatly increase the complexity of the template and lead to mistakes that are difficult to debug during development.  For guides on these see the Advanced topics below:

- [API Keys and throttling]( {{< relref path="api-gateway2.md" >}})
- [Cognito and Authorization]( {{< relref path="api-gateway3.md" >}})
- [Pythonic code and the PROXY_RESPONSE]( {{< relref path="api-gateway4.md" >}})

