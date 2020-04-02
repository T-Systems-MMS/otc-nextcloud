# Nextcloud Deployment on Open Telekom Cloud (OTC)

## Introduction

The following tutorial explains how to host the file sharing and collaboration platform [Nextcloud](https://nextcloud.com/) in the [Open Telekom Cloud](https://open-telekom-cloud.com/de) with the Cloud Container Engine. The advantage over a VM is less administration effort and faster setup of the platform with less needed technical expertise.

With the CCE we deploy a Nextcloud Image in a Managed Kubernetes Cluster. As persistent Storage we will use two Scalable File Services, one for config and one for the data. This tutorial will show you how to use a Enhanced Load Balancer to access your Nextcloud via your own created Domain-Name with a secured https connection.

The claim of this tutorial is to give a short and understandable guideline to get in touch with deploying images with the OTC Cloud Container Engine and also learn something about other services of the Open Telekom Cloud.

Notice that the Open Telekom Cloud and this guide can change over time, so it is possible that some steps are not right. If you have problems, questions or suggestions for improvements, feel free to open an Issue or a Pull Request. For other related stuff you can contact max.schwerdtner@t-systems.com.

If you are interested in other guides related to the Open Telekom Cloud we can recommend the [OTC Hands-On Training by Ulrich Schneider](https://community.open-telekom-cloud.com/community?id=community_blog&sys_id=a41f28cb13d78450d15ac969a674415a&view_source=featuredList).

## Collaborators

This tutorial was created at [T-Systems-Multimedia Solutions GmbH](https://www.t-systems-mms.com/) by [masp0](https://github.com/masp0), [MrWhiteHD](https://github.com/mrwhiteHD), [Nemental](https://github.com/Nemental) and [mschwrdtnr](https://github.com/mschwrdtnr).

## Tutorial

### 1. Create Virtual Private Cloud (VPC)

At first we will create a [Virtual Private Cloud (VPC)](https://docs.otc.t-systems.com/vpc/index.html). This is a logical isolated, configurable and manageable virtual network for [Elastic Cloud Server](https://docs.otc.t-systems.com/ecs/index.html).

1. Navigate to VPC and click "Create VPC Cluster"
2. Choose a VPC Name and a Subnet Name
3. Click "Create Now"
4. Go "Back to VPC List"
5. Choose your created VPC and activate SNAT
   > SNAT enables the simultaneous use of a public address. Otherwise we can not access over the internet.

### 2. Create Cloud Container Engine (CCE)

Now we will create a [Cloud Container Engine (CCE)](https://docs.otc.t-systems.com/cce/index.html). This is a scalabe, high-performance container service which is build on Docker technology and can scale your applications within seconds. We use the CCE to deploy Nextcloud without the need to create seperate VMs etc.

1. Navigate to CCE and click "Create VM Cluster"
2. Choose a Cluster Name
3. Set High Availability to "No"
   > High Availability is expensive and you do not need it for Nextcloud at the beginning.
4. Choose your created VPC and Subnet
5. Let the "Container Network Segment" automatically select
6. You have to check "I am aware of the above limitations and read the CCE Role Management Instructions" and can go to the next site
7. Choose a Node Name and the Specifications you need. The smallest flavor is enough.
8. Be sure that your created subnet is used
9. Choose a Key Pair. If you have none, create a key pair.
10. Go to the next page, skip it and check your Details on the last page. If everything is okay you can click "Submit". This will create your Cluster.
    <details>
    <summary>
    <b>Click here to view our CCE Details</b>
    </summary>
    <img src=/docs/assets/cce_details.png width=80%>
    </details>
11. Submit and wait a little bit. The Cluster will be created in 5 to 10 minutes. Meanwhile we can create the other related services.

### 3. Create Relational Database Service (RDS)

Now we will create the Database for our Nextcloud. We use the [Relational Database Service (RDS)](https://docs.otc.t-systems.com/rds/index.html) as a persistent and performant SQL Database.

1. Navigate to RDS and create a DB Instance
2. Choose a DB Instance Name
3. Choose the "DB Instance Type" _Single_ for development or testing scenerios. This is cheaper.
4. You can use the cheaper Storage Type "Common I/O" and the smallest possible Memory. According to Nextcloud 20GB per User is enough for the beginning.
5. Be sure that your created VPC will choosed
6. Choose an Administrator Password and click "Create Now"
7. Check the Details and Submit if everything is fine
   <details>
   <summary>
   <b>Click here to view our RDS Details</b>
   </summary>
   <img src=/docs/assets/rds_details.png width=80%>
   </details>

#### 3.1 Change the Security Group

To make the connection between the Database and your node possible, you need to follow this steps:

1. Go to your created RDS
2. Click on your choosed Security Group
3. Click on Inbound Rules and change the Protocol "All" to the source _0.0.0.0_
   > You can also link to your created node, but this was not tested by us.

### 5. Create SFS's as persistent shareable Volumes

If the cluster is created you can create the persistent Volumes in your CCE. This is necessary for sharing data between the servers. Notice that we will create two different Scalable File Services. One for your config (this can be a small storage) and one for the data. The RDS acts as storage for user-data, meta-data etc.

1. Go back to CCE, click on "Resource Management" on the left navbar and go to Storage
2. Choose [Scalable File Service](https://docs.otc.t-systems.com/sfs/index.html) and create a SFS Disk
3. Choose the name _nextcloud-data_, check the Details and click "Submit"
4. Create a second SFS Disk with the same settings and the name _nextcloud-config_

### 6. Create the Deployment

The next step is creating the Deployment.

1. Switch to Workloads on the left navbar and click "Create Deployment"
2. **Only choose one instance.** Nextcloud is not scalable at the time of this guide.
3. Click "Next" and "Add Container"
4. Select a Third-party Image with the adress "Nextcloud". The latest nextcloud image will be chosen from [dockerhub](https://hub.docker.com/_/nextcloud/).
5. Click "OK" and scroll down to "Environment Variables" and add one with Type _Added manually_, Variable Name _NEXTCLOUD_TRUSTED_DOMAINS_ and your later created domain adress. This is necessary for connecting with https.
   > We will create a DNS in Point 6.3. Later you can also add the EIP of your ELB.
6. Go to Data Storage, select Cloud Storage and add your created SFS Storages. Set the Container Path for your nextcloud-config storage to _/var/www/html/config_ and for your nextcloud-data storage to _/var/www/html/data_.
7. Go to the next page and add a Service with the Access Type _Node access_.
8. Set the Container Port to 80 and the Access Port as _Specified Port_ to 30080. Click "OK".
9. Skip the advanced settings and "Create" the Deployment

### 6. Create Elastic Load Balancer (ELB)

We will also need an [Elastic Load Balancer (ELB)](https://docs.otc.t-systems.com/elb/index.html) to make the node available for the internet.

1. Navigate to ELB and "Create Enhanced Load Balancer"
2. Choose your created VPC and the Bandwidth you want and a name.
3. Check the Details and Submit if everything is fine
   <details>
   <summary>
   <b>Click here to view our ELB Details</b>
   </summary>
   <img src=/docs/assets/elb_details.png width=100%>
   </details>

> Note: In this step an [Elastic IP](https://docs.otc.t-systems.com/eip/index.html) will be automatically created for you.

#### 6.1 Create a Certificate

To enter your page via https you need to have a Certificate. If you already have your own, go to Certificates and upload it with "Create Certificate". If you do not have one, view our short guide [How to create a self signed Certificate with openSSL](docs/CREATE_CERTIFICATE.md).

#### 6.2 Add ELB Listeners

Now we will create listeners for the https redirections.

1. Go to your created ELB "Listeners"
2. Add a https listener with the name "listener-https" and choose your server certificate.
3. Create a Server Group and deactivate "Health Check"
4. Now go to "Backend Server Groups" and add your nextcloud-cluster-node with the Backend Port _30080_ to your ELB.
5. Now add a http listener with the name "listener-http" and redirect it to your created https-listener.

#### 6.3 Create a Domain Name Service (DNS)

If you want you, can also create a DNS for your adress to access your page with a specific name. Otherwise you can use the EIP of your node.

1. Search for [DNS](https://docs.otc.t-systems.com/dns/index.html), click on "Public Zones", the Top-Level-Domain of OTC _otc-on-rails.de_ and add a Record Set.
2. Choose a name for your server
3. Add the EIP of your created ELB to "Value".

### 7. Nextcloud First Configurations

If the deployment creation was successfull you can open Nextcloud with your created DNS-Name.
At the first start you have to create an admin account. Then you have to set up the database.

1. Choose _MySQL/MariaDB_ cause we are using the Relational Database Service
   - Username: root
   - Password: _your created password for the RDS_
   - Name: _your RDS Name_
   - Adress: _the floating IP Adress of your RDS with the port number_
     > You can find the information at your RDS/Connection Information)
     <details>
     <summary>
     <b>Click here to view our Nextcloud Configuration Settings</b>
     </summary>
     <img src=/docs/assets/nextcloud_details.png width=35%>
     </details>
2. Now you can click on "Complete Installation" and your Nextcloud will configured.

To learn more about how to use Nextcloud you can visit the [Nextcloud Documentation](https://docs.nextcloud.com/).

## Troubleshooting

## Other useful links

- [Open Telekom Cloud Community](https://community.open-telekom-cloud.com/)
- [Open Telekom Cloud Documentation](https://docs.otc.t-systems.com/index.html)
- [OTC Hands-On Training by Ulrich Schneider](https://community.open-telekom-cloud.com/community?id=community_blog&sys_id=a41f28cb13d78450d15ac969a674415a&view_source=featuredList)

* [Nextcloud Documentation](https://docs.nextcloud.com/)
* [Nextcloud Deployment Recommendations](https://portal.nextcloud.com/article/nextcloud-deployment-recommendations-7.html)
