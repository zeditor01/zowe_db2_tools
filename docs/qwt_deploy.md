[home](https://github.com/zeditor01/zowe_db2_tools/blob/main/docs/ZPDT_Build_Path.md)

# Query Workload Tuner : Deploy Portable Software Instance

## Scope of this page

This page provides a worked example of deploying the Portable Software Instance (PSI) of Query Workload Tuner (QWT).

## Starting point

It is assumed that you have already ordered QWT from ShopZ, and downloaded the PSI.

Open the z/OSMF Software Management Application - Deployments page. From "Actions" Select New. 

![deploy_qwt_01](/images/deploy_qwt_01.jpg)

You will see a checklist in the next screen. Click the first step to Specify the properties of this deployment.

![deploy_qwt_02](/images/deploy_qwt_02.jpg)

Give this deployment a name (QWT) and optionally choose a category (Neale)

![deploy_qwt_03](/images/deploy_qwt_03.jpg)

From the checklist, choose "Select the software to deploy".

![deploy_qwt_04](/images/deploy_qwt_04.jpg)

Choose the QWT PSI that we previously downloaded.

![deploy_qwt_05](/images/deploy_qwt_05.jpg)

From the checklist, choose "Select the objective of this deployment".

![deploy_qwt_06](/images/deploy_qwt_06.jpg)

We want to create a new SMP/E CSI zone, residing on system S0W1. We are not deploying z/OS.

An SMP/E CSI dataset (Consolidated Software Inventory dataset) is a specialized VSAM key-sequenced data set used by IBM's System Modification Program Extended (SMP/E) to record and manage information about installed software products, system modifications (SYSMODs), and all related maintenance activity on a z/OS system.

![deploy_qwt_07](/images/deploy_qwt_07.jpg)

From the checklist, choose "Check for missing SYSMODS".

![deploy_qwt_08](/images/deploy_qwt_08.jpg)

Uncheck both reports to generate. These reports may be useful for adding software maintenance into an existing CSI. However, we are deploying a new portable software image into it's own CSI, so the ShopZ process should have provided the complete and correct list of SYSMODs.

![deploy_qwt_09](/images/deploy_qwt_09.jpg)

From the checklist, choose "Configure this deployment".

![deploy_qwt_10](/images/deploy_qwt_10.jpg)

There are multiple steps to configure the deployment. These steps allow us to control where deployment is placed within our z/OS system. review the welcome page to get familiar with the steps.

![deploy_qwt_11](/images/deploy_qwt_11.jpg)

Configure DLIBs : Yes we do want to copy the distribution zones for the software.

SMP/E maintains the concept of 3 zones.
* The Global Zone is a master index for the CSI and contains definitions for all other target and distribution zones.
* The Target Zone holds entries describing the contents and structure of target libraries, which contain the executable components and program products used by z/OS.
* The Distribution Zone contains entries for the distribution libraries, which store the master copy of all system elements and products. This zone is used for copying the installed product libraries to other z/OS systems.


![deploy_qwt_12](/images/deploy_qwt_12.jpg)

Configure Model : We are happy to accept the SMP/E Model supplied with the PSI. The following screens will provide us with plenty of scope to fine tune how it is deployed. 

![deploy_qwt_13](/images/deploy_qwt_13.jpg)


Configure SMP/E Zones : You many have naming standards for your SMP/E zones. I don't, so I accept the default target and distribution zone names.

![deploy_qwt_14](/images/deploy_qwt_14.jpg)

Configure Datasets : yes we are going to want to control the names and placements of datasets to be installed. Select All datasets, and choose "Modify" from the Actions pulldown.

![deploy_qwt_15](/images/deploy_qwt_15.jpg)

By default every dataset will have a meaningless HLQ. CB.ST313139 is derived from the ShopZ order number. We will change it to QWT universally, so that we can find all the datasets associated with QWT easily. 

We also choose to let Systems Managed Storage (SMS) choose where the datasets are deployed. The SMS configuration in this z/OS system contains pre-defined rules ( ACS rules ) that determine everything about where and how datasets are stored.

![deploy_qwt_16](/images/deploy_qwt_16.jpg)

Configure "Catalogs". Your system may have multiple ICF catalogs. This is a small ZPDT system. We will let everything be cataloged in the master catalog.

![deploy_qwt_17](/images/deploy_qwt_17.jpg)

From the checklist, choose "Volumes and Storage Classes". Nothing to do here because SMS is taking care of this.

![deploy_qwt_18](/images/deploy_qwt_18.jpg)

Configure Mountpoints.

Many z/OS products are shipped with one or more hierarchical files systems ( ZFS ) to store their contents. QWT ships three separate ZFS file systems.

We need to choose the ZFS paths that these filesystems are to be mounted on. Select All, and from the actions pulldown choose modify.


![deploy_qwt_19](/images/deploy_qwt_19.jpg)

We will simply remove the leading part of the path ( /applroot/CB/ST313139 ) so that the filesystems are mounted in their "standard" locations.


![deploy_qwt_20](/images/deploy_qwt_20.jpg)

From the main checklist, select "Define job settings".

![deploy_qwt_21](/images/deploy_qwt_21.jpg)

Review the job settings. Note the JCL dataset name. The JCL deployment jobs will be stored here.

![deploy_qwt_22](/images/deploy_qwt_22.jpg)

From the main checklist, select "Submit deployment jobs".

![deploy_qwt_23](/images/deploy_qwt_23.jpg)

We are presented with 5 jobs that required to deploy the PSI. We need to review and execute each in turn.


![deploy_qwt_24](/images/deploy_qwt_24.jpg)

Job #1 (IZUD01RA) is optional. It allows us to setup RACF dataset protections. The job would need to be edited to assign the correct groups to assign to the profiles and the correct IDs to have permissions. We can review the job either by clicking on the Job in the z/OSMF window, or by opening the job in the JCL dataset (IBMUSER.DM.D250820.T173424.CNTL). Open it in the ISPF editor for best text markup and highlighting.

![deploy_qwt_25](/images/deploy_qwt_25.jpg)

Being a ZPDT demo system, the RACF protection of datasets is not important. Hence, from the z/OSMF workflow I select "Override Complete" from  he Actions pulldown.

![deploy_qwt_26](/images/deploy_qwt_26.jpg)

With Job #1 marked as "override complete", I select Job #2 (Unzip datasets), and "Submit" from the Actions pulldown.

![deploy_qwt_27](/images/deploy_qwt_27.jpg)

I see it running...

![deploy_qwt_28](/images/deploy_qwt_28.jpg)

This job failed unexpectedly. I decided to leave this job failure in the worked example as an illustration of what to do when the push-button workflows fail.


![deploy_qwt_29](/images/deploy_qwt_29.jpg)

First thing I do is to review the job, to see what it is doing in detail. I open IBMUSER.DM.D250820.T173424.CNTL(IZUX02UZ) and see that it is defining a VSAM cluster for a ZFS, creating a directory (/tmp/izud-IBMUSER-T2325892) and mounting the ZFS there, so that it can run the Unzip job.

Next thing I do is go to SDSF to see the error messages, so that I can diagnose and resolve the problem.

# Grab a screenshot of the ZFS failure.

Strange thing is that I can't see any errors to diagnose. All that happens is that the Unzip step fails because the ZFS path is not present. There are no error messages suggesting that the previous steps to mount the ZFS have failed. Very strange.

When you experience problems that you don't understand, step back, take a cup of coffee or tea, and allow the explanation to reveal itself to you whilst you are relaxing.

With caffeine entering my body, I came up with the theory that even though the preceeding steps had return code 0, they cannot have been executed. looking at the SDSF output again, I realised that there is an absence of messages saying anything at all. And the echo commands should have written those messages to the job output. So perhaps the step exited without doing anything.

Eventually I realised that my mate Ben had "improved" the z/OS image I was using by upgrading the default USS shell to the bash shell. Hence, the very first command to enter the default shell failed, and the job step ended.

Being a decent chap, my mate Ben provide a fix. He gave me some logic to place in the .profile script, to NOT use the bash shell when the calling process is BPXBATCH. This gives me the productivity of the bash shell for interactive work, but doesn't break batch jobs (such as PSI deployments). The moral of the story is the IBM motto : Th1nk.

# Include a screenshop of .profile

Once the Unzip job runs, you can see all the z/OS datasets that have been unzipped.

![deploy_qwt_30](/images/deploy_qwt_30.jpg)

And we get a nice green "Complete" tick by Job #2.

Now we run Job #3, to install the ZFS datasets.

![deploy_qwt_31](/images/deploy_qwt_31.jpg)

And if we review the z/OS datasets again, we see that the QWT.OMVS.** datasets have been installed.

![deploy_qwt_32](/images/deploy_qwt_32.jpg)

Now we can submit Job #4, which renames the datasets.

![deploy_qwt_33](/images/deploy_qwt_33.jpg)

And we can see the datasets have lost their trailing .#

![deploy_qwt_34](/images/deploy_qwt_34.jpg)

Now we can run Job #5 to update the SMPE CSI datasets

![deploy_qwt_35](/images/deploy_qwt_35.jpg)

And we have 5 green ticks !
![deploy_qwt_36](/images/deploy_qwt_36.jpg)
![deploy_qwt_37](/images/deploy_qwt_37.jpg)
![deploy_qwt_38](/images/deploy_qwt_38.jpg)
![deploy_qwt_39](/images/deploy_qwt_39.jpg)

![deploy_qwt_40](/images/deploy_qwt_40.jpg)
![deploy_qwt_41](/images/deploy_qwt_41.jpg)
![deploy_qwt_42](/images/deploy_qwt_42.jpg)
![deploy_qwt_43](/images/deploy_qwt_43.jpg)
![deploy_qwt_44](/images/deploy_qwt_44.jpg)
![deploy_qwt_45](/images/deploy_qwt_45.jpg)
![deploy_qwt_46](/images/deploy_qwt_46.jpg)
![deploy_qwt_47](/images/deploy_qwt_47.jpg)
![deploy_qwt_48](/images/deploy_qwt_48.jpg)
![deploy_qwt_49](/images/deploy_qwt_49.jpg)

![deploy_qwt_50](/images/deploy_qwt_50.jpg)
![deploy_qwt_51](/images/deploy_qwt_51.jpg)


insert JCL in lieu of missing 52 jpg

![deploy_qwt_53](/images/deploy_qwt_53.jpg)
![deploy_qwt_54](/images/deploy_qwt_54.jpg)
![deploy_qwt_55](/images/deploy_qwt_55.jpg)
![deploy_qwt_56](/images/deploy_qwt_56.jpg)
![deploy_qwt_57](/images/deploy_qwt_57.jpg)
![deploy_qwt_58](/images/deploy_qwt_58.jpg)
![deploy_qwt_59](/images/deploy_qwt_59.jpg)

![deploy_qwt_60](/images/deploy_qwt_60.jpg)
![deploy_qwt_61](/images/deploy_qwt_61.jpg)
![deploy_qwt_62](/images/deploy_qwt_62.jpg)
![deploy_qwt_63](/images/deploy_qwt_63.jpg)

```
//IUZOSMF JOB (FB3),'ZOSMF PSI',CLASS=A,MSGCLASS=H,
//       NOTIFY=&SYSUID,REGION=0M,TIME=1440
//*      TYPRUN=HOLD
//*
/*JOBPARM SYSAFF=S0W1
//JOBLIB   DD DSN=QWT.CPAC.SCPPLOAD,
//            DISP=SHR
//*
//*******************************************************************
//* This step will run IEBUPDTE to create a BPXPRMDB member in
//* your PARMLIB dataset with the new filesystem structure.
//*******************************************************************
//BPXCS01 EXEC PGM=IEBUPDTE,COND=(4000,LT)
//SYSPRINT DD SYSOUT=*
//SYSUT1   DD DSN=QWT.CPAC.DB2.PARMLIB,
//            DISP=SHR
//SYSUT2   DD DSN=QWT.CPAC.DB2.PARMLIB,
//            DISP=SHR
//SYSIN    DD   DATA,DLM='%%'
./    ADD  NAME=BPXPRMDB,LIST=ALL
/*******************************************************************/
/*  This member, as specified in IEASYS00, will cause Unix System  */
/*  Services to come up with the filesystem  mounted.              */
/*******************************************************************/
MOUNT FILESYSTEM
('QWT.OMVS.SAFXROOT')
 MOUNTPOINT(
'/usr/lpp/IBM/afx/v1r2m0/bin')
 TYPE(ZFS) MODE(RDWR)
MOUNT FILESYSTEM
('QWT.OMVS.SDSNROT2')
 MOUNTPOINT(
'/usr/lpp/IBM/db2tms/v2r1')
 TYPE(ZFS) MODE(RDWR)
MOUNT FILESYSTEM
('QWT.OMVS.SGWROOT')
 MOUNTPOINT(
'/usr/lpp/IBM/qwtz')
 TYPE(ZFS) MODE(RDWR)
%%
//*
//NOTOK   EXEC PGM=CPPMAXRC,COND=((0,GE,BPXCS01),(4000,LT))
//*
//*******************************************************************
//* This step will create the mountpoint directory used in deployment
//*******************************************************************
//MKCOS1  EXEC PGM=BPXBATCH,COND=(4000,LT)
//STDPARM DD *
sh
mkdir -p 755
/usr/lpp/IBM/afx/v1r2m0/bin
/*
//STDOUT DD SYSOUT=*
//STDERR DD SYSOUT=*
//*
//*******************************************************************
//* This step will create the mountpoint directory used in deployment
//*******************************************************************
//MKCOS2  EXEC PGM=BPXBATCH,COND=(4000,LT)
//STDPARM DD *
sh
mkdir -p 755
/usr/lpp/IBM/db2tms/v2r1
/*
//STDOUT DD SYSOUT=*
//STDERR DD SYSOUT=*
//*
//*******************************************************************
//* This step will create the mountpoint directory used in deployment
//*******************************************************************
//MKCOS3  EXEC PGM=BPXBATCH,COND=(4000,LT)
//STDPARM DD *
sh
mkdir -p 755
/usr/lpp/IBM/qwtz
/*
//STDOUT DD SYSOUT=*
//STDERR DD SYSOUT=*
//*
//*******************************************************************
//* This step will mount the SMP/E filesystem dataset
//*******************************************************************
//STEP1   EXEC PGM=IKJEFT1B,COND=(4000,LT)
//SYSTSPRT DD   SYSOUT=*
//SYSTSIN  DD   *
  PROF MSGID WTPMSG
MOUNT FILESYSTEM +
('QWT.OMVS.SAFXROOT') +
MOUNTPOINT(+
'/usr/lpp/IBM/afx/v1r2m0/bin') +
TYPE(ZFS) MODE(RDWR)
MOUNT FILESYSTEM +
('QWT.OMVS.SDSNROT2') +
MOUNTPOINT(+
'/usr/lpp/IBM/db2tms/v2r1') +
TYPE(ZFS) MODE(RDWR)
MOUNT FILESYSTEM +
('QWT.OMVS.SGWROOT') +
MOUNTPOINT(+
'/usr/lpp/IBM/qwtz') +
TYPE(ZFS) MODE(RDWR)
/*
//NOTOK   EXEC PGM=CPPMAXRC,COND=((0,GE,STEP1),(4000,LT))
//*

```

That's all folks. QWT PSI is deployed. Next Step is Customisation.

[home](https://github.com/zeditor01/zowe_db2_tools/blob/main/docs/ZPDT_Build_Path.md)



