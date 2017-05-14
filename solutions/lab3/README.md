# Solution of exercise 3 #
This is the solution of the third exercise for learning Kubernetes. 

The purpose of this exercise is to switch from raw pods to *deployment* to deploy the MySQL container. This exercise introduces the following concepts:
- *Deployment*: The deployment declaration is very similar to the declaration of the pod itself but bings more flexibility in operations like updating the containers or managing its replicas. You can use the deployment to create new resources or update existing ones. 
- *Deployment controller*: The deployment controller will ensure that the state described is the actual state of the resources. For example, if your deployment is unstable, the deployment controller will rollback to an earlier Deployment revision.


### Changes made from the solution of exercise 2 ###
1. First, update the definition of the application by adjusting the ```kind``` and ```apiVersion``` as follows :

```yaml
kind: Deployment
apiVersion: extensions/v1beta1
```

2. Note that in the ```spec:``` section, the deployment is using a ```template``` which acts like a wrapper around the definition of the container that we are using in this deployment.

```yaml
spec:
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: wordpress
        tier: mysql
    spec:
      containers:
      - name: mysql
        image: mysql:8.0.1
```

The strategy indicates how the Pods that are already deployed with will be replaced with the new ones. 
We are using the ```Recreate``` strategy that kills the existing pods before deploying the new ones.


3. Deploy the MySQL application using the same command than you have been using previously. 


## Commonly seen mistakes ## 
The following are commonly seen mistake:
1. Ensure that  ```metadata``` and ```spec``` in the template have the same indent.


## Enable the connection with MySQL workbench ##
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


## Learn more ##

* You can learn more about kubernetes deployments at the following location [https://kubernetes.io/docs/concepts/workloads/controllers/deployment/](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)
* Take a closer look at the different strategies that are available for the deployments.

