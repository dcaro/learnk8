# Soltion of exercise 2 #
This is the solution of the second exercise for learning Kubernetes. 

The purpose of this exercise is to use Kubernetes secrets to store the password of the MySQL database:
- *Secrets*: The Kubernetes secrets is an object that you can use to store secrets of a small size like username and passwords. You can read more about [how to manage kubernetes secrets](https://kubernetes.io/docs/concepts/configuration/secret/).
- *secretKeyRef*: This indicates to Kubernetes that the value that we are looking for is stored in a secret in the cluster with the name and key indicated below. 

The secret can be created in Kubernetes with several methods. We are describing two here, you may select a different one of your choice.

### Option A - Create a secret with the command line ###
1. Create a file named ```password.txt``` into which you will store the MySQL password in clear.
2. Run the following command to create a Kubernetes secret based on the information stored in the file

```kubectl create secret generic mysql-pass --from-file=password=./password.txt```

This creates a secret called ```mysql-pass```and the password will be stored in a key named ```password```.

### Option B - Create a secret using a yaml file ###
1. Get the base64 encoding of the value of the password

```echo -n "the_password_to_code_in_base64" | base64```

2. Copy the base64 encoded password value in the ```secrets.yaml``` file as in the secrets.yaml provided as an example and run the following command: 

```kubectl create -f secrets.yaml```

### Verify the creation of the secret ### 

3. Verify that your secret has been created correctly: 

```kubectl describe secrets/mysql-pass```

The ```Data``` field should show onl one entry called ```password``` corresponding to the command executed on step 2.


4. Add a reference to the secret that you have just published by changing how the ```MYSQL_PASSORD``` is defined: 

```yaml
- name: "MYSQL_PASSWORD"
    valueFrom:
    secretKeyRef:
        name: mysql-pass
        key: password
```

This makes the pod look for the value of the password in the Kubernetes cluster secrets. It allows separation of responsabilities and higher confidentiality of those secrests.

5. Deploy the MySQL application using the same command than you have been using in exercise 1. 


## Commonly seen mistakes ## 

This will be extended as student report issues with this lab. At this time, no common issues have been identified.

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

You can learn more about kubernetes secrets at the following location [https://kubernetes.io/docs/concepts/configuration/secret/](https://kubernetes.io/docs/concepts/configuration/secret/)

