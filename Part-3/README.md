# Kubernetes part-3

```bash
kubectl get nodes
```

application runs as a container inside pods

* imperative way (using cli only)
    
* declarative way (using yaml file)
    

### **YAML**

Yaml ain't markup language

* object
    
* list
    

### **POD**

* how to create it in imperative way
    

```yaml
kubectl run <name-of-pod> --image=<image-name>
```

* yaml file for pod
    

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: example-pod
  labels:
    purpose: demonstrate-args
spec:
  containers:
  - name: example-container
    image: ubuntu
    command: ["/bin/echo", "Hello"]  
    args: ["Welcome", "to", "Kubesimplify"]
```

* CIDR range of node
    

```bash
kubectl get node <node-name> -oyaml | grep CIDR
```

* see pods
    

```bash
kubectl get pods
```

* create a namespace
    

```bash
kubectl create ns john
```

* view all namespaces
    

```bash
kubectl get ns
```

* describe all information of a pod
    

```bash
kubectl describe pod <name-of-pod>
```

```bash
kubectl get pods -owide
```

### **init container**

A pod can have multiple containers running apps within it, but it can also have one or more init container, which are run before the app containers are started.

* init containers always run to completion
    
* each init containers must complete successfully before the next one starts
    
* if a pod init container fails, the kubelet repeatedly restart that init container untill it succeeds. However, if the pod has a restart policy of never, and an init container fails during startup of the pod, kubernetes treats overall pod as failed.
    
* yaml file of pod with init container
    

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: bootcamp-pod
spec:
  volumes:
  - name: shared-data
    emptyDir: {}
  initContainers:
  - name: bootcamp-init
    image: busybox
    command: ['sh', '-c', 'wget -O /usr/share/data/index.html http://kubesimplify.com']
    volumeMounts:
    - name: shared-data
      mountPath: /usr/share/data
  containers:
  - name: nginx
    image: nginx
    volumeMounts:
    - name: shared-data
      mountPath: /usr/share/nginx/html
```

* multiple init container
    

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: init-demo-2
spec:
  initContainers:
  - name: check-db-service
    image: busybox
    command: ['sh', '-c', 'until nslookup db.default.svc.cluster.local; do echo waiting for db service; sleep 2; done;']
  - name: check-myservice
    image: busybox
    command: ['sh', '-c', 'until nslookup myservice.default.svc.cluster.local; do echo waiting for db service; sleep 2; done;']
  containers:
  - name: main-container
    image: busybox
    command: ['sleep', '3600']
```

* create services
    

```yaml
---
apiVersion: v1
kind: Service
metadata:
  name: db
spec:
  selector:
    app: demo1
  ports:
  - protocol: TCP
    port: 3306
    targetPort: 3306
---
apiVersion: v1
kind: Service
metadata:
  name: myservice
spec:
  selector:
    app: demo2
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
```

### **Sidecar container**

Kubernetes implements sidecar containers as a special case of init containers, sidecar containers remain running after Pod startup.

Sidecar containers are the secondary containers that run along with the main application container within the same Pod. These containers are used to enhance or to extend the functionality of the primary *app container* by providing additional services, or functionality.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: multi-container-pod
spec:
  volumes:
  - name: shared-data
    emptyDir: {}
  initContainers:
  - name: meminfo-container
    image: alpine
    restartPolicy: Always
    command: ['sh', '-c', 'sleep 5; while true; do cat /proc/meminfo > /usr/share/data/index.html; sleep 10; done;']
    volumeMounts:
    - name: shared-data
      mountPath: /usr/share/data
  containers:
  - name: nginx-container
    image: nginx
    volumeMounts:
    - name: shared-data
      mountPath: /usr/share/nginx/html
```
