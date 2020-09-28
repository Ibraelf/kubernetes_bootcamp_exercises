# Service Accounts

## Module

A _service account_ provides an identity for processes that run in a Pod.

#### Overview

At the end of this module, you will :

* _Learn to create a service accounts_
* _Learn how to manage access_
* _Learn how to secure a Pods_

#### Prerequisites

Create the directory `data/sa` in your home folder to manage the YAML file needed in this module.

```bash
mkdir ~/data/serviceaccount
```

## Create

Kubernetes service accounts provide access control to the Kubernetes API for services running in pods.

Kubernetes enables access control for workloads by providing service accounts. A service account represents an identity for processes that run in a pod. When a process is authenticated through a service account, it can contact the API server and access cluster resources. If a pod doesn’t have an assigned service account, it gets the default service account.

The _create_ command can create a Ingress object based on a yaml file definition.

#### Exercise n°1

Create a service account call myfirstserviceaccount.

```bash
kubectl create serviceaccount myfirstserviceaccount
```

#### Exercise n°2

Create a service account named _mysecondserviceaccount_ with a yaml file definition.

{% code-tabs %}
{% code-tabs-item title="~/data/serviceaccount/01\_sa.yaml" %}
```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: mysecondserviceaccount
```
{% endcode-tabs-item %}
{% endcode-tabs %}

Create a resource based on the previous yaml file definition.

```bash
kubectl apply  -f data/serviceaccount/01_sa.yaml
```

## Get

The _get_ command list the object asked. It could be a single object or a list of multiple objects comma separated. This command is useful to get the status of each object. The output can be formatted to only display some information based on some json search or external tools like `tr`, `sort`, `uniq`.

The default output display some useful information about each services :

* Name : the name of the newly created resource
* Secrets : the number of secrets associated
* Age : the age since his creation

#### Exercise n°1

 List the current service account resources created.

{% tabs %}
{% tab title="Command" %}
```bash
kubectl get serviceaccounts
```
{% endtab %}

{% tab title="CLI Return" %}
```bash
NAME                       SECRETS   AGE
default                    1         1h
myfirstserviceaccount      1         33m
```
{% endtab %}
{% endtabs %}

## Describe

Once an object is running, it is inevitably a need to debug problems or check the configuration deployed.

The _describe_ command display a lot of configuration information about the Service Accounts \(labels, annotations, etc.\).

This command is really useful to introspect and debug an object deployed in a cluster.

#### Exercise n°1

Describe the service account myfirstserviceaccount previously created.

{% tabs %}
{% tab title="Command" %}
```bash
kubectl describe serviceaccount myfirstserviceaccount
```
{% endtab %}

{% tab title="CLI Return" %}
```bash
Name:                myfirstserviceaccount
Namespace:           default
Labels:              <none>
Annotations:         <none>
Image pull secrets:  <none>
Mountable secrets:   myfirstserviceaccount-token-59xwk
Tokens:              myfirstserviceaccount-token-59xwk
Events:              <none>
```
{% endtab %}
{% endtabs %}

## Explain

Kubernetes come with a lot of documentation about his objects and the available options in each one. Those information can be fin easily in command line or in the official Kubernetes documentation.

The _explain_ command allows to directly ask the API resource via the command line tools to display information about each Kubernetes objects and their architecture.

#### Exercise n°1

Get the documentation of a specific field of a service account resource.

{% tabs %}
{% tab title="Command" %}
```bash
kubectl explain serviceaccount
```
{% endtab %}

{% tab title="CLI Return" %}
```bash
KIND:     ServiceAccount
VERSION:  v1

DESCRIPTION:
     ServiceAccount binds together: * a name, understood by users, and perhaps
     by peripheral systems, for an identity * a principal that can be
     authenticated and authorized * a set of secrets

FIELDS:
   apiVersion	<string>
     APIVersion defines the versioned schema of this representation of an
     object. Servers should convert recognized schemas to the latest internal
     value, and may reject unrecognized values. More info:
     https://git.k8s.io/community/contributors/devel/api-conventions.md#resources

   automountServiceAccountToken	<boolean>
     AutomountServiceAccountToken indicates whether pods running as this service
     account should have an API token automatically mounted. Can be overridden
     at the pod level.

   imagePullSecrets	<[]Object>
     ImagePullSecrets is a list of references to secrets in the same namespace
     to use for pulling any images in pods that reference this ServiceAccount.
     ImagePullSecrets are distinct from Secrets because Secrets can be mounted
     in the pod, but ImagePullSecrets are only accessed by the kubelet. More
     info:
     https://kubernetes.io/docs/concepts/containers/images/#specifying-imagepullsecrets-on-a-pod

   kind	<string>
     Kind is a string value representing the REST resource this object
     represents. Servers may infer this from the endpoint the client submits
     requests to. Cannot be updated. In CamelCase. More info:
     https://git.k8s.io/community/contributors/devel/api-conventions.md#types-kinds

   metadata	<Object>
     Standard object's metadata. More info:
     https://git.k8s.io/community/contributors/devel/api-conventions.md#metadata

   secrets	<[]Object>
     Secrets is the list of secrets allowed to be used by pods running using
     this ServiceAccount. More info:
     https://kubernetes.io/docs/concepts/configuration/secret

```
{% endtab %}
{% endtabs %}

## Delete

The _delete_ command delete resources by filenames, stdin, resources and names, or by resources and label selector.

A service account can be deleted only if it is not used by a running Kubernetes resource.

Note that the delete command does NOT do resource version checks, so if someone submits an update to a resource right when you submit a delete, their update will be lost along with the rest of the resource.

#### Exercise n°1

Delete the previous service account created in command line.

```bash
kubectl delete serviceaccount myfirstserviceaccount
```

## Module exercise

The purpose of this section is to manage each steps of the lifecycle of an application to better understand each concepts of the Kubernetes course.

The main objective in this module is to understand how to secure the internal access to respect the least privileges principles.

For more information about the application used all along the course, please refer to the _Exercise App &gt; Voting App_ link in the left panel.

Based on the principles explain in this module, try by your own to handle this steps. The development of a yaml file is recommended.

The file developed has to be stored in this directory : `~/data/votingapp/14_serviceaccounts`

{% tabs %}
{% tab title="Exercise" %}
1. Create a service account called vote
2. Create a service account called result
3. Configure the vote Pods to use the vote service accounts created
4. Configure the vote Result to use the result service accounts created
{% endtab %}

{% tab title="Solution" %}
Create the service accounts vote and result in command line.

```bash
# Create the vote service account
kubectl create serviceaccount vote

# Create the result service account
kubectl create serviceaccount result
```

Configure the vote Pods to use the vote service account.

{% code-tabs %}
{% code-tabs-item title="~/data/votingapp/14\_serviceaccounts/vote.yaml" %}
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: vote
  namespace: voting-app
spec:
  replicas: 1
  selector:
    matchLabels:
      name: vote
  strategy:
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 1
    type: RollingUpdate
  template:
    metadata:
      labels:
        name: vote
    spec:
      serviceAccountName: vote
      automountServiceAccountToken: false
      containers:
        - env:
           - name: "OPTION_A"
             valueFrom:
               configMapKeyRef:
                 name: vote
                 key: option_a
           - name: "OPTION_B"
             valueFrom:
               configMapKeyRef:
                 name: vote
                 key: option_b
          image: wikitops/examplevotingapp-vote:1.1
          imagePullPolicy: IfNotPresent
          name: vote
          ports:
            - containerPort: 8080
              name: vote
              protocol: TCP
```
{% endcode-tabs-item %}
{% endcode-tabs %}

Configure the result Pods to use the result service account.

{% code-tabs %}
{% code-tabs-item title="~/data/votingapp/14\_serviceaccounts/result.yaml" %}
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: result
  namespace: voting-app
spec:
  replicas: 1
  selector:
    matchLabels:
      name: result
  strategy:
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 1
    type: RollingUpdate
  template:
    metadata:
      labels:
        name: result
    spec:
      serviceAccountName: result
      automountServiceAccountToken: false
      containers:
        - env:
            - name: "POSTGRES_DB"
              valueFrom:
                secretKeyRef:
                  name: db
                  key: name
            - name: "POSTGRES_NAME"
              valueFrom:
                secretKeyRef:
                  name: db
                  key: user
            - name: "POSTGRES_PASSWORD"
              valueFrom:
                secretKeyRef:
                  name: db
                  key: password
          image: wikitops/examplevotingapp-result:1.1
          imagePullPolicy: IfNotPresent
          name: result
          ports:
            - name: result
              containerPort: 8080
```
{% endcode-tabs-item %}
{% endcode-tabs %}

Create the resources based on the previous yaml files definition.

```bash
kubectl apply  -f data/votingapp/14_serviceaccounts/
```
{% endtab %}
{% endtabs %}

## External documentation

Those documentations can help you to go further in this topic :

* Kubernetes official documentation to [manage service accounts](https://kubernetes.io/docs/reference/access-authn-authz/service-accounts-admin/)
* Kubernetes official documentation to [configure service accounts](https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/)

