# Steps to create and connect PowerVS with a VSI to a VPC with a VSI

## Preliminary requirements

1. Create an IBM Cloud account. To create an IBM Cloud account, see [Signing up for the IBM Cloud](https://cloud.ibm.com/registration).
2. Review the Identity and Access Management (IAM) information at [Managing Power Systems Virtual Servers (IAM)](https://cloud.ibm.com/docs/power-iaas?topic=power-iaas-managing-resources-and-users).
3. Create a public and private SSH key that you can use to securely connect to your Power Systems Virtual Server. To create a public and private SSH key, see [Adding an SSH key](https://cloud.ibm.com/docs/ssh-keys?topic=ssh-keys-adding-an-ssh-key).
4. (Optional) If you want to use a custom AIX or IBM i image, you must create an IBM Cloud Object Storage (COS) and upload it there. For more information, see [Deploying a custom image within a Power Systems Virtual Server]. Note that we did not need or use this with our demo.
5. (Optional) If you want to use a private network to connect to a Power Systems Virtual Server instance, order the [Direct Link Connect] service. You cannot create a private network during the VM provisioning process. You must first use the Power Systems Virtual Server user interface, command line interface (CLI), or application programming interfaced (API) to create one.

## Setup the PowerVS Service

1. Log in to the IBM catalog with your credentials.
2. In the catalog's search box, type Power Systems Virtual Server and click the Power Systems Virtual Server tile.
3. Specify a name for your service and choose where you'd like to deploy your Power Systems Virtual Server instance. For our demo we chose Sydney 04 (syd04). Optionally, you can select a Resource Group if you wish to separate this service from other resourcess. Otherwise, the Default Resource Group is used.
4. Click Create. You are redirected to the Resource List.
5. From the Resource List, select your Power Systems Virtual Server service under Services.

## Create a Private Subnet

1. Click on Subnets from the left menu
2. Click Create Subnet
3. Give the Subnet a name
4. Set the CIDR, eg. 192.168.50.0/24
5. (Note that a Cloud Connection will be needed, but will be set after creation of the VPC)
6. Click Create Subnet

## Create and Configure an IBM i Virtual Server Instance

1. Click Create instance under Virtual server instances. 
2. Give the instance a name
3. Choose an existing SSH key or create one to securely connect to your Power Systems Virtual Server.
4. For Boot Image, choose IBM i and select the latest available image (in our case IBMi-74-05-2984-1). No extra IBM i licenses are needed.
5. For Tier, choose Tier 3 (3 IOPs / GB ).
6. Select Machine Type as s922.
7. Increase memory to 8 GB.
8. For Public Networks, we turned this on, however it is not required.
9. For Private Network, select the Subnet created above.
10. Select "I agree to the Terms and conditions"
11. All other parameters can be left at their defaults. Click Create instance

## Create a Virtual Private Cloud

1. From the top left menu, select VPC Infrastructure.
2. From the left menu, under Networking, select VPCs.
3. Select Region as Sydney.
4. Click Create.
5. Give the VPC a name.
6. Change the Resource Group to match PowerVS (or leave as Default if the PowerVS Service is in Default).
7. The VPC will have subnets created for all 3 zones. Take note of the subnet for Sydney 2 zone - name and CIDR.
8. Click Create virtual private cloud

## Create a Virtual Server Instance

1. From the VPC Infrastructure menu on the left, under Compute, select Virtual server instances.
2. Click Create
3. 
    


[Direct Link Connect]: https://cloud.ibm.com/docs/power-iaas?topic=power-iaas-ordering-direct-link-connect#steps-to-order-direct-link-connect
[Deploying a custom image within a Power Systems Virtual Server]: https://cloud.ibm.com/docs/power-iaas?topic=power-iaas-deploy-custom-image
