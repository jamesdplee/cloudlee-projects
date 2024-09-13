# Optional Instructions

_Note: These instructions are provided in case you'd like to take a different approach to the step-by-step instructions that we follow for the mini project. It is a collection of instructions for different tasks you may wish to complete._  

# Configure Cloud Shell

## Create a Storage Account

Navigate to the Create a Resource hub in the Azure Portal https://portal.azure.com/#create/hub.   
Search for `storage account` (it may already be onscreen).  
Click `Create`.  
Configure the storage account with the following settings.  
Click `Next` to move through each section.  
If the setting is not specified below, leave it as-is.  

Use the following values for your storage account:
- **Subscription**: Choose your own subscription
- **Resource group**: cloudshellstore1-rg
- **Storage account name**: Enter any name you like, this MUST be unique
- **Region**: Pick a region close to you
- **Performance**: Standard
- **Redundancy**: GRS

Leave the other settings as-is.  

Click `Review + create`.  
Click `Create`.  

## Setup Cloud Shell

Click the `Cloud Shell` icon at the top right of the Azure Portal.  
Choose `Bash` if the Cloud Shell setup prompts you.  
Choose `Mount storage account` if asked whether to use a storage account or not.  
Select the `subscription` that you configured your storage account in.  
Click `Apply`.  
Choose `Select existing storage account`.  
Click `Next`.  
Choose the `Resource group` that you configured your storage account in.  
Select your `Storage account name` that you just created.  
Click on the `Create a file share` option.  
Type the name `cshell` and click `OK`.  
Click on `Select`.  

Cloud Shell should now launch with permanent storage.  


# Use VS Code in Cloud Shell

A lightweight version of VS Code is included in Cloud Shell. However at this time it is only available in the older (classic) version of Cloud Shell.

## Launching Classic Cloud Shell

Open Cloud Shell using the icon at the top, or https://shell.azure.com.  
Click `Settings` > `Go to classic version`, OR  
Click on the `Editor` and click `Confirm` to go to the classic version.  

## Launching VS Code

Within Cloud Shell, you can launch code by typing the following: 

```
code
```

Optionally, you can edit an existing file by typing:  
```
code filename.ext
```


