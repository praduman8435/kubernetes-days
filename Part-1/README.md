# Kubernetes-Part-1

### What is Kubernetes ?

Kubernetes, also known as k8s, is an open-source system that helps automate the deployment, scaling, and management of applications in containers. It is a container orchestration platform.

When you install Kubernetes, you create a cluster, which is a group of nodes. Nodes are physical or virtual machines that helps to run your containers.

***Virtual machines can also be orchestrated through Kubernetes by installing KubeVirt.***

*Kubernetes are:*

* **CNCF Graduated Project**
    
* **Inspired by Borg and Omega**
    
* **Launched in 2014**
    
* **Designed for Scale**
    
* **Run Anywhere**
    

### Why Kubernetes?

Kubernetes is popular because it offers powerful features:

1. **Autoscaling**: Automatically adjusts the number of running containers based on current demand, ensuring efficient resource usage and optimal performance.
    
2. **Autohealing**: Detects and replaces failed containers to maintain the health and availability of applications.
    
3. **Scheduling**: Efficiently allocates containers to nodes based on resource requirements and constraints, optimizing resource usage.
    
4. **Load Balancing**: Distributes network traffic across multiple containers to ensure no single container is overwhelmed.
    
5. **Storage Orchestration**: Automatically mounts the necessary storage systems, whether local, cloud-based, or network storage.
    
6. **Deployment Automation**: Automates the deployment, scaling, and rollback of applications, ensuring smooth updates and maintaining high availability.
    
7. **Monitoring and Logging Integration**: Provides integration with various tools for monitoring and logging, offering insights into application performance and health.
    

### Docker and Its Limitations

Docker is a platform for running containers, which are lightweight and temporary by nature. Here are some key issues with Docker:

* **Ephemeral Nature**: Containers have short lifespans. If a container stops or crashes, the application inside becomes inaccessible until someone restarts it.
    
* **Single Host Limitation**: Docker manages containers on a single machine, making it difficult to handle large-scale deployments.
    
* **Manual Monitoring**: Monitoring and managing a large number of containers manually is impractical. Running commands like `docker ps` to check container states isn't feasible at scale.
    
* **Handling Traffic**: When a container experiences a surge in traffic, Docker lacks built-in mechanisms to automatically distribute the load or scale up the number of containers, leading to potential performance issues.
    

### How Kubernetes Helps

Kubernetes addresses these problems with features like:

* **Autohealing**: Automatically detects and restarts failed containers, ensuring applications remain accessible without manual intervention.
    
* **Autoscaling**: Automatically adjusts the number of running containers based on traffic demand, ensuring that applications can handle increased traffic without performance degradation.
    
* **Multi-Host Orchestration**: Manages containers across multiple machines, enabling large-scale deployments and better resource utilization.
    

---

### Creation of a container using docker:

*firstly make sure you have installed Docker on your server*

* ***Pull nginx image from dockerHub on your local.***
    
    ```basic
    docker run -d --name my-nginx-container --memory 512m --cpus 1 nginx
    ```
    <p align="center">
  <img src="https://cdn.hashnode.com/res/hashnode/image/upload/v1715754053278/bb8d27ac-2bfd-4d7e-87b2-209643527e60.png">
</p>
    
    
* ***check image are runningm***
    
    ```bash
    docker ps
    ```
    <p align="center">
  <img src="https://cdn.hashnode.com/res/hashnode/image/upload/v1715754315784/a5e8f1b9-f7a9-4c89-ac3f-0d3dd76cb564.png">
</p>

    
* ***print process id (PID) of container nginx***
    
    ```bash
    docker inspect --format '{{.State.Pid}}' my-nginx-container
    
    (or)
    
    ps aux | grep '[n]ginx' | sort -n -k 2 | head -n 1 | awk '{print $2}'
    ```
    <p align="center">
  <img src="https://cdn.hashnode.com/res/hashnode/image/upload/v1718737283139/a64e64cc-6086-40fe-87ca-dafb1db70f6b.png">
</p>
    
* ***list namespaces of a process of linux***
    
    ```bash
    lsns -p <PID>
    ```
    <p align="center">
  <img src="https://cdn.hashnode.com/res/hashnode/image/upload/v1718737355110/1494f481-f5b6-4e7d-801d-a3ace6b7e108.png">
</p>
    

---

### Summary

> Kubernetes, or k8s, is an open-source container orchestration platform that automates deployment, scaling, and management of applications. It offers features like autoscaling, autohealing, efficient scheduling, load balancing, storage orchestration, deployment automation, and monitoring integration. Unlike Docker, which manages containers on a single host and requires manual intervention for scaling and monitoring, Kubernetes handles multi-host orchestration and automatically manages container health and scaling. The article also provides steps for creating a container using Docker.
