# Solution of exercise 4 #
This is the solution of the forth exercise for learning Kubernetes. 

After learning how to deploy the containers on Kubernetes, the purpose of this exercise is to add persistent storage and mount it in the container so that the MySQL database can be resilent to containers failures or restart. This exercise leverage the following three concepts about storage: 
- *StorageClass*: The StorageClass is a representation of a certain type of storage that exist in the specified cloud provider. It creates a level of abstraction from the cloud provider. For example a *StorageClass* named "slow" can refer to standard storage in West US with local redundancy in Azure. Any future reference to this will be made with the name "slow". When you create your storage class, you have to pass parameters that are specific to Azure storage. Detailed information about the latest developments made on StorageClass on Kubernetes is available on the following blog post: [http://blog.kubernetes.io/2017/03/dynamic-provisioning-and-storage-classes-kubernetes.html](http://blog.kubernetes.io/2017/03/dynamic-provisioning-and-storage-classes-kubernetes.html). A storageclass named "default" is created on your ACS Kubernetes cluster.
- *PersistentVolumeClaim*: Once the class has been created, it is possible to create a persistent volume claim (PVC). The PVC is a reservation of storage resources of the indicated class with certain type of access. In our case, we want to claim 10 Gigabytes of the cloud storage from the StorageClass called "slow" and only one node shall access this storage at one time. 
- *Volume*: The volume could be simplified to being a directory seen in the containers. The volumes, once mounted, point to the PVC that is explained above. the lifetime of a volume is tight to the lifetime of the POD in which it resides.


### Changes made from the solution of exercise 3 ###

In order to achieve the goal of the exercise we will do the following steps: 
1. Define the StorageClass 
2. Define the PersistentVolumeClaim
3. Modify our Deployment to use volumes in the persistent volume claim
4. Mount the volume in the container

Let's look at each step in details.
1. Define the StorageClass

At the begining of your definition file add the following code: 

```yaml
    kind: StorageClass
    apiVersion: storage.k8s.io/v1beta1
    metadata:
    name: slow
    provisioner: kubernetes.io/azure-disk
    parameters:
    skuName: Standard_LRS
    location: westus
```

You are defining a storage class named **slow** that will leverage Azure standard storage with local replication based in the **westus** region.
You can be specific and add the name of the azure storage account that you want to associate with this StorageClass. In that case, simply add the following parameter ```storageAccount: azure_storage_account_name``` after the location.

2. Define a PersistentVolumeClaim by adding the following code after the StorageClass definition.

```yaml
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: mysql-pv
  annotations:
    volume.beta.kubernetes.io/storage-class: slow
  labels:
    app: wordpress
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
```

The documentation indicates to use the property **StorageClass** however at the time of the writting of this exercise, Azure requires the use of **annotations**. This code will create a vhd file of the specified size in the Azure storage account that you have specified in the definition of the storageClass or will take the one that matches the best the location and skuName combination.
Additional details available here: [https://kubernetes.io/docs/concepts/storage/persistent-volumes/#azure-disk](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#azure-disk)

3. Add a volume in the definition of your deployment that will leverage the persisten volume claim that we have created before.
At the end of your deployment add the following code:

```yaml
    volumes: 
    - name: mysqlvolume
    persistentVolumeClaim:
        claimName: mysql-pv
```

Note that the ```claimName``` is the name of the persistent volume claim that we have defined before.

4. Finally mount this volume into the container and have MySQL use it to store the database and its logs. You will have to know where MySQL is storing its data in the container that you are using. For this, refer to the documentation located here [https://hub.docker.com/_/mysql/] (https://hub.docker.com/_/mysql/) and go to the section named **Where to Store Data**.
MySQL data will be located in ```/var/lib/mysql```, you need to indicate to the container to mount the volume that we have defined in step 3 under this location. 

In the description of the container, add the following code that will mount the volume named ```mysqlvolume``` under ```/var/lib/mysql```:

```yaml
    volumeMounts:
    - mountPath: /var/lib/mysql
        name: mysqlvolume
```

5. Deploy the MySQL application using the same command than you have been using previously. 


## Commonly seen issues ## 
The following are commonly seen issues:

1. The provisioning of the persistent storage may take some time and create confusion when you are looking at the deployment to be processed. Kubernetes returns the command prompt quickly but there are tasks running in the background, ensure that you are giving enough time for the process to complete.
2. If you look at the logs of the container, you may see some warning being reported, some of them are due to the time it takes to provision the persistent storage. Typical warnings may look like : 

```
FailedScheduling        [SchedulerPredicates failed due to PersistentVolumeClaim is not bound: "mysql-pv", which is unexpected., SchedulerPredicates failed due to PersistentVolumeClaim is not bound: "mysql-pv", which is unexpected., SchedulerPredicates failed due to PersistentVolumeClaim is not bound: "mysql-pv", which is unexpected.]
Scheduled               Successfully assigned mysql-1506261288-873wz to k8s-agent-164a2ee3-1
FailedMount             Unable to mount volumes for pod "mysql-1506261288-873wz_default(97565799-35e8-11e7-b472-000d3a35b396)": timeout expired waiting for volumes to attach/mount for pod "default"/"mysql-1506261288-873wz". list of unattached/unmounted volumes=[mysqlvolume]
FailedSync              Error syncing pod, skipping: timeout expired waiting for volumes to attach/mount for pod "default"/"mysql-1506261288-873wz". list of unattached/unmounted volumes=[mysqlvolume]
```


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

There is a some litterature availabe about storage in Kubernetes:
- Volumes: [https://kubernetes.io/docs/concepts/storage/volumes/](https://kubernetes.io/docs/concepts/storage/volumes/)
- In depth update on StorageClasses and PersistentVolumeClaims [http://blog.kubernetes.io/2017/03/dynamic-provisioning-and-storage-classes-kubernetes.html](http://blog.kubernetes.io/2017/03/dynamic-provisioning-and-storage-classes-kubernetes.html)
- All about Persistent Volumes: [https://kubernetes.io/docs/concepts/storage/persistent-volumes/](https://kubernetes.io/docs/concepts/storage/persistent-volumes/)
- Documentation about the MySQL container: [https://hub.docker.com/_/mysql/](https://hub.docker.com/_/mysql/). If you were to use a different application, you would need to know about where the application stores its data.
