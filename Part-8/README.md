# Kubernetes-part-8
<p align="center">
  <img src="https://cdn.hashnode.com/res/hashnode/image/stock/unsplash/jOqJbvo1P9g/upload/792a47f04af4ec8254a34c28a127e841.jpeg?w=1600&h=840&fit=crop&crop=entropy&auto=compress,format&format=webp">
</p>


### Understanding Kubernetes Services: A Comprehensive Guide

A Kubernetes Service is an object that helps you expose an application running in one or more Pods in your cluster. Since pod IP addresses can change when pods are created or destroyed, a service provides a stable IP that doesn't change. This ensures that both internal and external users can always connect to the right application, even if the pods behind it are constantly changing. By default the service that is created is `clusterIP`, it is useful for communication within the cluster.

* **To create a service (**`default: ClusterIP`**)**
    
    ```bash
    kubectl expose <deployment/pod-name> --port=80
    ```
    
* **To see the services**
    
    ```bash
    kubectl get svc -owide
    ```
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1727034767770/9198e3a6-10d1-472f-8b1f-009143034fe0.png align="center")
    

#### What Are Endpoints in Kubernetes?

Endpoints are objects that list the IP addresses and ports of the pods associated with a specific service. When you create a service in Kubernetes, it uses a selector to determine which pods it should communicate with. The Endpoints object automatically updates as pods are added or removed, ensuring that the service always knows where to send traffic. It play a crucial role in connecting services to the pods they manage.

> **Viewing Endpoints**
> 
> ```bash
> kubectl get endpoints <service-name>
> ```
> 
> ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1727034930881/8c184bd2-5f7c-4c36-80e9-53203a40394f.png align="center")

#### Exploring Different Types of Kubernetes Services

* ClusterIP
    
* NodePort
    
* LoadBalancer
    
* ExternalName
    
* Headless
    
* ExternalDNS
    

### Networking Fundamentals in Kubernetes: A Deep Dive

Inside each node, there's always a **veth**(`Virtual Ethernet`) **pair** for networking. When a pod runs, a special container called the **pause container** is also created. For example, if you create a pod with two containers, like **busybox** and **nginx**, there will actually be three containers: the two you defined and the pause container. The pod gets its own IP, and this IP is connected to an interface called **eth0**(`Ethernet interface`) inside the pod using `CNI`.

* Create a multi-container pod (`mcp.yml`)
    
    ```yaml
    apiVersion: v1
    kind: Pod
    metadata:
      name: shared-namespace
    spec:
      containers:
        - name: p1
          image: busybox
          command: ['/bin/sh', '-c', 'sleep 10000']
        - name: p2
          image: nginx
    ```
    
    ```bash
    kubectl apply -f mcp.yml
    ```
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1727035422081/02c5bbe4-15c2-4229-9829-76401385f3e6.png align="center")
    
* To see the pod’s IP
    
    ```bash
    kubectl get pod shared-namespace -owide
    ```
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1727035501759/932c5c16-6dd0-4461-9f7b-ebab8a292092.png align="center")
    
* Check the node where the pod is running and SSH into it
    
    ```bash
    ssh node01
    ```
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1727035555351/cb03278f-88d1-40a7-a2c7-587560689130.png align="center")
    
* To view the network namespaces created
    
    ```bash
    ip netns list
    ```
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1727035604459/eb73248a-0f9f-4c19-906f-df4b22663d57.png align="center")
    
* To find the pause container
    
    ```bash
    lsns | grep nginx
    ```
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1727035736678/89d8c4ba-3b5f-45c2-8bac-ad312301a9f4.png align="center")
    
* Get details of the pause container's namespaces (net, ipc, uts)
    
    ```bash
    lsns -p <PID-from-the-previous-command>
    ```
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1727036619786/dd3809c1-6b17-4746-8468-ce630f4354e0.png align="center")
    
* To check the list of all network namespaces
    
    ```bash
    ls -lt /var/run/netns
    ```
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1727035836018/33ba65eb-bb9f-4eb4-8ff7-daba9e073298.png align="center")
    
* Exec into the namespace or into the pod to see the ip link
    
    ```bash
    ip netns exec <namespace> ip link
    kubectl exec -it <pod-name> -- ip addr
    ```
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1727036968501/178cacd3-2285-4703-8e87-13c85fa98293.png align="center")
    
* Find the veth Pair
    
    * Once you run the above command, you may see an interface with a name like `eth@9`. The number after `@` represents the identifier of the virtual Ethernet (veth) pair interface.
        
    * To find the corresponding link on the Kubernetes node, you can search using this identifier. For example, if the number is `9`, run the following on the node
        
    
    ```bash
    ip link | grep -A1 ^9
    ```
    
    * This will show the details of the veth pair on the node
        
* **Inter Node communication**
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1727037359729/dcb73739-2675-44f4-bb1a-b9edafb5b36b.png align="center")
    

> Here, Traffic(packet) goes to `eth0` and then `veth1` acts as tunnel and traffic goes to `root namespace` and `bridge` resolve the destination address using the `ARP table` then `Veth1` send traffic(packet) to `pod B`. This all only happens if there is a single node

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1727037799205/fa47b17e-b08e-4357-bb56-fbef2aefbaac.png align="center")

### StatefulSet: Managing Stateful Applications in Kubernetes

It helps manage stateful applications where each pod needs a unique identity and keeps its own data. It makes sure pods are started, updated, and stopped one by one, in a set order. Each pod has a fixed name and can keep its data even if it is restarted or moved to another machine. This is useful for apps like databases, where data and order matter.

#### Key Differences Between Deployments and StatefulSets

| **Feature** | **Deployment** | **StatefulSet** |
| --- | --- | --- |
| **Use Case** | For stateless applications | For stateful applications |
| **Pod Identity** | Pods are interchangeable and do not have stable identities | Pods have unique, stable identities (e.g., `pod-0`, `pod-1`) |
| **Storage** | Typically uses ephemeral storage, data is lost when a pod is deleted | Each pod can have its own persistent volume attached |
| **Scaling Behavior** | Pods are scaled simultaneously and in random order | Pods are scaled sequentially (e.g., `pod-0` before `pod-1`) |
| **Pod Updates** | All pods can be updated concurrently | Pods are updated sequentially, ensuring one pod is ready before moving to the next |
| **Order of Pod Creation/Deletion** | No specific order in pod creation or deletion | Pods are created/deleted in a specific order (e.g., `pod-0`, `pod-1`) |
| **Network Identity** | Uses a ClusterIP service, no stable network identity | Typically uses a headless service, giving each pod a stable network identity |
| **Examples** | Microservices, stateless web apps | Databases (MySQL, Cassandra), distributed systems requiring unique identity or stable storage |
| **Use of Persistent Volumes** | Persistent volumes are shared across pods (if needed) | Each pod gets a dedicated persistent volume |

> The `Service` that is created in `StatefulSet` is the `ClusterIP: none` i.e., (`headless service`)

* **StatefulSet example for deploying a MySQL database in Kubernetes**
    
    ```yaml
    apiVersion: apps/v1
    kind: StatefulSet
    metadata:
      name: mysql
    spec:
      serviceName: "mysql-service"
      replicas: 3
      selector:
        matchLabels:
          app: mysql
      template:
        metadata:
          labels:
            app: mysql
        spec:
          containers:
          - name: mysql
            image: mysql:5.7
            ports:
            - containerPort: 3306
              name: mysql
            volumeMounts:
            - name: mysql-storage
              mountPath: /var/lib/mysql
      volumeClaimTemplates:
      - metadata:
          name: mysql-storage
        spec:
          accessModes: ["ReadWriteOnce"]
          resources:
            requests:
              storage: 1Gi
    ```
    
    * Each pod will have its own persistent storage (`mysql-storage`).
        
    * Pods will have stable network names (`mysql-0`, `mysql-1`, etc.), and their data will persist even if they restart.
        
* **Create a StatefulSet**
    
    ```yaml
    cat <<EOF | kubectl apply -f -
    apiVersion: apps/v1
    kind: StatefulSet
    metadata:
    name: postgres
    spec:
    serviceName: "postgres"
    replicas: 3
    selector:
    matchLabels:
    app: postgres
    template:
    metadata:
    labels:
    app: postgres
    spec:
    containers:
    - name: postgres
    image: postgres:13
    ports:
    - containerPort: 5432
    name: postgres
    env:
    - name: POSTGRES_PASSWORD
    value: "example"
    volumeMounts:
    - name: postgres-storage
    mountPath: /var/lib/postgresql/data
    volumeClaimTemplates:
    - metadata:
    name: postgres-storage
    spec:
    accessModes: [ "ReadWriteOnce" ]
    resources:
    requests:
    storage: 1Gi
    EOF
    ```
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1726909024059/d8691034-3b8f-4fdd-81ee-2ce4c0f3e793.png align="center")
    
* **A** `persistent volume` **and** `persistent volume claim` **is also created**
    
    ```bash
    kubectl get pv
    ```
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1726909331546/a1451085-b5ec-4629-8235-d3303b68232d.png align="center")
    
    ```bash
    kubectl get pvc
    ```
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1726909335672/57582e0d-5be9-4c19-a719-7cdb17f650ca.png align="center")
    
* **Check for pods that is created**
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1726909533722/5aa3f5db-9348-4eb3-8fc5-88f1b48bdaa0.png align="center")
    
* **Create a service (**`svc.yml`**)**
    
    ```yaml
    apiVersion: v1
    kind: Service
    metadata:
      name: postgres
      labels:
        app: postgres
    spec:
      ports:
      - port: 5432
        name: postgres
      clusterIP: None
      selector:
        app: postgres
    ```
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1726909682007/e8931793-58ac-4451-a44e-1d351bab5bf6.png align="center")
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1726909735841/b07da77b-02b0-4565-bbb6-c9152bb2f547.png align="center")
    
* **Increase the replicas and you will see the each pod have a ordered and fixed name**
    
    ```bash
    kubectl scale statefulset postgres --replicas=6
    ```
    
    ```bash
    kubectl get pod
    ```
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1726910106467/347b2175-fc31-485a-be0d-9f7247065427.png align="center")
    
* **If you delete a pod a new pod with the same name again created**
    
    ```bash
    kubectl delete pod postgres-3
    ```
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1726910233816/886c15f4-3136-4b00-84f6-3a39e65c5fcb.png align="center")
    
* **Check if the service is working or not**
    
    ```bash
    kubectl exec -it postgres-0 -- psql -U postgres
    ```
    

### NodePort: Exposing Your Kubernetes Application

A NodePort is a way to let people from outside the cluster access your app. It opens a specific port on every node in the cluster, allowing you to reach the app using the node's IP and the assigned port. This port is a number in the range `30000–32767`. You can access your application by visiting `<NodeIP>:<NodePort>`, where `NodeIP` is the IP address of any node in the cluster, and `NodePort` is the port assigned by Kubernetes.

#### Where it's useful ?

* **For testing and development**: You can quickly share access to apps without need of extra networking setup.
    
* **For small or internal projects**: If a company doesn't use cloud-based load balancers or advanced setups, NodePort is a simple solution to expose an app.
    

> YAML configuration for a NodePort service that exposes an nginx deployment or pod
> 
> ```yaml
> apiVersion: v1
> kind: Service
> metadata:
>   name: nginx-nodeport
>   labels:
>     app: nginx
> spec:
>   type: NodePort  # Specify the service type as NodePort
>   selector:
>     app: nginx  # This must match the labels of the nginx pods
>   ports:
>     - protocol: TCP
>       port: 80  # The port the service will listen on
>       targetPort: 80  # The port the nginx container is listening on
>       nodePort: 30008  # NodePort exposed (optional, Kubernetes can assign one if omitted)
> ```

#### Create a Pod and expose it using NodePort service

```bash
kubectl run nginx --image=nginx
kubectl expose pod nginx --type=NodePort --port 80
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1727105140410/67d640fd-661b-4317-92a9-efbf90edf1ce.png align="center")

**Access the Server** `http://192.168.1.4:32613`

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1727106216440/f2329250-44eb-4cd9-bfd7-fb0f112c25ad.png align="center")

* **Check NodePort and KUBE Rules**
    
    > to check the rules in the iptables firewall configuration related to Kubernetes NodePort services
    
    ```bash
    sudo iptables -t nat -L -n -v | grep -e NodePort -e KUBE
    ```
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1727106551462/6aff24b2-d087-4c72-bea0-7792b271cd4e.png align="center")
    
* **Check Specific NodePort Rules**
    
    ```bash
    sudo iptables -t nat -L -n -v | grep 32613
    ```
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1727106679408/43bfcd1c-f643-48d9-a327-ca797995c469.png align="center")
    
    > This indicates that traffic coming to port `32613` (your NodePort) is being redirected properly to the appropriate service.
    

### LoadBalancer: Making Your Application Public

It allows your application to be accessible to the public over the internet. When you create a service of this type, Kubernetes asks your cloud provider (like `AWS`, `GCP`, or `Azure`) to set up a load balancer automatically.

This service type is ideal when you want to expose an application, such as a web server, to users outside your cluster. The cloud provider provides an `external IP address`, which users can use to access your application.

The load balancer automatically distributes incoming traffic to all the pods running the service. This helps spread the traffic evenly and avoid overloading any single pod.

> **LoadBalancer YAML file**
> 
> ```yaml
> apiVersion: v1
> kind: Service
> metadata:
>   name: nginx-loadbalancer
> spec:
>   type: LoadBalancer  # This makes it a LoadBalancer service
>   selector:
>     app: nginx  # This selects the nginx pods
>   ports:
>     - protocol: TCP
>       port: 80         # Exposes port 80 (HTTP) to the public
>       targetPort: 80   # The port on the nginx container
> ```
> 
> #### How It Works:
> 
> 1. You create the **LoadBalancer** service.
>     
> 2. Kubernetes communicates with the cloud provider, and a load balancer is created.
>     
> 3. The load balancer assigns an **external IP address**.
>     
> 4. Traffic from the external IP is sent to your service, which distributes it to the nginx pods.
>     

* **Creating a LoadBalancer service in minikube**
    
    ```bash
    kubectl run nginx --image=nginx
    kubectl expose pod nginx --type=LoadBalancer --port=80
    ```
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1727110315954/0c508165-b196-45cb-9bbb-63878ab5d84b.png align="center")
    
* **Creates a network tunnel, making your LoadBalancer service accessible**
    
    ```bash
    minikube tunnel
    ```
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1727110572203/cd3f2ba4-6407-4359-ab56-fae09ddd2438.png align="center")
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1727110594870/e60f4c85-1994-427f-b561-8e461773bcc8.png align="center")
    
* **Access the Service at** `External-ip`
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1727110670807/563b2536-c0f8-4aa7-b79b-3dab581c0531.png align="center")
    

#### Why Avoid LoadBalancer in Production?

1. **Cost**: LoadBalancers can be expensive, as cloud providers charge for them.
    
2. **Scalability Problems**: They might struggle with sudden increases in traffic.
    
3. **Single Point of Failure**: If the LoadBalancer fails, all services using it can go down.
    
4. **Limited Flexibility**: They have fixed rules, making it hard to manage complex traffic needs.
    
5. **Performance Delays**: They can add extra time (latency) to how quickly users access your services.
    
6. **Dependence on Cloud Services**: If the cloud provider has issues, your services might become unavailable.
    
7. **Complex Setup**: Managing LoadBalancers can complicate your system, especially with multiple clusters.
    

#### Alternatives to LoadBalancer for Kubernetes

* **Ingress Controllers**: These are cheaper and allow more flexible traffic management.
    
* **Service Mesh**: They help manage traffic and improve observability without needing a LoadBalancer.
    

### ExternalName: Mapping Services to External DNS Names

The **ExternalName** service type is a special kind of service in Kubernetes that allows you to map a service to an external DNS name. Instead of using a cluster IP, it returns a `CNAME` record with the specified external name. We use it when we need to communicate to an external service (like `external API` or `database`) without changing your application code. Using this we can also communicate with services that is present in `other namespaces`.

> Example: Two services within two other namespaces communicate with each other
> 
> Use [`This file`](https://github.com/saiyam1814/Kubernetes-hindi-bootcamp/tree/main/part8/ExternalName) to get the Sourcecode

* Create two namespace
    
    ```bash
    kubectl create ns database-ns
    kubectl create ns application-ns
    ```
    
* Create the database pod and service
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1727115368577/3a5fc1d0-0a0b-491e-9e2e-0a64fd83ca0b.png align="center")
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1727115377815/cfdda7e6-a74b-4410-aa5b-ae1f0ac27e63.png align="center")
    
    ```bash
    kubectl apply -f db.yaml
    kubectl apply -f db_svc.yaml
    ```
    
* Create ExternalName service
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1727115300028/65f8c022-4f09-4066-bb27-fe42011993bd.png align="center")
    
    ```bash
    kubectl apply -f externam-db_svc.yaml
    ```
    
* Create Application to access the service Docker build
    
    ```bash
    docker build --no-cache --platform=linux/amd64 -t ttl.sh/saiyamdemo:1h .
    ```
    
* Create the pod
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1727115974019/eccd8060-429f-4b69-9f92-f894b4c205fb.png align="center")
    
    ```bash
    kubectl apply -f apppod.yaml
    ```
    
* Check the pod logs to see if the connection was successful
    
    ```bash
    kubectl logs my-application -n application-ns
    ```
    

### Ingress: Controlling User Access to Kubernetes Applications

**It** is a way to control how users access your applications running in Kubernetes. Think of it as a gatekeeper that directs traffic to the right place. To use ingress we firstly need to deploy an `Ingress Controller` on the cluster and the Ingress are implemented through `Ingress Controller`. Ingress controllers are the open source project of many companies such as `NGINX`, `Traefik`, `Istio Ingress`, etc.

Now, we create a service using `ClusterIP` and a resource using `Ingress`. User request firstly goes to `ingress controller` then ingress controller send it to `ingress` then after it goes to the `clusterIP` service configured by us and at last it goes to the pod. Here we deploy `Ingress Controller` as the `LoadBalancer`.

#### Why Use Ingress?

1. **Single Address**: Instead of having different addresses for each app, you can use one address for everything.
    
2. **Routing by URL**: You can send users to different apps based on the URL they visit. For example, `/app1` goes to one app and `/app2` goes to another.
    
3. **Secure Connections**: Ingress can handle secure connections (HTTPS) easily, so your apps are safe to use.
    
4. **Balancing Traffic**: It spreads out incoming traffic evenly, so no single app gets overloaded.
    

> #### Example
> 
> Imagine you have two apps:
> 
> * A website
>     
> * An API
>     
> 
> You can set up Ingress to route traffic like this:
> 
> * If someone goes to `/`, they get the website.
>     
> * If they go to `/api`, they get the API.
>     
> 
> Here’s how you might set it up:
> 
> ```yaml
> apiVersion: networking.k8s.io/v1
> kind: Ingress
> metadata:
>   name: example-ingress
> spec:
>   rules:
>   - host: "my-app.com"
>     http:
>       paths:
>       - path: /
>         pathType: Prefix
>         backend:
>           service:
>             name: website-service
>             port:
>               number: 80
>       - path: /api
>         pathType: Prefix
>         backend:
>           service:
>             name: api-service
>             port:
>               number: 80
> ```

* Create a deployment (`deploy.yaml`)
    
    ```yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
       name: nginx-deployment
    spec:
       replicas: 1
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
               volumeMounts:
               - name: config-volume
                 mountPath: /etc/nginx/nginx.conf
                 subPath: nginx.conf  # Ensure this matches the filename in the ConfigMap
               volumes:   
               - name: config-volume
                 configMap:
                 name: nginx-config
    ```
    
    ```yaml
    kubectl apply -f deploy.yaml
    ```
    
* Create a `clusterIP` service (svc.yaml)
    
    ```yaml
    apiVersion: v1
    kind: Service
    metadata:
       name: nginx-service
    spec:
       selector:
          app: nginx
        ports:
        - protocol: TCP
          port: 80
          targetPort: 80
    ```
    
    ```bash
    kubectl apply -f svc.yaml
    ```
    
* verify the service is created
    
    ```bash
    kubectl get svc
    ```
    
* Create a nginx.conf file
    
    ```nginx
    user  nginx;
    worker_processes  auto;
    
    error_log  /var/log/nginx/error.log notice;
    pid        /var/run/nginx.pid;
    
    events {
    worker_connections  1024;
    }
    
    http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;
    
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    '$status $body_bytes_sent "$http_referer" '
    '"$http_user_agent" "$http_x_forwarded_for"';
    
    access_log  /var/log/nginx/access.log  main;
    
    sendfile        on;
    #tcp_nopush     on;
    
    keepalive_timeout  65;
    
    #gzip  on;
    
    server {
    listen       80;
    server_name  localhost;
    
    location / {
    root   /usr/share/nginx/html;
    index  index.html index.htm;
    }
    
    location /public {
    return 200 'Access to public granted!';
    }
    
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
    root   /usr/share/nginx/html;
    }
    }
    }
    ```
    

* Create a ConfigMap
    
    ```bash
    kubectl create configmap nginx-config --from-file=nginx.conf
    ```
    
* Access the server
    
    ```nginx
    curl <cluster-IP>
    ```
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1727124231790/d638df6e-1f6e-4d01-9356-25a8f724fb9e.png align="center")
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1727124365615/d06b9753-85f6-401b-b263-1fbcfa5c948c.png align="center")
    

> Now we need to access this application from outside the cluster without using `NodePort` and `LoadBalancer` service

* Install `Ingress controller` first
    
    ```basic
    kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.9.4/deploy/static/provider/cloud/deploy.yaml
    ```
    
* Create a Ingress object (`ingress.yaml`)
    
    ```yaml
    apiVersion: networking.k8s.io/v1
    kind: Ingress
    metadata:
      name: bootcamp
    spec:
      ingressClassName: nginx
      rules:
      - host: "kubernetes.hindi.bootcamp"
        http:
          paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: nginx-service
                port:
                  number: 80
          - path: /public 
            pathType: Prefix
            backend:
              service:
                name: nginx-service
                port:
                  number: 80
    ```
    
* ssh onto the node where the pod is deployed and the change the /etc/hosts file
    
    ```yaml
    # add this in file
    <node-internal-IP> <nodeName>
    <node-internal-IP> kubernets.hindi.bootcamp
    ```
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1727125797335/802cb4be-a2a6-4caa-bb42-15cb1b0c93e2.png align="center")
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1727125853427/5a725d7c-30cd-40cf-94a6-aac359eb74fa.png align="center")
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1727125857934/df63eef1-53c5-40fd-8ecf-98bc74932dd9.png align="center")
    
* Now apply the create `ingress.yaml` file
    
    ```bash
    kubectl apply -f ingress.yaml
    ```
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1727126034264/5a8bdf43-49c9-43d5-a028-114ee8737bfa.png align="center")
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1727126080194/8b92f224-6457-40d9-a7f5-bd03f0d10ed7.png align="center")
    
* We need to connect user to the ingress controller
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1727126383167/a47e632d-cba7-4ba2-a5f4-7e8637e1d57b.png align="center")
    
    ```bash
    curl kubernetes.hindi.bootcamp:30418
    ```
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1727126520617/82ac228a-368b-422b-ae8b-cb4a40924f6c.png align="center")
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1727126576495/3011b087-b01a-4637-a274-34e5472ebabf.png align="center")
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1727126869944/05776b87-6b71-4798-8d8e-0e1f8d56f639.png align="center")
    

### ExternalDNS: Connecting Kubernetes to DNS Providers

It connects your Kubernetes cluster to a DNS provider (like AWS Route53, Google Cloud DNS, or Cloudflare) and updates DNS records when you deploy new services or ingresses. This makes your apps available under a specific domain name.

#### **How it works:**

* **Watch Kubernetes Resources:** ExternalDNS continuously monitors Kubernetes resources (like `Service`, `Ingress`, or `Endpoint` resources) for changes.
    
* **Determine Desired DNS Records:** Based on the services or ingress objects, it calculates the required DNS records.
    
* **Update DNS Provider:** It then communicates with a DNS provider (like AWS Route53, Google Cloud DNS, or Cloudflare) to create, update, or delete DNS records as needed.
    

#### Use Cases

* Automatically manage DNS entries for services or applications exposed via Kubernetes Ingress or LoadBalancer services.
    
* Keep DNS records up-to-date when service IPs or endpoints change.
    
* Use annotations to control which resources should have DNS records managed by ExternalDNS.
    

### Conclusion

> In this article, we explored various aspects of Kubernetes, including services, networking fundamentals, StatefulSets, and different types of services like NodePort, LoadBalancer, ExternalName, Ingress, and ExternalDNS. We delved into the specifics of how each service type functions, their use cases, and provided practical examples to illustrate their implementation. Understanding these components is crucial for effectively managing and deploying applications in a Kubernetes environment. By mastering these concepts, you can ensure your applications are scalable, resilient, and efficiently managed within your Kubernetes clusters.