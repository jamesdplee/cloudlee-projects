# Mini Project - Create a Container Web Server with an ARM Template

In this final stage of this mini project you are going to remove all resources that you deployed/created within your Azure subscription.   

# STAGE 5A - Delete the Resource Group

You can now delete all resources that we created.  

Go to the Azure Portal home page > Resource Groups https://portal.azure.com/#browse/resourcegroups.  
Click the Resource Group we created in stage 1, `webapp-auto-rg`.       
Click the `Delete resource group` button.  
Copy and paste the resource group name.  
Tick the `force delete` checkbox.     
Click on the `Delete` button.     
Confirm to `delete` the resources.  

_Note: You should have already deleted the manually deployed resources in `webapp-manual-rg` earlier. If you did not, be sure to do that now._  

# STAGE 5B - Delete the Key Vault

Make sure to delete the Key Vault resource group too if you did not deploy it to the same resource group above.  

# (OPTIONAL) STAGE 5C - Delete Cloud Shell Storage

If you deployed a new storage account to a new resource group as part of optionally using Cloud Shell, you may choose to delete that now. Storage accounts do have a much lower cost than resources like VMs, so if you plan to use Cloud Shell again, you may prefer to not delete it.  

# STAGE 5 - FINISH  

That's it. You're all done! Congratulations & I hope you enjoyed it.  

In other projects we'll look at other improvements you can make like using Azure Container Instances, load balancing, and more.  

_Remember:_ Labbing is fun and a great way to learn, but you definitely don't want any surprise bills. I always encourage those with labs to have a budget setup in their subscription to warn of any spending surprises (you can learn about Azure Budgets in my free course [here](https://learn.cloudlee.io/courses/getting-started-with-azure/lectures/45020709)). And consider setting yourself an alarm clock reminder for just before sleep if you're doing lots of labs to delete/shutdown resources.  


