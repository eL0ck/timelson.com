---
title: "Deploying Machine Learning Solutions"
date: 2020-02-05T08:18:19+10:00
draft: true
tags:
  - ML
  - AWS
---

At [Kablamo](https://kablamo.com.au) we are very fortunate in that the majority of our projects are novel innovations for our clients rather than reboots/rewrites or refreshment projects.  We've made a name for ourselves as being the team you call if you need insight and vision for aspirational projects where the pathway to success is unclear.

These projects are great because there's no legacy systems to drag forward.  You have the independence to design it right from the get-go.  But while sometimes as a consultant you can avoid working with legacy systems there will always be some level of legacy thinking that requires careful team augmentation.

Recently I was involved in one such project with a big 4 bank.  The bank had assigned a small team of Data Scientists to assist us with the project from the banks side.

# Ideal world

In an ideal world we'd have huge sets of clear, verified, raw data ready to drop into AWS.  We'd have an expert in the domain available to help us understand the nuances of the problem and refine our data into highly correlated features.

We'd spin up a bunch of enormous machines to model the Petabytes of data in parallel and we'd do it with Sagemaker containers from the very beginning so that the final model artifacts would deploy continuously to the same automatically scaling containers without a hitch.

The business requirement would be to take small data payloads and return even smaller responses.   There would be no row level heuristics that couldn't be expressed as feature.

Life would be grand.  The butterfly's would flutter lightheartedly around bambi as she pranced through the forest.

# The Real World

The reality is that the majority of the 'Data Science' community has limited understanding of software development methodologies.  Often they are used to producing one-off analyses used for strategic decision making presentations not the real-time, high throughput models required to serve an online user.  Many have a set of tools that they find works for them and struggle to understand why one framework/tool-set is any more deployable than another. Because *deployment* is such an abstract concept to them its hard to justify why time that could be spent understanding a problem should be spent learning a tool that replaces an existing one.

The result that they've put a lot of hard work into something that only works on their machine and thus violates the core tenant of *science* in general; to be repeatable.

# How to Reconcile the two

## Educate and be educated.

Having a consultant blow-in and start preaching does not work for everyone.  The fact is, there is a lot to learn from most clients about the domain they work in.  There's also a lot to teach.

Most of these old-school researchers really want to see their work being used at scale.  Its much more satisfying than getting a passing mention in that Powerpoint I talked about.

So discuss the mechanics of the cloud, the limitations, and the features.  Learn about their tool-set preference and find a compromise.  Be clear about the ramifications of the decisions you settle on together.  It will almost certainly rules out seamless scalability and deployment that comes with Sagemaker so, you'll need to deal with that with a team of people.  Make it clear what those choices will cost.


## Clear Division of Responsibilities

Spend time talking.  Always have patience for the basics.

Modelling isn't like regular software development but it is software and it needs to be developed.  You can't take a design doc and write tests that enforce a deterministic behaviour.  Often you don't see incremental progress but rather long lulls followed by breakthroughs.  Many software development managers don't have the skills or tools to deal with this.

Define the relationship between modelling team and deployment team in a similar way that you define an API.  Agree on a contract that allows you to work together.  This starts with a commitment from Modelling about what they can produce. Accuracy is not important but just whether or not something can be produced from the data and what format it can be produced in.

Once that is established use automated build processes that hold modelling to their commitment.  We'll come back to this.

Depending on your background and who you work with this may sound crazy but **You have to teach them Version Control**.  Even if they say they use it already. Get everyone in a room.  Open an issue, get them to create a branch and open a Pull Request.  Finally get them to check the status of the build that ran from the commit to the PR.

Be very careful at this point, its easy to loose people.  Build the lesson on a dummy example so everything works, under no circumstances use an existing code-base.  They have to see the process work.  They have to feel the liberation that comes from being able to share functional code across the team.  Its worth it.

The alternative is that you get an email once a week with an 'update to the model' with an ambiguous description of how good it is and you spend the next week trying to make it work.  The modelling team gets more an more disconnected from the production environment and so more incompatibility arises.  You spend all your time hacking together terrible solutions that are not performent and unmaintainable, and the best one; you're responsible for when a model doesn't behave the same way as it did on the authors Windows 7 desktop.

## Don't be too fussy :)

Once you get the modelling team enjoying the version control workflow, relax.  Help them get the builds to pass, explain why they broke.  At this point its easy and they're enjoying understanding how production hardware works.

Start adding tests and encourage them to as well.  Show them how the tests are ran and why breaking builds/tests are great.

Test end functionality only they're already struggling with all your rules so just make sure the code does what you need it to do.

Be generous during code reviews. They will enjoy doing them but make them about functionality not style or maintainability. Never do anything to risk loosing their motivation with the process.

# Conclusion

There's an 'us and them' mentality between software engineering and data science.  The changing landscape in recent years has meant that data scientists are not merely expected to communicate insights via power-point presentations once a month but deploy those insights to meet the demand expected of web applications.  Those that are unwilling to bridge the gap are also unable to use modern data warehouses.  As a result the distinction is breaking down.

Until it does, we have to find ways of achieving high standards in scalable solution deployment but in a way that fosters the growth of software development skills in those of us with varied backgrounds.


