# Oracle SOA Suite on OKE


> **Note**:   Oracle SOA Suite on Oracle Kubernetes Engine (OKE) is currently supported only for non-production use. The information provided in this document is a *preview* for customers who wish to experiment with Oracle SOA Suite on Oracle Kubernetes Engine (OKE) before it is supported for production use.

### Introduction
Use this Quick Start to create an Oracle SOA Suite domain deployment in a Kubernetes cluster with the Oracle WebLogic Server Kubernetes Operator. Note that this walkthrough is for demonstration purposes only, not for use in production.
These instructions assume that you are already familiar with Kubernetes. If you need more detailed instructions,
refer to the [Install Guide](https://oracle.github.io/fmw-kubernetes/soa-domains/installguide/).

### Set up Oracle SOA Suite on OKE

Perform the steps in this topic to create an Oracle Cloud Infrastructure Container Engine for Kubernetes (OKE) with Single Worker node and then create an Oracle SOA Suite osb domain type, which deploys a domain with Oracle Service Bus (OSB).

* [Step 1 - Prepare tenancy for OKE](#1-prepare-tenancy-for-oke)
* [Step 2 - Create a 'Quick Cluster'](#2-create-a-quick-cluster)
* [Step 3 - Get the Setup Scripts](#3-get-the-setup-scripts)
* [Step 4 - Prepare the worker node for Oracle SOA Suite domain](#4-prepare-the-worker-node-for-oracle-soa-suite-domain)
* [Step 5 - Install WebLogic Kubernetes Operator](#5-install-weblogic-kubernetes-operator)
* [Step 6 - Install the Traefik (ingress-based) Load Balancer](#6-install-the-traefik-ingress-based-load-balancer)
* [Step 7 - Create and Configure Oracle SOA Suite Domain](#7-create-and-configure-oracle-soa-suite-domain)
* [Step 8 - Scaling an Oracle Service Bus Cluster](#8-scaling-an-oracle-service-bus-cluster)

### 1. Prepare tenancy for OKE

You must have an Oracle Cloud Infrastructure enabled account.
To create the Container Engine for Kubernetes (OKE), complete the following steps:

* Create the network resources (VCN, subnets, security lists, etc.)
* Create a cluster
* Create a NodePool

This guide shows you the way the **Quick Create** feature creates and configures all the necessary resources for a three node Kubernetes cluster. All the nodes will be deployed in different availability domains to ensure high availability.

For more information about OKE and custom cluster deployment, see the Oracle Container Engine documentation.

Follow the below steps to prepare the tenancy for OKE:

1. Sign in using the link you got in an email during the registration process. If this is your first time signing in, you have to change the generated first-time password.

1. Open the OCI console and enter the Cloud tenant details provide in email and Click **Continue**

1. Next click **Continue** on Single Sign-On(SSO) page

1. Enter the **User name** and **Password** provided for you. Click **Sign In**.

1. Once you sign-in, lands at the Oracle Cloud Home page

1. Next create a policy that allows OKE to create resources in your tenancy. An OKE resource policy or policies let you specify which groups in your tenancy can perform certain tasks with the OKE API.
   Optionally, create more resource policies if you want to control which groups can access different parts of the OKE service.

    a) Open the navigation menu and Scroll down to **Governance and Administration**. Under **Identity**, select **Policies**.

    b) Select the desired compartment from the drop down list on the left. Click **Create Policy**.

    c) Enter the following and Click **Create**:
      *  **Name**: A unique name for the policy. The name must be unique across all policies in your tenancy. You cannot change this later.
      *  **Description**: A user friendly description.
      *  **Policy Versioning**: Select Keep Policy Current. It ensures that the policy stays current with any future changes to the service's definitions of verbs and resources.
      *  **Statement**: A policy statement. It MUST be: `allow service OKE to manage all-resources in tenancy`.
      *  **Tags**: Don't apply tags.

### 2. Create a 'Quick Cluster'
Before creating Cluster, we need to create the SSH Key Pair for access the worker node. Here we will use the OCI Cloud Shell to create the required SSH Key Pair.

1. To access the OCI Cloud Shell -

   Oracle Cloud Infrastructure (OCI) Cloud Shell is a web browser-based terminal, accessible from the Oracle Cloud Console. Cloud Shell provides access to a Linux shell, with a pre-authenticated Oracle Cloud Infrastructure CLI and other useful tools (Git, kubectl, helm, OCI CLI) to complete the operator tutorials. Cloud Shell is accessible from the Console. Your Cloud Shell will appear in the Oracle Cloud Console as a persistent frame of the Console, and will stay active as you navigate to different pages of the Console.
      Follow below steps to access the OCI Cloud Shell:

      a) Click the Cloud Shell icon in the Console header (top right area in the browser).

      b) Wait a few seconds for the Cloud Shell to appears. You can minimize and restore the terminal size at any time.

1. Creating an SSH Key Pair on the Command Line
   Follow below steps to create SSH key Pair on the OCI Cloud Shell:

      a) At the prompt, enter `ssh-keygen` and provide a name for the key when prompted. Optionally, include a passphrase. The keys will be created with the default values: RSA keys of 2048 bits.
      ```
      $ ssh-keygen
      ```
      
      Sample output with default:
      ```
      $ ssh-keygen
      Generating public/private rsa key pair.
      Enter file in which to save the key (/home/xxxx/.ssh/id_rsa): 
      Enter passphrase (empty for no passphrase): 
      Enter same passphrase again: 
      Your identification has been saved in /home/xxxx/.ssh/id_rsa.
      Your public key has been saved in /home/xxxx/.ssh/id_rsa.pub.
      The key fingerprint is:
      SHA256:r/hdcqGFsIeRZ+.... xxxx@10b6e755ce0a
      The key's randomart image is:
      +---[RSA 2048]----+
      |                 |
      |         . .  . o|
      |        + +  . o.|
      |         O ..   .|
      |        S +.oo.o |
      |     .   + oE=. =|
      |      o . =.+.+B=|
      |     . + = = .==*|
      |      o.+ o.ooooo|
      +----[SHA256]-----+
      
      $ ls  ~/.ssh/
      id_rsa  id_rsa.pub  known_hosts      
      ```

      b) We will use the content of `id_rsa.pub` for *PUBLIC SSH KEY* while creating the Cluster. And the `id_rsa` will be used later to ssh to worker node once created.


To create a 'Quick Cluster' with default settings and new network resources using Container Engine for Kubernetes, follow below steps:

1. In the OCI Console, open the navigation menu. Under **Solutions and Platform**, go to **Developer Services** and click **Container Clusters (OKE)**.

1. Choose a Compartment where you have created and attached policy in Step1.

1. On the Cluster List page, select the Compartment and click **Create Cluster**.

1. In the Create Cluster dialog, select **Quick Create** and click **Launch Workflow**.

1. On the Create Cluster page specify the below values:

    * **NAME**: soaoke
    * **COMPARTMENT**: Choose appropriate compartment
    * **KUBERNETES VERSION**: v1.17.9
    * **CHOOSE VISIBILITY TYPE**: Public
    * **SHAPE**: VM.Standard2.8  (Choose the available shape for worker node pool. The list shows only those shapes available in your tenancy that are supported by Container Engine for Kubernetes. See Supported Images and Shapes for Worker Nodes.)
    * **NUMBER OF NODES**:  1 (The number of worker nodes to create in the node pool, placed in the regional subnet created for the 'quick cluster').
    * **SPECIFY A CUSTOM BOOT VOLUME SIZE**: Click the check box. And then enter **BOOT VOLUME SIZE (IN GB)** as 250.
    * **Show Advanced Options** : Click on this and then enter **PUBLIC SSH KEY** with content from `id_rsa.pub`obtained in previous step.

    Click **Next** to review the details you entered for the new cluster.

1. Click **Create Cluster** to create the new network resources and the new cluster.

1. Container Engine for Kubernetes starts creating resources (as shown in the Creating cluster and associated network resources dialog).
Click **Close** to return to the Console.

1. Initially, the new cluster appears in the Console with a status of Creating. When the cluster has been created, it has a status of Active.

1. Click on the **Node Pools** on Resources and then **View** to view the Node Pool and worker node status

1. You can view the status of Worker node and make sure the Node State in Active and Kubernetes Node Condition is Ready.
Note:  The worker node gets listed in the kubectl command once the "Kubernetes Node Condition" is Ready.

1. Access the Cluster from the OCI Cloud Shell -

   Your Cloud Shell comes with the OCI CLI pre-authenticated, so there’s no setup to do before you can start using it. To complete the kubectl configuration, click Access Cluster on your cluster detail page and then Click on "Cloud Shell Access" as shown below. (If you moved away from that page, then open the navigation menu and under Developer Services, select Kubernetes Clusters. Select your cluster and go the detail page.

   a) A dialog appears which contains the customized OCI command that you need to execute, to create a Kubernetes configuration file.
Select the Copy link to copy the `oci ce...` command to Cloud Shell, then close the configuration dialog before you paste the command into the terminal.

     For example, the command looks like the following:
     ```
     $ oci ce cluster create-kubeconfig --cluster-id ocid1.cluster.oc1.THIS_IS_EXAMPLE_DONT_COPY_PASTE_FROM_HERE --file $HOME/.kube/config --region us-phoenix-1 --token-version 2.0.0
     ```
    
   b) Now check that kubectl is working, for example, using the get node command:
     ```
     $ kubectl get nodes
     NAME        STATUS   ROLES   AGE    VERSION
     10.0.10.2   Ready    node    142m   v1.17.9
     ```
     If you see the node's information, then the configuration was successful.

   c) Set up the RBAC policy for the OKE cluster -  In order to have permission to access the Kubernetes cluster, you need to authorize your OCI account as a cluster-admin on the OCI Container Engine for Kubernetes cluster. This will require your user OCID.In the Console, select your OCI user name and select User Settings. On the user details page, you will find the user OCID. Select Copy and paste it temporarily in a text editor.

   Then execute the role binding command using your(!) user OCID:

     ```
     $ kubectl create clusterrolebinding my-cluster-admin-binding --clusterrole=cluster-admin --user=<YOUR_USER_OCID>
     ```

     Sample Output is as below
     ```
     $ kubectl create clusterrolebinding my-cluster-admin-binding --clusterrole=cluster-admin --user=ocid1.user.oc1..AGAIN_THIS_IS_EXAMPLE
     clusterrolebinding.rbac.authorization.k8s.io/my-cluster-admin-binding created
     ```

Congratulation, now your OCI OKE environment is ready to deploy your Oracle SOA Suite domain.

### 3. Get the Setup Scripts

#### 3.1 Get the Oracle WebLogic Operator Scripts

1. Create WORKDIR for Oracle SOA Operator Scripts on OCI Cloud Shell
   ```
   $ export WORKDIR=$HOME/soa_3.0.1
   $ mkdir ${WORKDIR}
   ```

1. Download the supported version of the WebLogic Kubernetes operator source code from the operator github project.
   ```
   $ cd ${WORKDIR}
   $ git clone https://github.com/oracle/weblogic-kubernetes-operator.git --branch release/3.0.1
   ```

#### 3.2 Set Up the Oracle SOA Suite Setup Scripts

1. Download the Oracle SOA Suite Kubernetes deployment scripts from the Oracle SOA Suire repository and copy them to the WebLogic Kubernetes operator samples location
   ```
   $ cd ${WORKDIR}
   $ git clone https://github.com/oracle/fmw-kubernetes.git
   ```

1. Update the Oracle WebLogic Kubernetes operator samples with Oracle SOA Suite Scripts
   ```
   mv -f ${WORKDIR}/weblogic-kubernetes-operator/kubernetes/samples/scripts/create-soa-domain  ${WORKDIR}/weblogic-kubernetes-operator/kubernetes/samples/scripts/create-soa-domain_backup
   cp -rf ${WORKDIR}/fmw-kubernetes/OracleSOASuite/kubernetes/3.0.1/create-soa-domain  ${WORKDIR}/weblogic-kubernetes-operator/kubernetes/samples/scripts/

   mv -f ${WORKDIR}/weblogic-kubernetes-operator/kubernetes/samples/scripts/common  ${WORKDIR}/weblogic-kubernetes-operator/kubernetes/samples/scripts/common_backup
   cp -rf ${WORKDIR}/fmw-kubernetes/OracleSOASuite/kubernetes/3.0.1/common  ${WORKDIR}/weblogic-kubernetes-operator/kubernetes/samples/scripts/

   cp -rf ${WORKDIR}/fmw-kubernetes/OracleSOASuite/kubernetes/3.0.1/create-database  ${WORKDIR}/weblogic-kubernetes-operator/kubernetes/samples/scripts/

   mv -f ${WORKDIR}/weblogic-kubernetes-operator/kubernetes/samples/scripts/create-rcu-schema  ${WORKDIR}/weblogic-kubernetes-operator/kubernetes/samples/scripts/create-rcu-schema_backup
   cp -rf ${WORKDIR}/fmw-kubernetes/OracleSOASuite/kubernetes/3.0.1/create-rcu-schema  ${WORKDIR}/weblogic-kubernetes-operator/kubernetes/samples/scripts/

   mv -f ${WORKDIR}/weblogic-kubernetes-operator/kubernetes/samples/charts/ingress-per-domain  ${WORKDIR}/weblogic-kubernetes-operator/kubernetes/samples/charts/ingress-per-domain_backup
   cp -rf ${WORKDIR}/fmw-kubernetes/OracleSOASuite/kubernetes/3.0.1/ingress-per-domain  ${WORKDIR}/weblogic-kubernetes-operator/kubernetes/samples/charts/

   cp -rf ${WORKDIR}/fmw-kubernetes/OracleSOASuite/kubernetes/3.0.1/imagetool-scripts  ${WORKDIR}/weblogic-kubernetes-operator/kubernetes/samples/scripts/

   cp -rf ${WORKDIR}/fmw-kubernetes/OracleSOASuite/kubernetes/3.0.1/charts  ${WORKDIR}/weblogic-kubernetes-operator/kubernetes/samples/scripts/
   ```
#### 3.3 Set Up Helm

By default OCI Cloud Shell comes with preinstalled Helm. The current available Helm version of v3.0.1 on OCI Cloud Shell is not supported with Oracle WebLogic Kubernetes Operator 3.0.1. Hence perform below steps to install Helm locally at ${WORKDIR}/weblogic-kubernetes-operator.

1. Install Helm v3.1.3+:

   a. Download latest Helm from https://github.com/helm/helm/releases. Example to download Helm v3.3.4:
      ```
      $ cd ${WORKDIR}
      $ wget https://get.helm.sh/helm-v3.3.4-linux-amd64.tar.gz
      ```
   b. Unpack tar.gz:
      ```
      $ cd ${WORKDIR}
      $ tar -zxvf helm-v3.3.4-linux-amd64.tar.gz
      ```
   c. Find the Helm binary in the unpacked directory, and move it to its desired destination:
      ```
      $ cd ${WORKDIR}
      $ mv linux-amd64/helm  weblogic-kubernetes-operator
      ```
2. Run helm version to verify its installation:
   ```
   $ cd ${WORKDIR}/weblogic-kubernetes-operator
   $ ./helm version

### 4. Prepare the worker node for Oracle SOA Suite domain

#### 4.1 Login to worker node

1. Get the public IP address of the worker node 

   a. Open the navigation menu and under **Developer Services**, select **Kubernetes Clusters**.
   
   b. Select your cluster (for example `soaoke`) and then go to **Node Pools**. 
   
   c. Click the link from **Total Worker Nodes** which will take you to the worker node page. 
   
   d. Get the public IP of worker node from "Public IP: X.X.X.X" on worker node page.

1. At the OCI Cloud Shell command prompt, login to worker node ( Note: This will use the private SSH key `id_rsa` available at $HOME/.ssh. If you have placed the private SSH key in different location then you can login with `ssh -i /path/id_rsa opc@X.X.X.X ).
   ```
   $ ssh opc@X.X.X.X
   ```

#### 4.2 Extending the boot volume size
While creating Kubernetes Cluster, we have provided custom boot volume size of 250G. This is not available by default. You need to extend the volume to take advantage of the larger size. 

1. Login to worker node: 
   ```
   $ ssh opc@X.X.X.X
   ```

1. Execute the below commands to resize the filesystem and partition. **Note:** At the end, a reboot is triggered for changes to be effective.
   ```
   $ sudo sgdisk -d 3 /dev/sda
   $ sudo sgdisk -N 3 /dev/sda
   $ sudo partprobe /dev/sda
   $ sudo xfs_growfs -d /
   $ sudo reboot
   ```
   
1. From OCI Cloud Shell, wait for the worker node to get rebooted. Use the below command to watch for state of worker node to be in "Ready":
   ```
   $ kubectl get nodes -w
   ```

1. Once the worker node is accessible after reboot, login and confirm that "/" is showing as 250G:
   ```
    $ df -h /
   ```

For more details, see [Extending the Partition for a Boot Volume](https://docs.cloud.oracle.com/en-us/iaas/Content/Block/Tasks/extendingbootpartition.htm#Extending_the_Partition_for_a_Boot_Volume). 

#### 4.3 Setup Docker images and host path for domain home
In this step we will get the required Docker images to your worker node. Also we will setup host path for Oracle SOA Suite domain home which will be used later during Persistence Volume creation.

1. Login to worker node:
   ```
   $ ssh opc@X.X.X.X
   ```

1. Pull the operator image:

   ```
   $ docker pull oracle/weblogic-kubernetes-operator:3.0.1
   ```

1. Obtain the Oracle Database image and Oracle SOA Suite Docker image from the [Oracle Container Registry](https://container-registry.oracle.com):

   a. For first time users, to pull an image from the Oracle Container Registry, navigate to https://container-registry.oracle.com and log in using the Oracle Single Sign-On (SSO) authentication service. If you do not already have SSO credentials, click the Sign In link at the top of the page to create them.

     Use the web interface to accept the Oracle Standard Terms and Restrictions for the Oracle software images that you intend to deploy. Your acceptance of these terms are stored in a database that links the software images to your Oracle Single Sign-On login credentials.

     To obtain the image, log in to the Oracle Container Registry:
     ```
     $ docker login container-registry.oracle.com
     ```

   b. Find and then pull the Oracle Database image for 12.2.0.1:
     ```
     $ docker pull container-registry.oracle.com/database/enterprise:12.2.0.1-slim
     ```

   c. Find and then pull the prebuilt Oracle SOA Suite image 12.2.1.4 install image:

     ```
     $ docker pull container-registry.oracle.com/middleware/soasuite:12.2.1.4
     ```
     > Note: This image does not contain any Oracle SOA Suite product patches and can only be used for test and development purposes.

1. Create the directory in worker node that will be used for the Oracle SOA Suite domain home:
   ```
   $ sudo mkdir -p /scratch/k8s_dir
   $ sudo chown -R 1000:1000 /scratch/k8s_dir
   ```
### 5. Install WebLogic Kubernetes Operator

We will execute all kubectl commands from OCI Cloud Shell. In case you are logged into the worker node, make sure you exit before proceeding.

#### 5.1 Prepare for WebLogic Kubernetes Operator
1. Create a namespace `opns` for the operator
   ```
   $ kubectl create namespace opns
   ```

1. Create a service account for the operator in the operator’s namespace:
   ```
   $ kubectl create serviceaccount -n opns op-sa
   ```

#### 5.2 Install WebLogic Kubernetes Operator

1. Use helm to install and start the operator from the directory you just cloned:
   ```
   $ cd ${WORKDIR}/weblogic-kubernetes-operator
   $ ./helm install weblogic-kubernetes-operator kubernetes/charts/weblogic-operator \
   --namespace opns \
   --set image=oracle/weblogic-kubernetes-operator:3.0.1 \
   --set serviceAccount=op-sa \
   --set "domainNamespaces={}" \
   --wait
   ```
#### 5.3 Verify WebLogic Kubernetes Operator

1. Verify that the operator’s pod is running, by listing the pods in the operator’s namespace. You should see one for the operator.
  ```
  $ kubectl get pods -n opns
  ```

1. Verify that the operator is up and running by viewing the operator pod's logs:
  ```
  $ kubectl logs -n opns -c weblogic-operator deployments/weblogic-operator
  ```


The WebLogic Kubernetes Operator v3.0.0 has been installed.  Continue with Loadbalancer and SOA Domain setup.

### 6. Install the Traefik (ingress-based) Load Balancer

The Oracle WebLogic Server Kubernetes Operator supports three load balancers: Traefik, Voyager, and Apache. Samples are provided in the documentation.

This quickstart demonstrates how to install the Traefik Ingress controller to provide load balancing for Oracle SOA Suite Domain.

1. Create a namespace for Traefik
    ```
    $ kubectl create namespace traefik
    ```

1. Set up Helm for 3rd party services
   ```
   cd ${WORKDIR}/weblogic-kubernetes-operator
   $ ./helm repo add traefik https://containous.github.io/traefik-helm-chart
   ```

1. Install the Traefik operator in the traefik namespace with the provided sample values:
   ```
   $ cd ${WORKDIR}/weblogic-kubernetes-operator
   $ ./helm install traefik traefik/traefik \
    --namespace traefik \
    --values kubernetes/samples/charts/traefik/values.yaml \
    --set "kubernetes.namespaces={traefik}" \
    --set "service.type=LoadBalancer" \
    --wait
   ```

### 7. Create and Configure Oracle SOA Suite Domain
#### 7.1 Prepare for an Oracle SOA Suite Domain

1. Create a namespace that can host Oracle SOA Suite domains:
   ```
   $ kubectl create namespace soans
   ```

1. Use helm to configure the operator to manage SOA domains in this namespace:
   ```
   $ cd ${WORKDIR}/weblogic-kubernetes-operator
   $ ./helm upgrade weblogic-kubernetes-operator kubernetes/charts/weblogic-operator \
      --reuse-values \
      --namespace opns \
      --set "domainNamespaces={soans}" \
      --wait
   ```
1. Create Kubernetes Secrets

   a. For Domain ( using create-weblogic-credentials.sh)  : Create a Kubernetes secret for the Domain (username : weblogic and password: Welcome1) in the same Kubernetes namespace as the domain (soans):

      ```
      $ cd ${WORKDIR}/weblogic-kubernetes-operator/kubernetes/samples/scripts/create-weblogic-domain-credentials
      $ ./create-weblogic-credentials.sh -u weblogic -p Welcome1 -n soans -d soainfra -s soainfra-domain-credentials
      ```
   b. For RCU ( using create-rcu-credentials.sh) : Create a Kubernetes secret for the RCU in the same Kubernetes namespace as the domain (soans) with below details:

      >  * Schema user          : SOA1
      >  * Schema password      : Oradoc_db1                   
      >  * DB sys user password : Oradoc_db1
      >  * Domain name          : soainfra
      >  * Domain Namespace     : soans
      >  * Secret name          : soainfra-rcu-credentials

      ```
      $ cd ${WORKDIR}/weblogic-kubernetes-operator/kubernetes/samples/scripts/create-rcu-credentials
      $ ./create-rcu-credentials.sh -u SOA1 -p Oradoc_db1 -a sys -q Oradoc_db1 -d soainfra -n soans -s soainfra-rcu-credentials
      ```

1. Create Kubernetes Persistence Volume and PersistenceVolume Claim

    a. Update the `create-pv-pvc-inputs.yaml` with below values:

     >  * baseName: domain
     >  * domainUID: soainfra
     >  * namespace: soans
     >  * weblogicDomainStoragePath: /scratch/k8s_dir
     
     You can manually edit `create-pv-pvc-inputs.yaml` with above values or use the below sed commands to quickly update:

      ```
      $ cd ${WORKDIR}/weblogic-kubernetes-operator/kubernetes/samples/scripts/create-weblogic-domain-pv-pvc
      $ cp create-pv-pvc-inputs.yaml create-pv-pvc-inputs.yaml.orig
      $ sed -i -e "s:baseName\: weblogic-sample:baseName\: domain:g" create-pv-pvc-inputs.yaml
      $ sed -i -e "s:domainUID\::domainUID\: soainfra:g" create-pv-pvc-inputs.yaml
      $ sed -i -e "s:namespace\: default:namespace\: soans:g" create-pv-pvc-inputs.yaml
      $ sed -i -e "s:#weblogicDomainStoragePath\: /scratch/k8s_dir:weblogicDomainStoragePath\: /scratch/k8s_dir:g" create-pv-pvc-inputs.yaml
      ```

    c. Execute `create-pv-pvc.sh` script to create the PV and PVC configuration files.
      ```
      $ ./create-pv-pvc.sh -i create-pv-pvc-inputs.yaml -o output
      ```

    d. Create the PV and PVC using the configuration files created in previous step as shown below:
      ```
      $ kubectl create -f  output/pv-pvcs/soainfra-domain-pv.yaml
      $ kubectl create -f  output/pv-pvcs/soainfra-domain-pvc.yaml
      ```

1. Install and Configure the Database for Oracle SOA Suite Domain
   >  NOTE: This step is required only when standalone database was not already setup and the user wanted to use the database in a container.

   The Oracle Database Docker images are supported only for non-production use. For more details, see My Oracle Support note: Oracle Support for Database Running on Docker (Doc ID 2216342.1). For production use-case it is suggested to use a standalone db.
Sample provides steps to create the database in a container.

   a. Create a database in a container

      ```
      $ cd ${WORKDIR}/weblogic-kubernetes-operator/kubernetes/samples/scripts/create-oracle-db-service
      $ ./start-db-service.sh -i  container-registry.oracle.com/database/enterprise:12.2.0.1-slim -p none
      ```

      Once database is created successfully, you can use the database connection string, "oracle-db.default.svc.cluster.local:1521/devpdb.k8s", as an rcuDatabaseURL parameter in the create-domain-inputs.yaml file.

   b. Create SOA Schemas for say domain type osb

      To install SOA schemas, run the "create-rcu-schema.sh" script with below inputs:

     >  -s <RCU PREFIX>   Here: SOA1
     >  -t <SOA domain type>  Here: osb
     >  -i <SOASuite image>   Here: container-registry.oracle.com/middleware/soasuite:12.2.1.4

      ```
      $ cd ${WORKDIR}/weblogic-kubernetes-operator/kubernetes/samples/scripts/create-rcu-schema
      $ ./create-rcu-schema.sh \
        -s SOA1 \
        -t osb \
        -i container-registry.oracle.com/middleware/soasuite:12.2.1.4 \
        -q Oradoc_db1 \
        -r Oradoc_db1
      ```

Now the environment is ready to start the Oracle SOA Suite Domain creation with domain type as OSB.


#### 7.2 Create Oracle SOA Suite Domain

1. The sample scripts for Oracle SOA Suite domain deployment are available at `<weblogic-kubernetes-operator-project>/kubernetes/samples/scripts/create-soa-domain`. You must edit `create-domain-inputs.yaml` (or a copy of it) to provide the details for your domain.

    Update the `create-domain-inputs.yaml` with below values and rest default values will be used for domain creation:

    *  domainType: osb
    *  initialManagedServerReplicas: 1
    *  image: container-registry.oracle.com/middleware/soasuite:12.2.1.4
    *  clusterName: osb_cluster
    *  managedServerNameBase: osb_server
    *  managedServerPort: 9001
    
    You can manually edit `create-domain-inputs.yaml` with above values or use the below sed commands to quickly update:

    ```
    $ cd ${WORKDIR}/weblogic-kubernetes-operator/kubernetes/samples/scripts/create-soa-domain/domain-home-on-pv/
    $ cp create-domain-inputs.yaml create-domain-inputs.yaml.orig

    $ sed -i -e "s:domainType\: soa:domainType\: osb:g" create-domain-inputs.yaml
    $ sed -i -e "s:initialManagedServerReplicas\: 2:initialManagedServerReplicas\: 1:g" create-domain-inputs.yaml
    $ sed -i -e "s:image\: soasuite\:12.2.1.4:image\: container-registry.oracle.com/middleware/soasuite\:12.2.1.4:g" create-domain-inputs.yaml
    $ sed -i -e "s:clusterName\: soa_cluster:clusterName\: osb_cluster:g" create-domain-inputs.yaml
    $ sed -i -e "s:managedServerNameBase\: soa_server:managedServerNameBase\: osb_server:g" create-domain-inputs.yaml
    $ sed -i -e "s:managedServerPort\: 8001:managedServerPort\: 9001:g" create-domain-inputs.yaml
    ```

1. Run the create-domain.sh script to create a domain
    ```
    $ cd ${WORKDIR}/weblogic-kubernetes-operator/kubernetes/samples/scripts/create-soa-domain/domain-home-on-pv/
    $ ./create-domain.sh -i create-domain-inputs.yaml -o output
    ```

1. Create Kubernetes Domain object

    Once the create-domain.sh is success, it generates the "output/weblogic-domains/soainfra/domain.yaml" using which you can create the Kubernetes resource domain which starts the domain and servers as shown below:

    ```
    $ cd ${WORKDIR}/weblogic-kubernetes-operator/kubernetes/samples/scripts/create-soa-domain/domain-home-on-pv
    $ kubectl create -f output/weblogic-domains/soainfra/domain.yaml
    ```

1. Verify that Kubernetes Domain Object (name: soainfra)  is created:
    ```
    $ kubectl get domain -n soans
    NAME       AGE
    soainfra   3m18s
    ```

1. Once you create the domain, "introspect pod" will get created. This inspects the Domain Home and then start the "soainfra-adminserver" pod.  Once the "soainfra-adminserver" pod comes up successfully, then the managed server pod will be started.
Watch the "soans" namespace for the status of domain creation with below command
    ```
    $ kubectl get pods -n soans -w
    ```

1. Verify that SOA Domain server pods and services are created and in READY state:
    ```
    $ kubectl get all -n soans
    ```

#### 7.3 Configure Traefik to access in SOA Domain Services

1. Configure Traefik to manage Ingresses created in SOA Domain namespace (soans):
    ```
    $ cd ${WORKDIR}/weblogic-kubernetes-operator
    $ ./helm upgrade traefik traefik/traefik \
      --reuse-values \
      --namespace traefik \
      --set "kubernetes.namespaces={traefik,soans}" \
      --wait
    ```

1. Create an ingress for the domain in the domain namespace by using the sample Helm chart. Here we are passing traefik.hostname="", so that we can access all application URLs using the LoadBalancer (EXTERNAL-IP) of the traefik service.
    ```
    $ cd ${WORKDIR}/weblogic-kubernetes-operator
    $ ./helm install soa-traefik-ingress  kubernetes/samples/charts/ingress-per-domain \
      --namespace soans \
      --values kubernetes/samples/charts/ingress-per-domain/values.yaml \
      --set "traefik.hostname=""" \
      --set domainType=osb
    ```
1. Verify the created ingress per domain details with
    ```
    $ kubectl describe ingress soainfra-traefik -n soans
    ```
#### 7.4 Verify that you can access the Oracle SOA Suite domain URL

1. Get the EXTERNAL-IP of the traefik service. This is the public IP address of the load balancer that you will use to access the WebLogic Server Administration Console and domain URLs. 
   To print only the public IP address, execute this command:
   ```
   $ TRAEFIK_PUBLIC_IP=`kubectl describe svc traefik --namespace traefik | grep Ingress | awk '{print $3}'`
   $ echo $TRAEFIK_PUBLIC_IP
   ```
   
2. Validate the Domain of type osb with below credentials:
   *username*: weblogic
   *password*: Welcome1
   
   ```
   http://${TRAEFIK_PUBLIC_IP}/weblogic/ready
   http://${TRAEFIK_PUBLIC_IP}/console
   http://${TRAEFIK_PUBLIC_IP}/em
 
   OSB:
   http://${TRAEFIK_PUBLIC_IP}/servicebus
   ```
   
### 8. Scaling an Oracle Service Bus Cluster

The operator provides several ways to initiate scaling of WebLogic clusters, including:

    * On-demand, updating the domain resource directly (using `kubectl`).
    * Calling the operator's REST scale API, for example, from curl.
    * Using a WLDF policy rule and script action to call the operator's REST scale API.
    * Using a Prometheus alert action to call the operator's REST scale API.

Here we will discuss Scaling a Oracle Service Bus cluster using kubectl.
The easiest way to scale a OSB Cluster in Kubernetes is to simply edit the replicas property within a domain resource. 
To retain changes, edit the domain.yaml file and apply the changes using kubectl. Use your favorite editor to open the domain.yaml file.

Find the following part:
```
clusters:
- clusterName: osb_cluster
  serverStartState: "RUNNING"
  replicas: 1
```  

Modify `replicas` to 2 for osb_cluster and save the changes. You can use the `vi` editor for example:

Apply the changes using kubectl:
```
$ cd ${WORKDIR}/weblogic-kubernetes-operator/kubernetes/samples/scripts/create-soa-domain/domain-home-on-pv
$ vi output/weblogic-domains/soainfra/domain.yaml  #update replicas to 2 for osb_cluster
$ kubectl apply -f output/weblogic-domains/soainfra/domain.yaml
```


Check the changes in the number of pods using kubectl:
```
$ kubectl get pods -n soans
NAME                             READY     STATUS        RESTARTS   AGE
soainfra-adminserver                         1/1     Running     0          7d2h
soainfra-create-soa-infra-domain-job-9n89n   0/1     Completed   0          10d
soainfra-osb-server1                         1/1     Running     0          7d2h
```

Soon,  soainfra-osb-server2  will appear and will be ready within a few minutes.
```
$ kubectl get pods -n soans
NAME                             READY     STATUS        RESTARTS   AGE
soainfra-adminserver                         1/1     Running     0          7d2h
soainfra-create-soa-infra-domain-job-9n89n   0/1     Completed   0          10d
soainfra-osb-server1                         1/1     Running     0          7d2h
soainfra-osb-server2                         1/1     Running     0          3m51s
```

> **Note!** You can edit the existing (running) domain resource file directly by using the `kubectl edit` command. In this case, your `domain.yaml` file, available on your desktop, will not reflect the changes of the running domain resource.

```
$ kubectl edit domain soainfra -n soans
```

> **Note!** Do not use the Console to scale the cluster. The operator controls this operation. Use the operator options to scale your cluster deployed on Kubernetes.
