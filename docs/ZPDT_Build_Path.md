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
* DEV = Db2 Devops Experience. (The Db2 Experience for provisioning Db2 database objects for CI/CD development projects)
* CI/CD = Continuous Integration and Continuous Delivery.

The Excerises are structured as follows

## Step 1: Understand the starting point ZPDT image.
* Assumed ZPDT deployment
* Assumed z/OS deployed inventory

## Step 2: Deploy SQL Tuning Services for Db2 z/OS.
* Download Query Workload Tuner Portable Software Instance from ShopZ
* Deploy Query Workload Tuner Portable Software Instance to your z/OS system
* Customise Query Workload Tuner, to create an operational Liberty Server for it
* Installation Verification Testing of Query Workload Tuner
* Testing the Db2 for z/OS Developer Extension for VSCODE to invoke SQL Tuning Services

## Step 3: Deploy UMS and DAF underneath Zowe.

## Step 4: Configure DAF to utilise the Db2 Administration Tool.

## Step 5: Configure DAF to utilise SQL Tuning Services

Deploy Db2 Automation Experience

Configure DAF to utilise Db2 Automation Experience

Deploy Db2 Devops Experience

Configure DAF to utilise Db2 Devops Experience
