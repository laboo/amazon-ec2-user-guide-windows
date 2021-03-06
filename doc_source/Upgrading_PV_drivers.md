# Upgrading PV Drivers on Your Windows Instances<a name="Upgrading_PV_drivers"></a>

To verify which driver your Windows instance uses, open **Network Connections** in Control Panel and view the **Local Area Connection**\. Check whether the driver is one of the following:
+  AWS PV Network Device 
+  Citrix PV Ethernet Adapter 
+  RedHat PV NIC Driver 

Alternatively, you can check the output from the `pnputil -e` command\.

**Topics**
+ [Upgrade Windows Server Instances \(AWS PV Upgrade\)](#aws-pv-upgrade)
+ [Upgrade Windows Server 2016, Nano Edition \(AWS PV Upgrade\)](#aws-pv-upgrade-nano)
+ [Upgrade a Domain Controller \(AWS PV Upgrade\)](#aws-pv-upgrade-dc)
+ [Upgrade Windows Server 2008 and 2008 R2 Instances \(Redhat to Citrix PV Upgrade\)](#win2008-citrix-upgrade)
+ [Upgrade Your Citrix Xen Guest Agent Service](#citrix-pv-guest-agent-upgrade)

## Upgrade Windows Server Instances \(AWS PV Upgrade\)<a name="aws-pv-upgrade"></a>

Use the following procedure to perform an in\-place upgrade of AWS PV drivers, or to upgrade from Citrix PV drivers to AWS PV drivers on Windows Server 2008 R2, Windows Server 2012, Windows Server 2012 R2, or Windows Server 2016\. This upgrade is not available for RedHat drivers, or for other versions of Windows Server\.

**Important**  
If your instances uses Windows Server 2016, Nano edition, see [Upgrade Windows Server 2016, Nano Edition \(AWS PV Upgrade\)](#aws-pv-upgrade-nano)\. If your instance is a domain controller, see [Upgrade a Domain Controller \(AWS PV Upgrade\)](#aws-pv-upgrade-dc)\. The upgrade process for these instances is different than standard editions of Windows\. 

**To upgrade AWS PV drivers**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. In the navigation pane, choose **Instances**\.

1. Choose the instance that requires the driver upgrade, open the context \(right\-click\) menu, choose **Instance State**, and then choose **Stop**\.
**Warning**  
When you stop an instance, the data on any instance store volumes is erased\. Therefore, if you have any data on instance store volumes that you want to keep, be sure to back it up to persistent storage\.

1. After the instance is stopped, create a backup\. Open the context \(right\-click\) menu for the instance, choose **Image**, and then choose **Create Image**\.

1. From the context \(right\-click\) menu for the instance, choose **Instance State**, and then choose **Start**\.

1. Connect to the instance using Remote Desktop and prepare the instance for upgrade\. We recommend that you take all non\-system disks offline before you perform this upgrade\. Note that this step is not required if you are performing an in\-place update of AWS PV drivers\. We also recommend setting non\-essential services to **Manual** start\-up in the Services console\.

1. [Download](https://s3.amazonaws.com/ec2-windows-drivers-downloads/AWSPV/Latest/AWSPVDriver.zip) the latest driver package to the instance\.

1. Extract the contents of the folder and then run `AWSPVDriverSetup.msi`\.

After running the MSI, the instance automatically reboots and then upgrades the driver\. The instance will not be available for up to 15 minutes\. After the upgrade is complete and the instance passes both health checks in the Amazon EC2 console, connect to the instance using Remote Desktop and verify that the new driver was installed\. In Device Manager, under **Storage Controllers**, locate **AWS PV Storage Host Adapter**\. Verify that the driver version is the same as the latest version listed in the Driver Version History table\. For more information, see [AWS PV Driver Version History](xen-drivers-overview.md#pv-driver-history)\.

If you previously disabled [TCP Offloading](pvdrivers-troubleshooting.md#citrix-tcp-offloading) using Netsh for Citrix PV drivers we recommend that you re\-enable this feature after upgrading to AWS PV drivers\. TCP Offloading issues with Citrix drivers are not present in the AWS PV drivers\. As a result, TCP Offloading provides better performance with AWS PV drivers\.

If you previously applied a static IP address or DNS configuration to the network interface, you must reapply the static IP address or DNS configuration after upgrading AWS PV drivers\.

## Upgrade Windows Server 2016, Nano Edition \(AWS PV Upgrade\)<a name="aws-pv-upgrade-nano"></a>

The following procedure describes how to upgrade AWS PV drivers on the Windows Server 2016, Nano edition\. 

**Before You Begin**  
The following procedure uses PowerShell remoting to install the latest driver package on the instance\. Before you begin, verify that TCP port 5985 is open on the instance\.

**To upgrade AWS PV drivers on Nano edition**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. In the navigation pane, choose **Instances**\.

1. Choose the instance that requires the driver upgrade, open the context \(right\-click\) menu, choose **Instance State**, and then choose **Stop**\.
**Warning**  
When you stop an instance, the data on any instance store volumes is erased\. Therefore, if you have any data on instance store volumes that you want to keep, be sure to back it up to persistent storage\.

1. After the instance is stopped, create a backup\. Open the context \(right\-click\) menu for the instance, choose **Image**, and then choose **Create Image**\.

1. From the context \(right\-click\) menu for the instance, choose **Instance State**, and then choose **Start**\.

1. [Download](https://s3.amazonaws.com/ec2-downloads-windows/Drivers/AWSPVDriverNanoSetup.zip) and extract the latest driver package to your local computer\. The upgrade script runs on your local computer and creates a remote PowerShell session to your Nano instance to install the latest drivers\.

1. Navigate to the directory where you unzipped the installation package\.

1. Run `UpgradeDriver.ps1 –HostName <instance public DNS name>` \-UserName <*a user with admin rights*>\.

   For example, `UpgradeDriver.ps1 –HostName ec2-123-45-678-90.compute-1.amazonaws.com –UserName Administrator`\. 

1. When prompted, specify the instance password, and press Enter\.

The installation can take several minutes to complete\. After a successful installation, the system shows the following message: "Upgrade process finished successfully"\.

## Upgrade a Domain Controller \(AWS PV Upgrade\)<a name="aws-pv-upgrade-dc"></a>

Use the following procedure on a domain controller to perform either an in\-place upgrade of AWS PV drivers, or to upgrade from Citrix PV drivers to AWS PV drivers\.

**To upgrade a domain controller**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. In the navigation pane, choose **Instances**\.

1. Choose the instance that requires the driver upgrade, open the context \(right\-click\) menu, choose **Instance State**, and then choose **Stop**\.
**Warning**  
When you stop an instance, the data on any instance store volumes is erased\. Therefore, if you have any data on instance store volumes that you want to keep, be sure to back it up to persistent storage\.

1. After the instance is stopped, create a backup\. Open the context \(right\-click\) menu for the instance, choose **Image**, and then choose **Create Image**\.

1. From the context \(right\-click\) menu for the instance, choose **Instance State**, and then choose **Start**\.

1. Run the following command to configure Windows to boot into Directory Services Restore Mode \(DSRM\):

   ```
   bcdedit /set {default} safeboot dsrepair
   ```
**Warning**  
Before running this command, confirm that you know the DSRM password\. You'll need this information so that you can log in to your instance after the upgrade is complete and the instance automatically reboots\.

   The system must boot into DSRM because the upgrade utility removes Citrix PV storage drivers so it can install AWS PV drivers\. When Citrix PV storage drivers are not present, secondary drives will not be detected\. Domain controllers that use an NTDS folder on secondary drives will not boot because the secondary disk will not be detected\.
**Warning**  
After you run this command do not manually reboot the system\. The system will be unreachable because Citrix PV drivers do not support DSRM\.

1. Run the following command to add **DisableDCCheck** to the registry:

   ```
   reg add HKLM\SOFTWARE\Wow6432Node\Amazon\AWSPVDriverSetup /v DisableDCCheck /t REG_SZ /d true
   ```

1. [Download](https://s3.amazonaws.com/ec2-windows-drivers-downloads/AWSPV/Latest/AWSPVDriver.zip) the latest driver package to the instance\.

1. Extract the contents of the folder and then run `AWSPVDriverSetup.msi`\.

   After running the MSI, the instance automatically reboots and then upgrades the driver\. The instance will not be available for up to 15 minutes\. 

1. After the upgrade is complete and the instance passes both health checks in the Amazon EC2 console, connect to the instance using Remote Desktop\.
**Important**  
You must connect to the instance by specifying user name in the following format *hostname*\\administrator\. For example, Win2k12TestBox\\administrator\.

1. Run the following command to remove the DSRM boot configuration:

   ```
   bcdedit /deletevalue safeboot
   ```

1. Reboot the instance\.

1. To complete the upgrade process, verify that the new driver was installed\. In Device Manager, under **Storage Controllers**, locate **AWS PV Storage Host Adapter**\. Verify that the driver version is the same as the latest version listed in the Driver Version History table\. For more information, see [AWS PV Driver Version History](xen-drivers-overview.md#pv-driver-history)\.

1. Run the following command to delete **DisableDCCheck** from the registry:

   ```
   reg delete HKLM\SOFTWARE\Wow6432Node\Amazon\AWSPVDriverSetup /v DisableDCCheck
   ```

**Note**  
If you previously disabled [TCP Offloading](pvdrivers-troubleshooting.md#citrix-tcp-offloading) using Netsh for Citrix PV drivers we recommend that you re\-enable this feature after upgrading to AWS PV Drivers\. TCP Offloading issues with Citrix drivers are not present in the AWS PV drivers\. As a result, TCP Offloading provides better performance with AWS PV drivers\.

## Upgrade Windows Server 2008 and 2008 R2 Instances \(Redhat to Citrix PV Upgrade\)<a name="win2008-citrix-upgrade"></a>

Before you start upgrading your RedHat drivers to Citrix PV drivers, make sure you do the following:
+ Install the latest version of the EC2Config service\. For more information, see [Installing the Latest Version of EC2Config](UsingConfig_Install.md)\.
+ Verify that you have Windows PowerShell 2\.0 installed\. To verify the version that you have installed, run the following command in a PowerShell window:

  ```
  PS C:\> $PSVersionTable.PSVersion
  ```

  If you need to install version 2\.0, see [Windows Management Framework \(Windows PowerShell 2\.0, WinRM 2\.0, and BITS 4\.0\)](http://support2.microsoft.com/kb/968929/en-us) from Microsoft Support\.
+ Back up your important information on the instance, or create an AMI from the instance\. For more information about creating an AMI, see [Creating a Custom Windows AMI](Creating_EBSbacked_WinAMI.md)\. If you create an AMI, make sure that you do the following:
  + Write down your password\.
  + Do not run the Sysprep tool manually or using the EC2Config service\.
  + Set your Ethernet adapter to obtain an IP address automatically using DHCP\. For more information, see [Configure TCP/IP Settings](https://technet.microsoft.com/en-us/library/cc731673.aspx) in the Microsoft TechNet Library\.

**To upgrade Redhat drivers**

1. Connect to your instance and log in as the local administrator\. For more information about connecting to your instance, see [Connecting to Your Windows Instance](connecting_to_windows_instance.md)\.

1. In your instance, [download](https://s3.amazonaws.com/ec2-downloads-windows/Drivers/Citrix-Win_PV.zip) the Citrix PV upgrade package\.

1. Extract the contents of the upgrade package to a location of your choice\.

1. Double\-click the **Upgrade\.bat** file\. If you get a security warning, choose **Run**\.

1. In the **Upgrade Drivers** dialog box, review the information and choose **Yes** if you are ready to start the upgrade\.

1. In the **Red Hat Paravirtualized Xen Drivers for Windows uninstaller** dialog box, choose **Yes** to remove the RedHat software\. Your instance will be rebooted\.
**Note**  
If you do not see the uninstaller dialog box, choose **Red Hat Paravirtualize** in the Windows taskbar\.  
![\[Red Hat Paravirtualized in taskbar\]](http://docs.aws.amazon.com/AWSEC2/latest/WindowsGuide/images/win2003-citrix-taskbar.png)

1. Check that the instance has rebooted and is ready to be used\.

   1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

   1. On the **Instances** page, right\-click your instance and select **Get System Log**\.

   1. The upgrade operations should have restarted the server 3 or 4 times\. You can see this in the log file by the number of times `Windows is Ready to use` is displayed\.  
![\[Windows system log\]](http://docs.aws.amazon.com/AWSEC2/latest/WindowsGuide/images/win2008-sys-log.png)

1. Connect to your instance and log in as the local administrator\.

1. Close the **Red Hat Paravirtualized Xen Drivers for Windows uninstaller** dialog box\.

1. Confirm that the installation is complete\. Navigate to the `Citrix-WIN_PV` folder that you extracted earlier, open the `PVUpgrade.log` file, and then check for the text `INSTALLATION IS COMPLETE`\.  
![\[PVUpgrade log file\]](http://docs.aws.amazon.com/AWSEC2/latest/WindowsGuide/images/win2008-pvupgrade-log.png)

## Upgrade Your Citrix Xen Guest Agent Service<a name="citrix-pv-guest-agent-upgrade"></a>

If you are using Citrix PV drivers on Windows Server, you can upgrade the Citrix Xen guest agent service\. This Windows service handles tasks such as shutdown and restart events from the API\. You can run this upgrade package on any version of Windows Server, as long as the instance is running Citrix PV drivers\.

**Important**  
Do not perform these steps on Windows Server 2012 or 2012 R2 instances that are running AWS PV drivers\.

Before you start upgrading your drivers, make sure you back up your important information on the instance, or create an AMI from the instance\. For more information about creating an AMI, see [Creating a Custom Windows AMI](Creating_EBSbacked_WinAMI.md)\. If you create an AMI, make sure you do the following:
+ Do not enable the Sysprep tool in the EC2Config service\.
+ Write down your password\.
+ Set your Ethernet adapter to DHCP\. 

**To upgrade your Citrix Xen guest agent service**

1. Connect to your instance and log in as the local administrator\. For more information about connecting to your instance, see [Connecting to Your Windows Instance](connecting_to_windows_instance.md)\.

1. On your instance, [download](https://s3.amazonaws.com/ec2-downloads-windows/Drivers/Citrix-Win_PV.zip) the Citrix upgrade package\.

1. Extract the contents of the upgrade package to a location of your choice\.

1. Double\-click the **Upgrade\.bat** file\. If you get a security warning, choose **Run**\.

1. In the **Upgrade Drivers** dialog box, review the information and choose **Yes** if you are ready to start the upgrade\. 

1. When the upgrade is complete, the `PVUpgrade.log` file will open and contain the text `UPGRADE IS COMPLETE`\.

1. Reboot your instance\. 