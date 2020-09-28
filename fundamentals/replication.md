---
description: >-
  A ReplicaSet ensures that a specified number of pod replicas are running at
  any given time.
---

# ReplicaSet

## Module

A _ReplicaSet_’s purpose is to maintain a stable set of replica Pods running at any given time. As such, it is often used to guarantee the availability of a specified number of identical Pods.

#### Overview

At the end of this module, you will :

* _Learn what is a Replicaset_
* _Learn how to scale a Pod_
* _Understand why replication is part of High Availability_

#### Prerequisites

Create the directory `data/replicaset` in your home folder to manage the YAML file needed in this module.

```bash
mkdir ~/data/replicaset
```

## Create

A ReplicaSet can be easily scaled up or down by simply updating the .spec.replicas field in a YAML file definition. The ReplicaSet controller ensures that a desired number of pods with a matching label selector are available and operational

The _create_ command create a ReplicaSet object based on a yaml file definition to manage the number of a pods identified by his labels.

#### Exercise n°1

Create a ReplicaSet object to scale an Nginx Pod with 3 replicas.

{% code-tabs %}
{% code-tabs-item title="~/data/replicaset/01\_replicaset.yaml" %}
```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: mynginxreplicaset
  labels:
    app: mynginxreplicaset
    env: formation
spec:
  replicas: 3
  selector:
    matchLabels:
      app: mynginxreplicaset
    matchExpressions:
      - {key: app, operator: In, values: [mynginxreplicaset]}
  template:
    metadata:
      labels:
        app: mynginxreplicaset
        env: formation
    spec:
      containers:
      - name: nginx
        image: nginx
```
{% endcode-tabs-item %}
{% endcode-tabs %}

```bash
kubectl apply  -f data/replicaset/01_replicaset.yaml
```

## Get

The _get_ command list the object asked. It could be a single object or a list of multiple objects comma separated. This command is useful to get the status of each object. The output can be formatted to only display some information based on some json search or external tools like `tr`, `sort`, `uniq`.

The default output of a get ReplicaSet command display three replicas status :

* Name : the name of the newly created resource
* Desired : the number of replicas defined in the YAML file definition
* Current : the current number of replicas deployed in the cluster
* Ready : the number of production ready replicas in the cluster
* Age : the age since his creation

#### Exercise n°1

List the ReplicaSet created in the default namespace.

{% tabs %}
{% tab title="Command" %}
```bash
kubectl get replicaset
```
{% endtab %}

{% tab title="CLI Return" %}
```bash
NAME                DESIRED   CURRENT   READY     AGE
mynginxreplicaset   3         3         3         10s
```
{% endtab %}
{% endtabs %}

## Describe

Once an object is running, it is inevitably a need to debug problems or check the configuration deployed.

The _describe_ command display a lot of configuration information about the ReplicaSet\(s\) \(labels, resource requirements, etc.\) or any other Kubernetes objects, as well as status information about the ReplicaSet\(s\) and Pod \(state, readiness, restart count, events, etc.\).

This command is really useful to introspect and debug an object deployed in a cluster.

#### Exercise n°1

Describe the ReplicaSet created in the default namespace.

{% tabs %}
{% tab title="Command" %}
```bash
kubectl describe replicaset mynginxreplicaset
```
{% endtab %}

{% tab title="CLI Return" %}
```bash
Name:         mynginxreplicaset
Namespace:    default
Selector:     app=mynginxreplicaset,app in (mynginxreplicaset)
Labels:       app=mynginxreplicaset
              env=formation
Annotations:  <none>
Replicas:     3 current / 3 desired
Pods Status:  3 Running / 0 Waiting / 0 Succeeded / 0 Failed
Pod Template:
  Labels:  app=mynginxreplicaset
           env=formation
  Containers:
   nginx:
    Image:        nginx
    Port:         <none>
    Host Port:    <none>
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Events:
  Type    Reason            Age   From                   Message
  ----    ------            ----  ----                   -------
  Normal  SuccessfulCreate  48s   replicaset-controller  Created pod: mynginxreplicaset-qqcwc
  Normal  SuccessfulCreate  48s   replicaset-controller  Created pod: mynginxreplicaset-vmkfm
  Normal  SuccessfulCreate  48s   replicaset-controller  Created pod: mynginxreplicaset-st4kr

```
{% endtab %}
{% endtabs %}

## Explain

Kubernetes come with a lot of documentation about his objects and the available options in each one. Those information can be fin easily in command line or in the official Kubernetes documentation.

The _explain_ command allows to directly ask the API resource via the command line tools to display information about each Kubernetes objects and their architecture.

#### Exercise n°1

Get the documentation of a specific field of a resource.

{% tabs %}
{% tab title="Command" %}
```bash
kubectl explain replicaset.spec
```
{% endtab %}

{% tab title="CLI Return" %}
```bash
KIND:     ReplicaSet
VERSION:  extensions/v1beta1

RESOURCE: spec <Object>

DESCRIPTION:
     Spec defines the specification of the desired behavior of the ReplicaSet.
     More info:
     https://git.k8s.io/community/contributors/devel/api-conventions.md#spec-and-status

     ReplicaSetSpec is the specification of a ReplicaSet.

FIELDS:
   minReadySeconds	<integer>
     Minimum number of seconds for which a newly created pod should be ready
     without any of its container crashing, for it to be considered available.
     Defaults to 0 (pod will be considered available as soon as it is ready)

   replicas	<integer>
     Replicas is the number of desired replicas. This is a pointer to
     distinguish between explicit zero and unspecified. Defaults to 1. More
     info:
     https://kubernetes.io/docs/concepts/workloads/controllers/replicationcontroller/#what-is-a-replicationcontroller

   selector	<Object>
     Selector is a label query over pods that should match the replica count. If
     the selector is empty, it is defaulted to the labels present on the pod
     template. Label keys and values that must match in order to be controlled
     by this replica set. More info:
     https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/#label-selectors

   template	<Object>
     Template is the object that describes the pod that will be created if
     insufficient replicas are detected. More info:
     https://kubernetes.io/docs/concepts/workloads/controllers/replicationcontroller#pod-template

```
{% endtab %}
{% endtabs %}

Add the --recursive flag to display all of the fields at once without descriptions.

## Scale

Some Kubernetes objects like Deployment, ReplicaSet, Replication Controller, or Job can be easily scaled in command line.

The command line management has to be used only for debug purpose. In production environment, it's recommended to update the YAML file definition to ensure a consistent deployment.

#### Exercise n°1

Scale the ReplicaSet mynginxreplicaset to 5 in command line.

```bash
kubectl scale replicaset mynginxreplicaset --replicas=5
```

List the ReplicaSet created in the default namespace.

```bash
kubectl get replicaset
```

## Delete

The _delete_ command delete resources by filenames, stdin, resources and names, or by resources and label selector.

Deleting a ReplicaSet will automatically delete each Pods associated with.

Note that the delete command does NOT do resource version checks, so if someone submits an update to a resource right when you submit a delete, their update will be lost along with the rest of the resource.

#### Exercise n°1

Delete the ReplicaSet mynginxreplicaset in command line.

```bash
kubectl delete replicaset mynginxreplicaset
```

## Module exercise

The purpose of this section is to manage each steps of the lifecycle of an application to better understand each concepts of the Kubernetes course.

The main objective in this module is to extract each part of the Voting App in a distinct Pod and manage it with a ReplicaSet.

For more information about the application used all along the course, please refer to the _Exercise App &gt; Voting App_ link in the left panel.

Based on the principles explain in this module, try by your own to handle this steps. The development of a yaml file is recommended.

The file developed has to be stored in this directory : `~/data/votingapp/02_replicas`

{% tabs %}
{% tab title="Exercise" %}
1. Delete the Pods created in the previous module exercise
2. Create a ReplicaSet to replicate the worker Pod to 1
3. Ensure the Pod is replicate to 1 and it is up and running
4. Scale the worker Pods to 3 in command line
5. Ensure the worker Pod is the only Pods replicate to 3 and it is up and running
{% endtab %}

{% tab title="Solution" %}
Delete the previous Pods created

```bash
kubectl delete -f ~/data/votingapp/01_pods/pods.yaml
```

Yaml file definition to deploy the Voting App ReplicaSet.

{% code-tabs %}
{% code-tabs-item title="~/data/votingapp/02\_replicas/replicaset.yaml" %}
```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: db
  namespace: voting-app
  labels:
    app: voting-app
spec:
  replicas: 1
  selector:
    matchLabels:
      name: db
  template:
    metadata:
      labels:
        name: db
    spec:
      containers:
      - name: db
        image: postgres:10.4
---
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: redis
  namespace: voting-app
  labels:
    app: voting-app
spec:
  replicas: 1
  selector:
    matchLabels:
      name: redis
  template:
    metadata:
      labels:
        name: redis
    spec:
      containers:
      - name: redis
        image: redis
---
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: result
  namespace: voting-app
  labels:
    app: voting-app
spec:
  replicas: 1
  selector:
    matchLabels:
      name: result
  template:
    metadata:
      labels:
        name: result
    spec:
      containers:
      - name: result
        image: wikitops/examplevotingapp-result:1.1
---
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: vote
  namespace: voting-app
  labels:
    app: voting-app
spec:
  replicas: 1
  selector:
    matchLabels:
      name: vote
  template:
    metadata:
      labels:
        name: vote
    spec:
      containers:
      - name: vote
        image: wikitops/examplevotingapp-vote:1.1
---
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: worker
  namespace: voting-app
  labels:
    app: voting-app
spec:
  replicas: 1
  selector:
    matchLabels:
      name: worker
  template:
    metadata:
      labels:
        name: worker
    spec:
      containers:
      - name: worker
        image: wikitops/examplevotingapp-worker:1.1
```
{% endcode-tabs-item %}
{% endcode-tabs %}

Kubectl command to deploy the Pods based on the previous definition file.

```text
kubectl apply  -f data/votingapp/02_replicas/replicaset.yaml
```

Ensure the Pods is up and running.

```bash
kubectl get pods,replicaset -n voting-app
```

Scale the worker Pod to 3 replicas.

```bash
kubectl scale replicaset worker --replicas=3 -n voting-app
```

Ensure the worker Pod has been replicated.

```bash
kubectl get pods,replicaset -n voting-app
```
{% endtab %}
{% endtabs %}

## External documentation

Those documentations can help you to go further in this topic :

* Kubernetes official documentation on [ReplicaSet](https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/)
* Kubernetes official documentation on [label selector](https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/#label-selectors) to understand difference between ReplicationController and ReplicaSet

