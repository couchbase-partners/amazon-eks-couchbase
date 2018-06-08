# amazon-eks-couchbase

Nowadays it seems like everyone loves Kubernetes and our team at Couchbase is no different. The [Couchbase Operator](https://blog.couchbase.com/introducing-couchbase-operator/) makes managing a Couchbase cluster running on Kubernetes even easier.<br>

Kubernetes sums up what an [Operator](https://coreos.com/operators/) is quite well:

>An Operator represents human operational knowledge in software to reliably manage an application

With that established, the next question is how do we use that in the cloud and as a managed service.  Amazon's answer to that is [Amazon Elastic Container Service for Kubernetes (EKS)](https://aws.amazon.com/eks/)

Here is a summary of EKS:

>Amazon Elastic Container Service for Kubernetes (Amazon EKS) is a managed service that makes it easy for you to run Kubernetes on AWS without needing to install and operate your own Kubernetes clusters

## Major steps of this walkthrough

It may feel like there is alot going on here, but we can break the tasks into palatable parts.  The three major parts of this walkthrough are:

1. Create an EKS cluster ([userguide](https://docs.aws.amazon.com/eks/latest/userguide/getting-started.html))
2. Deploy the Couchbase Operator
3. Create a Couchbase cluster

Now let's tackle each section.

### Create an EKS Cluster

Amazon has provided some quality documentation on setting up your environment and creating an EKS cluster.  We will leverage that documentation for our purposes.

Follow the [userguide](https://docs.aws.amazon.com/eks/latest/userguide/getting-started.html), completing steps up to "__*Step 3: Launch and Configure Amazon EKS Worker Nodes*__".

We will be deploying the Couchbase Operator and a Couchbase cluster so we can ignore "Step 4: "Launch a Guest Book Application".

When we have completed the getting started steps we would should be at the point illustrated below:
![kubectlwatching](/images/EKS_kubectl_watch.png)

#### Validate the EKS setup

After following the userguide we should have two stacks [deployed](https://us-west-2.console.aws.amazon.com/cloudformation/home?region=us-west-2):

![cloudformation](/images/EKS_two_stacks.png)

We should also have a cluster in [EKS](https://console.aws.amazon.com/eks/home?region=us-west-2):

![eks](/images/EKS_active_cluster.png)

Let's verify our setup by checking our nodes using kubectl.

Run the command:

    kubectl get nodes

![kubectl](/images/EKS_kubectl_get_nodes.png)

### Role-based Access Control (RBAC)

#### Foundational Knowledge
Kubernetes uses RBAC which gives us flexibility on who and how our pods are controlled.  For those that are not familiar with RBAC let's break down the the concepts that are important for our walkthrough.

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

### Deploy the Couchbase Operator

Now that we have setup RBAC, we are finally ready to deploy the Couchbase Operator (which will manage our Couchbase clusters).  I wont go into the details of the Couchbase Operator here, but If you are interested in learning more you may find this [overview](http://docs.couchbase.com/prerelease/couchbase-operator/beta/overview.html) as a good starting point.

Continuing on...  Grab a local copy of the operator.yaml by running:

    curl -O  https://s3.amazonaws.com/packages.couchbase.com/kubernetes/beta/operator.yaml

Using your favorite text editor, add this line under env.  This will tell your nodes to request the EKS endpoint rather than the internal IP of the Kubernetes cluster.

    - name: KUBERNETES_SERVICE_HOST
      value: <your EKS endpoint>

Be sure to remove the __*https://*__ if you are copying the 'value' from the console.

Your file should be similar to the following:

![editoperatoryaml](/images/EKS_operator_yaml_edit.png)

Next we deploy the Couchbase Operator with:

    kubectl apply -f ~/operator.yaml

![operatorcreated](/images/EKS_kubectl_operator_created.png)

The Couchbase Operator is deployed.  Now we verify it by running:

    kubectl get deployments

![operatordeployed](/images/EKS_kubectl_get_deployments.png)

### Deploying a Couchbase Cluster

The Couchbase Operator is all set to manage Couchbase clusters.  We create a Couchbase cluster with the following commands:

    kubectl apply -f https://s3.amazonaws.com/packages.couchbase.com/kubernetes/beta/secret.yaml
    kubectl apply -f https://s3.amazonaws.com/packages.couchbase.com/kubernetes/beta/couchbase-cluster.yaml

We should be able to see something like this:

![couchbasecreated](/images/EKS_couchbase_created.png)

We can view the all of the pods by running:

    kubectl get pods

![getpodscluster](/images/EKS_kubectl_get_pods_cluster.png)

#### Accessing the Couchbase Web UI

Now we have a Couchbase cluster running!
To use the web console we will need setup port forwarding.

We do that with the kubectl command:

    kubectl port-forward cb-example-0000 8091:8091

We need to make sure we leave the command running in the terminal:

![portforward](/images/EKS_kubectl_port_forward.png)

Now we can open up a browser at http://localhost:8091

![loginscreen](/images/EKS_login.png)

We will login using username=`Administrator` and password=`password`.

We are in!

Click the 'Servers' link on the left side and we should see our clusters running.

![webui](/images/EKS_webui.png)

Fin
