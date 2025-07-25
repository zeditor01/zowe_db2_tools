[Back to Home Page](https://github.com/zeditor01/zowe_db2_tools/tree/main)

# Step 3. Deploying Unified Management Server

After installing Zowe, you will have a landscape like this.

![stage2](/images/zowestage2.jpg)

Now we wish to Deploy the Unified Management Server and Db2 Administration Foundation into Zowe, as illustrated below. 
UMS is implemented as a Zowe server extension and plug-in, integrating seamlessly into the Zowe Application Framework and desktop
UMS leverages the Zowe platform for UI, authentication, and REST services, appearing as a native component of the Zowe ecosystem.

![stage3](/images/zowestage3.jpg)

The components and dependencies of Unified Management Server are illustrated in the diagram below and described in the Knowledge Center [here](https://www.ibm.com/docs/en/umsfz/1.2.0?topic=zos-security-overview). From the top, users login to UMS, get authenticated by SAF (eg: RACF), and launch the Db2 Administration Foundation application, which makes REST API call to consume other services.

![izp_components](/images/izp_components.jpg)

This page is a worked example of the following steps to deploy UMS into Zowe.
1. install UMS and DAF code. (it makes sense to install DAF and UMS together).
2. edit ZWEYAML parmlib member to configure UMS to integrate with z/OS and Zowe'
3. Execute the UMS installation workflows (setting up RACF artefacts, and integrating ZWEYAML with zowe.yaml).
4. start the zowe server (and likely debug initial UMS startup problems).
6. test Zowe from a web browser.
   

## Notes on using this page.
This page is a simple worked example. It aims to communicate the key concepts and actions to experienced systems programmers. A more detailed audit trail of steps is available in [this supplementary page](https://github.com/zeditor01/zowe_db2_tools/blob/main/docs/x103_deploy_ums_tasks.md) 

## 1 install UMS and DAF code. (it makes sense to install DAF and UMS together).

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


## 2 Understanding the UMS security model
UMS and DB2 Administration Foundation now requires a security model with SAF authentication (useSAFOnly=true). This means that 
1. all users of UMS & DAF need to be authenticated against SAF ( RACF in this example ).
2. a powerful DBA ID is defined to perform all authorised Db2 tasks on behalf of authenticated users. 
3. The DBA userid is shielded from human use, and can be authenticated  by a digital certificate.
4. A Profile Qualifier is prepended to SAF profile checks to determine team membership and role assignment.
5. UMS uses Java Web Tokens (JWT) to enable secure single-signon sessions. JWTs encapsulate identity and authentication data in a JSON structure, and are cryptographically signed.
6. RACF Class IZP is used to control UMS applicaton roles and function security.
7. RACF Class CRYPTOZ is used for secure storage and handling of encryption keys/tokens.

The diagram below represents how JWT tokens are used to support sessions. A more detailed explanation of the security architecture for UMS is available in the knowledge center [here](https://www.ibm.com/docs/en/umsfz/1.2.0?topic=server-security-ums-zos)

![izp_tokens](/images/izp_tokens.jpg)

## 3 Edit the ZWEYAML parmlib member

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

## 4 Execute the UMS installation workflows (including integration of zowe.yaml with UMS ZWEYAML.

### Stop Zowe Now
These installation workflow steps will affect Zowe, which should be stopped before proceeding.

### 4.1 Generate the customised workflow jobs.
Customize and Run DAFUMS.IZP.I1.SIZPSAMP(IZPGENER) to generate the customised workflow jobs.

IZPGENER results in adding ENVIRON and JCLLIB libraries. The JCLLIB PDS contains the customised installation jobs. The ENVIRON PDS stored environment configuration details.
```
'DAFUMS.IZP.I1.ENVIRON' 
'DAFUMS.IZP.I1.JCLLIB'  
'DAFUMS.IZP.I1.PARMLIB' 
'DAFUMS.IZP.I1.SIZPSAMP'
```

Dataset 'DAFUMS.IZP.I1.JCLLIB' contains all the JCL's that may ( or may not ) need to be run. Each job is supplied with a verification job to check the outcome. You need to refer to [this page](https://www.ibm.com/docs/en/umsfz/1.2.0?topic=references-list-jcllib-members) to decide which jobs need to be run for this instance.

In this worked example, the sequence of jobs that I chose to run were as follows.

```
IZPA1.... N/A - allocates TEAMLIST, which is no longer used for access control because UMS now requires useSAFOnly: true
IZPA1V... verification job

IZPA2.... N/A - allocates USERLIST, which is no longer used for access control because UMS now requires useSAFOnly: true    
IZPA2V... verification job
  
IZPA3.... Required - allocates IZP.CUST.DBA.ENCRYPT, which is used to store the encryption token for the DBA id.  
IZPA3V... verification job
 
IZPB0R... N/A - Create a new group for surrogate users. This is not required for useSAFOnly: true   
IZPB0VR.. verification job
   
IZPB1R... Required - Create IZP class and add to the CDT.  
IZPB1VR.. verification job
    
IZPB2R... Required - Add security role profiles to the IZP class.   
IZPB2VR.. verification job
   
IZPB3R... N/A - Create generic profiles to secure userList and teamList data sets. This is not required if useSAFOnly=true.  
IZPB3VR.. verification job
   
IZPB4R... Required - Create RACF IZP resource profiles to define the UMS users and their roles.  
IZPB4VR.. verification job
   
IZPC1R... N/A - Add surrogate users to impersonate when accessing the userList and teamList data sets during runtime. This is not required if useSAFOnly=true.  
IZPC1VR.. verification job
   
IZPC2R... N/A - Grant surrogate user access to the userList and teamList profiles. This is not required if useSAFOnly=true.  
IZPC2VR.. verification job
   
IZPD1R... Required - Define CRYPTOZ resource profiles for the PKCS #11 token for UMS.   
IZPD1VR.. verification job
   
IZPD2R... Required - Grant system programmer and started task access to PKCS #11 resources. 
IZPD2VR.. verification job
   
IZPD3R... Required - Create the PKCS #11 token for UMS.   
IZPD3VR.. verification job
   
IZPD4R... Required - Add a new user to serve as the DBA user ID.  
IZPD4VR.. verification job
   
IZPD5R... Required - Connect the DBA user ID to the IZUUSER group for z/OSMF.  
IZPD5VR.. verification job
   
IZPD6R... Required - Grant the DBA user ID access to applications. If useSAFOnly=true, permits are not required for the surrogate users.   
IZPD6VR.. verification job
   
IZPD7R... Required - Creates function profiles in IZP class that are used when useSafOnly is enabled, which allow users to refresh the security cache.   
IZPD7VR.. verification job
   
IZPSTEPL. Required - concatenate datasets in PROCLIB member

IZPUSRMD. N/A - If useSafOnly is set to true or you are migrating from UMS 1.1, do not submit the IZPUSRMD JCL.

izp-encrypt-dba.sh ... Required - 

IZPIPLUG. Required - Install Zowe plugins using the zwe command.

IZPEXPIN. Required - LAUNCH THE IZP EXPERIENCE INTEGRATION SCRIPT
```


### 4.2 Execute the generated jobs for RACF-related resources ( IZPA3 through to IZPD7R )

These jobs are all customised from the IZPGENER job, and should be ready to execute. In each case, review JCL, submit and review output, and then run the associated verification job to confirm successful creation of the artefacts.



* IZPA3 - Allocate DAFUMS.IZP.I1.DBA.ENCRYPT
* IZPA3V - verifies it

* IZPB1R - Create IZP class and add to the CDT
* IZPB1RV - verifies it

* IZPB2R - Add security role profiles to the IZP class.  (IZP.SUPER* and IZP.ADMIN*)
* IZPB2RV - verifies it

* IZPB4R - Create RACF IZP resource profiles to define the UMS users and their roles.  
* IZPB4RV - verifies it

* IZPD1R - Define CRYPTOZ resource profiles for the PKCS #11 token for UMS.
* IZPD1RV - verifies it

* IZPD2R - Grant system programmer and started task access to PKCS #11 resources.
* IZPD2RV - verifies it

* IZPD3R - Create the PKCS #11 token for UMS.
* IZPD3RV - verifies it

* IZPD4R - Add a new user to serve as the DBA user ID.  (IZPDBA)
* IZPD4RV - verifies it

The generated job IZPD4R is incomplete and fails. Suggest you follow whatever local jobs exist for creating RACF IDs.

* IZPD5R - Connect IZPDBA to the IZUUSER group for z/OSMF
* IZPD5RV - verifies it

* IZPD6R -  Grant the DBA user ID access to applications.
* IZPD6RV - verifies it

* IZPD7R -  Creates function profiles in IZP class that are used when useSafOnly is enabled, which allow users to refresh the security cache.
* IZPD7RV - verifies it
                        

                    


### 4.3 IZPSTEPL. YES - concatenate datasets in PROCLIB member
This job updates the PROCLIB member for Zowe (ZWESASTC) to concatenate the libraries of zowe and UMS

```
/*MESSAGE UPDATE ZOWE AUX PROCLIB FOR IZP                                      
//SET1 SET UMSVLOC='/usr/lpp/IBM/izp/v1r2m0/bin'                               
//SERVER    EXEC PGM=BPXBATCH,REGION=800M,TIME=NOLIMIT,                        
//   PARM='SH &UMSVLOC/ums/opt/bin/izp-concatenate-proclib.sh'                 
//STDOUT   DD SYSOUT=*                                                         
//STDENV   DD *                                                                
STEPLIB_DATASET=DSND10.SDSNLOAD                                                
PROCLIB_MEMBER=ZWESASTC                                                        
IZP_HLQ=DAFUMS.IZP.I1                                                          
_BPXK_AUTOCVT=ON                                                               
_CEE_RUNOPTS=FILETAG(AUTOCVT,AUTOTAG) POSIX(ON) HEAPPOOLS(OFF) HEAPPOOLS64(OFF)
_TAG_REDIR_IN=TXT                                                              
_TAG_REDIR_OUT=TXT                                                             
_TAG_REDIR_ERR=TXT                                                             
/*                                                                             
```

Actually it executes a USS script called izp-concatenate-proclib.sh. SDSF job output below

```
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
```

### 4.4 Encrypt DBA credentials.
The script encrypts the DBA username and password, and stored them in the Encrypted Credentials Data Set: {components.izp.dataset.dbaEncryption} = DAFUMS.IZP.I1.DBA.ENCRYPT.

This step is another USS script, but it must be executed from within a USS shell in order that the submitted may respond to a prompt to enter the password for the IZPDBA userid. The script invocation is captured below:

```
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
```

You can eyeball the Encrypted Credentials Data Set: {components.izp.dataset.dbaEncryption} = DAFUMS.IZP.I1.DBA.ENCRYPT.

![db2_encrypt](/images/dba_encrypt.jpg)





### 4.5 IZPIPLUG. YES - Install Zowe plugins using the zwe command.
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




### 4.6 IZPEXPIN. YES - LAUNCH THE IZP EXPERIENCE INTEGRATION SCRIPT
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






## 5 start the zowe server (and likely debug initial UMS startup problems).

### 5.1 Prepare the PROCLIB member
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


### 5.2 Start Zowe (including UMS)
Start ZOWE
S ZWESISTC,REUSASID=YES
S ZWESLSTC

https://s0w1.dal-ebis.ihost.com:7554/zlux/ui/v1 




## 6 test Zowe from a web browser.


### 6.1 Clean Start of UMS

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




### 6.2 Open DAF

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






