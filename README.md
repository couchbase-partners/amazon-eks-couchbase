# amazon-eks-couchbase

This is a walkthrough of setting the [Couchbase Operator](https://blog.couchbase.com/introducing-couchbase-operator/) up on [AWS EKS](https://aws.amazon.com/eks/).  

## Background

EKS is in preview.  Your account must be whitelisted in order to use it.

You'll also probably want a copy of the documentation.  If you're whitelisted you'll get access to that.  There's a Slack channel too.

## Setting up EKS

EKS is only available in Oregon currently.  Run this template:
https://amazon-eks.s3-us-west-2.amazonaws.com/1.7.10/0.1/vpc_cloudformation.yml

Grab a copy of kubectl locally:

    curl -o kubectl https://amazon-eks.s3-us-west-2.amazonaws.com/1.7.10/0.1/kubectl-osx
    chmod +x ./kubectl
    cp ./kubectl $HOME/bin/kubectl && export PATH=$HOME/bin:$PATH
    echo 'export PATH=$HOME/bin:$PATH' >> ~/.bash_profile
