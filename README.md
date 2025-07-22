# Zowe-based Db2 z/OS tools
Worked examples for deploying IBM Db2 z/OS Tools inside the Zowe project.

IBM's Db2 tools have evolved to embrace a unified browser-based user interface, and integration of tooling services with RESTful APIs. 
These innovations are based around the Zowe foundation.
There is no sudden migration here.
The new paradigm augments the existing 3270 and JCL ways of working, allowing a choice of tooling interface.

This repository is a worked example of implementing the Zowe-based Db2 tools in a simple environment to encourage Db2 z/OS sites to deploy the new facilities.

# The Journey
The image below represents the journey that many DB2 z/OS sites will be making.

![thejourney](/images/thejourney.jpg)


The starting position of many Db2 z/OS sites will be similar to the left side of the diagram.

*  Most of the Db2 z/OS tools will be using 3270 interfaces, and batch JCL suites.
*  Client tools providing graphical user interfaces were usually unavailable to many sites that outsource desktop software management.
*  Sites that used Data Studio for graphical UI and developing stored procedures will know that Data Studio is no longer supported.
*  Data Server Manager has also been withdrawn from support.
*  Visual Explain tooling has been re-packaged as a z/OS liberty based SQL tuning server, accessible via Zowe or VS Code.

IBM has delivered new Db2 z/OS tooling interfaces since the Zowe project was released, as represented on the right side of the diagram.
* Zowe is a framework that provide API-drive integration for tools, and supports browser-based user interfaces directly from z/OS. 
* Db2 tools are deployed inside the Zowe framework to provide a unified UI experience, and integration via RESTful APIs.
* No more client or server installations are needed for DBAs: Everything runs on z/OS.
* Developers can access these database services from the ubiquitous VSCODE IDE.

There are two Graphical User Interfaces, aimed at different Db2 roles.
* The Db2 tooling in Zowe environment is targetted firmly at the DBA role.
* The Db2 tooling in the VSCODE environment is targetted firmly at the developer role.

The pages in this github repository provide a worked example for the 7 steps needed to add modern UI and integrations to the Db2 tools.


## Step 1. Pre-Requisite Maintenance

[Pre-Requisite Maintenance](https://github.com/zeditor01/zowe_db2_tools/blob/main/docs/pre-requisite_maintenance.md) describes software maintenance levels and PTFs for Db2 z/OS and DB2 Administration tool, to support the Zowe Db2 tools.   

## Step 2. Deploying Zowe


## Step 3. Deploying Unified Management Server


## Step 4. Deploying Db2 Administration Foundation


## Step 5 Deploying SQL Tuning Services


## Step 6. Deploying Db2 for z/OS Developer Extension in VSCODE


## Step 7. Deploying DB2 Automation Expert.



 
