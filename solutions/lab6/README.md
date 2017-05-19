# Solution of exercise 6 #
This is the solution of the sixth exercise for learning Kubernetes. 

In this exercise we are moving from having a signle yaml definition to creating a helm chart. You will have to follow those steps to create the Wordpress chart:

1. *Install helm*: You will install helm by following the instructions located here [https://github.com/kubernetes/helm#install](https://github.com/kubernetes/helm#install). Verify that you have the proper version of helm with the following command:

```helm version ```

The output should look like the following: 

```
Client: &version.Version{SemVer:"v2.4.1", GitCommit:"46d9ea82e2c925186e1fc620a8320ce1314cbb02", GitTreeState:"clean"}
Server: &version.Version{SemVer:"v2.4.1", GitCommit:"46d9ea82e2c925186e1fc620a8320ce1314cbb02", GitTreeState:"clean"}
```

2. *Create a blank chart*: Tun the following command to create a directory with the files initialized with a basic configuration: 

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

3. *Edit the description of your Chart*: Modify the content of the ```Chart.yaml``` file to insert your information. For example this nay look like the following: 

```
apiVersion: v1
description: A WordPress chart for Kubernetes
name: wordpress
version: 0.1.0
maintainers:
  - name: Damien Caro
    email: dcaro@microsoft.com
```

4. *Chart definition*: Create in the ```templates``` directory the files that are needed to define completely the wordpress application. You can use one file per application per type of object that you want to deploy.
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

5. *storage.yaml*: Copy and paste in this file the content from ```StorageClass``` section from the ```mysql-deployment-volume.yaml```. 
Replace the values for **name** and **location** with a variable that will point to the value in the ```values.yaml``` file as follows: 

```yaml
kind: StorageClass
apiVersion: storage.k8s.io/v1beta1
metadata:
  name: {{.Values.storage.classname }}
provisioner: kubernetes.io/azure-disk
parameters:
  skuName: Standard_LRS
  location: {{ .Values.location }}
```

6. *mysql-pvc.yaml*: Copy and paste in this file the content from ```PersistentVolumeClaim``` section from the ```mysql-deployment-volume.yaml``` definition file. 
Replace the values for **name**, **volume.beta.kubernetes.io/storage-class** and **storage** with variables that will point to the values that are in the ```values.yaml```file. The result will be as follows: 

```yaml
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: {{ .Values.mysql.persistentvolumeclaim.name }}
  annotations:
    volume.beta.kubernetes.io/storage-class: {{ .Values.storage.classname }}
  labels:
    app: wordpress
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: {{.Values.mysql.persistentvolumeclaim.size }}
```

7. Repeat the same process for each of the files that have been identified above in the chart definition. You can look at each files individually here [solutions/lab6/wordpress](./wordpress/)


8. Deploy WordPress by using helm

```helm install wordpress```

**Note:** you need to be in the directory that has your ```mychart``` directory. 

The output should look like the following: 

```
NAME:   toned-rattlesnake
LAST DEPLOYED: Fri May 19 12:08:24 2017
NAMESPACE: ascdev
STATUS: DEPLOYED

RESOURCES:
==> v1beta1/Deployment
NAME                         DESIRED  CURRENT  UP-TO-DATE  AVAILABLE  AGE
wpchart                      1        1        1           0          0s
toned-rattlesnake-wordpress  1        1        1           0          0s

==> v1beta1/StorageClass
NAME        TYPE
chartclass  kubernetes.io/azure-disk

==> v1/PersistentVolumeClaim
NAME       STATUS   VOLUME      CAPACITY  ACCESSMODES  STORAGECLASS  AGE
mysql-pvc  Pending  chartclass  0s
wp-pvc     Pending  chartclass  0s

==> v1/Service
NAME        CLUSTER-IP    EXTERNAL-IP  PORT(S)         AGE
wordpress   10.0.236.38   <pending>    80:31430/TCP    0s
mysqlchart  10.0.141.135  <pending>    3306:32289/TCP  0s
```

You can run ```helm status <name_of_your_deployment>``` to get the public IPs that have been assigned to your service and the status of the persistent volume claims. Keep in mind that you will not be able to access your wordpress deployment as long as the persistent volume claims remains in **Pending** state.
Once the IP and the PVC are cleared, you should be able to access your wordpress deployment. 

## Commonly seen issues 
The following are commonly seen issues:

1. The deployment will take some time (including the allocation of the public IP for the loadbalancer) and the connection to the database can timeout. If you recieve a timeout when you navigate to the publicIP of the LoadBalancer, try again after some time. 
Verify that your pods are running correclty and do not report any errors/
2. If Kubernetes cannot find a storage account that matches the properties of the persistent volume claim, the pvc will remain in pending state. Use the ```kubectl describe``` command to get the logs from the persistent volume claim and understand what is happening on the storage area.


## Going further
Congratulations, you have reached the end of this series of exercises to learn Kubernetes and Helm ! 

You can explore the following topics:
- *helm chart*: [https://github.com/kubernetes/charts](https://github.com/kubernetes/charts)
- *Contributing to a chart*: [https://github.com/kubernetes/charts#contributing-a-chart](https://github.com/kubernetes/charts#contributing-a-chart)
