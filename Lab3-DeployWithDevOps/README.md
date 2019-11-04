# Lab3 - Use Azure DevOps to manage build and deployment of your application

## Lab Goals: Deploying a Docker based web application to Azure App Service
In this lab, you will use Azure DevOps to automate the build and deployment of your application to Azure

### Learning Objectives
    - Integrate GitHub with Azure DevOps to automate build processes
    - Build Docker images using Azure DevOps 
    - Push and store the Docker images into Azure Container Registry
    - Deploy and run the images inside the Docker Containers


## Exercise 1 - Create a new project and clone the GitHub repo
1. Open the Azure DevOps portal [Azure DevOps](http://dev.azure.com) 
2. Use link `Sign in to Azure DevOps` and login using your assigned credentials
3. Select `+ New Project`
4. Set the **Project name** to a name of your choice
5. In the **Visibility** section, select `Private`
6. Select `Advanced`, in **Version control** select 'GIT' and in **Work item process** select 'Scrum' 
7. Select `Create` at the bottom right of the page5. Once your project has been created, select `Repos` from left-pane menu
6. From the Repos page, select `Import` in the `or import a repository` section
7. `Source type` should be set to 'Git', and in `Clone URL` enter this URL and leave `Requires authorization` unchecked
    https://github.com/MSTecs/Azure-DevDay-lab4-demoProject.git
8. Select `Import`
9. Select `Files` under `Repos` from the left-pane menu and you will see the cloned GitHub repo


## Exercise 2 - Configure Continuous Integration
### Task 1 - configure the DevOps Pipeline
1. Open another browser tab and login to the Azure portal with your assigned credentials  
2. Select `Resource groups` from the left-pane menu  then select the resource group 'Lab-3-xxxxx'
3. Select the resource of type 'Container registry', copy the `Login server` in the properties of the `Overview` page, and save them to a Notepad file
4. Using the breadcrumbs on the top-left of the page, select your resource group to go back to the resource group page
5. Select the resource of type 'SQL database', copy the `Server name` in the properties of the `Overview` page, and save them to a Notepad file
6. Return to the Azure DevOps portal
7. Select `Pipelines` from the left-pane menu and then select `Builds`.  Select `New pipeline`
8. Select `Use the classic editor to create a pipeline without YAML` option at the bottom of the page
9. Select `Azure Repos Git` and select your project and repository, leave the branch on 'master' 
10. Select `Continue`
11. Select the `Empty job` option at the top of the page
12. Select `Agent job 1` and change `Display name` to 'Docker'
13. Set `Agent pool` to 'Azure pipelines'
14. Set `Agent Specification` to 'ubuntu-16.04' from the dropdown
13. Scroll down to the `Execution Plan` section, and 'Job cancel timeout' to 1
14. From the top menu, select the dropdown for `Save & queue` and choose `Save` using the default values

### Task 2 - create Run services

In order to complete this task, you will need the original email link sent to you as it contains configuration information for this set of tasks

1. Select `+` next to your Docker agent on the left-pane
2. Search for 'docker compose' and then select `Add`
3. Select `Run a Docker Compose command` from the left-pane and set `Display name` to 'Run services' 
4. From `Azure subscription`, select the dropdown next to `Authorize` and select `Advanced options`
5. In the 'Add an Azure Resource Manager service connection' window, select `use the full version of the service connection dialog` towards the bottom
6. From your email link (noted above), set `Service principal client ID` to the email link value for  `Application/Client Id` 
7. From your email link (noted above), set `Service principal key` to the email link value for `Application Secret Key` 
5. Select `OK` from the window to create a service connection
6. Select `Azure Container Registry` using the dropdown.  In the resource group `Lab-3-xxxxx', a container registry service was created for you and this is the one you will use
7. From `Docker Compose File`, select `...` and search for the file 'docker-compose.ci.build.yml'
8. Scroll down to `Action` and select 'Run service images' from the dropdown
9. Uncheck `Run in Background`
10. Scroll down to the `Output Variables` section and set `Reference name` to 'Task1'
11. From the top menu, select the dropdown for `Save & queue` and choose `Save` using the default values

### Task 3 - create Build services

1. From the left-pane menu, select `+` next to `docker` and search for 'docker compose' and select `Add`
2. Select `Run a Docker Compose command` from the left-pane
3. Set `Display name` to 'Build services'
4. From `Azure subscription`, select your previously created service connection 
5. Select your `Azure Container Registry` from the dropdown
6. From `Docker Compose File`, select `...` and search for the file 'docker-compose.yml'
7. Enter 'DOCKER_BUILD_SOURCE=' in the `Environment Variables` dialogue
8. From `Action` select 'Build service images' from the dropdown
9. Scroll down to the `Output Variables` section and set `Reference name` to 'Task2'
10. From the top menu, select the dropdown for `Save & queue` and choose `Save` using the default values

### Task 4 - Create Push Services
1. From the left-pane menu, select `+` next to `docker` and search for 'docker compose' and select `Add`
2. Select `Run a Docker Compose command` from the left-pane
3. Set `Display name` to 'Push services'
4. From `Azure subscription`, select your previously created service connection 
5. Select your `Azure Container Registry` from the dropdown
6. From `Docker Compose File`, select `...` and search for the file 'docker-compose.yml'
7. Enter 'DOCKER_BUILD_SOURCE=' in the `Environment Variables` dialogue
8. From `Action` select 'Push service images' from the dropdown
9. Scroll down to the `Output Variables` section and set `Reference name` to 'Task3'
10. From the top menu, select the dropdown for `Save & queue` and choose `Save` using the default values

### Task 5: create Publish Artifact
1. From the left-pane menu, select `+` next to `docker` and search for 'Publish build artifacts' and select `Add` 
2. Select `Publish Artifact: drop` from the left-pane
3. Set `Display name` to 'Publish artifacts'
4. Set `Path to publish` to 'myhealthclinic.dacpac'
5. Set `Artifact name` to 'dacpac'
6. Scroll down to the `Output Variables` section and set `Reference name` to 'PublishBuildArtifacts2'
7. From the top menu, select the dropdown for `Save & queue` and choose `Save` using the default values

### Task 6 - Set variables
1. Select `Variables` from top menu
2. Create these variables by selecting the `+ Add` button
    - Name: BuildConfiguration Value: Release.  Check the `Settable at queue time` option
    - Name: BuildPlatform Value: Any CPU
3. From the top menu, select the dropdown for `Save & queue` and choose `Save` using the default values

### Task 7 - configure triggers for continuous integration
1. Select `Triggers` from top menu
2. Check `Enable continuous integration` 
3. From the top menu, select the dropdown for `Save & queue` and choose `Save` using the default values

### Task 8 - configure build options
1. Select `Options` from top menu
2. On the left-pane, set `Build number format` to '$(date:yyyyMMdd)$(rev:.r)'
3. On the right-pane, set `Build job timeout in minutes` to '0'
4. From the top menu, select the dropdown for `Save & queue` and choose `Save` using the default values

## Task 9 - Set the compute host
1. Select `Tasks` from top menu
2. Select `Pipeline` just below `Tasks`
3. Set `Agent Pool` to 'Azure Pipelines' from dropdown
4. Set `Agent Specification` to  'Ubuntu 16.04' from dropdown 
5. From the top menu, select the dropdown for `Save & queue` and choose `Save` using the default values

## Exercise 3 - Configure Continuous Delivery

### Task 1 - create the release pipeline
1. From the left-pane menu, select `Pipelines` and then select `Releases` 
2. Select `New pipeline` and select `Empty job` from top of page on the right side
3. Set `Stage name` to 'Dev'
4. Select `+ Add` from left-pane `Artifacts` section
5. Select `Build` from `Source type`
6. Set `Project` to your DevOps project name - it should be auto-suggested
7. Select your build pipeline from the `Source (build pipeline)` dropdown
8. Set `Default version` to 'Latest'
9. Set `Source alias` to the suggested value - it will be created from your DevOps project name
10. Select `Add`
11. Select the `Continuous deployment trigger` icon located in the `Artifacts` section on the left (it resembles a lightning bolt) and on the right set `Continuous deployment trigger` to 'Enabled'
12. Select `Save` from top menu and save using the default values

### Task 2 - configure the SQL database deployment
1. From the top menu, select `Tasks` 
2. Select `Agent job`
3. Set `Display name` to 'DB Deployment'
4. Select the `+ Add` button and create the variable
    - Name: sqlpackage, Condition: exists
5. Select `+` next to `DB Deployment` in left-pane and search for 'Azure SQL database deployment' and select `Add`
7. From the left-pane, select `Azure SQL Dacpac Task`
8. Set `Display name` to 'Execute Azure SQL : DacpacTask'
9. From `Azure subscription`, select your previously created service connection 
10. Set `Azure SQL Server` to '$(SQLserver)' 
11. Set `Database` to '$(DatabaseName)'
12. Set `Login` to ' $(SQLadmin)' (with a blank space at the beginning)
13. Set `Password` to '$(Password)' 
14. Scroll down to the section `Deployment Package`, and set `DACPAC File` to '$(System.DefaultWorkingDirectory)/**/*.dacpac' 
15. Scroll down to the section `Output Variables`, and set `Reference name` to 'SqlAzureDacpacDeployment1'
16. Select `Save` from the top and save using the default values

## Task 3 - create and configure the Web App deployment agent
1. From the left-pane, select `...` next to `Dev` and select `Add an Agent Job`
2. Select the new `Agent job` 
3. Set `Display name` to 'Web App deployment'  
4. Set `Agent pool` to 'Azure Pipelines' from the dropdown
4. Set `Agent Specification` to 'ubuntu-16.04'  from the dropdown
5. Select `Save` from the top and save using the default values

## Task 4 - create and configure the app service deployment task
1. Select `+` next to `Web App deployment` in the left-pane and search for 'Azure App Service Deploy' and select `Add`
2. Select your new `Azure App Service Deploy:` from the left-pane
4. Set `Task version` to  '3.*'  from the dropdown
5. Set `Display name` to 'Azure App Service Deploy' 
6. From `Azure subscription`, select your previously created service connection 
7. Set `App type` to  'Linux Web App' from the dropdown
8. Set `App Service Name` to the name of your App Service.  An App Service was pre-provisioned for you and you should select its suggestion 
9. Set `Registry or Namespace` to '$(ACR)'
10. Set `Image` to 'myhealth.web' 
11. Set `Tag` to '$(BUILD.BUILDID)'
12. Scroll down to the section `Output Variables`, and set `Reference name` to 'AzureRmWebAppDeployment1' 
13. Select `Save` from the top menu and save using the default values

## Task 5 - configure variables
1. Select `Variables` from the top menu
2. Select `+ Add` and enter the following

    - Name: ACR, Value: YOUR_ACR.azurecr.io, Scope: Release
        This is the value of the login server of the Lab-3-xxxx container registry service you copied previously
    - Name: DatabaseName, Value: mhcdb, Scope: Release
        this is the value of the server name of the Lab-3-xxxx SQL database you copied previously
    - Name: Password, Value: P2ssw0rd1234, Scope: Release
    - Name: SQLadmin, Value: sqladmin, Scope: Release
    - Name: SQLserver, Value: YOUR_DBSERVER.database.windows.net, Scope: Release

3. Select `Save` from the top menu and save using the default values


## Exercise 4 - Initiate the Continuous Integration Build and Deployment

1. From the left-pane menu, select `Repos` and then `Files`.  Navigate to the (ProjectName)/src/MyHealth.Web/Views/Home folder and open Index.cshtml file for editing
    - Modify the text 'JOIN US' to 'CONTACT US' on line 28, and then select `Commit` button accepting its default values.  This action will initiate an automatic build 
2. Select `Pipelines` and then `Builds` from the left-pane menu then select the build name just committed from the right-pane
    - The automatic will generate and push the docker image of the web application to the Azure Container Registry
    - You can watch each of the steps you created being executed in this build
    - Once the build is completed (all green check boxes), the build summary will be displayed.  The build may take a few minutes
3. From the Azure Portal, select the app service in the 'Lab-3-xxxxx' resource group and then select `Container settings` from the left-hand pane
    - Set `Image source` to `Azure Container Registry`
    - Set `Registry` to its default value from the dropdown
    - Set `Image` to 'myhealth.web' from the dropdown
    - Set `Tag` to 'latest' from the dropdown
    - Set `Continuous Deployment` to 'Off'
    - Select `Save`
3. From the Azure Portal, select your 'Azure Container Registry' service and then select `Repositories` option to view the generated docker images.  You should see a repository named 'myhealth.web'
4. From Azure Dev Ops portal, select `Pipelines` and then `Releases`. Select  the latest release, hover over `Dev` in `Stages`, and select `Logs` to view the details of the release in progress
    - Both the DB Deployment and Web App Deployment need to complete
    - Once the deployment is completed (all green check boxes), the deployment summary will be displayed
5. From the Azure Portal, select your app service and then `Overview` from the left-pane menu. Select the link next to the `URL` field to browse the application
6. Use the credentials Username: user and Password: P2ssw0rd@1 to login to the HealthClinic.Biz web application
    - If this is the first time using the application, you might have a delay as the services are initially hydrated


**Congratulations!  You have successfully built and deployed a functioning DevOps pipeline that includes continuous integration.  You have completed all the lab exercises, and you are free to explore more in the Azure environment provisioned for you.**
