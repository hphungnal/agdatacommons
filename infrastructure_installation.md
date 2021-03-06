## Executing the Terraform Plan
The starting point for the deployment of the infrastructure is an Azure subscription and a resource group that will host all the resources.  With these in place the administrator should perform the following tasks via the Azure Portal Cloud Shell:

1. Get a list of subscription ID and tenant ID values:

`az account list --query "[].{name:name, subscriptionId:id, tenantId:tenantId}"`

2. Set the SUBSCRIPTION_ID environment variable with the value for the subscription where the infrastructure will be deployed:

`az account set --subscription="${SUBSCRIPTION_ID}"`

3. Create a service principal for use by Terraform and Jenkins:

`az ad sp create-for-rbac --role="Contributor" --scopes="/subscriptions/${SUBSCRIPTION_ID}"`

Your appId, password, sp_name, and tenantId are returned. **Make a note of the appId and password** as these cannot be recovered after this point.

4. Save these values in a shell script:

```
#!/bin/sh
echo "Setting environment variables for Terraform"
export ARM_SUBSCRIPTION_ID=your_subscription_id
export ARM_CLIENT_ID=your_appId
export ARM_CLIENT_SECRET=your_password
export ARM_TENANT_ID=your_tenant_id
```

5. Execute the shell script to set the required environment variables.
Save the Terraform file to the console storage.

6. Initialize the Terraform deployment:

`terraform init`

The system should return a message indicating successful initialization.

7. Update the resource group name in the Terraform file.

8. Import the resource group into the terraform state file.

`terraform import azurerm_resource_group.<resource_group_name>`

9. Review the actions to be completed by the Terraform script with `terraform plan` command. When ready to create the resource group, apply the Terraform plan as follows:

`terraform apply`

Terraform will output the execution plan and prompt to proceed.  Answer yes to proceed.

After provisioning via Terraform the  client will have the option to administer the infrastructure via Terraform, the Azure Portal or Azure Client Interface.  If either of the latter two options are used, it will be critical not to return to Terraform without accurately updating the known state of the infrastructure using the import function in Terraform.  Failure to do so can lead to unwanted deletion and/or recreation of resources under management.  Always examine the plan before agreeing to any changes proposed by Terraform.
 
The Terraform plan, state files and access credentials can be stored on Azure cloud storage and accessed via the Cloud Shell that is part of the Azure portal.  The storage is encrypted and only accessible to pre-defined resources and accounts that have been granted access.  

The resource group will contain the production infrastructure and one or more copies of the stage, QA and test infrastructures.  Each copy of the infrastructure will be composed of a unique Public IP address, MariaDB database service instance, and container group with the Web and SOLR docker containers.
