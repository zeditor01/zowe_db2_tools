# Zowe Db2 z/OS tools - Build Path

This is the index page for the hands-on-learning exercise to deploy the complete Zowe Db2 Tools solution. 

The exercises are written to to allow a new-to-Z technical professional to gain practical experience in deploying z/OS software solutions.
* It is ***not*** an education programme to teach everything about the in-scope topic.
* It is a set of exercises aimed at giving practical experience of a complex deployment.

## Acronyms Used throughout this github repository.
* Zowe is an open source framework that provides modern, cloud-like interfaces, tools, and mainframe environments, specifically on z/OS systems.
* QWT = Query Workload Tuner. (a licensed software product to help SQL tuning)
* PSI = Portable Software Instance. (how we package software in ShopZ and z/OS)
* UMS = Unified Management Server. (a software framework into which Db2 and IMS tools are deployed)
* DAF = Db2 Administration Foundation. (the core Db2 z/OS experience for UMS)
* DAE = Db2 Automation Experience. (the Db2 Experience for automation fo housekeeping tasks)
* Db2 Devops = Db2 Devops Experience. (The Db2 Experience for provisioning Db2 database objects for CI/CD development projects)
* CI/CD = Continuous Integration and Continuous Delivery.

The Excerises are structured as follows

## Step 1: Understand the starting point ZPDT image.
* Assumed ZPDT deployment
* Assumed z/OS deployed inventory

## Step 2: Deploy SQL Tuning Services for Db2 z/OS.
* Download QWT Portable Software Instance from ShopZ
* Deploy QWT Portable Software Instance to your z/OS system
* Customise QWT, to create an operational Liberty Server for it
* Installation Verification Testing of QWT
* Testing the Db2 for z/OS Developer Extension for VSCODE to invoke SQL Tuning Services

## Step 3: Deploy UMS and DAF underneath Zowe.
* Download UMS and DAF Portable Software Instance from ShopZ
* Deploy UMS and DAF Portable Software Instance to your z/OS system
* Customise UMS and DAF, to create an operational Liberty Server for it
* Installation Verification Testing of UMS and DAF
  
## Step 4: Configure DAF to utilise the Db2 Administration Tool.
* Check the pre-requisite maintenance for Db2 Administration Tool.
* Configure access to the integration load libraries
* Test the additional function from DAF

## Step 5: Configure DAF to utilise SQL Tuning Services
* Configure DAF to access QWT
* Test the invocation of SQL Tuning Services from DAF.

## Step 6: Deploy Db2 Automation Experience
* Download DAE Portable Software Instance from ShopZ
* Deploy DAE Portable Software Instance to your z/OS system
* Customise DAE, to create an operational Liberty Server for it
* Installation Verification Testing of DAE

## Step 7: Configure DAF to utilise Db2 Automation Experience
* Configure DAF to access DAE
* Test the management of housekeeping profiles from DAF

# Step 8: Deploy Db2 Devops Experience
* Download Db2 Devops Portable Software Instance from ShopZ
* Deploy Db2 Devops Portable Software Instance to your z/OS system
* Customise Db2 Devops, to enable provisioning of DDL objects for development
* Installation Verification Testing of Db2 Devops

# Step 9: Configure DAF to utilise Db2 Devops Experience
* Not sure what's imvolved yet


