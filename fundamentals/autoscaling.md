---
description: >-
  Autoscaling is a feature in which the cluster is capable of increasing or
  decreasing the number of pods as the demand for service response need it.
---

# Autoscaling

Module Overview

At the end of this module, you will :

* _Learn the format of a YAML Autoscale file_
* _Learn how to manage a Autoscale_
* _Learn the composition of a Autoscale_

## Create

Looks up a Deployment, ReplicaSet, or ReplicationController by name and creates an autoscaler that uses the given resource as a reference. An autoscaler can automatically increase or decrease number of pods deployed within the system as needed.

Horizontal Pod Autoscaler automatically scales the number of pods in a deployment or replica set based on observed CPU, Memory or Custom Metrics utilization depending the API version used.

The Kubernetes basic autoscaling architecture can be schematized like this :

![](../.gitbook/assets/autoscaling_architecture.png)

The _create_ command can directly ask the API resource to create an HorizontalPodAutoscaler in command line or create an HorizontalPodAutoscaler object based on a yaml file definition.

#### Exercise n°1

1. Run a sample app based on a webserver to expose it on port 80.
2. Create an Horizontal Pod Autoscaler to automatically scale the Deployment if the CPU usage is above 50%.

```bash
# Run a sample app in the default namespace
kubectl run php-apache --image=k8s.gcr.io/hpa-example --requests=cpu=200m --limits=cpu=300m --expose --port=80

# Create an Horizontal Pod Autoscaler based on the CPU usage
kubectl autoscale deployment php-apache --cpu-percent=50 --min=3 --max=10
```

## Get

The _get_ command list the object asked. It could be a single object or a list of multiple objects comma separated. This command is useful to get the status of each object. The output can be formatted to only display some information based on some json search or external tools like `tr`, `sort`, `uniq`.

The default output display some useful information about each services :

* Name : The name for the newly created object.
* Reference : The object managed by the autoscaler, like  Pod name, a Deployment name ...
* Targets : The metrics defined to autoscale the referenced resource
* Minpods : The lower limit for the number of pods that can be set by the autoscaler.
* Maxpods : The upper limit for the number of pods that can be set by the autoscaler.
* Replicas : Current replicas number
* Age : the age of the object from his creation

#### Exercise n°1

Get the current HorizontalPodAutoscaler resources in the default namespace.

{% tabs %}
{% tab title="Command" %}
```bash
kubectl get hpa
```
{% endtab %}

{% tab title="CLI Return" %}
```bash
NAME         REFERENCE               TARGETS   MINPODS   MAXPODS   REPLICAS   AGE
php-apache   Deployment/php-apache   0%/50%   3         10        3          16m
```
{% endtab %}
{% endtabs %}

#### Exercise n°2

Stress the Pod created in the previous section and check the HorizontalPoMindAutoscaler associated.

{% tabs %}
{% tab title="Command" %}
```bash
# Connect to the Pod
kubectl run -i --tty load-generator --image=busybox /bin/sh

# Run a loop bash command in the container to stress the CPU
while true; do wget -q -O- http://php-apache.default.svc.cluster.local; done

# Check the Horizontal Pod Autoscaler status
kubectl get hpa
```
{% endtab %}

{% tab title="CLI Return" %}
```bash
NAME         REFERENCE               TARGETS   MINPODS   MAXPODS   REPLICAS   AGE
php-apache   Deployment/php-apache   76%/50%   3         10        5          17m
```
{% endtab %}
{% endtabs %}

#### Exercise n°3

Stop to stress the Pod previously created and check that the autoscaler come back to normal.

{% tabs %}
{% tab title="Command" %}
```bash
kubectl get hpa
```
{% endtab %}

{% tab title="CLI Return" %}
```bash
NAME         REFERENCE               TARGETS   MINPODS   MAXPODS   REPLICAS   AGE
php-apache   Deployment/php-apache   0%/50%    3         10        3          29m
```
{% endtab %}
{% endtabs %}

## Describe

Once an object is running, it is inevitably a need to debug problems or check the configuration deployed.

The _describe_ command display a lot of configuration information about the Horizontal Pod Autoscaler \(labels, annotations, etc.\) and the scale policy \(selector, type, number of pods, ...\).

This command is really useful to introspect and debug an object deployed in a cluster.

#### Exercise n°1

Describe one of the existing Autoscaler in the default namespace.

{% tabs %}
{% tab title="Command" %}
```bash
kubectl describe horizontalpodautoscaler php-apache
```
{% endtab %}

{% tab title="CLI Return" %}
```bash
Name:                                                  php-apache
Namespace:                                             default
Labels:                                                <none>
Annotations:                                           <none>
CreationTimestamp:                                     Wed, 06 Feb 2019 10:39:55 -0500
Reference:                                             Deployment/php-apache
Metrics:                                               ( current / target )
  resource cpu on pods  (as a percentage of request):  0% (0) / 50%
Min replicas:                                          3
Max replicas:                                          10
Conditions:
  Type            Status  Reason               Message
  ----            ------  ------               -------
  AbleToScale     True    ScaleDownStabilized  recent recommendations were higher than current one, applying the highest recent recommendation
  ScalingActive   True    ValidMetricFound     the HPA was able to successfully calculate a replica count from cpu resource utilization (percentage of request)
  ScalingLimited  False   DesiredWithinRange   the desired count is within the acceptable range
Events:
  Type     Reason                        Age                From                       Message
  ----     ------                        ----               ----                       -------
  Normal   SuccessfulRescale             36m                horizontal-pod-autoscaler  New size: 3; reason: Current number of replicas below Spec.MinReplicas
  Normal   SuccessfulRescale             31m                horizontal-pod-autoscaler  New size: 4; reason: cpu resource utilization (percentage of request) above target
  Normal   SuccessfulRescale             28m                horizontal-pod-autoscaler  New size: 5; reason: cpu resource utilization (percentage of request) above target)
  Normal   SuccessfulRescale             2m                 horizontal-pod-autoscaler  New size: 4; reason: All metrics below target
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
kubectl explain hpa.spec
```
{% endtab %}

{% tab title="CLI Return" %}
```bash
KIND:     HorizontalPodAutoscaler
VERSION:  autoscaling/v1

RESOURCE: spec <Object>

DESCRIPTION:
     behaviour of autoscaler. More info:
     https://git.k8s.io/community/contributors/devel/api-conventions.md#spec-and-status.

     specification of a horizontal pod autoscaler.

FIELDS:
   maxReplicas	<integer> -required-
     upper limit for the number of pods that can be set by the autoscaler;
     cannot be smaller than MinReplicas.

   minReplicas	<integer>
     lower limit for the number of pods that can be set by the autoscaler,
     default 1.

   scaleTargetRef	<Object> -required-
     reference to scaled resource; horizontal pod autoscaler will learn the
     current resource consumption and will set the desired number of pods by
     using its Scale subresource.

   targetCPUUtilizationPercentage	<integer>
     target average CPU utilization (represented as a percentage of requested
     CPU) over all the pods; if not specified the default autoscaling policy
     will be used.
```
{% endtab %}
{% endtabs %}

Add the --recursive flag to display all of the fields at once without descriptions.

## Delete

The _delete_ command delete resources by filenames, stdin, resources and names, or by resources and label selector.

Be careful on the deletion of an autoscaling object, this can have effects in the availability of the services associated.

Note that the delete command does NOT do resource version checks, so if someone submits an update to a resource right when you submit a delete, their update will be lost along with the rest of the resource.

#### Exercise n°1

Delete the previous autoscaling group in command line.

```bash
# Delete the HorizontalPodAutoscaler
kubectl delete hpa php-apache

# Delete the Pods
kubectl delete pods php-apache
```

## Module exercise

The purpose of this section is to manage each steps of the lifecycle of an application to better understand each concepts of the Kubernetes course.

The main objective in this module is to understand how to dynamically and automatically manage the number of Pods needed to handle the workload.

For more information about the application used all along the course, please refer to the _Exercise App &gt; Voting App_ link in the left panel.

Based on the principles explain in this module, try by your own to handle this steps. The development of a yaml file is recommended.

The file developed has to be stored in this directory : `/data/votingapp/09_autoscaling`

{% tabs %}
{% tab title="Exercise" %}
1. Manage the HorizontalPodAutoscaler of the worker Pods to :
   1. Ensure that the worker has minimum one Pods
   2. Ensure that the worker has maximum five Pods
   3. Ensure that the Pods is autoscaled when the CPU is above 80%.
{% endtab %}

{% tab title="Solution" %}
Create the HorizontalPodAutoscaler to manage the worker workload.

```bash
kubectl autoscale deployment worker -n voting-app --cpu-percent=80 --min=1 --max=5
```

This can be done with a yaml file definition :

```yaml
apiVersion: autoscaling/v2beta2
kind: HorizontalPodAutoscaler
metadata:
  name: worker
  namespace: voting-app
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: worker
  minReplicas: 1
  maxReplicas: 5
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 80
```
{% endtab %}
{% endtabs %}

## External documentation

Those documentations can help you to go further in this topic :

* Kubernetes official documentation on [Horizontal Pod Auto Scaling](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/) \(HPA\)
* Kubernetes official documentation [walkthrough HPA](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/)
* Kubernetes official documentation of [autoscale API](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#autoscale)



