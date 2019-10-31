# LAB GUIDE
## Lab: Deploying a Docker based web application to Azure App Service
### Learning Objectives
    - How to build custom Docker images using Azure DevOps Hosted Linux agent
    - How to push and store the Docker images in a private repository
    - How to Deploy and run the images inside the Docker Containers

### Pre-requisites
* Microsoft Azure Account: You'll need a valid and active Azure account for the Azure labs.
* You'll need an Azure DevOps account.

## Length
40 minutes

## Exercise 1: Create a new project in Azure DevOps
1. Enter in [Azure DevOps](http://dev.azure.com) and log in clicking in **Start free** button

![](images/1.png)

2. Login with the same credentials you were given for Azure if you are not logged in already.
3. Select `+ New Project`
4. Set the **Project name** you want
5. In **visibility** select **Private**
6. Click on **Advanced**, in **Version control** select **GIT** and in **Work item process** select **Scrum** then select on `Create` 

![](images/2.png)

5. Once your project has been created, select `Repos` from left-pane menu
6. In Repos page, select `Import` from 'or import a repository' option
7. in **Clone URL** option put this URL then click on import
``https://github.com/MSTecs/Azure-DevDay-lab4-demoProject.git``

![](images/4.png)

8. Now if you click again on files in the left-pane menu, you will see the code in your page

![](images/5.png)


## Exercise 2: Configure Continuous Integration
### Task 1: Configure your basic Pipeline DevOps
1. Log into the Azure portal with your assigned credentials  

- **Note** If you are using your own subscription, begin the deployment of Module 3 resources using this deployment script. Be sure to use lowercase letters with your initials Eg. - tjbmodule3.  Click the Deploy to Azure button to start <a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FMicrosoft%2Falmvm%2Fmaster%2Flabs%2Fvstsextend%2Fdocker%2Farmtemplate%2Fazuredeploy.json" target="_blank"><img src="http://azuredeploy.net/deploybutton.png" alt="Deploy to Azure"></a>

2. Select `Resource groups` and then select the resource group 'Lab-3-xxxxx'
![](images/6.png)

3. Select the resource of type 'Container registry' and take note of the `Login server` in the properties of the `Overview` menu - you'll need these values later.
![](images/7.png)

![](images/8.png)
4. Go back to your resources open your **SQL database** and take note of the **Server name**

![](images/9.png)

![](images/10.png)

5. Return to <a href="https://dev.azure.com">Azure DevOps</a>

6. Select `Pipelines` from the left-pane menu and then select `Builds`.  Select `New pipeline`

![](images/11.png)

7. Select `Use the classic editor`to create a pipeline without YAML option at the bottom of the page

![](images/12.png)

8. Select `Azure Repos Git` and select your project and click on Continue

![](images/13.png)

9. Select the `Empty job` option at the top of the page

![](images/14.png)

10. Select `Agent job 1` and change 'Display name' to 'Docker'
11. Scroll down to the 'Execution Plan' section, and 'Job cancel timeout' to 1
12. From the top menu, select the dropdown for `Save & queue` and choose `Save`; save using the default values

![](images/15.png)
**Your configuration should look like this**

![](images/16.png)

### Task 2: create your Run Services

1. Select `+` next to your Docker agent on the left-pane

![](images/17.png)

2. Search for 'docker compose' and select the first option, and then select `Add`

![](images/18.png)

3. Select the new item from the left-pane and the `Display name` to 'Run services' and select your azure subscription

![](images/19.png)

4. Select `Authorize`  to create a service connection to Azure. 

    If you receive an error authorizing the connection, Select `Advanced options` in the Authorize dropdown and follow the insturctions below

    ![](images/20.png)

    1. Click on **use the full version of the service connection dialog** option

    ![](images/21.png)

    2. Set `serviceConnection` on **Connection name** input.

    3. Put your **Service Principal Details** given at the beginning of the lab and click in **Ok**
        - **Application/Client Id**
        - **Application Secret Key**

        ![](images/22.png)

5. Select your container registry

![](images/23.png)

6. Select on ellipsis button (**...**) and search the file **docker-compose.ci.build.yml**

![](images/24.png)
![](images/25.png)

7. Scroll down to **Action** configuration and select **Run service images** on the dropdown
8. Uncheck the **Run in Background** option
9. Scroll down to **Output Variables** and set **Reference name** to 'Task1'
10. From the top menu, select the dropdown for `Save & queue` and choose `Save`; save using the default values

![](images/26.png)

### Task 3: create your Build services
1. From the left-pane menu, select the `+` button and search again for 'Docker compose' and select `Add`

![](images/27.png)
![](images/18.png)

2. Set **Display nae** to 'Build services'
3. Select your previously created service connection on **Azure subscription**
4. Select your **Azure Container registry**
5. Select the `...` button next to **Docker Compose File** and select the **docker-compose.yml** file
6. Enter 'DOCKER_BUILD_SOURCE=' in the **Environment Variables** dialogue

**Your config should look like this**
![](images/28.png)

7. Scroll down to **Action** and select **Build service images** from the dropdown
8. Scroll down to **Output Variables** and enter `Task2` on **Reference name**
9. From the top menu, select the dropdown for `Save & queue` and choose `Save`; save using the default values

![](images/29.png)

### Task 4: Create your Push services
1. Select the `+` plus button from the left-pane menu and search again for 'Docker compose' and select `Add`

![](images/30.png)
![](images/18.png)

2. Set **Display name** to 'Push services'
3. Select your previously created service connection on **Azure subscription**
4. Select your **Azure Container registry**
5. Select the `...` button next to **Docker Compose File** and select the **docker-compose.yml** file
6. Enter 'DOCKER_BUILD_SOURCE=' in the **Environment Variables** dialogue

**Your config should look like this**
![](images/31.png)

7. Scroll down to **Action** and select **Push service images** from the dropdown
8. Scroll down to **Output Variables** and enter `Task3` on **Reference name**
9. From the top menu, select the dropdown for `Save & queue` and choose `Save`; save using the default values

![](images/32.png)

### Task 5: create your Publish Artifact
1. Select the `+` plus button from the left-pane menu and search for 'Publish build artifacts' and select `Add`

![](images/33.png)
![](images/34.png)

2. Set **Display name** to 'Publish artifacts'
3. Set **Path to publish** to 'myhealthclinic.dacpac'
4. Set **Artifact name** to 'dacpac'
5. Scroll down to **Output Variables** and enter 'PublishBuildArtifacts2'
6. From the top menu, select the dropdown for `Save & queue` and choose `Save`; save using the default values

![](images/35.png)


### Task 6: Set your variables
1. Select `Variables` from top menu
2. Create these variables by selecing the `+ Add` button

- Name: BuildConfiguration Value: Release.  Check the `Settable at queue time` option
- Name: BuildPlatform Value: Any CPU
3. From the top menu, select the dropdown for `Save & queue` and choose `Save`; save using the default values

![](images/36.png)

### Task 7: config your triggers
1. Select `Triggers` from top menu
2. Check the `Enable continuous integration` option
3. From the top menu, select the dropdown for `Save & queue` and choose `Save`; save using the default values

![](images/37.png)

### Task 8: config your options
1. Select `Options` from top menu
2. On the left-pane, set `Build number format` to '$(date:yyyyMMdd)$(rev:.r)'
3. On the right-pane, set `Build job timeout in minutes` to '0'
4. From the top menu, select the dropdown for `Save & queue` and choose `Save`; save using the default values


![](images/38.png)

## Task 9: Set your host
1.Select `Tasks` from top menu
2. Select `Pipeline`
3. Set `Agent Pool` to 'Azure Pipelines' from dropdown
4. Set `Agent Specification` to  'Ubuntu 16.04' from dropdown 
5. From the top menu, select the dropdown for `Save & queue` and choose `Save`; save using the default values

![](images/41.png)

## Exercise 3: configure your Continuous Delivery
### Task 1: create the pipeline
1. From the left-pane menu, select `Pipelines` and then select `Releases` 
2. Select `New pipeline` and select `Empty job` from top of page

![](images/42.png)
![](images/43.png)

3. Set **Stage name** to 'Dev'

![](images/44.png)

4. Select `+ Add an artifact` from left-pane **Artifacts** section, and add the listed values from picture below

![](images/44A.png)

5. Select the `Continuous deployment trigger` and `Enable` **Continuous deployment trigger** on the right-pane
6. Select `Save` from top menu and save using default values

![](images/44b.png)

### Task 2: Config your DB deployment
1. From the top menu, select `Tasks` 
2. Select `Agent job`
3. Set **Display name** to 'DB Deployment'
4. Select the `+ Add` button and create the variable
    **Name:** sqlpackage **Condition:** exists
5. Select `+` next to `DB Deployment` in left-pane and search for 'azure SQL database deployment' and select `Add`

![](images/45.png)

7. From the left-pane, select `Azure SQL Dacpac Task`
8. Set **Display name** to 'Execute Azure SQL : DacpacTask'
9. Set **Azure Subscription** to your previously configured service service connection
10. Set **Azure SQL Server** to '$(SQLserver)' 
11. Set **Database** to '$(DatabaseName)'
12. Set **Login** to ' $(SQLadmin)' (with a blank space at the beginning)
13. Set **Password** to '$(Password)' 

![](images/46.png)

14. Scroll down to the section **Deployment Package**, and set **DACPAC File** to '$(System.DefaultWorkingDirectory)/**/*.dacpac' 
15. Scroll down to the section **Output Variables**, and set **Reference name** to 'SqlAzureDacpacDeployment1'
16. Select `Save` from the top and accept its defaults

![](images/47.png)

## Task 3: Config your Web App deployment
1. From the left-pane, select `...` next to `Dev` and select `Add an Agent Job`

![](images/48.png)

2. Click on your new `Agent job` 
3. Set **Display name** to 'Web App deployment'  
4. Set **Agent pool** to 'Hosted Ubuntu 1604'  from the dropdown
5. Select `Save` from the top and accept its defaults

![](images/49.png)

1. Select `+` next to `Web App deployment` in the left-pane and search for 'Azure App Service Deploy' and select `Add`

![](images/50.png)

*** THIS SECTION NEEDS TO BE FIXED ***
2. Select your new `Azure App Service Deploy:` from the left-pane
4. Set **Task version** to  '3.*'  from the dropdown
5. Set **Display name** to 'Azure App Service Deploy' 
6. Set **Azure subscription** to your previously created service connection 
7. Set **App type** to  'Web App on Linux' from the dropdown
8. Set **App Service Name** to 'webapp' 
9. Set `$(ACR)` on **Registry or Namespace** input

![](images/51.png)

10. Put `myhealth.web` in the **Image** input
11. Put `$(BUILD.BUILDID)` in **Tag** input
12. Put `AzureRmWebAppDeployment1` in **Reference name** on **Output Variables** section
13. Select `Save` from the top menu

![](images/52.png)

## Task 4: Config your variables
1. Select `Variables` from the top menu
2. Select `+ Add` and enter the following

- **Name:** ACR **Value:** YOUR_ACR.azurecr.io **Scope:** Release
- **Name:** DatabaseName **Value:** mhcdb **Scope:** Release
- **Name:** Password **Value:** P2ssw0rd1234 **Scope:** Release
- **Name:** SQLadmin **Value:** sqladmin **Scope:** Release
- **Name:** SQLserver **Value:** YOUR_DBSERVER.database.windows.net **Scope:** Release

![](images/53.png)

3. Select `Save` from the top and accept its defaults

![](images/54.png)


## Exercise 4: Initiate the CI Build and Deployment through code commit

1. From the left-pane menu, select `Repos` and then `Files`.  Navigate to the (ProjectName)/src/MyHealth.Web/Views/Home folder and open the Index.cshtml file for editing

![](images/55.png)

2. Modify the text 'JOIN US' to 'CONTACT US' on line 28, and then select `Commit` button accepting its default values.  This action would initiate an automatic build for the source code

![](images/56.png)

3. Select `Pipelines` and then `Builds` from the left-pane menu then select the commit name just saved from the right-pane

![](images/58.png)

![](images/59.png)

4. The Build will generate and push the docker image of the web application to the Azure Container Registry. Once the build is completed, the build summary will be displayed

![](images/60.png)

5. Open the Azure Portal App Service that was created at the beginning of this lab. From the left-pane menu, select `Container Settings` and provide the information per the pictures below and then click the `Save` button

![](images/61.png)

![](images/62.png)

![](images/63.png)

6. From the Azure Portal, select your 'Azure Container Registry' service and then select `Repositories` option to view the generated docker images

![](images/64.png)

7. From Azure Dev Ops portal, select `Pipelines` and then `Releases`. Select  the latest release and select on `Logs` to view the details of the release in progress

**note** In case doesn´t exist any release you can create a new one clicking on **create a release** and selecting the **Dev** from the pipeline

![](images/65.png)


8. From teh Azure Portal, select your app service and then `Overview` from the left-pane meny. Select the link next to the `URL` field to browse the application

9. Use the credentials **Username**: user and **Password**: P2ssw0rd@1 to login to the HealthClinic web application

![](images/69.png)

End of the lab
