# Mini Project - Create a Container Web Server with an ARM Template

_**DIAGRAM**: Stage 3 diagram coming soon._  
_**VIDEO**: Stage 3 video coming soon._  

In stage 3 you will configure a Key Vault to store the admin password that we have been typing in manually so far. We'll update the template files to reference this secret so it can be retrieved securely.  

The steps you will complete are as follows:
- Setup a new Key Vault with the admin password secret
- Create a new copy of your template files (stage3-)
- Modify the parameter to reference the Key Vault

_Note: If you would like to complete this stage of the mini project by yourself without following the **step-by-step** instructions, you can do so by completing the above goals. You will need to use the following for the parameter file:_   

```
    "adminPassword": {
        "reference": {
            "keyVault": {
            "id": "/subscriptions/SUBID/resourceGroups/RGNAME/providers/Microsoft.KeyVault/vaults/KVNAME"
            },
            "secretName": "adminPassword"
        }
    }
```

You can see more information about references secrets in this [Microsoft article](https://learn.microsoft.com/en-us/azure/azure-resource-manager/templates/key-vault-parameter?tabs=azure-cli#reference-secrets-with-static-id). We will use a static reference. 

# STAGE 3A - Configure Key Vault

First we need to create the Key Vault in the portal. 

Navigate to the Create a Resource hub in the Azure Portal https://portal.azure.com/#create/hub.   
Search for `key vault` (it may already be onscreen).  
Click `Create`.  
Configure the Key Vault with the following settings.  
Click `Next` to move through each section.  
If the setting is not specified below, leave it as-is.  

Use the following values for your Key Vault:
- **Basics**
    - **Subscription**: Choose your own subscription
    - **Resource group**: webapp-auto-rg (use existing)
    - **Key vault name**: create a unique name (whatever you like)
    - **Region**: Pick the same region as your VM
    - **Pricing tier**: Standard
    - **Days to retain deleted vaults**: Pick the lowest (7)
    - **Purge protection**: Disabled
- **Access Configuration**:
    - **Permission model**: Azure role-based access control
    - **Resource access**: Tick `Azure resource manager for template deployment`
- **Networking**
    - **Enable public access**: Ticked
    - **Public access**: All networks
- **Tags**: Leave empty
 
Click `Review + create`.  
Click `Create`.  
 
_Note: You should wait for deployment to complete before continuing to the next step._  

## Configure data plane access

Owners/administrators in Azure do not have data plane access to Key Vault by default. Follow the steps to grant yourself access:   

Navigate to the Key Vaults hub in the Azure Portal https://portal.azure.com/#browse/Microsoft.KeyVault%2Fvaults.   
Click on the vault you just created.  
Go to `Access control (IAM)`.   
Click `+ Add` > `Add role assignment`.  
Locate the `Key Vault Administrator` role (use search if needed).  
Select the role and then click `Next`.  
Choose `User, group, or service principal`.  
Click `+ Select members`.   
Locate and click the account you're logged in with now.  
Click `Select`.  
Click `Review + assign`.  
Click `Review + assign` again.  

_Wait for the success notification at the top of the screen before continuing to the next step._    

## Create the secret

Navigate to `Objects` > `Secrets` within the Key Vault.  
Click on `+ Generate/Import` at the top.  
Enter the secret as follows:

- **Upload options**: Manual
- **Name**: adminPassword
- **Secret value**: enter in a password of your choice
- Leave the other options as-is

Click `Create`.  

# STAGE 3B - Setup Template Files

Start by making a copy of the template files.  
You can choose to do this with a GUI, or with the CLI commands below:  

```
cd ~/projects/azure-vm-webapp-iac/
cp stage2-deploy.json stage3-deploy.json  
cp stage2-params.json stage3-params.json  
```

# STAGE 3C - Add the Key Vault Reference

Using your editor of choice, you will need to modify the parameter file. Use the code below as a guide.  

## Get your subscription ID

Navigate to the Azure home page.  
Click on `Subscriptions` https://portal.azure.com/#view/Microsoft_Azure_Billing/SubscriptionsBladeV2.  
Click your subscription.  
Note/copy the `Subscription ID` for the next step.  


## Update the parameter file

Open the `stage3-params.json` file you just made.  
Ensure you use the subscription ID (SUBID), resource group name (RGNAME) and Key Vault name (KVNAME) from your own environment.  

Modify the existing parameter to match the below with your updates:  

```
    "adminPassword": {
        "reference": {
            "keyVault": {
            "id": "/subscriptions/SUBID/resourceGroups/RGNAME/providers/Microsoft.KeyVault/vaults/KVNAME"
            },
            "secretName": "adminPassword"
        }
    },
```

# STAGE 3D - Deploy the Update

Let's now use our updated files to update our deployment.  

## Prepare with Azure CLI

Login to Azure if you haven't already (`az login`).  
Make sure you're in the correct folder (eg. `cd ~/projects/azure-vm-webapp-iac`).  
  

## Run the deployment

```
az deployment group create --resource-group webapp-auto-rg --template-file stage3-deploy.json --parameters @stage3-params.json 
```

# STAGE 3E - Test

When you created the deployment above, it should have completed with a large text output. You can check the details in the portal also.  

## Check the deployment status in the portal

Go to the Azure Portal home page > Resource Groups https://portal.azure.com/#browse/resourcegroups.  
Click the Resource Group we've been using, `webapp-auto-rg`.     
Click `Settings` > `Deployments`.  
Click `stage3-deploy`.  
The deployment should be complete without errors.  

## Check that the website is working  

_Only complete this step if the deployment is complete/successful above._  

Open the VM blade https://portal.azure.com/#browse/Microsoft.Compute%2FVirtualMachines.    
Click the `webvm1` VM (in webapp-auto-rg).  
Copy the public IP.  
Open the IP in a new tab (e.g. http://publicIP).  

You should now see the Aus-E-Mart web application.  

# STAGE 3 - FINISH  

This solution has several limitations which you will address within the upcoming stages of this mini project:

- ~~The resources were built manually (VM, VNet, NSG, etc)~~
- ~~The admin credentials for the VM are typed manually~~
- Docker had to be installed/configured manually
- The Web App was manually built and deployed


You can now move onto STAGE4.  

## Completed Resources

