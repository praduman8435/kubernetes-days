# Kubernetes-part-7
<p align="center">
  <img src="https://cdn.hashnode.com/res/hashnode/image/stock/unsplash/jOqJbvo1P9g/upload/792a47f04af4ec8254a34c28a127e841.jpeg?w=1600&h=840&fit=crop&crop=entropy&auto=compress,format&format=webp">
</p>

### Understanding ConfigMap: Simplify Your Kubernetes Configuration Management

It is used to store non-confidential configuration data in key-value pairs. It keep configuration separate from the application code, that makes it easier to manage and update without redeploying the entire application

> #### **Example 1: Passing Database User to MySQL Pod Using ConfigMap**

* **Create the ConfigMap (cm1.yaml)**: The ConfigMap will contain the database user and password
    
    ```yaml
    # cm1.yaml
    apiVersion: v1
    kind: ConfigMap
    metadata:
      name: bootcamp-configmap
    data:
      username: "saiyam"
      database_name: "exampledb"
    ```
    
    ```basic
    kubectl apply -f cm1.yaml
    ```
    
* **Create the Pod (pod1.yaml)**: The Pod will run a MySQL container and use the environment variables from the ConfigMap
    
    ```yaml
    # pod1.yaml
    apiVersion: v1
    kind: Pod
    metadata:
      name: mysql-pod
    spec:
      containers:
      - name: mysql
        image: mysql:5.7
        env:
          - name: MYSQL_USER
            valueFrom:
              configMapKeyRef:
                name: bootcamp-configmap
                key: username
          - name: MYSQL_DATABASE
            valueFrom:
              configMapKeyRef:
                name: bootcamp-configmap
                key: database_name
          - name: MYSQL_PASSWORD
            value: demo123  # Specify a strong password.
          - name: MYSQL_ROOT_PASSWORD
            value: demo345 # You should change this value.
        ports:
          - containerPort: 3306
            name: mysql
        volumeMounts:
          - name: mysql-storage
            mountPath: /var/lib/mysql
      volumes:
        - name: mysql-storage
          emptyDir: {}
    ```
    
    ```basic
     kubectl apply -f pod1.yaml
    ```
    
* **Access the MySQL Pod**: Once the Pod is running, connect to the MySQL container and verify the user setup
    
    ```basic
    kubectl exec -it mysql-pod -- mysql -u root -p
    ```
    
    > If asks for `password` , Use that defined in the `ConfigMap`
    
* **Run MySQL Commands**: Inside the MySQL prompt, run the following commands to verify the users and databases:
    
    ```sql
    SELECT user FROM mysql.user;
    SHOW DATABASES;
    ```
    

> #### **Example 2: Managing Dev/Prod Properties with ConfigMap**

* Create the ConfigMap (cm2.yaml)
    
    ```yaml
    apiVersion: v1
    kind: ConfigMap
    metadata:
      name: app-config-dev
    data:
      settings.properties: |
        # Development Configuration
        debug=true
        database_url=http://dev-db.example.com
        featureX_enabled=false
    
    ---
    
    apiVersion: v1
    kind: ConfigMap
    metadata:
      name: app-config-prod
    data:
      settings.properties: |
        # Production Configuration
        debug=false
        database_url=http://prod-db.example.com
        featureX_enabled=true
    ```
    
    ```basic
    kubectl apply -f cm2.yaml
    ```
    
* Create the Pod (pod2.yaml)
    
    ```yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: my-web-app
    spec:
      replicas: 1
      selector:
        matchLabels:
          app: my-web-app
      template:
        metadata:
          labels:
            app: my-web-app
        spec:
          containers:
          - name: web-app-container
            image: nginx  
            ports:
            - containerPort: 80
            env:
            - name: ENVIRONMENT
              value: "development"  
            volumeMounts:
            - name: config-volume
              mountPath: /etc/config
          volumes:
          - name: config-volume
            configMap:
              name: app-config-dev
    ```
    
    ```yaml
    kubectl apply -f pod2.yaml
    ```
    
* **Verify the ConfigMap is Mounted**: After the Pod is running, you can check if the `settings.properties` file has been correctly mounted and read its contents
    
    ```yaml
    kubectl exec -it app-pod -- cat /etc/config/settings.properties
    ```
    
    This should display the content of the `settings.properties` file:
    
    ```yaml
    # Development Configuration
    debug=true
    database_url=http://dev-db.example.com
    featureX_enabled=false
    ```
    
* **Switch to Production**: To use the **production** configuration instead, update the Deployment to mount `app-config-prod`.
    
    Change this part in `pod2.yaml`:
    
    ```yaml
    configMap:
      name: app-config-prod
    ```
    
    **Reapply the deployment**
    
    ```yaml
    kubectl apply -f pod2.yaml
    ```
    
    Now, if you check the configuration inside the pod, you’ll see the production settings:
    
    ```yaml
    # Production Configuration
    debug=false
    database_url=http://prod-db.example.com
    featureX_enabled=true
    ```
    

> #### **Example 3: Accessing ConfigMap Programmatically with Python**

* **read\_config.py:** This Python script reads and prints the contents of the `app-config` ConfigMap in the `default` namespace
    
    ```python
    from kubernetes import client, config
    
    def main():
        # Load the Kubernetes configuration
        config.load_incluster_config()
    
        v1 = client.CoreV1Api()
        config_map_name = 'app-config'
        namespace = 'default'
    
        try:
            # Read the ConfigMap
            config_map = v1.read_namespaced_config_map(config_map_name, namespace)
            print("ConfigMap data:")
            for key, value in config_map.data.items():
                print(f"{key}: {value}")
        except client.exceptions.ApiException as e:
            print(f"Exception when calling CoreV1Api->read_namespaced_config_map: {e}")
    
    if __name__ == '__main__':
        main()
    ```
    
* **Dockerfile:** This Dockerfile builds the image that runs the Python script.
    
    ```bash
    # Use a lightweight Python image
    FROM python:3.8-slim
    
    # Install the Kubernetes Python client
    RUN pip install kubernetes
    
    # Copy the Python script
    COPY read_config.py /read_config.py
    
    # Set the command to run the script
    CMD ["python", "/read_config.py"]
    ```
    
* **Build the Docker Image**: Build the Docker image using the provided `Dockerfile` and push it to a public image registry (in this case, `ttl.sh`)
    
    ```bash
    docker build -t ttl.sh/hindi-boot:1h .
    docker push ttl.sh/hindi-boot:1h
    ```
    
* **app.yaml**: This YAML file defines all the Kubernetes resources: the ConfigMap, Deployment, Role, and RoleBinding.
    
    ```yaml
    # ConfigMap storing some example properties
    apiVersion: v1
    kind: ConfigMap
    metadata:
      name: app-config
      namespace: default
    data:
      example.property: "Hello, world!"
      another.property: "Just another example."
    ---
    # Deployment to run the Python script in a pod
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: config-reader-deployment
      namespace: default
    spec:
      replicas: 1
      selector:
        matchLabels:
          app: config-reader
      template:
        metadata:
          labels:
            app: config-reader
        spec:
          containers:
          - name: config-reader
            image: ttl.sh/hindi-boot:1h
            imagePullPolicy: Always
    ---
    # Role granting read access to ConfigMaps in the default namespace
    apiVersion: rbac.authorization.k8s.io/v1
    kind: Role
    metadata:
      namespace: default
      name: config-reader
    rules:
    - apiGroups: [""]
      resources: ["configmaps"]
      verbs: ["get", "list", "watch"]
    ---
    # RoleBinding to attach the Role to the default ServiceAccount
    apiVersion: rbac.authorization.k8s.io/v1
    kind: RoleBinding
    metadata:
      name: read-configmaps
      namespace: default
    subjects:
    - kind: ServiceAccount
      name: default 
      namespace: default
    roleRef:
      kind: Role
      name: config-reader
      apiGroup: rbac.authorization.k8s.io
    ```
    
* **Apply the Kubernetes Resources**: Deploy the **ConfigMap**, **Deployment**, and **RBAC** resources to Kubernetes.
    
    ```basic
    kubectl apply -f app.yaml
    ```
    
* **Check the Logs**: After the pod is running, you can check the logs to see if the ConfigMap has been read successfully.
    
    ```basic
    kubectl logs -l app=config-reader
    ```
    
    The output should display the content of the **ConfigMap**
    
    ```basic
    ConfigMap data:
    example.property: Hello, world!
    another.property: Just another example.
    ```
    

### Secrets in Kubernetes: Securely Manage Sensitive Information

When deploying applications in Kubernetes, you often need to handle sensitive information such as passwords, API keys, or tokens. Storing these directly in your code or configuration files can be insecure. **Kubernetes Secrets** provide a way to securely store and manage sensitive information.

Unlike ConfigMaps (which store non-sensitive configuration data), Secrets are designed for security. They store data in an encoded format (Base64) and can be mounted into Pods or accessed as environment variables.

#### **Types of Kubernetes Secrets: Opaque, TLS, and Docker Registry**

* **Opaque Secret**: General-purpose secret, commonly used for storing credentials.
    
* **TLS Secret**: Specifically for storing SSL/TLS certificates.
    
* **Docker Registry Secret**: For storing Docker registry credentials, used for pulling private images.
    

#### **Example 1: Creating a Kubernetes Secret Using YAML**

> * **Encode your values** in Base64
>     
>     ```bash
>     echo -n 'my-db-username' | base64
>     echo -n 'my-db-password' | base64
>     ```
>     
>     This will return
>     
>     ```bash
>     bXktZGItdXNlcm5hbWU=
>     bXktZGItcGFzc3dvcmQ=
>     ```
>     
> * **Create the Secret YAML** file (`secret.yaml`)
>     
>     ```yaml
>     apiVersion: v1
>     kind: Secret
>     metadata: 
>       name: db-secret
>     type: opaque
>     data: 
>       username: bXktZGItdXNlcm5hbWU=
>       password: bXktZGItcGFzc3dvcmQ=
>     ```
>     
>     Apply the secret
>     
>     ```bash
>     kubectl apply -f secret.yaml
>     ```
>     
> * To view the secret
>     
>     ```bash
>     kubectl get secret
>     ```
>     
> * To decrypt the encrypted data
>     
>     ```bash
>     echo "put-encrypted-data" | base64 -d
>     ```
>     

#### **Example 2: Creating a Kubernetes Secret from the Command Line**

```bash
kubectl create secret generic db-secret \
  --from-literal=username=my-db-username \
  --from-literal=password=my-db-password
```

#### **Using Secrets in a Pod: Environment Variables and Volume Mounts**

Once the Secret is created, you can use it in a Pod as either environment variables or mounted as files

* #### Example: Using a Secret as Environment Variables
    
    ```yaml
    apiVersion: v1
    kind: Pod
    metadata:
      name: db-app
    spec:
      containers:
      - name: app-container
        image: my-app:latest
        env:
        - name: DB_USERNAME
          valueFrom:
            secretKeyRef:
              name: db-secret
              key: username
        - name: DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: db-secret
              key: password
    ```
    
    > Here, the **username** and **password** from the Secret are injected into the container as environment variables
    
* #### **Example: Mounting a Secret as a Volume**
    
    ```yaml
    apiVersion: v1
    kind: Pod
    metadata:
      name: app-with-secret
    spec:
      containers:
      - name: app-container
        image: my-app:latest
        volumeMounts:
        - name: secret-volume
          mountPath: /etc/secrets
          readOnly: true
      volumes:
      - name: secret-volume
        secret:
          secretName: db-secret
    ```
    
    > This will mount the Secret’s data into `/etc/secrets/` inside the container
    

#### **Storing SSH Private Keys in Kubernetes Secrets**

```basic
kubectl create secret generic my-ssh-key-secret \
--from-file=ssh-privatekey=/path/to/.ssh/id_rsa \
--type=kubernetes.io/ssh-auth
```

> `--from-file=ssh-privatekey=/path/to/.ssh/id_rsa`:
> 
> * `--from-file`: Specifies that the content for the Secret should come from a file.
>     
> * `ssh-privatekey`: The key under which the private key will be stored in the Secret.
>     
> * `/path/to/.ssh/id_rsa`: Path to the actual SSH private key file (usually located in `~/.ssh/id_rsa`).
>     
> 
> `--type=kubernetes.io/ssh-auth`:
> 
> * This specifies that the Secret is of type `kubernetes.io/ssh-auth`, which is used for SSH authentication.
>     

#### **Managing SSL/TLS Certificates with Kubernetes TLS Secrets**

```basic
kubectl create secret tls my-tls-secret \
--cert=path/to/cert/file \
--key=path/to/key/file
```

> `--cert=path/to/cert/file`:
> 
> * This specifies the path to the TLS certificate file. Replace `path/to/cert/file` with the actual path to your `.crt` (certificate) file.
>     
> 
> `--key=path/to/key/file`:
> 
> * This specifies the path to the TLS private key file. Replace `path/to/key/file` with the actual path to your private key file (typically `.key`).
>     

#### **MySQL Example: Combining ConfigMap and Secrets in Kubernetes**

* **create a configmap** `(myconfig.yaml)`
    
    ```yaml
    apiVersion: v1
    kind: configMap
    metadata:
       name: bootcamp-configMap
    data: 
       username: "praduman"
       database_name: "my-db"
    ```
    
    ```bash
    kubectl apply -f myconfig.yaml
    ```
    
* **create secrets** `(mysecret.yaml)`
    
    ```bash
    echo -n 'rootPass' | base64
    echo -n 'userPass' | base64
    ```
    
    ```yaml
    apiVersion: v1
    kind: Secret
    metadata: 
       name: mysql-root-pass
    type: Opaque
    data: 
       password: <use-base64-encoded>
    ---
    apiVersion: v1
    kind: Secret
    metadata: 
       name: mysql-user-pass
    type: Opaque
    data: 
       password: <use-base64-encoded>
    ```
    
    ```bash
    kubectl apply -f mysecret.yaml
    ```
    
* **YAML file of** `deployment` **using both** `configMap` **and** `Secret`**(deploy.yaml)**
    
    ```yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: mysql
    spec:
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
            env:
              - name: MYSQL_ROOT_PASSWORD
                valueFrom:
                  secretKeyRef:
                    name: mysql-root-pass
                    key: password
              - name: MYSQL_USER
                valueFrom:
                  configMapKeyRef:
                    name: bootcamp-configmap
                    key: username
              - name: MYSQL_PASSWORD
                valueFrom:
                  secretKeyRef:
                    name: mysql-user-pass
                    key: password
              - name: MYSQL_DATABASE
                valueFrom:
                  configMapKeyRef:
                    name: bootcamp-configmap
                    key: database_name
            ports:
              - containerPort: 3306
            volumeMounts:
              - name: mysql-storage
                mountPath: /var/lib/mysql
          volumes:
            - name: mysql-storage
              emptyDir: {}
    ```
    
    ```bash
    kubectl apply -f deploy.yaml
    ```
    
* **Access the MYSQL pod**
    
    ```bash
    kubectl exec -it <pod-name> -- mysql -u root -p
    ```
    

### Conclusion

> In this article, we explored the essential concepts of Kubernetes ConfigMaps and Secrets, highlighting their importance in managing configuration data and sensitive information securely. ConfigMaps allow you to decouple configuration from application code, making updates seamless without redeploying the entire application. Secrets, on the other hand, provide a secure way to handle sensitive data like passwords and API keys, ensuring they are not exposed in your codebase.
> 
> We provided practical examples demonstrating how to use ConfigMaps and Secrets in various scenarios, such as passing database credentials to a MySQL Pod, managing environment-specific properties, and accessing configuration programmatically with Python. Additionally, we covered the different types of Secrets, including Opaque, TLS, and Docker Registry Secrets, and how to use them in Pods as environment variables or volume mounts.
> 
> By understanding and implementing these Kubernetes features, you can enhance the security and manageability of your applications, ensuring a more robust and flexible deployment process.
