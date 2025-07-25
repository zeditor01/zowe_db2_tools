[Back to Home Page](https://github.com/zeditor01/zowe_db2_tools/tree/main)

# Step 3. Deploying Unified Management Server

After installing Zowe, you will have a landscape like this.

![stage2](/images/zowestage2.jpg)

Now we wish to Deploy the Unified Management Server and Db2 Administration Foundation into Zowe, as illustrated below. 
UMS is implemented as a Zowe server extension and plug-in, integrating seamlessly into the Zowe Application Framework and desktop
UMS leverages the Zowe platform for UI, authentication, and REST services, appearing as a native component of the Zowe ecosystem.

![stage3](/images/zowestage3.jpg)

This page is a worked example of the following steps to deploy UMS into Zowe.
1. install UMS and DAF code. (it makes sense to install DAF and UMS together).
2. edit ZWEYAML parmlib member to configure UMS to integrate with z/OS and Zowe'
3. Execute the UMS installation workflows (setting up RACF artefacts, and integrating ZWEYAML with zowe.yaml).
4. start the zowe server (and likely debug initial UMS startup problems).
6. test Zowe from a web browser.
   

## Notes on using this page.
This page is a simple worked example. It aims to communicate the key concepts and actions to experienced systems programmers. A more detailed audit trail of steps is available in [this supplementary page](https://github.com/zeditor01/zowe_db2_tools/blob/main/docs/x103_deploy_ums_tasks.md) 

## 2.1 install UMS and DAF code. (it makes sense to install DAF and UMS together).

The first task is to order Unified Management Server and Db2 Admin Foundation together on ShopZ. The PIDs to order are shown below.

![order_ums_daf](/images/order_ums_daf.jpg)


Follow the standard PSI installation workflows. My HLQ was "DAFUMS". I ended up with the following datasets after the SMPE installation.
* The DAFUMS.AFX.** datasets are the target and distribution libraries for Db2 Administration Foundation.
* The DAFUMS.CPAC.** and DAFUMS.SMPE.** datasets are the SMPE assets.
* The DAFUMS.IZP.** datasets are the target and distribution libraries for Unified Management Server.
* The DAFUMS.IZP.I1.** datasets are datasets supporting an instance of UMS. (created by post-SMPE installation workflows covered later)
* The DAFUMS.OMVS.** datasets are the ZFS filesystems for Unified Management Server and Db2 Administration Foundation. 


![dafums_datasets](/images/dafums_datasets.jpg)


The PSI workflows will create the USS paths and dynamically mount them. You will be required to permanently mount two ZFS filesystems as follows.

* UMS provides a ZFS called DAFUMS.OMVS.SIZPROOT - to mounted at /usr/lpp/IBM/izp/v1r2m0/bin
* DAF provides a ZFS called DAFUMS.OMVS.SAFXROOT - mounted at /usr/lpp/IBM/afx/v1r2m0/bin

![izpafx_mounts](/images/izpafx_mounts.jpg)

UMS Zowe plug-ins require Program Control authorization. In order to tag the files with this bit, the SMP/E install user requires BPX.FILEATTR.PROGCTL permission on the system, which can be achieved with the following command from a USS shell.

```IBMUSER:/Z31A/usr/lpp/IBM/izp/v1r2m0/bin: >extattr +p */zssServer/lib/*```



## 2.2 Edit the ZWEYAML parmlib member

The heart of the configuration of Zowe was a YAML file (zowe.yaml) which tied together all the elements of the zowe configuration.

Similarly, the heart of the configuration of Unified Management Server is it's YAML files. It uses a set of 3 YAML files (ZWEYAML, IZPDB2PM, IZPDAFPM) in the instance PARMLIB ( DAFUMS.IZP.I1.PARMLIB ) to tie together all the elements of the UMS configuration, including reference to the Zowe environment that UMS will be installed in. These 3 YAML files are to be created as 3 members of a PDS. Editing this YAML file correctly is critical.

Choose a naming standard for the datasets of the UMS instance. My SMPE installation created the UMS installation datasets under ```DAFUMS.IZP.**``` so I decided to create my instance datasets under the HLQ ```DAFUMS.IZP.I1.**```

Create a customisated copy of the installation SAMPLLIB (DAFUMS.IZP.SIZPSAMP) at (DAFUMS.IZP.I1.SIZPSAMP) and copy all 6 members over. (IZPALOPL, IZPCPYML, IZPCPYM2, IZPGENER, IZPMIGRA, IZPSYNCY ).

Edit and submit DAFUMS.IZP.I1.SIZPSAMP(IZPALOPL) ... to Allocate DAFUMS.IZP.I1.PARMLIB  

Edit and submit DAFUMS.IZP.I1.SIZPSAMP(IZPCPYML) ... to create the ZWEYAML default PARMLIB member (to be edited).... DAFUMS.IZP.I1.PARMLIB(ZWEYAML)

Edit DAFUMS.IZP.I1.PARMLIB(ZWEYAML). This is a very long and verbose dataset with incredibly strict syntax standards. An excerpt for the lines that I edited in included below, with line-numbered notes to explain the logic for my edits.

* line 28. specifies the USS path for any installed experiences. We are installing the Db2 Admin Foundation, so add the path.
* Lines 34 - 36. allows you to provide a Jon card for all the deployment jobs that will be generated.
* Line 42. specifies the USS path for UMS itself.
* Line 50. specifies the path of the workspace directory, which needs 755 permissions
* Line 66. specifies that UMS will always use the SAF (eg: RACF) for authentication. (Earlier versions of UMS allowed UMS-based authentication).
* Line 80. specifies that PASSWORD authentication will be used
* Lines 86, 94, 103 are left blank because we will use RACF keyrings rather that USS keystores to hold our certificates
* Lines 109 - 125. specifies the "DBA user" for UMS, and the name of the authentication token it will use.
* Lines 129 - 182. specifies the Keyrings and certificates that will be used by UMS.
* Lines 187 - 208. specifies the roles that will be used with the IZP RACF class.
* Lines 213 - 284. specifies the network and TLS identities
* Lines 296 - 340. specifies the locations of UMS and DB2 datasets
* Lines 345 - 377. specifies the tools discovery parameters, that we will come back to later.

```
000027     experiences:                   
000028       - /usr/lpp/IBM/afx/v1r2m0/bin

000033     jobCard:                                               
000034       - //IZPCUST1 JOB (FB3),'UMSCUST',CLASS=A,MSGCLASS=H, 
000035       - //       NOTIFY=&SYSUID,REGION=0M,TIME=1440        
000036       - //*                                                

000042     runtimeDirectory: "/usr/lpp/IBM/izp/v1r2m0/bin" 

000050     workspaceDirectory: "/global/ums"

000061     security:   
...
000066       useSAFOnly: true   
...
000080         defaultAuthenticationMechanism: PASSWORD 
...
000086         # defaultDbaUserCertificateLabel:     
000094         # defaultDbaUserCertificateLocation:  
000103         # defaultDbaUserCertificateKeystoreType:    
...
000109       profileQualifier:                                               
000110       #                                                               
000111       # Encryption using ICSF PKCS#11 services.                       
000112       #                                                               
000113       pkcs11:                                                         
000114         #                                                             
000115         # The user name of dba that goes with the encrypted password. 
000116         #                                                             
000117         dbaUser: IZPDBA                                               
000118         #                                                             
000119         # The pkcs#11 token where the secret key material is stored.  
000120         #                                                             
000121         token: IZPTOK                                                 
000122         #                                                             
000123         # Path to pkcs#11 provider module.                            
000124         #                                                             
000125         library: /usr/lpp/pkcs11/lib/csnpca64.so                      
...
000129       certificate:                                                                                                 
000134         allowSelfSigned: true                                        
...                                                        
000142         truststore:                                                                                                   
000151           location: "////ZWESVUSR/ZoweKeyring"                       
000159           type: "JCERACFKS" 
...
000163         keystore:                                              
000170           location: "////ZWESVUSR/ZoweKeyring"                       
000178           type: "JCERACFKS"                                                                              
000182           alias: "zowes0w1"   
...
000187       profilePrefix:             
000188         # Super Role             
000189         super: IZP.SUPER         
000190         # Administrator Role     
000191         admin: IZP.ADMIN         
...                        
000200       surrogateUser:             
000201         # Super Role             
000202         super: IZPSRGSP          
000203         # Administrator Role     
000204         admin: IZPSRGAD          
...                   
000208       surrogateGroup: IZPSRGRP   
...
000213     server:                            
000221       tlsVersionList: TLSv1.2,TLSv1.3  
000228       authType: STANDARD_JWT           
...
000232       port:           
000233         http: 12023   
000234         agent: 3444   
000235         gremlin: 8182                                  
...
000284       allSysnames: "S0W1"     
...
000296     dataset:                                                  
000300       runtimeHlq: "DAFUMS.IZP"                                
000305       hlq: "DAFUMS.IZP.I1"                                    
000310       parmlib: "DAFUMS.IZP.I1.PARMLIB"                         
000315       jcllib: "DAFUMS.IZP.I1.JCLLIB"                           
...
000319       loadLibrary:               
000320         db2: "DSND10.SDSNLOAD"   
000326         # izp:                   
...
000331       dbaEncryption: "DAFUMS.IZP.I1.DBA.ENCRYPT"      
000335       userList: "DAFUMS.IZP.I1.USERLIST"              
000340       teamList: "DAFUMS.IZP.I1.TEAMLIST"              
...
000345     toolsDiscovery:                                              
000349       enabled: true                                              
000360       discoverySearchPaths: []                                   
000361     #                                                            
000362     # Zowe app-specific parameters                               
000363     #                                                            
000364     zowe:                                                        
000365       job:                                                       
000366         #                                                        
000367         # Suffix for Java program which will be run by UMS.  Defa
000368         #                                                        
000369         suffix: IZP                                              
000370 zowe:                                                            
000371   # These zowe items are required by the IZP setup.  Do not edit.
000372   setup:                                                         
000373     zis:                                                         
000374      parmlib:                                                    
000375        keys:                                                     
000376          IZP.ZSSP.REG: list                                      
000377   useConfigmgr: true                                       
```

Once you are satisfied that the ZWEYAML is correctly configured, it can be used to generate all the installation workflow jobs with the desired customisations.

## 2.3 Execute the UMS installation workflows (including integration of zowe.yaml with UMS ZWEYAML.

### Stop Zowe Now
These installation workflow steps will affect Zowe, which should be stopped before proceeding.

### 2.3.1 Generate the customised workflow jobs.
Customize and Run DAFUMS.IZP.I1.SIZPSAMP(IZPGENER) to generate the customised workflow jobs.

IZPGENER results in adding ENVIRON and JCLLIB libraries. The JCLLIB PDS contains the customised installation jobs. The ENVIRON PDS stored environment configuration details.
```
'DAFUMS.IZP.I1.ENVIRON' 
'DAFUMS.IZP.I1.JCLLIB'  
'DAFUMS.IZP.I1.PARMLIB' 
'DAFUMS.IZP.I1.SIZPSAMP'
```

Dataset 'DAFUMS.IZP.I1.JCLLIB' contains all the JCL's that may ( or may not ) need to be run. You need to refer to [this page](https://www.ibm.com/docs/en/umsfz/1.2.0?topic=references-list-jcllib-members) to decide which jobs need to be run for this instance.

In this worked example, the sequence of jobs that I chose to run were as follows.

```
IZPA1.... N/A - allocates TEAMLIST
IZPA1V... verification job

IZPA2.... N/A - allocates USERLIST    
IZPA2V... verification job
  
IZPA3.... YES - allocates IZP.CUST.DBA.ENCRYPT   
IZPA3V... verify  
IZPB0R... N/A - Create a new group for surrogate users. This is not required for useSAFOnly.  
IZPB0VR.. verify  
IZPB1R... YES - Create IZP class and add to the CDT.  
IZPB1VR.. verify  
IZPB2R... YES - Add security role profiles to the IZP class.   
IZPB2VR.. verify  
IZPB3R... N/A - Create generic profiles to secure userList and teamList data sets. This is not required if useSAFOnly=true.  
IZPB3VR.. verify  
IZPB4R... YES - Create RACF IZP resource profiles to define the UMS users and their roles.  
IZPB4VR.. verify  
IZPC1R... N/A - Add surrogate users to impersonate when accessing the userList and teamList data sets during runtime. This is not required if useSAFOnly=true.  
IZPC1VR.. verify  
IZPC2R... N/A - Grant surrogate user access to the userList and teamList profiles. This is not required if useSAFOnly=true.  
IZPC2VR.. verify  
IZPD1R... YES - Define CRYPTOZ resource profiles for the PKCS #11 token for UMS.   
IZPD1VR.. verify  
IZPD2R... N/A - Grant system programmer and started task access to PKCS #11 resources. 
IZPD2VR.. verify  
IZPD3R... N/A - Create the PKCS #11 token for UMS. This is not required if you are  
IZPD3VR.. verify  
IZPD4R... YES - Add a new user to serve as the DBA user ID.  
IZPD4VR.. verify  
IZPD5R... YES - Connect the DBA user ID to the IZUUSER group for z/OSMF.  
IZPD5VR.. verify  
IZPD6R... YES - Grant the DBA user ID access to applications. If useSAFOnly=true, permits are not required for the surrogate users.   
IZPD6VR.. verify  
IZPD7R... YES - Creates function profiles in IZP class that are used when useSafOnly is enabled, which allow users to refresh the security cache.   
IZPD7VR.. verify  
IZPSTEPL. YES - concatenate datasets in PROCLIB member
IZPUSRMD. N/A - If useSafOnly is set to true or you are migrating from UMS 1.1, do not submit the IZPUSRMD JCL.
izp-encrypt-dba.sh
IZPIPLUG. YES - Install Zowe plugins using the zwe command.
IZPEXPIN. YES - LAUNCH THE IZP EXPERIENCE INTEGRATION SCRIPT
```


### 2.3.2 Allocate DAFUMS.IZP.I1.DBA.ENCRYPT
IZPA3
IZPA3V

### 2.3.3 Create IZP class and add to the CDT. 
IZPB1R... YES - Create IZP class and add to the CDT. (not convinced it worked with RALT commands all following the RDEF, without a refresh)
IZPB1RV... YES - OK
IZPB1RF... FIX - re-run RALTs - to be sure to be sure

### 2.3.4 Add security role profiles to the IZP class.  (IZP.SUPER* and IZP.ADMIN*)
IZPB2R... YES - Add security role profiles to the IZP class.  
IZPB2Rv

### 2.3.5 Create RACF IZP resource profiles to define the UMS users and their roles.  
IZPB4R
IZPB4RV

### 2.3.6 Define CRYPTOZ resource profiles for the PKCS #11 token for UMS. 
IZPD1R... YES - Define CRYPTOZ resource profiles for the PKCS #11 token for UMS. 
IZPD1RV

### 2.3.7 Grant system programmer and started task access to PKCS #11 resources.
IZPD2R... Grant system programmer and started task access to PKCS #11 resources. 
IZPD2VR.. verify  

### 2.3.8 Create the PKCS #11 token for UMS. This is not required if you are  
IZPD3R... Create the PKCS #11 token for UMS. This is not required if you are  
IZPD3VR.. verify  

### 2.3.9 Add a new user to serve as the DBA user ID.  (IZPDBA)
IZPD4R - yes with errors... you can ignore a non-zero return code.
IZPD4RV - no records found in zSecure

### 2.3.10 Connect IZPDBA to the IZUUSER group for z/OSMF
IZPD5R  
IZPD5RV

READY                         
CONNECT IZPDBA GROUP(IZUUSER) 
READY                         
END                           

### 2.3.11 Grant the DBA user ID access to applications. If useSAFOnly=true, permits are not required for the surrogate users.  
IZPD6R
IZPD6RV

n/a
Note: If the APPL class is active and OMVSAPPL is defined, submit the job IZPD6R to permit IZPSRGSP, IZPSRGAD, DBA user ID, UMS user, and the Zowe STC user (ZWESLSTC) read access on the OMVSAPPL resource. 
RLIST APPL OMVSAPPL
ICH13003I OMVSAPPL NOT FOUND
***                         


### 2.3.12 Creates function profiles in IZP class that are used when useSafOnly is enabled, which allow users to refresh the security cache. 
IZPD7R
IZPD7RV


### 2.3.13 IZPSTEPL. YES - concatenate datasets in PROCLIB member

 SDSF OUTPUT DISPLAY IZPCUST1 JOB04388  DSID   102 LINE 0       COLS 02- 81     
 COMMAND INPUT ===>                                            SCROLL ===> CSR  
********************************* TOP OF DATA **********************************
System Information:                                                             
z/OS S0W1 01.00 03 1090                                                         
ZOAU Version:                                                                   
2025/03/13 21:08:08 CUT v1.3.5.0 bcb459b6 7509 PH64192 1385 36baca08            
Python Version:                                                                 
Python 3.12.3                                                                   
Java Version:                                                                   
java version "11.0.24" 2024-07-16                                               
IBM Semeru Runtime Certified Edition for z/OS 11.0.24.1 (build 11.0.24+8)       
IBM J9 VM 11.0.24.1 (build z/OS-Release-11.0.24.1-b01, JRE 11 z/OS s390x-64-Bit 
OpenJ9   - 0f87f4c7844                                                          
OMR      - 55ddfd47ab0                                                          
IBM      - 3c87141                                                              
JCL      - 5f768fcf827 based on jdk-11.0.24+8)                                  
IZPPI0079I - Start of izp-concatenate-proclib.sh                                
IZPPI0123I - IBM Unified Management Server for z/OS version 1.2.0.9             
IZPPI0092I - Updated USER.Z31C.PROCLIB(ZWESASTC) with STEPLIB DSND10.SDSNLOAD   
IZPPI0080I - End of izp-concatenate-proclib.sh. Return code 0                   
******************************** BOTTOM OF DATA ********************************


### 2.3.14 Encrypt DBA credentials.

This step failed because I hadn't run jobs IZPD2R and IZPD3R.

izp-encrypt-dba.sh encrypts IZPDBA user's password

As an install user, run izp-encrypt-dba.sh from your UMS installation location. You need to provide the high-level qualifier of the environment data set. This is the YAML variable listed as {components.izp.dataset.hlq}.

{components.izp.runtimeDirectory}/ums/opt/bin/izp-encrypt-dba.sh {components.izp.dataset.hlq}

/usr/lpp/IBM/izp/v1r2m0/bin
DAFUMS.IZP.I1

Replace {components.izp.runtimeDirectory} and {components.izp.dataset.hlq}, on your command line, with the values of these parameters in the ZWEYAML member.
The izp-encrypt-dba.sh shell script populates the {components.izp.dataset.dbaEncryption} data set used to store the encrypted DBA credential. 
It is not recommended to use OMVS to complete this step because the password is visible on the screen in plain text. Use SSH to perform this step.

/usr/lpp/IBM/izp/v1r2m0/bin/ums/opt/bin/izp-encrypt-dba.sh DAFUMS.IZP.I1

For more information, refer to Updating UMS DBA user credentials.


/usr/lpp/IBM/izp/v1r2m0/bin/ums/opt/bin/izp-encrypt-dba.sh DAFUMS.IZP.I1
IZPPI0079I - Start of izp-encrypt-dba.sh
IZPPI0123I - IBM Unified Management Server for z/OS version 1.2.0.9
IZP Credential Encryption Utility
Using PKCS #11 token label: IZPTOK
Using path to PKCS #11 library file: /usr/lpp/pkcs11/lib/csnpca64.so
IZPSC0003E - Fail to create dba configuration, reason 'Invalid Token Label : IZPTOK'.Encryption Failed
IZPPI0202E - Could not encrypt credentials. Refer to the log file: /tmp/izp-n-20250704211308.log.
IZPPI0080I - End of izp-encrypt-dba.sh. Return code 1

===

run jobs IZPD2R and IZPD3R.
Then retry Encrypt DBA credentials.
When prompted for IZPDBA password - enter l0nep1ne

IBMUSER:/u/ibmuser: >/usr/lpp/IBM/izp/v1r2m0/bin/ums/opt/bin/izp-encrypt-dba.sh DAFUMS.IZP.I1
IZPPI0079I - Start of izp-encrypt-dba.sh
IZPPI0123I - IBM Unified Management Server for z/OS version 1.2.0.9
IZP Credential Encryption Utility
Using PKCS #11 token label: IZPTOK
Using path to PKCS #11 library file: /usr/lpp/pkcs11/lib/csnpca64.so
Using database administrator username: IZPDBA
Enter the password for the database administrator:

Reenter the password:

Supplied Credentials encrypted
IZPPI0080I - End of izp-encrypt-dba.sh. Return code 0
IBMUSER:/u/ibmuser: >



>>>>>>>>>>>>>> EXTRA
Create IZPDBA based on IBMUSER ( to get SYSADM etc... )
Set a password for IZPDBA (adcdmst)
logged on tso - changed pwd to COCACOLA
deleted DAFUMS.IZP.I1.DBA.ENCRYPT
re-allocated DAFUMS.IZP.I1.DBA.ENCRYPT
Re-Ran Encrypt DBA credentials.





### 2.3.15 IZPIPLUG. YES - Install Zowe plugins using the zwe command.
should find the base UMS plugins

Job4398 - 
 SDSF OUTPUT DISPLAY IZPCUST1 JOB04398  DSID   102 LINE 0       COLS 02- 81     
 COMMAND INPUT ===>                                            SCROLL ===> CSR  
********************************* TOP OF DATA **********************************
System Information:                                                             
z/OS S0W1 01.00 03 1090                                                         
ZOAU Version:                                                                   
2025/03/13 21:08:08 CUT v1.3.5.0 bcb459b6 7509 PH64192 1385 36baca08            
Python Version:                                                                 
Python 3.12.3                                                                   
Java Version:                                                                   
java version "11.0.24" 2024-07-16                                               
IBM Semeru Runtime Certified Edition for z/OS 11.0.24.1 (build 11.0.24+8)       
IBM J9 VM 11.0.24.1 (build z/OS-Release-11.0.24.1-b01, JRE 11 z/OS s390x-64-Bit 
OpenJ9   - 0f87f4c7844                                                          
OMR      - 55ddfd47ab0                                                          
IBM      - 3c87141                                                              
JCL      - 5f768fcf827 based on jdk-11.0.24+8)                                  
IZPPI0079I - Start of izp-install-plugins.sh.                                   
IZPPI0123I - IBM Unified Management Server for z/OS version 1.2.0.9             
IZPPI0122I - Installing IZP as a Zowe component, using manifest /usr/lpp/IBM/izp
             Target Zowe APF-authorized dataset ZWE200.CUST.ZWESAPL.            
             IZP workspace /global/ums.                                         
/tmp/.zweenv-3146/zwe-parmlib-8155 -> DAFUMS.IZP.I1.PARMLIB(ZWEYAML): text      
Temporary directory '/tmp/.zweenv-3146' created.                                
Zowe will remove it on success, but if zwe exits with a non-zero code manual cle
bos extend currSize=0x0 dataSize=0x178a chunk=0x1000 extend=0x178a              
Installing file or folder=/usr/lpp/IBM/izp/v1r2m0/bin                           
Install bin                                                                     
Process ums/opt/bin/izp-install.sh defined in manifest commands.install:        
2025-07-05 02:46:25 <ZWELS:50398022> IBMUSER INFO (zwe-components-install-proces
                                                                                
Successfully installed ZIS plugin: zos-newton-db2ifi                            
Successfully installed ZIS plugin: zos-newton-discovery                         
Successfully installed ZIS plugin: zos-newton-jobs                              
Successfully installed ZIS plugin: zos-newton-registry                          
Successfully installed ZIS plugin: zos-newton-security                          
Successfully installed ZIS plugin: zss-data-provider                            
Successfully installed ZIS plugin: cidb                                         
Successfully installed ZIS plugin: ums-security                                 
Successfully installed ZIS plugin: zos-newton-daj                               
Successfully installed ZIS plugin: hlv-discovery                                
- update zowe config /tmp/.zweenv-3146/.zowe-merged.yaml, key: "components.izp.e
  * Success                                                                     
Writing temp file for PARMLIB update. Command= cp -v "/tmp/.zweenv-3146/zwe-parm
bos extend currSize=0x0 dataSize=0x178a chunk=0x1000 extend=0x178a              
IZPPI0080I - End of izp-install-plugins.sh. Return code 0                       
******************************** BOTTOM OF DATA ********************************




### 2.3.16 IZPEXPIN. YES - LAUNCH THE IZP EXPERIENCE INTEGRATION SCRIPT
should find DAF

 SDSF OUTPUT DISPLAY IZPCUST1 JOB04407  DSID   102 LINE 0       COLS 02- 81     
 COMMAND INPUT ===>                                            SCROLL ===> CSR  
********************************* TOP OF DATA **********************************
System Information:                                                             
z/OS S0W1 01.00 03 1090                                                         
ZOAU Version:                                                                   
2025/03/13 21:08:08 CUT v1.3.5.0 bcb459b6 7509 PH64192 1385 36baca08            
Python Version:                                                                 
Python 3.12.3                                                                   
Java Version:                                                                   
java version "11.0.24" 2024-07-16                                               
IBM Semeru Runtime Certified Edition for z/OS 11.0.24.1 (build 11.0.24+8)       
IBM J9 VM 11.0.24.1 (build z/OS-Release-11.0.24.1-b01, JRE 11 z/OS s390x-64-Bit 
OpenJ9   - 0f87f4c7844                                                          
OMR      - 55ddfd47ab0                                                          
IBM      - 3c87141                                                              
JCL      - 5f768fcf827 based on jdk-11.0.24+8)                                  
IZPPI0079I - Start of izp-cp-exp.sh                                             
IZPPI0123I - IBM Unified Management Server for z/OS version 1.2.0.9             
IZPPI0121I - Updating /tmp/izp-merge-83952462 with new elements from /usr/lpp/IB
IZPPI0031I - File copy status: OK - IZPDB2PM                                    
IZPPI0121I - Updating /tmp/izp-merge-83952462 with new elements from /usr/lpp/IB
IZPPI0031I - File copy status: OK - IZPDAFPM                                    
IZPPI0049I - Experience post-installation status: /usr/lpp/IBM/afx/v1r2m0/bin/ad
alloc da('DAFUMS.IZP.I1.SAFXDBRM') dsorg(po) dsntype(library) tracks space(10,5)
alloc da('DAFUMS.IZP.I1.SAFXLLIB') dsorg(po) dsntype(library) tracks space(100,1
alloc da('DAFUMS.IZP.I1.SAFXSAMP') dsorg(po) dsntype(library) tracks space(10,5)
IZPPI0049I - Experience post-installation status: /usr/lpp/IBM/afx/v1r2m0/bin/ad
IZPPI0080I - End of izp-cp-exp.sh. Return code 0                                
******************************** BOTTOM OF DATA ********************************






## 2.4 start the zowe server (and likely debug initial UMS startup problems).

### 2.4.1 Prepare the PROCLIB member
Edit the ZOWE Started Task - USER.Z31C.PROCLIB(ZWESLSTC)

//ZWESLSTC  PROC RGN=0M,HAINST='__ha_instance_id__'   
//ZWELNCH  EXEC PGM=ZWELNCH,REGION=&RGN,TIME=NOLIMIT,                  
// PARM='ENVAR(_CEE_ENVFILE=DD:STDENV),POSIX(ON)/&HAINST.'             
//STEPLIB  DD   DSNAME=ZWE200.CUST.SZWEAUTH,                           
//             DISP=SHR                                                
//SYSIN    DD  DUMMY                                                   
//SYSPRINT DD  SYSOUT=*,LRECL=1600                                     
//SYSERR   DD  SYSOUT=*                                                
//********************************************************************/
//STDENV   DD  *                                
_CEE_ENVFILE_CONTINUATION=\                     
_CEE_RUNOPTS=HEAPPOOLS(OFF),HEAPPOOLS64(OFF)    
_EDC_UMASK_DFLT=0002                            
CONFIG=PARMLIB(DAFUMS.IZP.I1.PARMLIB(ZWEYAML))\ 
:FILE(/apps/zowe/v20/zowe.yaml)                 
/*                                              


### 2.4.2 Start Zowe (including UMS)
Start ZOWE
S ZWESISTC,REUSASID=YES
S ZWESLSTC

https://s0w1.dal-ebis.ihost.com:7554/zlux/ui/v1 




## 2.5 test Zowe from a web browser.


### 2.5.1 Clean Start of UMS

Session Renewal Error
05/07/2025, 13:42:23
Session could not be renewed. Logout will occur unless renewed. Click here to retry.

Open UMS ...
Error Request failed with status code 401


Eventually I found where the UMS logs were. ( /apps/zowe/v20/log or SDSF ZWESLSTC job output )
ZOWE logs and UMS logs are shared.

These logs - at the very end when I logon to ZOWE show an error
<ZWED:83952169> ZWESVUSR WARN (com.rs.auth.db2Auth,db2Auth.js:153) UMS login: connect ECONNREFUSED 192.168.1.171:12023

ECONNREFUSED means a problem with TCPIP address/port or firewall usually.
Or perhaps something as basic as UMS not started.

Hidden within the zowe/ums output is confirmation that UMS did not start
IZPPI0205E - Validation unsuccessful, it returned code 4 for DAFUMS.IZP.I1.PARMLIB(IZPDB2PM)
<ZWELNCH:83951980> ZWESVUSR ERROR ZWEL0038E failed to restart component izp, max retries reached


So, given that the reason for UMS not starting looks likely that DAFUMS.IZP.I1.PARMLIB(IZPDB2PM) prevented successful validation,
I edited it, and entered the HLQs for IZP and AFX/ADM


--> Conclusion: can't add a new experience until it is properly configured
Configure more advanced DB2 Admin Tool experiences. ( LIST THEM )
Edit the additional YAML files to locate the DB2AOC libraries
IZPDB2PM
IZPDAFPM




### 2.5.2 Open DAF

Open UMS.
Navigate to DAF.

Discover DALLASD

Register DALLASD
>>> Lots of errors.
>>> SSL connection 5046 - failed - need to setup trust from ZOWE/UMS to Db2 ???
>>> non-secure 5045 - failed - no SYSADM grant ; DSNTEP2 DDL connection failure ( APPLCOMPATs etc... )
>>> Rebind Db2 Connect Packages ; Rebind DSNTEP2
>>> Still failing... try re-IPL and fresh system
>>> Aha - when registering the DBDG subsystem, plan name for DSNTEP2 is DSNTEP13

Basic Catalog Navigation.
All food

Gen DDL - fails.
SQLCODE -444 : Procedure DBDGDDL not found.
Funny - Db2 Admin tool DDL GEN invoked DBDGDDL in DBGENV1 without fail.
But DAF does the same thing - and Load Module Not found

I was trying with usig Db2 Admin Tool "HLQ IZP_DB2_ADB_PREFIX: SADB" in DAFUMS.IZP.I1.PARMLIB(IZPDB2PM)
But I notice there are two Procedures in SYSROUTINES - One bound in 2022, the other bound today.
The DAF version of DDL gen will only generate DDL for a single object at a time.
Lets see if that will work ?

DAFUMS.IZP.I1.PARMLIB(IZPDB2PM)
000153 # Required Parameters                                                   
000154 # High-level qualifier (HLQ) and prefix for user data sets created and  
000155 # written to during various JCL Jobs execution. Eg, using the sample val
000156 # registring a Db2 subsystem will write to HLQ.IZP.DSN.SIZPTLIB         
000157 # It is recommended to use the UMS read/write HLQ specified by          
000158 # components.izp.dataset.hlq in ZWEYAML.                                
000159 # Sample value: HLQ.IZP.DSN                                             
000160 IZP_DB2_USR_HLQ: DAFUMS.IZP.I1                                          
000161 # Sample value (recommended): SIZP                                      
000162 IZP_DB2_USR_PREFIX: SIZP                                                
000163                                                                         
000164 # Required Parameters                                                   
000165 # High-level qualifier (HLQ) and prefix for IBM Db2 Administration Tool 
000166 # related data sets to be concatenated in various generated JCL.        
000167 # If using IBM Db2 Administration Tool for z/OS, specify HLQ where      
000168 # ADB functionality was SMPE installed.                                 
000169 # Otherwise enter the UMS read/write HLQ specified by                   
000170 # components.izp.dataset.hlq in ZWEYAML.                                
000171 # Sample value: HLQ.ADB.DSN                                             
000172 # Sample value: HLQ.IZP.DSN                                             
000173 IZP_DB2_ADB_HLQ: ADBD10                                                 
000174 # Sample value (if using IBM Admin Tool): SADB                          
000175 # Sample value (otherwise, using Admin Foundation files) : SAFX         
000176 IZP_DB2_ADB_PREFIX: SAFX                                                

recycle ZOWE
Retest


Config Steps
https://www.ibm.com/docs/en/umsfz/1.2.0?topic=installation-installing-db2-administration-foundation

Create a WLM environment by using the template in JCLLIB(WLMPROC) that points to the load libraries in <HLQ>.SIZPLLIB.






