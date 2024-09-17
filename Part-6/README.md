# Kubernetes-part-6
<p align="center">
  <img src="https://cdn.hashnode.com/res/hashnode/image/stock/unsplash/jOqJbvo1P9g/upload/792a47f04af4ec8254a34c28a127e841.jpeg?w=1600&h=840&fit=crop&crop=entropy&auto=compress,format&format=webp">
</p>

### Ensuring Application Availability with ReplicaSets

It helps keep your application running by ensuring the correct number of pod replicas are always available, making it essential for scaling and reliability. ReplicaSets are often managed by **Deployments**, which provide more features like rolling updates. When you create a Deployment, it automatically creates a ReplicaSet for managing pod replicas.

> **Example:**

If you set a ReplicaSet to have 3 pods, and one pod stops, the ReplicaSet will automatically create a new pod so that there are always 3 running.

* **Defining a ReplicaSet: YAML Example**
    
    ```yaml
    apiVersion: apps/v1
    kind: ReplicaSet
    metadata:
      name: nginx-rs
      labels:
        app: nginx
    spec:
      replicas: 3
      selector:
        matchLabels:
          app: nginx
      template:
        metadata:
          labels:
            app: nginx
        spec:
          containers:
          - name: nginx
            image: nginx:latest
            ports:
            - containerPort: 80
    ```
    
    This YAML file tells Kubernetes to create 3 identical pods running the `nginx` web server. It will automatically make sure there are always 3 pods running, so if one stops, Kubernetes will start another. Each pod runs on port 80, and all are labeled with `app: nginx` for identification.
    

* **Real-Time watch and monitor the status of your pods**
    
    ```basic
    kubectl get pod -w
    ```
    
* **To see the** `ReplicaSet` **present**
    
    ```basic
    kubectl get rs
    ```
    
* **To delete a replicaSet and all its pod**
    
    ```basic
    kubectl delete rs <rs-file>
    ```
    

#### **Understanding Propagation Policies in Kubernetes**

It is used to control how resources are deleted, especially when dealing with related resources like ReplicaSets and their pods. It decides whether the resources created by a ReplicaSet (like pods) should be deleted right away or left running when the ReplicaSet is removed.

#### Types of Propagation Policies:

* **Foreground**: Pods are deleted first, then the ReplicaSet.
    
    ```basic
    kubectl delete replicaset nginx-rs --cascade=foreground
    ```
    
    > **Another way**: using `kubectl proxy` with a `curl` command to delete a ReplicaSet using the Kubernetes API and specify a propagationPolicy
    > 
    > * **Start the Kubernetes Proxy**
    >     
    > 
    > ```basic
    > kubectl proxy --port=8080
    > ```
    > 
    > This command starts a local proxy that allows you to interact with the Kubernetes API at `localhost:8080` without needing to authenticate with tokens.
    > 
    > ```basic
    > kubectl proxy --port=8080
    > curl -X DELETE 'http://localhost:8080/apis/apps/v1/namespaces/default/replicasets/nginx-rs' \
    >      -d '{"kind":"DeleteOptions","apiVersion":"v1","propagationPolicy":"Foreground"}' \
    >      -H "Content-Type: application/json"
    > ```
    
* **Background**: ReplicaSet is deleted first, pods are deleted later.
    
    ```basic
    kubectl delete replicaset nginx-rs --cascade=background
    ```
    
    > **Another way**: using `kubectl proxy` with a `curl` command to delete a ReplicaSet using the Kubernetes API and specify a propagationPolicy
    > 
    > * **Start the Kubernetes Proxy**
    >     
    > 
    > ```basic
    > kubectl proxy --port=8080
    > ```
    > 
    > This command starts a local proxy that allows you to interact with the Kubernetes API at `localhost:8080` without needing to authenticate with tokens.
    > 
    > ```basic
    > curl -X DELETE 'http://localhost:8080/apis/apps/v1/namespaces/default/replicasets/nginx-rs' \
    >      -d '{"kind":"DeleteOptions","apiVersion":"v1","propagationPolicy":"Background"}' \
    >      -H "Content-Type: application/json"
    > ```
    
* **Orphan**: ReplicaSet is deleted, but the pods remain running.
    
    ```basic
    kubectl delete replicaset nginx-rs --cascade=orphan
    ```
    
    > **Another way**: using `kubectl proxy` with a `curl` command to delete a ReplicaSet using the Kubernetes API and specify a propagationPolicy
    > 
    > * **Start the Kubernetes Proxy**
    >     
    > 
    > ```basic
    > kubectl proxy --port=8080
    > ```
    > 
    > This command starts a local proxy that allows you to interact with the Kubernetes API at `localhost:8080` without needing to authenticate with tokens.
    > 
    > ```basic
    > curl -X DELETE 'http://localhost:8080/apis/apps/v1/namespaces/default/replicasets/nginx-rs' \
    >      -d '{"kind":"DeleteOptions","apiVersion":"v1","propagationPolicy":"Orphan"}' \
    >      -H "Content-Type: application/json"
    > ```
    

### Mastering Kubernetes Deployments

It is used to manage and automate the lifecycle of applications running in pods. It provides more advanced features than a ReplicaSet, such as rolling updates, rollback capabilities, and scaling.

* **Creating and Managing Deployments with YAML**
    
    ```yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: nginx-deployment
    spec:
      replicas: 3
      selector:
        matchLabels:
          app: nginx
      template:
        metadata:
          labels:
            app: nginx
        spec:
          containers:
          - name: nginx
            image: nginx:1.16.1
            ports:
            - containerPort: 80
    ```
    
    ```basic
    kubectl apply -f deployment.yaml
    ```
    
* **Get Deployment Status**
    
    ```basic
    kubectl get deployments
    ```
    
* **Scaling Deployments for Optimal Performance**
    
    ```basic
    kubectl scale deployment nginx-deployment --replicas=5
    ```
    
* **Updating Deployment Images**
    
    ```basic
    kubectl set image deployment/nginx-deployment nginx=nginx:1.17.0
    ```
    
* **Checking Deployment Status**
    
    ```basic
    kubectl rollout status deployment/nginx-deployment
    ```
    
* **Checking Rollout History**
    
    ```basic
    kubectl rollout history deploy/nginx-deployment
    ```
    
* **View Specific Revision**
    
    ```basic
    kubectl rollout history deploy/nginx-deployment --revision=2
    ```
    
* **Rolling Back to Previous Deployment Revisions**
    
    ```basic
    kubectl rollout undo deployment/nginx-deployment --to-revision=1
    ```
    
* **Pause a Rollout**
    
    ```basic
    kubectl rollout pause deployment/nginx-deployment
    ```
    
* **Edit Deployment Directly**
    
    ```basic
    kubectl edit deployment/nginx-deployment
    ```
    

#### **Recreate Strategy: Minimizing Downtime During Updates**

With this strategy, when a new version of an application is deployed, Kubernetes first **deletes all existing pods** before creating the new ones. This can cause some downtime because there will be a gap between the old pods being terminated and the new ones starting.

> **Example YAML for Recreate Strategy**
> 
> ```yaml
> apiVersion: apps/v1
> kind: Deployment
> metadata:
>   name: demo-deployment
> spec:
>   replicas: 3
>   strategy:
>     type: Recreate
>   selector:
>     matchLabels:
>       app: demo
>   template:
>     metadata:
>       labels:
>         app: demo
>     spec:
>       containers:
>       - name: demo
>         image: nginx:latest
>         ports:
>         - containerPort: 80
> ```
> 
> ```basic
> kubectl apply -f <recreate-yml-file>
> ```
> 
> ```basic
> kubectl set image deploy/demo-deployment demo=nginx:14.0
> ```

### Probes: Ensuring Pod Health and Readiness

This is used to check the health of pod. They help Kubernetes know when a pod is ready to start serving traffic or if it’s still alive and functioning correctly. If a probe fails, Kubernetes can take action like restarting the pod

#### Types of probes in Kubernetes

* **Liveness Probe:** Checks if the pod is alive. Restarts it if it’s stuck.
    
* **Readiness Probe:** Checks if the pod is ready to serve traffic. Stops sending traffic if it’s not ready.
    
* **Startup Probe:** Gives the pod time to fully start. Ensures it isn’t marked as failed too early.
    

> #### Implementing Probes in Deployment YAML
> 
> ```yaml
> apiVersion: apps/v1
> kind: Deployment
> metadata:
>   name: nginx-deployment
> spec:
>   replicas: 2
>   selector:
>     matchLabels:
>       app: nginx
>   template:
>     metadata:
>       labels:
>         app: nginx
>     spec:
>       containers:
>       - name: nginx
>         image: nginx:latest
>         ports:
>         - containerPort: 80
>         livenessProbe:
>           httpGet:
>             path: /
>             port: 80
>           initialDelaySeconds: 15
>           timeoutSeconds: 2
>           periodSeconds: 5
>           failureThreshold: 3
>         readinessProbe:
>           httpGet:
>             path: /
>             port: 80
>           initialDelaySeconds: 5
>           timeoutSeconds: 2
>           periodSeconds: 5
>           successThreshold: 1
>           failureThreshold: 3
>         startupProbe:
>           httpGet:
>             path: /
>             port: 80
>           initialDelaySeconds: 10
>           periodSeconds: 5
>           failureThreshold: 10
> ```
> 
> ```basic
> kubectl apply -f <yaml-file>
> ```

* **Liveness Probe**: Checks if the container is still running. If it fails 3 times (every 5 seconds), the container is restarted.
    
* **Readiness Probe**: Checks if the container is ready to serve traffic. If it fails 3 times, the container won’t receive traffic until it passes.
    
* **Startup Probe**: Gives the container 10 chances (every 5 seconds) to start before other probes begin checking it.
    

### Blue-Green Deployment: Minimizing Downtime and Risk

* A strategy for deploying applications that minimizes downtime and risk.
    
* It involves running two environments: `Blue` and `Green`
    
* `Blue` represents the current version of the application that's live and handling user traffic.
    
* `Green` is a new version of the application that is being prepared for release.
    

**Example:** You have an application running with version 1 (Blue) and want to deploy version 2 (Green)

> * Current Deployment (Blue) : This is the live version
>     
>     ```yaml
>     apiVersion: apps/v1
>     kind: Deployment
>     metadata:
>       name: myapp-blue
>     spec:
>       replicas: 2
>       selector:
>         matchLabels:
>           app: myapp
>           version: blue
>       template:
>         metadata:
>           labels:
>             app: myapp
>             version: blue
>         spec:
>           containers:
>           - name: myapp
>             image: myapp:1.0
>             ports:
>             - containerPort: 80
>     ```
>     
> * **New Deployment (Green)**: This is the new version
>     
>     ```yaml
>     apiVersion: apps/v1
>     kind: Deployment
>     metadata:
>       name: myapp-green
>     spec:
>       replicas: 2
>       selector:
>         matchLabels:
>           app: myapp
>           version: green
>       template:
>         metadata:
>           labels:
>             app: myapp
>             version: green
>         spec:
>           containers:
>           - name: myapp
>             image: myapp:2.0
>             ports:
>             - containerPort: 80
>     ```
>     
>     ```basic
>     kubectl apply -f myapp-green.yaml
>     ```
>     
> * **Service**: The service will route traffic to the live version (initially Blue). Later, it will be updated to point to the Green version.
>     
>     ```yaml
>     apiVersion: v1
>     kind: Service
>     metadata:
>       name: myapp-service
>     spec:
>       selector:
>         app: myapp
>         version: blue # Initially points to Blue
>       ports:
>       - protocol: TCP
>         port: 80
>         targetPort: 80
>       type: LoadBalancer
>     ```
>     
> * **Switch Traffic to Green**: Update the service to point to the Green version
>     
>     ```yaml
>     apiVersion: v1
>     kind: Service
>     metadata:
>       name: myapp-service
>     spec:
>       selector:
>         app: myapp
>         version: green # Now points to Green
>       ports:
>       - protocol: TCP
>         port: 80
>         targetPort: 80
>       type: LoadBalancer
>     ```
>     
>     ```basic
>     kubectl apply -f myapp-service.yaml
>     ```
>     
> 
> To rollback to Blue, just update the service selector back to the Blue version

### Canary Deployment: Gradual and Safe Application Updates

A strategy where a new version of an application is deployed to a small portion of users (e.g., 10%), while the rest continue using the old version. This allows you to test the new version in production with minimal risk. If it works well, you gradually increase its usage (e.g., 50%, then 100%) and If the canary version causes issues, you can quickly rollback and send all traffic back to the old version.

**Example :** Let's say you have version 1 of your app running and you want to canary release version 2

> * Existing Deployment (Version 1)
>     
>     ```yaml
>     apiVersion: apps/v1
>     kind: Deployment
>     metadata:
>       name: myapp-v1
>     spec:
>       replicas: 4
>       selector:
>         matchLabels:
>           app: myapp
>           version: v1
>       template:
>         metadata:
>           labels:
>             app: myapp
>             version: v1
>         spec:
>           containers:
>           - name: myapp
>             image: myapp:1.0
>             ports:
>             - containerPort: 80
>     ```
>     
> * **Canary Deployment (Version 2)**
>     
>     ```yaml
>     apiVersion: apps/v1
>     kind: Deployment
>     metadata:
>       name: myapp-canary
>     spec:
>       replicas: 1 # Canary version has only 1 replica
>       selector:
>         matchLabels:
>           app: myapp
>           version: canary
>       template:
>         metadata:
>           labels:
>             app: myapp
>             version: canary
>         spec:
>           containers:
>           - name: myapp
>             image: myapp:2.0
>             ports:
>             - containerPort: 80
>     ```
>     
>     ```basic
>     kubectl apply -f myapp-canary.yaml
>     ```
>     
> * **Service**: The service can be configured to split traffic between the two versions. Initially, it will send most traffic to version 1 (v1) and a small portion to the canary version
>     
>     ```yaml
>     apiVersion: v1
>     kind: Service
>     metadata:
>       name: myapp-service
>     spec:
>       selector:
>         app: myapp
>         version: v1
>       ports:
>       - protocol: TCP
>         port: 80
>         targetPort: 80
>       type: LoadBalancer
>     ```
>     
> * **Update Service**: Modify the service to route a small portion of traffic to the Canary deployment
>     
>     ```yaml
>     apiVersion: v1
>     kind: Service
>     metadata:
>       name: myapp-service
>     spec:
>       selector:
>         app: myapp
>       ports:
>       - protocol: TCP
>         port: 80
>         targetPort: 80
>       type: LoadBalancer
>     ```
>     
>     > You can use Kubernetes features like weighted routing in an Ingress controller, `to route a portion of traffic to the Canary` or manage it through a service mesh.
>     
> 
> * **Monitor Canary**: Check the performance and stability of the Canary version. If there are no issues, you can gradually increase the traffic directed to Canary.
>     
> * **Full Rollout**: Once confident in the Canary’s stability, update the service selector to point to the new version and increase the replicas of the Canary deployment while reducing the replicas of the Stable deployment.
>     
> * **Cleanup**: Remove the old stable deployment after the Canary version is fully rolled out and stable.
>     

**Canary deployments help ensure that new releases are stable and function correctly with minimal risk.**

### Conclusion

> In this article, we explored various aspects of Kubernetes, including ReplicaSets, Deployments, Probes, and deployment strategies like Blue-Green and Canary deployments. Understanding these concepts is crucial for managing and scaling applications efficiently in a Kubernetes environment. By leveraging these tools and strategies, you can ensure high availability, reliability, and seamless updates for your applications, ultimately enhancing your overall DevOps practices.
