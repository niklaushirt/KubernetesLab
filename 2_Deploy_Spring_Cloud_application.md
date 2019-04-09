# Deploy a Spring Cloud application on IBM Cloud Private
## Learn how to deploy a Spring Cloud Netflix environment and an opinionated Spring Cloud application on IBM Cloud Private.

IBM Cloud™ Private is an application platform where you can develop and manage on-premises, containerized applications. It's an integrated environment for managing containers that includes the container orchestrator Kubernetes, a private image repository, a management console, and monitoring frameworks. From the IBM Cloud Private graphical user interface, you can deploy, manage, monitor, and scale your applications. 

You can deploy application workloads that are based on Spring Cloud so that they run on IBM Cloud Private as Kubernetes resources. Because Spring Cloud applications use annotations that follow the declarative approach, the underlying Spring Cloud Netflix projects that offer those features must be available on IBM Cloud Private. 

In this tutorial, you deploy a Spring Cloud Netflix environment and an opinionated Spring Cloud application on IBM Cloud Private.

### Tutorial

As part of the Spring Cloud environment, you deploy these items on IBM Cloud Private: 

  * Config Server (centralized external configuration)

  * Eureka (service discovery)

  * Zuul (intelligent routing)

  * Hystrix (circuit breaker and dashboard)

  * Turbine (metrics stream aggregator)

  * Zipkin (distributed tracing) projects

You also deploy a Spring Boot application as-is, without changing the auto-configuration and binding annotations that are in the application.

In this tutorial, you complete these tasks:

  * Access the IBM Cloud Private Dashboard and set up kubectl

  * Deploy RabbitMQ

  * Add the ibmcase-spring package to the IBM Cloud Private catalog

  * Deploy the Spring Cloud package

  * Access the Eureka Dashboard

  * Deploy a sample Spring workload

[

### Prerequisites

](https://www.ibm.com/cloud/garage/tutorials/cloud-private-spring-cloud?task=0/)

  1. A conceptual understanding of how [Kubernetes](https://kubernetes.io/docs/concepts) works

  2. A high-level understanding of [Helm and Kubernetes package management](https://docs.helm.sh/architecture)

  3. A basic understanding of [IBM Cloud Private cluster architecture](https://www.ibm.com/support/knowledgecenter/SSBS6K_2.1.0/getting_started/architecture.html)

  4. Access to an operational IBM Cloud Private cluster:

    * To deploy IBM Cloud Private using Vagrant, see the instructions at [https://github.com/IBM/deploy-ibm-cloud-private/blob/master/docs/deploy-vagrant.md](https://github.com/IBM/deploy-ibm-cloud-private/blob/master/docs/deploy-vagrant.md).

    * To access a free trial of IBM Cloud Private, follow these steps:

a. Go to the [Blue Demo page](http://ibm.biz/try-icp) for IBM Cloud Private.

b. Log in with your IBMid.

c. Click **Reserve a Demo**. 

d. Enter the required information. After you submit your request, your environment is provisioned on IBM Cloud Private.

e. After a few minutes, check your email inbox for an email that contains instructions about the environment that you reserved.

f. When you log in to your environment for the first time, your VM is suspended. To start it, click the play icon in the upper-right corner. To access your environment, click **Boot/Master Node**. 

![Boot/Master Node](https://www.ibm.com/cloud/garage/images/tutorials/cloud-private-spring-cloud/spring-cloud1.png)

**Note:** After 2 hours of inactivity, the environment is suspended. If you want to resume your work, click the play icon in the upper-right corner. The restart process takes 3 - 5 minutes. When all the VMs are restarted, you can complete the tutorials by clicking the compute image on the Desktop image.

  5. [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/) installed on your machine

**Important:** The steps in this tutorial are specific to an IBM Cloud Private cluster that is hosted on Skytap. If you aren't using an IBM Cloud Private cluster on Skytap, be sure to use the URLs for your cluster instead of the Skytap URLs as you complete the tutorial.

Learn more about IBM Cloud Private

* * *

![](https://www.ibm.com/cloud/garage/images/tutorials/icons/privateCloud.svg)

[ Bring the speed of public and the control of private to your enterprise →  ](https://www.ibm.com/cloud/private)

[

### Task 1: Access the IBM Cloud Private Dashboard and set up kubectl

](https://www.ibm.com/cloud/garage/tutorials/cloud-private-spring-cloud?task=1/)

In this task, you connect to an IBM Cloud Private cluster and set up kubectl, the Kubernetes command-line client, to use the cluster.

  1. From the machine that hosts your environment, open a web browser. To access the IBM Cloud Private management console, go to one of these URLs:

    * If you're using a Skytap-based demo, go to `https://10.0.0.1:8443`.

    * If you're using a Vagrant-based local IBM Cloud Private instance, go to `https://192.168.27.100:8443`.

  2. Log in to the IBM Cloud Private Dashboard by typing `admin` for both the user name and password.

  3. In the upper-right corner of the Dashboard, click the **admin** user name link, and then click **Configure Client**. The kubectl configuration commands are shown. Copy the commands to the clipboard. Press the Esc key to close the window.

![kubectl commands](https://www.ibm.com/cloud/garage/images/tutorials/cloud-private-spring-cloud/spring-cloud3.png)

  4. Open a command-line window. Paste the commands and press Enter.

  5. Confirm that kubectl is correctly configured by entering this command:

`kubectl config current-context`

  6. Confirm that the output of the command is as follows:
    
    mycluster.context

kubectl is now set up on the Desktop node to manage IBM Cloud Private. Leave the IBM Cloud Private Dashboard open.

**Important**: In this tutorial, you use this command-line session to run all kubectl commands. Before you use this command-line session, confirm that kubectl is still configured to manage the cluster by entering the `kubectl config current-context` command. If the session expired, repeat steps 3 - 6 to configure kubectl.

[

### Task 2: Deploy RabbitMQ

](https://www.ibm.com/cloud/garage/tutorials/cloud-private-spring-cloud?task=2/)

In this tutorial, you use RabbitMQ as a message broker to publish configuration changes from the Spring Config server to all the other components and Spring workloads. IBM Cloud Private contains a RabbitMQ Helm package.

Deploy the RabbitMQ service by following these steps:

  1. Go to the IBM Cloud Private Dashboard. From the menu, click **Catalog**.

  2. Search for the `rabbitmq` package and click it to deploy it.

![RabbitMQ package](https://www.ibm.com/cloud/garage/images/tutorials/cloud-private-spring-cloud/spring-cloud4.png)

  3. Click **Configure**.

  4. For the release name, type `rabbitmq`. Use the default value for the target namespace.

  5. Select the check box to accept the license agreements.

  6. Click **Install**.

![Install](https://www.ibm.com/cloud/garage/images/tutorials/cloud-private-spring-cloud/spring-cloud5.png)

  7. Click **View Helm Release**. In the STATUS column, make sure that the word `DEPLOYED` is shown.

![View Helm Release](https://www.ibm.com/cloud/garage/images/tutorials/cloud-private-spring-cloud/spring-cloud6.png)

  8. Get the RabbitMQ password to use to install the spring-stack chart by entering this command:

`kubectl get secrets rabbitmq-rabbitmq -o=jsonpath='{.data.rabbitmq-password}' | base64 --decode; echo`

Make a note of the password.

[

### Task 3: Add the ibmcase-spring package to the IBM Cloud Private catalog

](https://www.ibm.com/cloud/garage/tutorials/cloud-private-spring-cloud?task=3/)

Spring Cloud Netflix projects are packaged for deployment as a Helm package named ibmcase-spring. The package is available in a public Git repository. 

Add the ibmcase-spring package to the IBM Cloud Private catalog by following these steps:

  1. Go to the IBM Cloud Private Dashboard. From the menu, click **Admin > Repositories**.

  2. Click **Add Repository**.

  3. For the repository name, type `ibmcase-spring`.

  4. For the URL, type `https://raw.githubusercontent.com/ibm-cloud-architecture/refarch-cloudnative-spring/master/docs/charts/`.

  5. Click **Add** to add the repository.

![Add repository](https://www.ibm.com/cloud/garage/images/tutorials/cloud-private-spring-cloud/spring-cloud7.png)

[

### Task 4: Deploy the Spring Cloud package

](https://www.ibm.com/cloud/garage/tutorials/cloud-private-spring-cloud?task=4/)

  1. Go to the IBM Cloud Private Dashboard. From the menu, click **Catalog**.

  2. Search for `spring-stack` and click it to deploy it.

  3. Configure the package:

a. Click **Configure**.

b. In the Configuration section, for the release name, type `spring-stack`.

c. In the global section, type these values:

    * rabbitmq.host: `rabbitmq-rabbitmq`
    * rabbitmq.username: `user`
    * rabbitmq.password: The password from Task 2, Step 8.

d. In the spring-config-server section, type these values:

    * spring.cloud.config.server.git.uri: `https://github.com/ibm-cloud-architecture/fortune-teller`
    * spring.cloud.config.server.git.searchPaths: `configuration`

e. In the spring-eureka-server section, for the service.type value, type `NodePort`.

f. In the spring-hystrix-dashboard section, for the service.type value, type `NodePort`.

g. In the spring-turbine-server section, for the service.type value, type `NodePort`.

h. In the spring-zuul-proxy section, for the service.type value, type `NodePort`.

i. In the spring-zipkin-server section, for the service.type value, type `NodePort`.

  4. Click **Install**.

  5. Click **View Helm Release**. In the STATUS column for spring-stack, make sure that the word `DEPLOYED` is shown.

  6. From the menu, click **Workloads > Deployments**. In the search field, type `spring-stack` to display the spring-stack deployments:

    * spring-strack-spring-config-server
    * spring-stack-spring-eureka-server
    * spring-stack-spring-hystrix-dashboard
    * spring-stack-spring-turbine-server
    * spring-stack-spring-zuul-proxy
    * bspring-stack-spring-zipkin-server
  7. Check the READY and AVAILABLE columns for each deployment, and make sure that a `1` is shown. That value represents the number of running instances.

![Deployments](https://www.ibm.com/cloud/garage/images/tutorials/cloud-private-spring-cloud/spring-cloud8.png)

You deployed the spring-stack package to install the Spring Cloud Netflix projects on IBM Cloud Private.

[

### Task 5: Access the Eureka Dashboard

](https://www.ibm.com/cloud/garage/tutorials/cloud-private-spring-cloud?task=5/)

  1. From the menu, click **Workloads > Deployments**. 

  2. Click **spring-stack-spring-eureka-server**. 

  3. Scroll to the Expose Details section and click the **access http** link.

![The access http link](https://www.ibm.com/cloud/garage/images/tutorials/cloud-private-spring-cloud/spring-cloud9.png)

All the components are ready for use when the "Instances currently registered with Eureka" table shows these values: `EUREKA`, `HYSTRIX-DASHBOARD`, and `ZIPKIN-TRACING`.

[

### Task 6: Deploy a sample Spring workload

](https://www.ibm.com/cloud/garage/tutorials/cloud-private-spring-cloud?task=6/)

Fortune Teller is a basic Spring Cloud sample application that uses Spring Cloud Netflix projects. The application is composed of two services: 

  * The Fortune Service, which serves up random fortune cookie fortunes
  * The Fortune user interface, which consumes the Fortune service

The Spring Config Server has a ConfigMap for spring-config-server-bootstrap. This ConfigMap contains the configuration data for fortune-teller deployments and is set in fortune-teller deployment files. 

In this task, you find the name of the ConfigMap file, download the fortune-teller deployment files, set the ConfigMap name in the deployment files, and deploy the fortune-teller workload. 

  1. Find the name of the ConfigMap by going to the IBM Cloud Private Dashboard, clicking **Workloads > ConfigMaps**, and searching for `spring-config-server-bootstrap`. 

The map is likely named spring-stack-spring-config-server-bootstrap.

Note the name of the map. 

![Search for the ConfigMap](https://www.ibm.com/cloud/garage/images/tutorials/cloud-private-spring-cloud/config-map.png)

  2. Download fortune-teller deployment files by running these commands:

    * `wget -O /tmp/ft-svc.yaml https://raw.githubusercontent.com/ibm-cloud-architecture/fortune-teller/master/fortune-teller-fortune-service/kube-deployment.yaml`

    * `wget -O /tmp/ft-ui.yaml https://raw.githubusercontent.com/ibm-cloud-architecture/fortune-teller/master/fortune-teller-ui/kube-deployment.yaml`

    * `wget -O /tmp/ft-ui-svc.yaml https://raw.githubusercontent.com/ibm-cloud-architecture/fortune-teller/master/fortune-teller-ui/kube-service.yaml`

  3. Set the spring-config-bootstrap ConfigMap name in the deployment files so that the fortune-service and fortune-ui pick up the spring-config server and its configuration. Then, fortune-service and fortune-ui can register against Eureka.

In these `sed` commands, replace `spring-stack-spring-config-server-bootstrap` with the name of the ConfigMap that you found in Step 1.

    * `sed -i "s/spring-config-server-bootstrap/spring-stack-spring-config-server-bootstrap/" /tmp/ft-svc.yaml`

    * `sed -i "s/spring-config-server-bootstrap/spring-stack-spring-config-server-bootstrap/" /tmp/ft-ui.yaml`

**Note:** If `sed` is not installed, or if `sed -i` does not work, open the `/tmp/ft-svc.yaml` file and the `/tmp/ft-ui.yaml` file in a text editor. Find and replace `spring-config-server-bootstrap` with the ConfigMap name that you found in Step 1.

  4. Deploy the fortune-teller workload by entering these commands:

    * `kubectl create -f /tmp/ft-svc.yaml`
    * `kubectl create -f /tmp/ft-ui.yaml`
    * `kubectl create -f /tmp/ft-ui-svc.yaml`
  5. Go to the IBM Cloud Private Dashboard. From the menu, click **Workloads > Deployments**. 

  6. Search for `fortune` to display the fortune-teller-fortune-service and the fortune-teller-ui deployments. Before you proceed to the next step, check the READY and AVAILABLE columns for each deployment and make sure that a `1` is shown. That value represents the number of running instances.

![Fortune Teller deployments](https://www.ibm.com/cloud/garage/images/tutorials/cloud-private-spring-cloud/spring-cloud10.png)

  7. Go to the Eureka dashboard and confirm that the FORTUNES and UI services are registered with Eureka.

![Eureka Dashboard](https://www.ibm.com/cloud/garage/images/tutorials/cloud-private-spring-cloud/eureka-dashboard2.png)

  8. Go to the IBM Cloud Private Dashboard and click **Workloads > Deployments**. Search for `fortune-teller`. Click **fortune-teller-ui**. 

  9. Scroll to the Expose Details section and click the **access http** link.

![Fortune Teller](https://www.ibm.com/cloud/garage/images/tutorials/cloud-private-spring-cloud/spring-cloud11.png)

[

### What's next

](https://www.ibm.com/cloud/garage/tutorials/cloud-private-spring-cloud?task=7/)
