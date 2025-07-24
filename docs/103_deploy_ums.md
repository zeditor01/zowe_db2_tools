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

This worked example is NOT intended as a replacement for the installation instructions in the [UMS Knowledge Center](https://www.ibm.com/docs/en/umsfz/1.2.0?topic=server-installing-unified-management). Think of it as a worked example that should be helpful in conjucntion with the UMS Knowledge Center.


## 2.1 install UMS and DAF code. (it makes sense to install DAF and UMS together).

Order Unified Management Server and Db2 Admin Foundation together on ShopZ.
![order_ums_daf](/images/order_ums_daf.jpg)



Follow the standard PSI installation workflows. My HLQ was "DAFUMS". 


You will be required to permanently mount two ZFS filesystems as follows.

* UMS provides a ZFS called DAFUMS.OMVS.SIZPROOT - to mounted at /usr/lpp/IBM/izp/v1r2m0/bin
* DAF provides a ZFS called DAFUMS.OMVS.SAFXROOT - mounted at /usr/lpp/IBM/afx/v1r2m0/bin

UMS Zowe plug-ins require Program Control authorization. In order to tag the files with this bit, the SMP/E install user requires BPX.FILEATTR.PROGCTL permission on the system, which can be achieved with the following command from a USS shell.

```IBMUSER:/Z31A/usr/lpp/IBM/izp/v1r2m0/bin: >extattr +p */zssServer/lib/*```



The main pages of this repository are written to provide guidance to experienced system programmers who don't need to be told how to perform a PSI install. If you want an expanded version of this page covering the basic systems programmer tasks, please switch to [This Page](https://github.com/zeditor01/zowe_db2_tools/blob/main/docs/x103_deploy_ums_tasks.md) 


## 2.2 edit ZWEYAML parmlib member to configure UMS to integrate with z/OS and Zowe'




## 2.3 Execute the UMS installation workflows (including integration of zowe.yaml with UMS ZWEYAML.




## 2.4 start the zowe server (and likely debug initial UMS startup problems).



## 2.5 test Zowe from a web browser.



