---
title: "GCP Datalab/ML-Engine - Walkthrough"
date: 2018-09-14T08:23:19+10:00
draft: false
tags:
  - GCP
  - ML
  - TensorFlow
  - Jupyter
---

In the following guide I use Google Cloud Platform (GCP) tools to work through an example modeling project from data exploration and development, to model training and finally the deployment of a prediction endpoint.

***This guide is part of a longer discussion comparing [Cloud Machine Learning Tools]( {{< relref path="CDS-1_Intro.md" >}} )***

---

GCP offer two services to achieve our aims:

**1. [Datalab](https://cloud.google.com/datalab/)** - a hosted version of the [jupyter notebook](http://jupyter.org/).  It has all the same shortcut keys (unlike the frustrating [CoLabratory Notebooks](https://research.google.com/colaboratory/faq.html) you may have used in Google Drive).  Really the only significant differences to your local instance is the styling and [Ungit](https://github.com/FredrikNoren/ungit) for accessing git repositories.

**2. [ML-Engine](https://cloud.google.com/ml-engine/docs/)** - the GCP service for managing remote training jobs, models, endpoints.

## Requirements

This walkthrough requires:

- A [GCP account](console.cloud.google.com). Sign up for $400 free credit.
- (Optional) Locally installed `gcloud` command line utilities: [Installing Google Cloud SDK](https://cloud.google.com/sdk/install)

If you choose not to install `gcloud` you may use the [Cloud Shell](https://cloud.google.com/shell/docs/) utility available when you open up the GCP console but be aware, Datalab will perform more slowly for reasons to be discussed in the review.

Lets get started.

## 1. Test Hosted Development Environment

***Is Datalab as good as a local notebook?***

### Start a Datalab Instance

Run the following command to start a notebook instance.  If not, log into the console and run it in cloud shell.

```bash
gcloud compute zones list  # Choose a zone close by
datalab create dlvm --zone australia-southeast1-b
```

Wait for the instance to spin-up (can take up to 5 minutes!).  When the machine is ready and an SSH tunnel is connected the command will return instructions to open the browser via a local port.

### Clone a repository
Datalab provides [Ungit](https://github.com/FredrikNoren/ungit) for git repository management.  This is a really slick tool.  If you have 6 minutes to be convinced of this, watch [this video](https://github.com/FredrikNoren/ungit) from creator Fredrik Noren.

Open it from the icon in the top right corner of the Datalab index page and clone the demo:

```bash
https://github.com/eL0ck/Cloud-Data-Science.git
```

Return to the overview, open the `Platform-Comparison/` directory and select the notebook `IrisCategorisation-TensorFlow.ipynb`

Run all cells, ensuring that all steps complete and visualisations display as expected.

*Congratulations: You have performed the steps for basic model development*

## 2. Refactor for ML Engine

*So we've made a model but what exactly is required to take our dev research model, thoroughly test it and deploy it as a productions service?*

The complete process is shown end-to-end in the notebook `Platform-Comparison/ML-Engine/IrisCategorisation-TensorFlow-MLE.ipynb`.  Follow along with the discussion below.

### Project setup

We assume our project will use vast amounts of data.  Realistically we will use [Google Cloud Storage](https://cloud.google.com/storage/) since we can store any data at very low costs.

First create a bucket in your region and give the service account access to it.  We'll copy the test and training data to the bucket where it can be later pulled by the training jobs.

*In practise our production datapipeline would be filling up our datastores, all we need is permission to access it.*

```bash
gsutil mb -p ${PROJECT} -l ${REGION} gs://${BUCKET}
gsutil -m acl ch -u ${SVC_ACCOUNT}:W gs://${BUCKET}
gsutil -m cp -r ../data gs://${BUCKET}
```

### Code requirements
To run TensorFlow on MLE we must conform to certain requirements:

- The model must be packaged as a typical python package with a `setup.py` so that it can be installed with pip
- The package must have an entrypoint that performs the training, evaluation and, if necessary, the export steps.

Our deployment package is in `Platform-Comparison/package/` and structured like so:

```bash
.
├── setup.py
└── trainer
    ├── __init__.py
    ├── model.py
    └── task.py
```

Notice here that `model.py` declares the character of our model and how it is to be trained. It is the same file as we send to AWS Sagemaker.  `task.py` is the entrypoint for the module and includes the TensorFlow boilerplate to complete training, evaluation and export steps.

MLE provides simple ways to ensure the package conforms to the requirements locally before submitting to cloud resources.  With:

```bash
gcloud ml-engine local train ....
```
And ...
```bash
gcloud ml-engine local predict ...
```

This is vital since it can be very slow way to resolve simple mistakes by submitting to remote machines and waiting for the log output.


## 3. Accelerated Training

Up till now we will have been training with very small step number to ensure the model compiles. The plan is to use Google cloud resources to perform thorough training that we can't perform with on-premise machines.

Now that we are ready to send the job off we can increase the training steps and give the model much more resources by setting the [Scale Tier](https://cloud.google.com/ml-engine/docs/tensorflow/machine-types).

Other options for improving training with cloud resources such as hyperparameter tuning, distributing the jobs over a cluster and memory reduction techniques are their own topics and will not be explored just yet but represent great potential for unlocking insights currently out of reach for researchers operating on-premise.

```bash
JOBNAME=${MODEL_NAME}_$(date -u +%y%m%d_%H%M%S)
echo $OUTDIR $REGION $JOBNAME
# Clear the Cloud Storage Bucket used for the training job
gsutil -m rm -rf ${OUTDIR}
gcloud ml-engine jobs submit training ${JOBNAME} \
   --region=${REGION} \
   --module-name=trainer.task \
   --package-path=${PWD}/../package/trainer \
   --job-dir=$OUTDIR \
   --staging-bucket=gs://${BUCKET} \
   --scale-tier=BASIC \
   --runtime-version=${TFVERSION} \
   -- \
    --outdir=${OUTDIR} \
    --train_steps=1000 \
    --bucket=${BUCKET} \
    --project=${PROJECT}  \
    --test_file=data/iris_test.csv  \
    --train_file=data/iris_training.csv
```

### Gotchas

Your package entrypoint needs to take the argument `--job-dir` which is opaquely passed to it.  If you don't you will receive the following error despite local testing completing successfully.

```
task.py: error: unrecognized arguments: --job-dir ...
```

## 4. Model Deployment

*So, we have a trained model and are happy with its accuracy.  How are we going to use it?*

This is where your devops team pops in to try and make sense of the abhorrent mess you've made in your radically divergent feature branch.  Over the course of some weeks they are going to interpret the undocumented manual steps that you use to superstitiously run the model and cobble it into a delivery pipeline.  You'll also need an architect and a team of developers to implement a service structure so that model clients, internal or external, can actually use it in a way decoupled from you and your ongoing research.

Or ... you can have GCP do that for you.

```bash
gcloud ml-engine models create ${MODEL_NAME} --regions ${REGION}

# Get latest model if multiple available
MODEL_LOCATION=$(gsutil ls gs://${BUCKET}/${MODEL_NAME}/${TRAINING_DIR}/export/exporter | tail -1)

echo "MODEL_LOCATION = ${MODEL_LOCATION}"

gcloud ml-engine versions create ${MODEL_VERSION} \
    --model ${MODEL_NAME} --origin ${MODEL_LOCATION} \
    --runtime-version ${TFVERSION}
```

This has deployed a REST endpoint that is publicly accessible from the internet for authorized clients.

*Really?! Its all done??*

Lets be convinced.

Authenticate as a member of the project and send some test data.

```bash

AUTH_TOKEN=$(gcloud auth print-access-token)
TEST_INSTANCE="{\"instances\":[$(cat test.json|head -1)]}"

curl -X POST \
    -H "Content-Type: application/json" \
    -H "Authorization: Bearer ${AUTH_TOKEN}" \
    -H "Content-Type: application/json" \
    --data ${TEST_INSTANCE} \
    https://ml.googleapis.com/v1/projects/${PROJECT}/models/iris:predict
```

**Yep, no docker images, no kubernetes clusters, no load balancing or even DNS**  Yet you have a completely decoupled prediction service ready to serve single or batch requests at scale.

---

To read more about how GCP and AWS compare for this task go to: [Cloud Machine Learning Tools]( {{< relref path="CDS-1_Intro.md" >}} )
