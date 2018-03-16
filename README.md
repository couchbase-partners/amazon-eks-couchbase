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

## Set up EKS

There are some nice instructions for setting EKS up in the Getting Started guide under `EKSDocs/userguide/getting-started.html`

Follow through the getting started guide until you have a vpc, cluster and worker nodes deployed.  Also make sure to get the kubectl.  There is no need to complete Step 4, setting up the Guest Book sample application.

Failing to get the latest doc can cause a world of pain as EKS seems to be evolving very quickly right now.

## Validate EKS setup

When that's all done, you should have two stacks deployed [here](https://us-west-2.console.aws.amazon.com/cloudformation/home?region=us-west-2):

![cloudformation](/images/cloudformation.png)

You should also have a cluster in EKS [here](https://console.aws.amazon.com/eks/home?region=us-west-2):

![eks](/images/eks.png)

Finally, you should be able to run your kubectl and see some nodes:

![kubectl](/images/kubectl.png)

## Deploying the Operator

Once you have an EKS cluster deployed and a running kubectl, you're ready to deploy the Operator.  The documentation on that is [here](http://docs.couchbase.com/prerelease/couchbase-operator/beta/overview.html).

To create the deployment, run this:

    kubectl create -f https://s3.amazonaws.com/packages.couchbase.com/kubernetes/beta/operator.yaml

Now check that it worked:

    kubectl get deployments

If you give it a minute or two it should show as available.
