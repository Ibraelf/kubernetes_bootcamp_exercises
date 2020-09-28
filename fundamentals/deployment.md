---
description: A deployment is a declarative way to manage the state of pods and replicas.
---

# Deployments

## Module 

A _Deployment_ controller provides declarative updates for Pods and ReplicaSets.

#### Overview

At the end of this module, you will :

* _Learn the format of a YAML Deployment file_
* _Learn how to manage a Deployment_
* _Learn how to manage the lifecycle of an application_

#### Prerequisites

Create the directory `data/deployment` in your home folder to manage the YAML file needed in this module.

```bash
mkdir ~/data/deployment
```

## Create

A Deployment controller provides declarative updates for Pods and ReplicaSets.

Describe a desired state in a Deployment object, and the Deployment controller changes the actual state to the desired state at a controlled rate. A Deployment object can create, update or delete resources or parameters.

The _create_ command can directly ask the API resource to create a Deployment in command line or create a Deployment object based on a yaml file definition.

#### Exercise n°1

Create a Deployment object in command line named mynginxdeploymentcli to deploy an Nginx Pod.

```bash
kubectl create deployment mynginxdeploymentcli --image=nginx
```

#### Exercise n°2

Deploy an Nginx Pod in a declarative way with a Deployment object. The ReplicaSet associated has to manage 3 replicas of the Pod. The rolling update strategy will be the default update strategy for this Deployment.

{% code-tabs %}
{% code-tabs-item title="~/data/deployment/01\_deployment.yaml" %}
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mynginxdeploymentyaml
  labels:
    app: mynginxdeploymentyaml
spec:
  replicas: 3
  strategy:
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 1
    type: RollingUpdate
  selector:
    matchLabels:
      app: mynginxdeploymentyaml
  template:
    metadata:
      labels:
        app: mynginxdeploymentyaml
    spec:
      containers:
      - name: nginx
        image: nginx
```
{% endcode-tabs-item %}
{% endcode-tabs %}

```bash
kubectl apply  -f data/deployment/01_deployment.yaml
```

#### Exercise n°3

Deploy a PostgreSQL database in a declarative way with a Deployment object. The ReplicaSet associated has to manage one replica of the Pod. The recreate strategy will be the default update strategy for this Deployment.

PostgreSQL need three environment variables to start correctly :

* POSTGRES\_DB : the database name to be created at the Pod creation
* POSTGRES\_USER : the username owner of the default database
* POSTGRES\_PASSWORD : the password of the newly created user

{% code-tabs %}
{% code-tabs-item title="~/data/deployment/02\_deployment.yaml" %}
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: postgres
spec:
  replicas: 1
  strategy:
    type: Recreate
  selector:
    matchLabels:
      app: postgres
  template:
    metadata:
      labels:
        app: postgres
    spec:
      containers:
        - env:
            - name: POSTGRESQL_DATABASE
              value: "postgresdb"
            - name: POSTGRES_USER
              value: "postgresadmin"
            - name: POSTGRES_PASSWORD
              value: "admin123"
          image: postgres:10.4
          imagePullPolicy: "IfNotPresent"
          name: postgres
          ports:
            - containerPort: 5432
```
{% endcode-tabs-item %}
{% endcode-tabs %}

```bash
kubectl apply  -f data/deployment/02_deployment.yaml
```

## Get

The _get_ command list the object asked. It could be a single object or a list of multiple objects comma separated. This command is useful to get the status of each object. The output can be formatted to only display some information based on some json search or external tools like `tr`, `sort`, `uniq`.

The default output display some useful information about each services :

* Name : the name of the newly created resource
* Desired : the number of desired running Pods, similar to a ReplicaSet output
* Current : the number of current running Pods, similar to a ReplicaSet output
* Up-to-date : the number of up to date Pods when a Deployment is updated
* Available : the number of production ready Pods, similar to a ReplicaSet output
* Age : the age since his creation

#### Exercise n°1

Get a list of the existing deployment object in the default namespace.

{% tabs %}
{% tab title="Command" %}
```bash
kubectl get deployment
```
{% endtab %}

{% tab title="CLI Return" %}
```bash
NAME                    DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
mynginxdeploymentcli    1         1         1            1           33s
mynginxdeploymentyaml   3         3         3            3           15s
```
{% endtab %}
{% endtabs %}

## Describe

Once an object is running, it is inevitably a need to debug problems or check the configuration deployed.

The _describe_ command display a lot of configuration information about the Deployment\(s\) \(labels, resource requirements, etc.\) or any other Kubernetes objects, as well as status information about the Deployment\(s\) and Pod \(state, readiness, restart count, events, etc.\).

This command is really useful to introspect and debug an object deployed in a cluster.

#### Exercise n°1

Describe a deployment in the default namespace.

{% tabs %}
{% tab title="Command" %}
```bash
kubectl describe deployment mynginxdeploymentcli
```
{% endtab %}

{% tab title="CLI Return" %}
```bash
Name:                   mynginxdeploymentcli
Namespace:              default
CreationTimestamp:      Wed, 13 Feb 2019 13:13:10 -0500
Labels:                 app=mynginxdeploymentcli
Annotations:            deployment.kubernetes.io/revision=1
Selector:               app=mynginxdeploymentcli
Replicas:               1 desired | 1 updated | 1 total | 1 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  1 max unavailable, 1 max surge
Pod Template:
  Labels:  app=mynginxdeploymentcli
  Containers:
   nginx:
    Image:        nginx
    Port:         <none>
    Host Port:    <none>
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
OldReplicaSets:  <none>
NewReplicaSet:   mynginxdeploymentcli-547487b8d (1/1 replicas created)
Events:
  Type    Reason             Age   From                   Message
  ----    ------             ----  ----                   -------
  Normal  ScalingReplicaSet  1m    deployment-controller  Scaled up replica set mynginxdeploymentcli-547487b8d to 1

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
kubectl explain deployment.spec
```
{% endtab %}

{% tab title="CLI Return" %}
```bash
KIND:     Deployment
VERSION:  extensions/v1beta1

RESOURCE: spec <Object>

DESCRIPTION:
     Specification of the desired behavior of the Deployment.

     DeploymentSpec is the specification of the desired behavior of the
     Deployment.

FIELDS:
   minReadySeconds	<integer>
     Minimum number of seconds for which a newly created pod should be ready
     without any of its container crashing, for it to be considered available.
     Defaults to 0 (pod will be considered available as soon as it is ready)

   paused	<boolean>
     Indicates that the deployment is paused and will not be processed by the
     deployment controller.

   progressDeadlineSeconds	<integer>
     The maximum time in seconds for a deployment to make progress before it is
     considered to be failed. The deployment controller will continue to process
     failed deployments and a condition with a ProgressDeadlineExceeded reason
     will be surfaced in the deployment status. Note that progress will not be
     estimated during the time a deployment is paused. This is set to the max
     value of int32 (i.e. 2147483647) by default, which means "no deadline".

   replicas	<integer>
     Number of desired pods. This is a pointer to distinguish between explicit
     zero and not specified. Defaults to 1.

   revisionHistoryLimit	<integer>
     The number of old ReplicaSets to retain to allow rollback. This is a
     pointer to distinguish between explicit zero and not specified. This is set
     to the max value of int32 (i.e. 2147483647) by default, which means
     "retaining all old RelicaSets".

   rollbackTo	<Object>
     DEPRECATED. The config this deployment is rolling back to. Will be cleared
     after rollback is done.

   selector	<Object>
     Label selector for pods. Existing ReplicaSets whose pods are selected by
     this will be the ones affected by this deployment.

   strategy	<Object>
     The deployment strategy to use to replace existing pods with new ones.

   template	<Object> -required-
     Template describes the pods that will be created.
```
{% endtab %}
{% endtabs %}

Add the --recursive flag to display all of the fields at once without descriptions.

## Update

Users expect applications to be available all the time and developers are expected to deploy new versions of them several times a day. In Kubernetes this is done with rolling updates. **Rolling updates** allow Deployments' update to take place with zero downtime by incrementally updating Pods instances with new ones. The new Pods will be scheduled on Nodes with available resources.

Similar to application Scaling, if a Deployment is exposed publicly, the Service will load-balance the traffic only to available Pods during the update. An available Pod is an instance that is available to the users of the application.

### Rolling updates

The updates can be done based on the yaml file update or in command line depending on the kind of update needed. Remember, the command line modification are temporary and can be overwritten by an automated lifecycle manage like a Continuous Deployment tool. In production, it is highly recommended to versioned the yaml file in a git repository and ensure that each modification are tested on multiple environment before the production deployment.

The Kubernetes basic Rolling Updates architecture can be schematized like this :

![Kubernetes Rolling Updates architecture](../.gitbook/assets/rolling_update_architecture.gif)

The _rollout_ command allows to directly ask the API resource via the command line tools to manage the life cycle of a Deployment like displaying the older release with the _history_ parameter.

{% hint style="info" %}
The rolling update method does not apply only to the _Deployment_ object but to the _DaemonSet_ and the _StatefulSet_ too.
{% endhint %}

#### Exercise n°1

1. Check the history of the previous Nginx deployment
2. Describe the deployment to note the current version
3. In command line, update the Nginx version to the 1.9.1 in the previous deployment
4. Describe the deployment to ensure the version has changed
5. Check the history of the deployment to note the new entry

{% tabs %}
{% tab title="Command" %}
```bash
kubectl rollout history deployment mynginxdeploymentyaml
kubectl get deployment mynginxdeploymentyaml -o=jsonpath='{$.spec.template.spec.containers[:1].image}'
kubectl set image deployment/mynginxdeploymentyaml nginx=nginx:1.9.1
kubectl get deployment mynginxdeploymentyaml -o=jsonpath='{$.spec.template.spec.containers[:1].image}'
kubectl rollout history deployment mynginxdeploymentyaml
```
{% endtab %}

{% tab title="CLI Return" %}
```bash
deployments "mynginxdeploymentyaml"
REVISION  CHANGE-CAUSE
1         <none>
```

```bash
nginx
```

```bash
deployment.apps "mynginxdeploymentyaml" image updated
```

```bash
nginx:1.9.1
```

```bash
deployments "mynginxdeploymentyaml"
REVISION  CHANGE-CAUSE
1         <none>
2         <none>
```
{% endtab %}
{% endtabs %}

#### Exercise n°2

Do the same operation to update Nginx in the 1.12.3 version but with a yaml file and add the `--record` parameter to the command. Get the deployment history to understand the difference between an update with or without the `--record` parameter.

{% code-tabs %}
{% code-tabs-item title="~/data/deployment/03\_deployment.yaml" %}
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mynginxdeploymentyaml
  labels:
    app: mynginxdeploymentyaml
spec:
  replicas: 3
  selector:
    matchLabels:
      app: mynginxdeploymentyaml
  template:
    metadata:
      labels:
        app: mynginxdeploymentyaml
    spec:
      containers:
      - name: nginx
        image: nginx:1.12.3
```
{% endcode-tabs-item %}
{% endcode-tabs %}

Update the Deployment in command line.

{% tabs %}
{% tab title="Command" %}
```bash
kubectl apply -f ~/data/deployment/03_deployment.yaml --record
kubectl rollout history deployment mynginxdeploymentyaml
```
{% endtab %}

{% tab title="CLI Return" %}
```bash
deployments "mynginxdeploymentyaml"
REVISION  CHANGE-CAUSE
1         <none>
2         <none>
3         kubectl apply --filename=/data/deployment/02_deployment.yaml --record=true
```
{% endtab %}
{% endtabs %}

### Rollback

In a Kubernetes cluster, a Rollback mean scaling up the corresponding ReplicaSet and scaled down the current one.

This can be done easily in command line by specifying the revision to rollback the deployment.

The _rollout_ command allows to directly ask the API resource via the command line tools to manage the life cycle of a Deployment like revert to an older version with the _undo_ parameter.

{% hint style="info" %}
The rolling update method does not apply only to the _Deployment_ object but to the _DaemonSet_ and the _StatefulSet_ too.
{% endhint %}

#### Exercise n°1

Rollback the Nginx deployment to the previous revision and check that the version of Nginx is back to 1.9.1.

```bash
kubectl rollout undo deployment mynginxdeploymentyaml --to-revision=2
```

## Delete

The _delete_ command delete resources by filenames, stdin, resources and names, or by resources and label selector.

Deleting a Deployment will automatically delete each resources associated like Pods and ReplicaSet.

Note that the delete command does NOT do resource version checks, so if someone submits an update to a resource right when you submit a delete, their update will be lost along with the rest of the resource.

#### Exercise n°1

Delete the previous deployment in command line.

```bash
kubectl delete deployment mynginxdeploymentcli mynginxdeploymentyaml postgres
```

## Module exercise

The purpose of this section is to manage each steps of the lifecycle of an application to better understand each concepts of the Kubernetes course.

The main objective in this module is to understand how to manage deployment object and their relation with Pods and ReplicaSet.

For more information about the application used all along the course, please refer to the _Exercise App &gt; Voting App_ link in the left panel.

Based on the principles explain in this module, try by your own to handle this steps. The development of a yaml file is recommended.

The file developed has to be stored in this directory : ~`/data/votingapp/03_deployments`

{% tabs %}
{% tab title="Exercise" %}
1. Delete the ReplicaSet deployed in the previous module exercise
2. Develop the Deployment yaml file to deploy the Voting App microservices.
3. Update the vote Deployment to attach two environment variables :
   1. Name the first OPTION\_A, her value has to be CATS
   2. Name the second OPTION\_B, her value has to be DOGS
4. Update the db, result and worker Deployment to attach three environment variables :
   1. Name the first POSTGRES\_NAME, her value has to be "voting"
   2. Name the second POSTGRES\_USER, her value has to be "voting"
   3. Name the third POSTGRES\_PASSWORD, her value has to be "password"
5. Ensure each Pods are up and running
{% endtab %}

{% tab title="Solution" %}
Delete the previous ReplicaSet created.

```bash
kubectl delete -f ~/data/votingapp/02_replicas/replicaset.yaml
```

Example of Deployment yaml file to easily manage each part of the Voting App in a single file definition.

{% code-tabs %}
{% code-tabs-item title="~/data/votingapp/03\_deployments/deployment.yaml" %}
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
            value: "voting"
          - name: "POSTGRES_USER"
            value: "voting"
          - name: "POSTGRES_PASSWORD"
            value: "password"
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
  name: redis
  namespace: voting-app
spec:
  replicas: 1
  selector:
    matchLabels:
      name: redis
  strategy:
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 1
    type: RollingUpdate
  template:
    metadata:
      labels:
        name: redis
    spec:
      containers:
        - image: redis:alpine
          imagePullPolicy: IfNotPresent
          name: redis
          ports:
            - containerPort: 6379
              name: redis
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
            value: "voting"
          - name: "POSTGRES_USER"
            value: "voting"
          - name: "POSTGRES_PASSWORD"
            value: "password"
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
            value: "CATS"
          - name: "OPTION_B"
            value: "DOGS"
          image: wikitops/examplevotingapp-vote:1.1
          imagePullPolicy: IfNotPresent
          name: vote
          ports:
            - containerPort: 8080
              name: vote
              protocol: TCP
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
            value: "voting"
          - name: "POSTGRES_USER"
            value: "voting"
          - name: "POSTGRES_PASSWORD"
            value: "password"
          image: wikitops/examplevotingapp-worker:1.1
          imagePullPolicy: IfNotPresent
          name: worker
```
{% endcode-tabs-item %}
{% endcode-tabs %}

Run the command to create the Deployments.

```bash
kubectl apply  -f data/votingapp/03_deployments/deployment.yaml
```

Ensure the objects have been created.

```bash
kubectl get all -n voting-app
```

{% hint style="info" %}
Check the logs of each Pods to ensure that each one has started. Note that some Pods are waiting connections.
{% endhint %}
{% endtab %}
{% endtabs %}

## External documentation

Those documentations can help you to go further in this topic :

* Kubernetes official documentation on [deployment](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)
* Kubernetes official documentation on [rolling updates](https://kubernetes.io/docs/tutorials/kubernetes-basics/update/update-intro/)
* Kubernetes official documentation on [environment variable](https://kubernetes.io/docs/tasks/inject-data-application/define-environment-variable-container/) management within a container



