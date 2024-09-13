# Mini Project - Create a Container Web Server with an ARM Template

_**DIAGRAM**: Stage 1 diagram coming soon._  
_**VIDEO**: Stage 1 video coming soon._  

The goals for stage 1 of this mini project are: 
- Manually create a Linux VM for the web app to run on 
- Install Docker on the VM
- Build and run the Aus-E-Mart web app container image

This is our manual starting point, and we'll evolve the solution to use more automation, security and infrastructure-as-code in the upcoming  project stages.  

_Note: If you would like to complete this stage of the mini project by yourself without following the **step-by-step** instructions, you can do so by completing the above goals with any settings of your choice. We will use the web app / dockerfile provided here: https://github.com/jamesdplee/ausemartweb.git._   

See the following sections for step-by-step instructions.  

# STAGE 1A - Login to your Azure Subscription  

Login to your own Azure Subscription with a user that has `Owner` permissions.  

Click [HERE](https://portal.azure.com) to login to the Azure Portal.  

_Note: In other projects we'll use a click-to-deploy template to setup the project resources for us. In this project we'll perform these steps manually ourselves instead, to learn how to create ARM templates._  

# STAGE 1B - Create a Virtual Machine for the Web App

Navigate to the Create a Resource hub in the Azure Portal https://portal.azure.com/#create/hub.   
Search for `virtual machine` (it may already be onscreen).  
Click `Create`.  
Configure the virtual machine with the following settings.  
Click `Next` to move through each section.  
If the setting is not specified below, leave it as-is.  

_Note: We will have the opportunity to change these settings later when we convert this solution into an ARM template._  

Use the following values for your virtual machine:
- **Basics**
    - **Subscription**: Choose your own subscription
    - **Resource group**: webapp-manual-rg (create new)
    - **Virtual machine name**: webvm1
    - **Region**: Pick a region close to you
    - **Availability**: Availability Zone, or None
    - **Image**: Ubuntu Server 24.04 LTS
    - **Size**: Standard_B1s (or any low price VM size)
    - **SSH / password**: We must use a password in this project (SSH is normally the preferred approach)
    - **Administrator account**: Enter a username and password of your choice
    - **Inbound port rules**: Allow SSH (22) - _this will create an NSG for you_
- **Disks**: Leave as-is
- **Networking**
    - **Virtual network**: Leave the automatic network as-is (e.g. webvm1-vnet)
    - **Public IP**: Create a new (or use the default) public IP
    - **NIC security**: Basic
    - **Public inbound ports**: Allow selected: SSH (22)
- **Management**: Leave as-is
- **Monitoring**: Leave as-is
- **Advanced**: Leave as-is
- **Tags**: Leave empty
 
Click `Review + create`.  
Click `Create`.  
 
_Note: You should wait for deployment to complete before continuing to the next step._  

# STAGE 1C - Configure the Network Security Group

At this point we have created many resources, including an NSG. For the NSG there are two changes we should make:
1. (Optional) All inbound rules should be locked down to just your IP
2. Add a rule to the NSG to allow inbound HTTP for the web app

Go to the Azure Portal home page > Resource Groups https://portal.azure.com/#browse/resourcegroups.  
Click the Resource Group we created above, `webapp-manual-rg`.   
Click the Network Security Group we created above, `webvm1-nsg`.  
Click `Settings` > `Inbound security rules`.   

## Restrict SSH inbound to your IP

Modify the existing SSH rule to lock it down to just your IP:   

Click on the SSH rule by clicking `SSH` in the name column.  
Click on the `Source` dropdown (which will currently say `Any`).  
Choose `My IP address`.  
Click `Save`.  

This should add your IP address as the only allowed source for the SSH rule.  
This means SSH is only accepted from your Internet connection.   

## Configure an inbound HTTP allow rule

You should still be in the `Inbound security rules` section.  
Click `+ Add` at the top.  

Add the following security rule:
- **Source**: Choose 'My IP Address' (or, less secure, 'Any')
- **Source port ranges**: *
- **Destination**: Any
- **Service**: HTTP this will autocomplete other settings
- **Action**: Allow
- **Priority**: 110 
- **Name**: inbound-allow-http

Click on `Add`.  

# STAGE 1D - Connect to the VM and Setup the Web App

Open the VM blade https://portal.azure.com/#browse/Microsoft.Compute%2FVirtualMachines.    
Click the `webvm1` VM.  
Copy the `Public IP address` into your clipboard (**DON'T CLICK THE OPEN LINK - just click the copy button**).  

SSH to the VM using one of the following methods.  

- For macOS, Linux, and Linux on Windows with WSL, PowerShell on Windows  
    `ssh username@vm-public-ip-address`  
- Use Cloud Shell in the Azure Portal   
    `ssh username@vm-public-ip-address`  
    Note this your inbound SSH NSG rule have the source set to `Any`
- For Windows you can optionally install [PuTTY](http://www.chiark.greenend.org.uk/~sgtatham/putty/download.html)  

You will need to enter `Yes` when prompted to accept the fingerprint when logging in through SSH.  
Once you have logged in to the VM with SSH, you can perform the following tasks.  


## Install updates

```
sudo apt-get update 
```

## Install docker

```
sudo apt-get install -y docker.io 
```

## Build & run the Aus-E-Mart web container

```
git clone https://github.com/jamesdplee/ausemartweb.git 
cd ausemartweb 
sudo docker build -t ausemartweb . 
sudo docker run -d -p 80:80 ausemartweb 
```

## Test the web app is running

Open the VM blade https://portal.azure.com/#browse/Microsoft.Compute%2FVirtualMachines.     
Click the `webvm1` VM.  
Copy the `Public IP address` into your clipboard (**DON'T CLICK THE OPEN LINK - just click the copy button**).  
Open that IP in a new tab (e.g. http://publicip).  
You should see the Aus-E-Mart homepage.    

Congratulations - you've manually setup a containerized web app to run on a VM.  

# STAGE 1 - FINISH  

You can shutdown the VM to save on costs, and we will delete it after completing the next stage.  

This solution has several limitations which you will address within the upcoming stages of this mini project:

- The resources were built manually (VM, VNet, NSG, etc)
- The admin credentials for the VM are typed manually
- Docker had to be installed/configured manually
- The Web App was manually built and deployed

And there are other items improvements that we will make in different projects:

- Single point of failure with no load balancing
- Difficult to manage SSH inbound allow rules
- Web application has no protection against common vulnerabilities/exploits
- Docker must be installed/managed/configured by you (compared to PaaS like ACI/AKS/ACA)

_Note: For SSH it is best practice to use SSH keys. However, in this demo we use a username + password because some organizations still require this and it also helps later in the project to demonstrate how we can securely store such information using Key Vault._  

You can now move onto STAGE2.  

