# Soltion of exercise 1 #
This is the solution of the first exercise for learning Kubernetes. 


## Creating the template for the MySQL pod

First we start with a **Pod**. In this case, the pod is a wrapper around the MySQL container. You can read more about [pods on the official Kubernetes documentation](https://kubernetes.io/docs/concepts/workloads/pods/pod-overview/). In this exercise we are pulling the official image of MySQL from DockerHub but it could be downloaded from a private Azure Container registry.

```yaml
kind: Pod
apiVersion: v1
metadata:
  name: mysql
  labels:
    tier: mysql
    app: wordpress
spec:
  containers:
  - name: mysql
    image: mysql:8.0.1
    ports:
      - containerPort: 3306         
        protocol: TCP
        name: mysql
```

Next, we want our server to have a specific database name ("sample"), username ('mysql") and password ("mysql).  
To do this we can specify enviroment variables in the template:

```yaml
env: 
- name: "MYSQL_ROOT_PASSWORD"
    value: "password"
- name: "MYSQL_USER"
    value: "mysql"
- name: "MYSQL_PASSWORD"
    value: "mysql"
- name: "MYSQL_DATABASE"
    value: "sample"
```

Putting it all together, our pod definition now looks like this:

```yaml
kind: Pod
apiVersion: v1
metadata:
  name: mysql
  labels:
    tier: mysql
    app: wordpress
spec:
  containers:
  - name: mysql
    image: mysql:8.0.1
    ports:
      - containerPort: 3306         
        protocol: TCP
        name: mysql
    env: 
    - name: "MYSQL_ROOT_PASSWORD"
      value: "password"
    - name: "MYSQL_USER"
      value: "mysql"
    - name: "MYSQL_PASSWORD"
      value: "mysql"
    - name: "MYSQL_DATABASE"
      value: "sample"
```

## Exposing MySQL workbench through a kubernetes service

We now want to expose our pod through a service.   
A service is an object that defines a logical set of Pods and the policy by which to access them. The set of Pods targeted by a Service is usually specified using a label selector. The official documentation has more about [services on Kubernetes](https://kubernetes.io/docs/concepts/services-networking/service/).

The expected template is pretty straightforward:

```yaml
kind: Service
apiVersion: v1
metadata:
  name: mysql
  labels:
    app: wordpress
spec:
  ports:
  - port: 3306
    targetPort: 3306
  selector:
    app: wordpress
    tier: mysql
```

Finally, we can put the pod and the service template in the same YAML file, as it is done in [./mysql-pod-novolume-nosecret.yaml](mysql-pod-novolume-nosecret.yaml)


## Deploying

You can deploy the solution to your cluster by running the following command

```kubectl create -f mysql-pod-novolume-nosecret.yaml```

Verify that your MySQL instance is running by running

```kubectl get pods```

## Commonly seen mistakes
The following are commonly seen mistake:

1. YAML is very sensitive to indentation, do not use "tab" in your files and use "space" instead.
2. "error: error converting YAML to JSON: yaml: line 10: did not find expected key"
    You have an identation error line 10 in your yaml file. Review carefully this line, check alignment and indentation.

    **Note**: The line indicated in the error message is the line of the description of the object, which may not be the line number in the file. 


## Enable the connection with MySQL workbench
With the configuration of your application as indicated in the associated solution, the MySQL instance is only accessible from within the cluster. to be able to connect to it, you will need to change the type of your service as follows.

1. Type 
```kubectl edit svc/mysql ```

2. Change the following line:

    ```type: ClusterIP``` 

    to
    
    ```type: LoadBalancer```

3. Run 

```kubectl get svc/mysql```

The **EXTERNAL-IP** is the public IP that you can use to connect to your server with MySQL Workbench. 
It might take a few minutes in a `<pending>` state before the IP address is successfully allocated.

4. Optional, if you want to clean your Kubernetes clusters, you can run the following command

```kubectl delete pods,services -l app=wordpress```


## Learn more 

You can learn more about kubernetes services at the following location [https://kubernetes.io/docs/concepts/services-networking/service/](https://kubernetes.io/docs/concepts/services-networking/service/)

