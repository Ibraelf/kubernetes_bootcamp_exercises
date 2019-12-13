# High Availability

## Module

_Healthchecks_ are a way of checking the health of some resource.

#### Overview

At the end of this module, you will :

* _Learn to monitor resources_
* _Learn to format it in a declarative mode_
* _Learn the different type of scalability_

#### Prerequisites

Create the directory `data/highavailability` in your home folder to manage the YAML file needed in this module.

```bash
mkdir ~/data/highavailability
```

## Liveness / Readiness Probe

Many applications running for long periods of time eventually transition to broken states, and cannot recover except by being restarted. Kubernetes provides _**liveness**_ probes to detect and remedy such situations.

Sometimes, applications are temporarily unable to serve traffic. For example, an application might need to load large data or configuration files during startup, or depend on external services after startup. In such cases, the Pods must not be killed and no data has to be send. Kubernetes provides _**readiness**_ probes to detect and mitigate these situations. A pod with containers reporting that they are not ready does not receive traffic through Kubernetes Services.

Readiness probes are configured similarly to liveness probes.

### Command Check

This health check uses a command to attempt to get command return status. If the probe get a response from the specific path, the container is considered healthy, if it can't it is considered a failure. 

#### Exercise n°1

Create an Pods based on the busybox Docker image to run a command and check his liveness.

{% code-tabs %}
{% code-tabs-item title="~/data/highavailability/01\_pods.yaml" %}
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: liveness-exec
spec:
  containers:
  - name: liveness
    image: k8s.gcr.io/busybox
    args:
    - /bin/sh
    - -c
    - touch /tmp/healthy; sleep 30; rm -rf /tmp/healthy; sleep 600
    livenessProbe:
      exec:
        command:
        - cat
        - /tmp/healthy
      initialDelaySeconds: 5
      periodSeconds: 5
    readinessProbe:
      exec:
        command:
        - cat
        - /tmp/healthy
      initialDelaySeconds: 5
      periodSeconds: 5
```
{% endcode-tabs-item %}
{% endcode-tabs %}

Create a resource based on the previous yaml file definition.

```bash
kubectl create -f ~/data/highavailability/01_pods.yaml
```

### HTTP Request

This health check uses HTTP Request to attempt to get a website path on a specified port. If the probe get a response from the specific path, the container is considered healthy, if it can't it is considered a failure. 

#### Exercise n°1

Create a Pods based on nginx Docker image and configure his liveness and readiness to check the HTTP response of the base path of the default nginx website on port 80.

{% code-tabs %}
{% code-tabs-item title="~/data/highavailability/02\_pods.yaml" %}
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: liveness-http
spec:
  containers:
  - name: nginx
    image: nginx
    livenessProbe:
      httpGet:
        path: /
        port: 80
      initialDelaySeconds: 3
      periodSeconds: 3
    readinessProbe:
      httpGet:
        path: /
        port: 80
      initialDelaySeconds: 3
      periodSeconds: 3
```
{% endcode-tabs-item %}
{% endcode-tabs %}

Create a resource based on the previous yaml file definition.

```bash
kubectl create -f ~/data/highavailability/02_pods.yaml
```

### TCP Request

This healthcheck uses TCP Socket to attempt to open a socket to a container on a specified port. If a connection is established, the container is considered healthy, if it can't it is considered a failure. 

#### Exercise n°1

Create a Pods based on nginx Docker image and configure his liveness and readiness to check the TCP response of the base path of the default nginx website on port 80.

{% code-tabs %}
{% code-tabs-item title="~/data/highavailability/03\_pods.yaml" %}
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: liveness-tcp
spec:
  containers:
  - name: nginx
    image: nginx
    livenessProbe:
      tcpSocket:
        port: 80
      initialDelaySeconds: 15
      periodSeconds: 20
    readinessProbe:
      tcpSocket:
        port: 80
      initialDelaySeconds: 5
      periodSeconds: 10
```
{% endcode-tabs-item %}
{% endcode-tabs %}

Create a resource based on the previous yaml file definition.

```bash
kubectl create -f ~/data/highavailability/03_pods.yaml
```

## Module exercise

The purpose of this section is to manage each steps of the lifecycle of an application to better understand each concepts of the Kubernetes course.

The main objective in this module is to understand how to make everything highly available on a Kubernetes cluster.

For more information about the application used all along the course, please refer to the _Exercise App &gt; Voting App_ link in the left panel.

Based on the principles explain in this module, try by your own to handle this steps. The development of a yaml file is recommended.

The file developed has to be stored in this directory : `~/data/votingapp/13_highavailability`

{% tabs %}
{% tab title="Exercise" %}
1. Update the database Deployment to :
   1. Ensure the liveness of the PostgreSQL service based on port 5432
   2. Ensure the readiness of the PostgreSQL service
2. Update the redis Deployment to : 
   1. Ensure the liveness of the Redis service based on port 6379
   2. Ensure the readiness of the Redis service
3. Update the result Deployment to :
   1. Ensure the liveness of the application based on the HTTP get of path /
4. Update the vote Deployment to :
   1. Ensure the liveness of the application based on the HTTP get of path /
{% endtab %}

{% tab title="Solution" %}
Create 2 PersistentVolumes used after by redis and db.

{% code-tabs %}
{% code-tabs-item title="~/data/votingapp/13\_highavailability/persistentvolume\_1g\_a.yaml" %}
```yaml
kind: PersistentVolume
apiVersion: v1
metadata:
  name: persistentvolume-1g-a
  namespace: voting-app
  labels:
    type: local
spec:
  storageClassName: manual
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "~/data/votingapp/13_highavailability/pv-1g-a/data"
```
{% endcode-tabs-item %}
{% endcode-tabs %}

{% code-tabs %}
{% code-tabs-item title="~/data/votingapp/13\_highavailability/persistentvolume\_1g\_b.yaml" %}
```yaml
kind: PersistentVolume
apiVersion: v1
metadata:
  name: persistentvolume-1g-b
  namespace: voting-app
  labels:
    type: local
spec:
  storageClassName: manual
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "~/data/votingapp/13_highavailability/pv-1g-b/data"
```
{% endcode-tabs-item %}
{% endcode-tabs %}

Create 2 PersistentVolumeClaims for db and redis based on the 2 previous PersistentVolumes.

{% code-tabs %}
{% code-tabs-item title="~/data/votingapp/13\_highavailability/persistentvolumeclaim\_db.yaml" %}
```yaml
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: db
  namespace: voting-app
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

{% code-tabs %}
{% code-tabs-item title="~/data/votingapp/13\_highavailability/persistentvolumeclaim\_redis.yaml" %}
```yaml
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: redis
  namespace: voting-app
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

Update the database Deployment yaml file definition.

{% code-tabs %}
{% code-tabs-item title="~/data/votingapp/13\_highavailability/deployment\_database.yaml" %}
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
                configMapKeyRef:
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
          livenessProbe:
            tcpSocket:
              port: db
            initialDelaySeconds: 15
            periodSeconds: 20
          name: db
          ports:
            - name: db
              containerPort: 5432
              protocol: TCP
          readinessProbe:
            tcpSocket:
              port: db
            initialDelaySeconds: 5
            periodSeconds: 10
          volumeMounts:
            - mountPath: /var/lib/postgresql
              name: db-data
      dnsPolicy: ClusterFirst
      restartPolicy: Always
      terminationGracePeriodSeconds: 30
      volumes:
        - name: db-data
          persistentVolumeClaim:
            claimName: db
```
{% endcode-tabs-item %}
{% endcode-tabs %}

Update the redis Deployment yaml file definition.

{% code-tabs %}
{% code-tabs-item title="~/data/votingapp/13\_highavailability/deployment\_redis.yaml" %}
```yaml
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
          livenessProbe:
            tcpSocket:
              port: redis
            initialDelaySeconds: 15
            periodSeconds: 20
          name: redis
          ports:
            - containerPort: 6379
              name: redis
              protocol: TCP
          readinessProbe:
            tcpSocket:
              port: redis
            initialDelaySeconds: 5
            periodSeconds: 20
          volumeMounts:
            - mountPath: /data
              name: redis-data
      dnsPolicy: ClusterFirst
      restartPolicy: Always
      terminationGracePeriodSeconds: 30
      volumes:
        - name: redis-data
          persistentVolumeClaim:
            claimName: redis
```
{% endcode-tabs-item %}
{% endcode-tabs %}

Update the vote Deployment yaml file definition.

{% code-tabs %}
{% code-tabs-item title="~/data/votingapp/13\_highavailability/deployment\_vote.yaml" %}
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
          livenessProbe:
            httpGet:
              path: /
              port: vote
            initialDelaySeconds: 3
            periodSeconds: 3
          name: vote
          ports:
            - containerPort: 8080
              name: vote
              protocol: TCP
```
{% endcode-tabs-item %}
{% endcode-tabs %}

Update the result Deployment yaml file definition.

{% code-tabs %}
{% code-tabs-item title="~/data/votingapp/13\_highavailability/deployment\_result.yaml" %}
```yaml
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
                  key: database-name
                  name: db
            - name: "POSTGRES_USERNAME"
              valueFrom:
                secretKeyRef:
                  key: database-user
                  name: db
            - name: "POSTGRES_PASSWORD"
              valueFrom:
                secretKeyRef:
                  key: database-password
                  name: db
            - name: "PORT"
              value: "8080"
          image: wikitops/examplevotingapp-result:1.1
          imagePullPolicy: IfNotPresent
          livenessProbe:
            httpGet:
              path: /
              port: result
            initialDelaySeconds: 3
            periodSeconds: 3
          name: result
          ports:
            - name: result
              containerPort: 8080
```
{% endcode-tabs-item %}
{% endcode-tabs %}

Create PV and PVC needed and update the each Deployment based on the previous yaml files.

```bash
kubectl apply -f /data/votingapp/13_highavailability/
```
{% endtab %}
{% endtabs %}

## External documentation

Those documentations can help you to go further in this topic :

* [Kubernetes official documentation](https://kubernetes.io/docs/setup/cluster-large/) on building a large cluster
* [Kubernetes official documentation](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-probes/) to configure readiness and liveness probe

