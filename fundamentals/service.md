# Services

## Module

A Kubernetes _Service_ is an abstraction which defines a logical set of Pods and a policy by which to access them.

#### Overview

At the end of the module, you will :

* _Learn how to expose a Pod internally_
* _Learn how the domain name resolution works on a Kubernetes cluster_
* _Learn how to expose a Pod externally_

#### Prerequisites

Create the directory `data/services` in your home folder to manage the YAML file needed in this module.

```bash
mkdir ~/data/services
```

## Create

Kubernetes Pods are mortal. They are born and when they die, they are not resurrected. ReplicaSets in particular create and destroy Pods dynamically. While each Pod gets its own IP address, even those IP addresses cannot be relied upon to be stable over time. This leads to a problem.

A Kubernetes Service is an abstraction which defines a logical set of Pods and a policy by which to access them. The set of Pods targeted by a Service is determined by a label selector.

Kubernetes manage four kinds of services depending on the exposition needed for the Pods attached :

| Type | Description |
| :--- | :--- |
| ClusterIP | Only accessed within the Kubernetes cluster |
| NodePort | Host exposition accessible from the local network |
| LoadBalancer | External exposition, Internet access |
| Endpoint | External endpoint definition to easily use it in the cluster |

The _create_ command can directly ask the API resource to create a Service in command line or create a Service object based on a yaml file definition.

#### Exercise n°1

Create an Nginx Deployement and expose the Nginx Deployemnt to be able to access it from the local network on HTTP \(80\) port in command line.

```bash
# Create an Nginx Pods
kubectl run nginx --image=nginx --port=80
# Expose the Nginx Pods on port 80
kubectl expose po nginx --port=80 --type=NodePort
```

#### Exercise n°2

Update the Nginx Pod in the namespace app-demo to be able to access it on HTTP \(80\) and HTTPS \(443\) port thanks to a YAML file.

{% code-tabs %}
{% code-tabs-item title="~/data/services/01\_service.yaml" %}
```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx
spec:
  ports:
  - name: http
    port: 80
    protocol: TCP
    targetPort: 80
  - name: https
    port: 443
    protocol: TCP
    targetPort: 443
  selector:
    run: nginx
  type: NodePort
```
{% endcode-tabs-item %}
{% endcode-tabs %}

Update the service with the new yaml file definition.

```bash
kubectl apply -f ~/data/services/01_service.yaml
```

## Get

The _get_ command list the object asked. It could be a single object or a list of multiple objects comma separated. This command is useful to get the status of each object. The output can be formatted to only display some information based on some json search or external tools like `tr`, `sort`, `uniq`.

The default output display some useful information about each services :

* Name : the name of the newly created resource
* Type : the type of exposition defined at service creation \(see previous table\).
* Cluster-IP : the internal cluster IP of the resources
* External-IP : the external resources IP of the resources, field configured when LoadBalancer type is used
* Port\(s\) : a list of ports opened in the pods managed by the service
* Age : the age since his creation

#### Exercise n°1

Get the list of existing service in the default namespace.

{% tabs %}
{% tab title="Command" %}
```bash
kubectl get service
```
{% endtab %}

{% tab title="CLI Return" %}
```bash
NAME         TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                      AGE
kubernetes   ClusterIP   10.96.0.1        <none>        443/TCP                      1h
nginx        NodePort    10.104.208.250   <none>        80:31006/TCP,443:30905/TCP   57
```
{% endtab %}
{% endtabs %}

## Describe

Once an object is running, it is inevitably a need to debug problems or check the configuration deployed.

The _describe_ command display a lot of configuration information about the Service\(s\) \(labels, resource requirements, etc.\) or any other Kubernetes objects, as well as status information about the Service\(s\) and Pod \(state, readiness, restart count, events, etc.\).

This command is really useful to introspect and debug an object deployed in a cluster.

#### Exercise n°1

Describe one of the existing service in the default namespace.

{% tabs %}
{% tab title="Command" %}
```bash
kubectl describe service nginx
```
{% endtab %}

{% tab title="CLI Return" %}
```bash
Name:                     nginx
Namespace:                default
Labels:                   run=nginx
Annotations:              kubectl.kubernetes.io/last-applied-configuration={"apiVersion":"v1","kind":"Service","metadata":{"annotations":{},"name":"nginx","namespace":"default"},"spec":{"ports":[{"name":"http","port":80,"proto...
Selector:                 run=nginx
Type:                     NodePort
IP:                       10.104.208.250
Port:                     http  80/TCP
TargetPort:               80/TCP
NodePort:                 http  31006/TCP
Endpoints:                172.17.0.7:80
Port:                     https  443/TCP
TargetPort:               443/TCP
NodePort:                 https  30905/TCP
Endpoints:                172.17.0.7:443
Session Affinity:         None
External Traffic Policy:  Cluster
Events:                   <none>

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
kubectl explain service.spec
```
{% endtab %}

{% tab title="CLI Return" %}
```bash
KIND:     Service
VERSION:  v1

RESOURCE: spec <Object>

DESCRIPTION:
     Spec defines the behavior of a service.
     https://git.k8s.io/community/contributors/devel/api-conventions.md#spec-and-status

     ServiceSpec describes the attributes that a user creates on a service.

FIELDS:
   clusterIP	<string>
     clusterIP is the IP address of the service and is usually assigned randomly
     by the master. If an address is specified manually and is not in use by
     others, it will be allocated to the service; otherwise, creation of the
     service will fail. This field can not be changed through updates. Valid
     values are "None", empty string (""), or a valid IP address. "None" can be
     specified for headless services when proxying is not required. Only applies
     to types ClusterIP, NodePort, and LoadBalancer. Ignored if type is
     ExternalName. More info:
     https://kubernetes.io/docs/concepts/services-networking/service/#virtual-ips-and-service-proxies

   externalIPs	<[]string>
     externalIPs is a list of IP addresses for which nodes in the cluster will
     also accept traffic for this service. These IPs are not managed by
     Kubernetes. The user is responsible for ensuring that traffic arrives at a
     node with this IP. A common example is external load-balancers that are not
     part of the Kubernetes system.

   externalName	<string>
     externalName is the external reference that kubedns or equivalent will
     return as a CNAME record for this service. No proxying will be involved.
     Must be a valid RFC-1123 hostname (https://tools.ietf.org/html/rfc1123) and
     requires Type to be ExternalName.

   externalTrafficPolicy	<string>
     externalTrafficPolicy denotes if this Service desires to route external
     traffic to node-local or cluster-wide endpoints. "Local" preserves the
     client source IP and avoids a second hop for LoadBalancer and Nodeport type
     services, but risks potentially imbalanced traffic spreading. "Cluster"
     obscures the client source IP and may cause a second hop to another node,
     but should have good overall load-spreading.

   healthCheckNodePort	<integer>
     healthCheckNodePort specifies the healthcheck nodePort for the service. If
     not specified, HealthCheckNodePort is created by the service api backend
     with the allocated nodePort. Will use user-specified nodePort value if
     specified by the client. Only effects when Type is set to LoadBalancer and
     ExternalTrafficPolicy is set to Local.

   loadBalancerIP	<string>
     Only applies to Service Type: LoadBalancer LoadBalancer will get created
     with the IP specified in this field. This feature depends on whether the
     underlying cloud-provider supports specifying the loadBalancerIP when a
     load balancer is created. This field will be ignored if the cloud-provider
     does not support the feature.

   loadBalancerSourceRanges	<[]string>
     If specified and supported by the platform, this will restrict traffic
     through the cloud-provider load-balancer will be restricted to the
     specified client IPs. This field will be ignored if the cloud-provider does
     not support the feature." More info:
     https://kubernetes.io/docs/tasks/access-application-cluster/configure-cloud-provider-firewall/

   ports	<[]Object>
     The list of ports that are exposed by this service. More info:
     https://kubernetes.io/docs/concepts/services-networking/service/#virtual-ips-and-service-proxies

   publishNotReadyAddresses	<boolean>
     publishNotReadyAddresses, when set to true, indicates that DNS
     implementations must publish the notReadyAddresses of subsets for the
     Endpoints associated with the Service. The default value is false. The
     primary use case for setting this field is to use a StatefulSet's Headless
     Service to propagate SRV records for its Pods without respect to their
     readiness for purpose of peer discovery.

   selector	<map[string]string>
     Route service traffic to pods with label keys and values matching this
     selector. If empty or not present, the service is assumed to have an
     external process managing its endpoints, which Kubernetes will not modify.
     Only applies to types ClusterIP, NodePort, and LoadBalancer. Ignored if
     type is ExternalName. More info:
     https://kubernetes.io/docs/concepts/services-networking/service/

   sessionAffinity	<string>
     Supports "ClientIP" and "None". Used to maintain session affinity. Enable
     client IP based session affinity. Must be ClientIP or None. Defaults to
     None. More info:
     https://kubernetes.io/docs/concepts/services-networking/service/#virtual-ips-and-service-proxies

   sessionAffinityConfig	<Object>
     sessionAffinityConfig contains the configurations of session affinity.

   type	<string>
     type determines how the Service is exposed. Defaults to ClusterIP. Valid
     options are ExternalName, ClusterIP, NodePort, and LoadBalancer.
     "ExternalName" maps to the specified externalName. "ClusterIP" allocates a
     cluster-internal IP address for load-balancing to endpoints. Endpoints are
     determined by the selector or if that is not specified, by manual
     construction of an Endpoints object. If clusterIP is "None", no virtual IP
     is allocated and the endpoints are published as a set of endpoints rather
     than a stable IP. "NodePort" builds on ClusterIP and allocates a port on
     every node which routes to the clusterIP. "LoadBalancer" builds on NodePort
     and creates an external load-balancer (if supported in the current cloud)
     which routes to the clusterIP. More info:
     https://kubernetes.io/docs/concepts/services-networking/service/#publishing-services---service-types

```
{% endtab %}
{% endtabs %}

Add the --recursive flag to display all of the fields at once without descriptions.

## Delete

The _delete_ command delete resources by filenames, stdin, resources and names, or by resources and label selector.

Deleting a Service will have no effect on the Pod state. It only delete the access to each resources managed by the service.

Note that the delete command does NOT do resource version checks, so if someone submits an update to a resource right when you submit a delete, their update will be lost along with the rest of the resource.

#### Exercise n°1

Delete the service and the deployment created previously in the default namespace.

```bash
# Delete the service previously created
kubectl delete service nginx

# Delete the previous deployment created
kubectl delete deployment nginx
```

## Module exercise

The purpose of this section is to manage each steps of the lifecycle of an application to better understand each concepts of the Kubernetes course.

The main objective in this module is to understand how to expose a service depending on the security level needed.

For more information about the application used all along the course, please refer to the _Exercise App &gt; Voting App_ link in the left panel.

Based on the principles explain in this module, try by your own to handle this steps. The development of a yaml file is recommended.

The file developed has to be stored in this directory : `~/data/votingapp/04_services`

{% tabs %}
{% tab title="Exercise" %}
1. Manage the access of each part of the Voting App based on those conditions :
   1. The PostgreSQL database  Deployment must only be access from the Kubernetes cluster on port 5432.
   2. The Redis queue Deployment must only be access from the Kubernetes cluster on port 6379.
   3. The vote and result Deployment must be publicly access on port 8080.
   4. The worker Pods must not be exposed.
2. Try to access the vote web site
3. Try to access the result web site
{% endtab %}

{% tab title="Solution" %}
An example of yaml definition file to manage the Pods access with Services resources.

{% code-tabs %}
{% code-tabs-item title="~/data/votingapp/04\_services/services.yaml" %}
```yaml
apiVersion: v1
kind: Service
metadata:
  name: db
  namespace: voting-app
spec:
  ports:
  - name: db
    port: 5432
    protocol: TCP
    targetPort: 5432
  selector:
    name: db
  type: ClusterIP
---
apiVersion: v1
kind: Service
metadata:
  name: redis
  namespace: voting-app
spec:
  ports:
    - name: redis
      port: 6379
      protocol: TCP
      targetPort: 6379
  selector:
    name: redis
  type: ClusterIP
---
apiVersion: v1
kind: Service
metadata:
  name: result
  namespace: voting-app
spec:
  ports:
  - name: 8080-tcp
    nodePort: 31001
    port: 8080
    protocol: TCP
    targetPort: 8080
  selector:
    name: result
  type: NodePort
---
apiVersion: v1
kind: Service
metadata:
  name: vote
  namespace: voting-app
spec:
  ports:
  - name: 8080-tcp
    nodePort: 31000
    port: 8080
    targetPort: 8080
  selector:
    name: vote
  type: NodePort
```
{% endcode-tabs-item %}
{% endcode-tabs %}

Create the Services resources based on the previous yaml file definition.

```bash
kubectl apply  -f data/votingapp/04_services/services.yaml
```
{% endtab %}
{% endtabs %}

## External documentation

Those documentations can help you to go further in this topic :

* Kubernetes official documentation on [services](https://kubernetes.io/docs/concepts/services-networking/service/)

