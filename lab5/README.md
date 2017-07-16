# Exercise 5 - Adding wordpress to the deployment #

In this exercise you are asked to deploy Wordpress on your Kubernetes cluster. This deployment of Wordpress shall use the MySQL deployment that you have done in the previous exercises and the website shall be resilient to container outages should they happen.

At the end of this exercise, you must be able to connect to your wordpress website and create your blog. If you delete or restart the wordpress container the website shoud come back up as it was before the operation.

**Note**: The official WordPress container image does not have a mysql instance, you must use the mysql instance that you have deployed previously.

<u>Resources</u>: 
* Get familiar with the official WordPress container image [https://hub.docker.com/_/wordpress/](https://hub.docker.com/_/wordpress/)
* Understand how the selector works with the service [https://kubernetes.io/docs/concepts/services-networking/connect-applications-service/#creating-a-service](https://kubernetes.io/docs/concepts/services-networking/connect-applications-service/#creating-a-service)
* Apache file system [https://httpd.apache.org/docs/trunk/getting-started.html](https://httpd.apache.org/docs/trunk/getting-started.html)


<u>**Assessment**</u>: Connect to your WordPress deployment and verify that you can create a website.

<u>**Estimated time**</u>: 30 minutes 

<u>**Solution**</u>: If you need help, you can find the solution of this exercise here: [solutions/lab5](../solutions/lab5/README.md)
