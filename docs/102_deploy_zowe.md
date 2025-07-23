[Back to Home Page](https://github.com/zeditor01/zowe_db2_tools/tree/main)

# Step 2. Deploying Zowe



There are 4 options for installing the Zowe server-side component
1. Convenience build (unpack a pax file)
2. SMPE build (FMID + PTFs)
3. Portable Software Instance
4. Containerised build

You can download the server-side component from the Zowe website, or you can order a PSI from ShopZ. I chose the the convenience build. I download V2.17 for this worked example.


With the zowe-convenience build, I downloaded ```zowe-2.17.0.pax``` and transferred it to my smp downloads folder at ```/u/smpe/smpnts/zcb```.

I then allocated a ZFS and mounted it at ```/usr/lpp/zwe/zwe217```. Permanent PARMLIB mount specification below.

```
MOUNT FILESYSTEM('ZOWE217.ZFS')       
      TYPE(ZFS)                       
      MODE(RDWR)                      
      NOAUTOMOVE                      
      MOUNTPOINT('/usr/lpp/zwe/zwe217') 
```

I then copied the pax file from ```/u/smpe/smpnts/zcb``` zcb into target ZFS ```/usr/lpp/zwe/zwe217``` for deployment.

Unpack the pax file with the Command : ```pax -rvf zowe-2.17.0.pax```

Results in
```
IBMUSER:/Z31A/usr/lpp/zwe/zwe217: >ls -al
total 1039280
drwxr-xr-x   8 OMVSKERN SYS1        8192 Jul 19 23:18 .
drwxr-xr-x   5 OMVSKERN OMVSGRP     8192 Jul 19 22:43 ..
-rw-r--r--   1 OMVSKERN SYS1        2350 Jul 18 08:31 DEVELOPERS.md
-rw-r--r--   1 OMVSKERN SYS1        3772 Jul 18 08:31 README.md
drwxr-xr-x   5 OMVSKERN SYS1        8192 Jul 18 08:31 bin
drwxr-xr-x  19 OMVSKERN SYS1        8192 Jul 18 08:34 components
-rw-r--r--   1 OMVSKERN SYS1       25982 Jul 18 08:31 example-zowe.yaml
drwxr-xr-x   7 OMVSKERN SYS1        8192 Jul 18 08:34 files
drwxr-xr-x   2 OMVSKERN SYS1        8192 Jul 18 08:35 fingerprint
drwxr-xr-x   2 OMVSKERN SYS1        8192 Jul 18 08:31 licenses
-rw-r--r--   1 OMVSKERN SYS1       14213 Jul 18 08:31 manifest.json
drwxr-xr-x   2 OMVSKERN SYS1        8192 Jul 18 08:31 schemas
-rw-r-----   1 OMVSKERN SYS1     531675648 Jul 19 22:45 zowe-2.17.0.pax
```

This matches the file structure described by the Zowe documentation

![zowe_directory](/images/zowedir.JPG)


## 3. Editing the zowe.yaml file.

Within /usr/lpp/zwe/zwe217 , copy example-zowe.yaml to zowe.yaml for editing.

The entire Zowe deployment is defined by the zowe.yaml file. This configuration file is used by
1. the ZWE INSTALL script to create the ZOWE installation datasets
2. the ZWE INIT script to customise the ZOWE instance
3. at Zowe runtime

If you've not come across yaml files yet, yaml stands for 'yet another markup language'. The zowe.yaml file is documented [here](https://docs.zowe.org/stable/appendix/zowe-yaml-configuration/). It has 6 sections

1. zowe (Defines global configurations specific to Zowe, including default values)
2. java (Defines Java configurations used by Zowe components)
3. node (Defines node.js configurations used by Zowe components)
4. zOSMF (Tells Zowe your z/OSMF configurations)
5. components (Defines detailed configurations for each Zowe component or extension)
6. haInstances (Defines customized configurations for each High Availability (HA) instance)

I prepared the following [zowe.yaml](https://github.com/zeditor01/using_zowe/blob/main/samples/zowe.yaml) file for my system. ***Hint: right mouse click + open link in new tab.***

Note the following parameters

setup-dataset section
* line 43 provides the HLQ where the ZOWE datasets will be installed later on using the 'ZWE INSTALL' script
* line 46 specifies your chosen proclib where the startup procedures will be created
* line 49 specifies your chose parmlib for plugins
* line 56 specifies the PDS to create the customisation JCL members in
* line 58 specifies the loadlib from the chosen installation method (convenience build in my case)
* line 60 specifies the APF-authorised loadlib from the chosen installation method (convenience build in my case)
* line 63 specifies the APF authorized LOADLIB for Zowe ZIS Plugins from the chosen installation method (convenience build in my case)

setup-security section (needs to be filled in to use RACF and RACF keyrings)
* line 69 specifies that we will use RACF security. This is an opensource product, and prepares security configurations for multiple security products)
* line 71 - 77 specifies the RACF groups to create and use (I used ZWEADMIN for all three groups)
* lines 81 and 83 specifies the user for the main Zowe task (ZWESVUSR) and for the ZIS server (ZWESIUSR)
* lines 85 - 91 specifies the started task names ZWESLSTC (main server) ZWESISTC (zis server) and ZWESASTC (zis auxilliary server)

certificate section. The example zowe.yaml file provides templates for 5 different certificate configurations. I chose scenario 3 - Zowe generated z/OS Keyring with Zowe generated certificates.
* line 180 specifies that I wish to use a RACK keyring to hold certificates
* line 185 specifies the Keyring name (ZoweKeyring)
* line 188 specifies the label of my certificate
* line 190 specifies the label of my CA certificate
* line 193 - 200 specifies options distinguished name values
* line 202 ensures that the certificate won't expire until well after i retire
* line 208 and 210 specify the hostname (s0w1.dal-ebis.ihost.com) and IP address (192.168.1.171) for my z/OS system

cacheing service section
line 259 - 256 specifies to use Non-RLS VSAM for the cacheing service

runtime directories
* line 282 specifies the runtimeDirectory: "/usr/lpp/zwe/zwe217"
* line 286 specifies the logDirectory: /global/zowe/logs
* line 290 specifies the workspaceDirectory: /global/zowe/workspace
* line 294 specifies the extensions directory, where plugins will be linked: /global/zowe/extensions

ports
* line 364 specifies the port that Zowe listens for browser connections on

generated certificates: Leave this section blank. It will be updated during the 'ZWE INIT' Script
* line 396 - 420 specify the RACF keystore and truststore to be used for certificates

java
* line 447 specifies the path to Java 8. Note that Zowe V2.18 depends on Java 8 for the install, but can run with current Java 17.

nodeJS
* line 462 specifies the path to nodeJS

z/OSMF
* lines 471 - 477 specify the connection details for z/OSMF
  
I left everything else from the example zowe.yaml file to default.

Be careful of case sensitive parameters (paths and certificate details)

Once you've edited the zowe.yaml file, everything else should flow easily.


## 4. Installing the ZOWE instance libraries.


The zowe.yaml file controls the ZWE INSTALL Script.

First, Check your USS environment variables. You may edit your .profile to include the following specifications

```
# JAVA                                                                                               
export JAVA_HOME=/usr/lpp/java/J8.0_64                                 
export PATH=$JAVA_HOME/bin:$PATH                                                                          
# Node.js                                                              
export NODE_HOME=/usr/lpp/IBM/cnj/v20r11                               
export PATH=/usr/lpp/IBM/cnj/v20r11/bin:$PATH                          
export PATH=/usr/lpp/zwe/zwe217/bin:$PATH                              
```

Open a new USS shell (in order to execute your .profile) and run the following command.

```
zwe install -v -c /usr/lpp/zwe/zwe217/zowe.yaml
```

It should chunter away and create the following z/OS datasets
```
'ZWE217.SZWEAUTH'                              A3USR5
'ZWE217.SZWEEXEC'                              A3USR2
'ZWE217.SZWELOAD'                              A3USR6
'ZWE217.SZWESAMP'                              A3USR7 
```

That's it : The execution libraries are in place.

## 5. Configuring the ZOWE instance.

You can run the commands individually - but the ```--update-config --allow-overwrite``` options don't work so well like that.

```
zwe init mvs --update-config --allow-overwrite -v -c /usr/lpp/zwe/zowe/zowe.yaml
zwe init security --update-config --allow-overwrite -v -c /usr/lpp/zwe/zowe/zowe.yaml
zwe init apfauth --update-config --allow-overwrite -v -c /usr/lpp/zwe/zowe/zowe.yaml
zwe init certificate --security-dry-run -v -c /usr/lpp/zwe/zowe/zowe.yaml
zwe init certificate --update-config --allow-overwrite -v -c /usr/lpp/zwe/zowe/zowe.yaml
zwe init vsam --update-config --allow-overwrite -v -c /usr/lpp/zwe/zowe/zowe.yaml
zwe init stc --update-config --allow-overwrite -v -c /usr/lpp/zwe/zowe/zowe.yaml
```

So - we did the whole schmoogle in one step. My experience was that this worked first time.

```
zwe init --update-config --allow-overwrite -v -c /usr/lpp/zwe/zowe/zowe.yaml
```

You might want to check some of the artefacts that were created. 
For example, a RACF query ```racdcert id(ZWESVUSR) listring(ZoweKeyring)``` to see the contents of the ZoweKeyring that was created.

```
Digital ring information for user ZWESVUSR:                           
                                                                      
  Ring:                                                               
       >ZoweKeyring<                                                  
  Certificate Label Name             Cert Owner     USAGE      DEFAULT
  --------------------------------   ------------   --------   -------
  zowes0w1ca                         CERTAUTH       CERTAUTH     NO   
                                                                      
  zowes0w1                           ID(ZWESVUSR)   PERSONAL     YES  
                                                                      
  zOSMFCA                            CERTAUTH       CERTAUTH     NO   
```

Under certificate model 3, Zowe created a self-signed certificate and CA Cert, and placed it in ZoweKeyring, alongside the zOSMF CA Cert.

You should also check your PROCLIB to see the members placed in it.

```
ZWESASTC
ZWESISTC
ZWESLSTC
```

## 6. Operating ZOWE.

A few pieces of housekeeping before starting Zowe

### 6.1 Edit zowe.yaml again

Need to edit the zowe.yaml file: Need to comment out ```createZosmfTrust: true``` for zowe config validate and zowe start

### 6.2 Edit Program Properties Table for these tasks

Edit USER.Z31A.PARMLIB(SCHEDAL)
```
PPT PGMNAME(ZWESIS01) NOSWAP KEY(4) /* Zowe cross memory           */
PPT PGMNAME(ZWESAUX)  NOSWAP KEY(4) /* Zowe auxiliary (AUX)        */
```

### 6.3 Start ZWESISTC

z/OS operator console command: ```S ZWESISTC,REUSASID=YES```

### 6.4 Start ZWESLSTC

z/OS operator console command: ```S ZWESLSTC```


## 7. Using the ZOWE Base Apps.

The URL to open ZOWE from my browser is https://s0w1.dal-ebis.ihost.com:7554/zlux/ui/v1 where 
* s0w1.dal-ebis.ihost.com is my hostname
* 7554 is the port that ZOWE is configured to listen on

Firing up the Apps should be self-explanatory. Screenshots below

MVS Explorer
![MVS](/images/MVS.JPG)

JES Explorer
![MVS](/images/JES.JPG)

USS Explorer
![USS](/images/USS.JPG)

TN3270 Emulator
![TN3270](/images/TN3270.JPG)



## 8. Subsequent Upgrades.

The scope of this worked example was limited to a simple deployment. However, anybody running ZOWE will want to know how to upgrade it and how to avoid downtime.

The short answer is to do a fresh ZOWE install, and switch traffic when ready.

This subject is addressed by the ZOWE documentation.

[upgrade zowe with zero downtime](https://docs.zowe.org/stable/user-guide/api-mediation/upgrade-zowe-no-downtime/)

