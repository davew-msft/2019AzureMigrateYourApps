# Lab 1 - Moving Your On-Premises Data Servers To Azure

## Services Used

* Azure Cosmos DB (with MongoDB API)
* Azure SQL Database
* Azure SQL Data Migration Service

### Exercise 1 - Create an Azure Cosmos DB account with Cloud Shell

Exercise 1 shows how to create a new Azure Cosmos DB account using the Azure Command Line Interface (CLI) and the Cloud Shell.

The services in this exercise include:

* [Azure Cosmos DB](https://docs.microsoft.com/azure/cosmos-db?WT.mc_id=msignitethetour-github-mig20)
* [Azure CLI](https://docs.microsoft.com/cli/azure?WT.mc_id=msignitethetour-github-mig20)
* [Azure Cloud Shell](https://docs.microsoft.com/en-us/azure/cloud-shell/overview?WT.mc_id=msignitethetour-github-mig20)

Follow these steps:

1. Open up the Azure portal
1. Click on the `>_` button in the toolbar, and wait for the Cloud Shell to initialize (it will take a few seconds).
![cloud shell command](https://docs.microsoft.com/en-us/azure/cloud-shell/media/overview/overview-bash-pic.png)
1. Select `bash` from the dropdown of the Cloud Shell window.

1. Create a _Resource Group_ to put the Azure Cosmos DB instance in. First off create 3 Bash variables:
  * Resource Group Name
  * Region tl host the Azure Cosmos DB instance
  * The account name

```language-bash
RESOURCE_GROUP_COSMOS='mig20-cosmosdbgroup'
LOCATION_COSMOS='eastus'
ACCOUNT_NAME_COSMOS='mig20cosmosdbaccount'
```
5. Then create the _Resource Group_ itself. (A _Resource Group_ is a logical location of resources, or Azure services grouped together.)

```language-bash
az group create --name $RESOURCE_GROUP_COSMOS --location $LOCATION_COSMOS
```

6. Then create the Azure Cosmos DB Account and place it into the resource group you just created.

```language-bash
az cosmosdb create \
    --resource-group $RESOURCE_GROUP_COSMOS \
    --name $ACCOUNT_NAME_COSMOS \
    --kind MongoDB \
    --locations "East US"=0 \
    --default-consistency-level "ConsistentPrefix" \
    --enable-multiple-write-locations true
```

This will take several minutes to spin up. When it is finished (you'll see a bunch of JSON indicating it's done) you can go into the portal and click on `Resource Groups` from the left hand side.

![resource group in portal image](images/resource-groups.png)

While you wait for your Cosmos DB instance to spin up you can move on to the next Lab on miragting a Mogo DB.  

1) Click 'New Session' Icon in the command shell toolbar.
<Insert Image Here for the Icon>
 
### Lab 2 - Migrate On-Premises MongoDB to Azure Cosmos DB

Here we are moving an on-premises MongoDB (as represented in this session by an Azure Linux VM running MongoDB) to Azure Cosmos DB using native MongoDB commands.

#### Prerequistes to run theh demos if on WSL (or macOS)

> In the original demo during Ignite The Tour, we used Ubuntu running on Windows Subsystem for Linux (WSL). You can use that, or you can continue to use the Cloud Shell if you like. These commands will work in either.
> If you do use Ubuntu on Windows, you will need to make sure to follow the [instructions here](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli?WT.mc_id=ignitethetour-github-mig20) to install the Azure CLI.

> You will also need to install the Linux VM's RSA certificate using the following commands:
> ```language-bash
> $RESOURCE_GROUP_NAME=<WHATEVER YOU USED ABOVE>
> $MONGO_VM_NAME=mongo
> az vm user update -u azureuser --ssh-key-value "$(< ~/.ssh/id_rsa.pub)" -g $RESOURCE_GROUP_NAME -n $MONGO_VM_NAME
> ```

During this demo we will be using the native MongoDB commands:

* `mongodump` to get the data out of the source MongoDB
* `mongorestore` to get the data into Azure Cosmos DB

The steps to run the demo are below:

1. Find out the IP address of the Linux VM.
  * Go into the Azure portal
  * Click on the `Resource Groups` on the right hand menu
  * Filter by the resource group name you created during the installation, and click into it
  * Find the item called `mongoPublicIP`, click on it, and copy the element called `IP address`.
  ![mongo public ip address node](images/mongo-public-ip.png)
2. Find out the host name of the Azure Cosmos DB
  * Back in the list of resources, click on the Azure Cosmos DB account that you're using
  * Click `Connection String` on the right hand side, then copy the value from `Host`, `Username`, `Primary password`, and `Primary Connection String`
  ![cosmos host value](images/cosmos-host.png)
  ![cosmos primary conx string](images/cosmos-primary-conx.png)
3. ssh into the Linux VM: `ssh azureuser@MONGO-IP-ADDRESS`
4. Run a mongo dump, which exports all the data to a file: `mongodump --collection inventory --db tailwind`
5. Then change into the directory that contains all dump files from the MongoDB server: `cd dump`
6. Change into the directory that contains our particular dump file: `cd tailwind`
7. Run a mongo restore:

```language-bash
mongorestore \
    --host <COSMOS HOST ADDRESS>:10255 \
    -u <COSMOS USER NAME> \
    -p <COSMOS PASSWORD> \
    --ssl \
    --sslAllowInvalidCertificates \
    inventory.bson \
    --numInsertionWorkersPerCollection 4 \
    --batchSize 24 \
    --db tailwind \
    --collection inventory
```
8. Switch the connection string to the Product Service website, so go back out to the overall Resources view and click on `<RESOURCEPREVIX>product` app service. (The `<RESOURCEPREFIX>` is the value you set during installation.)
9.Click on `Application Settings` from the right hand menu.
10. Enter the Cosmos DB `Primary Connection String` copied above into the _value_ portion for `DB_CONNECTION_STRING`. This is located in the `Application Settings` section.
![product service db connection string value](images/DB_CONNECTION_STRING.png)
11. The Product Descriptions are now coming from Cosmos DB - so let's prove that it's working by adding an item.
12. Go into your Azure Cosmos DB account and click on `Data Explorer`. Select the `tailwind` database.
![Azure Cosmos DB Data explorer](images/cosmos-data-explorer.png)
13. Expland the `tailwind` node, expand the `inventory` node, and select `Documents`.
14. Click on the `New Document` button and past the JSON found in `Files\new-item.json` (in this repo), and then click `Save`.
![New cosmos item](images/new-cosmos-item.png)
15. View the website again, and see the item show up at the very bottom.
16. Feel free to browse the portal for Azure Cosmos DB - especially `Replicate data globally`. Click on the map and data will instantly be moved.
![instantly replicate data](images/cosmos-replicate-data.png)
17. The `Metrics` tab shows how fast Azure Cosmos DB is responding.

### Demo 3 - Assess DB Migration Using the DB Migration Tool and Setup for SQL Managed Instance Migration

The next portion of the session will talk about the Inventory service. The inventory service is hosted on a SQL server and served by an ASP.NET core website. The Inventory service determines the quantity of a unit that's currently in stock.

On the home page of the web site, you can see the SQL database name hosting the inventory. Whether it's the on-premises db or the new SQL Managed Instance (MI).

The on-premises database in this case is modeled by a Windows 2012/SQL 2012 virtual machine.

> **IMPORTANT** This portion can be run on Windows only!
> To run this demo you will need the Microsoft Data Migration Assistant, follow [these instructions](https://docs.microsoft.com/sql/dma/dma-overview?WT.mc_id=msignitethetour-github-mig20) to install.
> You will also need the Microsoft Da

The steps to run the demo are as follows:

#### Assessment

1. Install the Data Migration Assistant, open it up
1. Create a new project
  * Project Type: `Assessment`
  * Project Name: `tailwind`
  * Source server type: `SQL server`
  * Target server type: `Azure SQL Database Managed Instance`
1. Click `Next`
1. Check `Check database compatibility`
1. Check `Check feature parity`
1. Click `Next`
1. Enter the SQL 2012 server name and authentication credentials.
  * The server name can be obtained through the portal. Open up the resource view and then click on `sql2012-ip`. When that opens copy the `IP Address`.
  ![sql 2012 IP node](sql-ip-address.png)
1. Select the `tailwind` database, click `Add`.
1. Click `Start Assessment`.

#### Migration

1. From the portal, view all the resources for the resource group you created, and select the managed instance.
![managed instance selection](images/sql.mi.png)
1. Notice the MI is running in its own Virtual Network
1. Back out to the overall resources view and open the `sqldms` or the `Azure Database Migration Service`.
1. Click on `Create new migration project`
  * Project name: `tailwind`
  * Source server type: `SQL Server`
  * Target server type: `Azure SQL Database Managed Instance`
  * Type of activity: `Offline data migration`
  * Click Save
1. Click `Create and run activity`

##### Migration Wizard

1. Source Detail
  * Source SQL Server Instance Name: The IP Address you obtained above for the SQL server 2012 IP Address.
  * Authentication type: `SQL Authentication`
  * User name: the user name value created when you ran the install
  * Password: the password value created when you ran the install
1. Select Target
  * Managed Instance host name: obtained from going into the SQL MI node from all resources view and copying the `Host name` value.
  ![sql mi host](images/sql-mi-host.png)
  * Authentication type: `SQL Authentication`
  * User anme: the user name value created when you ran the install
  * Password: the password value created when you ran the install
  * Click save
1. Select `tailwind` database from the source
1. Do not migrate logins
1. Give a name to the migration activity and don't validate the database.
1. Run the migration

### Demo 4 - Check SQL MI Status and Redeploy Web App's Container

Now you can change the inventory's app settings to point at the new SQL Managed instance.

The connection string will be of the format:

```
Server=tcp:40.114.36.51,1433;Initial Catalog=tailwind;User Id=USERNAME;Password=PASSWORD;MultipleActiveResultSets=False;Encrypt=True;TrustServerCertificate=True;Connection Timeout=30;
```


View the website to see it change.

## Learn More/Resources

* [Create an Azure Cosmos DB database built to scale](https://docs.microsoft.com/learn/modules/create-cosmos-db-for-scale/?WT.mc_id=msignitethetour-github-mig20)
* [Work with NoSQL data in Azure Cosmos DB](https://docs.microsoft.com/learn/paths/work-with-nosql-data-in-azure-cosmos-db/?WT.mc_id=msignitethetour-github-mig20)
* [Work with relationall data in Azure](https://docs.microsoft.com/learn/paths/work-with-relational-data-in-azure?WT.mc_id=msignitethetour-github-mig20)
* [Secure your cloud data](https://docs.microsoft.com/learn/paths/secure-your-cloud-data?WT.mc_id=msignitethetour-github-mig20)
* [Azure migration resources](https://azure.microsoft.com/migration?WT.mc_id=msignitethetour-github-mig20)
* [Microoft Data Migration Assistant](https://docs.microsoft.com/sql/dma/dma-overview?WT.mc_id=msignitethetour-github-mig20)
* [Azure total cost of ownership calculator](https://azure.microsoft.com/pricing/tco/calculator?WT.mc_id=msignitethetour-github-mig20)
* [Microsoft Learn](https://docs.microsoft.com/learn?WT.mc_id=msignitethetour-github-mig20)
