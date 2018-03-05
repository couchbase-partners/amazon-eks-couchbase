# amazon-eks-couchbase

This is a walkthrough of setting the [Couchbase Operator](https://blog.couchbase.com/introducing-couchbase-operator/) up on [AWS EKS](https://aws.amazon.com/eks/).  

## Background

EKS is in preview.  Your account must be whitelisted in order to use it.

To get started, download the following documentation:

    $ aws s3 cp s3://amazon-eks-docs/EKSDocs.zip .

The zip file is password protected.  If you're a Couchbase employee, ping Ben Lackey for the password.

To follow along for updates to the documentation, you can subscribe to the following SNS topic arn:

    ​arn:aws:sns:us-west-2:602401143452:eks-docs-update

Additionally, there's a Slack channel for EKS preview participants.  If you're a Couchbase employee and want added to it, let Ben know and he can get you added.

## Setting up EKS

There are some nice instructions for setting EKS up in the Getting Started guide under EKSDocs/userguide/getting-started.html

## Deploying the Operator

Once you have an EKS cluster deployed and a running kubectl, you're ready to deploy the Operator.  The instructions for deploying that are [here](http://docs.couchbase.com/prerelease/couchbase-operator/beta/overview.html).
