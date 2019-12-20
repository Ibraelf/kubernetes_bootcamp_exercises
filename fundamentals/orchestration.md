---
description: >-
  Orchestration is the automated arrangement, coordination, and management of
  computer systems, middleware, and services.
---

# Orchestration

## Module

_Orchestration_ is the automated arrangement, coordination, and management of computer systems, middleware, and services.

#### Overview

At the end of this module, you will :

* _Learn what a Kubernetes cluster is_
* _Learn how to manage it in command line_
* _Learn how to manage basic resources on a Kubernetes cluster_

#### Prerequisites

Create these directories `data/votingapp` and `data/orchestration` in your home folder to manage the YAML file needed in this module.

```bash
mkdir -p ~/data/votingapp ~/data/orchestration
```

## Command Line

The Kubernetes command-line tool, **kubectl**, is used to deploy and manage applications on Kubernetes. Using kubectl, you can inspect cluster resources, create, delete, and update components, look at your new cluster and bring up apps.

You must use a kubectl version that is within one minor version difference of your cluster. For example, a v1.12 client should work with v1.11, v1.12, and v1.13 master. Using the latest version of kubectl helps avoid unforeseen issues.

### Installation

There are a few methods to install kubectl, here are the basics depending on the operating system :

{% tabs %}
{% tab title="Debian / Ubuntu" %}

```bash
sudo apt-get update && sudo apt-get install -y apt-transport-https
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
echo "deb https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee -a /etc/apt/sources.list.d/kubernetes.list
sudo apt-get update
sudo apt-get install -y kubectl
```

{% endtab %}

{% tab title="CentOS / RedHat" %}

```text
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
EOF
yum install -y kubectl
```

{% endtab %}

{% tab title="MacOS" %}

#### From Brew

```bash
brew install kubernetes-cli
```

#### From binary

```text
# Download the binary
curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/darwin/amd64/kubectl

# Manage the execution right to the binary
chmod +x ./kubectl

# Move the binary to the PATH 
sudo mv ./kubectl /usr/local/bin/kubectl
```

{% endtab %}

{% tab title="Windows" %}

#### From Powershell

Execute the installation commands (specify the DownloadLocation)

```bash
Install-Script -Name install-kubectl -Scope CurrentUser -Force
install-kubectl.ps1 [-DownloadLocation <path>]
```

The installation program creates $HOME/.kube which is followed by the creation of a configuration file.
Replace the configuration file with your own.

#### From Chocolatey

```bash
choco install kubernetes-cli
```

#### From binary

```bash
curl -LO https://storage.googleapis.com/kubernetes-release/release/v1.17.0/bin/windows/amd64/kubectl.exe
```

Add the EXE binary to the PATH.

Access your personal directory:

```bash
cd %USERPROFILE%
```

Create the .kube directory:

```bash
mkdir .kube
```

Go to the .kube directory you just created:

```bash
cd .kube
```

Copy the kubeconfig file into this directory and name it config.

Test to make sure the version you have installed is up to date:

```bash
kubectl version
```

{% endtab %}
{% endtabs %}

For further information about Kubectl installation method, please refer to the [Kubernetes documentation](https://kubernetes.io/docs/tasks/tools/install-kubectl/).

### Configuration

In order for kubectl to find and access a Kubernetes cluster, it needs a kubeconfig file, which is created automatically when you create a cluster or successfully deploy a Minikube cluster. By default, kubectl configuration is located at `~/.kube/config`.

### Usage

Generally the command line format can be divide in three parts :

1. The kubectl command line binary
2. The action
3. The object  to manage

This can be represented like this :

```bash
kubectl <ACTION> <OBJECT>
```

#### Operations

Here is an exhaustive [list of actions](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands) that can be done :

| Operation | Description |
| :--- | :--- |
| annotate | Update the annotations on a resource |
| api-resources | Print the supported API versions on the server, in the form of "group/version" |
| apply | Apply a configuration to a resource by filename or stdin |
| attach | Attach to a running container |
| auth | Inspect authorization |
| autoscale | Auto-scale a Deployment, ReplicaSet, or ReplicationController |
| certificate | Modify certificate resources |
| cluster-info | Display cluster info |
| config | Modify kubeconfig files |
| convert | Convert config files between different API versions |
| cordon | Mark node as unschedulable |
| cp | Copy files and directories to and from containers |
| create | Create a resource from a file or from stdin |
| delete | Delete resources by filenames, stdin, resources and names, or by resources and label selector |
| describe | Show details of a specific resource or group of resources |
| drain | Drain node in preparation for maintenance |
| edit | Edit a resource on the server |
| exec | Execute a command in a container |
| explain | Documentation of resources |
| expose | Take a replication controller, service, deployment or pod and expose it as a new Kubernetes Service |
| get | Display one or many resources |
| label | Update the labels on a resource |
| logs | Print the logs for a container in a pod |
| patch | Update field\(s\) of a resource using strategic merge patch |
| plugin | Runs a command-line plugin |
| port-forward | Forward one or more local ports to a pod |
| proxy | Run a proxy to the Kubernetes API server |
| replace | Replace a resource by filename or stdin |
| rolling-update | Perform a rolling update of the given ReplicationController \(Deprecated\) |
| rollout | Manage the rollout of a resource |
| run | Run a particular image on the cluster |
| scale | Set a new size for a Deployment, ReplicaSet, Replication Controller, or Job |
| set | Set specific features on objects |
| taint | Update the taints on one or more nodes |
| top | Display Resource \(CPU/Memory/Storage\) usage |
| uncordon | Mark node as schedulable |
| version | Print the client and server version information |

#### Resource types

Here is an exhaustive list of [Kubernetes objects](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.16/) that can be managed :

| Resource type | Abbreviated alias |
| :--- | :--- |
| all | all |
| certificatesigningrequests | csr |
| clusterrolebindings | clusterrolebindings |
| clusterroles | clusterroles |
| componentstatuses | cs |
| configmaps | cm |
| controllerrevisions | controllerrevisions |
| cronjobs | cronjobs |
| customresourcedefinition | crd |
| daemonsets | ds |
| deployments | deploy |
| endpoints | ep |
| events | ev |
| horizontalpodautoscalers | hpa |
| ingresses | ing |
| jobs | jobs |
| limitranges | limits |
| namespaces | ns |
| networkpolicies | netpol |
| nodes | no |
| persistentvolumeclaims | pvc |
| persistentvolumes | pv |
| poddisruptionbudgets | pdb |
| podpreset | podpreset |
| pods | po |
| podsecuritypolicies | psp |
| podtemplates | podtemplates |
| replicasets | rs |
| replicationcontrollers | rc |
| resourcequotas | quota |
| rolebindings | rolebindings |
| roles | roles |
| secrets | secrets |
| serviceaccounts | sa |
| services | svc |
| statefulsets | sts |
| storageclasses | sc |

#### Exercise n°1

Get the cluster information in command line.

{% tabs %}
{% tab title="Command" %}
```bash
kubectl cluster-info
```
{% endtab %}

{% tab title="CLI Return" %}
```bash
Kubernetes master is running at https://192.168.99.100:8443
KubeDNS is running at https://192.168.99.100:8443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy
```
{% endtab %}
{% endtabs %}

#### Exercise n°2

Get the config deployed in the Kubernetes cluster.

{% tabs %}
{% tab title="Command" %}
```bash
kubectl config view
```
{% endtab %}

{% tab title="CLI Return" %}
```bash
apiVersion: v1
clusters:
- cluster:
    insecure-skip-tls-verify: true
    server: https://192.168.99.100:8443
  name: 192-168-99-100:8443
- cluster:
    certificate-authority: /home/treeptik/.minikube/ca.crt
    server: https://192.168.99.100:8443
  name: minikube
contexts:
- context:
    cluster: 192-168-99-100:8443
    user: admin/192-168-99-100:8443
  name: /192-168-99-100:8443/admin
- context:
    cluster: minikube
    user: minikube
  name: minikube
- context:
    cluster: 192-168-99-100:8443
    namespace: myproject
    user: developer/192-168-99-100:8443
  name: minishift
- context:
    cluster: 192-168-99-100:8443
    namespace: myproject
    user: developer/192-168-99-100:8443
  name: myproject/192-168-99-100:8443/developer
- context:
    cluster: 192-168-99-100:8443
    namespace: myproject
    user: system:admin/192-168-99-100:8443
  name: myproject/192-168-99-100:8443/system:admin
current-context: minikube
kind: Config
preferences: {}
users:
- name: admin/192-168-99-100:8443
  user:
    token: rAacwcx21t096cPZF6GY99P0VuM3Wi5Ju265_OaMN9Y
- name: developer/192-168-99-100:8443
  user:
    token: oWmdcNcXJmaMrlegTYnIoAfFRWlm0lFw4pldn8jxfrQ
- name: minikube
  user:
    client-certificate: /home/treeptik/.minikube/client.crt
    client-key: /home/treeptik/.minikube/client.key
- name: system:admin/192-168-99-100:8443
  user:
    client-certificate-data: REDACTED
    client-key-data: REDACTED

```
{% endtab %}
{% endtabs %}

#### Exercise n°3

Get each elements deployed in the cluster in command line.

{% tabs %}
{% tab title="Command" %}
```bash
kubectl get all --all-namespaces
kubectl get all -A
```
{% endtab %}

{% tab title="CLI Return" %}
```bash
# Pods
NAMESPACE     NAME                                       READY     STATUS    RESTARTS   AGE
kube-system   coredns-86c58d9df4-l2hlv                   1/1       Running   0          2m
kube-system   coredns-86c58d9df4-vwf67                   1/1       Running   0          2m
kube-system   default-http-backend-5ff9d456ff-r4fk8      1/1       Running   0          2m
kube-system   etcd-minikube                              1/1       Running   0          1m
kube-system   kube-addon-manager-minikube                1/1       Running   0          1m
kube-system   kube-apiserver-minikube                    1/1       Running   0          1m
kube-system   kube-controller-manager-minikube           1/1       Running   0          1m
kube-system   kube-proxy-s5frv                           1/1       Running   0          2m
kube-system   kube-scheduler-minikube                    1/1       Running   0          1m
kube-system   metrics-server-6fc4b7bcff-wsjsq            1/1       Running   0          2m
kube-system   nginx-ingress-controller-7c66d668b-sc6g8   1/1       Running   0          2m
kube-system   storage-provisioner                        1/1       Running   0          2m

# Services
NAMESPACE     NAME                   TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)         AGE
default       kubernetes             ClusterIP   10.96.0.1       <none>        443/TCP         2m
kube-system   default-http-backend   NodePort    10.99.155.8     <none>        80:30001/TCP    2m
kube-system   kube-dns               ClusterIP   10.96.0.10      <none>        53/UDP,53/TCP   2m
kube-system   metrics-server         ClusterIP   10.98.243.211   <none>        443/TCP         2m

# DaemonSets
NAMESPACE     NAME         DESIRED   CURRENT   READY     UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
kube-system   kube-proxy   1         1         1         1            1           <none>          2m

# Deployments
NAMESPACE     NAME                       DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
kube-system   coredns                    2         2         2            2           2m
kube-system   default-http-backend       1         1         1            1           2m
kube-system   metrics-server             1         1         1            1           2m
kube-system   nginx-ingress-controller   1         1         1            1           2m

# ReplicaSets
NAMESPACE     NAME                                 DESIRED   CURRENT   READY     AGE
kube-system   coredns-86c58d9df4                   2         2         2         2m
kube-system   default-http-backend-5ff9d456ff      1         1         1         2m
kube-system   metrics-server-6fc4b7bcff            1         1         1         2m
kube-system   nginx-ingress-controller-7c66d668b   1         1         1         2m
```
{% endtab %}
{% endtabs %}

#### Exercise n°4

Describe the fields associated with each supported API resource.

{% tabs %}
{% tab title="Command" %}
```bash
kubectl api-resources
```
{% endtab %}

{% tab title="CLI Return" %}
```bash
NAME                              SHORTNAMES   APIGROUP                       NAMESPACED   KIND
bindings                                                                      true         Binding
componentstatuses                 cs                                          false        ComponentStatus
configmaps                        cm                                          true         ConfigMap
endpoints                         ep                                          true         Endpoints
events                            ev                                          true         Event
limitranges                       limits                                      true         LimitRange
namespaces                        ns                                          false        Namespace
nodes                             no                                          false        Node
persistentvolumeclaims            pvc                                         true         PersistentVolumeClaim
persistentvolumes                 pv                                          false        PersistentVolume
pods                              po                                          true         Pod
podtemplates                                                                  true         PodTemplate
replicationcontrollers            rc                                          true         ReplicationController
resourcequotas                    quota                                       true         ResourceQuota
secrets                                                                       true         Secret
serviceaccounts                   sa                                          true         ServiceAccount
services                          svc                                         true         Service
mutatingwebhookconfigurations                  admissionregistration.k8s.io   false        MutatingWebhookConfiguration
validatingwebhookconfigurations                admissionregistration.k8s.io   false        ValidatingWebhookConfiguration
customresourcedefinitions         crd,crds     apiextensions.k8s.io           false        CustomResourceDefinition
apiservices                                    apiregistration.k8s.io         false        APIService
controllerrevisions                            apps                           true         ControllerRevision
daemonsets                        ds           apps                           true         DaemonSet
deployments                       deploy       apps                           true         Deployment
replicasets                       rs           apps                           true         ReplicaSet
statefulsets                      sts          apps                           true         StatefulSet
tokenreviews                                   authentication.k8s.io          false        TokenReview
localsubjectaccessreviews                      authorization.k8s.io           true         LocalSubjectAccessReview
selfsubjectaccessreviews                       authorization.k8s.io           false        SelfSubjectAccessReview
selfsubjectrulesreviews                        authorization.k8s.io           false        SelfSubjectRulesReview
subjectaccessreviews                           authorization.k8s.io           false        SubjectAccessReview
horizontalpodautoscalers          hpa          autoscaling                    true         HorizontalPodAutoscaler
cronjobs                          cj           batch                          true         CronJob
jobs                                           batch                          true         Job
certificatesigningrequests        csr          certificates.k8s.io            false        CertificateSigningRequest
backendconfigs                                 cloud.google.com               true         BackendConfig
daemonsets                        ds           extensions                     true         DaemonSet
deployments                       deploy       extensions                     true         Deployment
ingresses                         ing          extensions                     true         Ingress
networkpolicies                   netpol       extensions                     true         NetworkPolicy
podsecuritypolicies               psp          extensions                     false        PodSecurityPolicy
replicasets                       rs           extensions                     true         ReplicaSet
nodes                                          metrics.k8s.io                 false        NodeMetrics
pods                                           metrics.k8s.io                 true         PodMetrics
networkpolicies                   netpol       networking.k8s.io              true         NetworkPolicy
poddisruptionbudgets              pdb          policy                         true         PodDisruptionBudget
podsecuritypolicies               psp          policy                         false        PodSecurityPolicy
clusterrolebindings                            rbac.authorization.k8s.io      false        ClusterRoleBinding
clusterroles                                   rbac.authorization.k8s.io      false        ClusterRole
rolebindings                                   rbac.authorization.k8s.io      true         RoleBinding
roles                                          rbac.authorization.k8s.io      true         Role
scalingpolicies                                scalingpolicy.kope.io          true         ScalingPolicy
priorityclasses                   pc           scheduling.k8s.io              false        PriorityClass
storageclasses                    sc           storage.k8s.io                 false        StorageClass
volumeattachments                              storage.k8s.io                 false        VolumeAttachment
```
{% endtab %}
{% endtabs %}

## YAML file

YAML, which stands for Yet Another Markup Language, or YAML Ain’t Markup Language is a human-readable text-based format for specifying configuration-type information.

Using YAML for Kubernetes definitions gives a number of advantages, including:

* **Convenience:** Declaring all the parameters in a command line is no longer needed
* **Maintenance:** YAML files can be added to source control to track changes
* **Flexibility:** Easier to configure complex structure in a file than a command line

YAML is a superset of JSON, which means that any valid JSON file is also a valid YAML file.

The usual basic structure of a Kubernetes YAML file definition look like this :

```yaml
apiVersion: [...]
kind: [...]
metadata:
    - [...]
spec:
    - [...]
```

* **apiVersion** : API version of the object to declare
* **kind** : Kubernetes object to manage \(ex: Pod, Deployment, Service ...\)
* **metadata** : metadata of the object declared \(like labels, annotations ...\)
* **spec** : main part of a YAML file, specification of each parameter defining the object declared \(ex: image, replicas, volumes, secrets, environment ...\)

#### Exercise n°1

Extract the default namespace YAML file definition with the command line.

{% tabs %}
{% tab title="Command" %}
```bash
kubectl get namespace default -o yaml
```
{% endtab %}

{% tab title="CLI Return" %}
```bash
apiVersion: v1
kind: Namespace
metadata:
  creationTimestamp: 2019-02-13T16:56:00Z
  name: default
  resourceVersion: "9"
  selfLink: /api/v1/namespaces/default
  uid: 3db74a6c-2fb0-11e9-bf72-080027afe25b
spec:
  finalizers:
  - kubernetes
status:
  phase: Active
```
{% endtab %}
{% endtabs %}

#### Exercise n°2

Extract only the useful information from the default namespace it in a YAML file.

{% tabs %}
{% tab title="Command" %}
```bash
kubectl get namespace default -o yaml --export
```
{% endtab %}

{% tab title="CLI Return" %}
```bash
apiVersion: v1
kind: Namespace
metadata:
  creationTimestamp: null
  name: default
  selfLink: /api/v1/namespaces/default
spec:
  finalizers:
  - kubernetes
status:
  phase: Active
```
{% endtab %}
{% endtabs %}

## Architecture

_Master_ components provide the cluster’s control plane. Master components make global decisions about the cluster \(for example, scheduling\), and detecting and responding to cluster events \(starting up a new pod when a replication controller’s ‘replicas’ field is unsatisfied\).

_Master_ components can be run on any machine in the cluster. However, for simplicity, set up scripts typically start all master components on the same machine, and do not run user containers on this machine.

_Node_ components are worker machine in Kubernetes, previously known as a minion. They maintain running pods and provide the Kubernetes runtime environment. They are the resources pool that will be managed by the masters to schedule the requested objects.

A basic Kubernetes architecture can be schematized like this :

![Kubernetes architecture](../.gitbook/assets/kubernetes_architecture.png)

#### Exercise n°1

List the all nodes of the cluster and identify the roles of each one.

{% tabs %}
{% tab title="Command" %}
```bash
kubectl get nodes
```
{% endtab %}

{% tab title="CLI Return" %}
```bash
NAME       STATUS    ROLES     AGE       VERSION
minikube   Ready     master    11m       v1.13.2
```
{% endtab %}
{% endtabs %}

#### Exercise n°2

Describe one of the master node.

{% tabs %}
{% tab title="Command" %}
```bash
kubectl describe node HOSTNAME
```
{% endtab %}

{% tab title="CLI Return" %}
```bash
Name:               minikube
Roles:              master
Labels:             beta.kubernetes.io/arch=amd64
                    beta.kubernetes.io/os=linux
                    kubernetes.io/hostname=minikube
                    node-role.kubernetes.io/master=
Annotations:        kubeadm.alpha.kubernetes.io/cri-socket=/var/run/dockershim.sock
                    node.alpha.kubernetes.io/ttl=0
                    volumes.kubernetes.io/controller-managed-attach-detach=true
CreationTimestamp:  Wed, 13 Feb 2019 11:56:00 -0500
Taints:             <none>
Unschedulable:      false
Conditions:
  Type             Status  LastHeartbeatTime                 LastTransitionTime                Reason                       Message
  ----             ------  -----------------                 ------------------                ------                       -------
  MemoryPressure   False   Wed, 13 Feb 2019 12:07:32 -0500   Wed, 13 Feb 2019 11:55:50 -0500   KubeletHasSufficientMemory   kubelet has sufficient memory available
  DiskPressure     False   Wed, 13 Feb 2019 12:07:32 -0500   Wed, 13 Feb 2019 11:55:50 -0500   KubeletHasNoDiskPressure     kubelet has no disk pressure
  PIDPressure      False   Wed, 13 Feb 2019 12:07:32 -0500   Wed, 13 Feb 2019 11:55:50 -0500   KubeletHasSufficientPID      kubelet has sufficient PID available
  Ready            True    Wed, 13 Feb 2019 12:07:32 -0500   Wed, 13 Feb 2019 11:55:50 -0500   KubeletReady                 kubelet is posting ready status
Addresses:
  InternalIP:  10.0.2.15
  Hostname:    minikube
Capacity:
 cpu:                2
 ephemeral-storage:  16888216Ki
 hugepages-2Mi:      0
 memory:             6101440Ki
 pods:               110
Allocatable:
 cpu:                2
 ephemeral-storage:  15564179840
 hugepages-2Mi:      0
 memory:             5999040Ki
 pods:               110
System Info:
 Machine ID:                 0b1678d38b374464b90c69e54313c7e5
 System UUID:                90DE34E2-D436-41B3-AE70-A84228677DA2
 Boot ID:                    5afe6da9-7de1-4a30-abef-e27ce8793ecc
 Kernel Version:             4.15.0
 OS Image:                   Buildroot 2018.05
 Operating System:           linux
 Architecture:               amd64
 Container Runtime Version:  docker://18.6.1
 Kubelet Version:            v1.13.2
 Kube-Proxy Version:         v1.13.2
ExternalID:                  minikube
Non-terminated Pods:         (12 in total)
  Namespace                  Name                                        CPU Requests  CPU Limits  Memory Requests  Memory Limits
  ---------                  ----                                        ------------  ----------  ---------------  -------------
  kube-system                coredns-86c58d9df4-l2hlv                    100m (5%)     0 (0%)      70Mi (1%)        170Mi (2%)
  kube-system                coredns-86c58d9df4-vwf67                    100m (5%)     0 (0%)      70Mi (1%)        170Mi (2%)
  kube-system                default-http-backend-5ff9d456ff-r4fk8       20m (1%)      20m (1%)    30Mi (0%)        30Mi (0%)
  kube-system                etcd-minikube                               0 (0%)        0 (0%)      0 (0%)           0 (0%)
  kube-system                kube-addon-manager-minikube                 5m (0%)       0 (0%)      50Mi (0%)        0 (0%)
  kube-system                kube-apiserver-minikube                     250m (12%)    0 (0%)      0 (0%)           0 (0%)
  kube-system                kube-controller-manager-minikube            200m (10%)    0 (0%)      0 (0%)           0 (0%)
  kube-system                kube-proxy-s5frv                            0 (0%)        0 (0%)      0 (0%)           0 (0%)
  kube-system                kube-scheduler-minikube                     100m (5%)     0 (0%)      0 (0%)           0 (0%)
  kube-system                metrics-server-6fc4b7bcff-wsjsq             0 (0%)        0 (0%)      0 (0%)           0 (0%)
  kube-system                nginx-ingress-controller-7c66d668b-sc6g8    0 (0%)        0 (0%)      0 (0%)           0 (0%)
  kube-system                storage-provisioner                         0 (0%)        0 (0%)      0 (0%)           0 (0%)
Allocated resources:
  (Total limits may be over 100 percent, i.e., overcommitted.)
  CPU Requests  CPU Limits  Memory Requests  Memory Limits
  ------------  ----------  ---------------  -------------
  775m (38%)    20m (1%)    220Mi (3%)       370Mi (6%)
Events:
  Type    Reason                   Age                From                  Message
  ----    ------                   ----               ----                  -------
  Normal  NodeHasSufficientMemory  11m (x7 over 11m)  kubelet, minikube     Node minikube status is now: NodeHasSufficientMemory
  Normal  NodeHasNoDiskPressure    11m (x7 over 11m)  kubelet, minikube     Node minikube status is now: NodeHasNoDiskPressure
  Normal  NodeHasSufficientPID     11m (x9 over 11m)  kubelet, minikube     Node minikube status is now: NodeHasSufficientPID
  Normal  Starting                 11m                kube-proxy, minikube  Starting kube-proxy.
```
{% endtab %}
{% endtabs %}

#### Exercise n°3

Get more information about nodes in one command line.

{% tabs %}
{% tab title="Command" %}
```bash
kubectl get nodes -o wide
```
{% endtab %}

{% tab title="CLI Return" %}
```bash
NAME       STATUS    ROLES     AGE       VERSION   EXTERNAL-IP   OS-IMAGE            KERNEL-VERSION   CONTAINER-RUNTIME
minikube   Ready     master    12m       v1.13.2   <none>        Buildroot 2018.05   4.15.0           docker://18.6.1
```
{% endtab %}
{% endtabs %}

## Namespace

Kubernetes supports multiple virtual clusters backed by the same physical cluster. These virtual clusters are called namespaces.

Namespaces provide a scope for names. Names of resources need to be unique within a namespace, but not across namespaces.

Namespaces are a way to divide cluster resources between multiple users via the definition of resource quotas.

#### Exercise n°1

List all the default namespaces created by the installer.

{% tabs %}
{% tab title="Command" %}
```bash
kubectl get namespace
```
{% endtab %}

{% tab title="CLI Return" %}
```bash
NAME          STATUS    AGE
default       Active    10m
kube-public   Active    10m
kube-system   Active    10m
```
{% endtab %}
{% endtabs %}

#### Exercise n°2

Create the namespace _app-demo_ with the command line.

```bash
kubectl create namespace app-demo
```

#### Exercise n°3

Create a namespace _another-demo_ in declarative mode with a YAML file.

{% code-tabs %}
{% code-tabs-item title="~/data/orchestration/namespace.yaml" %}
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: another-demo
```
{% endcode-tabs-item %}
{% endcode-tabs %}

```bash
kubectl create -f ~/data/orchestration/namespace.yaml
```

#### Exercise n°4

Describe the namespace _app-demo_.

{% tabs %}
{% tab title="Command" %}
```bash
kubectl describe namespace app-demo
```
{% endtab %}

{% tab title="CLI Return" %}
```bash
Name:         app-demo
Labels:       <none>
Annotations:  <none>
Status:       Active

No resource quota.

No resource limits.
```
{% endtab %}
{% endtabs %}

#### Exercise n°5

Delete the namespace named "_another-demo_".

{% hint style="danger" %}
Be careful on the deletion of an object in Kubernetes, there is no rollback.
{% endhint %}

{% hint style="danger" %}
 Be careful on namespace deletion, each objects deployed within it will be deleted too.
{% endhint %}

```bash
# In command line
kubectl delete namespace app-demo

# With declarative file
kubectl delete -f ~/data/orchestration/namespace.yaml
```

## Labels

Labels are key/value pairs that are attached to objects, such as pods. Labels are intended to be used to specify identifying attributes of objects that are meaningful and relevant to users, but do not directly imply semantics to the core system. Labels can be used to organize and to select subsets of objects. Labels can be attached to objects at creation time and subsequently added and modified at any time. Each object can have a set of key/value labels defined. Each Key must be unique for a given object.

#### Exercise n°1

List all nodes of the cluster and display all their labels.

{% tabs %}
{% tab title="Command" %}
```bash
kubectl get nodes --show-labels
```
{% endtab %}

{% tab title="CLI Return" %}
```bash
NAME       STATUS    ROLES     AGE       VERSION   LABELS
minikube   Ready     master    18m       v1.13.2   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/hostname=minikube,node-role.kubernetes.io/master=
```
{% endtab %}
{% endtabs %}

#### Exercise n°2

Add the key/value pair : _random-key=random-value_ to the first node of the cluster.

```bash
kubectl label nodes HOSTNAME random-key=random-value
```

#### Exercise n°3

Delete the key/value pair : _random-key=random-value_ of the first node of the cluster.

```bash
kubectl label nodes HOSTNAME random-key-
```

## Module exercise

The purpose of this section is to manage each steps of the lifecycle of an application to better understand each concepts of the Kubernetes course. 

The main objective in this module is to create a namespace for a future application to isolate it and label the nodes to manage the deployment of each part of the application in the next modules.

For more information about the application used all along the course, please refer to the _Exercise App &gt; Voting App_ link in the left panel.

Based on the principles explain in this module, try by your own to handle this steps. Each steps has to be done in command line thanks to Kubectl.

{% tabs %}
{% tab title="Exercise" %}
1. Create a namespace called _voting-app_
2. Update one node with the key/value label : _type=database_
3. Update another node with the key/value label : _type=queue_
4. Ensure each nodes are correctly configured

{% hint style="info" %}
On single node cluster like Minikube, the key defined must be unique.
{% endhint %}
{% endtab %}

{% tab title="Solution" %}
```bash
kubectl get nodes
kubectl create namespace voting-app
kubectl label node HOSTNAME1 type=database
kubectl label node HOSTNAME2 type=queue
kubectl get nodes --show-labels
```
{% endtab %}
{% endtabs %}

## External documentations

Those documentations can help you to go further in this topic :

* Kubernetes official documentation of [the command line](https://kubernetes.io/docs/tasks/tools/install-kubectl/)
* Kubernetes official [command line cheat sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)
* Kubernetes official documentation of [namespace](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/)
* Kubernetes official documentation of [labels](https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/)
* Kubernetes official documentation of [recommended labels](https://kubernetes.io/docs/concepts/overview/working-with-objects/common-labels/)
* Kubernetes [API reference](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.12/) documentation
* [Kubectl official Reference](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#delete) documentation
* Kubernetes official documentation on [declarative object management](https://kubernetes.io/docs/concepts/overview/object-management-kubectl/declarative-config/)
* Kubernetes official documentation on [imperative object management](https://kubernetes.io/docs/concepts/overview/object-management-kubectl/imperative-config/)

