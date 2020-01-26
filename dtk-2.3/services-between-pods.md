---
layout: page
title: Services Between Pods
category: dtk-2.3
permalink: /dtk-2.3/services-bwtween-pods
chapter: 2
---

# Using Services for Pod communication
* Pods are born and they die. Never healed. New ones added and surplus ones killed.
* Controllers together with other components like Scheduler make sure that the Pods are doing the right thing. Controllers control the schedulers.
* Previous setup in [Scaling with Pods](/dtk-2.3/scaling-with-pods), had a issue. Both the API and DB containers were in the same Pod. That restricts scaling. One cannot scale without the other. And communication was happening via localhost.
* To split them up, will have to take care of communication.
* Need for Services:
    * We can spilt the API and DB into separate Pods.
    * Pods get their own IP address. But since Pods are short lived, their IP adress also cannot be relied on.
    * We need a stable never to be changed IP adress that forwards the requests to whichever Pod is currently running and can take the request.
    * ***Services*** fulfill that requirement

```
    - name: db
        image: mongo:3.3
        command: ["mongod"]
        args: ["--rest","--httpinterface"]
        ports:
            - containerPort: 28017
            protocol: TCP
```

* We customized the command and the arguments so that the Mongo DB exposes the REST interface. We also defined the containerPort. This will allow us to access the database through a service.

* Now using ***kubectl expose*** command we will expose the ReplicaSet as a Kubenetes service.
Other Kubernetes objects like Deployment, another Service, Pods can also be exposed in a similar way.

```
kubectl expose rs go-demo-2 \
--name=go-demo-2-svc \
--target-port=28017 \
--type=NodePort
```

* We are exposing a ReplicaSet named ***go-demo-2*** and createing the service with name ***go-demo-2-svc*** , port to be exposed is ***28017*** since MongoDb would be listening to it. Type would be ***NodePort***, which means this port would be exposed on every node in the cluster and the request would be routed to one of the Pods controlled by the ReplicaSet.

* Other types of Services:
    * ClusterIP (default type): Exposes port only within the cluster. No external access.
    When we created NodePort, ClusterIP was also created.
    * LoadBalancer: This used with cloud providers load balancer.

