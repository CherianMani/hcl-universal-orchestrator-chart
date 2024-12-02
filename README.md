# HCL Universal Orchestrator

## Introduction
HCL Universal Orchestrator is a cloud-native process orchestrator and requires to be deployed on a Kubernetes platform, either on a public or a private cloud infrastructure. A cloud deployment ensures access anytime anywhere and is a fast and efficient way to get up and running quickly. It also simplifies maintenance, lowers costs, provides rapid upscale and downscale, minimizes IT requirements and physical on-premises data storage.

As more and more organizations move their critical applications to the cloud, there is an increasing demand for solutions and services that help them easily migrate and manage their cloud environment.

To respond to the growing request to make automation opportunities more accessible, HCL Universal Orchestrator containers can be deployed into the following supported third-party cloud provider infrastructures:

- ![Amazon EKS](images/tagawseks.png "Amazon EKS") Amazon Web Services (AWS) Elastic Kubernetes Service (EKS)
- ![Microsoft Azure](images/tagmsa.png "Microsoft Azure") Microsoft&reg; Azure Kubernetes Service (AKS)
- ![Google GKE](images/taggke.png "Google GKE") Google Kubernetes Engine (GKE)
- ![OpenShift](images/tagOpenShift.png "OpenShift") OpenShift (OCP)

HCL Universal Orchestrator is a complete, modern solution to orchestrate calendar-based and event-driven tasks, business and IT processes. It enables organizations to gain complete visibility and control over attended or unattended workflows. From a single point of control, it supports multiple platforms and provides advanced integration with enterprise applications including ERP, Business Analytics, File Transfer, Big Data, and Cloud applications.

For more information about HCL Universal Orchestrator, see the product documentation library in [HCL Universal Orchestrator documentation](https://help.hcltechsw.com/UnO/v1.1/index.html).

## Details

By default, all microservices and the Dynamic Workload Console (console) are installed. 

To achieve high availability in an HCL Universal Orchestrator environment, the minimum base configuration is composed of 2 Dynamic Workload Consoles and 2 replicas of all microservices. For more details about HCL Universal Orchestrator and high availability, see: 


[An active-active high availability scenario](https://help.hcltechsw.com/UnO/v1.1/Mobile_guides/highavailability.html).

HCL Universal Orchestrator can be deployed across a single cluster, but you can add multiple instances of the product microservices by using a different namespace in the cluster. The product microservices can run in multiple failure zones in a single cluster.

 
## Supported Platforms

- ![Amazon EKS](images/tagawseks.png "Amazon EKS") Amazon Elastic Kubernetes Service (EKS) on amd64: 64-bit Intel/AMD x86
- ![Microsoft Azure](images/tagmsa.png "Microsoft Azure") Azure Kubernetes Service (AKS) on amd64: 64-bit Intel/AMD x86
- ![Google GKE](images/taggke.png "Google GKE") Google Kubernetes Engine (GKE) on amd64: 64-bit Intel/AMD x86
- ![OpenShift](images/tagOpenShift.png "OpenShift") OpenShift (OCP)
- Any Kubernetes platform from V1.29 and above

HCL Universal Orchestrator supports all the platforms supported by the runtime provider of your choice.

### Openshift support
You can deploy HCL Universal Orchestrator on Openshift by following the instruction in this documentation and using helm charts. 
Ensure you modify the value of the `waconsole.console.exposeServiceType` parameter from `LoadBalancer` to `Routes`.
	
## Accessing the container images

You can access the HCL Universal Orchestrator chart and container images from the Entitled Registry. See [Create the secret](#creating-the-secret) for more information about accessing the registry. The images are as follows:

 - hcl-uno-agentmanager
 - hcl-uno-gateway
 - hcl-uno-iaa
 - hcl-uno-scheduler
 - hcl-uno-storage
 - hcl-uno-toolbox
 - hcl-uno-audit
 - hcl-uno-timer
 - hcl-uno-executor
 - hcl-uno-eventmanager 
 - hcl-uno-orchestrator
 - hcl-uno-console
 - hcl-uno-notification


## Prerequisites
Before you begin the deployment process, ensure your environment meets the following prerequisites:

**Mandatory**
 - Kubectl v 1.29.4 or later
 - Kubernetes cluster v 1.29 or later
 - Helm v 3.12 or later
 - Messaging system: Apache Kafka v 3.4.0 or later OR Redpanda v 23.11 or later 
 - Database: MongoDB v 5 or later OR Azure Cosmos DB for MongoDB (vCore) OR DocumentDB for AWS deployment
 - HCL UnO agent: Java SDK21

**Strongly recommended**

 - Jetstack cert-manager

  We strongly recommend the use of a cert-manager as it automatically generates and updates the required certificates. You can choose not to use it, in which case you need to:
         
   - Create your own custom certificates
   - Insert the certificates inside the correct Kubernetes secrets
   - Make sure that the names of the Kubernetes secrets match the names specified in the `values.yaml`deployment file

**Optional**
-   Grafana and Prometheus for monitoring dashboard

The following are prerequisites specific to each supported cloud provider:

![Amazon EKS](images/tagawseks.png "Amazon EKS") 
- Amazon Kubernetes Service (EKS) installed and running
- AWS CLI (AWS command line)

![Microsoft Azure](images/tagmsa.png "Microsoft Azure") 
- Azure Kubernetes Service (AKS) installed and running
- azcli (Azure command line)

![Google GKE](images/taggke.png "Google GKE") 
- Google Kubernetes Engine (GKE) installed and running
- gcloud SDK (Google command line)


## Resources Required
  
 The following resources correspond to the default values required to manage a production environment. These numbers might vary depending on the environment.
 
| Component | Container resource limit | Container resource request |
|--|--|--|
|**uno-orchestrator microservice**  | CPU: 2, Memory: 1 GB  |CPU: 0.3, Memory: 1 GB|
|**Each remaining microservice**  | CPU: 2, Memory: 1 GB  |CPU: 0.3, Memory: 0.5 GB  |
|**Console**  | CPU: 4, Memory: 16 GB  |CPU: 1, Memory: 4 GB, Storage: 5 GB  |

No disk space is required for the microservices, however, at least 100 GB are recommended for Kafka and 100 GB for MongoDB. Requirements vary depending on your workload.

## Deploying

Deploying and configuring HCL Universal Orchestrator involves the following high-level steps:

1. [Creating the Namespace](#creating-the-namespace)
2. [Creating a Kubernetes Secret](#creating-the-secret) by accessing the entitled registry to store an entitlement key for the HCL Universal Orchestrator offering on your cluster. 
3. [Deploying the product components](#deploying-the-product-components)
4. [Verifying the deployment](#verifying-the-deployment)


### Creating the Namespace

To create the namespace, run the following command:

        kubectl create namespace <uno_namespace>
	

### Creating the Secret 

If you already have a license, then you can proceed to obtain your entitlement key. To learn more about acquiring an HCL Universal Orchestrator license, contact HWAinfo@hcl.com. 

Obtain your entitlement key and store it on your cluster by creating a [Kubernetes Secret](https://kubernetes.io/docs/concepts/configuration/secret/). Using a Kubernetes secret allows you to securely store the key on your cluster and access the registry to download the chart and product images. 

1. Access the entitled registry.  Contact your HCL sales representative for the login details required to access the HCL Entitled Registry.
2. To create a pull secret for your entitlement key that enables access to the entitled registry, run the following command:

         kubectl create secret docker-registry -n <uno_namespace> sa-<uno_namespace> --docker-server=<registry_server> --docker-username=<user_name> --docker-password=<password>
	   
	where,
	* <uno_namespace> represents the namespace where the product components are installed
	* <registry_server> is `hclcr.io`
	* <user_name> is the user name provided by your HCL representative
	* \<password> is the entitled key copied from the entitled registry `<api_key>`


### Deploying the product components		

Before starting to deploy the product components, make sure that all the [prerequisites](#prerequisites) are met.

To deploy HCL Universal Orchestrator, perform the following steps:

1. Log into the registry:
 
        helm registry login hclcr.io  
   
2. Pull the Helm chart:

        helm pull oci://hclcr.io/uno/hcl-uno-chart
	
**Note:** If you want to download a specific version of the chart use the `--version` option in the `helm pull` command.
	
3. Customize the deployment. Configure each product component by adjusting the values in the `values.yaml` file. The `values.yaml`file contains a detailed explanation for each parameter you must provide values 
   for the following mandatory parameters:
   
   **License**
   
   license: Indicates whether you accept the license or not. Supported values are accept and not accepted. You must set the value as accept.
   
   **database**

   * url: URL of the database.

          url: mongodb://hcl-uno-db-mongodb.db.svc.cluster.local:27017

   * type: database type and supported value is mongodb.
   * databaseName: database name and supported value is uno.
   * username: name of the database user.
   * password: password of the database user.
   * tls: specifies whether TLS is enabled or not and supported values are true or false.
   * tlsInsecure: tlsInsecure specifies whether certificate validation must be skipped or not and supported values are true or false.

    **kafka**

   * url: URL of the event streaming platform. For example,

          url: hcl-uno-kafka-0.kafka-headless.kafka.svc.cluster.local:9092

   * username: name of the kafka user.
   * password: password of the kafka user.
   * tls: specify whether TLS is enabled or not and supported values are true or false.
   * saslMechanism : specify the Kafka sasl mechanism of the user. For example, 
     
          saslMechanism: PLAIN

   * jaasConfig: specify the configuration string that defines the authentication details for the given SASL mechanism. For example,
    
          jaasConfig: org.apache.kafka.common.security.plain.PlainLoginModule required username="my-user" password="my-password";

   * securityProtocol: Specify the communication protocol that kafka supports. For example,
       
          securityProtocol: PLAINTEXT

   * tlsInsecure: tlsInsecure specifies whether certificate validation must be skipped or not and supported values are true or false.
  
    **Enable an OIDC provider**

      You can enable an OIDC user registry by configuring the `values.yaml` deployment file as follows:

        uno.authentication.oidc.enabled=true

      Enter the required values into the `authentication` section of the `values.yaml` file, according to your OIDC provider. If the OIDC you are using has custom certificates, to connect your machine to your 
      OIDC provider you must use the certificate as secret in the following parameter of under `certificates` section:

        uno.config.certificates.additionalCASecrets : Specify the secret.

   You can customize the deployment environment by setting up the following optional parameters: 

     
     **Session timeout**


   After a period of inactivity on the UI, users are automatically logged out. By default, the session timeout is set to 30 minutes, and a warning message appears 2 minutes before the session expires to 
   alert users about their inactivity. You can change the session timeout value by modifying the following parameter in the values.yaml file of the Helm chart:
         
        uno.config.console.sessionTimeoutMinutes : 30

     **Log out option**
   
      By default, the log out option is not enabled on the UI. If you want to display the log out icon, you must set the following parameter in the values.yaml file of the Helm chart:
   
        uno.config.console.enableLogout : true

     **Generative workflows and RAG**
   
     You can leverage the generative AI features of the UnO AI Pilot to turn a description into an effective workflow, as explained in Generative workflows.
     You can leverage the feature by enabling the following parameter of the values.yaml file of the Helm chart:
      
        uno.config.console.genai.enabled : true

   Optionally, you can set the proxy.

     **Justifications**
   
     Administrators can request to specify a justification when saving or editing items to keep track of the changes in the environment.
     To display the justification panel every time a user performs a change, set the following parameter in the values.yaml file of the Helm chart to true:
     
        uno.config.console.engine.justificationEnabled : true

      You can also define that a justification is mandatory so that users cannot save the changes until one of the justifications or also all of them have been provided. To require mandatory justifications, set 
      the related parameters in the values.yaml file of the Helm chart as follows:

	     
        uno.config.console.engine.justificationCategoryRequired : true
        uno.config.console.engine.justificationTicketNumberRequire: true
        uno.config.console.engine.justificationDescriptionRequired: true

      For more information about justifications, see [Keeping track of changes in your environment](https://help.hcl-software.com/UnO/v2.1/Deployment/justifications.html).

4. Deploy the instance by running the following command: 

        helm install -f values.yaml <uno_release_name> <repo_name>/hcl-uno-chart -n <uno_namespace>


 where 
   <uno_release_name> is the deployment name of the instance. 
**TIP:** Use a short name or acronym when specifying this value to ensure it is readable.

The following are some useful Helm commands:

* To list all of the Repo releases: 

        helm list -A
	
* To update the Helm release:

        helm upgrade <uno_release_name> <repo_name>/hcl-uno-chart -f values.yaml -n <uno_namespace>
		
* To update helm repo release:
  
        helm repo update
	
* To delete the Helm release: 

        helm uninstall <uno_release_name> -n <uno_namespace>

		

### Verifying the deployment 

After the deployment procedure is complete, you can validate the deployment to ensure that everything is working. 

To manually verify that the deployment has successfully completed, perform the following check:
 
Run the following command to verify the pods installed in the <uno_namespace>:
   
           kubectl get pods -n <uno_namespace>


### Security and verification for OCLI and UnO agent binaries 

To ensure the integrity and authenticity of the downloaded files, we use GPG (GNU Privacy Guard) encryption. You must have the GPG tool installed on your system to decrypt and verify the files. 

The Orchestration CLI and HCL UnO agent packages are signed with our private key. A corresponding .asc signature file accompanies the downloadable file. You can extract the file and use the public key to decrypt and verify the files. 

For more information on verifying a file with gpg keys, see [GnuPG documentation](https://www.gnupg.org/gph/en/manual.html). 

When you decrypt the files with the public key and if the signature is valid, you can see a message indicating the file is correctly signed and the key ID matches with the public key. If the signature is invalid, you can see an error message, means the file is corrupted. 

By verifying the file, you can ensure that it is not tampered during the download and can confirm the file is genuinely valid. You can download the public key from [here](https://github.com/CherianMani/hcl-universal-orchestrator-chart/blob/task/WA-135255/public_key.gpg.zip).
 
**Verify that the default engine connection is created from the Dynamic Workload Console**

Verifying the default engine connection depends on the network enablement configuration you implement. To determine the URL to be used to connect to the console, follow the procedure for the appropriate network enablement configuration.

**For load balancer:**

1. Run the following command to obtain the token to be inserted in https://\<loadbalancer>:9443/console to connect to the console:

![Amazon EKS](images/tagawseks.png "Amazon EKS") 
	
        kubectl get svc <workload_automation_release_name>-waconsole-lb  -o 'jsonpath={..status.loadBalancer.ingress..hostname}' -n <workload_automation_namespace>

![Microsoft Azure](images/tagmsa.png "Microsoft Azure")

       kubectl get svc <workload_automation_release_name>-waconsole-lb  -o 'jsonpath={..status.loadBalancer.ingress..ipaddress}' -n <workload_automation_namespace>
       

![Google GKE](images/taggke.png "Google GKE")

       kubectl get svc <workload_automation_release_name>-waconsole-lb  -o 'jsonpath={..status.loadBalancer.ingress..ipaddress}' -n <workload_automation_namespace>


2. With the output obtained, replace \<loadbalancer> in the URL https://\<loadbalancer>:9443/console.

**For ingress:**

1. Run the following command to obtain the token to be inserted in https://\<ingress>/console to connect to the console:


        kubectl get ingress/<workload_automation_release_name>-waconsole -o 'jsonpath={..host}'-n <workload_automation_namespace>
  
2.   With the output obtained, replace \<ingress> in the URL https://\<ingress>/console.

**Logging into the console:**

1. Log in to the console by using the URLs obtained in the previous step.

2. For the credentials, specify the user name (wauser) and password (wa-pwd-secret, the password specified when you created the secrets file to store passwords for the server, console and database).
	
3. From the navigation toolbar, select **Administration -> Manage Engines**.
	
4.  Verify that the default engine, **engine_<release_name>-gateway** is displayed in the Manage Engines list:

To ensure the Dynamic Workload Console logout page redirects to the login page, modify the value of the logout url entry available in file authentication_config.xml:


       <jndiEntry value="${logout.url}" jndiName="logout.url" />

where the logout.url string in jndiName should be replaced with the logout URL of the provider.

## Upgrading the product components

To upgrade HCL Universal Orchestrator, perform the following steps:

1. Configure each product component by adjusting the values in the `values.yaml` file. The `values.yaml`file contains a detailed explanation for each parameter.

2. Upgrade the instance by running the following command:

         helm upgrade <uno_release_name> <repo_name>/hcl-uno-chart -f values.yaml -n <uno_namespace>
		 
 where 
   <uno_release_name> is the deployment name of the instance. 
**TIP:** Use a short name or acronym when specifying this value to ensure it is readable.

**Note:** When upgrading HCL Universal Orchestrator from V1.1.0 to V1.1.2 or later, or from V1.1.1 to V1.1.2 or later, you must manually delete the older version of the executable job plug-in.
	   
## Uninstalling the Chart

 To uninstall the deployed components associated with the chart and clean up the orphaned Persistent Volumes, run:
 
         helm uninstall release_name -n <uno_namespace> 
  
	
 The command removes all of the Kubernetes components associated with the chart and uninstalls the <uno_release_name> release.
 	
## Configuring

Configuration parameters are available in the **values.yaml** files, together with explanatory comments.

The following procedures are ways in which you can configure the default deployment of the product components. They include the following configuration topics:

* [Network enablement](#network-enablement)
* [Scaling the product](#scaling-the-product)
* [Managing your custom certificates](#managing-your-custom-certificates)

### Network enablement

The HCL Universal Orchestrator server and console can use two different ways to route external traffic into the Kubernetes Service cluster:

* A **load balancer** service that redirects traffic
* An **ingress** service that manages external access to the services in the cluster

You can freely switch between these two types of configuration.

#### Network policy

You can specify an egress network policy to include a list of allowed egress rules for each components. Each rule allows traffic leaving the cluster which matches both the "to" and "ports" sections. For example, the following sample demonstrates how to allow egress to another destination:

networkpolicyEgress:

	- name: to-mdm
	  egress:
	  - to:
	    - podSelector:
	        matchLabels:
		  app.kubernetes.io/name: waserver
	    - port: 31116
	      protocol: TCP
	- name: dns
	  egress:
	    - to:
	      - namespaceSelector:
	          matchLabels:
		    name: kube-system
	    - ports:
	        - port: 53
		  protocol: UDP
		- port: 53
		  protocol: TCP

For more information, see [Network Policies](https://kubernetes.io/docs/concepts/services-networking/network-policies/).

#### Node affinity Required
You can also specify node affinity required to determine on which nodes a component can be deployed using custom labels on nodes and label selectors specified in pods. The following is an example:

nodeAffinityRequired:

	-key: iwa-node
	  operator: In
	  values:
	  - 'true'

where **iwa-node** represents the value of the node affinity required.

#### Load balancer service


  To configure a load balancer, follow these steps:

1. Locate the following parameters in the `values.yaml` file:

		exposeServiceType
		exposeServiceAnnotation

For more information about these configurable parameters, see the explanatory comments available in the **values.yaml** file.

2. Set the value of the `exposeServiceType` parameter to `LoadBalancer`.

3. In the `exposeServiceAnnotation` section, uncomment the lines in this section as follows:

![Amazon EKS](images/tagawseks.png "Amazon EKS") 

		service.beta.kubernetes.io/aws-load-balancer-type: nlb
		service.beta.kubernetes.io/aws-load-balancer-internal: "true"
		
![Microsoft Azure](images/tagmsa.png "Microsoft Azure") 

		service.beta.kubernetes.io/azure-load-balancer-internal: "true"

![Google GKE](images/taggke.png "Google GKE") 

		networking.gke.io/load-balancer-type: "Internal"


4. Specify the load balancer type and set the load balancer to internal by specifying "true".


#### Ingress service

  To configure an ingress for the server, follow these steps:

1. Locate the following parameters in the `values.yaml` file:

		exposeServiceType
		exposeServiceAnnotation

   For more information about these configurable parameters, see the explanatory comments available in the **values.yaml** file.

2. Set the value of the `exposeServiceType`parameter to `Ingress`.

3. In the `exposeServiceAnnotation` section, leave the following lines as comments:

![Amazon EKS](images/tagawseks.png "Amazon EKS") 

		#service.beta.kubernetes.io/aws-load-balancer-type:nlb
		#service.beta.kubernetes.io/aws-load-balancer-internal: "true"

![Microsoft Azure](images/tagmsa.png "Microsoft Azure")

		#service.beta.kubernetes.io/azure-load-balancer-internal: "true"	
		
![Google GKE](images/taggke.png "Google GKE") 

                #networking.gke.io/load-balancer-type: "Internal"

	
### Scaling the product 

HCL Universal Orchestrator is installed by default with autoscaling enabled. A single Dynamic Workload Console is installed. To scale the Dynamic Workload Console, increase or decrease the values of the `replicaCount` parameter in the `values.yaml` file and save the changes.

**Note**: HCL Universal Orchestrator Helm chart does not support scaling to zero nor proportional scaling.
		  
### Managing your custom certificates
    
  If you want to use custom certificates, set `useCustomizedCert:true` and use kubectl to apply the secret in the \<uno_namespace>.
 Type the following command:
 
 ```
kubectl create secret generic waserver-cert-secret --from-file=TWSClientKeyStore.kdb --from-file=TWSClientKeyStore.sth --from-file=TWSClientKeyStoreJKS.jks --from-file=TWSClientKeyStoreJKS.sth --from-file=TWSServerKeyFile.jks --from-file=TWSServerKeyFile.jks.pwd --from-file=TWSServerTrustFile.jks --from-file=TWSServerTrustFile.jks.pwd -n <workload-automation-namespace>   
 ``` 
  
    
> **Note:** if you set `db.sslConnection:true`, you must also set the `useCustomizedCert` setting to true on both UnO and Dynamic Workload Console charts and, in addition, you must add the following certificates in the customized SSL certificates secret on both UnO and Dynamic Workload Console charts:

  * ca.crt
  * tls.key
  * tls.crt

 Customized files must have the same name as the ones listed above.
         
If you want to use SSL connection to DB, set `db.sslConnection:true` and `useCustomizedCert:true`, then use kubectl to create the secret in the same namespace where you want to deploy the chart:

      bash
      $ kubectl create secret generic release_name-secret --from-file=TWSServerTrustFile.jks --from-file=TWSServerKeyFile.jks --from-file=TWSServerTrustFile.jks.pwd --from-file=TWSServerKeyFile.jks.pwd --namespace=<uno_namespace>
        
If you define custom certificates, you are in charge of keeping them up to date, therefore, ensure you check their duration and plan to rotate them as necessary. To rotate custom certificates, delete the previous secret and upload a new secret, containing new certificates. The pod restarts automatically and the new certificates are applied.

**Managing DocumentDB during AWS deployment**

When deploying Universal orchestrator on AWS, you can leverage DocumentDB, a fully managed NoSQL database service provided by AWS. You must configure few parameters in the values.yaml file to ensure compatibility with DocumentDB. The parameters that must be configured are as follows:
In **db** section,
* type: Specify the preferred remote database server, DocumentDB in this case.
* hostname: Specify the hostname or the IP address of the DocumentDB database server.
* name: Specify the name of the DocumentDB database server.
* port: Specify the port of the DocumentDB database server.
* sslConnection: Set the value to true.
* retryWrites: Only for documentDB. Supported values are true or false.
* readPreference: Only for documentDB. Specify the read preference. Supported values are primary, primaryPreferred, secondary, secondaryPreferred, nearest.

You must also set the following parameters under the **certificates** section to connect your machine to the DocumentDB. 
* UseCustomizedCert: Set the value to true and add the certificate as a secret.
* AdditionalCASecrets : Specify the name of the certificate that you set as a secret.

## Metrics monitoring 

HCL Universal Orchestrator uses Grafana to display performance data related to the product. This data includes metrics related to the server and console application server (Open Liberty), your workload, your workstations, message queues, the database connection status, and more. Grafana is an open source tool for visualizing application metrics. Metrics provide insight into the state, health, and performance of your deployments and infrastructure. HCL Universal Orchestrator cloud metric monitoring uses an opensource Cloud Native Computing Foundation (CNCF) project called Prometheus. It is particularly useful for collecting time series data that can be easily queried. Prometheus integrates with Grafana to visualize the metrics collected.



The following metrics are collected and available to be visualized in the preconfigured Grafana dashboard. The dashboard is named **<uno_namespace> <uno_release_name>**:

For a list of metrics exposed by HCL Universal Orchestrator, see [Exposing metrics to monitor your workload](https://help.hcltechsw.com/UnO/v1.1/Monitoring/awsrgmonprom.html).
  
  ### Setting the Grafana service
Before you set the Grafana service, ensure that you have already installed Grafana and Prometheus on your cluster. For information about deploying Grafana see [Install Grafana](https://github.com/helm/charts/blob/master/stable/grafana/README.md). For information about deploying the open-source Prometheus project see [Download Promotheus](https://github.com/helm/charts/tree/master/stable/prometheus).
  
1. Log in to your cluster. To identify where Grafana is deployed, retrieve the value for the \<grafana-namespace> by running:
  
          helm list -A
		  
2. Download the grafana_values.yaml file by running:

        helm get values grafana -o yaml -n <grafana-namespace> grafana_values.yaml

3. Modify the grafana_values.yaml file by setting the following parameter values:	

	    dashboards:
		     SCProvider: true
		     enabled: true
		     folder: /tmp/dashboards
		     label: grafana_dashboard
		     provider:
			 allowUiUpdates: false
			 disableDelete: false
			 folder: ""
			 name: sidecarProvider
			 orgid: 1
			 type: file
		     searchNamespace: ALL

4. Update the grafana_values.yaml file in the Grafana pod by running the following command:

    ` helm upgrade grafana stable/grafana -f grafana_values.yaml -n <grafana-namespace>`
					 
5. To access the Grafana console:

     a. Obtain the EXTERNAL-IP address value of the Grafana service by running:
	 
        kubectl get services -n <grafana-namespace>
		
     b. Browse to the EXTERNAL-IP address and log in to the Grafana console.		
 
### Viewing the preconfigured dashboard in Grafana

To get an overview of the cluster health, you can view a selection of metrics on the predefined dashboard:

1. In the left navigation toolbar, click **Dashboards**.

2. On the **Manage** page, select the predefined dashboard named **<uno_namespace> <uno_release_name>**.

For more information about using Grafana dashboards see [Dashboards overview](https://grafana.com/docs/grafana/latest/features/dashboard/dashboards/).


## Limitations

*  Limited to amd64 platforms.
*  Anonymous connections are not permitted.


## Documentation

To access the complete product documentation library for HCL Universal Orchestrator, see [HCL Universal Orchestrator documentation](https://help.hcltechsw.com/UnO/v1.1/index.html).



## Change history

### Added August 2023
First release


