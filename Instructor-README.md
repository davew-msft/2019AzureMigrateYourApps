# Microsoft Azure Training Day: Migrating Applications to the Cloud.

## Introduction

Welcome to the Migrating Applications to the Cloud Training.  This page will give you all the information you need to know in order to execute a successful event.  The first half of the day is based on a story.  That is a story based on Tailwind Traders.

The story goes that, Tailwind Traders acquired Northwind Traders earlier this year, they wanted to be sure that they could access their inventory in real time, which meant moving their existing web API alongside ours on Microsoft Azure.   Besides moving the local web aps and API's to Azure there is a need to move the On-Premise MongoDB and SQL Server to the cloud as well.

## Presentations 

The readme in the presentation folder will give you an update as to when that presentation was last validated.  Please check the date and augment as necessary to be accurate.  There are five presentations to match the five informational sessions in the agenda.  See the below section on Session Notes for more details on each of those.

## Lab Folders

The lab folders are self contained labs.  You should not need to go to other resources to run the labs.  The original code base of the database and application migration labs are from these events.  

[IgniteTheTour-DEV10](https://github.com/microsoft/IgniteTheTour/tree/master/DEV%20-%20Building%20your%20Applications%20for%20the%20Cloud/DEV10)

[IgniteTheTour-MIG20](https://github.com/microsoft/IgniteTheTour/tree/master/MIG - Migrating Applications to the Cloud/MIG20)

## Setupfiles

You should not need to do anything with the setup files.  There is a backup file for SQL that is used during the database migration lab.  There is also a MongoDB backup in case you need to create your own MongoDB server.  See notes about that in Session 3.

## Demo Setup

You Lab 1 and 2 together create a demo that shows an application migrated to the cloud.  You should create a finished environment so you can show the end result during session 1.  You have two choices to get an environment up and running:

1. Run though the labs yourself and in the end you should have the full demo
2. You can use this [IgniteTheTour-DEV10](https://github.com/microsoft/IgniteTheTour/tree/master/DEV%20-%20Building%20your%20Applications%20for%20the%20Cloud/DEV10) repo and deploy the deployment/deploy.sh script.  Please use the Azure Bash shell as we know it has all the dependencies.   That should create a fully working environment. 

## Session Notes

### Session 1 - Cloud Apps and Azure

- **Type** - Presentation
- **Objectives** 
  - Setup the Tailwind / Northwind story
  - Talk about benefits of moving to applications to the cloud
  - Show the Tailwind legacy architecture and what the new architecture will look like after migration
- **Skill Set**
  - Basic knowledge of Azure and cloud architectures

### Session 2 - Overview of Database Migrations 

- **Type** - Presentation
- **Objectives**
  - Educate on the different offerings of data storage on Azure
  - Talk about the different migration options and process
  - Outline what they are about to do in the hands on lab
- **Skill Set** 
  - Presenter should have 200-300 level knowledge of Azure data storage and migration options.
- **Notes**
  - This session is based on an ignite session.  You can view this [Video](https://techcommunity.microsoft.com/t5/Microsoft-Ignite-The-Tour/Moving-your-database-to-Azure/m-p/284149) for some pointers on delivery, but some information may be out of date.  The lab in session 3 is based on the demo in this session.

### Session 3 - Migrate SQL and Mongo Data to Azure

- **Type** - Hands on Activity 
- **Objectives** 
  - Have students successfully
    - Experience Azure through the portal and command line
    - Create a SQL VM in the cloud
    - Migrate the SQL VM data to a Azure SQL Instance via Azure Data Migration Services
    - Create a Cosmos DB 
    - Migrate data from a Linux based MongoDB to an instance of Cosmos DB
- **Skill Set**
  - Presenter should have 
    - 200-300 level knowledge of Azure Data Platforms
    - 200-300 level knowledge of Data Migration Assistant and Azure Data Migration Services
- **Notes**
  - Everything needed for the SQL Azure migration is in the lab
  - The MongoDB that you migrate from is a shared instance setup for this series.  It will be up and running until the end of 12/2019.  If you need to set one up for your use see the appendix notes below.



### Session 4 - Containerizing and Orchestration of applications on Azure

- **Type** - Presentation
- **Objectives**
  - Introduction to containers and their benefits
  - Options of running containers on Azure
  - Options of orchestration of containers on Azure
  - Outline the options they will use in the hands on activity
- **Skill Set**
  - 200-300 developer skills with knowledge of 
    - Containers / Docker
    - Orchestration engines 
      - Kubernetes
- **Notes**
  - The "Developing Kubernetes Applications" section of the presentation and the Draft/Helm Demos are time permitting.

### Session 5 - Migrating apps to App Services on Azure

- **Type** - Hand on Activity
- **Objectives**
  - Create an Azure container registry
  - Build containerized Web Apps for 2 API's and 1 Front End Service
  - Connect services to databases migrated in the first lab
  - Successfully get their new containerized web app to run
- **Notes**
  - Everything should be able to run from the Azure Bash shell.
  - There are a few manual replacements code in the configuration parameters of the Web Services.  Students could get caught in typos here.

### Session 6 - Serverless Computing - Bring you app to the next level

- **Type** - Presentation 
- **Objectives**
  - Educate students on pure PaaS offerings of Azure
  - Make clear the advantages of pure PaaS capabilities of services like Azure Functions bring as compared to containers.
- **Skill Set**
  - 200-300 level developer knowledge of 
    - Azure Serverless capability and advantages over other deployment methods.
- **Notes**
  - Due to time we were unable to create a lab for this session, but we felt it important to make sure that students understand what true PaaS and serverless options bring to the table. 
  - A simple Serverless API demo is described in the Notes section of the Demo slide and can be done in under 5 minutes (time permitting)

### Session 7 - DevOps, Deploying Your Applications Faster and Safer

- Type - Presentation 
- **Objectives**
  - Overview of the DevOps philosophy and process
- **Skill Set**
  - 200-300 DevOps
- **Notes**

### Session 8- 

- **Type** - Hands on Activity 

- **Objectives**

  - Create an Azure DevOps project
  - Successfully create a deployment pipeline
  - Successfully have pipeline update live code on a source code change*

- **Notes**

  - This lab is not connected to the source code from the previous lab.  It is complete independent and different.  Due to time we could not create a lab based on the previous code base, so this was reused from another training session.

  

### Appendix A - Create a Linux MongoDB

If you need a MongoDB to perform Lab 3.  Follow these steps:

1. In Azure press create new resource and type 'MongoDB'
2. Create a VM based on one of the MongoDB templates. 
   1. This session used the one by Jetware
3. Set it's IP address to static
4. Make sure two firewall ports are open for
   1. SSH - 22
   2. MongoDB - 27017
5. Once Created SSH to the VM 
6. Install the MongoDB client tools 
   1. sudo apt install mongo-clients
7. Download the inventory.bson and inventory.metadata.json using wget
8. Restore the database using mongorestore command
9. Create the user in the lab
   1. At the linux prompt type 'mongo'
   2. db.createUser({user: "labuser", pwd:"AzureMigrateTraining2019#", roles:[{role: "read", db:"tailwind"}]})

 

