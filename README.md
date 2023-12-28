
# Mover - Workspace ONE Migration Tool for Windows



# Overview

Mover is a tool for Workspace ONE Administrators that can be configured to facilitate an automated migration of a Windows device from either between two different Workspace ONE UEM Environments or have the same device re-enroll back into the _same_ UEM environment for user account/Directory changes and/or migrations.

**Mover now also supports migrating a Windows device managed by Microsoft Intune to Workspace ONE.**

Migrating a device managed by Intune has currently been tested for devices enrolled into Intune using OOBE. This process also leaves the device joined to **Azure Active Directory** and will remove all **Intune** CSPs and configurations. Applications that are deployed via Intune **are not** removed when migrating, as this is a limitation of Intune's software deployment capabilities. As is in Microsoft's documentation, if you want to remove applications before migration you must uninstall them first.


# High Level Process  

Below is a brief overview of the processes and tasks that Mover completes to migrate a device

-   A Workspace ONE Administrator deploys **Mover** to the Windows endpoint to be migrated. This can be done manually, or via application management using UEM etc.
-   After the **Mover** executables are on the device, a process runs the **mover_install.exe** binary which:

-   Enables the “Administrator” account on the PC
-   Sets a complex 15 char. password (not saved or stored anywhere in clear text), which is used by the autologon.exe process.

-   This is stored in an area of the Registry that is **only** accessible to the SYSTEM account.

-   Moves all the **Mover** files to C:\Recovery\OEM\Mover
-   Creates a scheduled task for the **Administrator** account, which triggers at logon, that calls the **migrate.exe** file from C:\Recovery\OEM\Mover
-   Sets the Administrator account to **auto logon**
-   Reboots the PC and starts the next phase.

-   The **Administrator** account **automatically logs in** and the task triggers.
-   The **migrate.exe** application reads its config from the **appsettings.json** file in the same directory, which includes the staging account details, server address, OG Name as well as parameters for the app.

-   You can force the app the run in **fullscreen** (default is **true**, _recommended_)

-   The **appsettings.json** also has a MdmSource parameter that is used to determine which migration scenario processes to follow.

You can also specify if you want to allow the Application to be able to be exited by the user (recommended to set **allow_exit = false** to stop interference with the process)

-   **Mover** will then download the latest AirwatchAgent.msi if **download_agent = true**, or will look for the agent in the same directory if **download_agent = false.**
-   It will then proceed with validation checks, disable Bitlocker, edit the install manifests to ensure that existing Applications do not uninstall(useful for re-enrollment into the same environment), uninstalls and reinstalls the agent with commandline provisioning.

After verifying that re-enrollment has successfully completed, **Mover** will re-enable Bitlocker, **disable** the Administrator account, and cleanup the scheduled migration task.

The device is now ready for the end user to log in to and complete the device reassignment process by authenticating with their credentials in Intelligent Hub.

# Downloads
The latest version of Mover is available here:

# Requirements

## Environment Details
___
If migrating from **Workspace ONE UEM** to **Workspace ONE UEM**:

-   Workspace ONE UEM Administrator accounts for both the **source** and **destination** environments, with permissions to manage staging accounts and devices

-   The _source_ Workspace ONE UEM Administrator account isn't _necessary_ but is useful for ensuring a successful migration.

-   Staging account credentials for the **destination Workspace ONE UEM** environment
-   **Workspace ONE UEM Destination** environment Organization Group ID & Device Services URL (eg. [https://ds1234.awmdm.com)](https://ds1234.awmdm.com))
-   Ability to deploy **Mover** on the device, or if completing manually, the ability to run mover_install.exe as an Administrator on the device

___
If migrating from **Microsoft Intune** to **Workspace ONE UEM**:

-   Administrator account for the **source** **Intune** Management Console.
-   Administrator account for the **destination** **Workspace ONE Environment**, with permissions to manage staging accounts and devices.
-   Staging account credentials for the **destination** **Workspace ONE UEM** environment
-   **Destination** environment Organization Group ID & Device Services URL (eg. [https://ds1234.awmdm.com)](https://ds1234.awmdm.com))
-   Intune permissions to deploy **Mover** on the device using the Win32 App deployment capabilities, or if completing manually, the ability to run mover_install.exe as an Administrator on the device.
___

# Device Requirements


-   Windows 10 or 11 Devices that can run supported versions of Workspace ONE Intelligent Hub
-   End Users **do not** need to be Administrators on the devices being migrated
-   The **mover_install.exe** process must run as an Administrator as it enables the local Administrator account on the device and schedules the migration task
-   Connectivity to the target environment is **mandatory** and internet connectivity is required to download the AirwatchAgent.msi

-   If you require a specific version of Intelligent Hub/Airwatch Agent, or if the endpoint _does not_ have internet access, ensure you place the required version of AirwatchAgent.msi in the root folder during installation.

-   The **Administrator** account on the device to be migrated **must exist** and **cannot** have been renamed etc.

# Usage Instructions

The interface is customizable by editing the "UI" section of the **appsettings.json** file (see below). Customers can also add their own **logo.png** file to the root directory to add their company logo, or remove it completely.



## Pre-Deployment Configuration

Before deploying the files to the endpoint, or running the migrate_install.exe **adjust** the included **appsettings.json** file and update the following parameters:

Under “**Settings**”:

_Note: These are all for the destination environment._

**Username:** Replace with your staging account username

**Password:** Replace with your staging account password

**OrgGroup:** Enter your OrgGroupID

**Server:** Enter your Device Services URL (including https:// )

**Fullscreen:** True forces Mover to take over the full screen of the device and disables the ability to minimize the application

**AllowExit:** If set to false, Mover is not allowed to exit and will ignore the close button presses

**RebootTime:** Not currently used

**MdmSource:** Specifies the source MDM to determine how to migrate the device

_You can also adjust the text that is displayed in the UI in the same **appsettings.json** file._

If you want to **remove** the VMware logo, delete the logo.png file

If you want to **add** a different company logo, add a new file called logo.png

## Deployment

### Workspace ONE UEM

Once you have configured the **appsettings.json** file, combine all the Mover files into a single .zip file.

You can then deploy this .zip file as a **Native Internal Application** in Workspace ONE UEM where the device is **currently** enrolled. After uploading the .zip file, use the following settings:

_Set the Install Command to be mover_install.exe_

<![if !vml]>![](file:///C:/Users/plindley/AppData/Local/Temp/msohtmlclip1/01/clip_image006.png)<![endif]>

_Set the Uninstall Command to be rm -force C:\Recovery\OEM\Mover_

<![if !vml]>![](file:///C:/Users/plindley/AppData/Local/Temp/msohtmlclip1/01/clip_image008.png)<![endif]>

_Set the Install Complete criteria to be file exists = C:\Recovery\OEM\Mover\mover.exe_

<![if !vml]>![](file:///C:/Users/plindley/AppData/Local/Temp/msohtmlclip1/01/clip_image010.png)<![endif]>

### IMPORTANT:

As soon as you assign this application and the device receives the installation command, **the device will start the migration immediately**.

**Intune**

Once you have configured the appsettings.json file, combine all the Mover files into a single directory.

If you do not have the **Intune Win32 Content Prep Tool** you can download it [here.](https://github.com/microsoft/Microsoft-Win32-Content-Prep-Tool) This is used for creating the .intunewin file for uploading to the Intune console.

When using the tool, the **source** directory is the whole Mover directory. The **install command** will be mover_install.exe.

Now upload the .intunewin file to your Intune Management Console using the below settings:


### IMPORTANT:

Take note of your assignment settings here. The above example sets this as **Available** which shows it in the Catalog but the user needs to "install" it to initate the migration. If you want to control the migration, add the devices to be migrated to the "required" section in phases etc.

As soon as you assign this application as **required**, or when the user initiates the install of the App, once the device receives the installation command, **the device will start the migration immediately**.

**Migration Demonstration**
