# Lab 1 - Migrate Your On-Premises Data Servers To Azure

## Lab Goals

The goals of this lab is to get you familiar with the Azure environment, portal and command line.  Everything you can do in the Azure Portal can also be done through a command line and scripting.  We will expose you to both methods of creating resources in Azure.  In this lab you will:

- Use the Azure Portal to create resources
- Use the Azure Cloud Shell to create resources
- Migrate a Linux Mongo DB to Azure Cosmos DB 
- Migrate a SQL Server 2017 DB to Azure SQL Database

## Setup Environment
You will need a few things in your environment setup for this lab.

- A SQL Server VM that will act as our on-premise SQL instance that you will migrate to Azure SQL DB
  -	A SQL Server VM has been pre-provisioned for this exercise
- An Azure SQL Database Instance.  This is the SQL PaaS service you will migrate the on-premise server to.
  - You will create this as part of the lab
- A Mongo DB that you will migrate data from to Cosmos DB
  - A public Mongo DB will be made available to you to access remotely.
- An Azure Cosmos DB instance
  - You will create this as part of the lab
- The Microsoft Data Migration Assistant 
  - You will install this on the pre-provisioned SQL VM
- An Azure Database Migration Service
  - An Azure Database Migration Service has been pre-provisioned for this exercise 


### Setup 0 - Create a unique prefix

In many cases you need to create a resource that has a unique name.  The easiest way to do this is to create a prefix that you can append to the front of the standard resource names.    As an exmple, Bill Smith needs a unique prefix so he decided to use his name and the first three digits of his phone number: his prefix is 'BS336'.  Any resources that need to be unique he can now put this in front of the standard name and it should be unique.  

One thing to consider is that some resources have a limit to how many characters are in a name.  So, keeping your prefix to under 6 characters.  Come up with a prefix you can use for all the labs.

**IMPORTANT: Whenever you see (prefix) in the labs, preplace that with the prefix you come up with.**


### Setup 1 - Verify On-premises SQL VM

1. Login to the Azure Portal http://portal.azure.com using your assigned credentials
2. Select `Virtual Machines` from the left-pane menu in the Azure Portal (you might need to expand the left pane)
3. Verify the presence of the VM named 'OnPremSQL' and its Status is 'running'
4. This is the SQL VM you will use as the source database for the migration


### Setup 2 - Verify Azure Database Migration Service

1. From the Azure Portal, select `Resource Group` from the left-pane menu then select the resource group named 'Lab-1-xxxxx'
2. Verify the presence of the Azure Database Migration Service
3. This migration service instance will be used when we migrate the on-premises SQL database to Azure

### Setup 3 - Create an Azure Cosmos DB account with Cloud Shell

Up until now you have used the Azure Web Portal to create needed resources.  In this exercise, you will create an Azure Cosmos DB account using the Azure Command Line Interface (CLI) and the Azure Cloud Shell.

The Azure Cloud Shell is a command shell that runs in your browser; it creates compute in the back end and a storage account.  You can run either PowerShell or Bash; for these exercises you will use Bash

1. Open the Azure portal
2. Select the `>_` button in the toolbar ![CreateBash](../images/CreateBash.png)
3. Select `Bash` on the Welcome Screen ![CreateBash1](../images/CreateBash1.png)
4. Select `Show advanced settings` ![CreateBash2](../images/CreateBash2.png)
5. Create a new storage account and file share in your resource group.  Use your (prefix) in the name ![CreateBash3](../images/CreateBash3.png)
   1. Use your existing resource group named 'Lab-1-xxxxx'
6. Select `Create Storage`
7. Wait for the shell to start
8. Make sure you are in  `Bash` from the dropdown of the Cloud Shell window.  ![CreateBash4](../images/CreateBash4.png)
9. Create 3 Bash variables by typing the following in the shell and press enter:
  * RESOURCE_GROUP_COSMOS - set to your resource group named 'Lab-1-xxxxx'  
  * LOCATION_COSMOS - set to 'eastus'
  * ACCOUNT_NAME_COSMOS - This MUST be unique.  Create a prefix that would be unique to you using your initials and a few digits

```language-bash
RESOURCE_GROUP_COSMOS='<your resource group named 'Lab-1-xxxxx'>'
LOCATION_COSMOS='eastus'
ACCOUNT_NAME_COSMOS='(prefix)migrationcosmos'
```
5. Copy the below command and execute it (you can paste in command window with a  right click).  This will create the Azure Cosmos DB Account using the Azure CLI (az commands)

```language-bash
az cosmosdb create --resource-group $RESOURCE_GROUP_COSMOS --name $ACCOUNT_NAME_COSMOS --kind MongoDB --locations regionName=$LOCATION_COSMOS
```

This will take several minutes to complete. When it is finished (you'll see JSON indicating it's done), go into the portal and select `Resource Groups` from the left-pane menu.  Select the 'Lab-1-xxxx' resource group to see your Cosmos DB account

While you wait for your Cosmos DB instance to spin up you can move on to the creation of the Azure SQL Database Instance


### Setup 4 - Create the Azure SQL Database Instance

You will now create a SQL Server PaaS instance - this is the target database for your SQL migration

1. Select the `Create a resource` button in the top-left of the Azure portal

2. Type 'SQL Database' in the search box and press enter

3. Select SQL Database

   ![SQLDatabaseCard](../images/SQLDatabaseCard.png)

4. Select `Create`

5. From the Basics tab in the menu

   1. Resource Group: Set to the 'Lab-1-xxxxx' resource group
   2. Database Name: (prefix)SQLDB
   3. Server - Select `Create New` on the Database detail section
      1. Server Name: (prefix)sqlserver
      2. Server Admin: 'migrateadmin'
      3. Password: 'AzureMigrateTraining2019#' 
      4. Location: Use 'East US'
      5. Select `OK` on the New Server pane
   4. Select the `Networking` tab on the top menu
      1. Select `Public endpoint` on the Connectivity method radio button
      2. Select `Yes` for both Firewall options (allow Azure services and add current IP)
   4. Select `Review + Create`
   5. Select `Create` once validation is complete

This will take a couple of minutes to complete.  Once finished, all of your setup is completed for Lab Exercises!


### Exercise 1  -  Assess DB Migration Using the DB Migration Tool Migrate Using Azure Database Migration 

The inventory service is hosted on a SQL server and served by an ASP.NET core website. The Inventory service determines the quantity of a unit that's currently in stock.  In this lab we will migrate the on premises SQL Server to an instance of SQL Azure DB.

We will be using the [Database Migratrion Tool](https://www.microsoft.com/en-us/download/details.aspx?id=53595)

#### Finish On-Prem Configuration

**This step can be skipped *if* you are using the VM and database we provide for you, noted above.  If you want to learn the steps to restore a database to an Azure VM you may do these steps.**

By now the SQL Server VM you created should be finished provisioning.  We need to do a couple of extra steps to get it ready to migrate. 

1. On the left hand side of the Azure portal click on the resource group icon
   ![RGIcon](../images/RGIcon.png)
2. Click on your Resource Group
   1. You should now see all the resources we created in the exercises above
3. Click on your SQL On Prem Virtual Machine you created
4. Click on Connect->download the RDP file and open that to RDP to the VM, login with the migrateadmin user we created in Setup 1. 
5. Update IE Security - The local server manager should launch on first login.
   1. Press Local Server on the left side 
   2. Press the IE Enhanced Security Configuration  on the right ![IE-EnhancedSec1](../images/IE-EnhancedSec1.png)
   3. Set that off for Administrator 
       ![IE-EnhancedSec2](../images/IE-EnhancedSec2.png)
   4. Close the Server Manager
6. Download and restore the database.  The inventory database is stored in the repository as a SQL .bakpac file needs to be restored.
   1. Download the TailwindInventory.bacpac backup file from the setupfiles directory of this Github Repo. https://github.com/chadgms/2019AzureMigrateYourApps/blob/master/setupfiles/TailwindInventory.bacpac
      ![SQLBackupDownload](../images/SQLBackupDownload.png)
   2. Click the Windows start menu and type 'SQL Server Management'
   3. Launch the SQL Server Management Studio and connect to the local SQL instance.
   4. Right click on the Database folder and select 'Import Data-tier Application'
   ![ImportDAC](../images/ImportDAC.png)
   
7. Follow the wizard to import the backup file you downloaded.
8. When complete - Right click on the database folder and select 'refresh'
9. You should now see the TailwindInventory DB installed.

#### Download Microsoft DMA to your local laptop

10. Open a web brower and either search for 'Microsoft Database Migration Assistant' or download from:
    1. https://www.microsoft.com/en-us/download/details.aspx?id=53595
11. Install the Data Migration Assistant

We are now all set to migrate our SQL Database.  We have a restored copy of the data on either this local server (if you did the previous section's steps) or the shared SQL Server, and we have the migration assistant ready to help us migrate the data to an Azure SQL Instance.

#### Assessment
First you need to do an assessment.  The tool will check the local db for compatibility issues.

**If using the shared lab SQL Server, use the following connection string information:**

* Server:  40.71.92.209
* SQL Login:  dbadmin
* SQL Password:  Password01!!
* Database:  TailwindInventory

1. Open the Data Migration Assistant from the desktop icon
2. Create a new project

- Project Type: `Assessment`
- Project Name: `tailwind`
- Source server type: `SQL server`
- Target server type: `Azure SQL Database`

1. Click `Create`
2. Check `Check database compatibility`
3. Check `Check feature parity`
4. Click `Next`
5. Enter the 'localhost' for server name and Windows authentication
6. UN-Check the Encrypt Connection Box
7. Select the `TailwindInventory` database, click `Add`
8. Click `Start Assessment`
9. You will see a report on compatibility issues.  There should only be one for this DB because Service Broker is turned on.  In a real situation you would mitigate the issues.  For this lab you do not have to worry about as you know service broker is not actually used.  In your real situation you would now have a list of possible incompatibilities that would have to be addressed. 

#### Schema Migration 

Now that you know our database can be migrated you will use the Migration tool to migrate JUST the schema information.  You will use the more robust Azure Database Migration Service for the data.

1. In the Data Migration Assistant create a new project

- Project Type: `Migration`
- Project Name: `tailwind`
- Source server type: `SQL server`
- Target server type: `Azure SQL Database`
- Migration Scope: `Schema Only`

1. Click `Create`
5. Click Connect
6. Select the `TailwindInventory` database, click `Next`
7. Target Server:  This will be the Azure SQL Server Instance you created.  
   1. In the Azure Portal click on the resource group icon and select your resource group.
   2. Find the SQL Server Instance you created.  It will be resource type of SQL Server
   3. Copy the server name on the right hand side of the overview page
   4. Paste that full name into the target server name of the wizzard
8. Choose SQL Server Authentication
9. User: migrateadmin
10. Password: 'AzureMigrateTraining2019#'
11. Press Connect
12. Choose your database and press next
13. Select all tables and press 'Generate SQL script'
14. Once the script is generated, you may review it
15. Press 'Deploy Schema'
16. You now have your schema successfully migrated to Azure SQL DB.

#### Data Migration

Now that you have the schema migrated, you now need to move the data.  You will use the Azure Data Migration Service for this as it is much more robust option and can migrate the data with very minimal downtime. 

1. In the Azure Portal click on the resource group icon and select your resource group.
2. Find the resource of type 'Azure Database Migration Service' and click it.
4. Click on `Create new migration project`
5. Project name: `tailwind`
6. Source server type: `SQL Server`
7. Target server type: `Azure SQL Database `
8. Type of activity: `Offline data migration`
10. Click `Create and run activity`

##### Migration Wizard

1. Source Detail
   1. Source SQL Server Instance Name: The IP Address of your SQL Server VM(you can open the portal in a new tab, click on your VM and get the IP )
   2. Authentication type: `SQL Authentication`
   3. User: migrateadmin
   4. Password: 'AzureMigrateTraining2019#'
   5. Uncheck the encrypt connection box
2. Select Target
   1. Full name of the Azure SQL instance
   2. Authentication type: `SQL Authentication'
   3. User: migrateadmin
   4. Password: 'AzureMigrateTraining2019#'
3. Click save
4. Select `TailwindInventory` database from the source
5. Select your database as the Target Database -> Save
6. Select all tables and click Save.
7. Give a name to the migration activity and select "Don't Validate the Database" in the database validation options.
8. Run the migration

**<u>Congratulations!</u>**  You have successfully migrated from the VM instance of SQL to Azure SQL DB!  You can check to see the data is there by using the portal based query tool.

By default Azure SQL Databases reject all traffic to them.  You were able to run the Azure Database Migration tool because you checked the box to allow other Azure services to connect to it.  I in order to connect to it via other tools you need to open an exception for our IP address through the firewall.

##### Modify SQL Firewall

1. Click on your resource group
2. Click on your Azure SQL Server Instnace
3. Click on Firewalls and virtual networks in the left hand pane
4. You will see your client IP address listed.   You need to create a new rule allowing that IP like this ![SetSQLIP](../images/SetSQLIP.png)
5. Press Save

##### Check your data in SQL

1. Click on your resource group
2. Click on your Azure SQL DB database
3. Click Query Editor on the left toolbar
4. Login as migrateadmin
5. Expand tables - you should see all your tables.
6. Run - 'select * from inventory' and you should see all your inventory data.

### Exercise 2 - Migrate On-Premises MongoDB to Azure Cosmos DB

The next step is to get the product database migrated to Azure.  Here you are moving an on-premises MongoDB (as represented in this session by an Azure Linux VM running MongoDB) to Azure Cosmos DB using native MongoDB commands.

#### Connect to the MongoDB Linux VM

You have a shared Linux VM that is simulating the production MogoDB product database.  You will connect remotely to this server in order to get a dump of data to put into the Cosmos DB.   The great thing about this is that Cosmos DB can use standard MongoDB tools.

You can do all this from the Azure Bash Shell

1. Launch a new Azure Command Shell.  You can either:
   1. Press the shell icon in the Azure Portal, as in the setup for the Cosmos DB
   2. Open a new browser tab to:  http://shell.azure.com for a full screen experience
   
2. Download the MongoDB client tools (you can paste into the shell with a right click)

   ```bash
   wget https://repo.mongodb.org/apt/ubuntu/dists/xenial/mongodb-org/4.0/multiverse/binary-amd64/mongodb-org-tools_4.0.11_amd64.deb
   ```

3.  Unpack the downloaded zip file 

   ```
   dpkg-deb -R mongodb-org-tools_4.0.11_amd64.deb ~/mongotools
   ```

4. Set the path to include the tools  

   ```
   export PATH=$PATH:~/mongotools/usr/bin
   ```

5. You now have the MongoDB client tools available to you.

#### Export the MongoDB Data

1. Dump the data from the remote MongoDB with the following Command

   1. ```bash
      mongodump --host 52.175.230.38 --username=labuser --password=AzureMigrateTraining2019# --db=tailwind --authenticationDatabase=tailwind
      ```

3. Check to see that you successfully dumped the data

   1. Check that the directory has a dump and tailwind directory that contains the .bson and metadata files.  Run the following ls commands:
      ![CheckMongoDump](../images/CheckMongoDump.png)



#### Check the Cosmos DB provisioning

Now that you have a copy of the data locally, you can use the mongorestore command to load that data into our Cosmos DB instance you created.

First check to make sure the Cosmos DB instance was created successfully.  

1. In the Azure portal click on the resource groups on the left toolbar
2. Click on your Resource Group.
3. You should see a resource of type 'Azure Cosmos DB account' - Click that
4. Click on 'Data Explorer' on the left navigation.  You should see something the following:
   ![cosmosempty](../images/cosmosempty.png)
5. Notice the collections is empty.  This is OK.  It shows you have a Cosmos Instance and after you restore you should have our product data



#### Restore the Data to our Cosmos DB

1. You will create a few environment variables to store Cosmos DB information for our restore command.

2. You will need the Cosmos DB username and password.  

   1. Navigate to the Cosmos DB account in the Azure portal.
   2. Click on the Connection String option on the left navigation
   3. The username and password for you Cosmos DB instance can be copied from here.

3. Go back to the Azure shell 

4. Create the following environment variables in that shell

   > ```language-bash
   > COSMOS_DB_NAME=<Host name from connection string properties>
   > COSMOS_USER=<username from connection string properties>
   > COSMOS_PWD=<primary password from connection string properties>
   > ```

   

5. Make sure you are still in the /dump/tailwind directory still

6. Copy and paste this command to run a mongo restore:

```language-bash
mongorestore \
    --host $COSMOS_DB_NAME:10255 \
    -u $COSMOS_USER \
    -p $COSMOS_PWD \
    --ssl \
    --sslAllowInvalidCertificates \
    inventory.bson \
    --db tailwind \
    --collection inventory
```
8. Go into your Azure Cosmos DB account and click on `Data Explorer`. 
9. Press the refresh button next to Collections if you don't see the tailwind collection
10. Select the `tailwind` database.
11. Expand the `tailwind` node, expand the `inventory` node, and select `Documents`
12. You should see the inventory item documents are now in Cosmos DB


![cosmospopulated](../images/cosmospopulated.png)



**Congratulations!**   You have now successfully moved both a SQL Server Database and a Mongo DB database to the Azure cloud!



## Learn More/Resources

* [Create an Azure Cosmos DB database built to scale](https://docs.microsoft.com/learn/modules/create-cosmos-db-for-scale/?WT.mc_id=msignitethetour-github-mig20)
* [Work with NoSQL data in Azure Cosmos DB](https://docs.microsoft.com/learn/paths/work-with-nosql-data-in-azure-cosmos-db/?WT.mc_id=msignitethetour-github-mig20)
* [Work with relationall data in Azure](https://docs.microsoft.com/learn/paths/work-with-relational-data-in-azure?WT.mc_id=msignitethetour-github-mig20)
* [Secure your cloud data](https://docs.microsoft.com/learn/paths/secure-your-cloud-data?WT.mc_id=msignitethetour-github-mig20)
* [Azure migration resources](https://azure.microsoft.com/migration?WT.mc_id=msignitethetour-github-mig20)
* [Microoft Data Migration Assistant](https://docs.microsoft.com/sql/dma/dma-overview?WT.mc_id=msignitethetour-github-mig20)
* [Azure total cost of ownership calculator](https://azure.microsoft.com/pricing/tco/calculator?WT.mc_id=msignitethetour-github-mig20)
* [Microsoft Learn](https://docs.microsoft.com/learn?WT.mc_id=msignitethetour-github-mig20)




