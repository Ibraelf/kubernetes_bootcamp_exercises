---
description: >-
  ConfigMaps allow you to decouple configuration artifacts from image content to
  keep containerized applications portable.
---

# ConfigMaps

## Module

_ConfigMaps_ allow you to decouple configuration artifacts from image content to keep containerized applications portable.

#### Overview

At the end of this module, you will :

* _Learn the format of a YAML ConfigMaps file_
* _Learn how to manage a ConfigMaps_
* _Learn the composition of a ConfigMaps_

#### Prerequisites

Create the directory `data/configmap` in your home folder to manage the YAML file needed in this module.

```bash
mkdir ~/data/configmap
```

## Create

ConfigMap can be created from directories, files, or literal values thanks to the Kubectl client.

Independently from the deployment method, the command line to create a ConfigMaps should look like this :

```bash
kubectl create configmap CONFIGMAP_NAME DATA_SOURCE
```

* CONFIGMAP\_NAME is the name to assign to the ConfigMap 
* DATA\_SOURCE is the directory, file, or literal value to draw the data from

The data source corresponds to a key-value pair in the ConfigMap, where

* key is the file name or the key provided on the command line
* value is the file contents or the literal value provided on the command line

The _create_ command can directly ask the API resource to create a Service in command line or create a ConfigMap object based on a yaml file\(s\) definition.

### From literal values

Multiple key-value pairs can be passed to the Kubectl client to create a single ConfigMap. Each pair provided on the command line is represented as a separate entry in the data section of the ConfigMap.

In this particular method, the key / value are explicitly defined in the command line and each key / value will generate a new entry in the ConfigMap.

The command line for this kind of ConfigMaps creation should look like this :

```bash
kubectl create configmap CONFIGMAP_NAME --from-literal=KEY_NAME=VALUE
```

#### Exercise n°1

Create a ConfigMap named mysimpleconfigmap with those key / value pairs :

* course=kubernetes
* subject=configmap
* subject.title=mysimpleconfigmap

```bash
kubectl create configmap mysimpleconfigmap --from-literal=course=kubernetes --from-literal=subject=configmap --from-literal=subject.title=MySimpleConfigMap
```

{% hint style="info" %}
This method is not recommended in production. It is recommended to version the ConfigMaps files in a sources repository to manage it with a CI/CD tool.
{% endhint %}

### From a directory

Like any other object in Kubernetes, multiple ConfigMap can be created based on files stocked in a directory. The Kubectl client take care to parse each file in the directory given to ensure that each content files are integrated as ConfigMaps.

In this particular method, the key of the ConfigMap will be the name of the file and the value will be the entire content of the file.

The command line for this kind of ConfigMaps creation should look like this :

```bash
kubectl create configmap CONFIGMAP_NAME --from-file=PATH_TO_DIRECTORY
```

#### Exercise n°1

1. Create this file : `~/data/configmaps/directory/configmap.properties`
2. Copy / paste the content below in the new file
3. Create a ConfigMap based on the directory

```bash
course=kubernetes
subject=configmaps
subject.title=mysecondconfigmap
```

```bash
kubectl create configmap myconfigmapdir --from-file=~/data/configmaps/directory/
```

### From a file

Like any other object in Kubernetes, multiple ConfigMap can be created based on multiple files stocked in a different directories. The Kubectl client take care to parse each file given to the command line to ensure that each content files are integrated as ConfigMaps.

In this particular method, the key of the ConfigMap will be the name of the file and the value will be the entire content of the file. In this method, it is possible to specify a key at the ConfigMap creation to avoid using the file name.

The command line for this kind of ConfigMaps creation should look like this :

```bash
# Without key specification
kubectl create configmap CONFIGMAP_NAME --from-file=PATH_TO_FILENAME

# With key specification
kubectl create configmap CONFIGMAP_NAME --from-file=KEY_NAME=PATH_TO_FILENAME
```

#### Exercise n°1

1. Create two properties files in this path : `~/data/configmap/directory/`
2. Put some properties in each one
3. Create a ConfigMap based on the two property files created

```bash
kubectl create configmap myconfigmapfile --from-file=PATH_FILE1 --from-file=PATH_FILE2
```

## Get

The _get_ command list the object asked. It could be a single object or a list of multiple objects comma separated. This command is useful to get the status of each object. The output can be formatted to only display some information based on some JSON search or external tools like `tr`, `sort`, `uniq`.

The default output display some useful information about each ConfigMaps :

* Name : the name of the newly created resource
* Data : the count of keys in the ConfigMap
* Age : the age since his creation

#### Exercise n°1

Get a list of ConfigMaps deployed in the default namespace.

{% tabs %}
{% tab title="Command" %}
```bash
kubectl get configmaps
```
{% endtab %}

{% tab title="CLI Return" %}
```bash
NAME                DATA      AGE
myconfigmapdir      1         21s
myconfigmapfile     1         7s
mysimpleconfigmap   3         58s
```
{% endtab %}
{% endtabs %}

## Describe

Once an object is running, it is inevitably a need to debug problems or check the configuration deployed.

The _describe_ command display a lot of configuration information about the ConfigMaps \(labels, resource requirements, etc.\) or any other Kubernetes objects, as well as status information about the ConfigMaps and Pod \(state, readiness, restart count, events, etc.\).

This command is really useful to introspect and debug an object deployed in a cluster.

#### Exercise n°1

Describe the ConfigMaps deployed in the default namespace with a file or a directory.

{% tabs %}
{% tab title="Command" %}
```bash
kubectl describe configmaps myconfigmapdir
```
{% endtab %}

{% tab title="CLI Return" %}
```bash
Name:         myconfigmapdir
Namespace:    default
Labels:       <none>
Annotations:  <none>

Data
====
cm:
----
course=kubernetes
subject=configmaps
subject.title=mysecondconfigmap

Events:  <none>
```
{% endtab %}
{% endtabs %}

#### Exercise n°2

Describe the ConfigMaps deployed in the default namespace with litteral values.

{% tabs %}
{% tab title="Command" %}
```bash
kubectl describe configmaps mysimpleconfigmap
```
{% endtab %}

{% tab title="CLI Return" %}
```bash
Name:         mysimpleconfigmap
Namespace:    default
Labels:       <none>
Annotations:  <none>

Data
====
subject.title:
----
MySimpleConfigMap
course:
----
kubernetes
subject:
----
configmap
Events:  <none>
```
{% endtab %}
{% endtabs %}

## Attach

Once a ConfigMap is created, it has to be attach to a pod to be able to read the data within it.

The data can be attach as :

* Environment variable : The application in the container has to be able to read data from those environment variables.
* File : The application in the container has to be able to read the content in the specific files and find the needed keys to get the right value.

Multiple ConfigMaps can be defined in a Pod, this is useful when the configuration has to be managed by different teams.

### In environment variable

There is two method to define the content of a ConfigMap in a Pod :

* Define each variable individually
* Define each key / value pairs in a ConfigMap as an environment variable automatically

Depending of the ConfigMap content, defining each variable individually can be much complex and more secure than giving access to all the content of a ConfigMap.

#### Exercise n°1

Based on the previous ConfigMaps created, create a Pod using the busybox image to display some part of the ConfigMap in single environment variables.

{% code-tabs %}
{% code-tabs-item title="~/data/configmap/01\_pod.yaml" %}
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myfirstconfigmapenv1
spec:
  containers:
    - name: busybox
      image: k8s.gcr.io/busybox
      command: [ "/bin/sh", "-c", "env" ]
      env:
        - name: CM_COURSE
          valueFrom:
            configMapKeyRef:
              name: mysimpleconfigmap
              key: course
        - name: CM_SUBJECT_TITLE
          valueFrom:
            configMapKeyRef:
              name: mysimpleconfigmap
              key: subject.title
```
{% endcode-tabs-item %}
{% endcode-tabs %}

Create the Pod to attach the ConfigMap.

```bash
kubectl apply  -f data/configmap/01_pod.yaml
```

Get the logs to ensure the Pods is running and configured.

{% tabs %}
{% tab title="Command" %}
```bash
kubectl logs myfirstconfigmapenv1
```
{% endtab %}

{% tab title="CLI Return" %}
```bash
[...]
CM_SUBJECT_TITLE=MySimpleConfigMap
CM_COURSE=kubernetes
[...]
```
{% endtab %}
{% endtabs %}

#### Exercise n°2

Based on the previous ConfigMaps created, create a Pod using the busybox image to display the entire ConfigMap in environment variables automatically.

{% code-tabs %}
{% code-tabs-item title="~/data/configmap/02\_pod.yaml" %}
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myfirstconfigmapenv2
spec:
  containers:
    - name: busybox
      image: k8s.gcr.io/busybox
      command: [ "/bin/sh", "-c", "env" ]
      envFrom:
      - configMapRef:
          name: mysimpleconfigmap
```
{% endcode-tabs-item %}
{% endcode-tabs %}

Create the Pod to attach the ConfigMap.

```bash
kubectl apply  -f data/configmap/02_pod.yaml
```

Get the logs to ensure the Pods is running and configured.

{% tabs %}
{% tab title="Command" %}
```bash
kubectl logs myfirstconfigmapenv2
```
{% endtab %}

{% tab title="CLI Return" %}
```bash
[...]
subject=configmap
course=kubernetes
[...]
subject.title=MySimpleConfigMap
[...]
```
{% endtab %}
{% endtabs %}

### In file path

With this method, the data of a ConfigMap is directly connected as a Volume to a Pod in a specific path in the container to be readable by the application deployed.

As the previous method, the content of a ConfigMap can entirely be copied in the specific file or only some specific keys can be defined to be available in the specific path.

{% hint style="danger" %}
Be careful in the path used to mount the volumes, if a file already exist, it will be erased to mount the ConfigMap Volume.
{% endhint %}

#### Exercise n°1

Based on the previous ConfigMaps created, create a Pod using the busybox image to display some keys of the ConfigMap mounted as Volumes.

{% code-tabs %}
{% code-tabs-item title="~/data/configmap/03\_pod.yaml" %}
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myfirstconfigmapfile1
spec:
  containers:
    - name: busybox
      image: k8s.gcr.io/busybox
      command: [ "/bin/sh","-c","cat /tmp/myconfigmap/mycourse" ]
      volumeMounts:
      - name: myconfigmap
        mountPath: /tmp/myconfigmap
  volumes:
    - name: myconfigmap
      configMap:
        name: mysimpleconfigmap
        items:
        - key: course
          path: mycourse
```
{% endcode-tabs-item %}
{% endcode-tabs %}

Create the Pod to attach the ConfigMap.

```bash
kubectl apply  -f data/configmap/03_pod.yaml
```

Get the logs to ensure the Pods is running and configured.

{% tabs %}
{% tab title="Command" %}
```bash
kubectl logs myfirstconfigmapfile1
```
{% endtab %}

{% tab title="CLI Return" %}
```text
kubernetes
```
{% endtab %}
{% endtabs %}

#### Exercise n°2

Based on the previous ConfigMaps created, create a Pod using the busybox image to display the entire content of a ConfigMap mounted as Volumes.

{% code-tabs %}
{% code-tabs-item title="~/data/configmap/04\_pod.yaml" %}
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myfirstconfigmapfile2
spec:
  containers:
    - name: busybox
      image: k8s.gcr.io/busybox
      command: [ "/bin/sh","-c","sleep 3600" ]
      volumeMounts:
      - name: myconfigmap
        mountPath: /tmp/myconfigmap
  volumes:
    - name: myconfigmap
      configMap:
        name: mysimpleconfigmap
```
{% endcode-tabs-item %}
{% endcode-tabs %}

Create the Pod to attach the ConfigMap.

```bash
kubectl apply  -f data/configmap/04_pod.yaml
```

Get the logs to ensure the Pods is running and configured.

{% tabs %}
{% tab title="Command" %}
```bash
kubectl exec myfirstconfigmapfile2 -- ls -ailh /tmp/myconfigmap
kubectl exec myfirstconfigmapfile2 -- cat /tmp/myconfigmap/course
```
{% endtab %}

{% tab title="CLI Return" %}
```bash
total 12[...]
6304104 lrwxrwxrwx    1 root     root            13 Feb 13 18:51 course -> ..data/course
6304105 lrwxrwxrwx    1 root     root            14 Feb 13 18:51 subject -> ..data/subject
6304106 lrwxrwxrwx    1 root     root            20 Feb 13 18:51 subject.title -> ..data/subject.title
```

```bash
kubernetes
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
kubectl explain configmap
```
{% endtab %}

{% tab title="CLI Return" %}
```bash
KIND:     ConfigMap
VERSION:  v1

DESCRIPTION:
     ConfigMap holds configuration data for pods to consume.

FIELDS:
   apiVersion	<string>
     APIVersion defines the versioned schema of this representation of an
     object. Servers should convert recognized schemas to the latest internal
     value, and may reject unrecognized values. More info:
     https://git.k8s.io/community/contributors/devel/api-conventions.md#resources

   binaryData	<map[string]string>
     BinaryData contains the binary data. Each key must consist of alphanumeric
     characters, '-', '_' or '.'. BinaryData can contain byte sequences that are
     not in the UTF-8 range. The keys stored in BinaryData must not overlap with
     the ones in the Data field, this is enforced during validation process.
     Using this field will require 1.10+ apiserver and kubelet.

   data	<map[string]string>
     Data contains the configuration data. Each key must consist of alphanumeric
     characters, '-', '_' or '.'. Values with non-UTF-8 byte sequences must use
     the BinaryData field. The keys stored in Data must not overlap with the
     keys in the BinaryData field, this is enforced during validation process.

   kind	<string>
     Kind is a string value representing the REST resource this object
     represents. Servers may infer this from the endpoint the client submits
     requests to. Cannot be updated. In CamelCase. More info:
     https://git.k8s.io/community/contributors/devel/api-conventions.md#types-kinds

   metadata	<Object>
     Standard object's metadata. More info:
     https://git.k8s.io/community/contributors/devel/api-conventions.md#metadata

```
{% endtab %}
{% endtabs %}

Add the --recursive flag to display all of the fields at once without descriptions.

## Delete

The _delete_ command delete resources by filenames, stdin, resources and names, or by resources and label selector.

A ConfigMaps can only be deleted when it is not attached to a Pod.

Note that the delete command does NOT do resource version checks, so if someone submits an update to a resource right when you submit a delete, their update will be lost along with the rest of the resource.

#### Exercise n°1

Delete the previous ConfigMaps created.

```bash
# Delete the Pods created to detach the ConfigMaps
kubectl delete pods myfirstconfigmapenv1 myfirstconfigmapenv2 myfirstconfigmapfile1 myfirstconfigmapfile2

# Delete the ConfigMaps created
kubectl delete configmaps myconfigmapdir mysimpleconfigmap myconfigmapfile
```

## Module exercise

The purpose of this section is to manage each steps of the lifecycle of an application to better understand each concepts of the Kubernetes course.

The main objective in this module is to understand how to externalized some configuration part of an application to simplify the development and the lifecycle of both application and configuration.

For more information about the application used all along the course, please refer to the _Exercise App &gt; Voting App_ link in the left panel.

Based on the principles explain in this module, try by your own to handle this steps. The development of a yaml file is recommended.

The file developed has to be stored in this directory : `~/data/votingapp/05_configmaps`

{% tabs %}
{% tab title="Exercise" %}
1. Delete the vote Deployment to be able to manage the environment variable with a ConfigMaps
2. Create a ConfigMaps resource to externalize some part of the vote Pods :
   1. Name the ConfigMaps _vote_
   2. The ConfigMaps must manage those data : 
      1. `option_a: "CATS"`
      2. `option_b: "DOGS"`
3. Update the Deployment of the vote Pods to attach the ConfigMaps as environment variables :
   1. The name of the `option_a` environment variable has to be OPTION\_A
   2. The name of the `option_b` environment variable has to be OPTION\_B
{% endtab %}

{% tab title="Solution" %}
Delete the vote Deployment.

```bash
kubectl delete deployment vote -n voting-app
```

Ccreate the ConfigMaps in command line.

```bash
kubectl create configmap vote -n voting-app --from-literal=option_a=CATS --from-literal=option_b=DOGS
```

An example of yaml definition file to update the Deployments of the vote Pods.

{% code-tabs %}
{% code-tabs-item title="~/data/votingapp/05\_configmaps/deployment.yaml" %}
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: vote
  namespace: voting-app
spec:
  replicas: 1
  selector:
    matchLabels:
      name: vote
  strategy:
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 1
    type: RollingUpdate
  template:
    metadata:
      labels:
        name: vote
    spec:
      containers:
        - env:
           - name: "OPTION_A"
             valueFrom:
               configMapKeyRef:
                 name: vote
                 key: option_a
           - name: "OPTION_B"
             valueFrom:
               configMapKeyRef:
                 name: vote
                 key: option_b
          image: wikitops/examplevotingapp-vote:1.1
          imagePullPolicy: IfNotPresent
          name: vote
          ports:
            - containerPort: 8080
              name: vote
              protocol: TCP
```
{% endcode-tabs-item %}
{% endcode-tabs %}

Create the resource based on the previous yaml file definition.

```bash
kubectl apply  -f data/votingapp/05_configmaps/deployment.yaml
```
{% endtab %}
{% endtabs %}

## External documentation

Those documentations can help you to go further in this topic :

* Kubernetes official documentation on [ConfigMaps](https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/)

