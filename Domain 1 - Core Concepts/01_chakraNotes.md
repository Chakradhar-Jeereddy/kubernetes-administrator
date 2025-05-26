## K8S exam points
 - Workload and scheduling : 15%
 - Storage: 10%
 - Troubleshooting: 30%
 - Cluster architecture, installation and configuration - 25%
 - Security and networking - 20%

### Challenges with docker
 - Challange1: Manual scaling
    - Developer needs to manually start additional containers to scale the application
    - Need to identify server with enough compute and storage to place the containers
 - Challange2: Lack of fault tolerance
    - If container fail or craches due to resource exhaustion, it wouldn't automatically restart without manual intervention
      in most of the cases.

 ### Key takeaways
     - Manual depoyment, scaling, networking and monitoring

## Container Orchestration
    - Automates the deployment, managment, scaling and networking of containers
    - Users wont login to each server and run docker commands
    - They directly interact with Orchestration tool, the tool will automate everything that was requested.

    - Solving challange: Scaling
          - Trafic is evenly distributed
          - We can use nodeSelectors, Affinity to control the placement of pods on nodes of the cluster
    - Solving fault tollerance
          - With the use of replicasets the failed pods are automatically relaunched to ensure high availability

 ## Other orechestration tools available in the market
    - Docker swarm
    - Kubernetes
    - Amazon EKS (Fully managed service)
    - AKS
    - GCP
    - IBM
    - Digital Ocean
 ## Kubernetes(K8s):
    - It is an open-source container archestration engine developed by google and is now being
      maintained by cloud native computing foundation.

 ## Installation options:
    - Go to k8s website, download individual components and integrate them one by one.
       - Go to stores get ingrediants and prepare food from scratch. (need skill to integrate)
    - Use tools like minikube, kubeadm to quickly setup K8s cluster.
       - Get ready-made mix, make it hot and serve.
    - Use managed kubernetes service
       - Order it from the restaurant.

  ## Methods to interact with control plane
      - API
      - CLI
      - GUI
  ## Authentication
     - kubeconfig file which includes the URL/DNS to reach the cluster, port and authentication token.
     - K8s by default look for the file with name config under ~/.kube/config file (without extension).
     - Download config from overview page on digitalocean site and move it to ~/.kube folder and rename file to "config".
     - If you rename the file to some other name, kubectl command will not work, in those cases explicitly supply the name
     - kubectl get pods --kubeconfig "/u01/costum-config"
  ## Installing kubectl(windows)
     - Download from https://kubernetes.io/releases/download/#binaries
     - Create .kube

  ## Basics of POD
    - A pod can contain one or more docker containers that share the same network, namespace and storage volume.
    - Communicate using localhost, web server<->database server.
    - Pod always runs on a node.
    - Node is a worker machine in kubernetes
    - Node is managed by kubernetes control plane.
    - Node can have multiple pods.
    ```
    Basic commands
    kubectl get nodes
    kubectl decribe node <name>
    kubectl descibe node/<name>
    kubectl run nginx_01 --image=nginx
    kubectl get pods
    kubectl get pods -o wide   #To check which node the pod is running"
    kubectl delete pod/<name> or kubectl delete pod <name>
    ```
  ## Methods to create objects in k8s
     - Two primary ways to create objects
       ```
       kubectl run ....
       kubectl apply -f <manifest file>
       ```
     - Manifest file contains information of resource to be created.
     - The file can be yaml or json

  ## Benifits of Manifest file
     - It can be stored in version control system like Git.
     - Useful to trak changes to infrastructure overtime and allows easy rollback to previous version.
     - Allows to define multiple dependent resources in a single file
     - Pods also interacts with other objects like "Secrets, Volumes, Service"
     ```
       Command to find all available resources
       kubectl api-resources
       apiVersion: [API version]
       Kind: [Resource type]
       Metadata:
         name: [Resource name]
       Spec:
         [Resource specification and configuration]
      eg: - 
      apiVersion: v1
      kind: pod
      metadata:
        name: nginx01
      spec:
        containers:
         - name: nginx01
           image: nginx:latest
      ```
      
  ## Genrating manifest file using kubectl command with dry run option
      - kubectl run nginx01 --image=nginx --dry-run=client -o yaml > pod.yaml

  ## Difference between container and pod
      - kubectl run command allows to run single container based pods
      - For pods with multiple containers, we must use manifest based approach.
      - To access container use the following command
         - kubectl exec -it <pod-name> -- bash
  ## Docker(Entrypoint, cmd), Kubernetes(command, args)
     ```
     cat Dockerfile
     FROM busybox:latest
     CMD ["nginx","-g","daemon off"]
     docker build -t busybox:ping .
     docker run busybox:ping
     Note: As soon as ping command finish, there is no additional process running, so the pod exit.
           When container start the ENRYPOINT executes first and then the CMD.
           When docker image is built from a docker file, it can have an entrypoint and cmd, defines 
           what container needs to run when it starts.
     eg:
     FROM ununtu
     ENTRYPOINT ["/bin/echo"]
     CMD ["Hello World"]
     output will be hello world

     In K8S manifest file we use "command" and "args"
     command: "array" (executes first)
     args: "array" (not mandatory, for easy understanding)
     Syntax:
     command: ["/bin/ping"]
     args: ["-c","30","googe.com"]
     Note: Once the ping finishes sending and receiving 3 requests, there is nothing else defined in CMD.
           There is no other process keeping the container running, it exist as soon as the initial command finishes.

    kubectl run nginx --image=nginx --command -- echo "hello world"
    kubectl run nginx --image=busybox:latest --command -- /bin/ping -c 10 google.com
    ```
## Different ways to define arguments and and commands
   - Array (JSON array notation, square brackets[]) - compact
   - Multiline yaml list for each item) if the argument is lengthy
  - The separation of commands and arrguments in k8s is a design choice, provides flexibilty and clarity.
      spec:
        containers:
          - "/bin/ping"
          - "-c"
          - "20"
          - "google.com"
## EXPOSE
  - It is only for documentation purpose to tell which port the application will listen to the person
    running the container, so that he can publish the port using "-p"
   - We can inpect the pod to find its port and then publish it.
     kubectl describe pod <pod-name>|more
    ```
    Dockefile
      FROM Ubuntu
      RUN app-get install service
      EXPOSE 9324
      CMD ["service"]
     ```
 ## Exposing port at kubernetes level
    ```
    ports:
      - containerPort: 8080
    ```
  ## Label and selectors
   - Labels are key and values pairs attached to objects like pods
   - Selectors are used to get objects based on labels
   - Assigning labels to kubernetes objects
   ```
   kubectl get pods --show-label
   kubectl get pods -l run=nginx
   kubectl label pod nginx env=dev
   kubectl get po --show-labels  (Assigning the labels)
   kubectl label pod nginx env-
   kubectl lavel pod nginx env=dev --dry-run=client -o yaml > nginx.yaml
   ```
 ## Replicasets
   - For maintaining a stable set of pods running at any given time.
   - Requirement to run three pods on nginx image at all times.
   - To overcome manual scaling and fault tolerance
   - In case of pod failure or crash, kubernetes will not automatically create it.
   - In case of spike in workload, we need to manually launch new containers.
   - Replicaset will overcome these challanges.
   - It will have logic to recreate pods when needed.
   ```
   kubectl get replicasets
   kubectl get rs
   kubectl apply -f .\rs.yaml   kubectl scale rs/rs --replicas=2
   https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/
   kubectl get rs
   kubectl get po --show-labels
   kubectl scale rs/frontend --replicas=3
   kubectl delete po/frontend-96tpm
   kubectl get po #new one gets launched
   kubectl scale rs/frontend --replicas=2
   kubectl get rs
   kubectl scale rs/frontend --replicas=0
   kubectl delete rs/frontend
   ```

## Deployments
  - Extensively used in production environment
  - Wrapper on top of replicaset
  - Overcomes the disadvantage of replica set.
  - Challange 1:
     - Maintains the current state of the running pod.
     - Changes applied to replicaset will not apply to running pods untill these are restarted.
     - Requirement is to run 3 pods with nginx at all time.
     - Later you need higher version of nginx
     - When we update pod template as part of replicaset, the existing pods are not updated.
     - Eg: change image from niginx to "httpd:latest" (apache)
     ```
     kubectl describe rs/frontend
     kubectl describe po/nginx-adad
     kubectl apply -f rs.yaml (after changing the image to apache)
     kubectl describe rs/frontend
     kubectl get po/nginx
     kubectl describe po/nginx  # changes wont get applied to replicaset
     ```
   - Challange 2:
     - No builtin rollback Mechanism.
     - Doesn't maintain revison/rollout history
     - Replicasets lack rollback machanism for reverting to a previous configuration
       in case of errors during an update.
    - Challange 3:
      - Label collision with replicaset selectors
      - When replicaset's selector matches the label of existing pod that it didn't create,
        it starts treating the pod as part of its managed pods. This can cause unintended concequences.
        ```
        eg:
        kubectl run frontend --image=nginx
        kubectl get po --show-labels
        kubectl label po/frontend run-
        kubectl label po/frontend tier=frontend
        kubectl get rs
        kubectl apply -f .\rs.yaml
        kubectl get rs
        kubectl get pods --show-label
        kubectl delete rs/frontend
        ```
   
   
   
      
    
    

