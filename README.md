# amazon-eks-couchbase

This is a walkthrough of setting the [Couchbase Operator](https://blog.couchbase.com/introducing-couchbase-operator/) up on [AWS EKS](https://aws.amazon.com/eks/).  

## Background

EKS is in preview.  Your account must be whitelisted in order to use it.

To get started, please download the following documentation:

    $ aws s3 cp s3://amazon-eks-docs/EKSDocs.zip .

The zip file is password protected.  If you're a Couchbase employee, ping Ben Lackey for the password.

To follow along for updates to our documentation, you can subscribe to the following SNS topic arn:

    ​arn:aws:sns:us-west-2:602401143452:eks-docs-update

Additionally, we host a Slack channel for EKS preview participants. Please let us know if you'd like to participate, and provide the email IDs to be invited.

## Setting up EKS

There are some nice instructions for setting EKS up in the Getting Started guide here:
file:///EKSDocs/userguide/getting-started.html
