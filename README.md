# Mover

**Automated MDM migration for Windows devices, to Workspace ONE UEM.**

Mover helps Workspace ONE administrators move Windows 10/11 devices:

- from **Microsoft Intune** to Workspace ONE UEM (the device stays joined to Entra ID),
- from one **Workspace ONE UEM** environment to another, or
- back into the **same** UEM environment, for example after a user or directory change.

Mover 2.x also has a **Generic MDM** source option. It runs an unenrollment script you supply to remove the source MDM, so other MDMs can be migrated.

[**Download the latest release**](https://github.com/tbwfdu/mover/releases) · [Documentation](pages/_pages) · [Demo video](https://youtu.be/k-Iyt-hFRuc)

---

## What's in Mover 2.x

Mover 2.x has three executables:

| Executable | Runs on | Purpose |
|:--|:--|:--|
| `MoverAdmin.exe` | The administrator's PC | A guided app that creates and validates the `settings.json` used for migration. |
| `MoverHelper.exe` | The device being migrated | Prepares the device and starts the migration when it receives `--initiate-migration`. |
| `Mover.exe` | The device being migrated | The full-screen shell that runs the migration and shows progress. |

**Changes from 1.x:**

- `settings.json` is generated and validated by **MoverAdmin**. You no longer edit `appsettings.json` by hand.
- **Staging and migrating are separate steps.** Deploying the Mover files no longer starts a migration. You start it later by running `MoverHelper.exe --initiate-migration`.
- The migration account can be a **new local administrator** (default username `mover`), the **built-in Administrator**, or an **existing admin account you specify**.
- Optional **settings encryption**. If you encrypt `settings.json`, pass `--passphrase <yourpassphrase>` when you initiate the migration.
- **Lockdown mode**, **minimal UI mode** and an optional **post-migration PowerShell script**.
- Options to **wait for Omnissa Tunnel and Offline Domain Join** before a migration counts as complete.
- A **localized MoverAdmin UI**: English, French, German, Dutch and Spanish.

## How it works

1. **Configure.** Run `MoverAdmin.exe` and enter the destination environment, device, migration and post-migration options. MoverAdmin validates the combination and writes `settings.json`.
2. **Stage.** Deploy `Mover.exe`, `MoverHelper.exe` and `settings.json` to the devices (see [Deployment](#deployment)). Do **not** deploy `MoverAdmin.exe`.
3. **Initiate.** When you're ready, send this command to the device:

   ```powershell
   Start-Process "C:\Temp\Mover\MoverHelper.exe" -ArgumentList @('--initiate-migration')
   ```

4. **Prepare.** `MoverHelper` configures the device and reboots it into the Mover shell.
5. **Migrate.** `Mover.exe` unenrolls the device from the source MDM, downloads the Intelligent Hub (`AirwatchAgent.msi`) and enrolls the device in the destination Workspace ONE UEM environment.
6. **Revert.** After checking that enrollment succeeded, Mover reverts every preparation change. The user then signs in to Intelligent Hub to finish device assignment.

### What Mover changes on the device during migration

These changes are temporary, and Mover reverts them when migration succeeds.

- Creates or enables a local administrator account (optionally with a random 15-character password) and sets it to auto-logon
- Suspends BitLocker and removes any logon banner
- Sets `Mover.exe` as the shell, so the device is in a kiosk-like state
- Removes Log off, Power options and Task Manager from the Ctrl+Alt+Del screen, and enables keyboard filtering on Pro/Enterprise
- Disables UAC for the duration of the migration
- Runs Mover full screen and stops it being closed (configurable)

See the [FAQ](pages/_pages/faq.md) for more detail.

## Requirements

**Destination (Workspace ONE UEM)**

- A staging account configured for single-user staging
- The Organization Group ID where devices will enroll
- The Device Services URL (`https://dsXXXX.awmdm.com`; use `ds`, not `cn` or `as`)

**Source MDM**

- *Workspace ONE UEM:* administrator access to the source and destination environments. The source account isn't strictly required but is recommended.
- *Intune:* administrator access to Intune and permission to deploy Win32 apps, or the ability to run `MoverHelper.exe` as an administrator by hand.
- *Generic MDM:* an unenrollment script that removes the OMA-DM accounts and any MDM-specific agents, and returns exit code `0`.

**Devices**

- Windows 10 or 11 that can run a supported version of Workspace ONE Intelligent Hub
- Connectivity to the destination environment. Internet access is needed to download `AirwatchAgent.msi`. Without it, place the version you want in the Mover folder.
- `MoverHelper.exe` must run as an administrator. End users don't need to be administrators.

## Deployment

Deliver `Mover.exe`, `MoverHelper.exe` and `settings.json` to each device. Choose a location standard users can't write to; the examples use `C:\Temp\Mover`. The release includes `ZipCurrentDirectory.ps1`, which zips the current directory for deployment.

- **Workspace ONE UEM:** deploy the zip as a *Native Internal Application* with install command `install.ps1`, uninstall command `rm -force C:\Temp\Mover`, and detection `file exists = C:\Temp\Mover\Mover.exe`. Set the assignment to automatic.
- **Intune:** package the folder with the [Win32 Content Prep Tool](https://github.com/microsoft/Microsoft-Win32-Content-Prep-Tool) and assign it as *required*.

To start migrations, run `MoverHelper.exe --initiate-migration` on the device. Use a UEM script or workflow with the execution context set to *User Context with Admin Privileges*. In Intune, use a Platform Script, or set the package install command to `MoverHelper.exe --initiate-migration` to start the migration on install.

The full step-by-step guide is in [Usage](pages/_pages/usage.md).

## Documentation

The documentation is a Jekyll site in [`pages/`](pages):

- [Getting Started](pages/_pages/gettingstarted.md): overview, process and requirements
- [Usage](pages/_pages/usage.md): MoverAdmin settings reference, validation, deployment and initiating a migration
- [FAQ](pages/_pages/faq.md): what configuration is applied to the device

To preview it locally, run `bundle install` and `bundle exec jekyll serve` in `pages/`.

## Limitations

- Applications deployed by Intune are **not** removed during migration (an Intune limitation). Uninstall them first if needed.
- Intune migrations have been tested with devices enrolled via OOBE.
- Starting a migration on a device that is already staged requires `MoverHelper.exe --initiate-migration`. Running the installer alone no longer does it.

## Support and license

Mover is a community project and is not an officially supported product. Report issues on the [GitHub issue tracker](https://github.com/tbwfdu/mover/issues).

Released under the [MIT License](LICENSE).
