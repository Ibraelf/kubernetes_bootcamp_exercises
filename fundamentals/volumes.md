---
description: >-
  A Kubernetes volume has an explicit lifetime, the same as the pod that
  encloses it. A volume outlives any containers that run within the pod, and
  data is preserved across container restarts.
---

# Volumes

## Module

Kubernetes _volumes_ are an abstraction of external storage that can be attached and shared by multiple Pods.

#### Overview

At the end of this module, you will :

* _Learn to persist data thanks to external volumes_
* _Learn to manage the volumes and the claim_
* _Learn to access data within a container_

#### Prerequisites

Create the directory `data/volumes` in your home folder to manage the YAML file needed in this module.

```bash
mkdir ~/data/volumes
```

## Create

On-disk files in a container are ephemeral, which presents some problems for non-trivial applications when running in containers :

* When a container crashes, they will automatically restart but the files will be lost. The container starts with a clean state.
* When running containers together in a Pod it is often necessary to share files between those containers.

The Kubernetes Volume abstraction solves both of these problems.

The Kubernetes basic architecture can be schematized like this :

![Kubernetes volumes architecture](../.gitbook/assets/volumes_architecture.png)

The _create_ command can create a PersistentVolume / PersistentVolumeClaim object based on a yaml file definition.

### EmptyDir

An EmptyDir Volume, as the name says, attach an empty volume to all the containers in a single Pod.

Containers in the Pod can all read and write the same files in the emptyDir volume, though that volume can be mounted at the same or different paths in each Container. When a Pod is removed from a node for any reason, the data in the emptyDir is deleted forever.

An EmptyDir Volume does not require the definition of an external resource like PersistentVolume or PersistentVolumeClaim. The definition is done directly in the yaml file of the Pod.

#### Exercise n°1

Create an Nginx Pod and attach an EmptyDir volume to it.

{% code-tabs %}
{% code-tabs-item title="~/data/volumes/01\_pods.yaml" %}
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myfirstemptydir
spec:
  containers:
  - name: nginx
    image: nginx
    volumeMounts:
    - name: nginx-storage
      mountPath: /data/test
  volumes:
  - name: nginx-storage
    emptyDir: {}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

Create the resource based on the previous yaml definition file.

```bash
kubectl create -f ~/data/volumes/01_pods.yaml
```

### PersistentVolume

A PersistentVolume is a piece of storage in a Kubernetes cluster. It is a resource in the cluster just like a node is a cluster resource. PersistentVolumes have a lifecycle independent of any individual pod that uses volumes.

**HostPath**

Kubernetes supports hostPath for development and testing on a single-node cluster. A hostPath PersistentVolume uses a file or directory on the Node to emulate network-attached storage.

HostPath volumes should not be used in a production cluster. Instead a cluster administrator would provision a network resource like a Google Compute Engine persistent disk, an NFS share, or an Amazon Elastic Block Store volume. Cluster administrators can also use StorageClasses to set up dynamic provisioning.

#### Exercise n°1

Create an hostPath volume based on that directory :`/data/nginx/conf`

{% code-tabs %}
{% code-tabs-item title="~/data/volumes/02\_hostpath.yaml" %}
```yaml
kind: PersistentVolume
apiVersion: v1
metadata:
  name: myfirsthostpathvolume
spec:
  storageClassName: manual
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/data/nginx/conf"
```
{% endcode-tabs-item %}
{% endcode-tabs %}

Create the resource based on the previous yaml definition file.

```bash
kubectl create -f ~/data/volumes/02_hostpath.yaml
```

## Claim

A PersistentVolumeClaim is a request for storage by a user. It is similar to a pod. Pods consume node resources and PersistentVolumeClaim consume PersistentVolume resources. Pods can request specific levels of resources \(CPU and Memory\). Claims can request specific size and access modes \(e.g., can be mounted once read/write or many times read-only\).

#### Exercise n°1

Create the PersistentVolumeClaim to claim the previous PersistentVolume provisioned.

{% code-tabs %}
{% code-tabs-item title="~/data/volumes/03\_persistentvolumeclaim.yaml" %}
```yaml
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: myfirsthostpathvolumeclaim
spec:
  storageClassName: manual
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
```
{% endcode-tabs-item %}
{% endcode-tabs %}

Create the resource based on the previous yaml definition file.

```bash
kubectl create -f ~/data/volumes/03_persistentvolumeclaim.yaml
```

## Attach

Undependently on the PersistentVolume type used, attach a Volume to a Pod can be done in the yaml file definition of a Pod.

#### Exercise n°1

Attach the previous PersistentVolumeClaim to an Nginx Pod to consume the hostPath PersistentVolume provisioned before.

{% code-tabs %}
{% code-tabs-item title="~/data/volumes/04\_pods.yaml" %}
```yaml
kind: Pod
apiVersion: v1
metadata:
  name: myfirsthostpathpod
spec:
  volumes:
    - name: myfirsthostpathvolume
      persistentVolumeClaim:
       claimName: myfirsthostpathvolumeclaim
  containers:
    - name: nginx
      image: nginx
      ports:
        - containerPort: 80
      volumeMounts:
        - mountPath: "/usr/share/nginx/html"
          name: myfirsthostpathvolume
```
{% endcode-tabs-item %}
{% endcode-tabs %}

Create the resource based on the previous yaml definition file.

```bash
kubectl create -f ~/data/volumes/04_pods.yaml
```

## Get

The _get_ command list the object asked. It could be a single object or a list of multiple objects comma separated. This command is useful to get the status of each object. The output can be formatted to only display some information based on some json search or external tools like `tr`, `sort`, `uniq`.

### PersistentVolume

The default output display some useful information about each services :

* Name : the name of the newly created resource
* Capacity : the storage capacity available in the volume
* Access modes : the access modes available \(read, write, owner\)
* Reclaim policy :  the management of the resource when pod is deleted
* Status : the status of the resource
* Claim : the claim object associated
* Storageclass : the storage class used to create the new resource
* Reason : the reason of the resource creation
* Age : the age since the creation resource 

#### Exercise n°1

List all existing Volumes in the default namespace.

{% tabs %}
{% tab title="Command" %}
```bash
kubectl get persistentvolume
```
{% endtab %}

{% tab title="CLI Return" %}
```bash
NAME                    CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS    CLAIM                                STORAGECLASS   REASON    AGE
myfirsthostpathvolume   1Gi        RWO            Retain           Bound     default/myfirsthostpathvolumeclaim   manual                   3m
```
{% endtab %}
{% endtabs %}

### PersistentVolumeClaim

The default output display some useful information about each services :

* Name : the name of the newly created resource
* Status : is the resource bounded, mounted, etc
* Volume : the PersitentVolume impacted by the newly created resource
* Capacity : volume storage size claimed
* Access mode : access mode of the newly created resource
* Storageclass : the name of the StorageClass associated
* Age : the age since the creation resource

#### Exercise n°1

List all existing Volume claims in the default namespace.

{% tabs %}
{% tab title="Command" %}
```bash
kubectl get persistentvolumeclaim
```
{% endtab %}

{% tab title="CLI Return" %}
```bash
NAME                         STATUS    VOLUME                  CAPACITY   ACCESS MODES   STORAGECLASS   AGE
myfirsthostpathvolumeclaim   Bound     myfirsthostpathvolume   1Gi        RWO            manual         2m
```
{% endtab %}
{% endtabs %}

## Describe

Once an object is running, it is inevitably a need to debug problems or check the configuration deployed.

The _describe_ command display a lot of configuration information about the Volumes \(labels, annotations, etc.\) and the claim policy of each resources \(storageclass, type, size, access, etc\).

This command is really useful to introspect and debug an object deployed in a cluster.

#### Exercise  n°1

Describe one of the existing PersistentVolume in the default namespace.

{% tabs %}
{% tab title="Command" %}
```bash
kubectl describe persistentvolume myfirsthostpathvolume
```
{% endtab %}

{% tab title="CLI Return" %}
```text
Name:            myfirsthostpathvolume
Labels:          <none>
Annotations:     pv.kubernetes.io/bound-by-controller=yes
Finalizers:      [kubernetes.io/pv-protection]
StorageClass:    manual
Status:          Bound
Claim:           default/myfirsthostpathvolumeclaim
Reclaim Policy:  Retain
Access Modes:    RWO
VolumeMode:      Filesystem
Capacity:        1Gi
Node Affinity:   <none>
Message:         
Source:
    Type:          HostPath (bare host directory volume)
    Path:          /data/nginx/conf
    HostPathType:  
Events:            <none>
```
{% endtab %}
{% endtabs %}

#### Exercise n°2

Describe one of the existing PersistentVolumeClaim in the default namespace.

{% tabs %}
{% tab title="Command" %}
```bash
kubectl describe persistentvolumeclaim myfirsthostpathvolumeclaim
```
{% endtab %}

{% tab title="CLI Return" %}
```bash
Name:          myfirsthostpathvolumeclaim
Namespace:     default
StorageClass:  manual
Status:        Bound
Volume:        myfirsthostpathvolume
Labels:        <none>
Annotations:   pv.kubernetes.io/bind-completed=yes
               pv.kubernetes.io/bound-by-controller=yes
Finalizers:    [kubernetes.io/pvc-protection]
Capacity:      1Gi
Access Modes:  RWO
VolumeMode:    Filesystem
Events:        <none>
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
kubectl explain persistentvolumes.spec
```
{% endtab %}

{% tab title="CLI Return" %}
```bash
KIND:     PersistentVolume
VERSION:  v1

RESOURCE: spec <Object>

DESCRIPTION:
     Spec defines a specification of a persistent volume owned by the cluster.
     Provisioned by an administrator. More info:
     https://kubernetes.io/docs/concepts/storage/persistent-volumes#persistent-volumes

     PersistentVolumeSpec is the specification of a persistent volume.

FIELDS:
   accessModes	<[]string>
     AccessModes contains all ways the volume can be mounted. More info:
     https://kubernetes.io/docs/concepts/storage/persistent-volumes#access-modes

   awsElasticBlockStore	<Object>
     AWSElasticBlockStore represents an AWS Disk resource that is attached to a
     kubelet's host machine and then exposed to the pod. More info:
     https://kubernetes.io/docs/concepts/storage/volumes#awselasticblockstore

   azureDisk	<Object>
     AzureDisk represents an Azure Data Disk mount on the host and bind mount to
     the pod.

   azureFile	<Object>
     AzureFile represents an Azure File Service mount on the host and bind mount
     to the pod.

   capacity	<map[string]string>
     A description of the persistent volume's resources and capacity. More info:
     https://kubernetes.io/docs/concepts/storage/persistent-volumes#capacity

   cephfs	<Object>
     CephFS represents a Ceph FS mount on the host that shares a pod's lifetime

   cinder	<Object>
     Cinder represents a cinder volume attached and mounted on kubelets host
     machine More info:
     https://releases.k8s.io/HEAD/examples/mysql-cinder-pd/README.md

   claimRef	<Object>
     ClaimRef is part of a bi-directional binding between PersistentVolume and
     PersistentVolumeClaim. Expected to be non-nil when bound. claim.VolumeName
     is the authoritative bind between PV and PVC. More info:
     https://kubernetes.io/docs/concepts/storage/persistent-volumes#binding

   csi	<Object>
     CSI represents storage that handled by an external CSI driver (Beta
     feature).

   fc	<Object>
     FC represents a Fibre Channel resource that is attached to a kubelet's host
     machine and then exposed to the pod.

   flexVolume	<Object>
     FlexVolume represents a generic volume resource that is
     provisioned/attached using an exec based plugin.

   flocker	<Object>
     Flocker represents a Flocker volume attached to a kubelet's host machine
     and exposed to the pod for its usage. This depends on the Flocker control
     service being running

   gcePersistentDisk	<Object>
     GCEPersistentDisk represents a GCE Disk resource that is attached to a
     kubelet's host machine and then exposed to the pod. Provisioned by an
     admin. More info:
     https://kubernetes.io/docs/concepts/storage/volumes#gcepersistentdisk

   glusterfs	<Object>
     Glusterfs represents a Glusterfs volume that is attached to a host and
     exposed to the pod. Provisioned by an admin. More info:
     https://releases.k8s.io/HEAD/examples/volumes/glusterfs/README.md

   hostPath	<Object>
     HostPath represents a directory on the host. Provisioned by a developer or
     tester. This is useful for single-node development and testing only!
     On-host storage is not supported in any way and WILL NOT WORK in a
     multi-node cluster. More info:
     https://kubernetes.io/docs/concepts/storage/volumes#hostpath

   iscsi	<Object>
     ISCSI represents an ISCSI Disk resource that is attached to a kubelet's
     host machine and then exposed to the pod. Provisioned by an admin.

   local	<Object>
     Local represents directly-attached storage with node affinity

   mountOptions	<[]string>
     A list of mount options, e.g. ["ro", "soft"]. Not validated - mount will
     simply fail if one is invalid. More info:
     https://kubernetes.io/docs/concepts/storage/persistent-volumes/#mount-options

   nfs	<Object>
     NFS represents an NFS mount on the host. Provisioned by an admin. More
     info: https://kubernetes.io/docs/concepts/storage/volumes#nfs

   nodeAffinity	<Object>
     NodeAffinity defines constraints that limit what nodes this volume can be
     accessed from. This field influences the scheduling of pods that use this
     volume.

   persistentVolumeReclaimPolicy	<string>
     What happens to a persistent volume when released from its claim. Valid
     options are Retain (default for manually created PersistentVolumes), Delete
     (default for dynamically provisioned PersistentVolumes), and Recycle
     (deprecated). Recycle must be supported by the volume plugin underlying
     this PersistentVolume. More info:
     https://kubernetes.io/docs/concepts/storage/persistent-volumes#reclaiming

   photonPersistentDisk	<Object>
     PhotonPersistentDisk represents a PhotonController persistent disk attached
     and mounted on kubelets host machine

   portworxVolume	<Object>
     PortworxVolume represents a portworx volume attached and mounted on
     kubelets host machine

   quobyte	<Object>
     Quobyte represents a Quobyte mount on the host that shares a pod's lifetime

   rbd	<Object>
     RBD represents a Rados Block Device mount on the host that shares a pod's
     lifetime. More info:
     https://releases.k8s.io/HEAD/examples/volumes/rbd/README.md

   scaleIO	<Object>
     ScaleIO represents a ScaleIO persistent volume attached and mounted on
     Kubernetes nodes.

   storageClassName	<string>
     Name of StorageClass to which this persistent volume belongs. Empty value
     means that this volume does not belong to any StorageClass.

   storageos	<Object>
     StorageOS represents a StorageOS volume that is attached to the kubelet's
     host machine and mounted into the pod More info:
     https://releases.k8s.io/HEAD/examples/volumes/storageos/README.md

   volumeMode	<string>
     volumeMode defines if a volume is intended to be used with a formatted
     filesystem or to remain in raw block state. Value of Filesystem is implied
     when not included in spec. This is a beta feature.

   vsphereVolume	<Object>
     VsphereVolume represents a vSphere volume attached and mounted on kubelets
     host machine

```
{% endtab %}
{% endtabs %}

Add the --recursive flag to display all of the fields at once without descriptions.

## Delete

The _delete_ command delete resources by filenames, stdin, resources and names, or by resources and label selector.

A Volume can only be deleted if it is not attached to a Pod. Use with caution, a deleted volumes cannot be recover.

Note that the delete command does NOT do resource version checks, so if someone submits an update to a resource right when you submit a delete, their update will be lost along with the rest of the resource.

#### Exercise n°1

Delete the previous volumes deployed in the default namespace.

```bash
# Delete the Pods previously created
kubectl delete pods myfirsthostpathpod myfirstemptydir

# Delete the PVC previously created
kubectl delete persistentvolumeclaim myfirsthostpathvolumeclaim

# Delete the volume previously created
kubectl delete persistentvolume myfirsthostpathvolume 
```

## Module exercise

The purpose of this section is to manage each steps of the lifecycle of an application to better understand each concepts of the Kubernetes course.

The main objective in this module is to understand how to share a static storage object to persist and share data of Pods.

For more information about the application used all along the course, please refer to the _Exercise App &gt; Voting App_ link in the left panel.

Based on the principles explain in this module, try by your own to handle this steps. The development of a yaml file is recommended.

The file developed has to be stored in this directory : `~/data/votingapp/07_volumes`

{% tabs %}
{% tab title="Exercise" %}
1. On the node labelized _type=database_, create this directory : `~/data/votingapp/07_volumes/database/data`
2. Create a PersistentVolume based on that local directory previously created. The storage capacity of this volume has to be 10Gi. 
3. Create the PersistentVolumeClaim to consume the PersistentVolume previously created. Manage this object to claim only 5Gi of the PersistentVolume.
4. Attach the volume to the database Pods
{% endtab %}

{% tab title="Solution" %}
Create the directory needed to store the data of the database Pods.

```bash
mkdir -p /data/votingapp/07_volumes/database/data
```

Create a PersistentVolume based on this local path.

{% code-tabs %}
{% code-tabs-item title="~/data/votingapp/07\_volumes/persistentvolume.yaml" %}
```yaml
kind: PersistentVolume
apiVersion: v1
metadata:
  name: pv-database
  namespace: voting-app
  labels:
    type: local
spec:
  storageClassName: manual
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/data/votingapp/07_volumes/database/data"
```
{% endcode-tabs-item %}
{% endcode-tabs %}

Create the PersistentVolume based on the yaml file.

```bash
kubectl create -f ~/data/votingapp/07_volumes/persistentvolume.yaml
```

Create a PersistentVolumeClaim based on the previous PersistentVolume.

{% code-tabs %}
{% code-tabs-item title="~/data/votingapp/07\_volumes/persistentvolumeclaim.yaml" %}
```yaml
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: pvc-database-local
  namespace: voting-app
spec:
  storageClassName: manual
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
```
{% endcode-tabs-item %}
{% endcode-tabs %}

Create the PersistentVolumeClaim based on the yaml file.

```bash
kubectl create -f ~/data/votingapp/07_volumes/persistentvolumeclaim.yaml
```

Update the database Deployment to attach the PersistentVolumeClaim and persist data.

{% code-tabs %}
{% code-tabs-item title="~/data/votingapp/07\_volumes/deployment.yaml" %}
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: db
  namespace: voting-app
spec:
  replicas: 1
  selector:
    matchLabels:
      name: db
  strategy:
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 1
    type: RollingUpdate
  template:
    metadata:
      labels:
        name: db
    spec:
      containers:
        - env:
            - name: "POSTGRES_DB"
              valueFrom:
                secretKeyRef:
                  name: db
                  key: POSTGRES_DB
            - name: "POSTGRES_USER"
              valueFrom:
                secretKeyRef:
                  name: db
                  key: POSTGRES_USER
            - name: "POSTGRES_PASSWORD"
              valueFrom:
                secretKeyRef:
                  name: db
                  key: POSTGRES_PASSWORD
          image: postgres:10.4
          imagePullPolicy: IfNotPresent
          name: db
          ports:
            - name: db
              containerPort: 5432
              protocol: TCP
          volumeMounts:
            - mountPath: /var/lib/postgresql
              name: db-data
      volumes:
        - name: db-data
          persistentVolumeClaim:
            claimName: pvc-database-local
```
{% endcode-tabs-item %}
{% endcode-tabs %}

Update the Deployment based on the yaml file.

```bash
kubectl apply -f ~/data/votingapp/07_volumes/deployment.yaml
```
{% endtab %}
{% endtabs %}

## External documentation

Those documentations can help you to go further in this topic :

* Kubernetes official documentation on [volumes](https://kubernetes.io/docs/concepts/storage/volumes/)
* Kubernetes official documentation on [persistent volumes](https://kubernetes.io/docs/concepts/storage/persistent-volumes/)
* Kubernetes official documentation on [how to attach a volume to a pod](https://kubernetes.io/docs/tasks/configure-pod-container/configure-volume-storage/)

