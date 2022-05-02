# Creating and connecting PowerVS with a VSI to a VPC with a VSI

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

1. Click on Subnets from the left menu.
2. Click Create Subnet.
3. Give the Subnet a name.
4. Set the CIDR, eg. 192.168.50.0/24.
5. (Note that a Cloud Connection will be needed, but will be set after creation of the VPC).
6. Click Create Subnet.

## Create and Configure an IBM i Virtual Server Instance

1. Click Create instance under Virtual server instances. 
2. Give the instance a name.
3. Choose an existing SSH key or create one to securely connect to your Power Systems Virtual Server.
4. For Boot Image, choose IBM i and select the latest available image (in our case IBMi-74-05-2984-1). No extra IBM i licenses are needed.
5. For Tier, choose Tier 3 (3 IOPs / GB ).
6. Select Machine Type as s922.
7. Increase memory to 8 GB.
8. For Public Networks, we turned this on, however it is not required.
9. For Private Network, select the Subnet created above.
10. Select "I agree to the Terms and conditions"
11. All other parameters can be left at their defaults. Click Create instance.

## Create a Virtual Private Cloud

1. From the top left menu, select VPC Infrastructure.
2. From the left menu, under Networking, select VPCs.
3. Select Region as Sydney.
4. Click Create.
5. Give the VPC a name.
6. Change the Resource Group to match PowerVS (or leave as Default if the PowerVS Service is in Default).
7. The VPC will have subnets created for all 3 zones. Take note of the subnet for Sydney 2 zone - name and CIDR.
8. Click Create virtual private cloud.

## Create a Virtual Server Instance

1. From the VPC Infrastructure menu on the left, under Compute, select Virtual server instances.
2. Click Create.
3. Check that location is set to zone Sydney 2.
4. Enter a name for the VSI.
5. Select the appropriate Resource Group, if not Default.
6. Select Operating system image. We created two VSIs - one using Red Hat Enterprise Linux, with Version ibm-redhat-8-4-minimal-amd64-2, and the other as a Windows Server, with Version ibm-windows-server-2016-full-standard-amd64-8.
7. Adjust the Profile, if needed - View all profiles - bx2-4x16 was used for Windows. The default bx2-2x8 was used for Linux.
8. Select an existing SSH key (or create a new one).
9. For Networking, select the VPC created above.
10. For Network Interfaces, check that the subnet for the Sydney 2 zone is selected.
11. Click Create virtual server.

## Create Cloud Connect

This will provide the connection between the private subnet of your PowerVS Service and the private subnet of your VPC.

Note that it may be required to turn on VRF (Virtual Router Forwarding) on your account, which requires that a support ticket be raised on the Engineering Team. [Instructions here](https://cloud.ibm.com/docs/account?topic=account-vrf-service-endpoint&mhsrc=ibmsearch_a&mhq=VRF&interface=ui).
    
Creating the Cloud Connect (also known as Direct Link Connect 2.0) is best done from the IBM Cloud Shell, which is accessed from the top right of the IBM Cloud web console. Once at the console:
```
ic target -r au-syd
ic target -g <resource group>
ic is vpcs
ic is vpc <your_vpc>
```
From the above, get the crn, which will look like: 
crn:v1:bluemix:public:is:au-syd:a/1dd42ed8fa355cd281b09533a423511d::vpc:r026-7af92dbe-8e45-4294-a802-2863dc213039

```
ic pi conc <dl_connection_name> --speed 5000 --vpc --vpcID <your_crn> --global-routing
```    
This will come back with something like: 
```
Cloud connection pvs-dl-150122 creation accepted. Please check the status later. 
                    
ID               25d1bc31-44da-4d4d-97e7-ae18d2974db1   
Name             pvs-dl-150122   
Link Status      idle   
Speed            5000   
Creation Date    2022-03-11T01:45:07.527Z   
Global Routing   true   
IBM IPAddress    169.254.0.1/30   
User IPAddress   169.254.0.2/30   
Metered          false   
              
Job ID     f181a918-eecb-4d91-a8e2-b23d545b3d99   
Job Link   pcloud/v1/cloud-instances/99937570ffa045919937418ba0202bc6/jobs/f181a918-eecb-4d91-a8e2-b23d535b3d99

```
You can check the status with:
```
ic pi con 25d1bc31-44da-4d4d-97e7-ae18d2974db1
```
That is,  ```ic pi con <ID> ```  as given above from the ic pi conc command.
    
Once the Direct Link Connection is created, you may need to add a static route to each server to ensure local traffic goes through the private subnets rather than through the external IPs. Eg. on Linux:
```
ip route add 192.168.50.0/24 via 10.245.64.1
```

You should now be able to ping and connect from the Instances within the VPC to the Instances within the PowerVS Service.

Note that by default, the VPC subnet is created with a Security Group (firewall) that only allows certain inbound ports. To add or modify:
In VPC Infrastructure > Network > Security Groups > pick the Security group used by the VSI > Rules > Create or Edit (both Inbound and Outbound).


[Direct Link Connect]: https://cloud.ibm.com/docs/power-iaas?topic=power-iaas-ordering-direct-link-connect#steps-to-order-direct-link-connect
[Deploying a custom image within a Power Systems Virtual Server]: https://cloud.ibm.com/docs/power-iaas?topic=power-iaas-deploy-custom-image
