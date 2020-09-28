---
description: >-
  Pods are the smallest deployable units of computing that can be created and
  managed in Kubernetes.
---

# Pods

## Module

_Pods_ are the smallest deployable units of computing that can be created and managed in Kubernetes.

#### Overview

At the end of this module, you will :

* _Learn the format of a YAML file_
* _Learn how to manage a Pods_
* _Learn how to manage containers_

#### Prerequisites

Create the directory `data/pods` in your home folder to manage the YAML file needed in this module.

```bash
mkdir ~/data/pods
```

## Run

The _run_ command creates a Pod based on the parameters specified, such as the image or the environment variables. This deployment is issued to the Kubernetes master which launches the Pods and containers required. Kubectl run is similar to _docker run_ but at a cluster level.

#### Exercise n°1

Run a busybox pod on the default namespace.

```bash
kubectl run busybox --image=busybox -n default --restart=Never
```

## Create

In imperative object configuration, the kubectl command specifies the operation \(create, replace, etc.\), optional flags and at least one file name. The file specified must contain a full definition of the object in YAML or JSON format.

The _create_ command create a resources based on the imperative declaration \(command line or YAML file\).

#### Exercise n°1

Create a pod that contain a single nginx container with a simple YAML file.

{% hint style="info" %}
The file created in this exercise will be reused, do not delete it.
{% endhint %}

{% code-tabs %}
{% code-tabs-item title="~/data/pods/01\_pods.yaml" %}
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-single-app
  labels:
    env: formation
spec:
  containers:
  - name: nginx
    image: nginx
```
{% endcode-tabs-item %}
{% endcode-tabs %}

```bash
kubectl apply  -f data/pods/01_pods.yaml
```

#### Exercise n°2

Create a pod that contain multiple containers : nginx, redis, postgres with a single YAML file.

{% code-tabs %}
{% code-tabs-item title="~/data/pods/02\_pods.yaml" %}
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-multi-app
  labels:
    env: formation
spec:
  containers:
  - name: nginx
    image: nginx:1.14
  - name: postgres
    image: postgres:10.4
  - name: redis
    image: redis
```
{% endcode-tabs-item %}
{% endcode-tabs %}

```bash
kubectl apply  -f data/pods/02_pods.yaml
```

## Apply

When using declarative object configuration, a user operates on object configuration files stored locally, however the user does not define the operations to be taken on the files. Create, update, and delete operations are automatically detected per-object by kubectl. This enables working on directories, where different operations might be needed for different objects.

The _apply_ command manage the status of resources in the Kubernetes cluster. It can create, update and delete resources based on one or more YAML files.

#### Exercise n°1

Update the image version of the previous nginx pods.

{% code-tabs %}
{% code-tabs-item title="~/data/pods/03\_pods.yaml" %}
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-single-nginx14
  labels:
    env: formation
spec:
  containers:
  - name: nginx
    image: nginx:1.14
```
{% endcode-tabs-item %}
{% endcode-tabs %}

```bash
kubectl apply -f ~/data/pods/03_pods.yaml
```

## Get

The _get_ command list the object asked. It could be a single object or a list of multiple objects comma separated. This command is useful to get the status of each object. The output can be formatted to only display some information based on some json search or external tools like `tr`, `sort`, `uniq`.

The default output display some useful information about each services :

* Name : the name of the newly created resource
* Ready : the amount of container ready in the Pods
* Status : the deployment status
* Restarts : the count that the Pods has restarted from his creation
* Age : the age since the Pods creation

#### Exercise n°1

List pods deployed on the current namespace.

{% tabs %}
{% tab title="Command" %}
```bash
kubectl get pods
```
{% endtab %}

{% tab title="CLI Return" %}
```bash
NAME                READY     STATUS      RESTARTS   AGE
busybox             0/1       Completed   0          4m
my-multi-app        3/3       Running     0          3m
my-single-nginx     1/1       Running     0          3m
my-single-nginx14   1/1       Running     0          3m
```
{% endtab %}
{% endtabs %}

#### Exercise n°2

List pods deployed on the namespace _default_.

{% tabs %}
{% tab title="Command" %}
```bash
kubectl get pods -n default
```
{% endtab %}

{% tab title="CLI Return" %}
```bash
NAME                READY     STATUS      RESTARTS   AGE
busybox             0/1       Completed   0          9m
my-multi-app        3/3       Running     0          8m
my-single-nginx     1/1       Running     0          8m
my-single-nginx14   1/1       Running     1          8m
```
{% endtab %}
{% endtabs %}

#### Exercise n°3

List all pods in all namespaces.

{% tabs %}
{% tab title="Command" %}
```bash
kubectl get pods --all-namespaces
```
{% endtab %}

{% tab title="CLI Return" %}
```bash
NAMESPACE     NAME                                       READY     STATUS      RESTARTS   AGE
default       busybox                                    0/1       Completed   0          10m
default       my-multi-app                               3/3       Running     0          9m
default       my-single-nginx                            1/1       Running     0          9m
default       my-single-nginx14                          1/1       Running     1          8m
kube-system   coredns-86c58d9df4-l2hlv                   1/1       Running     0          59m
kube-system   coredns-86c58d9df4-vwf67                   1/1       Running     0          59m
kube-system   default-http-backend-5ff9d456ff-r4fk8      1/1       Running     0          59m
kube-system   etcd-minikube                              1/1       Running     0          58m
kube-system   kube-addon-manager-minikube                1/1       Running     0          58m
kube-system   kube-apiserver-minikube                    1/1       Running     0          59m
kube-system   kube-controller-manager-minikube           1/1       Running     0          58m
kube-system   kube-proxy-s5frv                           1/1       Running     0          59m
kube-system   kube-scheduler-minikube                    1/1       Running     0          58m
kube-system   metrics-server-6fc4b7bcff-wsjsq            1/1       Running     0          59m
kube-system   nginx-ingress-controller-7c66d668b-sc6g8   1/1       Running     0          59m
kube-system   storage-provisioner                        1/1       Running     0          59m

```
{% endtab %}
{% endtabs %}

#### Exercise n°4

List all pods, services, deployments in the _kube-system_ namespace.

{% tabs %}
{% tab title="Command" %}
```bash
kubectl get pods,service,deployment -n kube-system
```
{% endtab %}

{% tab title="CLI Return" %}
```bash
NAME                                       READY     STATUS    RESTARTS   AGE
coredns-86c58d9df4-l2hlv                   1/1       Running   0          1h
coredns-86c58d9df4-vwf67                   1/1       Running   0          1h
default-http-backend-5ff9d456ff-r4fk8      1/1       Running   0          1h
etcd-minikube                              1/1       Running   0          1h
kube-addon-manager-minikube                1/1       Running   0          1h
kube-apiserver-minikube                    1/1       Running   0          1h
kube-controller-manager-minikube           1/1       Running   2          1h
kube-proxy-s5frv                           1/1       Running   0          1h
kube-scheduler-minikube                    1/1       Running   2          1h
metrics-server-6fc4b7bcff-wsjsq            1/1       Running   0          1h
nginx-ingress-controller-7c66d668b-sc6g8   1/1       Running   0          1h
storage-provisioner                        1/1       Running   0          1h

NAME                   TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)         AGE
default-http-backend   NodePort    10.99.155.8     <none>        80:30001/TCP    1h
kube-dns               ClusterIP   10.96.0.10      <none>        53/UDP,53/TCP   1h
metrics-server         ClusterIP   10.98.243.211   <none>        443/TCP         1h

NAME                       DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
coredns                    2         2         2            2           1h
default-http-backend       1         1         1            1           1h
metrics-server             1         1         1            1           1h
nginx-ingress-controller   1         1         1            1           1h

```
{% endtab %}
{% endtabs %}

## Logs

Once an application is running, it is inevitably a need to debug problems or check the configuration deployed.

The _logs_ command display the stdout and stderr logs of each container in a pod. The containers within a Pod must be started to get information otherwise nothing will be displayed.

This command is really useful to debug a container deployed in a Pod.

#### Exercise n°1

Create a new Pods that send a message in stdout.

{% code-tabs %}
{% code-tabs-item title="~/data/pods/04\_pods.yaml" %}
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: busybox-logs
spec:
  containers:
  - name: busybox
    image: busybox
    command: ["/bin/sh"]
    args: ["-c", "echo \"$(date) - INFO - My first logs output on $(hostname)\""]
  restartPolicy: Never
```
{% endcode-tabs-item %}
{% endcode-tabs %}

Create the resource based on the previous yaml file definition.

```bash
kubectl apply  -f data/pods/04_pods.yaml
```

Get the logs of the busybox-logs Pod created previously.

{% tabs %}
{% tab title="Command" %}
```bash
kubectl logs busybox-logs
```
{% endtab %}

{% tab title="CLI Return" %}
```bash
Thu Feb 28 22:19:56 UTC 2019 - INFO - My first logs output on busybox-logs
```
{% endtab %}
{% endtabs %}

## Describe

Once an application is running, it is inevitably a need to debug problems or check the configuration deployed.

The _describe_ command display a lot of configuration information about the container\(s\) and Pod \(labels, resource requirements, etc.\) or any other Kubernetes objects, as well as status information about the container\(s\) and Pod \(state, readiness, restart count, events, etc.\).

This command is really useful to introspect and debug a container deployed in a Pod.

#### Exercise n°1

Get the information of the pod busybox deployed on the default namespace.

{% tabs %}
{% tab title="Command" %}
```bash
kubectl describe po busybox
```
{% endtab %}

{% tab title="CLI Return" %}
```bash
Name:               busybox
Namespace:          default
Priority:           0
PriorityClassName:  <none>
Node:               minikube/10.0.2.15
Start Time:         Wed, 13 Feb 2019 12:45:52 -0500
Labels:             run=busybox
Annotations:        <none>
Status:             Succeeded
IP:                 172.17.0.7
Containers:
  busybox:
    Container ID:   docker://5a3892c9be87e3f15e752f99d127d6d84525d1a29f3a5bb1b11713a3ae373eb1
    Image:          busybox
    Image ID:       docker-pullable://busybox@sha256:7964ad52e396a6e045c39b5a44438424ac52e12e4d5a25d94895f2058cb863a0
    Port:           <none>
    Host Port:      <none>
    State:          Terminated
      Reason:       Completed
      Exit Code:    0
      Started:      Wed, 13 Feb 2019 12:45:55 -0500
      Finished:     Wed, 13 Feb 2019 12:45:55 -0500
    Ready:          False
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-wpxhp (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             False 
  ContainersReady   False 
  PodScheduled      True 
Volumes:
  default-token-wpxhp:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-wpxhp
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
Events:
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  18m   default-scheduler  Successfully assigned default/busybox to minikube
  Normal  Pulling    18m   kubelet, minikube  pulling image "busybox"
  Normal  Pulled     18m   kubelet, minikube  Successfully pulled image "busybox"
  Normal  Created    18m   kubelet, minikube  Created container
  Normal  Started    18m   kubelet, minikube  Started container

```
{% endtab %}
{% endtabs %}

## Exec

Execute a command in a container is sometimes needed for debug or development purpose.

The _exec_ command manage the SSH connection to a container deployed in a Pod.

Usually, the format is : _kubectl exec -it POD\_NAME -c CONTAINER\_NAME_

#### Exercise n°1

Connect to the pod _my-single-app_ in the namespace _default_.

```bash
kubectl exec -it my-single-app bash
```

Exit the container with this command.

```bash
exit
```

#### Exercise n°2

Connect to the postgres container in the _my-multi-app_ pod in the namespace default_._

{% tabs %}
{% tab title="Command" %}
```bash
kubectl exec -it my-multi-app -c postgres env
```
{% endtab %}

{% tab title="CLI Return" %}
```bash
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/lib/postgresql/11/bin
HOSTNAME=my-multi-app
TERM=xterm
KUBERNETES_PORT_443_TCP=tcp://10.96.0.1:443
KUBERNETES_PORT_443_TCP_PROTO=tcp
KUBERNETES_PORT_443_TCP_PORT=443
KUBERNETES_PORT_443_TCP_ADDR=10.96.0.1
KUBERNETES_SERVICE_HOST=10.96.0.1
KUBERNETES_SERVICE_PORT=443
KUBERNETES_SERVICE_PORT_HTTPS=443
KUBERNETES_PORT=tcp://10.96.0.1:443
GOSU_VERSION=1.11
LANG=en_US.utf8
PG_MAJOR=11
PG_VERSION=11.1-3.pgdg90+1
PGDATA=/var/lib/postgresql/data
HOME=/root

```
{% endtab %}
{% endtabs %}

## Edit

Sometimes it’s necessary to make narrow, non-disruptive updates to resources created. It is suggested to maintain a set of configuration files in source control, so that they can be maintained and versioned along with the code for the resources they configure. But sometimes for debug and development purpose, it could be useful to directly manage a Kubernetes object deployed in a cluster in real time.

The _edit_ command allows to directly edit any API resource retrieve via the command line tools. It will open the editor defined by your KUBE \_EDITOR, or EDITOR environment variables, or fall back to 'vi' for Linux or 'notepad' for Windows. Multiple objects can edit, although changes are applied one at a time.

#### Exercise n°1

Edit the my-single-app pods to update the image.

```bash
KUBE_EDITOR="nano" kubectl edit pods my-single-app
```

## Explain

Kubernetes come with a lot of documentation about his objects and the available options in each one. Those information can be find easily in command line or in the official Kubernetes documentation.

The _explain_ command allows to directly ask the API resource via the command line tools to display information about each Kubernetes objects and their architecture.

#### Exercise n°1

Get the documentation of a specific field of a resource.

{% tabs %}
{% tab title="Command" %}
```bash
kubectl explain pods.spec
```
{% endtab %}

{% tab title="CLI Return" %}
```bash
KIND:     Pod
VERSION:  v1

RESOURCE: spec <Object>

DESCRIPTION:
     Specification of the desired behavior of the pod. More info:
     https://git.k8s.io/community/contributors/devel/api-conventions.md#spec-and-status

     PodSpec is a description of a pod.

FIELDS:
   activeDeadlineSeconds	<integer>
     Optional duration in seconds the pod may be active on the node relative to
     StartTime before the system will actively try to mark it failed and kill
     associated containers. Value must be a positive integer.

   affinity	<Object>
     If specified, the pod's scheduling constraints

   automountServiceAccountToken	<boolean>
     AutomountServiceAccountToken indicates whether a service account token
     should be automatically mounted.

   containers	<[]Object> -required-
     List of containers belonging to the pod. Containers cannot currently be
     added or removed. There must be at least one container in a Pod. Cannot be
     updated.

   dnsConfig	<Object>
     Specifies the DNS parameters of a pod. Parameters specified here will be
     merged to the generated DNS configuration based on DNSPolicy.

   dnsPolicy	<string>
     Set DNS policy for the pod. Defaults to "ClusterFirst". Valid values are
     'ClusterFirstWithHostNet', 'ClusterFirst', 'Default' or 'None'. DNS
     parameters given in DNSConfig will be merged with the policy selected with
     DNSPolicy. To have DNS options set along with hostNetwork, you have to
     specify DNS policy explicitly to 'ClusterFirstWithHostNet'.

   enableServiceLinks	<boolean>
     EnableServiceLinks indicates whether information about services should be
     injected into pod's environment variables, matching the syntax of Docker
     links.

   hostAliases	<[]Object>
     HostAliases is an optional list of hosts and IPs that will be injected into
     the pod's hosts file if specified. This is only valid for non-hostNetwork
     pods.

   hostIPC	<boolean>
     Use the host's ipc namespace. Optional: Default to false.

   hostNetwork	<boolean>
     Host networking requested for this pod. Use the host's network namespace.
     If this option is set, the ports that will be used must be specified.
     Default to false.

   hostPID	<boolean>
     Use the host's pid namespace. Optional: Default to false.

   hostname	<string>
     Specifies the hostname of the Pod If not specified, the pod's hostname will
     be set to a system-defined value.

   imagePullSecrets	<[]Object>
     ImagePullSecrets is an optional list of references to secrets in the same
     namespace to use for pulling any of the images used by this PodSpec. If
     specified, these secrets will be passed to individual puller
     implementations for them to use. For example, in the case of docker, only
     DockerConfig type secrets are honored. More info:
     https://kubernetes.io/docs/concepts/containers/images#specifying-imagepullsecrets-on-a-pod

   initContainers	<[]Object>
     List of initialization containers belonging to the pod. Init containers are
     executed in order prior to containers being started. If any init container
     fails, the pod is considered to have failed and is handled according to its
     restartPolicy. The name for an init container or normal container must be
     unique among all containers. Init containers may not have Lifecycle
     actions, Readiness probes, or Liveness probes. The resourceRequirements of
     an init container are taken into account during scheduling by finding the
     highest request/limit for each resource type, and then using the max of of
     that value or the sum of the normal containers. Limits are applied to init
     containers in a similar fashion. Init containers cannot currently be added
     or removed. Cannot be updated. More info:
     https://kubernetes.io/docs/concepts/workloads/pods/init-containers/

   nodeName	<string>
     NodeName is a request to schedule this pod onto a specific node. If it is
     non-empty, the scheduler simply schedules this pod onto that node, assuming
     that it fits resource requirements.

   nodeSelector	<map[string]string>
     NodeSelector is a selector which must be true for the pod to fit on a node.
     Selector which must match a node's labels for the pod to be scheduled on
     that node. More info:
     https://kubernetes.io/docs/concepts/configuration/assign-pod-node/

   priority	<integer>
     The priority value. Various system components use this field to find the
     priority of the pod. When Priority Admission Controller is enabled, it
     prevents users from setting this field. The admission controller populates
     this field from PriorityClassName. The higher the value, the higher the
     priority.

   priorityClassName	<string>
     If specified, indicates the pod's priority. "system-node-critical" and
     "system-cluster-critical" are two special keywords which indicate the
     highest priorities with the former being the highest priority. Any other
     name must be defined by creating a PriorityClass object with that name. If
     not specified, the pod priority will be default or zero if there is no
     default.

   readinessGates	<[]Object>
     If specified, all readiness gates will be evaluated for pod readiness. A
     pod is ready when all its containers are ready AND all conditions specified
     in the readiness gates have status equal to "True" More info:
     https://github.com/kubernetes/community/blob/master/keps/sig-network/0007-pod-ready%!B(MISSING)%!B(MISSING).md

   restartPolicy	<string>
     Restart policy for all containers within the pod. One of Always, OnFailure,
     Never. Default to Always. More info:
     https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/#restart-policy

   runtimeClassName	<string>
     RuntimeClassName refers to a RuntimeClass object in the node.k8s.io group,
     which should be used to run this pod. If no RuntimeClass resource matches
     the named class, the pod will not be run. If unset or empty, the "legacy"
     RuntimeClass will be used, which is an implicit class with an empty
     definition that uses the default runtime handler. More info:
     https://github.com/kubernetes/community/blob/master/keps/sig-node/0014-runtime-class.md
     This is an alpha feature and may change in the future.

   schedulerName	<string>
     If specified, the pod will be dispatched by specified scheduler. If not
     specified, the pod will be dispatched by default scheduler.

   securityContext	<Object>
     SecurityContext holds pod-level security attributes and common container
     settings. Optional: Defaults to empty. See type description for default
     values of each field.

   serviceAccount	<string>
     DeprecatedServiceAccount is a depreciated alias for ServiceAccountName.
     Deprecated: Use serviceAccountName instead.

   serviceAccountName	<string>
     ServiceAccountName is the name of the ServiceAccount to use to run this
     pod. More info:
     https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/

   shareProcessNamespace	<boolean>
     Share a single process namespace between all of the containers in a pod.
     When this is set containers will be able to view and signal processes from
     other containers in the same pod, and the first process in each container
     will not be assigned PID 1. HostPID and ShareProcessNamespace cannot both
     be set. Optional: Default to false. This field is beta-level and may be
     disabled with the PodShareProcessNamespace feature.

   subdomain	<string>
     If specified, the fully qualified Pod hostname will be
     "<hostname>.<subdomain>.<pod namespace>.svc.<cluster domain>". If not
     specified, the pod will not have a domainname at all.

   terminationGracePeriodSeconds	<integer>
     Optional duration in seconds the pod needs to terminate gracefully. May be
     decreased in delete request. Value must be non-negative integer. The value
     zero indicates delete immediately. If this value is nil, the default grace
     period will be used instead. The grace period is the duration in seconds
     after the processes running in the pod are sent a termination signal and
     the time when the processes are forcibly halted with a kill signal. Set
     this value longer than the expected cleanup time for your process. Defaults
     to 30 seconds.

   tolerations	<[]Object>
     If specified, the pod's tolerations.

   volumes	<[]Object>
     List of volumes that can be mounted by containers belonging to the pod.
     More info: https://kubernetes.io/docs/concepts/storage/volumes

```
{% endtab %}
{% endtabs %}

Add the --recursive flag to display all of the fields at once without descriptions.

## Delete

The _delete_ command delete resources by filenames, stdin, resources and names, or by resources and label selector.

Some resources, such as pods, support graceful deletion. These resources define a default period before they are forcibly terminated \(the grace period\) but you may override that value with the --grace-period flag, or pass --now to set a grace-period of 1.

Note that the delete command does NOT do resource version checks, so if someone submits an update to a resource right when you submit a delete, their update will be lost along with the rest of the resource.

#### Exercise n°1

Delete the busybox pod deployed previously in the default namespace.

```bash
kubectl delete pods busybox -n default
```

#### Exercise n°2

Delete the nginx pod deployed previously in the default namespace with declarative method.

```bash
kubectl delete -f ~/data/pods
```

## Module exercise

The purpose of this section is to manage each steps of the lifecycle of an application to better understand each concepts of the Kubernetes course.

The main objective in this module is to deploy the Voting App Pods in his dedicated namespace.

For more information about the application used all along the course, please refer to the _Exercise App &gt; Voting App_ link in the left panel.

Based on the principles explain in this module, try by your own to handle this steps. The development of a yaml file is recommended.

The file developed has to be stored in this directory : `~/data/votingapp/01_pods`

{% tabs %}
{% tab title="Exercise" %}
1. Deploy each containers of the Voting App in a single Pod called `voting-app` in his dedicated namespace created in the previous module.
2. Ensure the Pods are up and running
3. Check the logs of each Pods
{% endtab %}

{% tab title="Solution" %}
Yaml file definition to deploy the Voting App containers.

{% code-tabs %}
{% code-tabs-item title="~/data/votingapp/01\_pods/pods.yaml" %}
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: db
  namespace: voting-app
spec:
  containers:
  - name: db
    image: postgres:10.4
---
apiVersion: v1
kind: Pod
metadata:
  name: redis
  namespace: voting-app
spec:
  containers:
  - name: redis
    image: redis
---
apiVersion: v1
kind: Pod
metadata:
  name: result
  namespace: voting-app
spec:
  containers:
  - name: result-app
    image: wikitops/examplevotingapp-result:1.1
---
apiVersion: v1
kind: Pod
metadata:
  name: vote
  namespace: voting-app
spec:
  containers:
  - name: voting-app
    image: wikitops/examplevotingapp-vote:1.1
---
apiVersion: v1
kind: Pod
metadata:
  name: worker
  namespace: voting-app
spec:
  containers:
  - name: worker
    image: wikitops/examplevotingapp-worker:1.1
```
{% endcode-tabs-item %}
{% endcode-tabs %}

Kubectl command to deploy the Pods based on the previous definition file.

```bash
kubectl apply  -f data/votingapp/01_pods/pods.yaml
```

Ensure the Pods are created.

```bash
kubectl get pods -n voting-app
```

Check the logs of each Pods to ensure the main process is running.

```bash
# Get the database logs
kubectl logs db -n voting-app

# Get the queue logs
kubectl logs redis -n voting-app

# Get the result logs
kubectl logs result -n voting-app

# Get the vote logs
kubectl logs vote -n voting-app

# Get the worker logs
kubectl logs worker -n voting-app
```
{% endtab %}
{% endtabs %}

## External documentation

Those documentations can help you to go further in this topic :

* Kubernetes official [cheat sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/) documentation
* [Kubectl official Reference](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#-strong-getting-started-strong-) documentation
* Kubernetes official documentation [pod overview](https://kubernetes.io/docs/concepts/workloads/pods/pod-overview/)
* Kubernetes official documentation on [object management](https://kubernetes.io/docs/concepts/overview/object-management-kubectl/overview/)
* Kubernetes official documentation on [list pods](https://kubernetes.io/docs/tasks/access-application-cluster/list-all-running-container-images/)
* Kubernetes official documentation on [introspection and debugging](https://kubernetes.io/docs/tasks/debug-application-cluster/debug-application-introspection/#using-kubectl-describe-pod-to-fetch-details-about-pods)
* Kubernetes official documentation on [getting a shell to a running container](https://kubernetes.io/docs/tasks/debug-application-cluster/get-shell-running-container/)
* Kubernetes official documentation on [resources management](https://kubernetes.io/docs/concepts/cluster-administration/manage-deployment/)

