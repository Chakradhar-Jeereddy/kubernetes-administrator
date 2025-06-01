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
  - Node selector: It uses a method of strict equality, where selectors must match with the label in the worker node.
  - Node afinity is same as nodeselector, it is more flexible and more expenssive.
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
    nodeaffinity:
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
```
- Also check assign pod to node documentation
```





