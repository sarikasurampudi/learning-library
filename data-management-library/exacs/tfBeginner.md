# Terraform and Oracle Cloud Infrastructure

## Table of Contents

[Overview](#overview)

[Prerequisites](#Prerequisites)

[Step 1: Generate SSH Keys](#practice-1-generate-ssh-keys)

[Step 2: Signing in OCI Console](#practice-2-signing-in-oci-console)

[Step 3: Collect the Required OCIDs for Terraform ](#practice-3-collect-the-required-ocids-for-terraform)

[Step 4: Create a Virtual Cloud Network](#practice-4-create-a-virtual-cloud-network)

[Step 5: Creating a Terraform Instance](#practice-5-creating-a-terraform-instance)

[Step 6: Generate an API Signing Key](#practice-6-generate-an-api-signing-key)

[Step 7: Upload the Public Key to api user](#practice-7-upload-the-public-key-to-api-user)

[Step 8:  Install and configure Terraform](#practice-8-install-and-configure-terraform)

### Need Help?
Please submit feedback or ask for help using our [LiveLabs Support Forum](https://community.oracle.com/tech/developers/categories/livelabsdiscussions). Please click the **Log In** button and login using your Oracle Account. Click the **Ask A Question** button to the left to start a *New Discussion* or *Ask a Question*.  Please include your workshop name and lab name.  You can also include screenshots and attach files.  Engage directly with the author of the workshop.

If you do not have an Oracle Account, click [here](https://profile.oracle.com/myprofile/account/create-account.jspx) to create one.
## Overview

The lab exercises are designed to complement your training, reinforcing the key concepts by applying and demonstrating what you learned in the presentation sessions. This lab book is comprised of individual exercises. These exercises allow you to get first hands-on exposure working with the Oracle Cloud Infrastructure (OCI) using a demo environment, where you will see how key features and functionality are deployed in the software. Using what you learn in the presentations and individual exercises working with the software, you will collaborate as a team in developing and delivering practice presentations.


## Prerequisites

- Oracle Cloud Infrastructure account credentials (User, Password, and Tenant)
- Region and compartment with limits for a few small instances (1.1 or 2.1 is sufficient)
- A VCN created with at least one subnet, internet gateway, default route to internet gateway and security groups allowing at least port 22/SSH in.
- Ability to SSH into compute instances

## Task 1: Generate SSH Keys

Generate SSH keys to be used later while launching an instance.

#### MAC/LINUX 

1) Generate ssh-keys for your machine if you don’t have one. As long as an id_rsa and id_rsa.pub key pair is present they can be reused. By default these are stored in ~/.ssh
```   
<copy>ssh-keygen</copy>
```
2) Make sure permissions are restricted, sometimes ssh will fail if private keys have permissive permissions.
``` 
<copy>chmod 0700 ~/.ssh  
chmod 0600 ~/.ssh/id_rsa  
chmod 0644 ~/.ssh/id_rsa.pub</copy>
```
#### FOR WINDOWS

1) Install git for windows. Download <https://github.com/git-for-windows/git/releases/download/v2.13.0.windows.1/Git-2.13.0-64-bit.exe> and install.

2) Open Git-bash:
    
  ![](https://raw.githubusercontent.com/oracle/learning-library/master/oci-library/L100-LAB/Compute_Services/media/image1.png)

3) Generate ssh-keys by running this command in Git Bash
```
<copy>ssh-keygen</copy>
```

```
Generating public/private rsa key pair.  
Enter file in which to save the key
(/c/Users/username/.ssh/id\_rsa):  
Created directory '/c/Users/username/.ssh'.  
Enter passphrase (empty for no passphrase):  
Enter same passphrase again:  
Your identification has been saved in /c/Users/username/.ssh/id\_rsa.  
Your public key has been saved in /c/Users/username/.ssh/id\_rsa.pub.  
```
> **Note**: In Git-Bash, C:\\Users\\username\\ is shown as /c/Users/username/

## Task 2: Signing in OCI Console

1) Open a supported browser and go to the Console URL. For example, [https://console.us-ashburn-1.oraclecloud.com](https://console.us-ashburn-1.oraclecloud.com).

2) Enter your tenant name: <Tenant> and click **Continue**

  ![](https://raw.githubusercontent.com/oracle/learning-library/master/oci-library/L100-LAB/Identity_Access_Management/img/image001.png)

3) Oracle Cloud Infrastructure is integrated with Identity Cloud Services, you will see a screen validating your Identity Provider. You can just click **Continue**.

  ![](https://raw.githubusercontent.com/oracle/learning-library/master/oci-library/L100-LAB/Identity_Access_Management/img/image002.png)

 4) Enter your user name and password

 - **Username:** cloud.admin
 - **Password:** _instructor will provide password_
 
![](https://raw.githubusercontent.com/oracle/learning-library/master/oci-library/L100-LAB/Identity_Access_Management/img/image003new.png)

When you sign in to the Console, the home page is displayed.

 ![](https://raw.githubusercontent.com/oracle/learning-library/master/oci-library/L100-LAB/Identity_Access_Management/img/image004.png)

The home page gives you quick links to the documentation and to Oracle Support.

## Task 3: Collect the Required OCIDs for Terraform
In order to automate with terraform we need to collect a few OCIDs in advance. One thing to note that along with a Tenant/Cloud Tenant, username and password you also received a Compartment Id. Compartments are sub-areas or sub-domains within a tenancy to which you have been delegated full control. In the next few steps we will be collecting the Tenancy OCID, the Compartment OCID and the User OCID.

1) After login go to **Menu** > **Administration** > **Tenancy Details**. Take a note of the Tenancy OCID information. Save on your notepad. 

![](https://raw.githubusercontent.com/oracle/learning-library/master/oci-library/DevOps/Terraform/img/image001.png)

2) The correct compartment for this exercise is **Demo**. If you don't select the correct compartment, none of the steps below will work.  On **Menu** select **Identity** > **Compartments**. Take a note of the Compartment OCID information. Save on your notepad. 

![](https://raw.githubusercontent.com/oracle/learning-library/master/oci-library/DevOps/Terraform/img/image002.png)

3) For this exercise, we will use **api.user**. Navigate to **Identity**> **Users** and take a note of OCID for the api.user.  Save on your notepad.

![](https://raw.githubusercontent.com/oracle/learning-library/master/oci-library/DevOps/Terraform/img/image003.png)

**Summary:** This exercise consisted of logging into the OCI Console and collecting the tenancy, compartment and user OCIDs. Save these for use later with terraform. You will have something similar to that:

![](https://raw.githubusercontent.com/oracle/learning-library/master/oci-library/DevOps/Terraform/img/image004.png)

## Task 4: Create a Virtual Cloud Network
A Virtual Cloud Network (VCN) is a virtual version of a traditional network including subnets, route tables, and gateways on which your compute instances run. Customers can bring their network topology to the cloud with VCN. Creating a VCN involves a few key aspects such as:

 - Allocate a private IP block for the cloud (CIDR range for the VCN).
 - Customers can bring their own RFC1918 IP addresses.
 - Create Subnets by partitioning the CIDR range into smaller networks (sub networks for front end, back end, database)
 - Create an optional Internet Gateway to connect VCN subnet with Internet. Instances created in this subnet will have a public IP address.
 - Create Route table with route rules for Internet access
 - Create Security List to allow relevant ports for ingress and egress access

 1) Create a Virtual Cloud Network with Public Subnets:

 - One public subnet per Availability Domain
 - An Internet Gateway
 - A corresponding route rule in the default route table
 - The default security list
 - The default set of DHCP options

> **Note:** You can launch one or more compute instances in a subnet. Each instance gets both a public and private IP address. The launch instance dialog now has a check box for choosing whether the instance has a public IP address. You can communicate with the instances over the Internet via the public IP address from your on-premises network.

2) Open the Console, click **Networking**.

3) Select a compartment on the left that you have permission to work in.

![](https://raw.githubusercontent.com/oracle/learning-library/master/oci-library/L100-LAB/Virtual_Cloud_Network/media/image001.png)

4) Click **Create Virtual Cloud Network** and enter the following details:

- **Create in Compartment:** This field defaults to the currently selected compartment. Select the compartment you want to create the VCN in, if not already selected.
- **Name:** Enter a name for your cloud network (for example, VCN-DEMO).

> **Note:** Enter a friendly name for the cloud network. It doesn't have  to be unique, and it cannot be changed later in the Console (but you can change it with the API).

![](https://raw.githubusercontent.com/oracle/learning-library/master/oci-library/L100-LAB/Virtual_Cloud_Network/media/image002.png)

5) Select **Create Virtual Cloud Network plus related resources**. The dialog box expands to list the items that will be created with your cloud network.

>**Note:** This option is the quickest way to get a working cloud network in the fewest steps.

6) Scroll to the bottom of the dialog box and click **Create Virtual Cloud Network**.

A confirmation page displays the details of the cloud network that you just created.

![](https://raw.githubusercontent.com/oracle/learning-library/master/oci-library/L100-LAB/Virtual_Cloud_Network/media/image003.png)


 The Virtual Cloud Network above has the following resources and characteristics:

  - CIDR block range of 10.0.0.0/16
  - An Internet Gateway
  - A route table with a default route rule to enable traffic between VCN and the Internet Gateway
  - A default security list that allows specific ingress traffic to and all egress traffic from the instance
  - A public subnet in each Availability Domain
  - The VCN will automatically use the Internet and VCN Resolver for DNS

## Task 5: Creating a Terraform Instance

Oracle Cloud Infrastructure Compute lets you provision and manage compute hosts, known as  instances. You can launch instances as needed to meet your compute and application requirements. After you launch an instance, you can access it securely from your computer, restart it, attach and detach volumes, and terminate it when you're done with it. Any changes made to the instance's local drives are lost when you terminate it. Any saved changes to volumes attached to the instance are retained.

1) Navigate to the **Compute** tab and click **Create Instance**. We will launch a VM instance for this lab.

2) The launch browser will automatically fill in the details for launching a single Oracle Linux. Please check each field for better understanding. 

 - **Name**: Terraform-Instance
 - **Shape**: VM.Standard 1.1 or VM.Standard1.2) or the shape available for your tenancy
 - **Availability Domain**:  (AD1, AD2 or AD3)
 - **Boot Volume**: Oracle-Provided OS Image
 - **Image Operating System**: Oracle Linux 7.5
 - **Shape type** : Virtual Machine
 - **Image Version**: Latest
 - **Boot Volume Configuration**: Use default
 - **ssh keys** : Attach the .pub key created in the Practice-1
 - **Networking** : 
   - **VCN Name**: VCN you created in Step 3
   - **Subnet**: Any subnet among the three ADs

![](https://raw.githubusercontent.com/oracle/learning-library/master/oci-library/L100-LAB/Compute_Services/media/Create1.png)

![](https://raw.githubusercontent.com/oracle/learning-library/master/oci-library/L100-LAB/Compute_Services/media/Create2.png)


Launching an instance is simple and intuitive with few options to select. Provisioning of the compute instance will complete in less than a minute and the instance state will change from provisioning to running.

3) Once the instance state changes to Running, you can SSH to the Public IP address of the instance.
    
![](https://raw.githubusercontent.com/oracle/learning-library/master/ospa-library/luna-labs/oci-compute/media/image017.png)

4) To connect to the instance, you can use ‘Terminal’ if you are using MAC or Gitbash if you are using Windows.

> **Note:** *For Oracle Linux VMs, the default username is **opc**
```
<copy>ssh -i /path/privateKey opc@PublicIP_Address</copy>
```

## Task 6: Generate an API Signing Key

You can use the following [OpenSSL](http://www.openssl.org/) commands to generate the key pair in the required PEM format:

1) Create a `.oci` directory to store the credentials:
```
<copy>mkdir ~/.oci</copy>
```
2) Generate the private key with one of the following commands:

```
<copy>openssl genrsa -out ~/.oci/oci_api_key.pem 2048</copy>
```
3) Ensure that only you can read the private key file:
```
<copy>chmod go-rwx ~/.oci/oci_api_key.pem</copy>
```
4) Generate the public key:
```
<copy>openssl rsa -pubout -in ~/.oci/oci_api_key.pem -out ~/.oci/oci_api_key_public.pem</copy>
```
5) Copy the contents of the public key to your notepad (you will need to paste the value into the Console later):
```
<copy>cat ~/.oci/oci_api_key_public.pem</copy>
```
6) Get the key's fingerprint with the following OpenSSL command.
```
<copy>openssl rsa -pubout -outform DER -in ~/.oci/oci_api_key.pem | openssl md5 -c</copy>
```

**Note: Copy the output of the step 5 and 6 and save to your notepad**


## Task 7:  Upload the Public Key to api user

You can upload the PEM public key in the Console, located at [https://console.us-ashburn-1.oraclecloud.com](https://console.us-phoenix-1.oraclecloud.com/). 

1) View the details for the api.user who will be calling the API with the key pair. Click **Identity**, click **Users**, and then select the **api.user** from the list.

2) Click  **Add Public Key**.

3) Paste the contents of the PEM public key in the dialog box and click  **Add**.

**Note**: The key's fingerprint is displayed (for example, 12:34:56:78:90:ab:cd:ef:12:34:56:78:90:ab:cd:ef). It must match the fingerprinted generated on step 6 from Practice-6


## Task 8: Install and configure Terraform

Oracle Cloud Infrastructure (OCI) platform can run both Oracle workloads and cloud native applications. In this hands-on lab the process of getting Terraform installed will be covered along with configuring the OCI Terraform plugin and performing some live test runs with Terraform using sample scripts.

1)  Connect to your Oracle Linux instance using SSH and run the following commands:
```
<copy>sudo yum install -y terraform terraform-provider-oci</copy>
```
2) Clone the following git hub repository:
```
<copy>git clone https://github.com/flavio-santino/terraform-examples.git</copy>
```
3) Inside of terraform-examples folder you will find two directories:
```
<copy>cd terraform-examples
ls -l
oracle-linux-instance
web-app-load-balancer</copy>
```
4) Enter on ``oracle-linux-instance`` folder and edit ``env_vars`` file:

**Note: You will have to adjust the variables to use the data you collected before**
```
<copy>cd oracle-linux-instance</copy>
```

```
<copy>vi env_vars</copy>
```

```
## Terraform Authentication Details
<copy>export TF_VAR_tenancy_ocid="<tenancy-ocid>"
export TF_VAR_user_ocid="<user-ocid>"
export TF_VAR_compartment_ocid="<compartment-ocid>"
export TF_VAR_fingerprint="<fingerprint>"
export TF_VAR_private_key_path="<path-to-oci_api_key.pem>"
export TF_VAR_region="us-ashburn-1"</copy>
```
```
### Public/private keys used on the instance
<copy>export TF_VAR_ssh_public_key=$(cat path-to-public-ssh-key)
export TF_VAR_ssh_private_key=$(cat path-to-private-ssh-key)</copy>
```

5) Save your ``env_vars`` file. Now you can deploy your infrastructure executing the following steps:
```
<copy>source env_vars</copy>
```

```
<copy>terraform init</copy>
```

```
<copy>terraform plan</copy>
```

```
<copy>terraform apply</copy>
```


