---
description: >-
  Objects of type secret are intended to hold sensitive information, such as
  passwords, OAuth tokens, and ssh keys.
---

# Secrets

## Module

Kubernetes _secret_ objects let you store and manage sensitive information, such as passwords, OAuth tokens, and ssh keys.

#### Overview

At the end of this module, you will :

* _Learn to manage sensitive data within the cluster_
* _Learn to deploy Secrets_
* _Learn to access secrets data within a container_

#### Prerequisites

Create the directory `data/secrets` in your home folder to manage the YAML file needed in this module.

```text
mkdir ~/data/secrets
```

## Create

A Secret is an object that contains a small amount of sensitive data such as a password, a token, or a key. Such information might otherwise be put in a Pod specification or in an image, putting it in a Secret object allows for more control over how it is used, and reduces the risk of accidental exposure.

Independently from the deployment method, the command line to create a Secrets should look like this :

```bash
kubectl create secret TYPE NAME SOURCES
```

* TYPE define which kind of Secret to create :
  * docker-registry : Create a secret for use with a Docker registry
  * generic : Create a secret from a local file, directory or literal value
  * tls : Create a TLS secret
* NAME define the name of the Secret, needed to attach it to a Pod
* SOURCES define the data source for the Secret content to encode :
  * --from-literal : Specify a key and literal value to insert in secret \(i.e. mykey=somevalue\)
  * --from-file : Key files can be specified using their file path, in which case a default name will be given to
  * --from-env-file : Specify the path to a file to read lines of key=val pairs to create a secret

The _create_ command can directly ask the API resource to create a Secret in command line or create a Secret object based on a yaml file definition.

### From literal values

Multiple key-value pairs can be passed to the Kubectl client to create a single Secret. Each pair provided on the command line is represented as a separate entry in the data section of the Secret.

In this particular method, the key / value are explicitly defined in the command line and each key / value will generate a new entry in the Secret.

The command line for this kind of Secrets creation should look like this :

```bash
kubectl create secret generic SECRET_NAME --from-literal=KEY=NAME
```

#### Exercise n°1

Create a Secret called myfirstsecret in command line from a literal value to encode the key / value pair : password=myclearpassword

```bash
kubectl create secret generic myfirstsecret --from-literal=password=myclearpassword
```

{% hint style="warning" %}
This method is not recommended in production. It is recommended to version the Secrets files in a sources repository to manage it with a CI/CD tool.
{% endhint %}

### From a file

Like any other object in Kubernetes, multiple Secrets can be created based on multiple files stocked in different directories. The Kubectl client take care to parse each file given to the command line to ensure that each content files are integrated as Secrets.

The command line for this kind of Secrets creation should look like this :

```bash
kubectl create secret generic SECRET_NAME --from-file=PATH_FILE
```

#### Exercise n°1

1. Create on or more file with some sensitive data.
2. Create a unique Secret based on the files created

```bash
# Create files needed for rest of example.
echo -n 'admin' > ~/data/secrets/username.txt
echo -n '1f2d1e2e67df' > ~/data/secrets/password.txt

# Create the Secrets based on the files created
kubectl create secret generic myfirstsecretfile --from-file=$HOME/data/secrets/username.txt --from-file=$HOME/data/secrets/password.txt
```

### Manually

A Secret can be create manually thanks to a yaml file and the Kubernetes API. This is useful to manage it easily in a source repository and update it via a CI/CD tool.

Pay attention to the format of data in the yaml file definition, the content has to be encoded in base64 to be correctly understand by the Kubernetes API.

#### Exercise n°1

Create a Secret manually in declarative mode.

{% code-tabs %}
{% code-tabs-item title="~/data/secrets/01\_secrets.yaml" %}
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: myfirstsecretmanual
type: Opaque
data:
  username: YWRtaW4=          # echo -n 'admin' | base64
  password: MWYyZDFlMmU2N2Rm  # echo -n '1f2d1e2e67df' | base64
```
{% endcode-tabs-item %}
{% endcode-tabs %}

```bash
kubectl create -f ~/data/secrets/01_secrets.yaml
```

## Get

The _get_ command list the object asked. It could be a single object or a list of multiple objects comma separated. This command is useful to get the status of each object. The output can be formatted to only display some information based on some json search or external tools like `tr`, `sort`, `uniq`.

The default output display some useful information about each services :

* Name : the name of the newly created resource
* Type : the type of Secrets, indication of type of content in the object
* Data : the count of keys in the Secrets
* Age : the age since his creation

#### Exercise n°1

Get a list of existing Secrets in the default namespace.

{% tabs %}
{% tab title="Command" %}
```bash
kubectl get secrets
```
{% endtab %}

{% tab title="CLI Return" %}
```bash
NAME                  TYPE                                  DATA      AGE
default-token-wpxhp   kubernetes.io/service-account-token   3         2h
myfirstsecret         Opaque                                1         2m
myfirstsecretfile     Opaque                                2         4s
```
{% endtab %}
{% endtabs %}

## Describe

Once an object is running, it is inevitably a need to debug problems or check the configuration deployed.

The _describe_ command display a lot of configuration information about the Secrets \(labels, resource requirements, etc.\) or any other Kubernetes objects, as well as status information about the Secrets and Pod \(state, readiness, restart count, events, etc.\).

This command is really useful to introspect and debug an object deployed in a cluster.

#### Exercise n°1

Describe one of the existing Secrets in the default namespace.

{% tabs %}
{% tab title="Command" %}
```bash
kubectl describe secrets myfirstsecretfile
```
{% endtab %}

{% tab title="CLI Return" %}
```bash
Name:         myfirstsecretfile
Namespace:    default
Labels:       <none>
Annotations:  <none>

Type:  Opaque

Data
====
username.txt:  5 bytes
password.txt:  12 bytes
```
{% endtab %}
{% endtabs %}

## Attach

Once a Secret is created, it has to be attach to a pod to be able to read the data within it.

The data can be attach as :

* Environment variable : The application in the container has to be able to read data from those environment variables.
* File : The application in the container has to be able to read the content in the specific files and find the needed keys to get the right value.

Multiple Secrets can be defined in a Pod, this is useful when the configuration has to be managed by different teams.

### In environment variable

Inside a container that consumes a secret in an environment variables, the secret keys appear as normal environment variables containing the base-64 decoded values of the secret data.

#### Exercise n°1

Based on the previous Secrets created, create a Pod using the busybox image to display some part of the Secret in single environment variables.

{% code-tabs %}
{% code-tabs-item title="~/data/secrets/02\_pods.yaml" %}
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myfirstsecretenv
spec:
  containers:
  - name: busybox
    image: busybox
    command: [ "/bin/sh", "-c", "env" ]
    env:
      - name: SECRET_PASSWORD
        valueFrom:
          secretKeyRef:
            name: myfirstsecret
            key: password
```
{% endcode-tabs-item %}
{% endcode-tabs %}

Create the resource based on the previous yaml file definition.

```bash
kubectl create -f ~/data/secrets/02_pods.yaml
```

Get the environment variable to ensure the configuration.

{% tabs %}
{% tab title="Command" %}
```bash
kubectl logs myfirstsecretenv
```
{% endtab %}

{% tab title="CLI Return" %}
```bash
[...]
SECRET_PASSWORD=myclearpassword
[...]
```
{% endtab %}
{% endtabs %}

### In file path

With this method, the data of a Secrets is directly connected as a Volume to a Pod in a specific path in the container to be readable by the application deployed.

#### Exercise n°1

Based on the previous Secrets created, create a Pod using the busybox image to display some part of the Secret in single file.

{% code-tabs %}
{% code-tabs-item title="~/data/secrets/03\_pods.yaml" %}
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myfirstsecretfile
spec:
  containers:
  - name: nginx
    image: nginx
    volumeMounts:
    - name: myfirstsecretmount
      mountPath: "/data/sensitive"
      readOnly: true
  volumes:
  - name: myfirstsecretmount
    secret:
      secretName: myfirstsecret
      items:
      - key: password
        path: myapp/my-password
```
{% endcode-tabs-item %}
{% endcode-tabs %}

Create the resource based on the previous yaml file definition.

```text
kubectl create -f ~/data/secrets/03_pods.yaml
```

Get the volume mount information.

{% tabs %}
{% tab title="Command" %}
```bash
kubectl exec -it myfirstsecretfile cat ~/data/sensitive/myapp/my-password
```
{% endtab %}

{% tab title="CLI Return" %}
```bash
myclearpassword
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
kubectl explain secrets
```
{% endtab %}

{% tab title="CLI Return" %}
```bash
KIND:     Secret
VERSION:  v1

DESCRIPTION:
     Secret holds secret data of a certain type. The total bytes of the values
     in the Data field must be less than MaxSecretSize bytes.

FIELDS:
   apiVersion	<string>
     APIVersion defines the versioned schema of this representation of an
     object. Servers should convert recognized schemas to the latest internal
     value, and may reject unrecognized values. More info:
     https://git.k8s.io/community/contributors/devel/api-conventions.md#resources

   data	<map[string]string>
     Data contains the secret data. Each key must consist of alphanumeric
     characters, '-', '_' or '.'. The serialized form of the secret data is a
     base64 encoded string, representing the arbitrary (possibly non-string)
     data value here. Described in https://tools.ietf.org/html/rfc4648#section-4

   kind	<string>
     Kind is a string value representing the REST resource this object
     represents. Servers may infer this from the endpoint the client submits
     requests to. Cannot be updated. In CamelCase. More info:
     https://git.k8s.io/community/contributors/devel/api-conventions.md#types-kinds

   metadata	<Object>
     Standard object's metadata. More info:
     https://git.k8s.io/community/contributors/devel/api-conventions.md#metadata

   stringData	<map[string]string>
     stringData allows specifying non-binary secret data in string form. It is
     provided as a write-only convenience method. All keys and values are merged
     into the data field on write, overwriting any existing values. It is never
     output when reading from the API.

   type	<string>
     Used to facilitate programmatic handling of secret data.

```
{% endtab %}
{% endtabs %}

Add the --recursive flag to display all of the fields at once without descriptions.

## Delete

The _delete_ command delete resources by filenames, stdin, resources and names, or by resources and label selector.

A Secrets can only be deleted when it is not attached to a Pod.

Note that the delete command does NOT do resource version checks, so if someone submits an update to a resource right when you submit a delete, their update will be lost along with the rest of the resource.

#### Exercise n°1

Delete the previous Secrets created.

```bash
# Delete Pods created
kubectl delete pods myfirstsecretenv myfirstsecretfile
# Delete Secrets created
kubectl delete secrets myfirstsecret myfirstsecretfile
```

## Module exercise

The purpose of this section is to manage each steps of the lifecycle of an application to better understand each concepts of the Kubernetes course.

The main objective in this module is to understand how to externalized some sensitive data of an application to simplify the development and the lifecycle of both application and secrets.

For more information about the application used all along the course, please refer to the _Exercise App &gt; Voting App_ link in the left panel.

Based on the principles explain in this module, try by your own to handle this steps. The development of a yaml file is recommended.

The file developed has to be stored in this directory : `~/data/votingapp/06_secrets`

{% tabs %}
{% tab title="Exercise" %}
1. Delete the db, result and worker Deployment to be able to update the Secrets management
2. Create a Secrets resource to externalize some part of the vote Pods :
   1. Named the Secrets _db_
   2. The Secrets must manage those data : 
      1. `name: voting`
      2. `password: mypassword`
      3. `user: voting`
3. Update the Deployment of the vote Pods to attach the Secrets as environment variables :
   1. The name of the `name` environment variable has to be POSTGRESQL\_DATABASE
   2. The name of the `password` environment variable has to be POSTGRESQL\_PASSWORD
   3. The name of the `user` environment variable has to be POSTGRESQL\_USERNAME
{% endtab %}

{% tab title="Solution" %}
Delete the current db Deployment.

```bash
kubectl delete deployment db result worker -n voting-app
```

Create the Secrets in command line.

```bash
kubectl create secret generic db -n voting-app --from-literal=POSTGRES_DB=voting --from-literal=POSTGRES_PASSWORD=mypassword --from-literal=POSTGRES_USER=voting
```

An example of yaml definition file to update the Deployments of each Pods.

{% code-tabs %}
{% code-tabs-item title="~/data/votingapp/06\_secrets/deployment.yaml" %}
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
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: result
  namespace: voting-app
spec:
  replicas: 1
  selector:
    matchLabels:
      name: result
  strategy:
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 1
    type: RollingUpdate
  template:
    metadata:
      labels:
        name: result
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
          image: wikitops/examplevotingapp-result:1.1
          imagePullPolicy: IfNotPresent
          name: result
          ports:
            - name: result
              containerPort: 8080
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: worker
  namespace: voting-app
spec:
  replicas: 1
  selector:
    matchLabels:
      name: worker
  strategy:
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 1
    type: RollingUpdate
  template:
    metadata:
      labels:
        name: worker
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
          image: wikitops/examplevotingapp-worker:1.1
          imagePullPolicy: IfNotPresent
          name: worker
```
{% endcode-tabs-item %}
{% endcode-tabs %}

Update the current Deployments resources.

```bash
kubectl create -f ~/data/votingapp/06_secrets/deployment.yaml
```
{% endtab %}
{% endtabs %}

## External documentation

Those documentations can help you to go further in this topic :

* Kubernetes official documentation on [Secrets](https://kubernetes.io/docs/concepts/configuration/secret/)
* Kubernetes official documentation to [distribute credentials securely with Secrets](https://kubernetes.io/docs/tasks/inject-data-application/distribute-credentials-secure/)

