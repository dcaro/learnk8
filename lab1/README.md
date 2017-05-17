# Exercise 1 - Running MySQL in a POD #

To complete this exercise you have to deploy MySQL on Kubernetes, using a template file, with the following characteristics: 
- Deploy a MySQL instance on your Kubernetes cluster in a single pod.
- The MySQL server shall have a database named "sample".
- Your instance shall have a user names *mysql* with *mysql* as the password.
- You shall be able to access your instance from outside of the cluster with MySQL workbench.

**Note**: You are not expected to use persistent volumes nor managing the secrets.

At the end of this exercise, launch [MySQL workbench](https://dev.mysql.com/downloads/workbench/) and connect to your deployed instance to verify that you can connect with the user name and password that you have created. Keep in mind that you will have to expose your mysql server to a public endpoint.


<u>Resources</u>: 

* Create your Kubernetes Cluster by following the steps described here:  [https://docs.microsoft.com/en-us/azure/container-service/container-service-kubernetes-walkthrough](https://docs.microsoft.com/en-us/azure/container-service/container-service-kubernetes-walkthrough)
* Review the pod definition, db-pod.yml file in the HOL [https://github.com/Azure/acs-demos/tree/master/training/kubernetes#db-podyml](https://github.com/Azure/acs-demos/tree/master/training/kubernetes#db-podyml)
* Understand the environment variables required by the MySQL container by reading the container documentation [https://hub.docker.com/_/mysql/](https://hub.docker.com/_/mysql/)
* Review the definition of a service:[https://github.com/kubernetes/kubernetes/tree/master/examples/guestbook#define-a-service](https://github.com/kubernetes/kubernetes/tree/master/examples/guestbook#define-a-service)

<u>**Assessment**</u>: Connect to MySQL with myworkbench and verify that the connection works

<u>**Estimated time**</u>: 45 minutes 

<u>**Solution**</u>: If you need help, you can find the solution of this exercise here: [solutions/lab1](../solutions/lab1/README.md)
