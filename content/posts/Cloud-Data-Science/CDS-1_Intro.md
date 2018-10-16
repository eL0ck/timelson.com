---
title: "Cloud Data Science - AWS or GCP ?"
date: 2018-10-11T08:23:19+10:00
draft: false
tags:
  - AWS
  - GCP
  - ML
  - Cloud Wars
  - TensorFlow
  - Jupyter
---

## Where are your data research tools taking you? {#where}

Something we often deal with on behalf of clients is the problem of getting models from the hands of researchers into an environment where they can be used in a reliable and scalable way for their intended use.  It can be an uncertain time.  Many such examples started as side-projects with unclear potential.  Data science and Machine Learning are often regarded as black-magic voodoo and as such they can end up without the normal organisational support you'd expect of a business venture.

The model is then shown to work so, in hurried excitement, management interprets this as 90% of the work being done and wants the models in production yesterday.  However the problem is that a couple of hacky scripts do not constitute a software project.

Often this means that data-scientists become responsible for its daily operation because no one else can get it to work (Read: "deal with its unreliability").

This is very limiting to the organisation and a recipe for productions disasters and job dissatisfaction as researchers are forced to assume roles they have no training for.  Basically your team becomes a bunch of expensive and skilled people doing job swaps with each other so that no one is good at their job.

Scenarios like this, expose numerous work-flow issues which can be overcome by fairly standard software development practices and a skilled DevOps/integration team.

They will be able to find a way to automate you terrible research work-flow that uses Excel files in Matlab.  They will be able to build a custom Docker image that has all of your bloated R libraries and obscure single use complied executables.  They'll get it all talking to each other with bash scripts that will give you blood-pressure issues when you so much as think about changing a config file. They will be able to find a way to mount external volumes with your unfiltered data from a sub-optimal data store.  And to do all that they'll be indisputably awesome.  If they're blazingly shit-hot they'll be able to make it half-scalable by running it in a cloud hosted Kubernetes or ECS cluster.

The problem with all of this is that, in many ways, the more capable your DevOps squad is, the more elaborate the system will become to run your model architecture and the longer you will go before you realise that you've been completely left behind.

The subject of this series is about avoiding as much of that as possible by:

> **Getting researchers working on the right tools from the beginning**.

## So what are the right tools ?

At any data-driven organisation you find yourself at, you will find a different cocktail of tools in use. Many researchers are still working with the first antiquated set-up they learned during undergrad 10 years ago.  It used to be Matlab.  Thank god that's over.  Whatever it is, it does the job to a minimum viable level, with small test sets of the data, when treated just right and given plenty of time.

When these are your criteria you are right to be confused by the plethora of options, and right not to waste your time if you already have something.  An exhaustive analysis is an intimidating thing.  If you choose to have slightly more demanding criteria however, the decision becomes much simpler.

The giant tech companies have built their brands on big data and insights into it.  If you want performance analytics on big data, the decisions about how your organisation should tool-up have largely been made for you.

**In this series I compare the offerings from Amazon Web Services (AWS) and Google Cloud Platform (GCP) for conducting data research, model development and most importantly; the promotion of it into production in a reliable and scalable way.**

*Re Azure: I have always been awestruck  by the amazing work done at Microsft in these fields (and secretly really want a Surface Book) but at this stage I'm still far too scared by all the mouse pounding I see in their (excellent) demo videos.*

## Evaluating the Cloud Offerings

***So what exactly are they offering and what about these fancy new services are more 'in the cloud' than my current setup?***

Both GCP and AWS claim to provide the following toys you can't get at home:

- Easy model development and deployment
- Accelerated training, comprising;
    - access to greater computational power, and
    - automated hyperparameter search
- Memory efficient access to large datasets

***Thats all good and well, but before I'm drinking the Kool-Aid I'm going to make sure I'm not going backwards first.***

In addition to the above we're going to evaluate and compare the development environment offered by each and mandate that it must perform at least equally to our local version in the following categories:

- Ease of use
- Integration with version control
- Configurability (package management etc)
- Editing non-notebook files

Finally, we will talk in general about other comparative advantages and disadvantages.

## The Test Case: Iris Identification using TensorFlow

To mock the process we use the Iris dataset^[https://en.wikipedia.org/wiki/Iris_flower_data_set].  For those not familiar with it, it is the *Hello world* of data science.

We choose TensorFlow because it is officially supported by both GCP ML-Engine^[https://cloud.google.com/ml-engine/docs/] and AWS Sagemaker^[https://docs.aws.amazon.com/sagemaker/latest/dg/tf.html] and is probably the most popular framework for implementing neural networks with growing momentum.  It topped the general category *Most Loved Frameworks, Libraries and Tools* section of the [2018 Stack-overflow Developer Survey](https://insights.stackoverflow.com/survey/2018#technology-most-loved-dreaded-and-wanted-frameworks-libraries-and-tools) competing in a broad category including React and Django.  Furthermore, it is heavily used and supported by Google for their in-house ML.  Compared to other common alternatives it is vastly more discussed in questions on StackOverflow.

{{< figure src="/CDS/tf_alternatives.png" link="https://insights.stackoverflow.com/trends?tags=tensorflow%2Ctheano%2Ccaffe%2Cscikit-learn&utm_source=so-owned&utm_medium=blog&utm_campaign=gen-blog&utm_content=blog-link&utm_term=incredible-growth-python" >}}

At a low-level it is a well designed numerical library optimized for high-performance architectures.  Whilst offering a large number of the most popular machine learning algorithms as pre-rolled estimators, an expert user can use it to perform any numerical operations.


## So whats the TL;DR? {#results}

After implementing the mock problem on both platforms (See the [*Detailed Review*]( {{< relref path="CDS-4_Detailed.md" >}})) the results are:

| Category                  |  AWS  |  GCP  |  Winner  | Reason |
| ------------------------- | :---: | :---: | :---------: | --------- |
| **Notebook Environment**  | :+1: | :-1: | ![AWS](/aws1.png) | Has terminal, Jupyter-lab and conda |
| **Notebook Default Security**     | :-1: | :sparkling_heart: | ![GCP](/gcp1.png) | SSH tunnel by default |
| **Notebook Sharing**      | :sparkling_heart: | :-1: | ![AWS](/aws1.png) | Pre-signed link by default |
| **Notebook Git**          | :+1: | :sparkling_heart: | ![GCP](/gcp1.png) | Ungit installed |
| **Notebook Other VCS**    | :+1: | :shit: | ![AWS](/aws1.png) | Datalab has no terminal |
| **Notebook Performance**  | :+1: | :-1: | ![AWS](/aws1.png) | Direct connection to server |
| **Model Development**     | :-1: | :+1: | ![GCP](/gcp1.png) | Supports local testing, AWS attempts to but fails  |
| **Model Deployment**      | :-1: | :+1: | ![GCP](/gcp1.png) | AWS export fails to serve with simple tensor |
| **Model Options**         | :+1: | :-1: | ![AWS](/aws1.png) | Huge range not limited to TF |
| **Supporting tooling**    | :+1: | :-1: | ![AWS](/aws1.png) | GCP python libs very poor |
| **Supporting regions**    | :+1: | :-1: | ![AWS](/aws1.png) | GCP very limited |
| **Large data sets**       | :sparkling_heart: | :-1: | ![AWS](/aws1.png) | Streaming from S3 available |


## Summary {#summary}

In the end, your choice will depend on where you are currently, what your in-house cloud expertise prefers and how willing you are to adapt.  If you currently have TensorFlow models that you want to serve in the cloud and reduce server management overhead, GCP MLE is a clear choice provided there's a region near you.  Otherwise, it might be desirable to develop new models using Sagemaker and leverage the mature supporting infrastructure of AWS.

Most importantly I hope this article has prompted you to consider where your efforts are best applied and given you some insight into the future of model serving at scale.

---

***For a more detailed feature discussion and demonstrations of the two cloud platforms [continue reading]( {{< relref path="CDS-4_Detailed.md" >}})***



