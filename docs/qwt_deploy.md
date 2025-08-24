# QWT Deploy

wwwwow


![deploy_qwt_01](/images/deploy_qwt_01.jpg)

![deploy_qwt_02](/images/deploy_qwt_02.jpg)

![deploy_qwt_03](/images/deploy_qwt_03.jpg)

![deploy_qwt_04](/images/deploy_qwt_04.jpg)

![deploy_qwt_05](/images/deploy_qwt_05.jpg)

![deploy_qwt_06](/images/deploy_qwt_06.jpg)

![deploy_qwt_07](/images/deploy_qwt_07.jpg)

![deploy_qwt_08](/images/deploy_qwt_08.jpg)

![deploy_qwt_09](/images/deploy_qwt_09.jpg)

![deploy_qwt_10](/images/deploy_qwt_10.jpg)

![deploy_qwt_11](/images/deploy_qwt_11.jpg)

![deploy_qwt_12](/images/deploy_qwt_12.jpg)

![deploy_qwt_13](/images/deploy_qwt_13.jpg)

![deploy_qwt_14](/images/deploy_qwt_14.jpg)

![deploy_qwt_15](/images/deploy_qwt_15.jpg)

![deploy_qwt_16](/images/deploy_qwt_16.jpg)

![deploy_qwt_17](/images/deploy_qwt_17.jpg)

![deploy_qwt_18](/images/deploy_qwt_18.jpg)

![deploy_qwt_19](/images/deploy_qwt_19.jpg)


![deploy_qwt_20](/images/deploy_qwt_20.jpg)

![deploy_qwt_21](/images/deploy_qwt_21.jpg)

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

missing image ?
![deploy_qwt_52](/images/deploy_qwt_52.jpg)

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


