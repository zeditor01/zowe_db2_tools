/ [Home Page](https://github.com/zeditor01/zowe_db2_tools/tree/main)

# Pre-Requisite Maintenance

1. Zowe pre-requisites
2. Unified Management Server pre-requisites
3. Checking if these pre-requisites are installed

## 1. Zowe pre-requisites
The worked examples in this repository are based around Zowe v2.18

The requirements for the current version of Zowe are documented at the [Zowe website](https://docs.zowe.org/stable/user-guide/zos-components-installation-checklist)

Before proceeding you should always check the latest requirements. However, the main System Requirements are listed here
* AXR (System REXX)
* CEA (Common Event Adapter - of z/OSMF)
* CIM (Common Information Model - of z/OSMF)
* CONSOLE and CONSPROF commands must exist in the authorised command table
* Java (V17 or later)
* NodeJS
* TSO Region Size - minimum 65,536 KB
* Userids - OMVS Segment


## 2. Unified Management Server pre-requisites

UMS needs
* z/OS V2.5 or later
* ICSF
* Java 17 
* RACF (or other SAF)
* minimum versions of ZOWE
* and a number of PTF levels are documented at the link above.

Check the [UMS Knowledge Center](https://www.ibm.com/docs/en/umsfz/1.2.0?topic=installation-prerequisite-hardware-software) for the current list of pre-requisites.


Some of the functionality of Unified Management Server and Db2 Admin Foundation depends on whether other tools are installed.
When this is the case, there will be a "tools discovery" PTF to the underlying tool, so that it's function may be identified and invoked.

A good example is the Storage tab for tablespaces and indexspaces. This depends on 
* the PTF for [APAR PH55177](https://www.ibm.com/support/pages/apar/PH55177) in Db2 Administration Tool version 13 to provide discovery service
* the PTF for [APAR PH54968](https://www.ibm.com/support/pages/apar/PH54968) in Db2 Administration Foundation to ask for discovery


## 3. Checking if these pre-requisites are installed


[This Page](https://github.com/zeditor01/zowe_db2_tools/blob/main/docs/x101_pre-requisites_tasks.md) provides worked examples of how to check whether the pre-requisites are satisfied on a z/OS system.
