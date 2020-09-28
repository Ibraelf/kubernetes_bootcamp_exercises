# StorageClass

## Module

A _StorageClass_ provides a way for administrators to describe the “classes” of storage they offer.

#### Overview

At the end of this module, you will :

* _Learn to persist data thanks to a dynamic volumes_
* _Learn to manage the volumes and the claim_
* _Learn to access data within a container_

#### Prerequisites

Create the directory `data/storageclass` in your home folder to manage the YAML file needed in this module.

```bash
mkdir ~/data/storageclass
```

## Create

StorageClasses are the foundation of dynamic provisioning, allowing cluster administrators to define abstractions for the underlying storage platform. Users simply refer to a StorageClass by name in the PersistentVolumeClaim \(PVC\) using the “storageClassName” parameter.

Storage is a critical part of running stateful containers, and Kubernetes offers powerful primitives for managing it. Dynamic volume provisioning, a feature unique to Kubernetes, allows storage volumes to be created on-demand. The storage resources can be dynamically provisioned using the provisioner specified by the StorageClass object. 

StorageClasses are essentially blueprints that abstract away the underlying storage provider, as well as other parameters, like disk-type.

The _create_ command can create a StorageClass object based on a yaml file definition.

#### Exercise n°1

Create a StorageClass object to automatically use the AWS EBS volumes.

{% code-tabs %}
{% code-tabs-item title="~/data/storageclass/01\_storageclass.yaml" %}
```yaml
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: aws-ebs-gp2
provisioner: kubernetes.io/aws-ebs
parameters:
  type: gp2
reclaimPolicy: Retain
mountOptions:
  - debug
volumeBindingMode: Immediate
```
{% endcode-tabs-item %}
{% endcode-tabs %}

Create the resource based on the previous yaml definition file.

```bash
kubectl apply  -f data/storageclass/01_storageclass.yaml
```

## Get

The _get_ command list the object asked. It could be a single object or a list of multiple objects comma separated. This command is useful to get the status of each object. The output can be formatted to only display some information based on some json search or external tools like `tr`, `sort`, `uniq`.

The default output display some useful information about each services :

* Name : the name of the newly created resource
* Provisioner : the target storage provisioner used to create the futur volumes
* Age : the age since his creation

#### Exercise n°1

Get the information of a Storage Class deployed in the default namespace.

{% tabs %}
{% tab title="Command" %}
```bash
kubectl get storageclass
```
{% endtab %}

{% tab title="CLI Return" %}
```bash
NAME                 PROVISIONER                AGE
aws-ebs-gp2          kubernetes.io/aws-ebs      9s
standard (default)   k8s.io/minikube-hostpath   2h
```
{% endtab %}
{% endtabs %}

## Describe

Once an object is running, it is inevitably a need to debug problems or check the configuration deployed.

The _describe_ command display a lot of configuration information about the StorageClass \(labels, annotations, etc.\) and the class definition depending on the external storage service \(provider, access, type ...\).

This command is really useful to introspect and debug an object deployed in a cluster.

#### Exercise n°1

Describe one of the existing StorageClass in the default namespace.

{% tabs %}
{% tab title="Command" %}
```bash
kubectl describe storageclass aws-ebs-gp2
```
{% endtab %}

{% tab title="CLI Return" %}
```bash
Name:                  aws-ebs-gp2
IsDefaultClass:        No
Annotations:           <none>
Provisioner:           kubernetes.io/aws-ebs
Parameters:            type=gp2
AllowVolumeExpansion:  <unset>
MountOptions:
  debug
ReclaimPolicy:      Retain
VolumeBindingMode:  Immediate
Events:             <none>
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
kubectl explain storageclass
```
{% endtab %}

{% tab title="CLI Return" %}
```bash
KIND:     StorageClass
VERSION:  storage.k8s.io/v1

DESCRIPTION:
     StorageClass describes the parameters for a class of storage for which
     PersistentVolumes can be dynamically provisioned. StorageClasses are
     non-namespaced; the name of the storage class according to etcd is in
     ObjectMeta.Name.

FIELDS:
   allowVolumeExpansion	<boolean>
     AllowVolumeExpansion shows whether the storage class allow volume expand

   allowedTopologies	<[]Object>
     Restrict the node topologies where volumes can be dynamically provisioned.
     Each volume plugin defines its own supported topology specifications. An
     empty TopologySelectorTerm list means there is no topology restriction.
     This field is only honored by servers that enable the VolumeScheduling
     feature.

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

   mountOptions	<[]string>
     Dynamically provisioned PersistentVolumes of this storage class are created
     with these mountOptions, e.g. ["ro", "soft"]. Not validated - mount of the
     PVs will simply fail if one is invalid.

   parameters	<map[string]string>
     Parameters holds the parameters for the provisioner that should create
     volumes of this storage class.

   provisioner	<string> -required-
     Provisioner indicates the type of the provisioner.

   reclaimPolicy	<string>
     Dynamically provisioned PersistentVolumes of this storage class are created
     with this reclaimPolicy. Defaults to Delete.

   volumeBindingMode	<string>
     VolumeBindingMode indicates how PersistentVolumeClaims should be
     provisioned and bound. When unset, VolumeBindingImmediate is used. This
     field is only honored by servers that enable the VolumeScheduling feature.

```
{% endtab %}
{% endtabs %}

Add the --recursive flag to display all of the fields at once without descriptions.

## Default Storage Class



The default Storage Class is defined by an annotation set to true : `storageclass.kubernetes.io/is-default-class`

This parameter can be updated in command line or by editing the yaml file of each Storage Class. If a Storage Class already exist, it is needed to disable the annotation to enable it on another object.

#### Exercise n°1

1. Disable the default StorageClass in the default namespace.
2. Configure the previous StorageClass created as default 

```bash
# Disable the current default StorageClass
kubectl patch storageclass CURRENT_DEFAULT_STORAGECLASS_NAME -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"false"}}}'
 
# Configure the new default StorageClass
kubectl patch storageclass NEW_DEFAULT_STORAGECLASS_NAME -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}' 
```

## Delete

The _delete_ command delete resources by filenames, stdin, resources and names, or by resources and label selector.

A StorageClass can only be deleted if it is not used by a running storage object.

Note that the delete command does NOT do resource version checks, so if someone submits an update to a resource right when you submit a delete, their update will be lost along with the rest of the resource.

#### Exercise n°1

Delete the previous storage class in command line.

```bash
kubectl delete storageclasses aws-ebs-gp2
```

## Module exercise

The purpose of this section is to manage each steps of the lifecycle of an application to better understand each concepts of the Kubernetes course.

The main objective in this module is to understand how to dynamically share a storage object to persist and share data of Pods.

For more information about the application used all along the course, please refer to the _Exercise App &gt; Voting App_ link in the left panel.

Based on the principles explain in this module, try by your own to handle this steps. The development of a yaml file is recommended.

The file developed has to be stored in this directory : `~/data/votingapp/08_storageclass`

{% tabs %}
{% tab title="Exercise" %}
1. Create a StorageClass based on the cloud provider.
2. Create a PersistentVolumeClaim to consume that StorageClass and create a storage object capacity of 10Gi.
3. Update the database Deployment to attach the previous PersistentVolumeClaim created.
{% endtab %}

{% tab title="AWS - Solution" %}
Create a StorageClass to consume the AWS Elastic Block Store \(EBS\).

{% code-tabs %}
{% code-tabs-item title="~/data/votingapp/08\_storageclass/storageclass.yaml" %}
```yaml
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: aws-ebs-gp2
provisioner: kubernetes.io/aws-ebs
parameters:
  type: aws-ebs-gp2
reclaimPolicy: Retain
mountOptions:
  - debug
volumeBindingMode: Immediate
```
{% endcode-tabs-item %}
{% endcode-tabs %}

Create a PersistentVolumeClaim to consume the StorageClass.

{% code-tabs %}
{% code-tabs-item title="~/data/votingapp/08\_storageclass/persistentvolumeclaim.yaml" %}
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-database-dynamic
  namespace: voting-app
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
storageClassName: aws-ebs-gp2
```
{% endcode-tabs-item %}
{% endcode-tabs %}

Update the database Deployment to consume the previous PersistentVolumeClaim.

{% code-tabs %}
{% code-tabs-item title="~/data/votingapp/08\_storageclass/deployment.yaml" %}
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
                  key: database-name
            - name: "POSTGRES_USER"
              valueFrom:
                secretKeyRef:
                  name: db
                  key: database-user
            - name: "POSTGRES_PASSWORD"
              valueFrom:
                secretKeyRef:
                  name: db
                  key: database-password
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
            claimName: pvc-database-dynamic
```
{% endcode-tabs-item %}
{% endcode-tabs %}

Create and update each part based on the previous yaml file.

```bash
kubectl apply -f ~/data/votingapp/08_storageclass/
```
{% endtab %}

{% tab title="Azure - Solution" %}
Create a StorageClass to consume the Azure Storage Disk.

{% code-tabs %}
{% code-tabs-item title="~/data/votingapp/08\_storageclass/storageclass.yaml" %}
```yaml
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: azure-disk-slow
provisioner: kubernetes.io/azure-disk
parameters:
  skuName: Standard_LRS
  location: eastus
  storageAccount: azure_storage_account_name
```
{% endcode-tabs-item %}
{% endcode-tabs %}

Create a PersistentVolumeClaim to consume the StorageClass.

{% code-tabs %}
{% code-tabs-item title="~/data/votingapp/08\_storageclass/persistentvolumeclaim.yaml" %}
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-database-dynamic
  namespace: voting-app
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
storageClassName: azure-disk-slow
```
{% endcode-tabs-item %}
{% endcode-tabs %}

Update the database Deployment to consume the previous PersistentVolumeClaim.

{% code-tabs %}
{% code-tabs-item title="~/data/votingapp/08\_storageclass/deployment.yaml" %}
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
                  key: database-name
            - name: "POSTGRES_USER"
              valueFrom:
                secretKeyRef:
                  name: db
                  key: database-user
            - name: "POSTGRES_PASSWORD"
              valueFrom:
                secretKeyRef:
                  name: db
                  key: database-password
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
            claimName: pvc-database-dynamic
```
{% endcode-tabs-item %}
{% endcode-tabs %}

Create and update each part based on the previous yaml file.

```bash
ubectl apply -f ~/data/votingapp/08_storageclass/
```
{% endtab %}

{% tab title="GCP - Solution" %}
Create a StorageClass to consume the GCP Persistent Disk \(GCEPersistentDisk\).

{% code-tabs %}
{% code-tabs-item title="~/data/votingapp/08\_storageclass/storageclass.yaml" %}
```yaml
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: gcp-gce-slow
provisioner: kubernetes.io/gce-pd
parameters:
  type: pd-standard
  replication-type: none
```
{% endcode-tabs-item %}
{% endcode-tabs %}

Create a PersistentVolumeClaim to consume the StorageClass.

{% code-tabs %}
{% code-tabs-item title="~/data/votingapp/08\_storageclass/persistentvolumeclaim.yaml" %}
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-database-dynamic
  namespace: voting-app
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
  storageClassName: gcp-gce-slow
```
{% endcode-tabs-item %}
{% endcode-tabs %}

Update the database Deployment to consume the previous PersistentVolumeClaim.

{% code-tabs %}
{% code-tabs-item title="~/data/votingapp/08\_storageclass/deployment.yaml" %}
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
                  key: database-name
            - name: "POSTGRES_USER"
              valueFrom:
                secretKeyRef:
                  name: db
                  key: database-user
            - name: "POSTGRES_PASSWORD"
              valueFrom:
                secretKeyRef:
                  name: db
                  key: database-password
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
            claimName: pvc-database-dynamic
```
{% endcode-tabs-item %}
{% endcode-tabs %}

Create and update each part based on the previous yaml file.

```bash
ubectl apply -f ~/data/votingapp/08_storageclass/
```
{% endtab %}
{% endtabs %}

## External documentation

Those documentations can help you to go further in this topic :

* Kubernetes official documentation on [storageclass](https://kubernetes.io/docs/concepts/storage/storage-classes/)

