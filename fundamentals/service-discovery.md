# Service Discovery

## Module overview

At the end of this module, you will :

* _Learn what is the Service Discovery_
* _Learn how to resolve a Pod and a Service by his DNS name_
* _Understand how to use it in your development_

{% hint style="info" %}
This module needs a DNS plugin to be deployed on the cluster. The default DNS plugin used in this module is CoreDNS. Ensure this module is up and running before continuing.
{% endhint %}

## Service discovery

Kubernetes DNS system is designed to manage the domain name of each Pods and Services in a Kubernetes cluster. [Kube-DNS](https://github.com/kubernetes/dns) and [CoreDNS](https://github.com/coredns/coredns) \(CoreDNS is a newer add-on that became a default DNS server as of Kubernetes v1.12\) are two established DNS solutions for defining DNS naming rules and resolving pod and service DNS to their corresponding cluster IPs. 

With DNS, Kubernetes services can be referenced by name that will correspond to any number of backend pods managed by the service. The naming scheme for DNS also follows a predictable pattern, making the addresses of various services more memorable. 

Services can also be referenced not only via a Fully Qualified Domain Name \(FQDN\) but also via only the name of the service itself.

Both add-ons schedule a DNS pod or pods and a service with a static IP on the cluster and both are named _kube-dns_.

Kubernetes DNS add-ons currently support forward lookups \(A records\), port lookups \(SRV records\), reverse IP address lookups \(PTR records\), and some other options.

### Service DNS records

Kubernetes Services support A records, CNAME, and SRV records.

#### A Record

A Record is the most basic type of a DNS record used to point a domain or subdomain to a certain IP address. The record consists of the domain name, the IP address to resolve it, and TTL in seconds. TTL stands for Time To Live, and is a sort of expiration date put on a DNS record. A TTL tells the DNS server how long it should keep a given record in its cache.

The default DNS A record assigned to a Service is formed based on this principles : `your-svc.your-namespace.svc.cluster.local`. This name resolves to a set of IPs of the pods selected by the Service. The DNS will not resolve this set to a specific IP automatically so the clients should take care of load balancing or round-robin selection from the set.

#### CNAME

CNAME records are used to point a domain or subdomain to another hostname. To achieve this, CNAMEs use the existing A record as their value. In Kubernetes, CNAME records can be used for cross-cluster service discovery with federated services.

#### SRV Records

SRV records facilitate service discovery by describing the protocol/s and address of certain services. 

An SRV record usually defines a symbolic name and the transport protocol \(e.g., TCP\) used as part of the domain name and defines the priority, weight, port, and target for a given service \(see the example below\).

```bash
_sip._tcp.example.com.   3600 IN    SRV 10       70     5060 srvrecord.example.com.
_sip._tcp.example.com.   3600 IN    SRV 10       20     5060 srvrecord2.example.com.
```

In Kubernetes, SRV Records are created for named ports. The SRV record takes the form of `_my-port-name._my-port-protocol.my-svc.my-namespace.svc.cluster.local`. For a regular service, this resolves to the port number and the domain name : `my-svc.my-namespace.svc.cluster.local`.

#### Exercise n°1

{% hint style="info" %}
On Minikube, CoreDNS is enable by default.
{% endhint %}

Create a Python HTTP server Deployment that listen on port 80.

{% code-tabs %}
{% code-tabs-item title="/data/servicediscovery/01\_deployment.yaml" %}
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: sd-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: webserver
  template:
    metadata:
      labels:
        app: webserver
    spec:
      containers:
      - image: nginx
        name: nginx
        ports:
          - name: http
            containerPort: 80
```
{% endcode-tabs-item %}
{% endcode-tabs %}

Create the Service to expose the previous Deployment internally on port 4000.

{% code-tabs %}
{% code-tabs-item title="/data/sevicediscovery/02\_service.yaml" %}
```yaml
kind: Service
apiVersion: v1
metadata:
  name: sd-service
spec:
  selector:
    app: webserver
  ports:
  - protocol: TCP
    port: 8080
    targetPort: http
```
{% endcode-tabs-item %}
{% endcode-tabs %}

Create the resources based on the previous yaml files definition.

```bash
kubectl create -f /data/servicediscovery/
```

Create a client Pod that will request the service by its name.

```bash
kubectl run sd-client --image=appropriate/curl --restart=Never -- /bin/sh -c "curl -s sd-service:8080"
```

Get the logs of the client Pod to get the DNS resolution information.

{% tabs %}
{% tab title="Command" %}
```bash
kubectl logs sd-client
```
{% endtab %}

{% tab title="CLI Return" %}
The log should return the default Nginx web page.

```bash
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

The response above indicates that Kube-DNS has correctly resolved the service name to the service’s ClusterIP and the service has successfully forwarded the client request to random backend pod picked in a round-robin fashion.
{% endtab %}
{% endtabs %}

### Pods DNS records

Kubernetes Pods support A records.

#### A Record

The default DNS A record assigned to a Pods is `pod-ip-address.my-namespace.pod.cluster.local`. 

For example, a pod with IP 172.12.3.4 in the namespace default with a DNS name of cluster.local would have an entry of the form `172-12-3-4.default.pod.cluster.local`.

Hostname and subdomain of a Pod can be overwritten with optional parameters defined in the spec field of the object declaration.

For example, a pod with its hostname set to `custom-host` , and subdomain set to `custom-subdomain`, in namespace `my-namespace`, will have the fully qualified domain name \(FQDN\) `custom-host.custom-subdomain.my-namespace.svc.cluster.local`.

## External documentation

Those documentations can help you to go further in this topic :

* Kubernetes official documentation on [DNS Pods management](https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/)
* Kubernetes official documentation on [cross-cluster DNS service discovery](https://kubernetes.io/docs/tasks/federation/federation-service-discovery/)

