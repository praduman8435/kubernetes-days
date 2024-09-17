# Kubernetes-part-5
<p align="center">
  <img src="https://cdn.hashnode.com/res/hashnode/image/stock/unsplash/jOqJbvo1P9g/upload/792a47f04af4ec8254a34c28a127e841.jpeg?w=1600&h=840&fit=crop&crop=entropy&auto=compress,format&format=webp">
</p>

### Efficient Kubernetes Scheduling with Kube-Scheduler

The kube-scheduler is Kubernetes default scheduler and part of the control plane. It selects the best node for new or unscheduled pods by filtering out nodes that don't meet the pod's requirements, known as feasible nodes. If no nodes are suitable, the pod remains unscheduled until one becomes available. The scheduler scores feasible nodes and selects the highest-scoring one to run the pod. This decision is then communicated to the API server in a process called binding. Custom schedulers can also be created if needed.

### Mastering Labels and Selectors for Resource Grouping

Labels and Selectors are standard methods to group things together. Labels are properties attached to each resources. Selectors help you to filter these resources. Kubernetes uses labels to connect different objects together.

* **A pod definition file with labels**
    
    ```yaml
    apiVersion: v1
    kind: Pod
    metadata:
     name: simple-webapp
     labels:
       app: App1
       function: Front-end
    spec:
     containers:
     - name: simple-webapp
       image: nginx
       ports:
       - containerPort: 8080
    ```
    
    ```basic
    kubectl apply -f <pod-definition-file>
    ```
    
* **To select the pod with labels**
    
    ```basic
    kubectl get pods --selector app=App1
    ```
    
* **To see the lable of a pod**
    
    ```basic
    kubectl get pods <pod-name> --show-labels
    ```
    
* **To label a pod once it is created**
    
    ```basic
    kubectl label pod <pod-name> key=value
    ```
    
* **To see pods with/without specific label**
    
    ```basic
    kubectl get pod -l <key>=<value>
    ```
    
    ```basic
    kubectl get pod -l <key>!=<value>
    ```
    
* **Imperative way to create deployment**
    
    ```basic
    kubectl create deploy bootcamp --image=nginx --replicas 3
    ```
    
* **Selection of pod using lable in replicasets**
    
    ```yaml
     apiVersion: apps/v1
     kind: ReplicaSet
     metadata:
       name: simple-webapp
       labels:
         app: App1
         function: Front-end
     spec:
      replicas: 3
      selector:
        matchLabels:
         app: App1
      template:
        metadata:
          labels:
            app: App1
            function: Front-end
        spec:
          containers:
          - name: simple-webapp
            image: simple-webapp
    ```
    
* **Selection of pod using lable in services**
    
    ```yaml
      apiVersion: v1
      kind: Service
      metadata:
       name: my-service
      spec:
       selector:
         app: App1
       ports:
       - protocol: TCP
         port: 80
         targetPort: 9376
    ```
    

### Understanding Kubernetes Namespaces for Resource Isolation

Namespaces are a way to divide cluster resources between multiple users, teams, or projects. They provide a scope for names, allowing multiple users to share the same cluster without interfering with each other.

**Kubernetes starts with four initial namespaces:**

1. **default**: A space where you start working in Kubernetes right away
    
2. **kube-node-lease**: Keeps track of whether each node in the system is healthy or not
    
3. **kube-public**: A place where anyone, even without special access, can see certain information
    
4. **kube-system**: Where Kubernetes itself stores its important stuff
    

* **To see all namespaces**
    
    ```basic
    kubectl get namespace
    ```
    
* **To create a namespace**
    
    ```basic
    kubectl create ns <name-of-namespace>
    ```
    

> Avoid creating namespaces with the prefix `kube-` since it is reserved for Kubernetes system namespaces

* **Creating two namespace and create deployment on each with nginx image and try to access the deployment on the other ns from the current ns**
    
    ```basic
    kubectl create ns frontent
    ```
    
    ```basic
    kubectl create ns backend
    ```
    
    ```basic
    kubectl create deploy demo -n frontent --image=nginx
    ```
    
    ```basic
    kubectl create deploy demo -n backend --image=nginx
    ```
    
    ```basic
    kubectl get deployment -n frontent -owide
    ```
    
* **To switch to a namespace**
    
    ```basic
    kubectl config set-context --current --namespace=backend
    ```
    

### Managing Resource Quotas in Kubernetes

This is a way to manage and limit resource usage in a specific namespace. It ensures that no single team, application, or user can consume too many resources (like CPU, memory, storage, or the number of objects) in a shared cluster. This Prevent one team or application from using all the resources in the cluster and also Manage resources efficiently.

#### **How Resource Quota Works**

Administrators set quotas at the **namespace** level, and Kubernetes enforces them. When a Pod or object (like a Persistent Volume or Service) is created, Kubernetes checks the quota and ensures that the creation doesn't exceed the defined limits

* **Create a namespace**
    
    ```basic
    kubectl create ns example-namespace
    ```
    

* **Create a resourceQuota**
    
    ```yaml
    apiVersion: v1
    kind: ResourceQuota
    metadata:
      name: example-quota
      namespace: example-namespace
    spec:
      hard:
        requests.cpu: "500m" # total amount of CPU that can be requested
        requests.memory: "200Gi" # total amount of memory that can be requested
        limits.cpu: "1" # total amount of CPU limit across all pods
        limits.memory: 400Gi # total amount of memory limit across all pods
        pods: "10" # total number of pods that can be created
    ```
    
    ```basic
    kubectl apply -f <RQ-name.yaml>
    ```
    
* **To check resouceQuota**
    
    ```basic
    kubectl get resourceQuota
    ```
    

### Ensuring Pod Scheduling Readiness

Pod Scheduling Readiness is a feature that lets you control when a Pod is ready to be placed on a node. It adds a delay in scheduling the Pod until certain conditions are met, making sure the Pod is fully prepared. This is helpful when the Pod depends on other services or resources to be available before it can start running smoothly

</p>
  <p align="center">
  <img src="https://kubernetes.io/docs/images/podSchedulingGates.svg">
</p>

**Example:** Waiting for a Service to Be Ready

* **Pod definition file that includes custom scheduling gates**
    
    ```yaml
    apiVersion: v1
    kind: Pod
    metadata:
      name: test-pod
    spec:
      schedulingGates:
      - name: example.com/service-ready
      containers:
      - name: pause
        image: registry.k8s.io/pause:3.6
    ```
    

* **Implement a Custom Controller**
    
    ```yaml
    # Pseudo-code for the custom controller
    if service_is_ready:
      remove_scheduling_gate(pod_name="test-pod", gate_name="example.com/service-ready")
    ```
    
* #### **Use the Custom Controller to Manage Scheduling**
    
    The custom controller will ensure that the Pod is only scheduled when the specified service is confirmed to be ready. Until then, the Pod will remain unscheduled.
    

> **This approach ensures that the Pod will only be scheduled when the necessary conditions are met, providing a more controlled scheduling process**

### Enhancing Availability with Pod Topology Spread Constraints

It help to ensure high availability by spreading Pods across different nodes, zones, or regions. It also helps to maintain balanced load by evenly distributing Pods to prevent overloading any single node or domain. It Improve fault tolerance by avoiding concentration in a single location

**Example**

> Kubernetes Deployment using Pod Topology Spread Constraints to evenly distribute Pods across different nodes

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: demo-app
spec:
  replicas: 4
  selector:
    matchLabels:
      app: demo-app
  template:
    metadata:
      labels:
        app: demo-app
    spec:
      containers:
      - name: app-container
        image: nginx
      topologySpreadConstraints:
      - maxSkew: 1
        topologyKey: "kubernetes.io/hostname"
        whenUnsatisfiable: DoNotSchedule
        labelSelector:
          matchLabels:
            app: demo-app
```

* **maxSkew**: degree to which the pod is evenly distributed
    
* **topologyKey**: key of node labels
    
* **whenUnsatisfiable**: If the constraints cannot be met, `DoNotSchedule` or `ScheduleAnyway`
    
* **labelSelector**: find maching pod
    

> To scale this deployment
> 
> ```basic
> kubectl scale deployment demo-app --replicas=6
> ```

When you need to prevent new workloads from being scheduled on a node due to issues with that node, you can `cordon` the node. Cordon marks the node as unschedulable, ensuring that no new pods are scheduled to it.

```basic
kubectl cordon <node-name>
```

To make that node again schedulable you need to uncordon that node

```basic
kubectl uncordon <node-name>
```

### Prioritizing Pods with Priority Classes

It helps the Kubernetes Scheduler decide which Pods should be scheduled first when resources are limited. Pods with higher priority values are scheduled before those with lower priority. Higher-priority Pods can displace lower-priority Pods if resources are scarce.

**Example of priorityClass**

> Here’s how you can create a PriorityClass and use it with a Pod

* **Define a PriorityClass**
    
    ```yaml
    apiVersion: scheduling.k8s.io/v1
    kind: PriorityClass
    metadata:
      name: demo-priority
    value: 1000000
    globalDefault: false
    description: "This priority class is for critical workloads."
    ```
    
* **Use the PriorityClass in a Pod:**
    
    ```yaml
    apiVersion: v1
    kind: Pod
    metadata:
      name: high-priority-pod
    spec:
      priorityClassName: demo-priority
      containers:
        - name: my-container
          image: my-image
    ```
    

### Understanding and Managing Pod Overhead

**Pod Overhead** is the extra amount of resources a Pod needs beyond what its containers require. It covers things like the Pod's management and networking needs. Pod Overhead helps ensure that enough resources are available for both the Pod and its additional needs, not just the containers inside it

#### **How It Works**

* ***Container Requests*:** You specify how much CPU and memory a container needs
    
* ***Add Overhead*:** Kubernetes adds extra resources for the Pod’s management and networking
    
* ***Total Resources*:** The total resources needed are the sum of container requests plus Pod Overhead
    

> #### **Example:** Specify Pod Overhead in a Pod’s resource request

**Define a** `Pod` with Overhead

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  containers:
  - name: my-container
    image: nginx
    resources:
      requests:
        memory: "512Mi"
        cpu: "500m"
  overhead:
    cpu: "100m"
    memory: "100Mi"
```

In this example:

* **Container Requests:** The container requests 512 Mi of memory and 500 milliCPU.
    
* **Pod Overhead:** An additional 100 Mi of memory and 100 milliCPU are reserved for the Pod’s overhead.
    

### Conclusion

> In this article, we delved into several advanced Kubernetes concepts, including scheduling, labels and selectors, namespaces, resource quotas, pod scheduling readiness, topology spread constraints, priority classes, and pod overhead. Mastering these features is essential for effectively managing and optimizing Kubernetes clusters. By utilizing these tools and techniques, you can ensure high availability, balanced resource usage, and controlled scheduling processes, ultimately leading to a more robust and resilient Kubernetes environment.
