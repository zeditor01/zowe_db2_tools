[Back to Home Page](https://github.com/zeditor01/zowe_db2_tools/tree/main)

# Step 3. Deploying Unified Management Server

After installing Zowe, you will have a landscape like this.
![stage2](/images/zowestage2.jpg)

Now we wish to Deploy the Unified Management Server and Db2 Administration Foundation into Zowe.
1. install UMS and DAF code. (it makes sense to install DAF and UMS together).
2. edit ZWEYAML parmlib member to configure UMS to integrate with z/OS and Zowe'
3. Execute the UMS installation workflows (including integration of zowe.yaml with UMS ZWEYAML.
4. start the zowe server (and likely debug initial UMS startup problems).
6. test Zowe from a web browser.
   
![stage3](/images/zowestage3.jpg)

## Notes on using this page.

This worked example is NOT intended as a replacement for the installation instructions in the [UMS Knowledge Center](https://www.ibm.com/docs/en/umsfz/1.2.0?topic=server-installing-unified-management). Think of it as a worked example that should be helpful in conjucntion with the UMS Knowledge Center.

The main pages of this repository are written to provide guidance to experienced system programmers who don't need to be told how to perform a PSI install. If you want an expanded version of this page covering the basic systems programmer tasks, please switch to [This Page](https://github.com/zeditor01/zowe_db2_tools/blob/main/docs/x103_deploy_ums_tasks.md) 

## 2.1 install UMS and DAF code. (it makes sense to install DAF and UMS together).

Order Unified Management Server and Db2 Admin Foundation together on ShopZ.
![order_ums_daf](/images/order_ums_daf.jpg)



Follow the standard PSI installation workflows. My HLQ was "DAFUMS". I ended up with the following datasets after the SMPE installation.
* The DAFUMS.AFX.** datasets are the target and distribution libraries for Db2 Administration Foundation.
* The DAFUMS.CPAC.** and DAFUMS.SMPE.** datasets are the SMPE assets.
* The DAFUMS.IZP.** datasets are the target and distribution libraries for Unified Management Server.
* The DAFUMS.IZP.I1.** datasets are datasets supporting an instance of UMS. (created by post-SMPE installation workflows covered later)
* The DAFUMS.OMVS.** datasets are the ZFS filesystems for Unified Management Server and Db2 Administration Foundation. 


![dafums_datasets](/images/dafums_datasets.jpg)


You will be required to permanently mount two ZFS filesystems as follows.

* UMS provides a ZFS called DAFUMS.OMVS.SIZPROOT - to mounted at /usr/lpp/IBM/izp/v1r2m0/bin
* DAF provides a ZFS called DAFUMS.OMVS.SAFXROOT - mounted at /usr/lpp/IBM/afx/v1r2m0/bin

UMS Zowe plug-ins require Program Control authorization. In order to tag the files with this bit, the SMP/E install user requires BPX.FILEATTR.PROGCTL permission on the system, which can be achieved with the following command from a USS shell.

```IBMUSER:/Z31A/usr/lpp/IBM/izp/v1r2m0/bin: >extattr +p */zssServer/lib/*```



## 2.2 edit ZWEYAML parmlib member to configure UMS to integrate with z/OS and Zowe'

The heart of the configuration of Zowe was a YAML file (zowe.yaml) which tied together all the elements of the zowe configuration.

Unified Management Server follows the same pattern. It also uses a set of 3 YAML files to tie together all the elements of the UMS configuration, including reference to the Zowe environment that UMS will be installed in. These 3 YAML files are to be created as 3 members of a PDS. Editing this YAML file correctly is critical.

Choose a naming standard for the datasets of the UMS instance. My SMPE installation created the UMS installation datasets under ```DAFUMS.IZP.**``` so I decided to create my instance datasets under the HLQ ```DAFUMS.IZP.I1.**```

Create a customisated copy of the installation SAMPLLIB ```DAFUMS.IZP.SIZPSAMP``` at ```DAFUMS.IZP.I1.SIZPSAMP``` and copy all 6 members over. (IZPALOPL, IZPCPYML, IZPCPYM2, IZPGENER, IZPMIGRA, IZPSYNCY ).

Edit and submit DAFUMS.IZP.I1.SIZPSAMP(IZPALOPL) ... which Allocates DAFUMS.IZP.I1.PARMLIB  

Edit and submit DAFUMS.IZP.I1.SIZPSAMP(IZPCPYML) ... Creates the ZWEYAML default PARMLIB member (to be edited).... DAFUMS.IZP.I1.PARMLIB(ZWEYAML)

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




## 2.4 start the zowe server (and likely debug initial UMS startup problems).



## 2.5 test Zowe from a web browser.



