---
layout: page
title: Rollingback or Forward
category: dtk-2.3
permalink: /dtk-2.3/rollback-or-forward
chapter: 5
---

# Rollingback or Forward

* Rolling back a release that introduced database changes is often not possible. Even when itis, rolling forward is usually a better option when practicing continuous deployment withhigh-frequency releases limited to a small scope of changes.
* If you do frequent releases then rolling back a few hours of work can be the best option. It depends from project to project.

```
// invoking the rollback command
kubectl rollout undo -f deploy/go-demo-2-api.yml 
deployment.apps/go-demo-2-api rolled back

// describing to check the events that transpired
kubectl describe -f deploy/go-demo-2-api.yml 
Name:                   go-demo-2-api
Namespace:              default
CreationTimestamp:      Fri, 31 Jan 2020 14:42:29 +0000
Labels:                 <none>
Annotations:            deployment.kubernetes.io/revision: 3
                        kubernetes.io/change-cause: kubectl create --filename=deploy/go-demo-2-api.yml --record=true
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
    Image:      vfarcic/go-demo-2
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
NewReplicaSet:   go-demo-2-api-86469df75d (3/3 replicas created)
Events:
  Type    Reason             Age   From                   Message
  ----    ------             ----  ----                   -------
  Normal  ScalingReplicaSet  70s   deployment-controller  Scaled up replica set go-demo-2-api-86469df75d to 1
  Normal  ScalingReplicaSet  70s   deployment-controller  Scaled down replica set go-demo-2-api-6c8867677b to 2
  Normal  ScalingReplicaSet  70s   deployment-controller  Scaled up replica set go-demo-2-api-86469df75d to 2
  Normal  ScalingReplicaSet  66s   deployment-controller  Scaled down replica set go-demo-2-api-6c8867677b to 1
  Normal  ScalingReplicaSet  66s   deployment-controller  Scaled up replica set go-demo-2-api-86469df75d to 3
  Normal  ScalingReplicaSet  65s   deployment-controller  Scaled down replica set go-demo-2-api-6c8867677b to 0
```
* We can see from the events section that the Deployment initiated rollback and, from there on, the process we experienced before was reversed. It started increasing the replicas of the older ReplicaSet,and decreasing those from the latest one. Once the process is finished, the older ReplicaSet became active with all the replicas, and the newer one was scaled down to zero.
* Checking the revision history now:
```
kubectl rollout history -f deploy/go-demo-2-api.yml
deployment.apps/go-demo-2-api

REVISION  CHANGE-CAUSE
2         kubectl set image api=vfarcic/go-demo-2:2.0 --filename=deploy/go-demo-2-api.yml --record=true
3         kubectl create --filename=deploy/go-demo-2-api.yml --record=true
```

## Undo Rollouts
* There are scenarios where rolling forward is not an option. In those cases we can rollback using the ***rollout undo*** command.

```
kubectl rollout undo -f deploy/go-demo-2-api.yml

kubectl describe deploy/go-demo-2-api.yml
```
* Looking at the events we see that the process started increasing the replicas of the older ReplicaSet and decreasing those from the latest ones.


***Past rollouts:***
```
kubectl rollout history -f deploy/go-demo-2-api.yml 

deployment.apps/go-demo-2-api 
REVISION  CHANGE-CAUSE
2         kubectl set image api=vfarcic/go-demo-2:2.0 --filename=deploy/go-demo-2-api.yml --record=true
3         kubectl create --filename=deploy/go-demo-2-api.yml --record=true
```

***Going back to a certain revision:***
If we were do rollout out few more versions of the code, there can be a use case to rollback to a certain revision in the rollout history.
```
kubectl set image -f deploy/go-demo-2-api.yml api=vfarcic/go-demo-2:3.0 --record

kubectl rollout status -f deploy/go-demo-2-api.yml 
deployment "go-demo-2-api" successfully rolled out

kubectl set image -f deploy/go-demo-2-api.yml api=vfarcic/go-demo-2:4.0 --record
deployment.apps/go-demo-2-api image updated

kubectl rollout history -f deploy/go-demo-2-api.yml 
deployment.apps/go-demo-2-api 
REVISION  CHANGE-CAUSE
2         kubectl set image api=vfarcic/go-demo-2:2.0 --filename=deploy/go-demo-2-api.yml --record=true
3         kubectl create --filename=deploy/go-demo-2-api.yml --record=true
4         kubectl set image api=vfarcic/go-demo-2:3.0 --filename=deploy/go-demo-2-api.yml --record=true
5         kubectl set image api=vfarcic/go-demo-2:4.0 --filename=deploy/go-demo-2-api.yml --record=true
```

* Now say we would like go back to revision #2, which is the image tag version 2.0

```
kubectl rollout undo -f deploy/go-demo-2-api.yml --to-revision 2
deployment.apps/go-demo-2-api rolled back

kubectl rollout history -f deploy/go-demo-2-api.yml 
deployment.apps/go-demo-2-api 
REVISION  CHANGE-CAUSE
3         kubectl create --filename=deploy/go-demo-2-api.yml --record=true
4         kubectl set image api=vfarcic/go-demo-2:3.0 --filename=deploy/go-demo-2-api.yml --record=true
5         kubectl set image api=vfarcic/go-demo-2:4.0 --filename=deploy/go-demo-2-api.yml --record=true
6         kubectl set image api=vfarcic/go-demo-2:2.0 --filename=deploy/go-demo-2-api.yml --record=true
```

* revision #6 is now tag version 2.0 and, ***if this was the “real” application running in a production cluster, our users wouldcontinue interacting with the version of our software that actually works.***

## Rolling back failed deployments
Other than a bad deployment there are other cases that needs a rollback. Cases like deployment failures, where say Pods didn't get created.

```
kubectl set image -f deploy/go-demo-2-api.yml api=vfarcic/go-demo-2:does-not-exists --record
deployment.apps/go-demo-2-api image updated


cloud_user@anirban1c:~/k8s-specs$ kubectl get pods -l type=api
NAME                             READY   STATUS         RESTARTS   AGE
go-demo-2-api-586448d5dd-gxksl   0/1     ErrImagePull   0          58s
go-demo-2-api-586448d5dd-rft67   0/1     ErrImagePull   0          58s
go-demo-2-api-6c8867677b-5zj5n   1/1     Running        1          4h27m
go-demo-2-api-6c8867677b-t5dnb   1/1     Running        1          4h27m

kubectl rollout status -f deploy/go-demo-2-api.yml 
error: deployment "go-demo-2-api" exceeded its progress deadline
```

But in real pipeline we would like to automate the process of detecting a failed deployment other than waiting for time out and checking ***status***.

Fortunately the ***status*** command return a non-zero return.
```
echo $?
1

kubectl rollout undo -f deploy/go-demo-2-api.yml 
deployment.apps/go-demo-2-api rolled back

kubectl get pods -l type=apiNAME                             READY   STATUS    RESTARTS   AGE
go-demo-2-api-6c8867677b-5zj5n   1/1     Running   1          4h35m
go-demo-2-api-6c8867677b-q7jb5   1/1     Running   0          9s
go-demo-2-api-6c8867677b-t5dnb   1/1     Running   1          4h35m
```

The ***undo*** corrected the situation now.

## Merging definitions in a Single file


```
cat deploy/go-demo-2.yml 
apiVersion: apps/v1
kind: Deployment
metadata:
  name: go-demo-2-db
  labels:
    type: db
    service: go-demo-2
    vendor: MongoLabs
spec:
  selector:
    matchLabels:
      type: db
      service: go-demo-2
  strategy:
    type: Recreate
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

---

apiVersion: v1
kind: Service
metadata:
  name: go-demo-2-db
spec:
  ports:
  - port: 27017
  selector:
    type: db
    service: go-demo-2

---

apiVersion: apps/v1
kind: Deployment
metadata:
  name: go-demo-2-api
  labels:
    type: api
    service: go-demo-2
    language: go
spec:
  replicas: 3
  selector:
    matchLabels:
      type: api
      service: go-demo-2
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

---

apiVersion: v1
kind: Service
metadata:
  name: go-demo-2-api
spec:
  type: NodePort
  ports:
  - port: 8080
  selector:
    type: api
    service: go-demo-2
```

```
kubectl create -f deploy/go-demo-2.yml --record --save-config
deployment.apps/go-demo-2-db created
service/go-demo-2-db created
deployment.apps/go-demo-2-api created
service/go-demo-2-api created
```

All four objects(two Deployments and two Services) were created, and we can move on and explore ways to update multiple objects with a single command.


## Updating Multiple Objects
Even though most of the time we send requests to specific objects, almost everything is happeningusing selector labels. When we updated the Deployments, they looked for matching selectors tochoose which ReplicaSets to create and scale. They, in turn, created or terminated Pods also using the matching selectors. Almost everything in Kubernetes is operated using label selectors. It’s just that sometimes that is obscured from us.

We do not have to update an object only by specifying its name or the YAML file where its definition resides. We can also use labels to decide which object should be updated. That opens some interesting possibilities since the selectors might match multiple objects.

Say we have another deployment of Mongo DB, with the same labels.

```
cat deploy/different-app-db.yml 
apiVersion: apps/v1
kind: Deployment
metadata:
  name: different-app-db
  labels:
    type: db
    service: different-app
    vendor: MongoLabs
spec:
  selector:
    matchLabels:
      type: db
      service: different-app
  template:
    metadata:
      labels:
        type: db
        service: different-app
        vendor: MongoLabs
    spec:
      containers:
      - name: db
        image: mongo:3.3
        ports:
        - containerPort: 28017
```
When compared with the go-demo-2-db Deployment, the only difference is in the ***service*** label. Both have the ***type*** set to db

```
kubectl create -f deploy/different-app-db.yml 
deployment.apps/different-app-db created
```

Since we want to update the Deployments of DB, will use the below:
```
kubectl get deployment --show-labels

NAME               READY   UP-TO-DATE   AVAILABLE   AGE   LABELS
different-app-db   1/1     1            1           69s   service=different-app,type=db,vendor=MongoLabs
go-demo-2-api      3/3     3            3           63m   language=go,service=go-demo-2,type=api
go-demo-2-db       1/1     1            1           63m   service=go-demo-2,type=db,vendor=MongoLabs

kubectl get deployments -l type=db,vendor=MongoLabs

NAME               READY   UP-TO-DATE   AVAILABLE   AGE
different-app-db   1/1     1            1           115m
go-demo-2-db       1/1     1            1           178m
```

By using the label filter (-l) we were able to filter Pods for Mongo Db's
```
kubectl set image deployments -l type=db,vendor=MongoLabs db=mongo:3.4 --record
deployment.extensions/different-app-db image updated
deployment.extensions/go-demo-2-db image updated
```
Shows that two Deployments were updated. Can be verified using ***describe***
```
kubectl describe -f deploy/go-demo-2.yml
Containers:
   db:
    Image:        mongo:3.4
```

