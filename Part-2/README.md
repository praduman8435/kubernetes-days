# Kubernetes-part-2
<p align="center">
  <img src="https://cdn.hashnode.com/res/hashnode/image/stock/unsplash/jOqJbvo1P9g/upload/792a47f04af4ec8254a34c28a127e841.jpeg?w=1600&h=840&fit=crop&crop=entropy&auto=compress,format&format=webp">
</p>

### **Kubernetes tools for creating clusters**

* kubeadm
    
* kops
    
* ksctl
    

*when you create a cluster using these tools, it is called a self-managed Kubernetes cluster*

### **Some managed kubernetes cluster**

* EKS - AWS
    
* GKE - GCP
    
* Redhat - OpenShifts
    
* AKS - Azure
    
* DKE - Digital Ocean
    

*we generally create self-managed k8s cluster if we have our own servers*

### **Tools for running Kubernetes clusters in a local environment:**

* kind
    
* minikube
    
* rancher desktop
    
* docker desktop
    

### Understanding Kubernetes Architecture: Master and Worker Nodes

Kubernetes architecture is divided into master nodes and worker nodes, each with specific components that handle different tasks.

</p>
  <p align="center">
  <img src="https://cdn.hashnode.com/res/hashnode/image/upload/v1715781229351/06180e99-d0e6-4906-bc45-77f33cdd87f9.png">
</p>

#### **Master Node**

The master node is the control center of the Kubernetes cluster. It manages the cluster and coordinates all activities.

* **ETCD**: A key-value store that keeps all the cluster's data and configuration. It is crucial for maintaining the cluster's state.
    
* **Kube-API Server**: The main management point for the cluster. It handles requests from users, other components, and external agents, and updates the cluster's state.
    
* **Kube-Scheduler**: Assigns tasks to worker nodes based on available resources and needs, making sure resources are used efficiently.
    
* **Controller Manager**: Keeps the cluster in the desired state by managing various controllers, like the replication controller and endpoint controller.
    
* **Cloud Controller Manager**: Handles cloud-specific tasks. It allows Kubernetes to interact with cloud provider APIs for things like load balancers and storage.
    

#### **Worker Node**

Worker nodes run the applications. They handle the workload by running pods, which are the smallest deployable units in Kubernetes.

* **Kubelet**: An agent on each worker node that makes sure containers are running in pods as they should.
    
* **Kube-Proxy**: Manages network communication for the pods, ensuring proper routing within the cluster and to external networks.
    
* **Container Runtime Interface (CRI)**: The software that runs the containers, like Docker or containerd, managing their lifecycle.
    
* **Pod**: The smallest deployable unit in Kubernetes, consisting of one or more containers that share storage, network, and configuration. Pods are scheduled on worker nodes and managed by the kubelet.
    

### Step-by-Step Guide: K8s Installation Using KOPS on EC2

Create an [**EC2 instance**](https://itspraduman.hashnode.dev/how-to-connect-to-ec2-instance-easily-without-password) and connect it to your local machine or use your personal laptop.

Required dependencies:

1. Python3
    
2. AWS CLI
    
3. kubectl
    

**KOPS Installation**

```bash
curl -LO https://github.com/kubernetes/kops/releases/download/$(curl -s https://api.github.com/repos/kubernetes/kops/releases/latest | grep tag_name | cut -d '"' -f 4)/kops-linux-amd64

chmod +x kops-linux-amd64

sudo mv kops-linux-amd64 /usr/local/bin/kops
```

**Set up AWS CLI configuration on your EC2 Instance or Laptop.**

*Run* `aws configure`

### Kubernetes Cluster Installation

Follow these steps carefully and read each command before executing.

1. **Create an S3 bucket** to store KOPS objects:
    
    ```bash
    aws s3api create-bucket --bucket kops-abhi-storage --region us-east-1
    ```
    
2. **Create the cluster**:
    
    ```bash
    kops create cluster --name=demok8scluster.k8s.local --state=s3://kops-abhi-storage --zones=us-east-1a --node-count=1 --node-size=t2.micro --master-size=t2.micro --master-volume-size=8 --node-volume-size=8
    ```
    
3. **Edit the configuration**:
    
    > ***Important: Edit the cluster configuration as there are multiple resources created that won't fall into the free tier.***
    
    ```bash
    kops edit cluster demok8scluster.k8s.local
    ```
    
4. **Build the cluster**:
    
    ```bash
    kops update cluster demok8scluster.k8s.local --yes --state=s3://kops-abhi-storage
    ```
    
    > ***This will take a few minutes to complete.***
    
5. **Verify the cluster installation**:
    
    ```bash
    kops validate cluster demok8scluster.k8s.local
    ```
    

### **kubectl commands**

* **Nodes present in the cluster**
    
    ```bash
    kubectl get nodes
    ```
    
* **Create a pod**
    
    ```bash
    kubectl run nginx --image=nginx
    ```
    

---

### How kubectl Commands Work: Behind the Scenes

*when we give a command, we are generally communicate with the cluster*

**How we did that ?**

*using the config file, which are present inside the .kube folder on the controlplane*

```bash
cat .kube/config
```

### **How things work**

> *Let's say we need to create a pod using a command*
> 
> `kubectl run my-pod --image=nginx`

*when we want to do any work using kubectl command. firstly, they make a RestAPI call to API server. After that 3 more things happen :*

* Authentication
    
* Authorization
    
* Admission
    

> To ensure the request is valid and the person making the request is authorized to perform the task without violating any policies.

The scheduler looks for the best node based on taints/tolerations, affinity, and node selector to run the workload. Then, kubelet runs the workload and talks to CRI to pull the image and start it. The status is updated. Kube-proxy acts like iptables, and your workload runs. Health checks pass and inform the API server that the pod is running, and data is stored in ETCD.

**Inside kubeconfig file we have :**

* user information
    
* cluster information
    
* context information
    
</p>
  <p align="center">
  <img src="https://cdn.hashnode.com/res/hashnode/image/upload/v1715834899537/cda9db26-9dcd-4827-9751-413d89bbf63f.png">
</p>

* Inside context, we define which user is connected to which cluster.
    
* Inside users, we define the users present.
    
* Inside clusters, we define the clusters present.
    

> ***Command to view config file :***

```bash
kubectl config view
```

</p>
  <p align="center">
  <img src="https://cdn.hashnode.com/res/hashnode/image/upload/v1715835730812/b0fba319-dab0-4643-97d5-7f502d3f6f42.png">
</p>

> ***Command to view contexts :***

```bash
kubectl config get-contexts
```

</p>
  <p align="center">
  <img src="https://cdn.hashnode.com/res/hashnode/image/upload/v1715835865681/51224f0a-98f7-4865-bf42-a45ea5663d0b.png">
</p>

> ***Command to view the users :***

```bash
kubectl config get-users
```

</p>
  <p align="center">
  <img src="https://cdn.hashnode.com/res/hashnode/image/upload/v1718804912951/89e862ef-cfad-493c-a704-29f0ea8c0303.png">
</p>

---

### Creating a New User in Kubernetes: A Detailed Guide

* **Create a private key using openssl**
    
    ```bash
    openssl genrsa -out praduman.key 2048
    ```
    
    ```bash
    openssl req -new -key praduman.key -out praduman.csr -subj "/CN=praduman/O=group1"
    ```
    </p>
  <p align="center">
  <img src="https://cdn.hashnode.com/res/hashnode/image/upload/v1718807880743/21660be5-f5d9-4ca7-8e83-9da7e5a86f90.png">
</p>
    
* **Encode created CSR to base64**
    
    ```bash
    cat praduman.csr | base64 | tr -d '\n'
    ```
    </p>
  <p align="center">
  <img src="https://cdn.hashnode.com/res/hashnode/image/upload/v1715836303598/620fd344-b09f-450d-8e28-e8ba1d0ce201.png">
</p>
    
* **Create CSR**
    
    ```bash
    vi csr.yaml
    ```
    
    ```yaml
    apiVersion: certificates.k8s.io/v1
    kind: CertificateSigningRequest
    metadata:
        name: praduman
    spec:
        request: <BASE64-encoded CSR>
        signinerName: kubernetes.io/kube-apiserver-client
        usages:
        - client auth
    ```
    
    > Replace base64\_csr with the previous output
    
* **Apply yaml file and approve the certificate to the user**
    
    ```bash
    kubectl apply -f csr.yaml
    kubectl certificate approve praduman
    ```

    </p>
  <p align="center">
  <img src="https://cdn.hashnode.com/res/hashnode/image/upload/v1715838970607/bc6f7939-00bc-4917-9339-c5f5b039080e.png">
</p>
    
* **Get the crt specific to the user using jsonpath**
    
    ```bash
    kubectl get csr praduman -o jsonpath='{.status.certificate}' | base64 --decode > praduman.crt
    ```
    
    > a `praduman.crt` file is created
    
* **Create a role and role binding**
    
    ```basic
    vim role.yaml
    ```
    
    ```yaml
     kind: Role
     apiVersion: rbac.authorization.k8s.io/v1
     metadata:
       namespace: default
       name: pod-reader
     rules:
     - apiGroups: [""]
       resources: ["pods"]
       verbs: ["get", "watch", "list"]
    ---
     kind: RoleBinding
     apiVersion: rbac.authorization.k8s.io/v1
     metadata:
       name: read-pods
       namespace: default
     subjects:
     - kind: User
       name: praduman
       apiGroup: rbac.authorization.k8s.io
     roleRef:
       kind: Role
       name: pod-reader
       apiGroup: rbac.authorization.k8s.io
    ```
    
    ```basic
    kubectl apply -f role.yaml
    ```

    </p>
  <p align="center">
  <img src="https://cdn.hashnode.com/res/hashnode/image/upload/v1715839653601/48313cf1-ae3f-424e-8359-27aaef7b4693.png">
</p>
    
* **Set credentials**
    
    ```bash
    kubectl config set-credentials praduman --client-certificate=praduman.crt --client-key=praduman.key
    ```
    
* **Create context**
    
    ```bash
    kubectl config set-context praduman-context --cluster=kubernetes --namespace=default --user=praduman
    ```
    
* **view contexts**
    
    ```bash
    kubectl config get-contexts
    ```
    

> you will see a context with name praduman-context in the default namespace and kubernetes cluster

* **use the created context**
    
    ```bash
    kubectl config use-context praduman-context
    ```
    

> a new user is now created and now you can use it

---

### **How kubectl command works**

* Firstly, it searches for the kubeconfig file.
    
* If you set this variable in the current shell, then it first looks for the config file that you provide.
    
    ```bash
    export KUBECONFIG=path-to-your-config-file
    ```
    
* Another way to do this is to provide the kubeconfig file path along with the command.
    
    ```bash
    kubectl get pod --kubeconfig ~/.kube/config
    ```
    

> Let's say you have 3 clusters. Each cluster must have its own kubeconfig file. How will you merge all the config files?

* first method
    
    ```bash
    export KUBECONFIG=/path/to/first/config:/path/to/second/config:/path/to/third/config
    ```
    
* second method (use kubectx tool)
    
    > we use kubectx when we have multiple cluster, users and context for Faster way to switch between clusters (context) in kubectl
    
    ```bash
    # switch to another cluster that's in kubeconfig
    $ kubectx minikube
    Switched to context "minikube".
    
    # switch back to previous cluster
    $ kubectx -
    Switched to context "oregon".
    
    # rename context
    $ kubectx dublin=gke_ahmetb_europe-west1-b_dublin
    Context "gke_ahmetb_europe-west1-b_dublin" renamed to "dublin".
    ```
    

---

### **GVR (Group-Version-Resource)**

Think of GVR as the address for finding a type of resource in Kubernetes

* **Group**: A broad category (like a section in a library)
    
* **Version**: A specific edition within that category (like a book edition)
    
* **Resource**: The actual item you want (like a specific book)
    

**Example**: `apps/v1/deployments`

* **Group**: `apps` (the section for applications)
    
* **Version**: `v1` (the first edition of this section)
    
* **Resource**: `deployments` (the specific item you’re looking for, like a book on deployments)
    

### **GVK (Group-Version-Kind)**

GVK is similar, but it focuses on the type of item you’re working with

* **Group**: Same broad category
    
* **Version**: Same specific edition
    
* **Kind**: The specific type or class of the item
    

**Example**: `apps/v1/Deployment`

* **Group**: `apps`
    
* **Version**: `v1`
    
* **Kind**: `Deployment` (the exact type of item you’re dealing with, like a specific chapter in the book)
    

> * **GVR** tells you where to find the resource (`apps/v1/deployments`)
>     
> * **GVK** tells you exactly what the resource is (`apps/v1/Deployment`)
>     

---

### Communicate with the API server without using kubectl and kubeconfig

> *wanted to talk to kubernetes api server without kubectl or kubeconfig ?*
> 
> *we will talk to kubernetes directly through api server*

* **create a service account**
    
    ```bash
    kubectl create serviceaccount praduman -n default
    ```
    
* **create cluster role binding**
    
    ```bash
    kubectl create clusterrolebinding praduman-clusteradmin-binding --clusterrole=cluster-admin --serviceaccount=default:praduman
    ```
    
* **create a token**
    
    ```bash
    kubectl create token praduman
    ```
    
    ```bash
    Token=<output-of-the-above-cmd>
    ```
    
    ```bash
    APISERVER=$(kubectl config view --minify -o jsonpath='{.clusters[0].cluster.server}')
    ```
    
* **create deployment**
    
    ```bash
    curl -X POST $APISERVER/apis/apps/v1/namespaces/default/deployments \
      -H "Authorization: Bearer $TOKEN" \
      -H 'Content-Type: application/json' \
      -d @deploy.json \
      -k
    ```
    
* **list pods**
    
    ```bash
    curl -X GET $APISERVER/api/v1/namespaces/default/pods \
      -H "Authorization: Bearer $TOKEN" \
      -k
    ```
    

### Conclusion

> In conclusion, this article provides a detailed exploration of Kubernetes, covering its architecture, tools for creating and managing clusters, and practical steps for setting up a Kubernetes cluster using KOPS on EC2. It distinguishes between self-managed and managed Kubernetes clusters, highlights tools for local development, and delves into the components of master and worker nodes. Additionally, it offers commands and procedures for cluster creation, configuration, and user management, including direct interaction with the Kubernetes API server using REST calls. This comprehensive guide serves as a valuable resource for anyone looking to understand and implement Kubernetes effectively.
