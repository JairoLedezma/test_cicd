# Table of Contents
<!--ts-->
   * [Sample DMN Project Deployment](#sample-dmn-Project)
   * [Openshift CI/CD Pipeline](#openshift-cicd-pipeline)
<!--te-->

# Sample-DMN-Project
- This repository contains the source code for a sample DMN Project which implements the iteration over list of objects on each of which business logic validation is performed.
- It can be deployed directly to a `Immutable Kie-server-v7.11.0` running on a openshift cluster configured for S2I(Source to Image).
- This guide has the template of the kie server in the template folder, aswell as the environment variables that are needed to run it. 
- The jenkins pipeline does all the work when it comes to creating image streams, build configs, deployment configs and secrets. NOTE: the secrets will have to be populated by the user after deployment of application in order for the kieserver to function properly.
## Prerequisites
- OpenShift v4.7.11 Cluster Web Console access with cluster-admin rights<br /><br />
## Installation


 ### Step 1. Setting up the OpenShift Cluster
 - Login to your OpenShift Cluster Web Console or Terminal and create a new project
 	- Due to the limitations of the openshift plugin on jenkins, we have to do this ourselves.

# Openshift-CI/CD-Pipeline

### Prerequisites
- JDK 11

### Setup
- **Step 1: Installing Jenkins on your local machine**
  - The latest version of Jenkins can be found [here](https://www.jenkins.io/). Download the Generic Java package (.war) file. Make sure java version 8 or higher is installed. Java version can be checked by using the following command on a terminal on the local machine:
 
		java --version
    If you do not have JDK installed, the following link can be used
   <b>JDK 11</b>: [link](https://www.oracle.com/java/technologies/javase-downloads.html#JDK11)
- **Step 2: Starting the Jenkins Automation tool**
  - Run the following command to cd into the Downloads folder (or where your jenkins.war file is):

		cd ~/Downloads

  - Run the following command to start the Jenkins Application
		java -jar jenkins.war
		
- **Step 3: Setting up Jenkins**
  - Go to [localhost:8080](http://localhost:8080) to see the homepage of Jenkins, enter the admin password at which point the Jenkins configuration will start. It's a simple one-time admin configuration. Select install suggested plug-ins to get started quickly.

- **Step 4: Installing plugins**
  - Plugins can be installed via **Dashboard**&nbsp;&rarr;&nbsp;**Manage Jenkins**&nbsp;&rarr;&nbsp;**Manage Plugins**
  - Click on "Available" tab and search the following plug-ins
  - To work with OpenShift, following plug-ins are required:
    - OpenShift Client
    - OpenShift Login
    - OpenShift Sync

  - Artifactory Plug-ins:
    - Artfactory

  - To add the functionality of having multiple users with roles and permissions, install:
    - Role-based Authorization Strategy

   Select the above plug-ins and hit **Install without Restart** button.
- **Step 5 Installing and Setting up JFrog Artifactory Server:**

  - Download the OSS (Open Source Solution) version of Artifactory from [here](https://releases.jfrog.io/artifactory/bintray-artifactory/org/artifactory/oss/jfrog-artifactory-oss/[RELEASE]/jfrog-artifactory-oss-[RELEASE]-darwin.tar.gz)
  - Unzip the tar file by double clicking on the downloaded tar.gz file or using the following command:
 `tar -xvf [zip fileName]`
  - After the file is unzipped, open up a terminal and cd into
 `artifactory-oss/app/bin/`
  - If the file is unzipped into the downloads folder, use the following command to cd into the folder:
 `cd ~/Downloads/artifactory-oss-7.21.12/app/bin/`
  - Once you are in the bin folder, run the following command to start the Artifactory server
 `./artifactory.sh start`
  - If you get a malware prompt:
    - Open settings
    - Select security & privacy settings
    - In the general tab, click the lock near the bottom left
    - Note that the “allowed apps” list will update as you run the ./artifactory.sh start command
    - While the program is running, you will be prompted to move to trash, or close. Click close then look at the “allowed apps” list in your settings.  A new entry will appear each time you hit close.  Click “always allow” then restart the ./artifactory.sh start command.  Repeat this process until the installation is complete.

  - It should start the artifactory server on the following URL : [http://localhost:8082](http://localhost:8082). Wait for artifactory to start and be ready to serve

```
Default login credentials:
	Username: admin
	Password: password
```

- **Step 6 Setting up Artifactory server:**
  - Step 1: Set up the admin password (Optional, you can skip if you want to use the default password). Select any easy password that you can remember, for example *Admin123!*
  - Step 2: Set Base URL
    - Skip
  - Step 3: Configure Proxy Server
    - Skip
  - Step 4: Create a Repository/ Repositories (if it shows up)
    - Skip

- **Step 7 Creating Maven Repositories on JFrog Artifactory Server**
  - Go to Administration section(Gear icon in the left navigation pane, on the top)
  - Click on Repositories&nbsp;&rarr;&nbsp;Repositories&nbsp;&rarr;&nbsp;Local (Local tab in the main area)<br />
  - Click Add Repositories from top right corner,
    - Select Local Repository
    - Select Maven
    - Enter Repository key as : libs-release-local
    - Click Save & Finish in the bottom right corner

  - Click Add Repositories from top right corner,
    - Select Local Repository
    - Select Maven
    - Enter Repository key as : libs-snapshot-local
    - Click Save & Finish in the bottom right corner

   **NOTE: Use these credentials when configuring JFrog server in Jenkins**:

   Go to **Administration**&nbsp;&rarr;&nbsp;**Identity and Access**&nbsp;&rarr;&nbsp;**Users**
  - Click on “New User” in the top right corner
    - Username: { username } (example: deployer)
    - Email: { email } (example: deployer@artifactory.com)
    - Roles : Administrator Platform - Checked
    - Password: { password } (example: Password123!)
    - Click Save in the bottom right corner

- **Step 8 Configure Jenkins Global Tool Configuration**
  - This step is to configure tools such as java, maven and openshift client (oc) to be used by pipelines. Tools name given under these settings will be called under the pipeline tool section.

  - Go to Manage Jenkins&nbsp;&rarr;&nbsp;Global Tool Configuration

    - OpenShift Client Tools
      - Click on "Add OpenShift Client Tools"
      - Name : oc
      - Install automatically : checked
      - Click on "Add OpenShift Client Tools
      - Choose "Extract *.zip/*.tar.gz"
      - Label : Leave empty
      - Download URL for binary archive : https://mirror.openshift.com/pub/openshift-v4/clients/oc/latest/macosx/oc.tar.gz
    ** IMPORTANT: Use oc cli whose version matches the openshift cluster**
      - Subdirectory of extracted archive : Leave empty

	  Click **Apply**

    - JDK Installation
      - Click on "Add JDK"
      - Name : jdk11
      - Install Automatically : uncheck
      - JAVA_HOME :
      Use command `echo $JAVA_HOME` which will print out the required path
	  /Library/Java/JavaVirtualMachines/jdk-11.0.12.jdk/Contents/Home/

	  Click **Apply**

    - Maven Installation
      - Click on "Add Maven"
      - Name : maven-3.6.3
      - Install automatically : checked
      - Install from Apache Version : 3.6.3

     Click **Apply & Save**

- **Step 9 : Jenkins Configurations : Configure OpenShift Cluster Details**
   - Go to Manage Jenkins -> Configure System

   - OpenShift Client Plugin
      - Cluster Configurations : Click on "Add OpenShift Cluster"
      - Cluster Name : openshift-cluster
      - API Server URL : *yourOcpClusterUrl*
      - Disable TLS Verify : checked
      - Credentials : Click on "Add"
      	- Kind : "OpenShift Token for OpenShift Client Plugin (2nd option)"
     	- ID : {unique Id}
     	- Token: Get a login token from your Openshift 4 cluster. Insert everything after "token=" up to the space.
      - Click "Add"
      - Credentials : Choose from drop down {unique id}
      - Default-project: Type in name of the namespace you made earlier

   Click on **Apply & Save**

   - JFrog
      - Cluster Configurations : Click on “Add JFrog Platform Instance”
      - Instance Id : localhost-jfrog-server
      - JFrog Platform URL : { localhost-url } (generally http://localhost:8082)
      - Default Deployer Credentials:
      - Username : { JFrog Login Username }
      - Password : { JFrog Login Password }

   Click on **Test Connection** and see if it works, if it does then hit **Apply and Save**

- **Step 10 : Create an OpenShift Pipeline**
  - The pipeline will use parameters to capture git repo, branch, project namespace, openshift cluster name (as configured above earlier) and the Build Configuration.

  - Open Jenkins&nbsp;&rarr;&nbsp;Dashboard &nbsp;&rarr;&nbsp;New Item

  - Creating and Configuring the Pipeline

   - Enter an item name : OpenShift Pipeline
     - Select "Pipeline" & Click "OK"
     - Under General : Check / Select "This project is parameterized"

   - Click on "Add Parameter" & Choose "String Parameter"
     - Name : GIT_URL
     - Default-Value : this git repo

   - Click on "Add Parameter" & Choose "String Parameter"
     - Name : BRANCH
     - Default-Value : master

   - Click on "Add Parameter" & Choose "String Parameter"
     - Name : CLUSTER_NAME
     - Default Value : openshift-cluster
**IMPORTANT - This should match with earlier defined cluster name**

   - Click on "Add Parameter" & Choose "String Parameter"
     - Name : PROJECT_NAME
     - Default Value : < name of your namespace on openshift >

   - Click on "Add Parameter" & Choose "String Parameter"
     - Name : BUILD_CONFIG
     - Default Value : kie-server-kieserver

   - Click on "Add Parameter" & Choose "String Parameter"
     - Name : DEPLOYMENT_CONFIG
     - Default Value : kie-server-kieserver
     - 
   - Click on "Add Parameter" & Choose "Boolean Parameter"
     - Name : NEW_PROJECT
     - Default Value: checked ✅ 
    

  - Under Pipeline Section
     - Select Pipeline from SCM in the definition section.
     - Select Git in SCM
     - Enter this git repo url in the github URL
     - GIT_URL : https://github.com/JairoLedezma/ocp4-cicd-pipeline.git
     - Credentials : none (for public repositories)

     - ScriptPath: Jenkinsfile
     - Lightweight Checkout : selected

   Hit **Apply & Save**

Run the pipeline:

 - Click on Build with Parameters
  - IT_URL : https://github.com/JairoLedezma/ocp4-cicd-pipeline.git
  - BRANCH : master
  - CLUSTER_NAME : openshift-cluster
  - PROJECT_NAME : < your openshift namespace >
  - BUILD_CONFIG : kie-server-kieserver
  - DEPLOYMENT_CONFIG : kie-server-kieserver
  - NEW_PROJECT : checked
  	- NOTE : If you made changes to the template-replace.yaml file uncheck this to run again. This triggers a replace function to replace the old config file with a new one refecting your changes.	
 		
 
Click on **Build**

A succesful build will look like this

![Screen Shot 2021-08-23 at 1 39 26 PM](https://user-images.githubusercontent.com/61709408/130500191-11d94030-1c71-44ea-b1a0-356cc837a411.png)

## We can now verify that our our new application is available in our openshift web console.

![Screen Shot 2021-08-23 at 1 48 41 PM](https://user-images.githubusercontent.com/61709408/130501379-f1e669e0-2dbc-48be-9872-626b3a14ea96.png)

- Note: For the kie server to function correctly, after finishing the build on jenkins go to your web-console and under the project where the application was deployed to, select the secrets tab. The template added the secrets (credentials, https-keystore) but, it did not populate them. This is information that is unique to your build. If you did not make changes to any of the yaml files the go to secrets and from the drop down select edit secret then use the following credentials:
```
Credentials
	KIE_ADMIN_PWD = kie-redhat1!
	KIE_ADMIN_USER = kie-admin
```

Creating a secret for out https-keystore is more complicated than just adding what the usernames are. For this secret, the kieserver is looking for a certificate. We can accomplish this by logging into our openshift cluster through the terminal. Grab a login token from your cluster and follow along with the following commands.
```
oc project <project name>
```
- We are now in the project namespace
- We create a new keystore.jks file in the next step. NOTE: this is added to your local machine not OCP4 cluster.
- The `-storepass` is the password for our keystore, by default, the password is set to `mykeystorepass`
```
keytool -genkeypair -alias jboss -keyalg RSA -keystore keystore.jks -storepass mykeystorepass --dname "CN=jsmith,OU=Engineering,O=mycompany.com,L=Raleigh,S=NC,C=US"
```
-Generate a certificate signing request.

Run the following command to generate a certificate signing request using the public key from the keystore you created in previous step
```
keytool -certreq -keyalg RSA -alias jboss -keystore keystore.jks -file certreq.csr
```
Next we must create an SSL certificate for HTTP access to KIE Server, Business central, smart router, https-keystore and provide it to your OpenShift environment as a secret. Run the following commnads to accomplish this. Ensure that your are on the correct project as these secrets will be only for this project.
```
oc create secret generic kieserver-app-secret --from-file=keystore.jks
```
```
oc create secret generic decisioncentral-app-secret --from-file=keystore.jks
```
```
oc create secret generic smartrouter-app-secret --from-file=keystore.jks
```

- Delete the empty https-keystore under the secret tab. Generete the new secret with the following command.
```
oc create secret generic https-keystore --from-file=keystore.jks
```
	
## Our kie server is now ready to be built again. On the web console, click on our kie-server and select start build. After the build finishes, click on start rollout. This will bring up our pods that the kieserver runs on. We can now acess our kie server!
![Screen Shot 2021-08-23 at 2 49 58 PM](https://user-images.githubusercontent.com/61709408/130510708-ae83c63f-2d54-4227-9004-be3544e99ecb.png)
![Screen Shot 2021-08-23 at 2 50 53 PM](https://user-images.githubusercontent.com/61709408/130510719-4bc7e942-18da-4499-beb9-82fb6a28bcf4.png)




