# Lab2 - Containerizing  applications on Azure

## Lab Goals

The goal of this lab is to take an existing web application and move it from a standard deployment using on premises web farms to a container infrastructure on Azure.  This will enable a much more saleable environment with a lot less management.   In this lab you will:

- Create a container registry in Azure
- Build an application for containers leveraging Azure compute
- Deploy the containers to scalable web instances.



## Setup Environment 

You will need a few things in your environment setup for this lab.

- Source code for the Inventory and Product services.  
- Source code for the front end web site.
- Azure Container Repository


### Setup 1 - Pull down the source code

The source code for all three projects are in this repo.  You will first pull it all down locally in our Azure Bash Shell.

1. Launch a new Azure Command Shell.  You can either:

   1. Press the shell icon in the Azure Portal, as in the setup for the Cosmos DB
   2. Open a new browser tab to:  http://shell.azure.com for a full screen experience
2. Pull down the source code locally.  Run the following git command 

   ```bash
   git clone https://github.com/chadgms/2019AzureMigrateYourApps
   ```
3. If you do an ls command you should see the repo on your local shell
   ![ShowCodeRepo](../images/ShowCodeRepo.png)


### Setup 2 - Create an Azure Container Registry

1. In the Azure portal click on the new resource button
2. Type 'Container Registry' in the search bar - press enter
3. Select `Create`
4. Fill in properties
   1. Registry name: (prefix)acr
   2. Resource group: 'Lab-2-xxxxx'
   2. Location: default
   3. Admin User: Enable
   4. SKU: Standard
5. Creation should only take a few minutes, but you can review key features while you are waiting here: https://docs.microsoft.com/en-us/azure/container-registry/container-registry-intro


## Activity 1 - Build your services for docker containers

Now you'll get into the really exciting stuff!  There's an existing code base for two services and web front end.  In this activity, you will compile the code, put them in a docker container and put the container image into the container registry.  Typically you would need Docker installed and configured;however, Azure Container Registry Service can run the build and containerization. You only need to point it to the source code.

### Build the code and deploy to ACR

1. In your Azure Cloud Shell, set an environment variable to the name of your Azure Container Registry (ACR).  It is the name you put in the Registry Name property when you created the registry -  (prefix)acr.   (Do NOT use the full .azurecr.io UNC, only the name)

   ```bash
   MYRG='Lab-2-xxxxx'
   MYACR='(prefix)acr'
   MYID='(prefix)' - the prefix you have been using during this workshop
   ```
2. Run the following command to build the code and push the product service image to the ACR.

   ```bash
   az acr build -t product-service:latest -r $MYACR ./2019AzureMigrateYourApps/Lab2-ContainerizingApplications/src/product-service
   ```
3. When finished you can confirm that the container was deployed to ACR in the portal.
   1. Click on your resource groups in the portal left navigation bar
   2. Click on your resource group
   3. Locate your ACR resource type 'Container Register' - Click it
   4. Click 'Repositories' in the left navigation
   5. You should see 'product-service' in the repositories list ![DeployContainer](../images/DeployContainer.png)
4. Build the inventory service and frontend containers.****

   ```
   az acr build -t inventory-service:latest -r $MYACR ./2019AzureMigrateYourApps/Lab2-ContainerizingApplications/src/inventory-service/InventoryService.Api
   ```

   ```
   az acr build -t frontend-service:latest -r $MYACR ./2019AzureMigrateYourApps/Lab2-ContainerizingApplications/src/frontend
   ```
5. Verify it exists in the ACR same as before


### Create Web Apps

Now that you have compiled code, you need to deploy them to a compute platform.  In this lab, you are going to use Azure Web App for Containers, but there are many ways to run containers in Azure.  Your instructor will explore other options.  See reference links below for more information.

#### Product Service App

1. Press the create resource button in the Azure portal
2. Search for 'Web app for containers' and press enter
3. Press `Create`
4. Fill out parameters as follows
   1. Basics Tab
      1. **Resource Group:** use Lab-2-xxxxx
      2. **Name:** (prefix)product
      3. **Publish:** Docker Container
      4. **OS:** Linux
      5. **Region:** Use 'East US'
      6. **Service Plan:** Press Create New
         1. Rename to:  (prefix)serviceplan
      7. **Pricing:** Change Size - > Dev/Test -> B1
   2. From the Docker tab on the top menu:
      1. **Options:** Single Container
      2. **Image Source:** Azure Container Registry
      3. **Registry:** Pick your ACR
      4. **Image:** Select the product-service image
      5. **<u>Tag</u>:** latest
   3. Select `Review + create`
   4. Select `Create`

#### Inventory Service App

1. Select `Create a resource` button in the Azure portal
2. Search for 'Web app for containers' and press enter
3. Select `Create`
4. Fill out parameters as follows
   1. Basics Tab
      1. **Resource Group:** use Lab-2-xxxxx
      2. **Name:** (prefix)inventory
      3. **Publish:** Docker Image
      4. **OS:** Linux
      5. **Region:** <u>Use the same region as the Product Service</u>
      6. **Service Plan:** Pick the same service plan that you created for the Product Service.  You do not need to create more than one service plan.  All the Web Apps can share the compute.
   2. From the Docker tab on the top menu:
      1. **Options:** Single Container
      2. **Image Source:** Azure Container Registry
      3. **Registry:** Pick your ACR
      4. **Image:** Select the inventory-service image
      5. **Tag:** latest
   3. Select `Review + create`
   4. Press `Create`
#### Front End App

1. Press the create resource button in the Azure portal
2. Search for 'Web app for containers' and press enter
3. Press Create
4. Fill out parameters as follows
   1. Basics Tab
      1. **Resource Group:** use Lab-2-xxxxx
      2. **Name:** (prefix)frontend
      3. **Publish:** Docker Image
      4. **OS:** Linux
      5. **Region:** <u>Use the same region as the Product Service</u>
      6. **Service Plan:** Pick the same service plan that you created for the Product Service.  You do not need to create more than one service plan.  All the Web Apps can share the compute.
   2. Docker Tab:
      1. **Options:** Single Container
      2. **Image Source:** Azure Container Registry
      3. **Registry:** Pick your ACR
      4. **Image:** Select the frontend-service image
      5. **Tag:** latest
   3. Select `Review + create`
   4. Select `Create`

### Service Configuration 

You now have web apps created for all the resources.  The last thing you need to do is configure application environment variables like connections strings.  When the services start up they can read the environment variables which enables you to make configurations at runtime.  

#### Product Service

The product service uses the NOSQL data that was in the on-premise MongoDB.  You successfully migrated that data to Cosmos DB, and now you need to configure the Product Service to use the migrated database.

> **Note:** If you were unable to finish Lab 1 and do not have a CosmosDB with data in it.  See the appendix at the end of this lab for a connection string to a shared instance you can use.


##### Get the Cosmos DB connection string

1. Select `Resource groups` in the Azure portal, select 'Lab-1-xxxxx' resource group
2. Select your Cosmos DB Account
3. Select `Connection String` in the left-pane menu
4. Copy the Primary Connection String (NOT the primary password)

##### Set the Web App Properties

1. Select `Resource groups` in the Azure portal, select 'Lab-1-xxxxx' resource group
2. Select your product service resource of type 'App Service'
3. Select `Configuration` on the left-pane menu
4. Here you will see some default application setting, and you will add a few more.
5. Select `+ New application setting` to add each of these NAME/VALUE pairs
   1. **Name**: COLLECTION_NAME   **Value**: inventory - > Press OK
   2. **Name**: DB_CONNECTION_STRING  **Value**:  (paste in the Cosmos DB connection String)
      1. **IMPORTANT:** You need to add the database name 'tailwind' to the connection string.  You will see the server address:port and the the /?ssl flag like this: 
   
         ```
         ...azure.com:10255/?ssl...
         ```
   
         You add the tailwind database name between them like this: 
   
         ```
         ..azure.com:10255/tailwind?ssl...
         ```
         
         ![cosmosconnectstring](../images/cosmosconnectstring.png)
   
6. You should have two app settings something like this ![productappsettings](../images/productappsettings.png)
7. Select `Save` from top menu

**Note**: Connection strings can also be resolved from [Key Vault](https://docs.microsoft.com/en-us/azure/key-vault/) using [Key Vault references](https://docs.microsoft.com/en-us/azure/app-service/app-service-key-vault-references).  You are not using Key Vault for this lab, but these are good references to review.


#### Inventory Service

The inventory service needs to be pointed to the SQL Database that now lives in Azure SQL Azure

##### Get the Azure SQL Connection String

1. Select `Resource groups` in the Azure portal, select 'Lab-1-xxxxx' resource group
2. Select the tailwind database (resource type: SQL database)
3. Select `Connection strings` from the left-pane menu
4. Copy the ADO.NET connection string

##### Set the Web App Properties

1. Select `Resource groups` in the Azure portal, select 'Lab-2-xxxxx' resource group
2. Select on your inventory service resource of type 'App Service'
3. Select `Configuration` on the left-pane menu
4. Here you will add a Connection String 
5. Select `+ New connection string`
   1. Name: 'InventoryContext'
   2. Value:  (paste in the SQL connection String>)
   3. Update the SQL Connection string:
      1. User ID='migrateadmin'
      2. Password='AzureMigrateTraining2019#' ![SQLConnectionString](../images/SQLConnectionString.png)
   4. Type: SQLAzure
6. Press OK
7. Press Save


#### Front End Web Site

The last thing you need to do is to tell our front end web site the URL's to our web services for product and inventory.  

##### Get base URL's

You can get the base URL's for inventory and product services by clicking on their overview page and looking at the URL property on the right hand side.

1. Select `Resource groups` in the Azure portal, select 'Lab-2-xxxxx' resource group
2. Select your inventory or product service resource of type 'App Service'
3. Take note / copy the URL 

##### Set Front End Web App Properties

You will now set the Front End application settings using the Azure Cli:  Copy the below code and replace the base URL placehoders with the URL's of your services.

> Note: If you closed the command window you may have lost the values for the initial environment variables you setup, then you need to redefine MYRG and MYID again.

```
az webapp config appsettings set --resource-group $MYRG --name "$MYID"frontend --settings INVENTORY_SERVICE_BASE_URL='<your inventory base url>' PRODUCT_SERVICE_BASE_URL='<your product service base url>'
```

#### Run the App!

That is is it!  You are done migrating the data and deploying a modern application through containers. The last thing to do is to run the app and make sure it works! (Note: the web page will render pretty quickly, but the data may take 2-3 minutes to show up.  If it takes longer than 3-4 minutes, then connect with one of the workshop leaders)

1. Select `Resource groups` in the Azure portal, select 'Lab-2-xxxxx' resource group
2. Select your front end service resource of type 'App Service'
3. From the `Overview` menu item, select `URL` in the right properties pane to launch your application


***Your web app should now be live!  Congratulations!***

![finalwebsite](../images/finalwebsite.png)





## Appendix 

### Shared Connection Strings

If you could not complete Lab 1 and do not have either an Azure SQL Instance or Cosmos DB instance  completed with data then you can use the connection strings below for your Inventory and Product services.

Your proctor will supply the connection strings.
```



## Learn More/Resources

- 
