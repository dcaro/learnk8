# Exercise 4 - Add persistent storage to store the MySQL database #

The challenge with containers is that the files that are writte within the container are ephemeral and will be lost when the container restarts. In this exercise you will use persistent volumes to add non ephemeral storage in Azure to store the MySQL database.

At the end of this exercise, launch MySQL workbench and connect to your deployed instance to verify that you can connect with your secrets. Keep in mind that you will have to expose your mysql server to a public endpoint.

**Notes**: You are expected to use a deployment and an Azure-disk storage.


<u>Resources</u>: 
* Get the necessary familiarity with Kubernetes volumes  [https://kubernetes.io/docs/concepts/storage/volumes/#background](https://kubernetes.io/docs/concepts/storage/volumes/#background) and how the volumes are mounted in a container running in Kubernetes [https://kubernetes.io/docs/concepts/storage/volumes/#example-pod](https://kubernetes.io/docs/concepts/storage/volumes/#example-pod)
* Familiarize yourself with the concept of Persistent Volumes [https://kubernetes.io/docs/concepts/storage/persistent-volumes/#introduction](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#introduction)
* Understand how to use Persistent Volume Claim with Azure [https://kubernetes.io/docs/concepts/storage/persistent-volumes/#persistentvolumeclaims](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#persistentvolumeclaims)
**Note:** At the time this exercise is being written, **storageClassName** is not supported on Azure and you will have to use **annotations**


<u>Assessment</u>: Connect to MySQL with myworkbench and verify that the connection works with the password stored in the file.

<u>Estimated time</u>: 45 minutes 

<u>Solution</u>: If you need help, you can find the solution of this exercise here: []()