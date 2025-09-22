# Zowe Db2 z/OS tools - Build Path

This is the index page for the hands-on-learning exercise to deploy the complete Zowe Db2 Tools solution. 

The exercises are written to to allow a new-to-Z technical professional to gain practical experience in deploying z/OS software solutions.
* It is ***not*** an education programme to teach everything about the in-scope topic.
* It is a set of exercises aimed at giving practical experience of a complex deployment.

## Acronyms Used throughout this github repository.
* Zowe is an open source framework that provides modern, cloud-like interfaces, integration points and tools for z/OS systems.
* QWT = Query Workload Tuner. (a licensed software product to help SQL tuning)
* PSI = Portable Software Instance. (how we package software in ShopZ and z/OS)
* UMS = Unified Management Server. (a software framework into which Db2 and IMS tools are deployed)
* DAF = Db2 Administration Foundation. (the core Db2 z/OS experience for UMS)
* DAE = Db2 Automation Experience. (the Db2 Experience for automation fo housekeeping tasks)
* DOE = Db2 Devops Experience. (The Db2 Experience for provisioning Db2 database objects for CI/CD development projects)
* CI/CD = Continuous Integration and Continuous Delivery.

The Excerises are structured as follows

## Step 1: Understand the starting point ZPDT image.
1.1 [Assumed ZPDT deployment](https://github.com/zeditor01/zowe_db2_tools/blob/main/docs/1.1%20Assumed%20ZPDT%20Deployment.md)  
1.2 [Assumed z/OS deployed inventory](https://github.com/zeditor01/zowe_db2_tools/blob/main/docs/1.2%20Assumed%20zOS%20Inventory.md)  

## Step 2: Deploy SQL Tuning Services (QWT) for Db2 z/OS.
2.1 [Download QWT Portable Software Instance from ShopZ](https://github.com/zeditor01/zowe_db2_tools/blob/main/docs/2.1%20QWT%20Download.md)<br>
2.2 [Deploy QWT Portable Software Instance to your z/OS system](https://github.com/zeditor01/zowe_db2_tools/blob/main/docs/2.2%20QWT%20PSI%20Deploy.md)<br>
2.3 [Customise QWT, to create an operational Liberty Server for it](https://github.com/zeditor01/zowe_db2_tools/blob/main/docs/2.3%20QWT%20Customize.md)<br>
2.4 [Installation Verification Testing of QWT](https://github.com/zeditor01/zowe_db2_tools/blob/main/docs/qwt_ivp.md)<br>
2.5 [Testing the Db2 for z/OS Developer Extension for VSCODE to invoke SQL Tuning Services](https://github.com/zeditor01/zowe_db2_tools/blob/main/docs/2.5%20QWT%20Usage%20from%20VSCODE.md)<br>

## Step 3: Deploy UMS and DAF underneath Zowe.
3.1 [Download UMS and DAF Portable Software Instance from ShopZ](https://github.com/zeditor01/zowe_db2_tools/blob/main/docs/3.1%20DAF%20UMS%20Download.md)<br>
3.2 [Deploy UMS and DAF Portable Software Instance to your z/OS system](https://github.com/zeditor01/zowe_db2_tools/blob/main/docs/3.2%20DAF%20UMS%20Deploy.md)<br>
3.3 [Customise UMS and DAF, to create an operational Liberty Server for it](https://github.com/zeditor01/zowe_db2_tools/blob/main/docs/3.3%20DAF%20UMS%20Customise.md)<br>
3.4 [Installation Verification Testing of UMS and DAF](https://github.com/zeditor01/zowe_db2_tools/blob/main/docs/3.4%20DAF%20UMS%20Installation%20Verification.md)<br>

  
## Step 4: Configure DAF to utilise the Db2 Administration Tool.
4.1 [Configure Integration with Db2 Administration Tool]()<br>
4.2 [Validate Integration with Db2 Administration Tool]()<br>

## Step 5: Configure DAF to utilise SQL Tuning Services
5.1 [Configure Integration with Query Workload Tuner](https://github.com/zeditor01/zowe_db2_tools/blob/main/docs/5.1%20Configure%20Integration%20with%20QWT.md)<br>
5.2 [Validate Integration with Query Workload Tuner]()<br>

## Step 6: Deploy Db2 Automation Experience
6.1 [Download DAE Portable Software Instance from ShopZ](https://github.com/zeditor01/zowe_db2_tools/blob/main/docs/6.1%20DAE%20Download.md)<br>
6.2 [Deploy DAE Portable Software Instance to your z/OS system](https://github.com/zeditor01/zowe_db2_tools/blob/main/docs/6.2%20DAE%20PSI%20Deploy.md)<br>
3. Customise DAE, to create an operational Liberty Server for it
4. Installation Verification Testing of DAE

## Step 7: Configure DAF to utilise Db2 Automation Experience
1. Configure DAF to access DAE
2. Test the management of housekeeping profiles from DAF

# Step 8: Deploy Db2 Devops Experience
8.1 [Download DOE Portable Software Instance from ShopZ](https://github.com/zeditor01/zowe_db2_tools/blob/main/docs/8.1%20DOE%20Download.md)<br>
8.2 [Deploy DOE Portable Software Instance to your z/OS system](https://github.com/zeditor01/zowe_db2_tools/blob/main/docs/8.2%20DOE%20PSI%20Deploy.md)<br>
3. Customise Db2 Devops, to enable provisioning of DDL objects for development
4. Installation Verification Testing of Db2 Devops

# Step 9: Configure DAF to utilise Db2 Devops Experience
1. Not sure what's imvolved yet
2. or more


