Back to / [Repository Home Page](https://github.com/zeditor01/zowe_db2_tools/tree/main) / [Pre-Requisites Page](https://github.com/zeditor01/zowe_db2_tools/blob/main/docs/101_pre-requisites.md)


# Checking if these pre-requisites are installed

Experienced systems programmers will know how to check all these pre-requisites. 

The target audience for this github repository includes novice z/OS persons. The remainder of this section is aimed at novice systems programmers who may want a little guidance on how to check these prequisites on their system.

## 1 AXR (System REXX)
This is a standard component of z/OS, and should be present. Check it by issuing console command ```d a,axr``` and check the system log for positive confirmation, like below.

![checkaxr](/images/check_axr.jpg)

If system REXX is installed, but not active, you can start it with console command ```START AXRPSTRT```

## 2 CEA (Common Event Adapter - of z/OSMF)
CEA should be present. The Common Event Adapter (CEA) is an essential component for many z/OSMF (z/OS Management Facility) services, providing event delivery and the ability to manage TSO/E user address spaces. Check it by issueing console command ```d a,cea``` and check the system log for positive confirmation.

![checkcea](/images/check_cea.jpg)

You can start CIM with the z/OS console command ```F CEA,MODE=FULL```

## 3 CIM (Common Information Model - of z/OSMF)
CIM should be present. The CIM component (Common Information Model) is a foundational element for z/OSMF, enabling a standardized way to manage and monitor z/OS systems using industry data models and APIs. Check it by issueing console command ```D A,CFZCIM``` and check the system log for positive confirmation.

![checkcfzcim](/images/check_cfzcim.jpg)

You can start CIM with the z/OS console command ```S CFZCIM```


## 4 CONSOLE and CONSPROF commands must exist in the authorised command table


The Authorized Command Table is typically a member in SYS1.PARMLIB named IKJTSOxx

These members define which TSO/E commands can run authorized. View the table using ISPF dataset browsing. My applicable PARMLIB member is USER.Z31C.PARMLIB(IKJTSOZZ) as shown below with CONSPROF listed as an authorized TSO command. I confess that CONSOLE is not listed as aan authorised command, but doesn't seem to have caused a problem so far.

![ikjtsozz](/images/ikjtsozz.jpg)


## 5 Java (V17 or later)

Open a USS shell ( for example with ssh from a terminal client ) and enter the command ```java -version```.

![checkjava](/images/check_java.jpg)

The example above only shows Java V11. Java 17 may be installed on the z/OS system, but may not be set in my shell profile.

You can see which versions of java are installed by checking ```/usr/lpp/IBM/java```

![java_versions](/images/java_versions.jpg)

You can check the environment parameters for any user by executing the ```env``` command 

![java_env](/images/java_env.jpg)

You can control the environment parameters for any user by editing their .profile

![javaprofile](/images/java_profile.jpg)


## 6 NodeJS

Check the install path. NodeJS is installed by convention in path ```/usr/lpp/IBM/cnj/```


## 7 TSO Region Size - minimum 65,536 KB

Logon to the RACF Panels, and select user profiles, and display the user profile you want to.

![racf01](/images/racf01.jpg)

check the segments that you wish to view ( e.g. TSO and OMVS )

![racf02](/images/racf02.jpg)

The report shows details of the TSO and OMVS segments, including max regiion size

![racf03](/images/racf03.jpg)


## 8 Userids - OMVS Segment

The previous check showed the existence of an OMVS segment, and the properties of that segment.

## 9 ICSF

The Integrated Cryptographic Service Facility (ICSF) is a software component of z/OS providing cryptographic APIs and services that leverage IBM mainframe cryptographic hardware.
Check it by issueing console command ```D A,CSF``` and check the system log for positive confirmation.

![checkcsf](/images/check_csf.jpg)


## 10 PTFs for DB2 Administration Tool
Products like Db2 Administration Tool are installed using SMP/E ( System Modification Program Extended ). SMPE uses CSI datasets ( Consolidated Software Inventory ) to record which software products and components are installed, and what maintenance has been applied to them.

If you access SMP/E through the ISPF menue, you can run a CSI Query as follows.

From the SMP/E main menu, select "1" and clear the name of the CSI dataset, in order to retrieve a list of all the CSIs in this system.

![smp01](/images/smp01.jpg)

Some of the CSI names will reflect the name of the products that they store SMPE data about. The 3 character component code for Db2 Administration tool is ADB. The CSI zone for Db2 Administration Tool is ```ADBD10.GLOBAL.CSI```

![smp02](/images/smp02.jpg)

Go back to the main SMPE panel, and select option 3 to perform a CSI query

![smp03](/images/smp03.jpg)

CSI datasets have 3 zones. Any code such as PTFs are referred to as SYSMODS. The SMP mainenance process moves these SYSMODS through the three zones in the CSI dataset.
1. SYSMODS are received (from ShopZ) into the Global Zone
2. SYSMODS are applied into the Target Zone (in order to be useable)
3. SYSMODS are accepted into the Distribution Zone, once fully tested (in order for the software to be copied to other LPARs)

Option 2 is a cross zone query, which will show if a SYSMOD exists in any or all 3 zones.

![smp04](/images/smp04.jpg)

So, we enter the APAR number ( or PTF number ) into the cross zone query panel.

![smp05](/images/smp05.jpg)

And we find out whether that SYSMOD is installed in this system.

![smp06](/images/smp06.jpg)

When I found out that PH55177 was not installed in my system, I ordered it from ShopZ. However, when I ran the SMPE jobs to receive the fix, the SMPE process that there were no SYSMODS to apply. I was confused.

It turns out that this installation of DB2 Administration Tool was installed from a code level that was made available after PH55177 was produced, and the function from this PTF is built into the installed product code, even though the SMPE CSI is unaware of it. I was able to check this by reading the details of PH55177, and learning that it places 2 members (ADBDSCVP and ADBDSCVS) into the SADBSAMP PDS. These are the members that support "Discovery" by Db2 Administration Foundation of Services provided by Db2 Administration Tool. We will use these member later in the deployment project.

![smp08](/images/smp08.jpg)




