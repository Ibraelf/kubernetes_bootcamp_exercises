# Quotas & Limits

## Module

When several users or teams share a cluster with a fixed number of nodes, there is a concern that one team could use more than its fair share of resources. _Resource quotas_ are a tool for administrators to address this concern.

#### Overview

At the end of this module, you will :

* _Learn the format of a YAML Resource Quota and Limit Range file_
* _Learn how to automatically define limits to Pods_
* _Learn the composition of a ResourceQuota_

#### Prerequisites

Create the directory `data/quotas` in your home folder to manage the YAML file needed in this module.

```bash
mkdir ~/data/quotas
```

## Create

The most basic resource metrics for a pod are CPU and memory. Kubernetes provides requests and limits to pre-allocate resources and limit resource usage, respectively.

Limits restrict the resource usage of a pod as follows:

1. If its memory usage exceeds the memory limit, this pod is out of memory \(OOM\) killed.
2. If its CPU usage exceeds the CPU limit, this pod is not killed, but its CPU usage is restricted to the limit.

### ResourceQuota

Kubernetes provides the ResourceQuota object to set constraints on the number of Kubernetes objects by type and the amount of resources \(CPU and memory\) in a namespace.

1. One or more ResourceQuota objects can be created in a namespace.
2. If the ResourceQuota object is configured in a namespace, requests and limits must be set during deployment; otherwise, pod creation is rejected.
3. To avoid this problem, the LimitRange object can be used to set the default requests and limits for each pod.

#### Exercise n°1

{% code-tabs %}
{% code-tabs-item title="~/data/quotas/resourcequota.yaml" %}
```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: myfirstresourcequota
  namespace: default
spec:
  hard:
    requests.cpu: "3"
    requests.memory: 1Gi
    limits.cpu: "5"
    limits.memory: 2Gi
    pods: "5"
```
{% endcode-tabs-item %}
{% endcode-tabs %}

Create the object based on the previous yaml file definition.

```bash
kubectl apply  -f data/quotas/resourcequota.yaml
```

### LimitRange

The LimitRange object is used to set the default resource requests and limits as well as minimum and maximum constraints for each pod in a namespace.

#### Exercise n°1

{% code-tabs %}
{% code-tabs-item title="~/data/limits/limitrange.yaml" %}
```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: myfirstlimitrange
  namespace: default
spec:
  limits:
  - default:  # default limit
      memory: 512Mi
      cpu: 2
    defaultRequest:  # default request
      memory: 256Mi
      cpu: 0.5
    max:  # max limit
      memory: 800Mi
      cpu: 3
    min:  # min request
      memory: 100Mi
      cpu: 0.3
    maxLimitRequestRatio:  # max value for limit / request
      memory: 2
      cpu: 2
    type: Container # limit type, support: Container / Pod / PersistentVolumeClaim
```
{% endcode-tabs-item %}
{% endcode-tabs %}

Create the object based on the previous yaml file definition.

```bash
kubectl apply  -f data/limits/limitrange.yaml
```

## Get

The _get_ command list the object asked. It could be a single object or a list of multiple objects comma separated. This command is useful to get the status of each object. The output can be formatted to only display some information based on some json search or external tools like `tr`, `sort`, `uniq`.

### ResourceQuota

The default output display some useful information about each services :

* Name : the name of the newly created object
* Age : the age since his creation

#### Exercise n°1

Get the resource quota created in the default namespace.

{% tabs %}
{% tab title="Command" %}
```bash
kubectl get resourcequota
```
{% endtab %}

{% tab title="CLI Return" %}
```bash
NAME                   AGE
myfirstresourcequota   27s
```
{% endtab %}
{% endtabs %}

### LimitRange

The default output display some useful information about each services :

* Name : the name of the newly created object
* Age : the age of the object since his creation

#### Exercise n°1

Get the limit range create in the default namespace.

{% tabs %}
{% tab title="Command" %}
```text
kubectl get limitrange
```
{% endtab %}

{% tab title="CLI Return" %}
```bash
NAME                AGE
myfirstlimitrange   1m
```
{% endtab %}
{% endtabs %}

## Describe

Once an object is running, it is inevitably a need to debug problems or check the configuration deployed.

The _describe_ command display a lot of configuration information about the Resource Quotas and Limits \(labels, annotations, etc.\) and the amount of resource \(default, memory, cpu, ...\).

This command is really useful to introspect and debug an object deployed in a cluster.

#### Exercise n°1

Describe one of the existing resource quota in the default namespace.

{% tabs %}
{% tab title="Command" %}
```bash
kubectl describe resourcequota myfirstresourcequota
```
{% endtab %}

{% tab title="CLI Return" %}
```bash
Name:            myfirstresourcequota
Namespace:       default
Resource         Used  Hard
--------         ----  ----
limits.cpu       0     5
limits.memory    0     2Gi
pods             0     5
requests.cpu     0     3
requests.memory  0     1Gi
```
{% endtab %}
{% endtabs %}

#### Exercise n°2

Describe one of the existing limit range in the default namespace.

{% tabs %}
{% tab title="Command" %}
```bash
kubectl describe limitrange myfirstlimitrange
```
{% endtab %}

{% tab title="CLI Return" %}
```bash
Name:       myfirstlimitrange
Namespace:  default
Type        Resource  Min    Max    Default Request  Default Limit  Max Limit/Request Ratio
----        --------  ---    ---    ---------------  -------------  -----------------------
Container   cpu       300m   3      500m             2              2
Container   memory    100Mi  800Mi  256Mi            512Mi          2
```
{% endtab %}
{% endtabs %}

## Explain

Kubernetes come with a lot of documentation about his objects and the available options in each one. Those information can be fin easily in command line or in the official Kubernetes documentation.

The _explain_ command allows to directly ask the API resource via the command line tools to display information about each Kubernetes objects and their architecture.

#### Exercise n°1

Get the documentation of a specific field of a resource quota.

{% tabs %}
{% tab title="Command" %}
```bash
kubectl explain resourcequota
```
{% endtab %}

{% tab title="CLI Return" %}
```bash
KIND:     ResourceQuota
VERSION:  v1

DESCRIPTION:
     ResourceQuota sets aggregate quota restrictions enforced per namespace

FIELDS:
   apiVersion	<string>
     APIVersion defines the versioned schema of this representation of an
     object. Servers should convert recognized schemas to the latest internal
     value, and may reject unrecognized values. More info:
     https://git.k8s.io/community/contributors/devel/api-conventions.md#resources

   kind	<string>
     Kind is a string value representing the REST resource this object
     represents. Servers may infer this from the endpoint the client submits
     requests to. Cannot be updated. In CamelCase. More info:
     https://git.k8s.io/community/contributors/devel/api-conventions.md#types-kinds

   metadata	<Object>
     Standard object's metadata. More info:
     https://git.k8s.io/community/contributors/devel/api-conventions.md#metadata

   spec	<Object>
     Spec defines the desired quota.
     https://git.k8s.io/community/contributors/devel/api-conventions.md#spec-and-status

   status	<Object>
     Status defines the actual enforced quota and its current usage.
     https://git.k8s.io/community/contributors/devel/api-conventions.md#spec-and-status
```
{% endtab %}
{% endtabs %}

#### Exercise n°2

Get the documentation of a specific field of a limit range.

{% tabs %}
{% tab title="Command" %}
```bash
kubectl explain limitrange
```
{% endtab %}

{% tab title="CLI Return" %}
```bash
KIND:     LimitRange
VERSION:  v1

DESCRIPTION:
     LimitRange sets resource usage limits for each kind of resource in a
     Namespace.

FIELDS:
   apiVersion	<string>
     APIVersion defines the versioned schema of this representation of an
     object. Servers should convert recognized schemas to the latest internal
     value, and may reject unrecognized values. More info:
     https://git.k8s.io/community/contributors/devel/api-conventions.md#resources

   kind	<string>
     Kind is a string value representing the REST resource this object
     represents. Servers may infer this from the endpoint the client submits
     requests to. Cannot be updated. In CamelCase. More info:
     https://git.k8s.io/community/contributors/devel/api-conventions.md#types-kinds

   metadata	<Object>
     Standard object's metadata. More info:
     https://git.k8s.io/community/contributors/devel/api-conventions.md#metadata

   spec	<Object>
     Spec defines the limits enforced. More info:
     https://git.k8s.io/community/contributors/devel/api-conventions.md#spec-and-status
```
{% endtab %}
{% endtabs %}

## Delete

The _delete_ command delete resources by filenames, stdin, resources and names, or by resources and label selector.

Be careful on the deletion of a quota or a limit object, this can have effects in the availability of the services associated by increasing the resource consumption.

Note that the delete command does NOT do resource version checks, so if someone submits an update to a resource right when you submit a delete, their update will be lost along with the rest of the resource.

#### Exercise n°1

Delete the previous resource quotas and limit ranges created in command line.

```bash
# Delete the resource quota
kubectl delete resourcequota myfirstresourcequota

# Delete the limit range
kubectl delete limitrange myfirstlimitrange
```

## Module exercise

The purpose of this section is to manage each steps of the lifecycle of an application to better understand each concepts of the Kubernetes course.

The main objective in this module is to understand how to dynamically and automatically manage the limits of CPU and memory of Pods and manage resource quotas of the namespace.

For more information about the application used all along the course, please refer to the _Exercise App &gt; Voting App_ link in the left panel.

Based on the principles explain in this module, try by your own to handle this steps. The development of a yaml file is recommended.

The file developed has to be stored in this directory : `~/data/votingapp/09_quotas`

{% tabs %}
{% tab title="Exercise" %}
1. Limit the amount of available resources in the voting-app namespace has below :
   1. 5 CPU unit can be requested
   2. 4Gi of memory can be requested 
   3. The limit of CPU unit available is 7
   4. The limit of memory available is 6Gi
2. Set the default limits of container resources has below :
   1. By default, a container can request 256Mi of memory and 0.5 of CPU unit
   2. The default limits is 512Mi of memory and 1 CPU unit
3. Check that the default limitation policy deployed is not added to the current running Pods.
4. Restart one or all Pods to update the default limitation policy
5. Describe the restarted Pods to get the limitation
{% endtab %}

{% tab title="Solution" %}
Create the resource quotas to limits the resource allocation.

{% code-tabs %}
{% code-tabs-item title="~/data/votingapp/09\_quotas/quotas.yaml" %}
```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: voting
  namespace: voting-app
spec:
  hard:
    requests.cpu: "5"
    requests.memory: 4Gi
    limits.cpu: "7"
    limits.memory: 6Gi
---
apiVersion: v1
kind: LimitRange
metadata:
  name: voting
  namespace: voting-app
spec:
  limits:
  - default:
      memory: 512Mi
      cpu: 1
    defaultRequest:
      memory: 256Mi
      cpu: 0.5
    type: Container
```
{% endcode-tabs-item %}
{% endcode-tabs %}

Create the resource based on the previous yaml file definition.

```bash
kubectl apply  -f data/votingapp/09_quotas/quotas.yaml
```
{% endtab %}
{% endtabs %}

## External documentation

Those documentations can help you to go further in this topic :

* Kubernetes official documentation on [resource quotas](https://kubernetes.io/docs/concepts/policy/resource-quotas/)
* Kubernetes official documentation on [how to manage resource quotas in a namespace](https://kubernetes.io/docs/tasks/administer-cluster/manage-resources/quota-memory-cpu-namespace/)
* Kubernetes official documentation on[ the configuration of CPU limits ](https://kubernetes.io/docs/tasks/administer-cluster/manage-resources/cpu-default-namespace/)
* Kubernetes official documentation on[ ](https://kubernetes.io/docs/tasks/administer-cluster/manage-resources/cpu-default-namespace/)[the configuation of memory limits](https://kubernetes.io/docs/tasks/administer-cluster/manage-resources/memory-default-namespace/)

