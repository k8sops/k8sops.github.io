---
layout: page
title: Deploying Zero Downtime
category: dtk-2.3
permalink: /dtk-2.3/deploying-zero-downtime
chapter: 3
---

The desired state of our applications is changing all the time. The most common reasons for newstates are new releases. The process is relatively simple. We make a change and commit it to a coderepository. We build it, and we test it. Once we’re confident that it works as expected, we deployit to a cluster. It does not matter whether that deployment is to a development, test, staging, orproduction environment. We need to deploy a new release to a cluster, even when that is a single-node Kubernetes running on a laptop. No matter how many environments we have, the processshould always be the same or, at least, as similar as possible.

## Kubernetes Deployment
Just as we are not supposed to create Pods directly but using other controllers like ReplicaSet, weare not supposed to create ReplicaSets either. Kubernetes Deployments will create them for us.

```
cat deploy/go-demo-2-db.yml

apiVersion: apps/v1
kind: Deployment
metadata:
  name: go-demo-2-db
spec:
  selector:
    matchLabels:
      type: db
      service: go-demo-2
  template:
    metadata:
      labels:
        type: db
        service: go-demo-2
        vendor: MongoLabs
    spec:
      containers:
      - name: db
        image: mongo:3.3
        ports:
        - containerPort: 28017

```

* RS declaration earlier for DB is same as the above. The only differnce is the ***kind***
* We will regularly add --record to the kubectl create commands. This allows us to track each change to our resources such as a Deployments.

```
kubectl create -f deploy/go-demo-2-db.yml --record

kubectl get -f deploy/go-demo-2-db.yml

deployment.apps/go-demo-2-db created
```

* Describing the deployment

```
kubectl describe -f deploy/go-demo-2-db.yml

Name:                   go-demo-2-db
Namespace:              default
CreationTimestamp:      Fri, 31 Jan 2020 03:01:25 +0000
Labels:                 <none>
Annotations:            deployment.kubernetes.io/revision: 1
                        kubernetes.io/change-cause: kubectl create --filename=deploy/go-demo-2-db.yml --record=true
Selector:               service=go-demo-2,type=db
Replicas:               1 desired | 1 updated | 1 total | 1 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  service=go-demo-2
           type=db
           vendor=MongoLabs
  Containers:
   db:
    Image:        mongo:3.3
    Port:         28017/TCP
    Host Port:    0/TCP
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
  Progressing    True    NewReplicaSetAvailable
OldReplicaSets:  <none>
NewReplicaSet:   go-demo-2-db-b449d94f (1/1 replicas created)
Events:
  Type    Reason             Age   From                   Message
  ----    ------             ----  ----                   -------
  Normal  ScalingReplicaSet  109s  deployment-controller  Scaled up replica set go-demo-2-db-b449d94f to 1

```

* Upon observing the ***Events*** section we notice the deployment created a ReplicaSet.
* Listing all the objects:

```
kubectl get all

NAME                              READY   STATUS    RESTARTS   AGE
pod/go-demo-2-db-b449d94f-zc6kj   1/1     Running   0          5m4s

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   23d

NAME                           READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/go-demo-2-db   1/1     1            1           5m4s

NAME                                    DESIRED   CURRENT   READY   AGE
replicaset.apps/go-demo-2-db-b449d94f   1         1         1       5m4s
```

* There is no difference between the use of Deployment vs ReplicaSet.

* Real advantage of Deployment is when we try to change of aspect of our deployment.

![alt text](images/deployment_stack.png "Deployment Stack")

## Sequence of Events when deploying a Deployment
1. Kubernetes client (kubectl) sent a request to the API server requesting the creation of a Deployment defined in the deploy/go-demo-2-db.yml file.
2. The deployment controller is watching the API server for new events and it detected that there is a new Deployment object.
3. The deployment controller creates a new ReplicaSet object.

![alt text](images/deployment_sequence_diagram.png "Deployment Sequence diagram")

## Updating Deployments
Upgrading the Mongo DB to 3.4 version.

```
kubectl set image -f deploy/go-demo-2-db.yml db=mongo:3.5 --record
# db is the name of the container in spec.

Events:
  Type    Reason             Age   From                   Message
  ----    ------             ----  ----                   -------
  Normal  ScalingReplicaSet  26s   deployment-controller  Scaled up replica set go-demo-2-db-84ccb8747 to 1
  Normal  ScalingReplicaSet  18s   deployment-controller  Scaled down replica set go-demo-2-db-b449d94f to 0
```
* We can see that it created a new ReplicaSet and that it scaled the old ReplicaSet to 0. If, in your case,the last line did not appear, you’ll need to wait until the new version of themongoimage is pulled. Instead of operating directly on the level of Pods, the Deployment created a new ReplicaSet which, in turn,produced Pods based on the new image.Once they became fully operational, it scaled the old ReplicaSet to 0. Since we are running a ReplicaSet with only one replica, it might not be clear why it used that strategy. When we create a Deployment for the API, things will become more evident.To be on the safe side, we might want to retrieve all the objects from the cluster.

```

kubectl get all

NAME                               READY   STATUS    RESTARTS   AGE
pod/go-demo-2-db-84ccb8747-jh2b9   1/1     Running   0          118s

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   23d

NAME                           READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/go-demo-2-db   1/1     1            1           7h40m

NAME                                     DESIRED   CURRENT   READY   AGE
replicaset.apps/go-demo-2-db-84ccb8747   1         1         1       118s
replicaset.apps/go-demo-2-db-b449d94f    0         0         0       7h40m

```

### Pod Name hash
* The Pod name is a hash which matches the hash of the ReplicaSet. 84ccb8747
* If we destroy the Deployment and re-create the name would remain the same. Since this is name is derived by hashing the PodTemplate of the ReplicaSet.
* That way Deployment would know if anything related Pods has changed in PodTemplate and if it does would create a new ReplicaSet.
* ***kubctl set image*** command has an alternative. ***kubectl edit***
```
kubectl edit -f deploy/go-demo-2-db.yml
# you’ll need to type:qfollowed by the enter key to exit
```
* ***set iamge** is the better option and also used in CI/CD process.
* Another ***alternative*** is to update the yaml file and do ***apply***

Finishing the database setup by creating the service.
```
kubectl create -f deploy/go-demo-2-db-svc.yml --record
```

## Zero Downtime Deployments
* game of 9's. Atleast 99.99 or even 99.999 percent availability.
* The reason we’re discussing failures and scalability lies in the nature of immutable deployments. If a Pod is unchangeable, the only way to update it with a new release is to destroy the old ones andput the Pods based on the new image in their place.
* Destruction of Pods is not much different from failures. In both cases, they cease to work. On the other hand, fault tolerance (re-scheduling) is areplacement of failed Pods. The only essential difference is that new releases result in Pods being replaced with new ones based on the new image.
* Deployment definition of the API:

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: go-demo-2-api
spec:
  replicas: 3
  selector:
    matchLabels:
      type: api
      service: go-demo-2
  minReadySeconds: 1
  progressDeadlineSeconds: 60
  revisionHistoryLimit: 5
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 1
  template:
    metadata:
      labels:
        type: api
        service: go-demo-2
        language: go
    spec:
      containers:
      - name: api
        image: vfarcic/go-demo-2
        env:
        - name: DB
          value: go-demo-2-db
        readinessProbe:
          httpGet:
            path: /demo/hello
            port: 8080
          periodSeconds: 1
        livenessProbe:
          httpGet:
            path: /demo/hello
            port: 8080
```

* ****minReadySeconds*** defines the minimum number of seconds before Kubernetes starts considering the Pods healthy. We put the value of this field to 1 second. The default value is 0, meaning that the Pods will be considered available as soon as they are ready and, when specified, ***livenessProbe*** returns OK. If in doubt, omit this field and leave it to the default value of 0.

* ***revisionHistoryLimit*** It defines the number of old ReplicaSets we can rollback. Like most of the fields, it is set to the sensible default value of 10. We changed it to 5 and, as a result, we will be able to rollback to any of the previous five ReplicaSets.

* The strategy can be either the ***RollingUpdate*** or the ***Recreatetype*** 
  * ***Recreatetype*** : Kill's all the existing Pods before an update. Recreate resembles the processes we used in the past when the typical strategy for deploying a new release was first to stop the existing one and then put a new one in its place. This approach inevitably leads to downtime. The only case when this strategy isuseful is when applications are not designed for two releases to coexist. Unfortunately, that is still more common than it should be.Ifyou’re in doubt whether your application is like that,ask yourself the following question. Would there be an adverse effect if two different versions of my application are running in parallel? If that’s the case, a Recreatestrategy might be a good choice and you must be aware that you cannot accomplish zero-downtime deployments.

  * ***RollingUpdate*** : the default type, for a good reason. It allows us to deploy new releases without downtime. It creates a new ReplicaSet with zero replicas and, depending on other parameters, increases the replicas of the new one, and decreases those from the old one. The process is finished when the replicas of the new ReplicaSet entirely replace those from the old one.

  * When RollingUpdate is the strategy of choice, it can be fine-tuned with the ***maxSurge*** and ***maxUnavailable*** fields.The former defines the maximum number of Pods that can exceed the desired number (set using replicas). It can be set to an absolute number (e.g.,2) or a percentage (e.g.,35%). The total number of Pods will never exceed the desired number (set using replicas) and the maxSurge combined. The default value is 25%. maxUnavailable defines the maximum number of Pods that are not operational. If, for example, the number of replicas is set to 15 and this field is set to 4, the minimum number of Pods that would run at any given moment would be 11. Just as the maxSurge field, this one also defaults to 25%. If this field is not specified, there will always be at least 75% of the desired Pods. In most cases, the default values of the Deployment specific fields are a good option.

  * The ***template*** is the same PodTemplate we used before.Best practice is to be explicit with image tags like we did when we set mongo:3.3. Best option would have been to say something like ***vfarcic/go-demo-2:1.0***
  It is okay for initial cluster setup.

  ```
  kubectl create -f deploy/go-demo-2-api.yml --record
  deployment.apps/go-demo-2-api created

  kubectl get all
  
  NAME                                 READY   STATUS    RESTARTS   AGE
  pod/go-demo-2-api-86469df75d-sq6mv   1/1     Running   0          48s
  pod/go-demo-2-api-86469df75d-trbd9   1/1     Running   0          48s
  pod/go-demo-2-api-86469df75d-vlknk   1/1     Running   0          48s
  pod/go-demo-2-db-84ccb8747-jh2b9     1/1     Running   1          4h3m

  NAME                   TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)     AGE
  service/go-demo-2-db   ClusterIP   10.109.243.89   <none>        27017/TCP   3h28m
  service/kubernetes     ClusterIP   10.96.0.1       <none>        443/TCP     24d

  NAME                            READY   UP-TO-DATE   AVAILABLE   AGE
  deployment.apps/go-demo-2-api   3/3     3            3           48s
  deployment.apps/go-demo-2-db    1/1     1            1           11h

  NAME                                       DESIRED   CURRENT   READY   AGE
  replicaset.apps/go-demo-2-api-86469df75d   3         3         3       48s
  replicaset.apps/go-demo-2-db-84ccb8747     1         1         1       4h3m
  replicaset.apps/go-demo-2-db-b449d94f      0         0         0       11h
  ```

  * Now the we have 3 Pods of API running we will roll out changes using a newer version of image.

  ```
  kubectl set image -f deploy/go-demo-2-api.yml api=vfarcic/go-demo-2:2.0 --record
  deployment.apps/go-demo-2-api image updated

  kubectl rollout status -w -f deploy/go-demo-2-api.yml
  deployment "go-demo-2-api" successfully rolled out

  kubectl describe -f deploy/go-demo-2-api.yml

  Name:                   go-demo-2-api
  Namespace:              default
  CreationTimestamp:      Fri, 31 Jan 2020 14:42:29 +0000
  Labels:                 <none>
  Annotations:            deployment.kubernetes.io/revision: 2
                          kubernetes.io/change-cause: kubectl set image api=vfarcic/go-demo-2:2.0 --filename=deploy/go-demo-2-api.yml --record=true
  Selector:               service=go-demo-2,type=api
  Replicas:               3 desired | 3 updated | 3 total | 3 available | 0 unavailable
  StrategyType:           RollingUpdate
  MinReadySeconds:        1
  RollingUpdateStrategy:  1 max unavailable, 1 max surge
  Pod Template:
    Labels:  language=go
            service=go-demo-2
            type=api
    Containers:
    api:
      Image:      vfarcic/go-demo-2:2.0
      Port:       <none>
      Host Port:  <none>
      Liveness:   http-get http://:8080/demo/hello delay=0s timeout=1s period=10s #success=1 #failure=3
      Readiness:  http-get http://:8080/demo/hello delay=0s timeout=1s period=1s #success=1 #failure=3
      Environment:
        DB:    go-demo-2-db
      Mounts:  <none>
    Volumes:   <none>
  Conditions:
    Type           Status  Reason
    ----           ------  ------
    Available      True    MinimumReplicasAvailable
    Progressing    True    NewReplicaSetAvailable
  OldReplicaSets:  <none>
  NewReplicaSet:   go-demo-2-api-6c8867677b (3/3 replicas created)
  Events:
    Type    Reason             Age    From                   Message
    ----    ------             ----   ----                   -------
    Normal  ScalingReplicaSet  7m38s  deployment-controller  Scaled up replica set go-demo-2-api-86469df75d to 3
    Normal  ScalingReplicaSet  2m     deployment-controller  Scaled up replica set go-demo-2-api-6c8867677b to 1
    Normal  ScalingReplicaSet  2m     deployment-controller  Scaled down replica set go-demo-2-api-86469df75d to 2
    Normal  ScalingReplicaSet  2m     deployment-controller  Scaled up replica set go-demo-2-api-6c8867677b to 2
    Normal  ScalingReplicaSet  116s   deployment-controller  Scaled down replica set go-demo-2-api-86469df75d to 0
    Normal  ScalingReplicaSet  116s   deployment-controller  Scaled up replica set go-demo-2-api-6c8867677b to 3
  ```
### How scale up & down transpired
We can see that the number of desired replicas is 3. The same number was updated and all are available. At the bottom of the output are events associated with the Deployment. The process started by increasing the number of replicas of the new ReplicaSet (go-demo-2-api-86469df75d) to 1. Next, it decreased the number of replicas of the old ReplicaSet (go-demo-2-api-6c8867677b) to2. The same process of increasing replicas of the new, and decreasing replicas of the old ReplicaSet continued until the new one got the desired number (3), and the old one dropped to zero.There was no downtime throughout the process.Users would receive a response from the application no matter whether they sent it before, during, or after the update. The only important thing is that,during the update, a response might  have come from the old or the new release. During the update process, both releases were running in parallel.

