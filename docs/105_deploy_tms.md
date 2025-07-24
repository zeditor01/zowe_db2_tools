[Back to Home Page](https://github.com/zeditor01/zowe_db2_tools/tree/main)

# Step 5 Deploying SQL Tuning Services

SQL Tuning Services for Db2 z/OS is independent of the Zowe infrastructure, but can be accessed from Db2 Administration Foundation by configuring access to the SQL Tuning Server's RESTful APIs.

This page will cover the deployment of SQL Tuning Services Liberty Server, and AT-TLS connection from Db2 Administration Foundation.
![stage3a](/images/zowe_tms.jpg)


Contents
1. Two options to get SQL Tuning Services
2. Install Query Workload Tuner
3. Deploy Query Workload Tuner
4. Operating the SQL Tuning Server
5. Connection from Db2 Administration Foundation to SQL Tuning Services


## 5.1 Two options to get SQL Tuning Services
IBM provides a base capability for SQL Tuning as part of Db2 z/OS V13. This is a no charge feature, but it (Database Services Expansion Pack) must be ordered from ShopZ. It includes Visual Explain, Statistics Advisor and the ability to Capture the Query environment for IBM Support cases.

IBM offers an optional licensed tool (Query Workload Tuner) with a number of advanced tuning capabilities. ( advisors for indexes and access paths, comparison of access paths, virtual index analyzer, entire SQL workload tuning and more). A full comparision of features is provided in the Db2 knowledgecenter [here](https://www.ibm.com/docs/en/db2-for-zos/13.0.0?topic=services-overview)

Decide which option you want to install, and order the approriate package from Shop, as show below.

Order Database Services Expansion Pack from Shopz
![order_tms](/images/order_tms.jpg)

Order Query Workload Tuner from Shopz
![order_qwt](/images/order_qwt.jpg)

This worked example using Query Workload tuner.

## 5.2 Install Query Workload Tuner
This is a standard portable software instance installation, which will not be covered in the main page. 
## 5.3 Deploy Query Workload Tuner

## 5.4 Operating the SQL Tuning Server

## 5.5 Connection from Db2 Administration Foundation to SQL Tuning Services





