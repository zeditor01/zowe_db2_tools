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
![deploy_qwt_23](/images/deploy_qwt_23.jpg)
![deploy_qwt_24](/images/deploy_qwt_24.jpg)
![deploy_qwt_25](/images/deploy_qwt_25.jpg)
![deploy_qwt_26](/images/deploy_qwt_26.jpg)
![deploy_qwt_27](/images/deploy_qwt_27.jpg)
![deploy_qwt_28](/images/deploy_qwt_28.jpg)
![deploy_qwt_29](/images/deploy_qwt_29.jpg)

![deploy_qwt_30](/images/deploy_qwt_30.jpg)
![deploy_qwt_31](/images/deploy_qwt_31.jpg)
![deploy_qwt_32](/images/deploy_qwt_32.jpg)
![deploy_qwt_33](/images/deploy_qwt_33.jpg)
![deploy_qwt_34](/images/deploy_qwt_34.jpg)
![deploy_qwt_35](/images/deploy_qwt_35.jpg)
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




