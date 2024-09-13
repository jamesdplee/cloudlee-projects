# Mini Project - Create a Container Web Server with an ARM Template

_**DIAGRAM**: Stage 2 diagram coming soon._  
_**VIDEO**: Stage 2 video coming soon._  

In stage 2 of this mini project you are going to create an ARM template for the web server. The ARM template can be used to more consistently, simply, and securely deploy resources instead of the manual approach we completed in stage 1.  

The steps you will now complete in stage 2 are as follows:
- Download the ARM template from stage 1
- Modify the ARM template to allow HTTP inbound
- Delete the manual deployment from stage 1

_Note: You can complete the above steps by yourself if you wish. No further information is required. For step-by-step instructions, continue to read the following._  


# STAGE 2A - Create an ARM Template

_Note: You can perform the following Azure CLI commands wherever you wish (on your computer, on a development VM, [Azure Cloud Shell](https://shell.azure.com), etc). Be aware we will need the template files you're generating, and we will be editing them and deploying them._

Make sure to use the latest version of [Azure CLI](https://learn.microsoft.com/en-us/cli/azure/install-azure-cli).  

Get started by logging in to your subscription:
```
az login
```

## Get deployment files for the resources you created

_Note: For all of the commands below, make sure you use the values that you used when creating resources, or that you see from commands in your Azure subscription. They won't always match the values I've provided as exdamples below._  

View the deployment with Azure CLI:  

```
az deployment group list -g webapp-manual-rg
```

Scroll to the top & look for the `name` in the output.  
It will be something like: `CreateVm-canonical.0001-com-ubuntu-server-jammy-2`.  
Take a copy of the `name` value.  

_Note: The deployment name and template files wer actually generated by the Azure Portal when you created the VM. Deployments in the portal mostly just use templates._   

View all of the information about the deployment with the following command (using your values):  

```
az deployment group show -g webapp-manual-rg -n "CreateVm-canonical.0001-com-ubuntu-server-jammy" 
```

## Create a folder to save your template files

```
cd ~
mkdir projects
cd projects
mkdir azure-vm-webapp-iac
cd azure-vm-webapp-iac
```

## Export the ARM template that the Azure Portal created

```
az deployment group export -g webapp-manual-rg -n "CreateVm-canonical.0001-com-ubuntu-server-jammy" > stage2-deploy.json
```

## Save the parameters file that the Azure Portal created

```
az deployment group show -g webapp-manual-rg -n "CreateVm-canonical.0001-com-ubuntu-server-jammy" --query "properties.parameters" > stage2-params.json
```

# STAGE 2B - Update the NSG in the Template

At this point we need to update the NSG with the following:
1. (Optional) Lock down the inbound SSH to just your IP
2. Add the inbound HTTP allow rule

The template files we have downloaded above will only include the configuration that you specified when the VM was deployed (not the additional NSG changes we made after).  

_Note: You can generate an ARM template from the resources/configuration as they are right now, instead of exporting the original ARM template, by using the command below. This is included for your information only and isn't the approach we're taking in this mini project. This can help when working with ARM templates when you're not sure what values/properties/resources to include in your templates - as you can configure it first in the Portal and then see what the template files look like._   

`az group export -g webapp1-rg`  

## Locate the existing NSG rule in the parameters file

The Azure Portal created an ARM template and parameters file in a modular way, meaning it can easily be modified. For this reason you'll find the NSG rule(s) are not hard-coded in the ARM template as you might expect. Instead, the NSG rule is added as a parameter.  

These are some options for how you can edit the files:
- If using Azure CLI on your computer, your files are already local and can be edited there
- If using Cloud Shell, your files are "in the cloud" and you can:
    - Click on the 'Editor' button or type `code` to change to Classic Cloud Shell and edit there
    - Click on 'Manage files' and upload/download the files to/from your computer

**You will need [VS Code](https://code.visualstudio.com/download) installed (or be using Classic Cloud Shell) to perform the following commands. Alternatively just open the files in whatever editor you like.**   

Open the parameters file we created earlier: 

```
code stage2-params.json  
```

The parameter file should include a parameter for the NSG rules like this:

```
"networkSecurityGroupRules": {
    "type": "Array",
    "value": [
    {
        "name": "SSH",
        "properties": {
        "access": "Allow",
        "destinationAddressPrefix": "*",
        "destinationPortRange": "22",
        "direction": "Inbound",
        "priority": 300,
        "protocol": "TCP",
        "sourceAddressPrefix": "*",
        "sourcePortRange": "*"
        }
    }     
    ]
},

```

## Modify the parameter to include an HTTP rule

Modify the parameter to include the inbound HTTP rule we require. Once complete, it should look something like this:

```
"networkSecurityGroupRules": {
    "type": "Array",
    "value": [
    {
        "name": "SSH",
        "properties": {
        "access": "Allow",
        "destinationAddressPrefix": "*",
        "destinationPortRange": "22",
        "direction": "Inbound",
        "priority": 300,
        "protocol": "TCP",
        "sourceAddressPrefix": "*",
        "sourcePortRange": "*"
        }
    },
    {
        "name": "Web",
        "properties": {
        "access": "Allow",
        "destinationAddressPrefix": "*",
        "destinationPortRange": "80",
        "direction": "Inbound",
        "priority": 200,
        "protocol": "TCP",
        "sourceAddressPrefix": "*",
        "sourcePortRange": "*"
        }
    }      
    ]
},
```

Save and close the file.   

## (Optional) Modify the parameter for your IP

You can optionally modify the `"sourceAddressPrefix": "*",` of each rule to lock down to your IP address for added security. The `*` means anyone can connect via SSH or HTTP.   

If you're uncertain what this should look like, you can also optionally check out the built-in [MS Azure Resource Explorer](https://resources.azure.com) to browse through your existing configuration.  

## (Optional) Modify any settings you wish

At this stage you'll have template and parameter files that can be used to deploy the solution with all the same settings/resources as you did manually in stage 1.   

You can now modify any other parameters that you like, such as the VM size, name, username, etc. 

**Do not hardcode the password in any of the template files. We will capture it via input further below when deploying the solution.**  


# STAGE 2C - Deploy the Solution

We will now use Azure CLI to deploy the solution using the new template files.  
Make sure you're still logged in with Azure CLI or Cloud Shell.  
Make sure you're still in the project folder. 
Use the following commands if needed:

```
cd ~/projects/azure-vm-webapp-iac/
az login
```

## Create a new Resource Group

Create a new resource group, choosing your own location.  

```
az group create -n webapp-auto-rg -l australiaeast
```

You can find your location using this command

```
az account list-locations --output table
```

## Capture a password for your VM

It isn't great that we're entering in a password here manually (we'll fix later). To make it a little bit better for now, we will capture the password using a variable instead.  

Type in the command below and then enter your password for the VM when prompted.  

```
read -s -p "Enter an Admin Password: " adminPassword
```

## Deploy the solution using the template files

```
az deployment group create --resource-group webapp-auto-rg --template-file stage2-deploy.json --parameters @stage2-params.json adminPassword=$adminPassword
```


# STAGE 2D - Test

*Note: Wait until the VM is running before continuing.*  

## Confirm SSH is working

Open the VM blade https://portal.azure.com/#browse/Microsoft.Compute%2FVirtualMachines.    
Click the `webvm1` VM (in webapp-auto-rg).  
Browse to `Networking` > `Network settings`.    
Check that the NSG has been created/configured with HTTP and SSH rules.  
Copy the public IP address from here or the overview section.  
Test that you can login with SSH using the username & password you specified.  

```
ssh username@vm-public-ip-address
```

## Confirm there is no webapp via HTTP

Open the VM blade https://portal.azure.com/#browse/Microsoft.Compute%2FVirtualMachines.    
Click the `webvm1` VM (in webapp-auto-rg).  
Copy the public IP.  
Open the IP in a new tab (e.g. http://publicIP).  
No website will load (we will configure it in stage 3).  

# STAGE 2 - FINISH  

As long as the VM is up and running and you can connect, that is all you need to do. We will not manually configure the web app this time, instead we will automate the process in the next stage.  

## Delete old resources

You can now delete the old resources that we manually configured.  

Go to the Azure Portal home page > Resource Groups https://portal.azure.com/#browse/resourcegroups.  
Click the Resource Group we created in stage 1, `webapp-manual-rg`.      
Click the `Delete resource group` button.  
Copy and paste the resource group name.  
Tick the `force delete` checkbox.     
Click on the `Delete` button.     
Confirm to `delete` the resources.  

## Status

This solution has several limitations which you will address within the upcoming stages of this mini project:

- ~~The resources were built manually (VM, VNet, NSG, etc)~~
- Docker had to be installed/configured manually
- The Web App was manually built and deployed
- The admin credentials for the VM are typed manually

You can now move onto STAGE3.  

## Completed Resources

If for any reason you'd like to refer to the files that I use in the solution videos, you can find them below:

- [stage2-deploy.json](/azure-vm-webapp-iac/01_LABSETUP/stage2-deploy.json)
- [stage2-params.json](/azure-vm-webapp-iac/01_LABSETUP/stage2-params.json)

__Note: Your files will like very similar, but not identical, to these files.__  