---
layout: page
title: Hybrid - Microsoft Endpoint Manager - Intune enrolment for iOS devices
menu: abac
---

## Apple Automated Device Enrollment

`Microsoft Endpoint Manager > Devices > Enroll devices > Apple enrollment > Enrollment program tokens`

* Name: `<Agency Acronym> iOS ADE enrolment`
* Description: -
* Platform: `iOS`
* Token
  * Token name: `Intune ADE`
  * Program type: `Apple Business Manager`
  * Apple ID: `<Agency ID>@<Agency>.gov.au`
  * Token: `Generated in Apple Business Manager Portal below`

`Apple Business Manager Portal > Settings > Device Management Settings`

* Server Name: `Intune MDM`
* Authorisation information: `PEM file from Microsoft Endpoint Manager`

## Apple Enrollment Profile

* Name `<Agency Acronym> iOS enrollment profile`
* Description: -
* User Affinity & Authentication Method
  * User affinity: `Enroll with User Affinity`
  * Select where users must authenticate: `Company Portal`
  * Install Company Portal with VPP: `Use Token`
  * Run Company Portal in Single App Mode until authentication: `No`
* Management Options
  * Supervised: `Yes`
* Locked enrollment: `Yes`
* Sync with computers: `Deny All`
* Device Name
  * Apply device name template (supervised only): `Yes`
  * Device Name Template: `<Agency Acronym>-{SERIAL}-{DEVICETYPE}`
* Setup Assistant
  * Department: `<Agency Name>`
  * Department Phone: `<Agency Support Telephone Number>`
* Setup Assistant Screens
  * Passcode: `Show`
  * Location Services: `Show`
  * Restore: `Hide`
  * Apple ID: `Show`
  * Terms and Conditions: `Hide`
  * Touch ID: `Show`
  * Apple Pay: `Show`
  * Zoom: `Show`
  * Siri: `Hide`
  * Diagnostic Data: `Hide`
  * Restore Completed: `Hide`
  * Software Update Completed: `Hide`
  * Display Tone: `Show`
  * Privacy: `Hide`
  * Android Migration: `Hide`
  * Onboarding:
  * Watch Migration: `Hide`
  * Screen Time: `Hide`
  * Software Update: `Show`
  * SIM Setup: `Show`
  * Appearance: `Show`
  * Device to Device Migration: `Hide`
  * Registration: `Show`
  * FileVault: `Hide`
  * iCloud diagnostics: `Hide`
  * iCloud Storage: `Hide`

### Default Profile

* Set Default Profile:`<Agency Acronym> iOS enrollment profile`

## Enrolment types (preview)

`Microsoft Endpoint Manager > Devices > Enroll devices > Apple enrollment > Enrollment types (preview)`

* Name: `iOS DEP Enrolment`
* Description: -
* Settings
  * Enrollment types: `Device enrollment`
* Assignments
  * Included groups:
```
rol-Agency-Users
Dev Admins
rol-Agency-Administrators
Office365_Grant_AzureAD_Join
```
  * Excluded groups: -



