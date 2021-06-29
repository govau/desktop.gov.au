---
layout: page
title: Hybrid - Microsoft Endpoint Manager - Intune configuration for Windows devices
menu: abac
---

## Profiles

The ABAC settings for the Agency Microsoft Endpoint Manager - Intune (Intune) Profiles can be found below. This includes macro security, Windows Hello, block admins, delivery optimisation, disable Adobe Flash, Microsoft Store, Defender, network boundary, OneDrive, timezone, security baseline fixes, Bitlocker, and Windows 10 Enterprise settings.

Please note, if a setting is not mentioned in the below, it should be assumed to have been left at its default setting.

The following can be found at `Microsoft Endpoint Manager > Devices > Configuration profiles`

### ACSC-EdgeBrowser

* Name: `ACSC-EdgeBrowser`
* Description: -
* Type: `Windows 10 and later`
* Profile Type: `Administrative Templates`
* Scope tags: `Default`
* Assignments
  * Included groups: `rol-Agency-Administrators`, `rol-Agency-Users`
  * Excluded groups: -

Configuration settings

ClassType | CategoryPath | DisplayName | Value | Enabled
--- | --- | --- | --- | --- |
Device | \Microsoft Edge | Allow importing of saved passwords |  | False
Device | \Microsoft Edge | Allow importing of extensions |  | False
Device | \Microsoft Edge | Allow users to proceed from the HTTPS warning page |  | False
Device | \Microsoft Edge | Control where developer tools can be used | Don't allow using the developer tools | True
Device | \Microsoft Edge | Configure Do Not Track |  | True
Device | \Microsoft Edge\Cast | Enable Google Cast |  | False
Device | \Microsoft Edge\Cast | Show the cast icon in the toolbar |  | False
Device | \Microsoft Edge\Content settings | Default pop-up window setting | Do not allow any site to show popups | True
Device | \Microsoft Edge\Password manager and protection | Enable Microsoft Edge to save user passwords |  | False
Device | \Microsoft Edge\SmartScreen settings | Configure Microsoft Defender SmartScreen |  | True
Device | \Microsoft Edge\SmartScreen settings | Configure Microsoft Defender SmartScreen to block potentially unwanted apps |  | True
Device | \Microsoft Edge\SmartScreen settings | Force Microsoft Defender SmartScreen checks on downloads from trusted sources |  | True
Device | \Microsoft Edge\SmartScreen settings | Prevent bypassing Microsoft Defender SmartScreen prompts for sites |  | True
Device | \Microsoft Edge\SmartScreen settings | Prevent bypassing of Microsoft Defender SmartScreen warnings about downloads |  | True

### ACSC-Jan2020-MacroSecurity

* Name: `ACSC-Jan2020-MacroSecurity`
* Description: `Only digitally signed macros are enabled (hardened implementation)`
* Type: `Windows 10 and later`
* Profile Type: `Administrative Templates`
* Scope tags: `Default`
* Assignments
  * Included groups: `rol-Agency-Administrators`, `rol-Agency-Users`
  * Excluded groups: -

Configuration settings

ClassType | CategoryPath | DisplayName | Value | Enabled
--- | --- | --- | --- | --- 
user | \Microsoft Publisher 2016\Disable Items in User Interface\Custom | Disable commands | 19092 | true
user | \Microsoft Excel 2016\Excel Options\Security\Trust Center | VBA Macro Notification Settings | 3 | true
user | \Microsoft Excel 2016\Excel Options\Security\Trust Center | Turn off trusted documents |  | true
user | \Microsoft Excel 2016\Excel Options\Security\Trust Center | Turn off Trusted Documents on the network |  | true
user | \Microsoft Excel 2016\Excel Options\Security\Trust Center | Block macros from running in Office files from the Internet |  | true
user | \Microsoft Excel 2016\Excel Options\Security\Trust Center | Trust access to Visual Basic Project  |  | False
user | \Microsoft Excel 2016\Excel Options\Security\Trust Center\Trusted Locations | Disable all trusted locations |  | true
user | \Microsoft Excel 2016\Excel Options\Security\Trust Center\Trusted Locations | Allow Trusted Locations on the network |  | false
user | \Microsoft Excel 2016\Disable Items in User Interface\Custom | Disable commands | 19092 | true
user | \Microsoft Excel 2016\Excel Options\Security | Scan encrypted macros in Excel Open XML workbooks | 0 | true
user | \Microsoft Access 2016\Application Settings\Security\Trust Center | Turn off Trusted Documents on the network |  | true
user | \Microsoft Access 2016\Application Settings\Security\Trust Center\Trusted Locations | Disable all trusted locations |  | true
user | \Microsoft Access 2016\Application Settings\Security\Trust Center | Turn off trusted documents |  | true
user | \Microsoft Access 2016\Application Settings\Security\Trust Center | VBA Macro Notification Settings | 3 | true
user | \Microsoft Access 2016\Application Settings\Security\Trust Center\Trusted Locations | Allow Trusted Locations on the network |  | false
user | \Microsoft Access 2016\Disable Items in User Interface\Custom | Disable commands | 19092 | true
user | \Microsoft Word 2016\Word Options\Security\Trust Center | Turn off trusted documents |  | true
user | \Microsoft Word 2016\Word Options\Security\Trust Center\Trusted Locations | Disable all trusted locations |  | true
user | \Microsoft Word 2016\Disable Items in User Interface\Custom | Disable commands | 19092 | true
user | \Microsoft Word 2016\Word Options\Security\Trust Center | Block macros from running in Office files from the Internet |  | true
user | \Microsoft Word 2016\Word Options\Security\Trust Center | Trust access to Visual Basic Project |  | false
user | \Microsoft Word 2016\Word Options\Security\Trust Center | Turn off Trusted Documents on the network |  | true
user | \Microsoft Word 2016\Word Options\Security\Trust Center\Trusted Locations | Allow Trusted Locations on the network |  | false
user | \Microsoft Word 2016\Word Options\Security\Trust Center | Scan encrypted macros in Word Open XML documents | 0 | true
user | \Microsoft Word 2016\Word Options\Security\Trust Center | VBA Macro Notification Settings | 3 | true
user | \Microsoft PowerPoint 2016\PowerPoint Options\Security\Trust Center | Turn off trusted documents |  | true
user | \Microsoft PowerPoint 2016\PowerPoint Options\Security\Trust Center\Trusted Locations | Disable all trusted locations |  | true
user | \Microsoft PowerPoint 2016\PowerPoint Options\Security\Trust Center\Trusted Locations | Allow Trusted Locations on the network |  | false
user | \Microsoft PowerPoint 2016\PowerPoint Options\Security | Scan encrypted macros in PowerPoint Open XML presentations | 0 | true
user | \Microsoft PowerPoint 2016\PowerPoint Options\Security\Trust Center | Trust access to Visual Basic Project |  | false
user | \Microsoft PowerPoint 2016\Disable Items in User Interface\Custom | Disable commands | 19092 | true
user | \Microsoft PowerPoint 2016\PowerPoint Options\Security\Trust Center | Turn off Trusted Documents on the network |  | true
user | \Microsoft PowerPoint 2016\PowerPoint Options\Security\Trust Center | VBA Macro Notification Settings | 3 | true
user | \Microsoft PowerPoint 2016\PowerPoint Options\Security\Trust Center | Block macros from running in Office files from the Internet |  | true
user | \Microsoft Project 2016\Project Options\Security\Trust Center | VBA Macro Notification Settings | 3 | true
user | \Microsoft Project 2016\Project Options\Security\Trust Center | Disable all trusted locations |  | true
user | \Microsoft Project 2016\Project Options\Security\Trust Center | Allow Trusted Locations on the network |  | false
user | \Microsoft Visio 2016\Visio Options\Security\Trust Center | Allow Trusted Locations on the network |  | false
user | \Microsoft Visio 2016\Visio Options\Security\Trust Center | Turn off trusted documents |  | true
user | \Microsoft Visio 2016\Visio Options\Security\Trust Center | VBA Macro Notification Settings | 3 | true
user | \Microsoft Visio 2016\Visio Options\Security\Trust Center | Block macros from running in Office files from the Internet |  | true
user | \Microsoft Visio 2016\Disable Items in User Interface\Custom | Disable commands | 19092 | true
user | \Microsoft Visio 2016\Visio Options\Security\Trust Center | Disable all trusted locations |  | true
user | \Microsoft Visio 2016\Visio Options\Security\Macro Security | Load Microsoft Visual Basic for Applications projects from text |  | false
user | \Microsoft Visio 2016\Visio Options\Security\Trust Center | Turn off Trusted Documents on the network |  | true
user | \Microsoft Visio 2016\Visio Options\Security\Macro Security | Enable Microsoft Visual Basic for Applications project creation |  | false
user | \Microsoft Office 2016\Security Settings\Trust Center | Allow mix of policy and user locations |  | false
user | \Microsoft Office 2016\Security Settings | Disable VBA for Office applications |  | false
user | \Microsoft Office 2016\Security Settings | Macro Runtime Scan Scope | 2 | true
user | \Microsoft Office 2016\Security Settings | Disable all Trust Bar notifications for security issues |  | true
user | \Microsoft Office 2016\Security Settings | Automation Security | 2 | true
user | \Microsoft Outlook 2016\Security\Trust Center | Apply macro security settings to macros, add-ins and additional actions |  | true
user | \Microsoft Outlook 2016\Security\Trust Center | Security setting for macros | 3 | true
user | \Microsoft Outlook 2016\Disable Items in User Interface\Custom | Disable command bar buttons and menu items | 19092 | true
user | \Microsoft Publisher 2016\Security\Trust Center | VBA Macro Notification Settings | 3 | true
user | \Microsoft Publisher 2016\Security | Publisher Automation Security Level | 2 | true

### ACSC-WindowsHelloforBusiness

* Name: `ACSC-WindowsHelloforBusiness`
* Description: -
* Type: `Windows 10 and later`
* Profile Type: `Identity protection`
* Configuration settings
  * Configure Windows Hello for Business: `Disable`
  * Use security keys for sign-in: `Not configured`
* Scope tags: `Default`
* Assignments
  * Included groups: `rol-Agency-Administrators`, `rol-Agency-Users`
  * Excluded groups: `grp-iPhone-Devices`, `grp-iOS-Devices`

### Agency-BlockAdminTerminal-User

* Name: `Agency-BlockAdminTerminal-User`
* Description: `This policy blocks users from accessing CMD & PowerShell terminals`
* Type: `Windows 10 and later`
* Profile Type: `Custom`
* Configuration settings
  * OMA-URI Settings
    * Name: `Agency-BlockAdminTerminal-User`
    * Description: `This policy blocks users from accessing CMD & PowerShell terminals`
    * OMA-URI: `./Vendor/MSFT/AppLocker/ApplicationLaunchRestrictions/000000001/EXE/Policy`
    * Custom XML:

```xml
<RuleCollection Type="Exe" EnforcementMode="Enabled">
  <FilePathRule Id="e16ce5e4-67f2-4ebf-ad01-c81fc8f28cd5" Name="All Files" Description="" UserOrGroupSid="S-1-5-32-544" Action="Allow">
    <Conditions>
      <FilePathCondition Path="*"/>
    </Conditions>
  </FilePathRule>
  <FilePathRule Id="9eb15b2e-f9c2-42d4-8692-ad1a0f6a0722" Name="All files" Description="Allows user to run files except powershell" UserOrGroupSid="S-1-1-0" Action="Allow">
    <Conditions>
      <FilePathCondition Path="*"/>
    </Conditions>
    <Exceptions>
      <FilePublisherCondition PublisherName="O=MICROSOFT CORPORATION, L=REDMOND, S=WASHINGTON, C=US" ProductName="MICROSOFTÂ® WINDOWSÂ® OPERATING SYSTEM" BinaryName="POWERSHELL.EXE">
        <BinaryVersionRange LowSection="*" HighSection="*"/>
      </FilePublisherCondition>
      <FilePublisherCondition PublisherName="O=MICROSOFT CORPORATION, L=REDMOND, S=WASHINGTON, C=US" ProductName="MICROSOFTÂ® WINDOWSÂ® OPERATING SYSTEM" BinaryName="POWERSHELL_ISE.EXE">
        <BinaryVersionRange LowSection="*" HighSection="*"/>
      </FilePublisherCondition>
      <FilePublisherCondition PublisherName="O=MICROSOFT CORPORATION, L=REDMOND, S=WASHINGTON, C=US" ProductName="MICROSOFTÂ® WINDOWSÂ® OPERATING SYSTEM" BinaryName="CMD.EXE">
        <BinaryVersionRange LowSection="*" HighSection="*"/>
      </FilePublisherCondition>
      <FilePublisherCondition PublisherName="O=MICROSOFT CORPORATION, L=REDMOND, S=WASHINGTON, C=US" ProductName="MICROSOFTÂ® WINDOWSÂ® OPERATING SYSTEM" BinaryName="REGEDIT.EXE">
        <BinaryVersionRange LowSection="*" HighSection="*"/>
      </FilePublisherCondition>
    </Exceptions>
  </FilePathRule>
</RuleCollection>
```

* Scope tags: `Default`
* Assignments
  * Included groups: `rol-Agency-Users`
  * Excluded groups: -

### Agency-DeliveryOptimisation

* Name: `Agency-DeliveryOptimisation`
* Description: -
* Type: `Windows 10 and later`
* Profile Type: `Delivery Optimization`
* Configuration settings
  * Download mode: `HTTP blended with peering behind same NAT (1)`
  * Bandwidth optimization type: `Percentage`
  * Maximum foreground download bandwidth (in %): `70`
  * Maximum background download bandwidth (in %): `25`
  * Delay background HTTP download bandwidth (in seconds): `60`
  * Delay foreground HTTP download bandwidth (in seconds): `60`
  * Minimum RAM required for peer caching (in GB): `4`
  * Minimum disk size required for peer caching (in GB): `32`
  * Minimum content file size for peer caching (in MB): `5`
  * Minimum battery level required to upload (in %): `40`
  * Maximum cache age (in days): `7`
  * Maximum cache size type: `Percentage`
  * Maximum cache size (in %): `20`
* Scope tags: `Default`
* Assignments
  * Included groups: All devices
  * Excluded groups: -

### Agency-DisableAdobeFlashIE-User

* Name: `Agency-DisableAdobeFlashIE-User`
* Description: `This policy setting turns off Adobe Flash in Internet Explorer and prevents application from using Internet Explorer technology to instantiate Flash object`
* Type: `Windows 10 and later`
* Profile Type: `Administrative Templates`
* Configuration settings
  * Turn off Adobe Flash in Internet Explorer and prevent applications from using Internet Explorer technology to instantiate Flash objects: `Enable`
* Scope tags: `Default`
* Assignments
  * Included groups: `rol-Agency-Administrators`, `rol-Agency-Users`
  * Excluded groups: -

### Agency-MicrosoftStore-User

* Name: `Agency-MicrosoftStore-User`
* Description: `This policy blocks the Windows 10 public store but allows access to the business store`
* Type: `Windows 10 and later`
* Profile Type: `Custom`
* Configuration settings
  * OMA-URI Settings
    * Name: `BlockWindows10Store`
    * Description: `Not configured`
    * OMA-URI: `./Vendor/MSFT/Policy/Config/ApplicationManagement/RequirePrivateStoreOnly`
    * Data Type: `Integer`
    * Integer value: `1` 
* Scope tags: `Default`
* Assignments
  * Included groups: `rol-Agency-Administrators`, `rol-Agency-Users`
  * Excluded groups: -

### Agency-MSDefenderATP

* Name: `Agency-MSDefenderATP`
* Description: -
* Type: `Windows 10 and later`
* Profile Type: `Microsoft Defender ATP (Windows 10 Desktop)`
* Configuration settings
  * Microsoft Defender ATP client configuration package type: `Onboard`
  * Sample sharing for all files: `Enabled`
  * Expedite telemetry reporting frequency: `Enabled`
* Scope tags: `Default`
* Assignments
  * Included groups: `rol-Agency-Administrators`, `rol-Agency-Users`
  * Excluded groups: -

### Agency-Network-Boundary

* Name: `Agency-Network-Boundary`
* Description: -
* Type: `Windows 10 and later`
* Profile Type: `Network Boundary`
* Configuration settings
  * Network Boundary
    * Boundary type: `Cloud resources`
    * Value: `<Agency>.sharepoint.com|<Agency>-my.sharepoint.com|<Agency>-files.sharepoint.com|tasks.office.com|protection.office.com|meet.lync.com|teams.microsoft.com|www.yammer.com|yammer.com|persona.yammer.com|outlook.office.com|outlook.office365.com|attachments.office.net|<Agency>.crm.dynamics.com|<Agency>.visualstudio.com|<Agency>.powerbi.com`
* Scope tags: `Default`
* Assignments
  * Included groups: `rol-Agency-Administrators`, `rol-Agency-Users`
  * Excluded groups: -

### Agency-OneDrive-Device

* Name: `Agency-OneDrive-Device`
* Description: -
* Type: `Windows 10 and later`
* Profile Type: `Administrative Templates`
* Scope tags: `Default`
* Assignments
  * Included groups: `rol-Agency-Administrators`, `rol-Agency-Users`
  * Excluded groups: -

Configuration settings

ClassType | CategoryPath | DisplayName | Value | Enabled
--- | --- | --- | --- | --- 
machine | All Settings\OneDrive | Silently sign in users to the OneDrive sync client with their Windows credentials |  | true
machine | All Settings\OneDrive | Silently move Windows known folders to OneDrive | xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx | true
machine | All Settings\OneDrive | Prevent users from syncing libraries and folders shared from other organizations |  | true
machine | All Settings\OneDrive | Require users to confirm large delete operations |  | true
machine | All Settings\OneDrive | Set the sync client update ring | Enterprise | true
machine | All Settings\OneDrive | Prevent users from redirecting their Windows known folders to their PC |  | true
machine | All Settings\OneDrive | Use OneDrive Files On-Demand | xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx | true

### Agency-OneDrive-User

* Name: `Agency-OneDrive-User`
* Description: -
* Type: `Windows 10 and later`
* Profile Type: `Administrative Templates`
* Scope tags: `Default`
* Assignments
  * Included groups: `rol-Agency-Administrators`, `rol-Agency-Users`
  * Excluded groups: -

Configuration settings

ClassType | CategoryPath | DisplayName | Value | Enabled
--- | --- | --- | --- | --- 
user | \OneDrive | Coauthor and share in Office desktop apps |  | true
user | \OneDrive | Prevent users from syncing personal OneDrive accounts |  | true

### Agency-TimeZoneEST

* Name: `Agency-TimeZoneEST`
* Description: `AUS Eastern Standard Time`
* Type: `Windows 10 and later`
* Profile Type: `Custom`
* Configuration settings
  * OMA-URI Settings
    * Name: `Set TimeZone`
    * Description: `Set TimeZone`
    * OMA-URI: `./Device/Vendor/MSFT/Policy/Config/TimeLanguageSettings/ConfigureTimeZone`
    * Data type: `String`
    * Value: `AUS Eastern Standard Time`
* Scope tags: `Default`
* Assignments
  * Included groups: `grp-Windows-10-Devices`, `rol-Agency-Administrators`, `rol-Agency-Users`
  * Excluded groups: -

### SecBaselineFix-AT

* Name: `SecBaselineFix-AT`
* Description: -
* Type: `Windows 10 and later`
* Profile Type: `Administrative Templates`
* Scope tags: `Default`
* Assignments
  * Included groups: `rol-Agency-Administrators`, `rol-Agency-Users`
  * Excluded groups: -

Configuration settings

ClassType | CategoryPath | DisplayName | Value | Enabled
--- | --- | --- | --- | --- 
computer | \Windows Components\Internet Explorer\Security Features\Add-On Management | Turn off Adobe Flash in Internet Explorer and prevent applications from using Internet Explorer technology to instantiate Flash object |  | true

### SecBaselineFix-EP

* Name: `SecBaselineFix-EP`
* Description: -
* Type: `Windows 10 and later`
* Profile Type: `Endpoint protection`
* Configuration settings
  * Microsoft Defender Exploit Guard
    * Flag credential stealing from the Windows local security authority subsystem: `Enable`
    * Process creation from Adobe Reader (beta): `Enable`
    * Office apps injecting into other processes (no exceptions): `Block`
    * Office apps/macros creating executable content: `Block`
    * Office apps launching child processes : `Block`
    * Win32 imports from Office macro code: `Block`
    * Process creation from Office communication products (beta): `Enable`
    * Obfuscated js/vbs/ps/macro code: `Block`
    * js/vbs executing payload downloaded from Internet (no exceptions): `Block`
    * Untrusted and unsigned processes that run from USB: `Block`
    * Execution of executable content (exe, dll, ps, js, vbs, etc.) dropped from email (webmail/mail client) (no exceptions): `Block`
    * Folder protection : `Audit only`
    * Network protection: `Enable`
* Scope tags: `Default`
* Assignments
  * Included groups: `rol-Agency-Administrators`, `rol-Agency-Users`
  * Excluded groups: `grp-NCSC-Devices`, `grp-ACSC-Devices`

### Windows 10 BitLocker

* Name: `Windows 10 BitLocker`
* Description: -
* Type: `Windows 10 and later`
* Profile Type: `Endpoint protection`
* Configuration settings
  * Windows Encryption
    * Encrypt devices: `Require`
    * Warning for other disk encryption: `Block`
    * Allow standard users to enable encryption during Azure AD Join: `Allow`
    * Configure encryption methods: `Enable`
    * Encryption for operation system drives: `XTS-AES 256-bit`
    * Encryption for fixed data-drives: `XTS-AES 256-bit`
    * Encryption for removable data-drives: `XTS-AES 256-bit`
    * Additional authentication at startup: `Require`
    * OS drive recovery: `Enable`
    * Recovery options in the BitLocker setup wizard: `Block`
    * Save BitLocker recovery information to Azure Active Directory: `Enable`
    * Client-driven recovery password rotation: `Key rotation enabled for Azure AD-joined devices`
    * Store recovery information in Azure Active Directory before enabling BitLocker: `Require`
    * Write access to fixed data-drive not protected by BitLocker: `Block`
    * Fixed drive recovery: `Enable`
    * Write access to removable data-drive not protected by BitLocker: `Block`
* Scope tags: `Default`
* Assignments
  * Included groups: `grp-Security-Baseilnes`, `rol-Agency-Administrators`, `rol-Agency-Users`
  * Excluded groups: -

### Windows 10 Enterprise activation

* Name: `Windows 10 Enterprise Activation`
* Description: -
* Type: `Windows 10 and later`
* Profile Type: `Edition upgrade or mode switch`
* Configuration settings
  * Edition to upgrade to: `Windows 10 Enterprise`
  * Product Key: `#####-#####-#####-#####-#####`
* Scope tags: `Default`
* Assignments
  * Included groups: `rol-Agency-Administrators`, `rol-Agency-Users`
  * Excluded groups: -
* Applicability Rules
  * Rule: `Assign profile if`
  * Property: `OS edition`
  * Value: `Windows 10 Professional`

### Windows Defender ATP offboarding

* Name: `Windows Defender ATP offboarding`
* Description: -
* Type: `Windows 10 and later`
* Profile Type: `Custom`
* Configuration settings
  * OMA-URI Settings
    * Name: `Defender ATP offboarding`
    * Description: `Not configured`
    * OMA-URI: `./Device/Vendor/MSFT/WindowsAdvancedThreatProtection/Offboarding`
    * Data Type: `String`
    * Value: `{"body":"{\"orgIds\":[\"ec4ecef5-068d-4d33-8baf-93bca8a6bdc8\"],\"orgId\":\"ec4ecef5-068d-4d33-8baf-93bca8a6bdc8\",\"expirationTimestamp\":132255273658317066,\"version\":\"1.24\"}","sig":"lA+/Tjbd3aBLuue1eugXQ0CuKTX4zB2xyK22XNgTo9GGQnEY1NNHPKYCthtUWZs/hhYJOaOTfVaM4qf+tgsWeakiMO3D1+SXQJWx4HIAQFsDyXqq0H4mgXVEll4ggwdWw+cM+1gPqmB8vL9rNl+AOIu+w/kqoIwnODINXV8+G+1Ako3zgmvkAN+OeslY0QIonjMh6u9kaHY+pV/RFJBierBOEJSEJTOmj4MufT21r5fkPITwarMuXXZDzOPna16WehawzvoomRVUGcJ+Qs89h0gTOGAleWCnEjInMGWNNymdFxKVkyAy0yv8pYpiOJgfgZCJyj0Szd4MJeCGEscS0w==","sha256sig":"BdDJy+l+wVnSwEoyYiThOEnSCGY8smqXQ3NGeXTfwjWD/gP41+QE0MefYba8LW+pI+2IOCJGmxlo2qf59bPTWP8kX5vieXARhT3pCuIKsBb0/JnaZ1Y+7wvTTM/xNIV/he4DkDfVinIhIUF1AkmCGZCqFyF36MO/l7GUZsDp3Ru4R1o6ObVFQCGkJ6DQUtfmp5MZS5Ee8TNiHOvB64O62O93CKmdJZkc/1D77TSuc4VYQzj5MN7zgY4/kl65lD1koMTHZG67dJvGzafAyUrEA5eLO61TrG8dDPVMjvc887OhLu4hnAKtr0hs2sU4tChzmb1l9/6bXUzY007iYdHLaA==","cert":"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","chain":["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"]}`
* Scope tags: `Default`
* Assignments
  * Included groups: `grp-RemoveFromDefenderATP`
  * Excluded groups: -

## Scripts

The following can be found at `Intune > Devices > Scripts`

### Intune log folder shortcut

* Profile name: `Intune log folder shortcut`
* Script settings
  * PowerShell script: [IntuneLogFolder.ps1](/assets/files/abac/IntuneLogFolder.txt)
  * Run this script using the logged-on credentials: `No`
  * Enforce script signature check: `No`
  * Run script in 64 bit PowerShell Host: `No`
* Scope tags: `Default`
* Assignments
  * Included groups: `rol-Agency-Administrators`, `rol-Agency-Users`
  * Excluded groups: -

### Block OLE Extensions

* Profile name: `Intune Block OLE Extension`
* Script settings
  * PowerShell script: [oleblockingscript.ps1](/assets/files/abac/oleblockingscript.txt)
  * Run this script using the logged-on credentials: `Yes`
  * Enforce script signature check: `No`
  * Run script in 64 bit PowerShell Host: `No`
* Scope tags: `Default`
* Assignments
  * Included groups: `rol-Agency-Administrators`, `rol-Agency-Users`
  * Excluded groups: -

### Remove built-in apps

* Profile name: `RemoveBuiltInApps`
* Script settings
  * PowerShell script: [RemoveBuiltInApps.ps1](/assets/files/abac/RemoveBuiltInApps.txt)
  * Run this script using the logged-on credentials: `No`
  * Enforce script signature check: `No`
  * Run script in 64 bit PowerShell Host: `No`
* Scope tags: `Default`
* Assignments
  * Included groups: `grp-Windows-10-Devices`
  * Excluded groups: -
