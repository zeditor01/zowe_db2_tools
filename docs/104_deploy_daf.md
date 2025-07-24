[Back to Home Page](https://github.com/zeditor01/zowe_db2_tools/tree/main)

# Step 4. Configuring Db2 Administration Foundation


Db2 Administration Administration Foundation has a relatively simple set of catalog browsing capabilities by itself. If you have Db2 Administration Tool installed, then you can configure UMS to "discover" additional tools capabilities from Db2 Administration Tool, and expose them to Db2 Administration Foundation.
![stage3a](/images/zowe_adm.jpg)

This worked example is NOT intended as a replacement for the installation instructions in the [UMS Knowledge Center](https://www.ibm.com/docs/en/umsfz/1.2.0?topic=server-installing-unified-management). Think of it as a worked example that should be helpful in conjucntion with the UMS Knowledge Center.




```
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
```
