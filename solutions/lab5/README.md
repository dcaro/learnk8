# Solution of exercise 5 #
This is the solution of the fifth exercise for learning Kubernetes. 

In this exercise we are leveraging the official WordPress container image for our deployment and we will connect it to the MySQL instance that we have deployed in the previous exercises. You will need to apply the previous learnings and have some understanding of WordPress and Apache.

- *WordPress*: We are using the offical container from Docker Hub [https://hub.docker.com/_/wordpress/](https://hub.docker.com/_/wordpress/), more specifically the ```4.6.1-apache```. The container needs several environment variables to run: 
    - **WORDPRESS_DB_HOST**: This indicates what is the name or the IP of the container that runs the mysql database. In our case we will use the name of the Service that has been defined in the previous exercise. 
    ```yaml
      env: 
        - name: "WORDPRESS_DB_HOST"
      value: mysql
    ```
    - **WORDPRESS_DB_PASSWORD**: This environment variable indicate the password to use to access the MySQL database that we have indicated previousy. We will leverage the kubernetes secrets that we have created in exercise 2. It also gives the insurance that the password is located in only one location.
    ```yaml
        - name: "WORDPRESS_DB_PASSWORD"
          valueFrom:
            secretKeyRef:
              name: mysql-pass
              key: password
    ```
- *Apache*: In the container that we have chosen we run Apache. The default location for the document root in Apache is "/var/www/html". In order to preserve the content of the website in the case of a failure, we want to use a volume on a persistent storage for those data. We will then use the VolumeMounts:
```yaml
    volumeMounts:
    - mountPath: /var/www/html
        name: wordpressvolume
```

Let's get things together for this exercise: 
1. You need to have a Service defined in your yaml. Make sure that you are using the loadbalancer type to allow the external communication to wordpress on Apache.

```yaml
kind: Service
apiVersion: v1
metadata:
  name: wordpress
  labels:
    app: wordpress
spec:
  ports:
    - port: 80
  selector:
    app: wordpress
    tier: frontend
  type: LoadBalancer
```

2. Create a persistent volume claim to store the apache documents on a reliable location

```yaml
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: wordpress-pv
  annotations:
    volume.beta.kubernetes.io/storage-class: slow
  labels:
    app: wordpress
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
```

3. Add a deployment for wordpress that uses a volume on the PersistentVolumeClaim that is defined above and point at the mysql instance that has been deployed previously:

```yaml
kind: Deployment
apiVersion: extensions/v1beta1
metadata:
  name: wordpress
  labels:
    app: wordpress
spec:
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: wordpress
        tier: frontend
    spec:
      containers:
      - name: wordpress
        image: wordpress:4.6.1-apache
        ports:
          - containerPort: 80
            name: wordpress
        volumeMounts:
        - mountPath: /var/www/html
          name: wordpressvolume
        env: 
        - name: "WORDPRESS_DB_HOST"
          value: mysql
        - name: "WORDPRESS_DB_PASSWORD"
          valueFrom:
            secretKeyRef:
              name: mysql-pass
              key: password
      volumes: 
      - name: wordpressvolume
        persistentVolumeClaim:
          claimName: wordpress-pv
```

5. Deploy WordPress by using

```kubectl create -f wordpress-delpoyment.yaml```. 


## Commonly seen issues 
The following are commonly seen issues:

1. The deployment will take some time (including the allocation of the public IP for the loadbalancer) and the connection to the database can timeout. If you recieve a timeout when you navigate to the publicIP of the LoadBalancer, try again after some time. 
Verify that your pods are running correclty and do not report any errors/
2. If your pods are in an unstable state, do not hesitate to delete them, the deployment controller will redeploy one container based on the template that has been created. 


## Enable the connection to WordPress
1. Verify that the pods are running.

```yaml
kubectl get pods
NAME                         READY     STATUS    RESTARTS   AGE
mysql-1519762796-78mgw       1/1       Running   0          24m
wordpress-2799185977-kcj6n   1/1       Running   0          24m
```

2. Verify that the WordPress service has been allocated a public IP.

```yaml
kubectl get svc
NAME         CLUSTER-IP     EXTERNAL-IP      PORT(S)        AGE
kubernetes   10.0.0.1       <none>           443/TCP        21h
mysql        10.0.174.195   <none>           3306/TCP       7m
wordpress    10.0.95.171    XX.XX.XX.XX   80:31104/TCP   6m

```

3. Access the IP that has been allocated to the wordpress service with a browser and verify that you can access WordPress

## Going further
Congratulations, you have reached the end of this series of exercises to learn Kubernetes ! 
You can explore the following topics:
- *NameSpace*: [https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/)
- *DaemonSets*: [https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/)
