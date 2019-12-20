# F5-BIGIP Virtual Server for Virtual Private Cloud using Custom Image

Use this template to create F5-BIGIP virtual server using custom image from your IBM Cloud account in IBM Cloud [VPC Gen2](https://cloud.ibm.com/vpc-ext/overview) by using Terraform or IBM Cloud Schematics.  Schematics uses Terraform as the infrastructure-as-code engine.  With this template, you can create and manage infrastructure as a single unit as follows. For more information about how to use this template, see the IBM Cloud [Schematics documentation](https://cloud.ibm.com/docs/schematics).

_The F5-BIGIP virtual server instance only supports BYOL at this time._

## Prerequisites

- Must have access to [Gen 2 VPC](https://cloud.ibm.com/vpc-ext/network/vpcs).
- The given VPC must have at least one subnet IP address unassigned - the F5-BIGIP VSI will be assigned a IP Address from the user provided subnet as an input.

## Costs

When you apply template, the infrastructure resources that you create incur charges as follows. To clean up the resources, you can [delete your Schematics workspace or your instance](https://cloud.ibm.com/docs/schematics?topic=schematics-manage-lifecycle#destroy-resources). Removing the workspace or the instance cannot be undone. Make sure that you back up any data that you must keep before you start the deletion process.


* _VPC_: VPC charges are incurred for the infrastructure resources within the VPC, as well as network traffic for internet data transfer. For more information, see [Pricing for VPC](https://cloud.ibm.com/docs/vpc-on-classic?topic=vpc-on-classic-pricing-for-vpc).
* _VPC Custom Image_: The template will copy over a custom F5 image - this can be a one time operation.  F5 virtual instances can be created from the custom image.  VPC charges per custom image.
* _F5-BIGIP Instances_: The price for your virtual server instances depends on the flavor of the instances, how many you provision, and how long the instances are run. For more information, see [Pricing for Virtual Servers for VPC](https://cloud.ibm.com/docs/infrastructure/vpc-on-classic?topic=vpc-on-classic-pricing-for-vpc#pricing-for-virtual-servers-for-vpc).

## Dependencies

Before you can apply the template in IBM Cloud, complete the following steps.


1.  Ensure that you have the following permissions in IBM Cloud Identity and Access Management:
    * `Manager` service access role for IBM Cloud Schematics
    * `Operator` platform role for VPC Infrastructure
2.  Ensure the following resources exist in your VPC Gen 2 environment
    - VPC
    - SSH Key
    - VPC has a subnet
    - _(Optional):_ A Floating IP Address to assign to the F5-BIGIP instance post deployment

## Configuring your deployment values

When you select the [`F5-1arm-offering`template](https://cloud.ibm.com/catalog/content/F5-1arm-offering) from the IBM Cloud catalog, you can set up your deployment variables from the `Create` page. Once the template is applied, IBM Cloud Schematics  provisions the resources based on the values that were specified for the deployment variables.

### Required values
Fill in the following values, based on the steps that you completed before you began.

| Key | Definition |
| --- | ---------- |
| `zone` | The VPC Zone that you want your VPC virtual servers to be provisioned. To list available zones, run `ibmcloud is zones` |
| `resource_group` | The resource group to use. If unspecified, the account's default resource group is used. To list available resource groups, run `ibmcloud resource groups` |
| `vpc_name` | The name of your VPC in which F5-BIGIP VSI is to be provisioned. |
| `ssh_key_name` | The name of your public SSH key to be used for F5-BIGIP VSI. Follow [Public SSH Key Doc](https://cloud.ibm.com/docs/vpc-on-classic-vsi?topic=vpc-on-classic-vsi-ssh-keys) for creating and managing ssh key. |
| `vnf_image_copy` | Copy vendor custom image to your IBM Cloud account VPC Infrastructure (y/n)? First time, the image needs to be copied to your cloud account. It should be `y` for the first time. For the next runs, customer can skip image copy, if image is already copied by entering `n`. Accepted values in this field are `y` or `n` |
| `vnf_vpc_image_name` | The name of the F5 Custom Image to be provisioned in your IBM Cloud account and (if already available) to be used to create the F5-BIGIP virtual server instance. |
| `vnf_profile` | The profile of compute CPU and memory resources to be used when provisioning the vnf instance. To list available profiles, run `ibmcloud is instance-profiles`. |
| `vnf_instance_name` | The name of the VNF instance to be provisioned. |
| `subnet_id` | The ID of the subnet where the VNF instance will be deployed. Click on the subnet details in the VPC Subnet Listing to determine this value | 
| `ibmcloud_endpoint` | The IBM Cloud environment `cloud.ibm.com` or `test.cloud.ibm.com` |

### Outputs
After you apply the template your VPC resources are successfully provisioned in IBM Cloud, you can review information such as the virtual server IP addresses and VPC identifiers in the Schematics log files, in the `Terraform SHOW and APPLY` section.

| Variable Name | Description | Sample Value |
| ------------- | ----------- | ------------ |
| `resource_name` | Name of the F5-BIGIP instance | N/A |
| `resource_status` | Status of the F5-BIGIP instance | `Running` or `Failed` |
| `VPC` | The VPC ID | `r134-7a9df886-xxxx-yyyy-zzzz-67c6dd202337` |
| `f5_admin_portal` | Web url to interact with F5-BIGIP admin portal - `https://<Floating IP>:8443` | `https://192.168.1.1:8443` |

## Notes

If there is any failure during F5-BIGIP VSI creation, the created resources must be destroyed before attempting to instantiate again. To destroy resources go to `Schematics -> Workspaces -> [Your Workspace] -> Actions -> Delete` to delete  all associated resources. <br/>
1. The BYOI image download may take a while. Timeout has been set 30 Minutes.
2. Do not modify or delete subnets and floating/public IPs used by the F5-BIGIP instance.

# Post F5-BIGIP Instance Spin-up

1. From the VPC list, confirm the F5-VSI is powered ON with green button
2. Assign a Floating IP to the F5-VSI. Refer the steps below to associate floating IP
    - Go to `VPC Infrastructure Gen 2` from IBM Cloud
    - Click `Floating IPs` from the left pane
    - Click `Reserve floating IP` -> Click `Reserve IP`
    - There will be a (new) Floating IP address with status `Unassociated`
    - Click Three Dot Button corresponding to the Unassociated IP address -> Click `Associate`
    - Select F5 instance from `Instance to be associated` column.
    - After clicking `Associate`, you can see the IP address assigned to your F5 Instance.
3. In the Security group,  here are the steps to open the port 8443
    - Go to `VPC Infrastructure Gen 2` from IBM Cloud
    - Click `Security groups` from the left pane
    - Click the security group which is corresponding to your VPC
    - Click `New Rule` in "Inbound Rules" column.
    - Select `Protocol` as "TCP"
    - Select `Port Range` under Port
    - Give `8443` port number in `Port min` and `Port max`
    - Select `Source type` as `Any`
    - Click `Save`, and a new rule will be added to your security group
4. From the browser, open `https://<Floating IP>:8443`, login using user and password 
5. In First login, it will ask to reset the password.
6. Once the password got reset. You will be able to login to F5 instance admin portal.
