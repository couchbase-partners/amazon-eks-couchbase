# amazon-eks-couchbase

Nowadays it seems like everyone loves Kubernetes and our team at Couchbase is no different. The [Couchbase Operator](https://blog.couchbase.com/introducing-couchbase-operator/) makes managing a Couchbase cluster running on Kubernetes even easier.<br>

Kubernetes sums up what an [Operator](https://coreos.com/operators/) is quite well:

>An Operator represents human operational knowledge in software to reliably manage an application

With that established, the next question is how do we use that in the cloud and as a managed service.  Amazon's answer to that is [Amazon Elastic Container Service for Kubernetes (EKS)](https://aws.amazon.com/eks/)

Here is a summary of EKS:

>Amazon Elastic Container Service for Kubernetes (Amazon EKS) is a managed service that makes it easy for you to run Kubernetes on AWS without needing to install and operate your own Kubernetes clusters

## EKS is in preview so...

Amazon Elastic Container Service for Kubernetes EKS is currently in preview.  In order to use it your account must be whitelisted.  Additionally, there's a Slack channel for EKS preview participants.  If you're a Couchbase employee and want to join, let Ben know and he can get you added.

Be sure to keep the documentation (EKSDocs.zip that will be referenced in the next section) up to date, this may help you troubleshoot issues that may arise.

For our walkthrough there are three major steps.

1. Create an EKS cluster
2. Deploying the Couchbase Operator
3. Create a Couchbase cluster 

### Create an EKS Cluster

Amazon has provided some quality documentation, on how to setup your environment and how to create an EKS cluster.  We will leverage that documentation for our purposes.
Follow the guide until you have kubectl installed, a vpc, a cluster and worker nodes deployed. Don't deploy the DNS add on as that seems to cause problems.

We will be deploying the Couchbase Operator and a Couchbase cluster so we can ignore Step 4, setting up the Guest Book sample application.

To get started, download the documentation (the password is **98c56b357**):

```aws s3 cp s3://amazon-eks-docs/EKSDocs.zip .```

Unzip EKSDocs.zip and then untar and unzip the userguide.tar.gz:
![userguidetar](/images/EKS_userguide_tar.png)

The instructions we will use can be found at EKSDocs/userguide/getting-started.html:
![userguidhtml](/images/EKS_getting_started.png)

When we have completed the getting started steps we would should be at the point illustrated below:
![kubectlwatching](/images/EKS_kubectl_watch.png)

#### Validate the EKS setup

After following the userguide we should have two stacks [deployed](https://us-west-2.console.aws.amazon.com/cloudformation/home?region=us-west-2):

![cloudformation](/images/EKS_two_stacks.png)

We should also have a cluster in [EKS](https://console.aws.amazon.com/eks/home?region=us-west-2):

![eks](/images/EKS_active_cluster.png)

Let's verify our setup by checking our nodes using kubectl.

Run the command:

```kubectl get nodes```

![kubectl](/images/EKS_kubectl_get_nodes.png)

### Deploying the Operator

Once you have an EKS cluster deployed and a running kubectl, you're ready to deploy the Operator.  The documentation on that is [here](http://docs.couchbase.com/prerelease/couchbase-operator/beta/overview.html).
Now that we have all the environment prerequisites in place, we are ready to deploy the Couchbase Operator.

In preparation for creating the deployment change to your home directory:

    cd ~

Now we get a local copy of the operator.yaml by running:

    curl -O  https://s3.amazonaws.com/packages.couchbase.com/kubernetes/beta/operator.yaml

Using your favorite text editor, add this line under env.  This will tell your nodes to request the EKS endpoint rather than the internal IP of the Kubernetes cluster.

    - name: KUBERNETES_SERVICE_HOST
      value: <your AKS endpoint>

That should give you something like:

![editoperatoryaml](/images/EKS_operator_yaml_edit.png)

Next we deploy the Couchbase Operator with:

![operatorcreated](/images/EKS_kubectl_operator_created.png)
EKS_kubectl_operator_created.png
The Couchbase Operator is deployed.  Now we verify it by running:

    kubectl get deployments

![operatordeployed](/images/EKS_kubectl_get_deployments.png)

### Role-based Access Control (RBAC)

#### Foundational Knowledge
Kubernetes uses RBAC which gives us flexibility on how our pods are controlled.  For those that are not familiar with RBAC I will break down the the concepts that are important for our walkthrough.

The first concept will cover is the **_service account_** (as explained by Kubermetes):
>A service account provides an identity for processes that run in a Pod.

Next we will tackle the concept of a **_cluster role_**.  Roles exist as well, but for simplicity we will focus on cluster roles.  Again as explained by Kubernetes:

>In the RBAC API, a role contains rules that represent a set of permissions. Permissions are purely additive (there are no “deny” rules). A role can be defined within a namespace with a Role, or cluster-wide with a ClusterRole.

Now that we understand what a service account and cluster role is, the final step is to assign a cluster role to a service account and that is the cluster role binding step (you guessed it, as explained by Kubernetes).

>Finally, a ClusterRoleBinding may be used to grant permission at the cluster level and in all namespaces. The following ClusterRoleBinding allows any user in the group “manager” to read secrets in any namespace.

We haven't discussed namespaces but you may think of them as a way to make virtual clusters out of a single Kubernetes cluster.  We are using the "default" namespace that is always available, and we are using a cluster role so namespaces are not that important for our walkthrough.

#### Setting up RBAC

In order for the Couchbase Operator to be able to create resources within our Kubernetes cluster we have to grant it permission.  The RBAC process has a few simple steps:

1. Create a Service Account (or using an existing one)
2. Create a clusterRole (or use an existing one)
3. Bind clusterRole(s) to the ServiceAccount

OK now let's create a new Service Account.

Create a file called **serviceaccount-couchbase-operator.yaml** with the contents below:

```
kind: ServiceAccount
apiVersion: v1
metadata:
  name: couchbase-operator
  namespace: default
```
Run the command to create the Service Account:

    kubectl apply -f serviceaccount-couchbase-operator.yaml

Next we will create a _cluster role_ and bind it to our newly created _service account_ (all in the same file).

Create a file named **clusterrole-couchbase-operator.yaml** with the content below.

```
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRole
metadata:
  name: couchbase-operator
rules:
- apiGroups:
  - couchbase.database.couchbase.com
  resources:
  - couchbaseclusters
  verbs:
  - "*"
- apiGroups:
  - apiextensions.k8s.io
  resources:
  - customresourcedefinitions
  verbs:
  - "*"
- apiGroups:
  - ""
  resources:
  - pods
  - services
  - endpoints
  - persistentvolumeclaims
  - events
  - secrets
  verbs:
  - "*"
- apiGroups:
  - apps
  resources:
  - deployments
  verbs:
  - "*"

RoleBinding:
---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRoleBinding
metadata:
  name: couchbase-operator
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: couchbase-operator
subjects:
- kind: ServiceAccount
  name: couchbase-operator
  namespace: default
---
```

Now we can run the command using that file. Remember this is creating the Cluster Role and Binding the role to our Service Account.

    kubectl apply -f clusterrole-couchbase-operator.yaml

![rbacsetup](/images/EKS_rbac.png)

### Deploying a Couchbase Cluster

The Couchbase Operator is all set to manage Couchbase clusters.  We create a Couchbase cluster with the following commands:

    kubectl create -f https://s3.amazonaws.com/packages.couchbase.com/kubernetes/beta/secret.yaml
    kubectl create -f https://s3.amazonaws.com/packages.couchbase.com/kubernetes/beta/couchbase-cluster.yaml

We should be able to see something like this:

![couchbasecreated](/images/EKS_kubectl_get_deployments.png)

We can view the all of the pods by running:

    kubectl get pods

![getpodscluster](/images/EKS_kubectl_get_pods_cluster.png)

#### Accessing the Couchbase Web UI

Now we have a Couchbase cluster running!
To use the web console we will need setup port forwarding.

We do that with the kubectl command:

    kubectl port-forward cb-example-0000 8091:8091

We need to make sure we leave the command running in the terminal:

![portforward](/images/GKE_port_forward.png)

Now we can open up a browser at http://localhost:8091

![loginscreen](/images/GKE_loginscreen.png)

We will login using username=`Administrator` and password=`password`.

We are in! Click the 'Servers' link on the left side and we should see our clusters running.

![webui](/images/EKS_webui.png)

Fin
