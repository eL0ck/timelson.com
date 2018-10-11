---
title: "Blog Origins"
date: 2018-05-22T08:23:19+10:00
draft: true
categories:
  - Blogging
tags:
  - AWS
---

## Motivations

Two main reasons:

1. Some of my notes repositories have become bloated.  The Aim is to write better summaries that improve with review.

2. I wanna be like [boyter.org](http://boyter.org). As such, this whole thing is a direct rip-off of his blog.  This is a guy, that, if you're doing something different to, you're probably doing it wrong.

## Setting up the blog

Blogs are not something I'm particularly interested in so I got the basics from [Ben Boyter](http://boyter.org) and budgeted 2 hours to get this setup.

### Steps

1.  **Register the domain** -
Went to AWS console and registered [timelson.com](timelson.com).  Had it 15 minutes later for $12/annum

2. **Installed and setup** -
See: [Hugo](https://gohugo.io/getting-started/quick-start/)

3. **Choose a Theme** -
Add the theme to your hugo project.
```bash
git submodule add https://github.com/yihui/hugo-xmin themes/xmin
```
... then, in your `config.toml`:
```toml
...
theme = "xmin"
...
```

4. **Write an initial blog post** -
I'm writing it now.  Observing the changes live by using the hugo server with:
```bash
hugo server -D
```

4. **Generate static files**
```bash
hugo -v
```

5. **Setup S3 for static hosting** - Because I hate servers.
[AWS - static website hosting](https://docs.aws.amazon.com/AmazonS3/latest/dev/website-hosting-custom-domain-walkthrough.html)
```bash
aws s3 sync --delete public/ s3://timelson.com/
```
**And you're done!**

6. **CI the whole thing** -
The repository is in Bitbucket so eventually I'll put step 4-6 into the `bitbucket-pipelines.yaml` so that a blog post is as simple as a git commit.
(The same is true for github just use travis.ci.)

## Advantages

- No server managment
- Cheap as chips
- I can write a blog in my editor, git commit and its up, just as you would contribute to a code project.
