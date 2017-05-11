# Exercise 2 - Adding secrets for MySQL in POD #

This exercice is about using a password for the MySQL database that is not exposed in the yaml file describing the application. You will modify the file that you have created during the first exercise and change how the password is passed to the MySQL container:
- You have two options to achieve this lab. Select the one that you prefer, in the solution we will provide both.
- The password shall not appear in the file describing the application.

**Note**: You are not expected to use persistent volumes.

At the end of this exercise, launch MySQL workbench and connect to your deployed instance to verify that you can connect with your secrets. Keep in mind that you will have to expose your mysql server to a public endpoint.

<u>Resources</u>: 
* Review how to create your own secrets in Kubernetes: [https://kubernetes.io/docs/concepts/configuration/secret/#creating-your-own-secrets](https://kubernetes.io/docs/concepts/configuration/secret/#creating-your-own-secrets)
* Understand the options available to create secrets in Kubernetes:
[https://kubernetes.io/docs/concepts/configuration/secret/#creating-a-secret-manually](https://kubernetes.io/docs/concepts/configuration/secret/#creating-a-secret-manually)

<u>**Assessment**</u>: Connect to MySQL with myworkbench and verify that the connection works with the password stored in the file.

<u>**Estimated time**</u>: 30 minutes 

<u>**Solution**</u>: If you need help, you can find the solution of this exercise here: [solutions/lab2](../solutions/lab2/README.md)