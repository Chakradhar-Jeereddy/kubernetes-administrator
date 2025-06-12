## Deployments
- It make use of replicasets(wrapper around replicaset) to create managed pods.
- Example: Camera is replicaset and lense is deployment. Lense enhences and extends the features of the camera.

## Benifits
- Rolling updates/upgrades
- Versioning and inbuilt rollback

## Rolling update:
- Create three pods using deployment with name nginx-pod-x and with image nginx
- Update the image to apache and apply using kubectl command
```
kubectl apply -f deployment.yaml
kubectl get rs
kubectl get pods -w
```
- It first deploy new replicaset with new pods with new configuration and then terminates the existing configuration.

## Rollout history
- Deployment allows us to inspect the history of out deployments
```
kubectl rollout history deploy/ngnix-deployment.
Revision  change-cause
1
2
kubectl get rs
Old replica sets are maintained.
```
## Comparision between replicaset and deployment
```
Features               Replicaset           Deployment
Abstraction-level      Lower-level          Higher-level
Primary function       Ensure a specific    Manage replicasets and handle updates
                       number of replicas
                       are running
Rolling updates        Not supported        Suported based on configuration stratergies
Rollback               Not supported        Supported with version history
Usecases               Simple static        Dynamic workload with frequent changes
                       workloads
```
```
- Use case:
  kubectl create deployment --image=nginx
  kubectl get deploy
  kubectl get po
  kubectl get rs
- Create manifest file
  kubectl create deployment nginx-deployment --image=nginx --dryrun=client -o yaml > deploy.yaml
  kubectl apply -f deploy.yaml
  kubectl rollout history deploy/nginx-deployment
  Revision        Change cause
    1                <None>
  kubectl set --help
  kubectl set image deployment/nginx-deployment nginx=httpd:latest
  kubectl get pods -w
  kubectl rollout history deploy/nginx-deployment
  Revision        Change cause
    1                <None>
    2                <None>
  kubectl get rs
  kubectl describe rs <old>
    image=nginx
  kubectl describe rs <new>
    image=httpd:latest
  kubectl rollout undo deployment nginx-deployment
  kubectl rollout undo deployment --help
  kubectl rollout undo deployment nginx-deployment --to-revision=2
  kubectl scale --replicas=3 deploy/nginx-deployment
  kubectl delta deployment nginx-deployment
  ```
## Multi worker nodes or kubernetes
- Challange 1
   - A single worker node might not have enough resources (CPU, Memory) to handle the load of all the pods.
         Control plane ->  Node1/workernode
  - Adding more worker nodes allows you to distribute workload accross the worker nods.
  - Control plane [node1, node2, node3]
  - Benifit: Fault tollerance and load balancing
  - If one worker node fails, the application running on it will become unavailable.
  - With multiple worker nodes, kubernetes can reschedule those applications onto healthy nodes, ensuring continues
     service availability.
  - It is not necessary for all worker nodes to have same hardware specifications.
    Worker node                  Hardware specifications
    Worker node1                 2GB of RAM and 2 core CPU
    Worker node2                 4GB of RAM and 4 core CPU
    Woeker node3                 8GB of RAM and 4 core CPU

  ## Demo
  ```
     kubectl create deployment/nginx-deployment --image=nginx --replicas=3
     kubectl get po -o wide # This command will show which node the pod is running
     Create additional worker nodes in digital ocean [Resize or autoscale and make it 2]
     After adding node create deployment with replicas=5
     kubectl create deployment nginx-deployment --image=nginx --replicas=5
     kubectl get pods -o wide
  ```
  ## Node selector
  - Use cases: Applications may require more CPU and memory, while few applications require minimum compute resources.

  ## Understanding the topic
  - Node Selector: Mechanisum used to control the placement of pods on specific nodes within a cluster.
  - Requirement: AppA pod requires high memory and CPU. Run it on big node.
  - Demo
  ```
  kubectl create deployment temp-deploy --image=nginx --replicas=3
  spec:
    nodeSelector:
      size: medium
    containers:
      - name: nginx
        image: nginx
   nodeSelectors are used to run certain workloads on specific nodes with special hardware or configuration.
   (CPU, high-memory nodes)
   To segregate workloads for compliance, performance or isolation purpose.
  Make sue nodes are labeled
  kubectl get nodes
  kubectl label node <name> size=large
  kubectl get po
  kubectl label node size-
  kubectl describe node <name>
  ```
  ## Deamonset:  Ensure all nodes run a copy of a pod.
  - Whenever new nodes are added to the cluster, pods will be created in them.
  - Example: Anti-virus pod, Mertics collection agent pod, log collection agent pod.
  - Common types of agents for daemonsets
     Anti-virus & Marlware scanning agents
     Log collection agents to collect logs from the nodes
     Monitoring and metrics collection agents to collect metrics about nodes.
  - If one worker nodes goes down daemon set will not run the pod on the available nodes like the deployment.
  ## Node afinity
  - nodeSelector: It uses a method of strict equality, where selectors must match with the label in the worker node.
  - nodeAafinity is same as nodeSelector, it is more flexible and more expressive.
  Requirement: Run AppA pod on any node except micro.
  ```
  Operators in node afinity.
  In                Matches value
  Notin             matches vaue
  Exists            matches key
  Doesnotexist      matches key
  ```
  - Hard and soft preferences
     1) Requirement (Hard requirement)
     2) Preffered (soft requirement)
  - Requiremet: Run AppA pod in extra large node (prefferred).
```
Nodeafinity:
apiVersion: v1
kind: pod
metadata:
  name: nginx
spec:
  affinity:
    nodeAffinity:
      requirementDuringSchedulingIgnoreDuringExecution:
            nodeSelectionTerms:
              - matchingExpressions:
                 - key: disktype
                   operator: in
                   values:
                     - ssd
containers:
  - name: nginx
    image: nginx
---------------------------------------------------
apiVersion: v1
kind: Pod
metadata:
  name: chakra
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
       nodeSelectorTerms:
       - matchExpressions:
          - key: size
            operator: DoesNotExist
  containers:
  - name: chakra
    image: httpd
--------------------------------------------------------------
apiVersion: v1
kind: Pod
metadata:
  name: chakra
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
       nodeSelectorTerms:
       - matchExpressions:
          - key: size
            operator: Exists
  containers:
  - name: chakra
    image: httpd
------------------------------------------------------
apiVersion: v1
kind: Pod
metadata:
  name: chakra
spec:
  affinity:
    nodeAffinity:
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 1
        preference:
          matchExpressions:
          - key: size
            operator: Exists
  containers:
  - name: chakra
    image: httpd
```

```
- Also check assign pod to node documentation
https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/
```
## Resource and Limits
 - Challange 1:
   Two applications A and B deployed as pod/containters, one of the two may cosume more resources than expected and affect other pod in the node.
 - Challange 2:
   A Pod may require a certian amount of memory and compute to run properly.
   A pod need 512MB of RAM and 2 core CPU to run optimally.
 - Requirement:
   Requests(minimum) and Limits(maximum) are two ways we can control the amount of resources that can be assigned to pod.
   Requests: Minimum CPU/RAM a container is guarnteed to get. Pod is scheduled/placed on a node that has enough resources.
   Limits: The max amount of CPU and Memory container is allowed to use.
   If container exceeds the limit, CPU will be throttled and for high memory container/pod gets killed.
 - Advantage:
   Prevents resource stravation on node.
   Ensures no single container consumes all resources on nodes.
   Ensure fair resource allocation
   Avoids over provisioning
   Improves stability and performance of your applications.
```
apiVersion: v1
kind: Pod
metadata:
  name: chakra
spec:
  containers:
  - name: chakra
    image: httpd
    resources:
     requests: 
      cpu: '0.1'
      memory: '128Mi'
     limits:
      cpu: '0.5'
      memory: '250Mi'
```
## Priorityclass
 Pods can have priority
 Advantages of priority class
 Influences scheduling order:- Pods with high priority are placed ahead in scheduling queue.
                               The scheduler tries to schedule higher priority pods before lower-prioroty pods.
 Preemption: If a higher priority pods can't be scheduled due to insufficient resources, the scheduler can evict(preempt) lower-priority pods from the node to make room for higher priority pod.
 PriorityClass object can have an integer value smaller or equal to 1 billion.
 ```
Test case -
Create higher and lower priority classes
And assign to pods and observe the eviction/preempt

apiVersion: scheduling.k8s.io/v1
description: high priority
kind: PriorityClass
metadata:
  creationTimestamp: null
  name: high
preemptionPolicy: PreemptLowerPriority
value: 1000

apiVersion: scheduling.k8s.io/v1
description: low priority
kind: PriorityClass
metadata:
  creationTimestamp: null
  name: low
preemptionPolicy: PreemptLowerPriority
value: 100

kubectl get pc
NAME                      VALUE        GLOBAL-DEFAULT   AGE     PREEMPTIONPOLICY
high                      1000         false            2m36s   PreemptLowerPriority
low                       100          false            20s     PreemptLowerPriority
system-cluster-critical   2000000000   false            44h     PreemptLowerPriority
system-node-critical      2000001000   false            44h     PreemptLowerPriority

apiVersion: v1
kind: Pod
metadata:
  name: pod1
spec: 
  priorityClassName: low 
  containers:
  - name: pod1
    image: httpd
    resources:
     requests: 
      cpu: '0.1'
      memory: '128Mi'
     limits:
      cpu: '0.5'
      memory: '250Mi'

apiVersion: v1
kind: Pod
metadata:
  name: pod2
spec: 
  priorityClassName: high 
  containers:
  - name: pod2
    image: httpd
    resources:
     requests: 
      cpu: '0.1'
      memory: '128Mi'
     limits:
      cpu: '0.5'
      memory: '250Mi'

Result:
C:\Users\chakr>kubectl get po -w
NAME   READY   STATUS    RESTARTS   AGE
pod1   1/1     Running   0          21s
pod2   0/1     Pending   0          0s
pod1   1/1     Running   0          25s
pod1   1/1     Terminating   0          25s
pod2   0/1     Pending       0          0s
pod1   1/1     Terminating   0          25s
pod1   0/1     Completed     0          26s
pod2   0/1     Pending       0          1s
pod2   0/1     ContainerCreating   0          1s
pod1   0/1     Completed           0          27s
pod1   0/1     Completed           0          27s
pod2   1/1     Running             0          3s
```
   
   




