---
layout: page
title: Getting Started
include_in_header: true
order: 1
---
# Getting Started

Mover is a tool for Workspace ONE Administrators that can be configured to facilitate an automated migration of a Windows device from either between two different Workspace ONE UEM Environments or have the same device re-enroll back into the _same_ UEM environment for user account/Directory changes and/or migrations.

**Mover also supports migrating a Windows device managed by Microsoft Intune to Workspace ONE.**

Migrating a device managed by Intune has currently been tested for devices enrolled into Intune using OOBE. This process also leaves the device joined to **Azure Active Directory** and will remove all **Intune** CSPs and configurations. Applications that are deployed via Intune **are not** removed when migrating, as this is a limitation of Intune's software deployment capabilities. As is in Microsoft's documentation, if you want to remove applications before migration you must uninstall them first.



## High Level Process

Below is a brief overview of the processes and tasks that Mover completes to migrate a device

- A Workspace ONE Administrator downloads the **Mover** files and unzips it locally on _their_ PC to do the initial configuration.

- Once installed, there will be two executables in the destination directory:
  ### Mover.exe
    - The shell that facilitates the migration tasks and is used on the device **to be migrated** 
  
  ### MoverAdmin.exe
    - Used by the Workspace ONE Administrator to generate the settings.json file used for migration

  ### MoverHelper.exe
    - Called remotely by the source MDM and configures the device **to be migrated** with the settings in the settings.json file created by the Administrator  
  <br>

- A Workspace ONE Administrator then deploys **Mover.exe** & **MoverHelper.exe** (two executables + settings.json) to the Windows endpoint to be migrated.
  - This can be done at any time to prepare the device for the migration.

- When the Administrator is ready to migrate the device, a remote command is sent to the device _(via script, logon script, scheduled task etc.)_ that calls:
  
    > **"MoverHelper.exe --initiate-migration"**

-   This will then prepare the endpoint for the migration, reboot the device and migrate the device from the source MDM to the destination MDM automatically.



## Environment Requirements

### ℹ️ If migrating from **Workspace ONE UEM** to **Workspace ONE UEM**:

-   Workspace ONE UEM Administrator accounts for both the **source** and **destination** environments, with permissions to manage staging accounts and devices

-   The _source_ Workspace ONE UEM Administrator account isn't _necessary_ but is useful for ensuring a successful migration.

-   Staging account credentials for the **destination Workspace ONE UEM** environment
-   **Workspace ONE UEM Destination** environment Organization Group ID & Device Services URL (eg. [https://ds1234.awmdm.com)](https://ds1234.awmdm.com))
-   Ability to deploy **Mover** on the device, or if completing manually, the ability to run mover_install.exe as an Administrator on the device

### ℹ️ If migrating from **Microsoft Intune** to **Workspace ONE UEM**:

-   Administrator account for the **source** **Intune** Management Console.
-   Administrator account for the **destination** **Workspace ONE Environment**, with permissions to manage staging accounts and devices.
-   Staging account credentials for the **destination** **Workspace ONE UEM** environment
-   **Destination** environment Organization Group ID & Device Services URL (eg. [https://ds1234.awmdm.com)](https://ds1234.awmdm.com))
-   Intune permissions to deploy **Mover** on the device using the Win32 App deployment capabilities, or if completing manually, the ability to run MoverHelper.exe as an Administrator on the device.

## Device Requirements

-   Windows 10 or 11 Devices that can run supported versions of Workspace ONE Intelligent Hub
-   End Users **do not** need to be Administrators on the devices being migrated
-   The **MoverHelper.exe** process must run as an Administrator as it configures the migration Administrator account on the device and sets up the Mover interface
-   Connectivity to the target environment is **mandatory** and internet connectivity is required to download the AirwatchAgent.msi
-   If you require a specific version of Intelligent Hub/Airwatch Agent, or if the endpoint _does not_ have internet access, ensure you place the required version of AirwatchAgent.msi in the root folder during installation.