---
description: >-
  Kubernetes networks define how the pods in the same namespace will communicate
  with each other and the network endpoint.
---

# Networks

## Module

_Networking_ is a central part of Kubernetes, but it can be challenging to understand exactly how it is expected to work.

#### Overview

At the end of this module, you will :

* _Learn what is a CNI_
* _Learn to manage network rules_
* _Learn to secure the communication within a cluster_

#### Prerequisites

Create the directory `data/networks` in your home folder to manage the YAML file needed in this module.

```bash
mkdir ~/data/networks
```

## Create

A network policy is a specification of how groups of pods are allowed to communicate with each other and other network endpoints.

NetworkPolicy resources use labels to select pods and define rules which specify what traffic is allowed to the selected pods.

By default, if no policies exist in a namespace, then all ingress and egress traffic is allowed to and from pods in that namespace

The _create_ command can create a NetworkPolicy object based on a yaml file definition.

#### Exercise n°1

Deploy a default Network Policy for each resources in the default namespace to deny all ingress and egress traffic.

{% code-tabs %}
{% code-tabs-item title="~/data/networks/01\_networkpolicy.yaml" %}
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: defaultnetworkpolicy
  namespace: default
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress
```
{% endcode-tabs-item %}
{% endcode-tabs %}

Create the resource based on the previous yaml file definition.

```bash
kubectl apply  -f data/networks/01_networkpolicy.yaml
```

#### Exercise n°2

Create a network policy to :

* Allow the ingress traffic from :
  * The IP range : 172.17.0.0/16
  * The namespace transvers
  * The Pods labelized with the key _role_ and the value _frontend_
  * The port 5432
* Allow the egress traffic to :
  * The local network 10.0.0.0/24 on port 5432

{% code-tabs %}
{% code-tabs-item title="~/data/networks/02\_networkpolicy.yaml" %}
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: testnetworkpolicy
  namespace: default
spec:
  podSelector:
    matchLabels:
      role: db
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - ipBlock:
        cidr: 172.17.0.0/16
    - namespaceSelector:
        matchLabels:
          env: transvers
    - podSelector:
        matchLabels:
          role: frontend
    ports:
    - protocol: TCP
      port: 5432
  egress:
  - to:
    - ipBlock:
        cidr: 10.0.0.0/24
    ports:
    - protocol: TCP
      port: 5432
```
{% endcode-tabs-item %}
{% endcode-tabs %}

Create the resource based on the previous yaml file definition.

```bash
kubectl apply  -f data/networks/02_networkpolicy.yaml
```

## Get

The _get_ command list the object asked. It could be a single object or a list of multiple objects comma separated. This command is useful to get the status of each object. The output can be formatted to only display some information based on some json search or external tools like `tr`, `sort`, `uniq`.

The default output display some useful information about each services :

* Name : the name of the newly created resource
* Pod-selector : the labels used to associate the Network Policy with the Pods
* Age :  the age since his creation

#### Exercise n°1

{% tabs %}
{% tab title="Command" %}
```bash
kubectl get networkpolicy
```
{% endtab %}

{% tab title="CLI Return" %}
```bash
NAME                   POD-SELECTOR   AGE
defaultnetworkpolicy   <none>         2m
testnetworkpolicy      role=db        5s
```
{% endtab %}
{% endtabs %}

## Describe

Once an object is running, it is inevitably a need to debug problems or check the configuration deployed.

The _describe_ command display a lot of configuration information about the NetworkPolicies \(labels, annotations, etc.\) and the specific policies associated to the current NetworkPolicy \(type, port, selectors, etc\).

This command is really useful to introspect and debug an object deployed in a cluster.

#### Exercise n°1

Describe one of the existing Network Policy in the default namespace.

{% tabs %}
{% tab title="Command" %}
```bash
kubectl describe networkpolicy testnetworkpolicy
```
{% endtab %}

{% tab title="CLI Return" %}
```bash
Name:         testnetworkpolicy
Namespace:    default
Created on:   2019-02-13 14:59:12 -0500 EST
Labels:       <none>
Annotations:  <none>
Spec:
  PodSelector:     role=db
  Allowing ingress traffic:
    To Port: 6379/TCP
    From IPBlock:
        CIDR: 172.17.0.0/16
        Except: 172.17.1.0/24
    From NamespaceSelector: project=myproject
    From PodSelector: role=frontend
  Allowing egress traffic:
    To Port: 5978/TCP
    To IPBlock:
        CIDR: 10.0.0.0/24
        Except: 
  Policy Types: Ingress, Egress
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
kubectl explain networkpolicy.spec
```
{% endtab %}

{% tab title="CLI Return" %}
```bash
KIND:     NetworkPolicy
VERSION:  extensions/v1beta1

RESOURCE: spec <Object>

DESCRIPTION:
     Specification of the desired behavior for this NetworkPolicy.

     DEPRECATED 1.9 - This group version of NetworkPolicySpec is deprecated by
     networking/v1/NetworkPolicySpec.

FIELDS:
   egress	<[]Object>
     List of egress rules to be applied to the selected pods. Outgoing traffic
     is allowed if there are no NetworkPolicies selecting the pod (and cluster
     policy otherwise allows the traffic), OR if the traffic matches at least
     one egress rule across all of the NetworkPolicy objects whose podSelector
     matches the pod. If this field is empty then this NetworkPolicy limits all
     outgoing traffic (and serves solely to ensure that the pods it selects are
     isolated by default). This field is beta-level in 1.8

   ingress	<[]Object>
     List of ingress rules to be applied to the selected pods. Traffic is
     allowed to a pod if there are no NetworkPolicies selecting the pod OR if
     the traffic source is the pod's local node, OR if the traffic matches at
     least one ingress rule across all of the NetworkPolicy objects whose
     podSelector matches the pod. If this field is empty then this NetworkPolicy
     does not allow any traffic (and serves solely to ensure that the pods it
     selects are isolated by default).

   podSelector	<Object> -required-
     Selects the pods to which this NetworkPolicy object applies. The array of
     ingress rules is applied to any pods selected by this field. Multiple
     network policies can select the same set of pods. In this case, the ingress
     rules for each are combined additively. This field is NOT optional and
     follows standard label selector semantics. An empty podSelector matches all
     pods in this namespace.

   policyTypes	<[]string>
     List of rule types that the NetworkPolicy relates to. Valid options are
     Ingress, Egress, or Ingress,Egress. If this field is not specified, it will
     default based on the existence of Ingress or Egress rules; policies that
     contain an Egress section are assumed to affect Egress, and all policies
     (whether or not they contain an Ingress section) are assumed to affect
     Ingress. If you want to write an egress-only policy, you must explicitly
     specify policyTypes [ "Egress" ]. Likewise, if you want to write a policy
     that specifies that no egress is allowed, you must specify a policyTypes
     value that include "Egress" (since such a policy would not include an
     Egress section and would otherwise default to just [ "Ingress" ]). This
     field is beta-level in 1.8
```
{% endtab %}
{% endtabs %}

Add the --recursive flag to display all of the fields at once without descriptions.

## Delete

The _delete_ command delete resources by filenames, stdin, resources and names, or by resources and label selector.

Be careful on the deletion of a network policy, this can isolate resource or expose it and cause damaged to your services.

Note that the delete command does NOT do resource version checks, so if someone submits an update to a resource right when you submit a delete, their update will be lost along with the rest of the resource.

#### Exercise n°1

Delete the previous network policy in command line.

```bash
kubectl delete networkpolicy defaultnetworkpolicy testnetworkpolicy
```

## Module exercise

The purpose of this section is to manage each steps of the lifecycle of an application to better understand each concepts of the Kubernetes course.

The main objective in this module is to understand how to secure the communication between Pods.

For more information about the application used all along the course, please refer to the _Exercise App &gt; Voting App_ link in the left panel.

Based on the principles explain in this module, try by your own to handle this steps. The development of a yaml file is recommended.

The file developed has to be stored in this directory : `~/data/votingapp/11_networks`

{% tabs %}
{% tab title="Exercise" %}
1. Create the default NetworkPolices of the voting-app namespace to :
   1. Deny all Ingress traffic
   2. Allow all Egress traffic
2. Create the NetworkPolicies of the database Pods to :
   1. Allow the Ingress traffic on port 5432 only from the result and worker Pods
3. Create the NetworkPolicies of the redis Pods to :
   1. Allow the Ingress traffic on port 6379 only from the vote and worker Pods
4. Create the NetworkPolicies of the result Pods to :
   1. Allow the Ingress traffic on port 8080 only from everywhere
5. Create the NetworkPolicies of the vote Pods to :
   1. Allow the Ingress traffic on port 8080 only from everywhere
{% endtab %}

{% tab title="Solution" %}
An example yaml file definition to handle the NetworkPolicies.

{% code-tabs %}
{% code-tabs-item title="~/data/votingapp/11\_networks/networkpolicies.yaml" %}
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: voting
  namespace: voting-app
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress
  egress:
    - {}
---
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: db
  namespace: voting-app
spec:
  podSelector:
    matchLabels:
      name: db
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          name: result
    - podSelector:
        matchLabels:
          name: worker
    ports:
    - protocol: TCP
      port: 5432
---
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: redis
  namespace: voting-app
spec:
  podSelector:
    matchLabels:
      name: redis
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          name: vote
    - podSelector:
        matchLabels:
          name: worker
    ports:
    - protocol: TCP
      port: 6379
---
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: result
  namespace: voting-app
spec:
  podSelector:
    matchLabels:
      name: result
  policyTypes:
  - Ingress
  ingress:
  - from:
    - ipBlock:
        cidr: 0.0.0.0/0
    ports:
    - protocol: TCP
      port: 8080
---
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: vote
  namespace: voting-app
spec:
  podSelector:
    matchLabels:
      name: vote
  policyTypes:
  - Ingress
  ingress:
  - from:
    - ipBlock:
        cidr: 0.0.0.0/0
    ports:
    - protocol: TCP
      port: 8080
```
{% endcode-tabs-item %}
{% endcode-tabs %}

Create the NetworkPolicies based on the previous yaml file.

```bash
kubectl apply  -f data/votingapp/11_networks/networkpolicies.yaml
```
{% endtab %}
{% endtabs %}

## External documentation

Those documentations can help you to go further in this topic :

* Kubernetes official documentation on [Network Policy](https://kubernetes.io/docs/concepts/services-networking/network-policies/)
* [Kubernetes network plugins (CNI) benchmark](https://itnext.io/benchmark-results-of-kubernetes-network-plugins-cni-over-10gbit-s-network-updated-august-2020-6e1b757b9e49)

