---
title: "Cloud Data Science - Detailed Platform Review"
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

***This article follows from a prior introduction to [Cloud Data Science]( {{< relref path="CDS-1_Intro.md" >}} ) on GCP and AWS***

### Walk-throughs {#walk}

For those interested in experiencing the tools themselves, read the following walk-throughs to set-up demo projects:

- [GCP MLE/Datalab - Walk-through]( {{< relref path="CDS-3_GCP.md" >}})
- [AWS Sagemaker - Walk-through]( {{< relref path="CDS-2_AWS.md" >}})

The following notebooks are used in the demo's to demonstrate development work-flow and deployment:

- [Model Development Workflow Example](https://github.com/eL0ck/Cloud-Data-Science/blob/master/Platform-Comparison/IrisCategorisation-TensorFlow.ipynb)
- [MLE Deployment](https://github.com/eL0ck/Cloud-Data-Science/blob/master/Platform-Comparison/ML-Engine/IrisCategorisation-TensorFlow-MLE.ipynb)
- [Sagemaker Deployment](https://github.com/eL0ck/Cloud-Data-Science/blob/master/Platform-Comparison/Sagemaker/IrisCategorisation-TensorFlow-SageMaker.ipynb)

---

## Detailed Review {#review}

***Having now worked with both platforms, what are the pros and cons of each and which should we be using?***

### Development Environment

Essentially the notebook environments provided are the same.  Anyone who is confident with Jupyter notebooks will easily transition to either platform.  So while both offerings are excellent, it's the small things that cause AWS to pip GCP in this area.

By default, the user of a GCP Datalab notebook accesses it through an SSH tunnel bound to a local port.  Whilst it is possible to restrict access based on user identity and IP ranges, a default AWS Sagemaker notebook is created with a pre-signed URL granting access to anyone who has the link.  These differences have comparative advantages and disadvantages.

**Datalab is easier to secure** - If all users have GCP credentials and `gcloud` they are able to very easily create their own notebooks, sandboxed away from other users by inheritance of their project permissions.  The `gcloud` commands to create, stop and connect to Datalab instances are straightforward requiring no extra abstraction layer by business admins.

To create a Sagemaker notebook with comparative security would require some intermediary application to be used to ensure the required access controls are set on the notebook and the user received an execution role under which to operate Sagemaker tasks so as not to infringe on other users data.

**Sharing is easier on Sagemaker** - Because of the pre-signed URL, it is slightly easier to share a notebook amongst a whole team perhaps via a web page.  To perform the same tasks using Datalab the user would have to use the `glcoud` command to start the SSH tunnel.

**Performance** - The SSH tunnel used by Datalab does introduce some latency.  Generally, it's not a problem if you connect to a notebook instance using local `gcloud` and the notebook server is located in a nearby region.  If either of these conditions is not met however, the user experience suffers to such an extent as to make it unusable.  Whilst I generally love the way GCP have added Cloudshell to their console, it means that a Datalab instance started from there will first connect you via https to the server running Cloudshell, then to an SSH tunnel between it and the Datalab instance, then finally to the notebook server.  In comparison, the user connects directly to the AWS Sagemaker notebooks via https and offers no discernible lag even when using regions some distance away.  To test it for yourself.  Open up a Datalab instance from Cloudshell and try and use tab completion.  For instance, import pandas and type `pandas.Dat<tab><tab>`.

**Source Control** - If your team solely use git then Datalab is excellent.  It comes with the very slick tool, [Ungit](https://github.com/FredrikNoren/ungit), allowing users less confident with the git command line to visually manage their code bases in a very intuitive way.  Watch [this excellent demo video](https://github.com/FredrikNoren/ungit) to be convinced.  If you use any other version control you will be unable to get it on your Datalab instance without hackily using the command line through an IPython notebook, and you will have to install the application every time you start the instance.  The lack of a terminal in Datalab is a real disappointing.

That said, by using Ungit, you are able to input your git credentials once and use your local browser cache to store it. In comparison, in Sagemaker you either have an user restricted instance or input your repository credentials every time you access the remote server.  Storing credentials locally as you would normally do on a private machine means that anyone with Sagemaker access in your AWS account can open the notebook and assume your identity in the code base.

**Environment management** -  An AWS Sagemaker instance comes pre-loaded with `conda` and a couple of standardised environments for you to isolate your development process from any other projects you may be developing at the same time.  It seems that GCP intend Datalab to be used in a 'one project only' way since any packages installed go into the global python path.  The lack of an environment manager says more about the tools supported on each platform.  Really MLE is intended for use with TensorFlow, and Datalab is intended to develop it.  Sagemaker, on the other hand, has a broad array of model to use completely off the shelf, TensorFlow being only one of the available options.

**File Management** - In Datalab it seems impossible to create a raw text file.  I wonder how they expect a data-scientist to create the deployment package required by MLE from a notebook.  There are ways around it such as using `%%writefile` or `%save` or just more embedded bash, but leaves a bit of a stink.

I consider the lack of a terminal to be a major problem with Datalab.

**Overall Usability** - Both notebook environments have the keyboard shortcuts you expect as a Jupyter user so expect to hit the ground running.  One slightly irritating omission in Datalab is the inability to `<Shift><tab>` to reveal the docstrings of the object the cursor is on.  You'll have to use `?` instead which brings the docstrings up at the side, which is nicer than Jupyter IMHO.  The styling is nicer in Datalab, but the system control in Sagemaker is liberating.  In addition, Sagemaker are implementing the Jupyter-lab interface which may be more appealing to those used to an IDE environment such as PyCharm or Spyder.

### Integration and Deployment

Both AWS and GCP have documented examples of the iris problem:

- [Sagemaker Iris](https://github.com/awslabs/amazon-sagemaker-examples/sagemaker-python-sdk/tensorflow_iris_dnn_classifier_using_estimators/tensorflow_iris_dnn_classifier_using_estimators.ipynb)
- [TensorFlow Iris](https://www.tensorflow.org/guide/premade_estimators)

I choose to reimplement the problem from scratch because I want to experience the process from start to finish and discover the nuanced requirements of each platform along the way.  I also think it's safe to assume that published examples work as expected and any tailoring to the platform may not be obvious to the new user.  Just like an accurate model, the quality is not judged on the training data but how well it generalises to unseen examples.

In doing so I discovered shortcomings of both platforms, the more significant of which are highly restrictive, hidden requirements of Sagemaker's TensorFlow implementation.

**Model Deployment** - The steps to deploy a model are trivially simple on both providers.  It is truly amazing at how easily a user completely ignorant of web serving concepts can have a scalable REST endpoint serving their model.  A non-blocking gripe I have with the current MLE deployment process is that it is profoundly simpler to perform a model review and deployment via bash rather than the python API.  I find this clunky and inefficient.  You will see in the deployment notebook a vast amount of code simply for populating the shell variables for later use in embedded bash code blocks.  Below in *Supporting Tooling*, I show an example of the comparative immaturity of the GCP's python APIs to its bash tools.

AWS, on the other hand, has the excellent `boto3` python library.  `boto3` is so superior that for day-to-day AWS tasks that I use it in place of the bash tool `awscli` through an IPython shell.  This makes for much cleaner operation of the deployment process by not having to drop into a bash sub-shell and deal with I/O issues associated with that.  To highlight that, observe the line where we call `gcloud ml-engine local predict`.  In order to restrict the output to only the class id's I have to use `awk`.  I'd much prefer to use a python library and receive a `dict` back.

> Having come to grips with the necessity of embedded bash to use MLE, I am very impressed with it

Perhaps most impressive is the ability to verify the model before submission for remote/distributed training.  This is quite an important feature.  One of the major advantages offered by the cloud providers is the ability to train using much more computational resources than you have on-premise.  This allows us to try far more complex models than we every could before and consume larger data sources.

The practical problem that arises is that to start a remote job takes some minutes as the borrowed machines come up with their required environments and model tasks.  If we have made a minor mistake with our deployment (such as an uncalled `input_fn` in Sagemaker or unparsed command line input in MLE) our mistake only becomes clear after the 5 minutes required to bring up a large cluster of expensive machines.  Ideally, we have a way of running the exact command locally that will be run remotely with an arbitrarily small number of training epochs.

GCP MLE provided extremely good tools to verify our process before committing to an expensive and timely cloud training job.  Sagemaker on the other hand, does not provide a straightforward method to do this, however, the simplicity of the single python file operating in a pre-defined container does reduce the likelihood of minor mistakes.  MLE's test deployment also allows the model's endpoint and serving function to be tested locally which is an absolutely critical feature.  As seen in the linked Sagemaker notebook walk-through, I had problems getting the serving function to serve properly from Sagemaker but could only discover that after successfully training and deploying the model, potentially a lengthy and expensive task.

**Model Packaging** - GCP and AWS have gone to the extremes of the spectrum in this category and there are problems with each that I feel would be remedied by a middle ground approach.

MLE requires that we create a complete python package with `setup.py` rather than just a single training script like Sagemaker.  Observe that in the `package/` directory of the [example project](https://github.com/eL0ck/Cloud-Data-Science) the only file with anything specific to our model is `model.py`.  The rest is TensorFlow boilerplate and python packaging requirements.  To me, this is unnecessary complexity that may reduce uptake by data-scientists or require the time of integration developers.

In comparison, Sagemaker can take a single python file, in our case `model.py`.  It must conform to [certain requirements](https://github.com/aws/sagemaker-python-sdk/blob/master/src/sagemaker/tensorflow/README.rst#preparing-the-tensorflow-training-script) but, because of that, the packaging and boilerplate complexity is abstracted away.

**A very strange and confusing requirement** that's worth a mention is that `train_input_fn` and `eval_input_fn` must be called!

{{< highlight python "linenos=table,hl_lines=7 12,linenostart=68" >}}
# model.py

# N.B The functions are actually called here!

def train_input_fn(training_dir, hyperparams):
    """Returns input function that would feed the model during training"""
    return _input_fn('train', training_dir, 'iris_training.csv')()


def eval_input_fn(training_dir, hyperparams):
    """Returns input function that would feed the model during evaluation"""
    return _input_fn('eval', training_dir, 'iris_test.csv')()
{{< / highlight >}}

Those already familiar with TensorFlow will know that we pass an input function rather than actual input because of TensorFlow's delayed execution model.  We are giving it the ability to consume inputs, at some later stage. As such it is strange to see Sagemaker require us to call these, and something of a *gotcha* that is easily skimmed over.

Initially I was impressed by the simplicity of this templated approach but unfortunately, to achieve it, the Sagemaker container runs the code in a way that is unclear to the user and thus very difficult to debug.  In the Sagemaker example notebook, the model trains and deploys as expected but when it comes to getting predictions from the endpoint, it errors uninformatively.

In comparison, for MLE we explicitly write the way we expect the model to be trained and deployed so we can easily run it locally and debug.  If there was a problem with the serving function like this we would do a `gcloud ml-engine local train` and `gcloud ml-engine local predict` without an accurate model, just to verify the serving function is acceptable.

Noting that the `serving_input_fn` is the same function correctly serving predictions on MLE.  When the Sagemaker endpoint is called however, we receive the errors:

Locally:

> ModelError: An error occurred (ModelError) when calling the InvokeEndpoint operation: Received server error (500) from model with message "". See https://ap-southeast-2.console.aws.amazon.com/cloudwatch/home?region=ap-southeast-2#logEventViewer:group=/aws/sagemaker/Endpoints/sagemaker-tensorflow-2018-10-04-07-40-50-661 in account XXXXXXXXXXXX for more information.

Following the link above to CloudWatch logs:

> [2018-10-01 03:23:37,290] ERROR in serving: u''

Obviously, this is a bug with the Sagemaker container code, but running a development version of this container is not a simple process.  As such I'm still working on it, unable to deploy the endpoint to Sagemaker.  Open issues:

- [Vague Error Issue](https://github.com/aws/sagemaker-python-sdk/issues/413)
- [Add Instructions for Contributing to the project](hhttps://github.com/aws/sagemaker-tensorflow-container/issues/85)
- [Custom Tensor Name](https://github.com/aws/sagemaker-python-sdk/issues/394)

All things considered, I am much more confident providing a little boilerplate TensorFlow when it grantees that my model isn't bastardised behind the scenes by some opaque code hidden within a container that is approximately impossible to debug.  That said, I am quite confused as to why MLE has to `pip install` our package and run it from the shell.  It seems to me just as easy to have the container download the deployment package to the working directory and call the `task.py` with the `python` executable.  It wouldn't even require changes to the `gcloud ml-engine` command, you still provide the path to the package and the name of the file to be used as the entry point.  The upside would be:

- option to deploy model as a single file
- remove the need for researcher to understand python packaging
- remove the need to test compilation of the package installation, simply call `python task.py`
- **remove the god awful requirement for command line option parsing** since the entry-point file would be invoked directly.

> *What can i say, the GCP team seem to really love bash!*

In summary, it is clear what MLE is doing with your model.  It could be tidied up a little by doing away with all the bash sub-processes but all-in-all it works very very well.

**Supporting Tooling** - Both platforms require model submissions to be written in python 2.7.  I don't know why.  I'm kinda sick of going on about it really.  Its just so annoying to still see it used.

Above I mentioned that GCP's bash tools are superior to their corresponding python libraries.  I suppose it's time to back it up with an example.  Here is the process of creating a bucket and copying data-files to Cloud Storage.

In bash:

```bash
gsutil mb -p ${PROJECT} -l ${REGION} gs://${BUCKET}
gsutil -m cp data gs://${BUCKET}/
```

Yet the with python API the best we can get is:

```python
from google.cloud import storage

gs = storage.Client()

bucket = gs.create_bucket(
  bucket_name=BUCKET,
  project=PROJECT,
  # Not possible to specify a region here !!
)

# Now we have to traverse the `data/` directory and send each file individually !!
blob = bucket.blob(target_file_name)
blob.upload_from_filename(source_file_name)
```

It's crazy but it's actually easier and simpler to run `gcloud` and `gsutil` commands inside `%%bash` code cells rather than using google python libraries.  As well as being clunky and not particularly pythonic they are missing basic functionality such as the ability to specify a bucket region.  I'm sure its possible and the source code of `gsutil` will have all the answers but I don't think your run-of-the-mill researcher is going to dive in that deep just to upload some files.  Note also that the `-m` flag provides parallelism for the operation if possible, imagine what that would look like in python 2.7.  Disappointing right?

**Cloud Features** - It comes as no surprise that AWS leads the way in available regions and features.  It is arguably the case that our example project has some bias toward GCP by choosing TensorFlow.  Sagemaker has an excellent array of [off-the-shelf algorithms](https://docs.aws.amazon.com/sagemaker/latest/dg/algos.html) highly optimised for cloud deployment and tailored to specific tasks.  It is available in the regions closest to me (Sydney) and offers superior access to massive data sources using pipe mode to stream filtered S3 data.

GCP on the other hand:

```python
ERROR: (gcloud.ml-engine.jobs.submit.training) INVALID_ARGUMENT: Field: region Error: The provided GCE region 'australia-southeast1' is not available. Options are [asia-east1, europe-west1, europe-west4, us-central1, us-east1, us-west1].
- '@type': type.googleapis.com/google.rpc.BadRequest
  fieldViolations:
  - description: The provided GCE region 'australia-southeast1' is not available.
      Options are [asia-east1, europe-west1, europe-west4, us-central1, us-east1,
      us-west1].
    field: region
```

As of the time of writing the closest regions to train and serve my model in are over 7000 kilometres away in Taiwan/Tokyo (`asia-east1`/`asia-northeast1`).

***Return to [Cloud Data Science]( {{< relref path="CDS-1_Intro.md#summary" >}} )***

