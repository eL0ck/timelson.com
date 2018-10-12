---
title: "AWS Sagemaker - Walk-through"
date: 2018-10-12T08:23:19+10:00
draft: false
tags:
  - AWS
  - ML
  - TensorFlow
  - Jupyter
---

In the following guide, I use Amazon Web Services (AWS) tools to work through an example modelling project from data exploration and development, to model training and finally the deployment of a prediction endpoint.

***This guide is part of a longer discussion comparing [Cloud Machine Learning Tools]( {{< relref path="CDS-1_Intro.md" >}} )***

---

[AWS Sagemaker](https://aws.amazon.com/sagemaker/) is the service on offer to meet these requirements.

## Requirements

This walk-through requires:

- An [AWS account](https://aws.amazon.com/). New users get free credit on 'free tier' items.

Let's get started.

## 1. Test Hosted Development Environment

### Start a Sagemaker Notebook Instance

Log in to the AWS console and from the service list choose *Amazon Sagemaker*. Create a Notebook instance, if it's not obvious, use the [official guide](https://docs.aws.amazon.com/sagemaker/latest/dg/howitworks-create-ws.html).  Once the notebook instance in *InService*, open it with the link.

### Clone a repository
Click on the `New` drop-down menu in the upper-right corner of the notebook homepage and choose `terminal`.  Using the terminal, clone the notebook:

```bash
cd SageMaker
git clone https://github.com/eL0ck/Cloud-Data-Science.git
cd Cloud-Data-Science
```

Back in the notebook *Home* page, navigate to `Cloud-Data-Science/Platform-Comparison/` and open up the notebook `IrisCategorisation-TensorFlow.ipynb`.  This notebook is written under python version 3.6 so select the `conda_tensorflow_p36` kernel when prompted.

Now run all cells as you would locally to simulate model exploration and development.

---

To test the rest of our requirements we need to modify the code.  Open `IrisCategorisation-TensorFlow-Sagemaker` using kernel `conda_tensorflow_p27`.

## 2. Refactor for Sagemaker
To run TensorFlow on Sagemaker we must conform to certain requirements:

- Implement the [required functions](https://github.com/aws/sagemaker-python-sdk/blob/master/src/sagemaker/tensorflow/README.rst#preparing-the-tensorflow-training-script)
- Use python 2.7
- Use a version of TensorFlow between not greater than 1.10

AWS's `sagemaker` python SDK provides a great deal of abstraction from the underlying services to help you get up and running quickly.  One such example is the following, function that in one line uploads data to a default bucket in S3.

First, save the data to S3 since this is most likely where we will be storing our data in production.
```python
# Upload the locally stored data to `s3://sagemaker-{region}-{account}/iris/
s3_data = sm_session.upload_data(path='data', key_prefix='iris/')
```

*Surprisingly, its possible to create any number of buckets matching that pattern for accounts other than your own.  I wonder how this function deals with that*

## 3. Accelerated training
Unfortunately, it is not clear what steps Sagemaker takes with the provided `train_input_fn`, `eval_input_fn`, `serving_input_fn` and `estimator_fn` so whilst we can verify the model completes a training run and exports a model artifact we can't confidently know how it will go on Sagemaker until we send it off.

We do that with the commands:

```python
from sagemaker.tensorflow import TensorFlow

#Bucket location to save your custom code in tar.gz format.
custom_code_upload_location = 's3://{}/iris/code'.format(bucket)
#Bucket location where results of model training are saved.
model_artifacts_location = 's3://{}/iris/artifacts'.format(bucket)

estimator = TensorFlow(entry_point='../package/trainer/model.py',
                       role=execution_role,
                       framework_version='1.10',
                       output_path=model_artifacts_location,
                       code_location=custom_code_upload_location,
                       train_instance_count=1,
                       training_steps=100,
                       evaluation_steps=10,
                       train_instance_type='ml.c4.xlarge')
estimator.fit(s3_data)
```

This takes the model file `model.py` in which we have met the specifications and runs up the number and type of machines specified by the `train_instance_count` and `train_instance_type` variables.

### Gotchas

1.  Your `eval_input_fn` and `training_input_fn` must be **called**.  Unlike when passing to TensorFlow normally.  Notice the empty `()` on each.

2.  `estimator.fit()` may take a local file path OR and S3 path.  This is very handy for testing before sending to Sagemaker and eliminates boilerplate for fetching S3 objects.

## 4. Model Deployment

Provided you have defined a `serving_input_fn` deploying the model to a fully managed REST API is as simple as:

```python
iris_predictor = estimator.deploy(initial_instance_count=1,
                                  instance_type='ml.t2.medium')
```

**Unfortunately I have not found a way to provide input to this endpoint that is acceptable:**

```python
>>> dict(sample0)
{'PetalLength': 1.7, 'PetalWidth': 0.5, 'SepalLength': 5.1, 'SepalWidth': 3.3}
```

The following attempts all produce the same ambiguous error

```python
# iris_predictor.predict(dict(sample0)
# iris_predictor.predict(list(sample0.values))
# iris_predictor.predict({ u'': [dict(sample0)]})
iris_predictor.predict({u'' : list(sample0.values)})
```

```python
ModelError: An error occurred (ModelError) when calling the InvokeEndpoint operation: Received server error (500) from model with message "". See https://<yourlogs> in account XXXXXXXXXXXX for more information.
```

Then, following the link to Cloudwatch logs:

```
ERROR in serving: u''
```

```python
Traceback (most recent call last):
File "/usr/local/lib/python2.7/dist-packages/container_support/serving.py", line 182, in _invoke
self.transformer.transform(content, input_content_type, requested_output_content_type)
File "/usr/local/lib/python2.7/dist-packages/tf_container/serve.py", line 281, in transform
return self.transform_fn(data, content_type, accepts), accepts
File "/usr/local/lib/python2.7/dist-packages/tf_container/serve.py", line 208, in f
prediction = self.predict_fn(input)
File "/usr/local/lib/python2.7/dist-packages/tf_container/serve.py", line 223, in predict_fn
return self.proxy_client.request(data)
File "/usr/local/lib/python2.7/dist-packages/tf_container/proxy_client.py", line 66, in request
request_fn = self.request_fn_map[self.prediction_type]
```

```
KeyError: u''
```

Cool.

---

Currently, I am unable to actually get predictions from my deployed endpoint despite using an off-the-shelf serving function shown to work in Google ML-Engine.

---
To read more about how GCP and AWS compare for this task go to: [Cloud Machine Learning Tools]( {{< relref path="CDS-1_Intro.md" >}} )

