# Mini Project - Create a Container Web Server with an ARM Template

_**DIAGRAM**: Stage 4 diagram coming soon._  
_**VIDEO**: Stage 4 video coming soon._  

In stage 4 you are going to modify the ARM template files you created earlier to automate the configuration of docker and the build/deploy of the web app container, so this doesn't need to be performed manually.  

The steps you will complete are as follows:
- Create a new copy of your template files (stage4-)
- Add the VM Extension resource to your template
- Add the script content as a parameter

_Note: If you would like to complete this stage of the mini project by yourself without following the **step-by-step** instructions, you can do so by completing the above goals. You will need to use the following:_   

- Add the `Microsoft.Compute/virtualMachines/extensions` resource to your template
- Configure the script using one of the below approaches:
    - Use `"script": "[base64(parameters('scriptContent'))]"` and include the commands performed from stage 1 as a parameter
    - OR use `commandToExecute` + `fileUris` with the script stored online in Azure Storage or Github

You can see more information about possible options in this [Microsoft article](https://learn.microsoft.com/en-us/azure/virtual-machines/extensions/custom-script-linux).  


# STAGE 4A - Setup Template Files

Start by making a copy of the template files.  
You can choose to do this with a GUI, or with the CLI commands below:  

```
cd ~/projects/azure-vm-webapp-iac/
cp stage3-deploy.json stage4-deploy.json 
cp stage3-params.json stage4-params.json 
```

# STAGE 4B - Add Custom Script Extension

Using your editor of choice, you will need to modify the deploy and parameter files. Use the code below, ensuring you match it to any changes you have made to parameters, etc.  

## Update the template file

Open the `stage4-deploy.json` file you just made.  
Add the following new resource to your template:  

```
    {
        "type": "Microsoft.Compute/virtualMachines/extensions",
        "apiVersion": "2021-07-01",
        "name": "[concat(parameters('virtualMachineName'), '/customScriptExtension')]",
        "location": "[resourceGroup().location]",
        "dependsOn": [
          "[resourceId('Microsoft.Compute/virtualMachines', parameters('virtualMachineName'))]"
        ],
        "properties": {
          "publisher": "Microsoft.Azure.Extensions",
          "type": "CustomScript",
          "typeHandlerVersion": "2.1",
          "autoUpgradeMinorVersion": true,
          "settings": {
            "script": "[base64(parameters('scriptContent'))]"
          }
        }
    }
```

Make sure you add a `,` after the previous resource.   
For example if this is the last resource in your ARM template, the previous resource will end with `},`.  
If you add this before other resources, you'll instead need to add the comma to the code above.  
Refer to the complete file listed at the end to see the complete version.  

You will also need to add the parameter to the deploy template, as below:  

```
    "scriptContent": {
        "type": "string"
    }   
```        

Once again be sure to add the `,` as required.  

## Update the parameters file

Now we need the parameter value in the parameter file.   

Open the `stage4-params.json` file.   
Add the following:  

```
    "scriptContent": {
        "value": "#!/bin/bash\nsudo apt-get update\nsudo apt-get install -y docker.io\ngit clone https://github.com/jamesdplee/ausemartweb.git\ncd ausemartweb\nsudo docker build -t ausemartweb .\nsudo docker run -d -p 80:80 ausemartweb"
    } 
```    

This means that when we deploy the ARM template using our new files, a Custom Script Extension will be created that run the commands we ran manually in stage 1. Usually you would create a file to store your script and retrieve it via HTTP, but you can also pass script commands like this.   

The use of `\n` above is the equivalent of adding a new line.  

# STAGE 4C - Deploy the Update

Let's now use our updated files to update our deployment.  

## Prepare with Azure CLI

Login to Azure if you haven't already (`az login`).  
Make sure you're in the correct folder (eg. `cd ~/projects/azure-vm-webapp-iac`).  
If you closed your computer/CLI, recapture the password (`read -s -p "Enter an Admin Password: " adminPassword`).  

## Deploy the update

```
az deployment group create --resource-group webapp-auto-rg --template-file stage3-deploy.json --parameters @stage3-params.json adminPassword=$adminPassword
```

# STAGE 4D - Test

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

# STAGE 4 - FINISH  

This solution has several limitations which you will address within the upcoming stages of this mini project:

- ~~The resources were built manually (VM, VNet, NSG, etc)~~
- ~~The admin credentials for the VM are typed manually~~
- ~~Docker had to be installed/configured manually~~
- ~~The Web App was manually built and deployed~~

You can now move onto STAGE4.  

## Completed Resources

If for any reason you'd like to refer to the files that I use in the solution videos, you can find them below:  

- [stage4-deploy.json](/azure-vm-webapp-iac/01_LABSETUP/stage4-deploy.json)  
- [stage4-params.json](/azure-vm-webapp-iac/01_LABSETUP/stage4-params.json)  