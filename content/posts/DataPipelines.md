---
title: "Data Pipelines"
date: 2019-04-12T08:23:19+10:00
draft: true
tags:
  - AWS
  - GCP
  - ML
  - Cloud Wars
  - Jupyter
  - Apache beam
  - Map reduce
---

## Does your tooling handle your evolving needs?

So many goddamn choices.  And to go with it, so many goddamn opinions.  Give a Data Scientist a task and you can guess the year they left academia by the tools they choose to get it done.  Add to this the general intimidation the lay-person feels when talking about cloud solutions at a level any deeper than regurgitating a blog post they read once (I'm looking at you ... through your screen) the result is that you to can experience the pleasures of ad-hoc, inflexible data-pipeline that is not only an on-going time-sink but cost way to much time to create too.

In general (and I will return for the exception), the serverless cloud solution is the one to choose.  Why? Well firstly the public clouds were developed through a real business need that probably far exceeds yours.  Furthermore, its the path of least resistance so is receiving the most adoption and gets the highest level of support.  All of this means that for 98% of use cases, you can define a methodology and toolset in the same way as you define a code style and engineering methodology and that these will always work for you.

The problem I encounter as a Consultant is that the *additional features* that I think are great freebies are often perceived by the uninitiated as *additional complexity*.  Take for example a REST API proof-of-concept.  I have mistakenly pitched the AWS API Gateway option as the "Dynamically Auto-scaling, load-balanced option" that it is and received the response: "Look, I just want to avoid all the bells and whistles for now.  We only need to demonstrate this to a small group of potential investors.  Can we just run an express app on an EC2?".  Nowadays I'd like to think I read the situation better and pitch it as the "Serverless, maintenance free, 3 hours to complete option" that it also is.  What's my point? The truth is that for almost all cases, and there are exceptions, the cloud-native solution is the best for both getting started *and* production.

## Data Pipelines

It must be one of the most citation-free statistics in the field but nonetheless I'm going to quote it; apparently 80% of Data Scientists work is spend cleaning and preparing the dataset.  Now if you let your Data Scientists prepare their data using whatever method academia was using 6 years ago, you **absolutely will** have to employ a bunch of other people to port it into a production and they absolutely will introduce bugs that absolutely will destroy the model.  The original modeller absolutely will have to work on it themselves as a matter of pride and thus learn the production method they should have used to start with.  All-in-all you will have spend X months and wasted X extra human resources doing something that could have been done in the beginning.

### So what is the correct method for Data Preparation?
Make it the cloud based option. The tools with the most capability are also generally the cheapest and have the most flexibility to deal with changes in the future.  They are also guaranteed to scale and have growing support.

> Caveat: Performance sensitive, non web-applications (since that is what 99% of tech is developed for and what the cloud providers require for their own businesses).  People look for an excuse to use the trendy new tool so they sound edgy at sleezy business lunches.  E.g Phoneix using BigQuery instead of just Postgress.  Summary: Analyse your needs and responde accordingly.

### Example: Taxi cab data

Lets take a look at a simple example that would be required to do a preliminary examination of a dataset in order to understand it but also later used in preparing composite features for model training.  First we'll examine an approach locally then we'll look at the same process using AWS and GCP.  Finally we'll assume we have a highly successful model using this pipeline and discuss the steps to put it into production.

## Ideas:

Compare queries on large data BigQuery and Redshift.
[Examples here](https://aws.amazon.com/blogs/big-data/visualize-over-200-years-of-global-climate-data-using-amazon-athena-and-amazon-quicksight/)

Do basic ETL on the taxifare data to get summary statistics and view histograms like in Coursera lab
Lab: Time-Windowed Features - TaxiCab

GCP Process:
- Use dataFlow with apache beam to define a feature development process for each row.
- Send rows to big query for batch training
- Predict on incoming stream using same beam code to engineer composite features

## AWS Process:
- ?
- Create a Redshift cluster ?
- Use Glue to send to

### 1. Copy files to S3
Create a GCP compute instance to do the transfer
```
<!--gsutil cp gs://asl-ml-immersion/nyctaxicab/* s3://glue-5d843dk4/-->
gsutil -m rsync -rd gs://asl-ml-immersion/nyctaxicab/* s3://glue-5d843dk4/
```

### 2. Perform the Data enhancements

Follow the tutorial steps and answer the questions.
