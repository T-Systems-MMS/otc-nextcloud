# Nextcloud Deployment on Open Telekom Cloud (OTC)

## Introduction

The following tutorial explains how to host the file sharing and collaboration platform [Nextcloud](https://nextcloud.com/) in the [Open Telekom Cloud](https://open-telekom-cloud.com/de) with the Cloud Container Engine. The advantage over a VM is less administration effort and faster setup of the platform with less technical expertise.

Care has been taken to ensure that the process is easy to understand. If not described otherwise, the default settings can be used. tbd.

## Collaborators

This tutorial was created at [T-Systems-Multimedia Solutions GmbH](https://www.t-systems-mms.com/) by @masp0, @mrwhiteHD, @nemental and @mschwrdtnr.

<!-- todo: write names with links -->

## Tutorial

### 1. Create Virtual Private Cloud (VPC)

At first we will create a [Virtual Private Cloud (VPC)](https://docs.otc.t-systems.com/vpc/index.html).

1. Navigate to VPC and click "Create VPC Cluster"
2. Choose a VPC Name and a Subnet Name
3. Click "Create Now"
4. Go "Back to VPC List"
5. Choose your created VPC and activate SNAT <!-- Why ?-->

### 2. Create Cloud Container Engine (CCE)

Now we will create a [Cloud Container Engine (CCE)](https://docs.otc.t-systems.com/cce/index.html).

1. Navigate to CCE and click "Create VM Cluster"
2. Choose a Cluster Name
3. High Availability: "No" <!-- Why? -->
4. Choose your created VPC and Subnet
5. Let the "Container Network Segement" automatically select <!-- Why? -->
6. You have to check "I am aware of the above limitations and read the CCE Role Management Instructions" and can go to the next site
7. Choose a Node Name and the Specifications you need. The smallest flavor could be enough
8. Be sure that your created subnet is used
9. Choose a Key Pair. If you have none, create a key pair.
10. Go to next the page, skip one and check your Details on the last page. If everything is okay you can click "Submit". This will create your Cluster.
    <details>
    <summary>
    <b>Click here to view our CCE Details</b>
    </summary>
    <img src=/docs/assets/cce_details.png width=100%>
    </details>
11. Submit and wait a little bit. The Cluster will be created in 5 to 10 minutes. Meanwhile we can create the other related services.

### 3. Create Relational Database Service (RDS)

Now we will create the Database for our Nextcloud. We use the [Relational Database Service (RDS)](https://docs.otc.t-systems.com/rds/index.html).

1. Navigate to RDS and create a DB Instance
2. Choose a DB Instance Name
3. Choose the "DB Instance Type" Single for development or testing scenerios. This is cheaper.
4. You can use the smallest Memory and the cheaper Storage Type "Common I/O". According to Nextcloud 20GB per User is enough for the beginning.
5. Be sure that your created VPC will choosen
6. Choose an Administrator Password and click "Create Now"
7. Check the Details and Submit if everything is fine
   <details>
   <summary>
   <b>Click here to view our RDS Details</b>
   </summary>
   <img src=/docs/assets/rds_details.png width=100%>
   </details>

<!-- Hinzufügen der Security Group
Source stand nur defualt drin, aber brauchen 0.0.0.0
wo soll der Pubkt hin?
-->

### 5. Create a persistent Volume

If the cluster is created you can create the persistent Volume.

1. Go back to CCE, go to "Resource Management" on the left navbar and go to Storage
2. Choose [Scalable File Service](https://docs.otc.t-systems.com/sfs/index.html) and create a SFS Disk
3. Choose a Name (e.g. nextcloud-data), check the Details and click "Submit"
4. Create a second SFS Disk with the same settings and the name "nextcloud-config"
   <!-- Warum werden zwei benötigt? -->

### 6. Create the Deployment

The last step is creating the Deployment.

1. Switch to Workloads on the left navbar and click "Create Deployment"
2. **Only choose one instance.** Nextcloud is not scalable at the time of this guide.
3. Click "Next" and "Add Container"
4. Select a Third-party Image with the adress "Nextcloud". The latest nextcloud image will be chosen from [dockerhub](https://hub.docker.com/_/nextcloud/).
5. Click "OK" and scroll down to "Environment Variables" and add one with Type _Added manually_, Variable Name _NEXTCLOUD_TRUSTED_DOMAINS_ and your in Point X created adress. <!-- Warum ist dieser Punkt nötig? Wir erstellen die Adresse erst später. Wenn wir über die IP anstatt den DNS Namen zugreifen möchten erhalten wir die Meldung, dass Zugriff nur über vertrauenswürdige Domains möglich ist-->
6. Go to Data Storage, select Cloud Storage and add your created SFS Storages. Set the Container Path for your nextcloud-config storage to _/var/www/html/config_ and for your nextcloud-data storage to _/var/www/html/data_.
7. Go to the next page and add a Service with the Access Type _Node access_.
8. Set the Container Port to 80 and the Access Port as _Specified Port_ to 30080. Click "OK".
9. Skip the advanced settings and "Create" the Deployment

### 6. Create Elastic Load Balancer (ELB)

We will also need an [Elastic Load Balancer (ELB)](https://docs.otc.t-systems.com/elb/index.html).

1. Navigate to ELB and "Create Enhanced Load Balancer"
2. Choose your created VPC and the Bandwidth you want and a name.
3. Check the Details and Submit if everything is fine
   <details>
   <summary>
   <b>Click here to view our ELB Details</b>
   </summary>
   <img src=/docs/assets/ELB_Details.png width=100%>
   </details>

#### 6.1 Create a Certificate

<!-- Sollte umgeschrieben werden -->

To enter your page via https you need to have a Certificate. If you already have your own, go to Certificates and upload it with "Create Certificate". If you do not have one, view our short guide [How to create a self signed Certificate with openSSL](docs/CREATE_CERTIFICATE.md).

#### 6.2 Add ELB Listeners

Now we will create listeners for the https redirections.

1. Go to your created ELB "Listeners"
2. Add a https listener with the name "listener-https" and choose your server certificate.
3. Create a Server Group and deactivate "Health Check"
4. Now go to "Backend Server Groups" and add your nextcloud-cluster-node with the Backend Port _30080_ to your ELB.
5. Now add a http listener with the name "listener-http" and redirect it to your created https-listener.

Note: In this step an [Elastic IP](https://docs.otc.t-systems.com/eip/index.html) will be automatically created for you.

#### 6.3 Create a Domain Name Service (DNS)

<!-- more information?-->

If you want you can also create a DNS for your adress to open your page with a specific name. Otherwise you can use the EIP of your node.

1. Search for [DNS](https://docs.otc.t-systems.com/dns/index.html), click on "Public Zones", the Top-Level-Domain of OTC _otc-on-rails.de_ and add a Record Set.
2. Choose a name for your server
3. Add your EIP of your created ELB to "Value".

### 7. Nextcloud First Configurations

If the deployment creation was successfull you can open nextcloud on your created DNS-Name.
At the first start you have to create an admin account. Then you have to set up the database.

1. Choose _MySQL/MariaDB_ cause we are using the Relational Database Service
   1a. Username: root
   1b. Password: _your created password for the RDS_
   1c. Name: _your RDS Name_
   1d. Adress: _the floating IP Adress of your RDS with the port number_
   (You can find the information at your RDS/Connection Information)
   <details>
   <summary>
   <b>Click here to view our Nextcloud Configuration Settings</b>
   </summary>
   <img src=/docs/assets/nextcloud_details.png width=50%>
   </details>
2. Now you can click on "Complete Installation" and your Nextcloud will configured.

## Troubleshooting

## Other useful links
