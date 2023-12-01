+++
title = "Why use AWS Lambda for Data Science?"
description = "An intro to serverless for data science"
date = 2020-06-23
[taxonomies]
tags = ["serverless", "AWS", "llambda", "talk"]
+++

## Motivation

Serverless is a way to deploy code, without having to manage the
infrastructure underneath it. In AWS terms this means there is an
compute instance that runs your code, except you don’t control it, and
that might be a good thing. It only exists when it’s asked for by
something else, and therefore you only pay for the work it does. If you
need to do work concurrently you get a new instance, which also goes
away as soon as you don’t need it, so it’s scaleable.

For a data scientist this is an interesting prospect for a number of
reasons. The first is keeping you hands clean. Not all people in this
role come from a ‘operations’ background. Many of us are analysts first,
and graduate into the role. However, that shouldn’t mean we don’t ‘own
our deployments’. However, it also means that we might not have the
background, time or inclination to really get into the nitty-gritty.
Managed infrastructure, that can scale seamlessly out of the box is a
nice middle ground. We can still manage our own deployments, but theres
less to worry about than owning your own EC2 instances, let alone a
fleet of them. The way that the instances themselves die off is also
valuable. We may be doing work that requires 24/7 processing, but often,
we aren’t. Why pay for a box which might have 50% required utilisation
time, or even less?

### Limits

Just like in everything there is a balance. There are *physical* limits
to this process. I’ve had success deploying data science assets in this
architecture, but if you can’t fit your job in these limits, this
already isn’t for you. Sure, data science *can be* giant machine
learning models on huge hardware with massive data volumes, but we have
to be honest and acknowledge that it isn’t always. K.I.S.S. should apply
to everything.

If you can get good enough business results with a linear regression,
don’t put in 99% more effort to train the new neural network hotness to
get a 2% increase in performance. Simplicity in calculation, deployment,
and explainability *matter*.

> “No ML is easier to manage than no ML” ©
> [@julsimon](https://twitter.com/julsimon/status/1124383078313537536)

## Getting started

``` python
from scipy import stats
import numpy as np

np.random.seed(12345678)

x = np.random.random(10)
y = 1.6*x + np.random.random(10)

slope, intercept, r_value, p_value, std_err = 
  stats.linregress(x, y)
```

This is a nonsense linear regression. IMHO it’s a data science ‘Hello
World’. Let’s make it an AWS Lambda serverless function.

``` diff
+ import json
from scipy import stats
import numpy as np
+ def lambda_handler(event, context):
  np.random.seed(12345678)

x = np.random.random(10)
y = 1.6*x + np.random.random(10)

slope, intercept, r_value, p_value, std_err = stats.linregress(x, y) 
+   return_body = {
  +       "m": slope, "c": intercept,"r2": r_value ** 2, 
  +       "p": p_value, "se": std_err
  +   }
+   return {"body": json.dumps(return_body)}
```

These changes achieve 3 things:

1.  Turning a *script* into a *function*
2.  Supplying the function arguments `event` and `context`
3.  Formatting the return as json

These are required as AWS Lambda needs a *function*. This is so that its
*event driven architecture* can feed in data through `event`, and so
that it’s `json` formatted data can both be received by your function,
and then also the response be returned by that function into the rest of
the system.

You can then open up the AWS console in a browser, navigate to the
Lambda service, and then copy and paste this into this screen:

![aws console lambda editor](https://raw.githubusercontent.com/DaveParr/dev.to-posts/master/snakes-lambdas_files/basic.png)

You can then hit run and…

![aws console lambda editor with an error
message](https://raw.githubusercontent.com/DaveParr/dev.to-posts/master/snakes-lambdas_files/basic-fail.png)

What happened? Well, because it’s a *managed* instance, the function
doesn’t know what `scipy` is. It’s not installed on the cloud, it was
installed on your machine…

## Layers

AWS lambda doesn’t `pip install ....`. Seeing as these run on compute
instances that turn up when needed, and are destroyed when not needed,
with no attached storage, you need to find a way to tell AWS what your
dependencies are, or you’ll just have to write super-pure base Python!
Well, that may not be *strictly* true. `json` is *built in by default to
every instance*, so is `boto3`, but what about our data science buddies?
`numpy`, `scipy` are *[published by
aws](https://aws.amazon.com/blogs/aws/new-for-aws-lambda-use-any-programming-language-and-share-common-components/) as layers*. Layers are bundles of code, that contain the dependencies you need to run the functions you write.
So in this case we can open the ‘layers’ view in AWS and attach these to our function.

![layers](https://raw.githubusercontent.com/DaveParr/dev.to-posts/master/snakes-lambdas_files/layers.png)

Now that you’ve attached all your dependencies with layers, go ahead and
run your function again.

![editor](https://raw.githubusercontent.com/DaveParr/dev.to-posts/master/snakes-lambdas_files/basic-success.png)

Success! So now you know the basics of how to put some Python data
science into practice on AWS Lambda.

This is a companion post to my talk on using data science in AWS lambda.
If you’re keen to know more, and can’t wait for me to write it all up
here. You can get the gist of the whole talks from [this repo](https://github.com/DaveParr/snakes_and_lambdas)
