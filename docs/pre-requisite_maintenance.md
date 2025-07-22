[Back to Home Page](https://github.com/zeditor01/zowe_db2_tools/tree/main)

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

Experienced systems programmers will know how to check all these pre-requisites. The remainder of this section is aimed at novice systems programmers who may want a little guidance on how to check these prequisites on their system.

### 3.1 AXR (System REXX)
This is a standard component of z/OS, and should be present. Check it by issuing console command ```d a,axr``` and check the system log for positive confirmation, like below.

![checkaxr](/images/check_axr.jpg)

If system REXX is installed, but not active, you can start it with console command ```START AXRPSTRT```

### 3.2 CEA (Common Event Adapter - of z/OSMF)
CEA should be present. The Common Event Adapter (CEA) is an essential component for many z/OSMF (z/OS Management Facility) services, providing event delivery and the ability to manage TSO/E user address spaces. Check it by issueing console command ```d a,cea``` and check the system log for positive confirmation.

![checkcea](/images/check_cea.jpg)

You can start CIM with the z/OS console command ```F CEA,MODE=FULL```

### 3.3 CIM (Common Information Model - of z/OSMF)
CIM should be present. The CIM component (Common Information Model) is a foundational element for z/OSMF, enabling a standardized way to manage and monitor z/OS systems using industry data models and APIs. Check it by issueing console command ```D A,CFZCIM``` and check the system log for positive confirmation.

![checkcfzcim](/images/check_cfzcim.jpg)

You can start CIM with the z/OS console command ```S CFZCIM```


### 3.4 CONSOLE and CONSPROF commands must exist in the authorised command table


The Authorized Command Table is typically a member in SYS1.PARMLIB named IKJTSOxx

These members define which TSO/E commands can run authorized. View the table using ISPF dataset browsing. My applicable PARMLIB member is USER.Z31C.PARMLIB(IKJTSOZZ) as shown below with CONSPROF listed as an authorized TSO command. I confess that CONSOLE is not listed as aan authorised command, but doesn't seem to have caused a problem so far.

![ikjtsozz](/images/ikjtsozz.jpg)


### 3.5 Java (V17 or later)

Open a USS shell ( for example with ssh from a terminal client ) and enter the command ```java -version```.

![checkjava](/images/check_java.jpg)

The example above only shows Java V11. Java 17 may be installed on the z/OS system, but may not be set in my shell profile.

You can see which versions of java are installed by checking ```/usr/lpp/IBM/java```

![java_versions](/images/java_versions.jpg)

You can check the environment parameters for any user by executing the ```env``` command 

![java_env](/images/java_env.jpg)

You can control the environment parameters for any user by editing their .profile

![javaprofile](/images/java_profile.jpg)


### 3.6 NodeJS


### 3.7 TSO Region Size - minimum 65,536 KB


### 3.8 Userids - OMVS Segment


### 3.9 ICSF


### 3.10 RACF (or other SAF)


### 3.11 PTFs for DB2 Administration Tool







