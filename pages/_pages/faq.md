---
layout: page
title: FAQs
include_in_header: true
order: 3
---

# Frequently Asked Questions
{: .no_toc }

## Page Contents
{: .no_toc .text-delta }

* TOC
{:toc}



<br>
### What configuration is applied to the device when preparing it for migration?

The default configuration that is applied to the endpoint (temporarily) for the migration is outlined below.

Mover will use the settings defined in settings.json to:

- Configure a Local Administrator account on the endpoint and optionally set a random 15 character password.
  - The Administrator Account can be a newly created account (default)
  - The built-in Administrator account (this will re-enable the account temporarily)
  - Or you can specify an existing local administrator account with a supplied password
- Suspends Bitlocker (if present)
- Removes any Logon Banners (if present)
- Configures the device to Auto-Logon as the above Administrator Account
- Enforces Mover.exe as the default shell and places the endpoint in a Kiosk-like state
- Removes Logon/Log Off, Power Options and Task Manager from the CTRL+ALT+DEL screen
- If Windows Pro/Enterprise, enable keyboard filtering to further prevent access to Task Manager
- Disables Power Options access
- Enforces Mover fullscreen and inhibits it from being closed
- Disables User Account Control to ensure automated migration steps
- Sets the Administrator account to auto logon
- Reboots the PC and starts the next phase.
The device will now automatically reboot and migration will proceed.

After verifying that re-enrollment has successfully completed, Mover will revert any configuration made in the preparation stages.

The device is now ready for the end user to log in to and complete the device reassignment process by authenticating with their credentials in Intelligent Hub.
