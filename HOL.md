<a name="Title" />
#Upload VMs from On-Premise to Cloud#

---
<a name="Overview" />
## Overview ##

In this lab, you will upload an existing Hyper-V vhd-file from your **DC01** VM to Windows Azure. In this case, **DC01** represents the "on-premise" location.

After the upload, the vhd-file can be used to create a new virtual machine in Windows Azure, or can be added as a data disk to an existing Azure virtual machine.

>Note: In this lab, in order to avoid a dependency on the bandwidth between any on-premise location and the Azure datacenters, we have copied vhd files in the previous lab and created a new VM from those files called DC01 running on Azure.  Although DC01 is not truly on-premise, you would perform the exact same steps to manually upload a real on-premise VM.

The lab will use PowerShell commands to upload the vhd-file.

<a name="Exercise1" />
## Excercise 1: Upload VMs from on-premise to Cloud ##

**Tasks in this Lab Module**

* [Configure Windows Azure PowerShell](#Ex1Task1)
* [Upload the vhd-files to Windows Azure](#Ex1Task2)

<a name="Ex1Task1" />
###Task 1 – Configure Windows Azure PowerShell###

1. In the Azure portal, in the **Virtual Machines** section, select **DC01**, and then click **Connect**.

	![Connecting to the Virtual Machine](images/Do-You-Want-To-Save-This-file-dialog.png?raw=true "Connecting to the Virtual Machine")

	>Note: For the purpose of this lab, the DC01 represents the on-premise environment.
You will _upload_ a vhd-file from DC01 to the storage account in Azure.

1. Log on to the DC01 with the credentials:

	| Field | Value |
	|--------|--------|
	| User name | **Contoso\Administrator** |
	| Password | **Passw0rd!** |

	![Log on using your credentials.](./images/Enter-Your-Credentials.png?raw=true "Log on using your credentials.")

1. In the DC01 virtual machine, on the **Start** menu, click **Windows Azure PowerShell**.
_Windows Azure PowerShell is already installed in DC01._

	![Start Windows Azure PowerShell](./images/Start-Windows-Azure-PowerShell.png?raw=true "Start Windows Azure PowerShell")

1. In the Windows Azure PowerShell window, run the following snippet
and then press **Y** to confirm.

	````PowerShell 
	Set-ExecutionPolicy  RemoteSigned
	````

	![Setting Execution Policy](./images/Setting-Execution-Policy.png?raw=true "Set Execution policy")

	To establish a connection between Azure PowerShell and your Azure account, you must install a certificate in the Azure environment. You can do this manually, by providing a loading a suitable certificate, or do this automatically with the command below.

1. In the PowerShell window, run

	````PowerShell 
	Get-AzurePublishSettingsFile
	````

	![Getting Azure Publish Settings](./images/Getting-Azure-Publish-Settings.png?raw=true "Getting Azure Publish Settings")

1. On the Sign In page, log on with your Microsoft Account (Live ID) related to the Azure subscription.
_After log on, this opens the **https://windows.azure.com/download/publishprofile.aspx** page._

	![Windows-Azure-Logon-Page-](images/Windows-Azure-Logon-Page-.png?raw=true)

	![Save-Publish-Settings-](images/Save-Publish-Settings-.png?raw=true)

1. In the browser, in the yellow notification bar, click **Save As**.
Save the **.publishsettings** file in the **C:\Files** folder.
Close the browser window.

	![Saved Publish Settings](images/Saved-Publish-Settings-.png?raw=true "Saved Azure Publish Settings")

1. In the PowerShell window, run

	````PowerShell 
	Import-AzurePublishSettingsFile  -PublishSettingsFile  C:\Files\<myfilename>.publishsettings
	````
![Importing Azure Publish Settings to PowerShell](images/Importing-Azure-Publish-Settings-to-PowerShell.png?raw=true "Importing Azure Publish Settings to PowerShell")

1. To display the SubscriptionName, and the generated management certificate, run

	````PowerShell 
	Get-AzureSubscription
	````

	![Displaying Subscription Information](./images/displaying-subscription-information.png?raw=true "Displaying Subscription Information")

1. To set the default subscription and storage account. You need to replace NNNN by the unique number you selected earlier. The Azure PowerShell command for uploading a vhd-file requires a storage account. In the PowerShell window, run

	````PowerShell 
	Set-AzureSubscription  -SubscriptionName <mysubname>  -CurrentStorageAccount "itcampNNNN"
Get-AzureSubscription
	````

	![Setting Default Subscription Information](./images/Setting-Default-Subscription-Information.png?raw=true "Setting Default Subscription Information")

<a name="Ex1Task2" />
###Task 2 – Upload the vhd-files to Windows Azure##
1. In the DC01 virtual machine, use Windows Explorer to open the **C:\Files** folder.

	![Locating-VHD-Image-On-Disk](images/Locating-VHD-Image-On-Disk.png?raw=true "Locating VHD Image On Disk")

	>The folder contains as sample Hyper-V vhd-file named **itcamp-vhdfile.vhd**.
The current size of the vhd-file is **800 MB**.
You will upload this vhd-file to Windows Azure storage.

1. In the C:\Files folder, right-click itcamp-vhdfile.vhd, and click **Mount**.
_After a few moments, the vhd-file will be mounted as drive F:.

	![Mounting-VHD-Image](images/Mounting-VHD-Image.png?raw=true "Mounting VHD Image")

1. Use Windows Explorer to open **Computer**.

	![Mounted-VHD-Image](images/Mounted-VHD-Image.png?raw=true "Mounted VHD Image")

	>Notice that the F: drive has a capacity of 3.99 GB.
The vhd-file is of dynamically-expanding type, with a maximum capacity of 4 GB.

1. Right-click **Local Disk (F:)**, and then click **Eject**.

	![Ejecting-a-Mounted-VHD](images/Ejecting-a-Mounted-VHD.png?raw=true "Ejecting a Mounted VHD")

	Azure only supports **fixed size** vhd-files. As you will see in the next step, the Azure PowerShell command to upload a vhd-file automatically converts the dynamically expanding vhd-file to fixed size.

	Also, note that Azure supports the **vhd**-file format, not the new **vhdx**-file format yet. However, you can use Hyper V in Windows Server 2012 to convert a vhdx-file to vhd-file format (if it is less than 127 GB in size).

	For lab purposes, this is a small sample Hyper-V vhd-file. You will not load it as virtual machine after the upload.

1. In the PowerShell window, run the following command on a single line

	````PowerShell 
	Add-AzureVHD 
		-LocalFilePath "C:\Files\itcamp-vhdfile.vhd" 
		-Destination "http://itcampNNNN.blob.core.windows.net/itcamp/itcamp-vhdfile.vhd"
	````

	>The command calculates a hash of the file contents, creates the blob container (itcamp) if it does not exist yet, and then uploads the file to the storage account. The PowerShell cmdlet also converts the dynamically expanding vhd-file to a fixed size file.

	> After a few moments, a new 4 GB fixed-size vhd-file is available in the storage account.
	
	![Calculating-MD5-Hash](images/Calculating-MD5-Hash.png?raw=true "Calculating MD5 Hash")

	![Adding-VHD](images/Adding-VHD.png?raw=true "Adding VHD")

1. After the upload is completed, go to the Azure portal.
In the Azure portal, in the left column, select **Virtual Machines**.
On the Virtual Machine page, select the **Disks** tab.

	![Selecting-VM](images/Selecting-VM.png?raw=true "Selecting VM")

	> **Note:** Before the availability of the **Add-AzureVHD** cmdlet, Microsoft provided a command line tool named **csupload.exe** with similar functionality. That command line tool is now depreciated.
The csupload.exe tool automatically created a Disk for the uploaded vhd-file. With the Add-AzureVHD cmdlet, you run the **Add-AzureDisk** cmdlet separate so you can specify whether you want the disk to be for data or OS.  Specifying an OS disk will make the disk bootable._

1. On the toolbar, click the **Create** button.
In the Create a disk from a VHD dialog box, under **VHD URL**, click the folder (Browse) icon.

	![Creating-a-Disk-From-a-VHD](images/Creating-a-Disk-From-a-VHD.png?raw=true "Creating a Disk From a VHD")

1. In the Browse cloud storage dialog box, in the left pane, expand storage account **itcamp6567**, and then select storage container **itcamp**.
![Verifying-Uploaded-VHD](images/Verifying-Uploaded-VHD.png?raw=true "Verifying Uploaded VHD")
	>Notice that in the storage browser, you can find the newly uploaded itcamp-vhdfile.vhd, with a fixed size of 4 GB. This result confirms that the vhd file is uploaded._

1. Close the Browse cloud storage dialog box.
Close the Create a disk from a VHD dialog box.
In this lab exercise, the vhd-file was uploaded for demonstration purposes.
If you want to load the vhd-file as data disk for a VM, or as OS disk for a new VM, you can use the same PowerShell commands that we used in an earlier exercise to create a VM based on an available vhd-file.

	For reference, here are the commands to load the vhd-files as OS disk for a new VM:

	```` PowerShell 
	# Defines source vhd-files in storage
	$vhdfile = "http://itcampNNNN.blob.core.windows.net/itcamp/itcamp-vhdfile.vhd"

	# Creates OS disk
	Add-AzureDisk  -OS  Windows  -Medialocation  $vhdfile  -DiskName  "itcamp-vhdfile"

	# Defines VM configuration, including size, and network endpoint
	$vm = New-AzureVMConfig  -Name  "NewVM"  -DiskName  "itcamp-vhdfile" 	-InstanceSize "Small"  |
   Add-AzureEndPoint -Name  "RemoteDesktop"  -LocalPort  3389  -Protocol  TCP

	# Creates new VM in new hosted service
	New-AzureVM  -ServiceName  "itcampNNNN"  -AffinityGroup  "agdomain" -VMs  $vm
	````





