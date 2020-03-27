# Nextcloud Deployment on Open Telekom Cloud (OTC)

## Introduction

The following tutorial explains how to host the file sharing and collaboration platform [Nextcloud](https://nextcloud.com/) in the [Open Telekom Cloud](https://open-telekom-cloud.com/de) with the Cloud Container Engine. The advantage over a VM is less administration effort and faster setup of the platform with less technical expertise.

Care has been taken to ensure that the process is easy to understand. If not described otherwise, the default settings can be used. tbd.

## Tutorial

### 1. Create Virtual Private Cloud (VPC)

At first we will create a [Virtual Private Cloud (VPC)](https://docs.otc.t-systems.com/vpc/index.html).

1. Navigate to VPC and click "Create VPC Cluster"
2. Choose a VPC Name and a Subnet Name
3. Click "Create Now"
4. Go "Back to VPC List"
5. Choose your created VPC and activate SNAT

### 2. Create Cloud Container Engine (CCE)

Now we will create a [Cloud Container Engine (CCE)](https://docs.otc.t-systems.com/cce/index.html).

1. Navigate to CCE and click "Create VM Cluster"
2. Choose a Cluster Name
3. High Availability: "No"
4. Choose your created VPC and Subnet
5. You have to check "I am aware of the above limitations and read the CCE Role Management Instructions" and can go to the next site
6. Choose a Node Name and the Specifications you need. The smallest flavor could be enough
7. Be sure that your created subnet is used
8. Choose a Key Pair, if you have not one, create a key pair
9. Go to next the page, skip one and check your Details on the last page. If everything is okay you can click "Submit". This will create your Cluster.
   <details>
   <summary>
   <b>Click here to view our CCE Details</b>
   </summary>
   <img src=/docs/assets/CCE_Details.png width=75%>
   </details>
10. Submit and wait a little bit. The Cluster will be created in 5 to 10 minutes. Meanwhile we can create the other related services.

### 3. Create Relational Database Service (RDS)

Now we will create the Database for our Nextcloud. We use the [Relational Database Service (RDS)](https://docs.otc.t-systems.com/rds/index.html).

1. Navigate to RDS and create a DB Instance
2. Choose a DB Instance Name
3. You can use the smallest Memory and the cheaper Storage Type "Common I/O". According to Nextcloud 20GB per User is enough for the beginning.
4. Be sure that your created VPC will choosen
5. Choose an Administrator Password and click "Create Now"
6. Check the Details and Submit if everything is fine
   <details>
   <summary>
   <b>Click here to view our RDS Details</b>
   </summary>
   <img src=/docs/assets/RDS_Details.png width=100%>
   </details>

### 4. Create Elastic Load Balancer (ELB)

We will also need an [Elastic Load Balancer (ELB)](https://docs.otc.t-systems.com/elb/index.html).

1. Navigate to ELB and "Create Enhanced Load Balancer"
2. Choose your created VPC and the Bandwidth you want
3. Choose a name for your ELB and create the Load Balancer.
4. Check the Details and Submit if everything is fine
   <details>
   <summary>
   <b>Click here to view our ELB Details</b>
   </summary>
   <img src=/docs/assets/ELB_Details.png width=100%>
   </details>

Note: In this step an [Elastic IP](https://docs.otc.t-systems.com/eip/index.html) will be automatically created for you.

### 5. Create a persistent Volume

If the cluster is created you can create the persistent Volume.

1. Go back to CCE, go to "Resource Management" on the left navbar and go to Storage
2. Choose [Elastic Volume Service (EVS)](https://docs.otc.t-systems.com/evs/index.html) and create an EVS Disk
3. Choose a Name, check the Details and click "Submit"
   <details>
      <summary>
      <b>Click here to view our EVS Details</b>
      </summary>
      <img src=/docs/assets/EVS_Details.png width=50%>
      </details>

### 6. Create the Deployment

The last step is creating the Deployment.

1. Switch to Workloads on the left navbar and click "Create Deployment"
2. Only choose one instance. Nextcloud is not scalable at the time of this guide.
3. Click "Next" and "Add Container"
4. Select a Third-party Image with the adress "Nextcloud". The latest nextcloud image will be chosen from [dockerhub](https://hub.docker.com/_/nextcloud/)
5. Click "OK" and scroll down to "Data Storage"
6. Select Cloud Storage and add one. Choose your created EVS Storage and set the container path to _/var/www/html_
7. Click "OK" and skip the next page.
8. Now add a Service with the Access Type "Load Balancer"
9. Choose the automatic created ELB under "Automatic creation". Set the Container Port to _80_ and the Access Port to _80_
10. Click next, skip the advanced settings and "Create" the Deployment

### 7. Nextcloud Configuration

If the deployment creation was successfull you can open nextcloud on the given address under Workloads/Deployments. tbd..

## Other useful links
