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
    * ***Services*** fulfill that requirement:

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
* Events that happens with Service:
    1. Kubernetes client (kubectl) sent a request to the API server requesting the creation of the Service based on Pods created through the go-demo-2 ReplicaSet.
    2. Endpoint controller is watching the API server for new service events. It detected that there is a new Service object.
    3. Endpoint controller created endpoint objects with the same name as the Service, and it used Service selector to identify endpoints (in this case the IP and the port of go-demo-2 Pods).
    4. kube-proxy is watching for service and endpoint objects. It detected that there is a new Service and a new endpoint object.
    5. kube-proxy added iptables rules which capture traffic to the Service port and redirect it to endpoints. For each endpoint object, it adds iptables rule which selects a Pod.
    6. The kube-dns add-on is watching for Service. It detected that there is a new service.
    7. The kube-dns added db’s record to the dns server (skydns).

![alt text](images/services-events.png "services events")
![alt text](images/services-cluster-view.png "services cluster view")

```
kubectl describe svc go-demo-2-svc

Name:                     go-demo-2-svc
Namespace:                default
Labels:                   <none>
Annotations:              <none>
Selector:                 service=go-demo-2,type=backend
Type:                     NodePort
IP:                       10.101.135.25
Port:                     <unset>  28017/TCP
TargetPort:               28017/TCP
NodePort:                 <unset>  30326/TCP
Endpoints:                10.244.1.10:28017,10.244.2.8:28017
Session Affinity:         None
External Traffic Policy:  Cluster
Events:                   <none>
```

* Since the Service is associated with the Pods created through the ReplicaSet, it inherited all their labels. The selector matches the one from the ReplicaSet. The Service is not directly associated with the ReplicaSet (or any other controller) but with Pods through matching labels.

```
PORT=$(kubectl get svc go-demo-2-svc -ojsonpath="{.spec.ports[0].nodePort}")
```

* Next is the Node Port type which exposes ports to all the nodes. Since Node Port automatically created Cluster IP type as well,all the Pods in the cluster can access the TargetPort. The Port is set to 28017.That is the port that the Pods can use to access the Service. Since we did not specify it explicitly when we executed the command, its value is the same as the value of theTargetPort, which is the portof the associated Pod that will receive all the requests.NodePortwas generated automatically sincewe did not set it explicitly. It is the port which we can use to access the Service and, therefore, thePods from outside the cluster. In most cases, it should be randomly generated, that way we avoid any clashes.


