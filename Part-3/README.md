# Kubernetes part-3
<p align="center">
  <img src="https://cdn.hashnode.com/res/hashnode/image/stock/unsplash/jOqJbvo1P9g/upload/792a47f04af4ec8254a34c28a127e841.jpeg?w=1600&h=840&fit=crop&crop=entropy&auto=compress,format&format=webp">
</p>

### Imperative vs. Declarative: Two Approaches to Kubernetes Infrastructure Management

1. **Imperative:** Directly command Kubernetes to perform actions step-by-step
    
    > To create a pod in imperative way we need to run a single command `kubectl run my-pod —image=nginx`
    
2. **Declarative:** Define the desired state in a configuration file and let Kubernetes manage it
    
    > To create a pod in imperative way we need to write a pod definition file( `a yaml file` ) first
    > 
    > ```yaml
    > apiVersion: v1
    > kind: Pod
    > metadata:
    >   name: nginx
    > spec:
    >   containers:
    >   - name: my-container
    >     image: nginx
    > ```
    > 
    > then run command `kubectl apply -f filename.yaml`
    

### Understanding YAML: The Backbone of Kubernetes Configuration

* **YAML** stands for (YAML Ain't Markup Language)
    
* It is easy & comes with human-readable format
    
* Indentation matters alot
    
* It is used for configuring Kubernetes resources
    
* It allows you to define and organize settings in a structured way
    
* It makes easy to create and manage complex configurations
    

**YAML file to create a pod**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  containers:
  - name: my-container
    image: nginx
```

> ***Command used to apply a YAML configuration :***
> 
> ```basic
> kubectl apply -f filename.yaml
> ```

This command tells Kubernetes to create or update resources based on the specifications defined in the YAML file

### Key Concepts in YAML: Objects, Lists, Comments, and Multi-line Strings

1. **Objects:** Collections of key-value pairs, similar to dictionaries in Python. They are defined using indentation.
    
    **Example:**
    
    ```yaml
    person:
      name: Alice
      age: 30
      city: New York
    ```
    
    In this example, `person` is an object with three key-value pairs: `name`, `age`, and `city`.
    
2. **Lists:** Sequences of items, where each item is prefixed by a hyphen (`-`) and space.
    
    **Example:**
    
    ```yaml
    yamlCopy codefruits:
      - Apple
      - Banana
      - Cherry
    ```
    
    Here, `fruits` is a list containing three items: `Apple`, `Banana`, and `Cherry`.
    
3. **Comments:** YAML allows comments, start with a `#`. These are ignored by YAML and are used for adding explanations.
    
    **Example:**
    
    ```yaml
    # This is a comment
    server:
      host: localhost
      port: 8080
    ```
    
4. **Multi-line Strings:** YAML provides ways to handle multi-line strings using `|` (literal block) or `>` (folded block).
    
    **Literal Block (**`|`):
    
    ```yaml
    description: |
      This is a multi-line
      string that preserves newlines.
    ```
    
    **Folded Block (**`>`):
    
    ```yaml
    description: >
      This is a multi-line
      string that folds into a single line.
    ```
    

### Pods in Kubernetes: The Fundamental Unit of Deployment

> Application runs as a pod in kubernetes

* Pod is the smallest unit in kubernetes
    
* Inside pod the containers run
    
* It holds one or more containers that works together
    
* No two similar kind of container can run in a single pod
    
* Each Pod gets its own IP address
    
* Containers in the same Pod can communicate with each other using `localhost`
    
* Pods can have storage that containers in the Pod can use to share files
    

**Example**: YAML file to create a Pod

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  containers:
  - name: my-container
    image: nginx
    ports:
    - containerPort: 80
```

This file creates a Pod called `my-pod` with a single container that runs the `nginx` web server.

> Creating same pod in imperative way
> 
> ```basic
> kubectl run my-pod --image=nginx --port=80
> ```

* Command to list all the pods running in the cluster is `kubectl get pods`
    
* If you want more information about a specific pod the use `kubectl describe pod pod-name`
    

### Pod Lifecycle: From Creation to Termination

It represents the different phases a pod goes through from creation to termination.

1. **Pending**
    
    * The pod is created but not yet running. It’s waiting for the scheduler to assign it to a node or for container images to be pulled
        
2. **Running**
    
    * The pod is assigned to a node, and at least one container is running. However, the container might face issues like:
        
        * **CrashLoopBackOff**: The container repeatedly crashes and restarts
            
        * **Error**: The container terminates with an error
            
3. **Succeeded**
    
    * All containers in the pod have finished successfully and won't restart
        
4. **Failed**
    
    * One or more containers in the pod have terminated with an error, and the pod is not being restarted.
        

> **Termination:**
> 
> When a pod is deleted, it enters in Terminating state, where Kubernetes stops and removes its containers

### Namespaces in Kubernetes: Organizing and Isolating Resources

**namespaces** are used to organize and isolate resources within a cluster. They allow different teams, projects, or environments (like dev, test, and prod) to run without conflicts. Each namespace has its own resources, and you can apply resource limits and access controls per namespace. It makes easier to manage and separate resources in a large Kubernetes cluster

**Key commands:**

* Create a namespace
    
    ```basic
    kubectl create namespace my-namespace
    ```
    
* List resources in a specific namespace
    
    ```basic
    kubectl get pods -n my-namespaceview all namespaces
    ```
    

* view all namespaces
    
    ```basic
    kubectl get ns
    ```
    

### Init Containers: Setting Up Your Pod for Success

**Init Container** is like a helper that runs before the main part of your app starts in a pod. It’s used to perform setup tasks or ensure certain conditions are met before the main container runs

* Init containers always run before any other container in the pod
    
* A pod can have one or more init containers
    
* The main container will not start until the Init container completes finishes successfully
    
* If a pod’s init container fails, the kubelet repeatedly restart that init container untill it succeeds
    
* If the pod has a `restartpolicy` of never, and an init container fails during startup of the pod, kubernetes treats overall pod as failed
    
* The regular init containers do not support the `lifecycle`, `livenessProbe`, `readinessProbe` or `startupProbe` fields
    

> **Init container definition file**
> 
> ```yaml
> apiVersion: v1
> kind: Pod
> metadata:
>   name: bootcamp-pod
> spec:
>   volumes:
>   - name: shared-data
>     emptyDir: {}
>   initContainers:
>   - name: bootcamp-init
>     image: busybox
>     command: ['sh', '-c', 'wget -O /usr/share/data/index.html http://kubesimplify.com']
>     volumeMounts:
>     - name: shared-data
>       mountPath: /usr/share/data
>   containers:
>   - name: nginx
>     image: nginx
>     volumeMounts:
>     - name: shared-data
>       mountPath: /usr/share/nginx/html
> ```
> 
> **Multiple init containers definition file**
> 
> ```yaml
> apiVersion: v1
> kind: Pod
> metadata:
>   name: init-demo-2
> spec:
>   initContainers:
>   - name: check-db-service
>     image: busybox
>     command: ['sh', '-c', 'until nslookup db.default.svc.cluster.local; do echo waiting for db service; sleep 2; done;']
>   - name: check-myservice
>     image: busybox
>     command: ['sh', '-c', 'until nslookup myservice.default.svc.cluster.local; do echo waiting for db service; sleep 2; done;']
>   containers:
>   - name: main-container
>     image: busybox
>     command: ['sleep', '3600']
> ```
> 
> Definition file of services for multiple init container
> 
> ```yaml
> apiVersion: v1
> kind: Service
> metadata:
>   name: db
> spec:
>   selector:
>     app: demo1
>   ports:
>   - protocol: TCP
>     port: 3306
>     targetPort: 3306
> ---
> apiVersion: v1
> kind: Service
> metadata:
>   name: myservice
> spec:
>   selector:
>     app: demo2
>   ports:
>   - protocol: TCP
>     port: 80
>     targetPort: 80
> ```

### Sidecar Containers: Enhancing Your Main Application

**Sidecar Container** is an additional container that runs alongside the main container in a pod. It helps enhance or support the main container’s functionality

* The sidecar container runs in the same pod as the main container, sharing the same network and storage resources
    
* It can act as a proxy server to handle communication between the main container and external services
    

> **Example of multiple container with a sidecar container**
> 
> ```yaml
> apiVersion: v1
> kind: Pod
> metadata:
>   name: multi-container-pod
> spec:
>   volumes:
>   - name: shared-data
>     emptyDir: {}
>   initContainers:
>   - name: meminfo-container
>     image: alpine
>     restartPolicy: Always
>     command: ['sh', '-c', 'sleep 5; while true; do cat /proc/meminfo > /usr/share/data/index.html; sleep 10; done;']
>     volumeMounts:
>     - name: shared-data
>       mountPath: /usr/share/
> data
>   containers:
>   - name: nginx-container
>     image: nginx
>     volumeMounts:
>     - name: shared-data
>       mountPath: /usr/share/nginx/html
> ```
> 
> **Example of a pod with a sidecar container**
> 
> ```yaml
> apiVersion: v1
> kind: Pod
> metadata:
>   name: sidecar-example
> spec:
>   containers:
>     - name: main-container
>       image: main-app-image
>       ports:
>         - containerPort: 80
>     - name: sidecar-container
>       image: logging-agent-image
>       volumeMounts:
>         - name: shared-storage
>           mountPath: /logs
>   volumes:
>     - name: shared-storage
>       emptyDir: {}
> ```

* **Main Container**: Runs the primary application
    
* **Sidecar Container**: Runs a logging agent that collects logs from the shared storage directory
    

### Conclusion

> In conclusion, Kubernetes offers a robust and flexible platform for managing containerized applications. By understanding the different ways to define and manage infrastructure, such as imperative and declarative approaches, users can choose the method that best suits their needs. YAML plays a crucial role in configuring Kubernetes resources, providing a human-readable format that simplifies the creation and management of complex configurations. Pods, as the smallest unit in Kubernetes, encapsulate containers and their lifecycle, ensuring efficient resource utilization and communication. Namespaces help in organizing and isolating resources, making it easier to manage large clusters. Additionally, init containers and sidecar containers extend the functionality of pods, allowing for more sophisticated application setups. By mastering these concepts, users can effectively leverage Kubernetes to deploy, scale, and manage their applications in a cloud-native environment.
