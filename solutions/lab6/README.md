# Solution of exercise 6 #
This is the solution of the sixth exercise for learning Kubernetes. 

In this exercise we are moving from having a signle yaml definition to creating a helm chart. You will have to follow those steps to create the Wordpress chart:

- *Install helm*: You will install helm by following the instructions located here [https://github.com/kubernetes/helm#install](https://github.com/kubernetes/helm#install). Verify that you have the proper version of helm with the following command:

```helm version ```

The output should look like the following: 

```
Client: &version.Version{SemVer:"v2.4.1", GitCommit:"46d9ea82e2c925186e1fc620a8320ce1314cbb02", GitTreeState:"clean"}
Server: &version.Version{SemVer:"v2.4.1", GitCommit:"46d9ea82e2c925186e1fc620a8320ce1314cbb02", GitTreeState:"clean"}
```

- *Create a blank chart*: Tun the following command to create a directory with the files initialized with a basic configuration: 

```helm create mychart``` 

This creates a directory that looks like this: 

```
.
├── charts
├── Chart.yaml
├── templates
│   ├── deployment.yaml
│   ├── _helpers.tpl
│   ├── ingress.yaml
│   ├── NOTES.txt
│   └── service.yaml
└── values.yaml
```

- *Edit the description of your Chart*: Modify the content of the ```Chart.yaml``` file to insert your information. For example this nay look like the following: 

```
apiVersion: v1
description: A WordPress chart for Kubernetes
name: wordpress
version: 0.1.0
maintainers:
  - name: Damien Caro
    email: dcaro@microsoft.com
```

- *Chart definition*: Create in the ```templates``` directory the files that are needed to define completely the wordpress application. You can use one file per application per type of object that you want to deploy.
Based on what has been deployed in the previous exercise, create and delete files to obtain the following structure :

```
├── charts
├── Chart.yaml
├── templates
│   ├── _helpers.tpl
│   ├── mysql-deployment.yaml
│   ├── mysql-pvc.yaml
│   ├── mysql-service.yaml
│   ├── NOTES.txt
│   ├── storage.yaml
│   ├── wp-deployment.yaml
│   ├── wp-pvc.yaml
│   └── wp-service.yaml
└── values.yaml
```

Now, edit each files individually to create your Chart.

- *storage.yaml*: Copy and paste in this file the content from ```StorageClass``` section from the ```mysql-deployment-volume.yaml```. 
Replace the values for **name** and **location** with a variable that will point to the value in the ```values.yaml``` file as follows: 


kind: StorageClass
apiVersion: storage.k8s.io/v1beta1
metadata:
  name: {{.Values.storage.classname }}
provisioner: kubernetes.io/azure-disk
parameters:
  skuName: Standard_LRS
  location: {{ .Values.location }}




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
