#Deploy a cloud-native microservices application on IBM Cloud Private

### Tutorial

In this tutorial, you install and run a cloud-native microservices application on an IBM® Cloud Private platform on Kubernetes. The application implements a simple storefront that displays a catalog of computing devices. People can search for and buy products from the application's web interface. For a reference implementation diagram for the application, see [Microservices with Kubernetes](https://www.ibm.com/cloud/garage/architectures/microservices/1_0).

The application is composed of several microservices. With microservices, an application can be partitioned into smaller independent services that communicate with each other. This structure allows for the application to be developed, deployed, and managed by different teams. When you implement microservices and incorporate the [Circuit Breaker pattern](https://www.ibm.com/cloud/garage/content/manage/practice_circuit_breaker_pattern/), your application can remain partially operational even when one of the microservices is unavailable.

The application is called "BlueCompute" in the source code. You can see the code for the application on [GitHub](https://github.com/ibm-cloud-architecture/refarch-cloudnative-kubernetes). 

  * The user interface is provided by the web application which also acts as a BFF component, following the Backends for Frontends pattern. In this layer, front-end developers write back-end logic for the front end. The Web BFF is implemented by using the Node.js Express Framework. The microservices are packaged as Docker containers and are managed by the IBM Cloud Private Kubernetes cluster.

  * The BFF invokes a layer of reusable microservices that are written in Java™. The microservices run inside IBM Cloud Private using Docker and retrieve their data from databases. The catalog service retrieves items from a searchable JSON datasource by using ElasticSearch. The inventory service uses MySQL. ElasticSearch and MySQL databases are implemented as Docker containers.

In this tutorial, you will explore the following key capabilities: 

  * Access the IBM Cloud Private management console
  * Deploy a cloud-native microservices application to IBM Cloud Private
  * Validate the deployment

### Access the IBM Cloud Private Management Console

In this task, you connect to an IBM Cloud Private cluster and log in to the IBM Cloud Private management console. From the management console, you can manage the IBM Cloud Private cluster platform, workloads, security, and catalog.

  1. From a machine that is hosting your environment, open a web browser and go to one of the following URLs to access the IBM Cloud Private management console:

    * If you're using a Skytap-based demo, go to `https://10.0.0.1:8443`.
    * If you're using a Vagrant-based local IBM Cloud Private instance, go to `https://192.168.27.100:8443`.
  2. Log in by typing `admin` for the user name and password.

![image-20181016204939-1](https://live-ibm-dte.pantheonsite.io/sites/default/files/inline-images/image-20181016204939-1.png)

### Deploy the Application Workload to IBM Cloud Private

Application workloads can be deployed to run on an IBM Cloud Private cluster. The deployment of an application workload must be made available as a Helm package. Such packages must either be made available for deployment on a Helm repository, or loaded into an IBM Cloud Private internal Helm repository.

In this task, you use the IBM Cloud Private management console to deploy the Bluecompute Community Edition (bluecompute-ce) application workload that is hosted on a Helm repository.

  1. Go to the IBM Cloud Private management console that you opened in task 1.

  2. In the upper-left corner, click the menu and click **Manage > Helm Repositories**.

![image-20181024133111-1](https://live-ibm-dte.pantheonsite.io/sites/default/files/inline-images/image-20181024133111-1.png)

  3. Click **Add repository** to add the Helm repository that hosts the sample bluecompute-ce application workload.

a. For the name, type `ibmcase`.

b. For the URL, type ` http://gitlab.10.0.0.1.nip.io/skytap/refarch-cloudnative-kubernetes/raw/spring/docs/charts/bluecompute-ce`

**Note:** This URL might not be accessible from the browser.

c. Click **Add**.

![image](https://live-ibm-dte.pantheonsite.io/sites/default/files/inline-images/image.jpeg)

d. Click **Sync Repositories** and then click **OK **and wait for 3~4 minutes for the synchronization to complete.

  4. Click the menu and click **Catalog > Helm Charts**. The catalog might take a few seconds to open.

  5. Expand **Repositories** and select the **ibmcase** check box. The Catalog contents are filtered to show only the packages that are hosted on the ibmcase Helm repository.

![image](https://live-ibm-dte.pantheonsite.io/sites/default/files/inline-images/image_62.png)

  6. To deploy the application, find the chart **bluecompute-ce **and click on it. The package deployment page opens.

![image-20181024133332-4](https://live-ibm-dte.pantheonsite.io/sites/default/files/inline-images/image-20181024133332-4.png)

  7. By default, the latest version of the chart is selected. Click **Configure** to set any deployment values.

![image-20181024133352-5](https://live-ibm-dte.pantheonsite.io/sites/default/files/inline-images/image-20181024133352-5.png)

  8. Type `bluecompute-ce` for the release name and make sure the target namespace is set to **default**. Accept the default values for the other configuration settings. Click **Install**.

![image-20181024133403-6](https://live-ibm-dte.pantheonsite.io/sites/default/files/inline-images/image-20181024133403-6.png)

IBM Cloud Private starts processing the Helm charts that are in the package to deploy five microservices, three database backend services, and a web front-end application.

  9. After the deployment for all the resources is completed, an installation confirmation message is shown. Click **View Helm Release**.

![image-20181024133415-7](https://live-ibm-dte.pantheonsite.io/sites/default/files/inline-images/image-20181024133415-7.png)

  10. Wait a few minutes for the resources that are deployed to the underlying Kubernetes cluster to become operational and available.

a. Go to the release details page by clicking **bluecompute-ce** on the Helm releases page. Scroll down and check the AVAILABLE column in the DEPLOYMENT section to ensure that all resources have a nonzero value, which indicates that at least one instance of each resource is available.

b. Scroll further down the page and verify that the **customer-create-user** job has also completed successfully.

You might need to refresh the page a few times.

![image-20181024133433-8](https://live-ibm-dte.pantheonsite.io/sites/default/files/inline-images/image-20181024133433-8.png)

  11. After all the deployment resources are available, scroll up on the bluecompute-ce Helm releases page and click **Launch**.

![image](https://live-ibm-dte.pantheonsite.io/sites/default/files/inline-images/image_63.png)

Next, you verify that the deployed application is functional.

### Validate the Deployed Application

Now the application is installed in your IBM Cloud Private cluster, make sure that it's working as expected.

  1. Go to the browser tab that is displaying the sample application.

**Note:** If you see a `Your connection is not secure` message, click **Advanced** and then click **Add Exception** to add the security exception.

![image-20181024134342-1](https://live-ibm-dte.pantheonsite.io/sites/default/files/inline-images/image-20181024134342-1.png)

  2. Click **BROWSE ITEM CATALOG** to load the list of items.

![image-20181024134356-2](https://live-ibm-dte.pantheonsite.io/sites/default/files/inline-images/image-20181024134356-2.png)

  3. Click **Log in**. For the user name, type `user`, and for the password, type `passw0rd`. Click **Sign in**.

![image-20181024134406-3](https://live-ibm-dte.pantheonsite.io/sites/default/files/inline-images/image-20181024134406-3.png)

  4. Click one of the items to show the item details page.

  5. Select an item quantity and click **Buy**. A message is displayed to confirm that your order was placed successfully.

  6. Click **Profile** and view the order history. Review the date, item, and quantity of the order that you placed.

![image-20181024134417-4](https://live-ibm-dte.pantheonsite.io/sites/default/files/inline-images/image-20181024134417-4.png)

  7. Click **Logout**.

You validated that the application was deployed and is working.

### Summary

In this tutorial, you created, deployed, and validated a cloud-native application in a Kubernetes cluster. To continue learning about IBM Cloud Private and microservices, refer to these helpful resources: 

  * Watch the [Deploying Microservices in IBM Private Cloud](https://www.ibm.com/cloud/garage/videos/deploying-microservices-to-cloud) webinar.
  * For more information about private cloud, see the [private cloud architecture overview](https://www.ibm.com/cloud/garage/architectures/private-cloud). 
  * Want to try IBM Cloud Private locally on your computer? [Download IBM Cloud Private Community Edition](https://www.ibm.com/account/reg/us-en/signup?formid=urx-20295). It's free! 
  * Want guidance from IBM Cloud Garage experts? Check out the [IBM Cloud Private service offering](https://www.ibm.com/cloud/garage/services/private-cloud). 
