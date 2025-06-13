## Service in kubernetes
- The pods that gets created in the worker node have a private IP associated with them.
- Pod in the same cluster can communicate with other pods.
- Containers in same pod can communicate with each other.
- If the IP address of the backend pod is hardcoded in the frontend pod, it creates issues since pods can be ephemeral.
- port: port of the service
- target-port: port of the application running in container/pod

- Service: It acts as gateway that distibutes trafic to backend pods.
- Endpoint: It contains the IP address of the backend pods.
   - We need to manually assign IP of the backend pods to endpoints.
- The service by default sends request to endpoints not the pods.
- To find the pods the traffic needs to be sent use selectors and labels and not endpoints.
- If using endpoints, its name should be same as service.

- With selector, no need to worry about how many backend pods added or removed.
- Using service, users outside of cluster can also connect to pods internal to cluster. (Exposing pod).

Benfits: -
 Pods are ephemeral and their IPs change frequently. Services provide a stable endpoint.
 Service distribute traffic across multiple pod replicas.
 Service enables exposing applications to external traffic like the internet.

 Types of services
  - Clusterip (default) for communication between service and backend pods withing cluster.
  - NodePort : To enable external communucation using a stable port (Range:30000-32767) and worker node IP.
        - A port gets opened in all workernodes.
  - LoadBalancer: The service automatically creates loadbalancer with external ip for managed kubernetes
       - For on-premise additional configuration for a loadbalancer is required.
       - Loadbalancer IP is used to communicate with service instead of worker node IP.
  - Externalname: Maps service to external DNS name.
  - Endpoint: It contains the address of the underlying pods to which the service will route the traffic.


Challange with Nodeport:
- To access nodeport based service, we need to make a request to IP of worker node:NodePort.
Solution is loadbalancer service: This type of service creats an external load balancer in a cloud provider and routes the request received in load balancer to underlying nodeport.
- Eliminates the need of specifying worker node IP and nodeport to access the underlying service, can make use of external IP of loadbalancer to access the backend service.
- Traffic from service is forwarded to appropriate pods.
- Loadbalancer internally uses nodeport to access the service. Nodeport port gets enabled in all worker nodes of the cluster.
```
Commands:
kubectl get service
kubectl get svc
kubectl create service clusterip my-service --tcp=80:80
kubectl create service nodeport my-service --tcp=80:80  # Automatically enables a port on all workder nodes.
kubectl create service loadbalancer my-service --tcp-80:80 #Automatically creates external loadbalancer.
kubectl expose po/chakra --name=my-service --port=80 --target-port=80 --type=loadbalancer # it creates a service and assigns it to the pod.
```

# Ingress
- Challenges with loadbalancer type service
   - If you have multiple services to expose multiple applications running as pods/containers in the cluster.
   - Multiple loadbalancers needs to be created which is expensive.
   - One loadbalancer supports only one service.

  - Ingress: It acts as entrypoint that routes traffic to specific services based on rules we define.
  - Ingress components: 1) Ingress resource 2) Ingress controllers.
                                                      Example-service -> pod1
      Controller-------->
                                                      kplabs-service -> pod2
      Rules         forward                          
      aks-internal example-service
      aws-internal kplabs-service
       
When we create ingress it automatically creates loadbalancer.
```
kubectl create ingress <name> --rule=host/path=service:port
kubectl create ingress chakra --rule=example.internet/*=example-service:80
kubectl descibe ingress/my-ingress
```

## Ingress routing patterns
1) Name based virtual hosting
- Traffic is routed based on the host header in the http request.
     Ingress
  Name based          Virtual hosting
  host:example-internal    host:kplabs-internal
  service:example-service  Service: kplabs-service

  2) Path based routing
     Traffic is routed based on the URL path in the http request.
     /app1            /app2  and host=*
     
     






 
  
