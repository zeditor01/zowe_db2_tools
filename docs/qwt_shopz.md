[home](https://github.com/zeditor01/zowe_db2_tools/blob/main/docs/ZPDT_Build_Path.md)

# Query Workload Tuner : Shop Z Download

## Step 2: Deploy SQL Tuning Services for Db2 z/OS.
1. [Download QWT Portable Software Instance from ShopZ](https://github.com/zeditor01/zowe_db2_tools/blob/main/docs/qwt_shopz.md) <<< You are here.
2. [Deploy QWT Portable Software Instance to your z/OS system](https://github.com/zeditor01/zowe_db2_tools/blob/main/docs/qwt_deploy.md)
3. [Customise QWT, to create an operational Liberty Server for it](https://github.com/zeditor01/zowe_db2_tools/blob/main/docs/qwt_customise.md)
4. [Installation Verification Testing of QWT](https://github.com/zeditor01/zowe_db2_tools/blob/main/docs/qwt_ivp.md)
5. Testing the Db2 for z/OS Developer Extension for VSCODE to invoke SQL Tuning Services

## Scope of this page

This page provides a worked example of downloading the Portable Software Instance of Query Workload Tuner (QWT) from ShopZ download server to your z/OS image.

## Starting point

It is assumed that you have already ordered QWT from ShopZ. The order should provide you with a SERVER XML block that allows you to perform a secure download from Shop Z. (The download creditials are valid for a short period, and these have since expired).

```
=== Order Size and File System Size Information ========================
                                                                        
The size of your order is 400 MB                                        
                                                                        
You need space in the file system used by z/OSMF Software Management    
Add Portable Software Instance for approximately twice the size of your 
order. To convert to 3390 cylinders, multiply the number of MB by 1.25  
and then multiply by 2.                                                 
                                                                        
For example, for a size of 5000 MB, then:                               
( (5,000 MB) * (1.25 CYL/MB) ) * 2 = 12,500 cylinders                   
                                                                        
== Server XML for Add Portable Software Instance From Download Server ==
You can copy the below statements into the z/OSMF Software Management   
Server XML box.                                                         
                                                                        
<SERVER                                                                 
  host="deliverycb-mul.dhe.ibm.com"                                     
  user="P078t735"                                                       
  pw="k35385q0669036E"                                                  
  >                                                                     
  <PACKAGE                                                              
      file="2025081900019/PROD/content/GIMPAF.XML"                      
      hash="550DF876335EB76C92991E6DCC721BF1D96E92D0"                   
      id="ST313139.content"                                             
   >                                                                    
  </PACKAGE>                                                            
</SERVER>     

```

z/OSMF software management application provides the necessary tooling to request the download. Point your browser at z/OSMF ( https://s0w1.dal-ebis.ihost.com:10443/zosmf ) and open the Software Management Appliction. Select "Portable Software Instances" (PSI) and then from the Actions pulldown request to Add a PSI from the Download Server, as shown below.

![shopz_qwt_01](/images/shopz_qwt_01.jpg)

For Step 1 of 4
1. Give the PSI a name (QWT)
2. optionally assign a category (Neale)
3. paste in the Server XML block
4. select the system name to be installed to (S0W1)
5. specify the USS directory where the download will be saved (/u/smpe/smpnts/QWT)

![shopz_qwt_02](/images/shopz_qwt_02.jpg)

For Step 2 of 4
1. Accept the default Client XML and Job card, or edit as required.
2. Note the JCL dataset name that will be allocated for the download jobs. (in case you need to edit it to resolve problems)
3. Press Next 

![shopz_qwt_03](/images/shopz_qwt_03.jpg)

For Step 3 of 4
1. Select the Download job
2. From the Actions pulldown, choose submit

![shopz_qwt_04](/images/shopz_qwt_04.jpg)

For your interest, you can open an shell into USS, and navigate to the download path. You should see all the files being downloaded as follows.

![shopz_qwt_05](/images/shopz_qwt_05.jpg)

When the job finishes, you should see a green "Complete" tick. Press Next.

![shopz_qwt_06](/images/shopz_qwt_06.jpg)

For Step 4 of 4
1. Review the name and description of the downloaded PSI
2. press "Finish"

![shopz_qwt_07](/images/shopz_qwt_07.jpg)

You will now see the downloaded PSIs. You can use the "Switch To" button to move onto Deploylents.

![shopz_qwt_08](/images/shopz_qwt_08.jpg)

That's all folks. You have downloaded the PSI. Next Step is to Deploy it.



