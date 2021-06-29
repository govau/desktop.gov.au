---
layout: page
title: Microsoft Endpoint Manager - Intune security baselines
menu: abac
---

## ACSC1709Guidance-DefenderATPSecurityBaseline_v3

`Microsoft Endpoint Manager > Endpoint security > Security baselines > Microsoft Defender ATP Baseline`

* Name: `ACSC1709Guidance-DefenderATPSecurityBaseline_v3`
* Description: `Version 3 baseline`
* Platform: `Windows 10 and later`
* Security Baselines: `Microsoft Defender ATP Baselines`
* Associated Profiles: `1`
* Configuration settings
  * Application Guard
    * Turn on Application Guard for Edge (Options): `Not Configured`
    * Windows network isolation policy: `Not Configured`
  * BitLocker
    * Require storage cards to be encrypted (mobile only): `Yes`
    * Enable full disk encryption for OS and fixed data drives: `Not Configured`
    * BitLocker system drive policy: `Configure`
    * Configure encryption method for Operating System drives: `AES 256bit XTS`
    * BitLocker fixed drive policy: `Configure`
    * Block write access to fixed data-drives not protected by BitLocker: `Yes`
    * Configure encryption method for fixed data-drives: `AES 256bit XTS`
    * BitLocker removable drive policy: `Configure`
    * Configure encryption method for removable data-drives: `AES 256bit XTS`
    * Block write access to removable data-drives not protected by BitLocker: `Yes`
  * Browser
    * Require SmartScreen for Microsoft Edge: `Yes`
    * Block malicious site access: `Yes`
    * Block unverified file download: `Yes`
  * Data Protection
    * Block direct memory access: `Yes`
  * Device Guard
    * Turn on credential guard: `Enable with UEFI lock`
  * Device Installation
    * Hardware device installation by device identifiers: `Block hardware device installation`
    * Remove matching hardware devices: `Yes`
    * Hardware device identifiers that are blocked: `PCI\CC_0C0A`
    * Hardware device installation by setup classes: `Block hardware device installation`
    * Remove matching hardware devices: `Yes`
    * Hardware device identifiers that are blocked: `{d48179be-ec20-11d1-b6b8-00c04fa372a7}`
  * DMA Guard
    * Enumeration of external devices incompatible with Kernel DMA Protection: `Not Configured`
  * Endpoint Detection and Response
    * Sample sharing for all files: `Yes`
    * Expedite telemetry reporting frequency: `Yes`
  * Firewall
    * Disable stateful File Transfer Protocol (FTP): `Yes`
    * Number of seconds a security association can be idle before it's deleted: `300`
    * Preshared key encoding: `UTF8`
    * Certificate revocation list (CRL) verification: `Not Configured`
    * Packet queuing: `Not Configured`
    * Firewall profile private: `Configure`
    * Inbound connections blocked: `Yes`
    * Unicast responses to multicast broadcasts required: `Yes`
    * Stealth mode required: `Yes`
    * Outbound connections required: `Not Configured`
    * Inbound notifications blocked: `Yes`
    * Global port rules from group policy merged: `Yes`
    * Stealth mode blocked: `Yes`
    * Firewall enabled: `Allowed`
    * Authorized application rules from group policy not merged: `Yes`
    * Connection security rules from group policy not merged: `Yes`
    * Incoming traffic required: `Yes`
    * Policy rules from group policy not merged: `Yes`
    * Firewall profile public: `Configure`
    * Inbound connections blocked: `Yes`
    * Unicast responses to multicast broadcasts required: `Yes`
    * Stealth mode required: `Yes`
    * Outbound connections required: `Not Configured`
    * Authorized application rules from group policy merged: `Yes`
    * Inbound notifications blocked: `Yes`
    * Global port rules from group policy merged: `Yes`
    * Stealth mode blocked: `Yes`
    * Firewall enabled: `Allowed`
    * Connection security rules from group policy not merged: `Yes`
    * Incoming traffic required: `Yes`
    * Policy rules from group policy not merged: `Yes`
    * Firewall profile domain: `Configure`
    * Unicast responses to multicast broadcasts required: `Yes`
    * Authorized application rules from group policy merged: `Yes`
    * Inbound notifications blocked: `Yes`
    * Global port rules from group policy merged: `Yes`
    * Stealth mode blocked: `Yes`
    * Firewall enabled: `Allowed`
    * Connection security rules from group policy not merged: `Yes`
    * Policy rules from group policy not merged: `Yes`
  * Microsoft Defender
    * Run daily quick scan at: `2 AM`
    * Scheduled scan start time: `2 AM`
    * Configure low CPU priority for scheduled scans: `Yes`
    * Block Office communication apps from creating child processes: `Enable`
    * Block Adobe Reader from creating child processes: `Enable`
    * Scan incoming email messages: `Yes`
    * Turn on real-time protection: `Yes`
    * Number of days (0-90) to keep quarantined malware: `0`
    * Defender system scan schedule: `User defined`
    * Additional amount of time (0-50 seonds) to extend cloud protection timeout: `0`
    * Scan mapped network drives during a full scan: `Yes`
    * Turn on network protection: `Yes`
    * Scan all downloaded files and attachments: `Yes`
    * Block on access protection: `Yes`
    * Scan browser scripts: `Yes`
    * Block user access to Microsoft Defender app: `Yes`
    * Maximum allowed CPU usage (0-100 percent) per scan: `50`
    * Scan type: `Quick scan`
    * Enter how often (0-24 hours) to check for security intelligence updates: `4`
    * Defender sample submission consent: `Send safe samples automatically`
    * Cloud-delivered protection level: `Not Configured`
    * Scan archive files: `Yes`
    * Turn on behavior monitoring: `Yes`
    * Scan removable drives during full scan: `Yes`
    * Scan network files: `Yes`
    * Defender potentially unwanted app action: `Block`
    * Turn on cloud-delivered protection: `Yes`
    * Block Office applications from injecting code into other processes: `Block`
    * Block Office applications from creating executable content: `Block`
    * Block JavaScript or VBScript from launching downloaded executable content: `Block`
    * Enable network protection: `Enable`
    * Block untrusted and unsigned processes that run from USB: `Block`
    * Block credential stealing from the Windows local security authority subsystem (lsass.exe): `Enable`
    * Block executable content download from email and webmail clients: `Block`
    * Block all Office applications from creating child processes: `Block`
    * Block execution of potentially obfuscated scripts (js/vbs/ps): `Block`
    * Block Win32 API calls from Office macro: `Block`
  * Microsoft Defender Security Center
    * Block users from editing the Exploit Guard protection interface: `Yes`
  * Smart Screen
    * Block users from ignoring SmartScreen warnings: `Yes`
    * Require apps from store only: `Yes`
    * Turn on Windows SmartScreen: `Yes`
  * Windows Hello for Business
    * Block Windows Hello for Business: `Not Configured`

## ACSC1709Guidance-EdgeSecurityBaseline_Settings

`Microsoft Endpoint Manager > Endpoint security > Security baselines > Microsoft Edge Baseline`

* Name: `ACSC1709Guidance-EdgeSecurityBaseline`
* Description: `ACSC1709Guidance-EdgeSecurityBaseline`
* Platform: `Windows 10 and later`
* Security Baselines: `Microsoft Edge Baseline`
* Associated Profiles: `1`
* Configuration settings
  * Microsoft Edge
    * Supported authentication schemes: `Enabled`
    * Supported authentication schemes: `NTLM`, `Negotiate`
    * Default Adobe Flash setting: `Enabled`
    * Default Adobe Flash setting: `Block the Adobe Flash Plugin`
    * Control which extensions cannot be installed: `Enabled`
    * Extension IDs the user should be prevented from installing (or * for all): `*`
    * Allow user-level native messaging hosts (installed without admin permissions): `Disabled`
    * Enable saving passwords to the password manager: `Disabled`
    * Prevent bypassing Microsoft Defender SmartScreen prompts for sites: `Enabled`
    * Prevent bypassing of Microsoft Defender SmartScreen warnings about downloads: `Enabled`
    * Enable site isolation for every site: `Enabled`
    * Configure Microsoft Defender SmartScreen: `Enabled`
    * Allow users to proceed from the SSL warning page: `Disabled`
    * Minimum SSL version enabled: `Enabled`
    * Minimum SSL version enabled: `TLS 1.2`
* Scope tags: `Default`
* Assignments
  * Included groups: `rol-Agency-Administrators`, `rol-Agency-Users`
  * Excluded groups: -

## ACSC1709Guidance-MDMSecurity

`Microsoft Endpoint Manager > Endpoint security > Security baselines > Windows 10 Security Baseline`

* Name: `ACSC1709Guidance-MDMSecurity`
* Description: `ACSC1709Guidance-MDMSecurity`
* Platform: `Windows 10 and later`
* Configuration settings
  * Above Lock
    * Voice activate apps from locked screen: `Disabled`
    * Block display of toast notifications: `Yes`
  * App Runtime
    * Microsoft accounts optional for Windows Store apps: `Enabled`
  * Application management
    * Block app installations with elevated privileges: `Yes`
    * Block user control over installations: `Yes`
    * Block game DVR (desktop only): `Yes`
  * Auto Play
    * Auto play default auto run behavior: `Do not execute`
    * Auto play mode: `Disabled`
    * Block auto play for non-volume devices: `Enabled`
  * BitLocker
    * BitLocker removable drive policy: `Configure`
    * Block write access to removable data-drives not protected by BitLocker: `Yes`
  * Browser
    * Block Password Manager: `Yes`
    * Require SmartScreen for Microsoft Edge: `Yes`
    * Block malicious site access: `Yes`
    * Block unverified file download: `Yes`
    * Prevent user from overriding certificate errors: `Yes`
  * Connectivity
    * Configure secure access to UNC paths: `Configure Windows to only allow access to the specified UNC paths after fulfilling additional security requirements`
    * Hardened UNC path list:
      * Security flags 1: `requireMutualAuthentication,requireIntegrity`
      * Server path 1: `\\*\SYSVOL`
      * Security flags 2: `requireMutualAuthentication,requireIntegrity`
      * Server path 2: `\\*\NETLOGON`
    * Block downloading of print drivers over HTTP: `Enabled`
    * Block Internet download for web publishing and online ordering wizards: `Enabled`
  * Credentials Delegation
    * Remote host delegation of non-exportable credentials: `Enabled`
  * Credentials UI
    * Enumerate administrators: `Disabled`
  * Data Protection
    * Block direct memory access: `Yes`
  * Device Guard
    * Virtualization based security: `Enable VBS with secure boot`
    * Enable virtualization based security: `Yes`
    * Launch system guard: `: `Enabled`
    * Turn on credential guard: `Enable with UEFI lock`
  * Device Installation
    * Hardware device installation by device identifiers: `Block hardware device installation`
    * Remove matching hardware devices: `Yes`
    * Hardware device identifiers that are blocked: `PCI\CC_0C0A`
    * Hardware device installation by setup classes: `Block hardware device installation`
    * Remove matching hardware devices: `Yes`
    * Hardware device identifiers that are blocked: `{d48179be-ec20-11d1-b6b8-00c04fa372a7}`
  * Device Lock
    * Require password: `Yes`
    * Required password: `Alphanumeric`
    * Password expiration (days): `60`
    * Password minimum character set count: `3`
    * Prevent reuse of previous passwords: `24`
    * Minimum password length: `8`
    * Number of sign-in failures before wiping device: `10`
    * Block simple passwords: `Yes`
    * Password minimum age in days: `1`
    * Prevent use of camera: `Enabled`
    * Prevent slide show: `Enabled`
  * DMA Guard
    * Enumeration of external devices incompatible with Kernel DMA Protection: `Block all`
  * Event Log Service
    * Application log maximum file size in KB: `65536`
    * System log maximum file size in KB: `65536`
    * Security log maximum file size in KB: `196608`
  * Experience
    * Block Windows Spotlight: `Yes`
  * Exploit Guard: `Upload XML`
  
```xml
<?xml version="1.0" encoding="UTF-8"?>
<MitigationPolicy>
  <AppConfig Executable="ONEDRIVE.EXE">
    <DEP Enable="true" EmulateAtlThunks="false"/>
    <ASLR Enable="true" ForceRelocateImages="true"/>
    <ImageLoad BlockRemoteImageLoads="true"/>
    <Payload EnableExportAddressFilter="true" EnableExportAddressFilterPlus="true" EnableImportAddressFilter="true" EnableRopStackPivot="true" EnableRopCallerCheck="true" EnableRopSimExec="true"/>
  </AppConfig>
  <AppConfig Executable="firefox.exe">
    <DEP Enable="true" EmulateAtlThunks="false"/>
    <ASLR ForceRelocateImages="true" RequireInfo="false" BottomUp="true" HighEntropy="false"/>
  </AppConfig>
  <AppConfig Executable="fltldr.exe">
    <DEP Enable="true" EmulateAtlThunks="false"/>
    <ImageLoad BlockRemoteImageLoads="true"/>
    <ChildProcess DisallowChildProcessCreation="true"/>
    <Payload EnableExportAddressFilter="true" EnableExportAddressFilterPlus="true" EnableImportAddressFilter="true" EnableRopStackPivot="true" EnableRopCallerCheck="true" EnableRopSimExec="true"/>
  </AppConfig>
  <AppConfig Executable="GROOVE.EXE">
    <DEP Enable="true" EmulateAtlThunks="false"/>
    <ASLR Enable="true" ForceRelocateImages="true"/>
    <ImageLoad BlockRemoteImageLoads="true"/>
    <Payload EnableExportAddressFilter="true" EnableExportAddressFilterPlus="true" EnableImportAddressFilter="true" EnableRopStackPivot="true" EnableRopCallerCheck="true" EnableRopSimExec="true"/>
    <ChildProcess DisallowChildProcessCreation="false"/>
  </AppConfig>
  <AppConfig Executable="Acrobat.exe">
    <DEP Enable="true" EmulateAtlThunks="false"/>
    <ASLR ForceRelocateImages="true" RequireInfo="false" BottomUp="true" HighEntropy="false"/>
    <Payload EnableExportAddressFilter="true" EnableExportAddressFilterPlus="true" EnableImportAddressFilter="true" EnableRopStackPivot="true" EnableRopCallerCheck="true" EnableRopSimExec="true"/>
  </AppConfig>
  <AppConfig Executable="AcroRd32.exe">
    <DEP Enable="true" EmulateAtlThunks="false"/>
    <ASLR ForceRelocateImages="true" RequireInfo="false" BottomUp="true" HighEntropy="false"/>
    <Payload EnableExportAddressFilter="true" EnableExportAddressFilterPlus="true" EnableImportAddressFilter="true" EnableRopStackPivot="true" EnableRopCallerCheck="true" EnableRopSimExec="true"/>
  </AppConfig>
  <AppConfig Executable="chrome.exe">
    <DEP Enable="true" EmulateAtlThunks="false"/>
  </AppConfig>
  <AppConfig Executable="EXCEL.EXE">
    <DEP Enable="true" EmulateAtlThunks="false"/>
    <ASLR Enable="true" ForceRelocateImages="true"/>
    <Payload EnableExportAddressFilter="true" EnableExportAddressFilterPlus="true" EnableImportAddressFilter="true" EnableRopStackPivot="true" EnableRopCallerCheck="true" EnableRopSimExec="true"/>
  </AppConfig>
  <AppConfig Executable="iexplore.exe">
    <DEP Enable="true" EmulateAtlThunks="false"/>
    <ASLR ForceRelocateImages="true" RequireInfo="false" BottomUp="true" HighEntropy="false"/>
    <Payload EnableExportAddressFilter="true" EnableExportAddressFilterPlus="true" EnableImportAddressFilter="true" EnableRopStackPivot="true" EnableRopCallerCheck="true" EnableRopSimExec="true"/>
  </AppConfig>
  <AppConfig Executable="INFOPATH.EXE">
    <DEP Enable="true" EmulateAtlThunks="false"/>
    <ASLR Enable="true" ForceRelocateImages="true"/>
    <Payload EnableExportAddressFilter="true" EnableExportAddressFilterPlus="true" EnableImportAddressFilter="true" EnableRopStackPivot="true" EnableRopCallerCheck="true" EnableRopSimExec="true"/>
  </AppConfig>
  <AppConfig Executable="java.exe">
    <DEP Enable="true" EmulateAtlThunks="false"/>
    <Payload EnableExportAddressFilter="true" EnableExportAddressFilterPlus="true" EnableImportAddressFilter="true" EnableRopStackPivot="true" EnableRopCallerCheck="true" EnableRopSimExec="true"/>
  </AppConfig>
  <AppConfig Executable="javaw.exe">
    <DEP Enable="true" EmulateAtlThunks="false"/>
    <Payload EnableExportAddressFilter="true" EnableExportAddressFilterPlus="true" EnableImportAddressFilter="true" EnableRopStackPivot="true" EnableRopCallerCheck="true" EnableRopSimExec="true"/>
  </AppConfig>
  <AppConfig Executable="javaws.exe">
    <DEP Enable="true" EmulateAtlThunks="false"/>
    <Payload EnableExportAddressFilter="true" EnableExportAddressFilterPlus="true" EnableImportAddressFilter="true" EnableRopStackPivot="true" EnableRopCallerCheck="true" EnableRopSimExec="true"/>
  </AppConfig>
  <AppConfig Executable="LYNC.EXE">
    <DEP Enable="true" EmulateAtlThunks="false"/>
    <ASLR Enable="true" ForceRelocateImages="true"/>
    <Payload EnableExportAddressFilter="true" EnableExportAddressFilterPlus="true" EnableImportAddressFilter="true" EnableRopStackPivot="true" EnableRopCallerCheck="true" EnableRopSimExec="true"/>
  </AppConfig>
  <AppConfig Executable="MSACCESS.EXE">
    <DEP Enable="true" EmulateAtlThunks="false"/>
    <ASLR Enable="true" ForceRelocateImages="true"/>
    <Payload EnableExportAddressFilter="true" EnableExportAddressFilterPlus="true" EnableImportAddressFilter="true" EnableRopStackPivot="true" EnableRopCallerCheck="true" EnableRopSimExec="true"/>
  </AppConfig>
  <AppConfig Executable="MSPUB.EXE">
    <DEP Enable="true" EmulateAtlThunks="false"/>
    <ASLR Enable="true" ForceRelocateImages="true"/>
    <Payload EnableExportAddressFilter="true" EnableExportAddressFilterPlus="true" EnableImportAddressFilter="true" EnableRopStackPivot="true" EnableRopCallerCheck="true" EnableRopSimExec="true"/>
  </AppConfig>
  <AppConfig Executable="OIS.EXE">
    <DEP Enable="true" EmulateAtlThunks="false"/>
    <Payload EnableExportAddressFilter="true" EnableExportAddressFilterPlus="true" EnableImportAddressFilter="true" EnableRopStackPivot="true" EnableRopCallerCheck="true" EnableRopSimExec="true"/>
  </AppConfig>
  <AppConfig Executable="OUTLOOK.EXE">
    <DEP Enable="true" EmulateAtlThunks="false"/>
    <ASLR Enable="true" ForceRelocateImages="true"/>
    <Payload EnableExportAddressFilter="true" EnableExportAddressFilterPlus="true" EnableImportAddressFilter="true" EnableRopStackPivot="true" EnableRopCallerCheck="true" EnableRopSimExec="true"/>
  </AppConfig>
  <AppConfig Executable="plugin-container.exe">
    <DEP Enable="true" EmulateAtlThunks="false"/>
    <Payload EnableExportAddressFilter="true" EnableExportAddressFilterPlus="true" EnableImportAddressFilter="true" EnableRopStackPivot="true" EnableRopCallerCheck="true" EnableRopSimExec="true"/>
  </AppConfig>
  <AppConfig Executable="POWERPNT.EXE">
    <DEP Enable="true" EmulateAtlThunks="false"/>
    <ASLR Enable="true" ForceRelocateImages="true"/>
    <Payload EnableExportAddressFilter="true" EnableExportAddressFilterPlus="true" EnableImportAddressFilter="true" EnableRopStackPivot="true" EnableRopCallerCheck="true" EnableRopSimExec="true"/>
  </AppConfig>
  <AppConfig Executable="PPTVIEW.EXE">
    <DEP Enable="true" EmulateAtlThunks="false"/>
    <ASLR Enable="true" ForceRelocateImages="true"/>
    <Payload EnableExportAddressFilter="true" EnableExportAddressFilterPlus="true" EnableImportAddressFilter="true" EnableRopStackPivot="true" EnableRopCallerCheck="true" EnableRopSimExec="true"/>
  </AppConfig>
  <AppConfig Executable="VISIO.EXE">
    <DEP Enable="true" EmulateAtlThunks="false"/>
    <ASLR Enable="true" ForceRelocateImages="true"/>
    <Payload EnableExportAddressFilter="true" EnableExportAddressFilterPlus="true" EnableImportAddressFilter="true" EnableRopStackPivot="true" EnableRopCallerCheck="true" EnableRopSimExec="true"/>
  </AppConfig>
  <AppConfig Executable="VPREVIEW.EXE">
    <DEP Enable="true" EmulateAtlThunks="false"/>
    <ASLR Enable="true" ForceRelocateImages="true"/>
    <Payload EnableExportAddressFilter="true" EnableExportAddressFilterPlus="true" EnableImportAddressFilter="true" EnableRopStackPivot="true" EnableRopCallerCheck="true" EnableRopSimExec="true"/>
  </AppConfig>
  <AppConfig Executable="WINWORD.EXE">
    <DEP Enable="true" EmulateAtlThunks="false"/>
    <ASLR Enable="true" ForceRelocateImages="true"/>
    <Payload EnableExportAddressFilter="true" EnableExportAddressFilterPlus="true" EnableImportAddressFilter="true" EnableRopStackPivot="true" EnableRopCallerCheck="true" EnableRopSimExec="true"/>
  </AppConfig>
  <AppConfig Executable="wmplayer.exe">
    <DEP Enable="true" EmulateAtlThunks="false"/>
    <Payload EnableExportAddressFilter="false" EnableExportAddressFilterPlus="false" EnableImportAddressFilter="false" EnableRopStackPivot="true" EnableRopCallerCheck="true" EnableRopSimExec="true"/>
  </AppConfig>
  <AppConfig Executable="wordpad.exe">
    <DEP Enable="true" EmulateAtlThunks="false"/>
    <Payload EnableExportAddressFilter="true" EnableExportAddressFilterPlus="true" EnableImportAddressFilter="true" EnableRopStackPivot="true" EnableRopCallerCheck="true" EnableRopSimExec="true"/>
  </AppConfig>
</MitigationPolicy>
```

  * File Explorer
    * Block data execution prevention: `Disabled`
    * Block heap termination on corruption: `Disabled`
  * Firewall
    * Firewall profile domain: `Configure`
    * Inbound connections blocked: `Yes`
    * Outbound connections required: `Not Configured`
    * Inbound notifications blocked: `Yes`
    * Firewall enabled: `Allowed`
    * Firewall profile private: `Configure`
    * Inbound connections blocked: `Yes`
    * Outbound connections required: `Not Configured`
    * Inbound notifications blocked: `Yes`
    * Firewall enabled: `Allowed`
    * Firewall profile public: `Configure`
    * Inbound connections blocked: `Yes`
    * Outbound connections required: `Not Configured`
    * Inbound notifications blocked: `Yes`
    * Firewall enabled: `Allowed`
    * Connection security rules from group policy not merged: `Yes`
    * Policy rules from group policy not merged: `Yes`
  * Internet Explorer
    * Internet Explorer encryption support: `TLS v1.1`, `TLS v1.2`
    * Internet Explorer prevent managing smart screen filter: `Enable`
    * Internet Explorer restricted zone script Active X controls marked safe for scripting: `Disable`
    * Internet Explorer restricted zone file downloads: `Disable`
    * Internet Explorer certificate address mismatch warning: `Enabled`
    * Internet Explorer enhanced protected mode: `Enabled`
    * Internet Explorer fallback to SSL3: `No sites`
    * Internet Explorer software when signature is invalid: `Disabled`
    * Internet Explorer check server certificate revocation: `Enabled`
    * Internet Explorer check signatures on downloaded programs: `Enabled`
    * Internet Explorer processes consistent MIME handling: `Enabled`
    * Internet Explorer bypass smart screen warnings: `Disabled`
    * Internet Explorer bypass smart screen warnings about uncommon files: `Disabled`
    * Internet Explorer crash detection: `Disabled`
    * Internet Explorer download enclosures: `Disabled`
    * Internet Explorer ignore certificate errors: `Disabled`
    * Internet Explorer disable processes in enhanced protected mode: `Enabled`
    * Internet Explorer security settings check: `Enabled`
    * Internet Explorer Active X controls in protected mode: `Disabled`
    * Internet Explorer users adding sites: `Disabled`
    * Internet Explorer users changing policies: `Disabled`
    * Internet Explorer block outdated Active X controls: `Enabled`
    * Internet Explorer include all network paths: `Disabled`
    * Internet Explorer internet zone access to data sources: `Disable`
    * Internet Explorer internet zone automatic prompt for file downloads: `Disabled`
    * Internet Explorer internet zone copy and paste via script: `Disable`
    * Internet Explorer internet zone drag and drop or copy and paste files: `Disable`
    * Internet Explorer internet zone less privileged sites: `Disable`
    * Internet Explorer internet zone loading of XAML files: `Disable`
    * Internet Explorer internet zone .NET Framework reliant components: `Disable`
    * Internet Explorer internet zone allow only approved domains to use ActiveX controls: `Enabled`
    * Internet Explorer internet zone allow only approved domains to use tdc ActiveX controls: `Enabled`
    * Internet Explorer internet zone scripting of web browser controls: `Disabled`
    * Internet Explorer internet zone script initiated windows: `Disabled`
    * Internet Explorer internet zone scriptlets: `Disable`
    * Internet Explorer internet zone smart screen: `Enabled`
    * Internet Explorer internet zone updates to status bar via script: `Disabled`
    * Internet Explorer internet zone user data persistence: `Disabled`
    * Internet Explorer internet zone allow VBscript to run: `Disable`
    * Internet Explorer internet zone do not run antimalware against ActiveX controls: `Disabled`
    * Internet Explorer internet zone download signed ActiveX controls: `Disable`
    * Internet Explorer internet zone download unsigned ActiveX controls: `Disable`
    * Internet Explorer internet zone cross site scripting filter: `Enabled`
    * Internet Explorer internet zone drag content from different domains across windows: `Disabled`
    * Internet Explorer internet zone drag content from different domains within windows: `Disabled`
    * Internet Explorer internet zone protected mode: `Enable`
    * Internet Explorer internet zone include local path when uploading files to server: `Disabled`
    * Internet Explorer internet zone initialize and script Active X controls not marked as safe: `Disable`
    * Internet Explorer internet zone java permissions: `Disable java`
    * Internet Explorer internet zone launch applications and files in an iframe: `Disable`
    * Internet Explorer internet zone logon options: `Prompt`
    * Internet Explorer internet zone navigate windows and frames across different domains: `Disable`
    * Internet Explorer internet zone run .NET Framework reliant components signed with Authenticode: `Disable`
    * Internet Explorer internet zone security warning for potentially unsafe files: `Prompt`
    * Internet Explorer internet zone popup blocker: `Enable`
    * Internet Explorer intranet zone do not run antimalware against Active X controls: `Disabled`
    * Internet Explorer intranet zone initialize and script Active X controls not marked as safe: `Disable`
    * Internet Explorer intranet zone java permissions: `High safety`
    * Internet Explorer local machine zone do not run antimalware against Active X controls: `Disabled`
    * Internet Explorer local machine zone java permissions: `Disable java`
    * Internet Explorer locked down internet zone smart screen: `Enabled`
    * Internet Explorer locked down intranet zone java permissions: `Disable java`
    * Internet Explorer locked down local machine zone java permissions: `Disable java`
    * Internet Explorer locked down restricted zone smart screen: `Enabled`
    * Internet Explorer locked down restricted zone java permissions: `Disable java`
    * Internet Explorer locked down trusted zone java permissions: `Disable java`
    * Internet Explorer processes MIME sniffing safety feature: `Enabled`
    * Internet Explorer processes MK protocol security restriction: `Enabled`
    * Internet Explorer processes notification bar: `Enabled`
    * Internet Explorer prevent per user installation of Active X controls: `Enabled`
    * Internet Explorer processes protection from zone elevation: `Enabled`
    * Internet Explorer remove run this time button for outdated Active X controls: `Enabled`
    * Internet Explorer processes restrict Active X install: `Enabled`
    * Internet Explorer restricted zone access to data sources: `Disable`
    * Internet Explorer restricted zone active scripting: `Disable`
    * Internet Explorer restricted zone automatic prompt for file downloads: `Disabled`
    * Internet Explorer restricted zone binary and script behaviors: `Disable`
    * Internet Explorer restricted zone copy and paste via script: `Disable`
    * Internet Explorer restricted zone drag and drop or copy and paste files: `Disable`
    * Internet Explorer restricted zone less privileged sites: `Disable`
    * Internet Explorer restricted zone loading of XAML files: `Disable`
    * Internet Explorer restricted zone meta refresh: `Disabled`
    * Internet Explorer restricted zone .NET Framework reliant components: `Disable`
    * Internet Explorer restricted zone allow only approved domains to use Active X controls: `Enabled`
    * Internet Explorer restricted zone allow only approved domains to use tdc Active X controls: `Enabled`
    * Internet Explorer restricted zone scripting of web browser controls: `Disabled`
    * Internet Explorer restricted zone script initiated windows: `Disabled`
    * Internet Explorer restricted zone scriptlets: `Disabled`
    * Internet Explorer restricted zone smart screen: `Enabled`
    * Internet Explorer restricted zone updates to status bar via script: `Disabled`
    * Internet Explorer restricted zone user data persistence: `Disabled`
    * Internet Explorer restricted zone allow vbscript to run: `Disable`
    * Internet Explorer restricted zone do not run antimalware against Active X controls: `Disabled`
    * Internet Explorer restricted zone download signed Active X controls: `Disable`
    * Internet Explorer restricted zone download unsigned Active X controls: `Disable`
    * Internet Explorer restricted zone cross site scripting filter: `Enabled`
    * Internet Explorer restricted zone drag content from different domains across windows: `Disabled`
    * Internet Explorer restricted zone drag content from different domains within windows: `Disabled`
    * Internet Explorer restricted zone include local path when uploading files to server: `Disabled`
    * Internet Explorer restricted zone initialize and script Active X controls not marked as safe: `Disable`
    * Internet Explorer restricted zone java permissions: `Disable java`
    * Internet Explorer restricted zone launch applications and files in an iFrame: `Disable`
    * Internet Explorer restricted zone logon options: `Anonymous`
    * Internet Explorer restricted zone navigate windows and frames across different domains: `Disable`
    * Internet Explorer restricted zone run Active X controls and plugins: `Disable`
    * Internet Explorer restricted zone run .NET Framework reliant components signed with Authenticode: `Disable`
    * Internet Explorer restricted zone scripting of java applets: `Disable`
    * Internet Explorer restricted zone security warning for potentially unsafe files: `Disable`
    * Internet Explorer restricted zone protected mode: `Enable`
    * Internet Explorer restricted zone popup blocker: `Enable`
    * Internet Explorer processes restrict file download: `Enabled`
    * Internet Explorer processes scripted window security restrictions: `Enabled`
    * Internet Explorer security zones use only machine settings: `Enabled`
    * Internet Explorer use Active X installer service: `Enabled`
    * Internet Explorer trusted zone do not run antimalware against Active X controls: `Disabled`
    * Internet Explorer trusted zone initialize and script Active X controls not marked as safe: `Disable`
    * Internet Explorer trusted zone java permissions: `High safety`
    * Internet Explorer auto complete: `Disabled`
  * Local Policies Security Options
    * Block remote logon with blank password: `Yes`
    * Minutes of lock screen inactivity until screen saver activates: `15`
    * Smart card removal behavior: `Lock workstation`
    * Require client to always digitally sign communications: `Yes`
    * Prevent clients from sending unencrypted passwords to third party SMB servers: `Yes`
    * Require server digitally signing communications always: `Yes`
    * Prevent anonymous enumeration of SAM accounts: `Yes`
    * Block anonymous enumeration of SAM accounts and shares: `Yes`
    * Restrict anonymous access to named pipes and shares: `Yes`
    * Allow remote calls to security accounts manager: `O:BAG:BAD:(A;;RC;;;BA)`
    * Prevent storing LAN manager hash value on next password change: `Yes`
    * Authentication level: `Send NTLMv2 response only. Refuse LM and NTLM`
    * Minimum session security for NTLM SSP based clients: `Require NTLM V2 and 128 encryption`
    * Minimum session security for NTLM SSP based servers: `Require NTLM V2 and 128 bit encryption`
    * Administrator elevation prompt behavior: `Prompt for credentials on the secure desktop`
    * Standard user elevation prompt behavior: `Prompt for credentials on the secure desktop`
    * Detect application installations and prompt for elevation: `Yes`
    * Only allow UI access applications for secure locations: `Yes`
    * Require admin approval mode for administrators: `Yes`
    * Use admin approval mode: `Yes`
    * Virtualize file and registry write failures to per user locations: `Yes`
  * Microsoft Defender 
    * Block Adobe Reader from creating child processes: `Enable`
    * Block Office communication apps from creating child processes: `Enable`
    * Enter how often (0-24 hours) to check for security intelligence updates: `4`
    * Defender schedule scan day: `No scheduled scan`
    * Cloud-delivered protection level: `Not configured`
    * Scan network files: `Yes`
    * Turn on real-time protection: `Yes`
    * Scan archive files: `Yes`
    * Turn on behavior monitoring: `Yes`
    * Turn on cloud-delivered protection: `Yes`
    * Scan incoming email messages: `Yes`
    * Scan removable drives during full scan: `Yes`
    * Block Office applications from injecting code into other processes: `Block`
    * Block Office applications from creating executable content: `Block`
    * Block all Office applications from creating child processes: `Block`
    * Block Win32 API calls from Office macro: `Block`
    * Block execution of potentially obfuscated scripts (js/vbs/ps): `Block`
    * Block JavaScript or VBScript from launching downloaded executable content: `Block`
    * Block executable content download from email and webmail clients: `Block`
    * Block credential stealing from the Windows local security authority subsystem (lsass.exe): `Block`
    * Defender potentially unwanted app action: `Block`
    * Block untrusted and unsigned processes that run from USB: `Block`
    * Enable network protection: `Enable`
    * Defender sample submission consent: `Send safe samples automatically`
  * MS Security Guide
    * SMB v1 client driver start configuration: `Disable`
    * Apply UAC restrictions to local accounts on network logon: `Enabled`
    * Structured exception handling overwrite protection: `Enabled`
    * SMB v1 server: `Disabled`
    * Digest authentication: `Disabled`
  * MSS Legacy
    * Network IPv6 source routing protection level: `Highest protection`
    * Network IP source routing protection level: `Highest protection`
    * Network ignore NetBIOS name release requests except from WINS servers: `Enabled`
    * Network ICMP redirects override OSPF generated routes: `Disabled`
  * Power
    * Require password on wake while on battery: `Enabled`
    * Require password on wake while plugged in: `Enabled`
    * Standby states when sleeping while on battery: `Disabled`
    * Standby states when sleeping while plugged in: `Disabled`
  * Remote Assistance
    * Remote Assistance solicited: `Disable Remote Assistance`
  * Remote Desktop Services
    * Remote desktop services client connection encryption level: `High`
    * Block drive redirection: `Enabled`
    * Block password saving: `Enabled`
    * Prompt for password upon connection: `Enabled`
    * Secure RPC communication: `Enabled`
  * Remote Management
    * Block client digest authentication: `Enabled`
    * Block storing run as credentials: `Enabled`
    * Client basic authentication: `Disabled`
    * Basic authentication: `Disabled`
    * Client unencrypted traffic: `Disabled`
    * Unencrypted traffic: `Disabled`
  * Remote Procedure Call
    * RPC unauthenticated client options: `Authenticated`
  * Search
    * Disable indexing encrypted items: `Yes`
  * Smart Screen
    * Turn on Windows SmartScreen: `Yes`
    * Block users from ignoring SmartScreen warnings: `Yes`
  * System
    * System boot start driver initialization: `Good and unknown`
  * Wi-Fi
    * Block Automatically connecting to Wi-Fi hotspots: `Yes`
    * Block Internet sharing: `Yes`
  * Windows Connection Manager
    * Block connection to non-domain networks: `Enabled`
  * Windows Hello for Business
    * Block Windows Hello for Business: `Enabled`
  * Windows Ink Workspace
    * Ink Workspace: `Enabled`
  * Windows PowerShell
    * PowerShell script block logging: `Enabled`
* Scope tags: `Default`
* Assignments
  * Included groups: `rol-Agency-Users`
  * Excluded groups: -

## ACSC1709Guidance-MDMSecurity-AdminUsers

`Microsoft Endpoint Manager > Endpoint security > Security baselines > Windows 10 Security Baseline`

* Name: `ACSC1709Guidance-MDMSecurity-AdminUsers`
* Description: `Does not block WinRM Basic Auth to allow Exchange PowerShell Management Console`
* Platform: `Windows 10 and later`
* Configuration settings
  * Above Lock
    * Voice activate apps from locked screen: `Disabled`
    * Block display of toast notifications: `Yes`
  * App Runtime
    * Microsoft accounts optional for Windows Store apps: `Enabled`
  * Application management
    * Block app installations with elevated privileges: `Yes`
    * Block user control over installations: `Yes`
    * Block game DVR (desktop only): `Yes`
  * Auto Play
    * Auto play default auto run behavior: `Do not execute`
    * Auto play mode: `Disabled`
    * Block auto play for non-volume devices: `Enabled`
  * BitLocker
    * BitLocker removable drive policy: `Configure`
    * Block write access to removable data-drives not protected by BitLocker: `Yes`
  * Browser
    * Block Password Manager: `Yes`
    * Require SmartScreen for Microsoft Edge: `Yes`
    * Block malicious site access: `Yes`
    * Block unverified file download: `Yes`
    * Prevent user from overriding certificate errors: `Yes`
  * Connectivity
    * Configure secure access to UNC paths: `Configure Windows to only allow access to the specified UNC paths after fulfilling additional security requirements`
    * Hardened UNC path list:
      * Security flags 1: `requireMutualAuthentication,requireIntegrity`
      * Server path 1: `\\*\SYSVOL`
      * Security flags 2: `requireMutualAuthentication,requireIntegrity`
      * Server path 2: `\\*\NETLOGON`
    * Block downloading of print drivers over HTTP: `Enabled`
    * Block Internet download for web publishing and online ordering wizards: `Enabled`
  * Credentials Delegation
    * Remote host delegation of non-exportable credentials: `Enabled`
  * Credentials UI
    * Enumerate administrators: `Disabled`
  * Data Protection
    * Block direct memory access: `Yes`
  * Device Guard
    * Virtualization based security: `Enable VBS with secure boot`
    * Enable virtualization based security: `Yes`
    * Launch system guard: `Enabled`
    * Turn on credential guard: `Enable with UEFI lock`
  * Device Installation
    * Hardware device installation by device identifiers: `Block hardware device installation`
    * Remove matching hardware devices: `Yes`
    * Hardware device identifiers that are blocked: `PCI\CC_0C0A`
    * Hardware device installation by setup classes: `Block hardware device installation`
    * Remove matching hardware devices: `Yes`
    * Hardware device identifiers that are blocked: `{d48179be-ec20-11d1-b6b8-00c04fa372a7}`
  * Device Lock
    * Require password: `Yes`
    * Required password: `Alphanumeric`
    * Password expiration (days): `60`
    * Password minimum character set count: `3`
    * Prevent reuse of previous passwords: `24`
    * Minimum password length: `8`
    * Number of sign-in failures before wiping device: `10`
    * Block simple passwords: `Yes`
    * Password minimum age in days: `1`
    * Prevent use of camera: `Enabled`
    * Prevent slide show: `Enabled`
  * DMA Guard
    * Enumeration of external devices incompatible with Kernel DMA Protection: `Block all`
  * Event Log Service
    * Application log maximum file size in KB: `65536`
    * System log maximum file size in KB: `65536`
    * Security log maximum file size in KB: `196608`
  * Experience
    * Block Windows Spotlight: `Yes`
  * Exploit Guard: `Upload XML`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<MitigationPolicy>
    <AppConfig Executable="ONEDRIVE.EXE">
        <DEP Enable="true" EmulateAtlThunks="false" />
        <ASLR Enable="true" ForceRelocateImages="true" />
        <ImageLoad BlockRemoteImageLoads="true" />
        <Payload EnableExportAddressFilter="true" EnableExportAddressFilterPlus="true" EnableImportAddressFilter="true" EnableRopStackPivot="true" EnableRopCallerCheck="true" EnableRopSimExec="true" />
    </AppConfig>
    <AppConfig Executable="firefox.exe">
        <DEP Enable="true" EmulateAtlThunks="false" />
        <ASLR ForceRelocateImages="true" RequireInfo="false" BottomUp="true" HighEntropy="false" />
    </AppConfig>
    <AppConfig Executable="fltldr.exe">
        <DEP Enable="true" EmulateAtlThunks="false" />
        <ImageLoad BlockRemoteImageLoads="true" />
        <ChildProcess DisallowChildProcessCreation="true" />
        <Payload EnableExportAddressFilter="true" EnableExportAddressFilterPlus="true" EnableImportAddressFilter="true" EnableRopStackPivot="true" EnableRopCallerCheck="true" EnableRopSimExec="true" />
    </AppConfig>
    <AppConfig Executable="GROOVE.EXE">
        <DEP Enable="true" EmulateAtlThunks="false" />
        <ASLR Enable="true" ForceRelocateImages="true" />
        <ImageLoad BlockRemoteImageLoads="true" />
        <Payload EnableExportAddressFilter="true" EnableExportAddressFilterPlus="true" EnableImportAddressFilter="true" EnableRopStackPivot="true" EnableRopCallerCheck="true" EnableRopSimExec="true" />
        <ChildProcess DisallowChildProcessCreation="false" />
    </AppConfig>
    <AppConfig Executable="Acrobat.exe">
        <DEP Enable="true" EmulateAtlThunks="false" />
        <ASLR ForceRelocateImages="true" RequireInfo="false" BottomUp="true" HighEntropy="false" />
        <Payload EnableExportAddressFilter="true" EnableExportAddressFilterPlus="true" EnableImportAddressFilter="true" EnableRopStackPivot="true" EnableRopCallerCheck="true" EnableRopSimExec="true" />
    </AppConfig>
    <AppConfig Executable="AcroRd32.exe">
        <DEP Enable="true" EmulateAtlThunks="false" />
        <ASLR ForceRelocateImages="true" RequireInfo="false" BottomUp="true" HighEntropy="false" />
        <Payload EnableExportAddressFilter="true" EnableExportAddressFilterPlus="true" EnableImportAddressFilter="true" EnableRopStackPivot="true" EnableRopCallerCheck="true" EnableRopSimExec="true" />
    </AppConfig>
    <AppConfig Executable="chrome.exe">
        <DEP Enable="true" EmulateAtlThunks="false" />
    </AppConfig>
    <AppConfig Executable="EXCEL.EXE">
        <DEP Enable="true" EmulateAtlThunks="false" />
        <ASLR Enable="true" ForceRelocateImages="true" />
        <Payload EnableExportAddressFilter="true" EnableExportAddressFilterPlus="true" EnableImportAddressFilter="true" EnableRopStackPivot="true" EnableRopCallerCheck="true" EnableRopSimExec="true" />
    </AppConfig>
    <AppConfig Executable="iexplore.exe">
        <DEP Enable="true" EmulateAtlThunks="false" />
        <ASLR ForceRelocateImages="true" RequireInfo="false" BottomUp="true" HighEntropy="false" />
        <Payload EnableExportAddressFilter="true" EnableExportAddressFilterPlus="true" EnableImportAddressFilter="true" EnableRopStackPivot="true" EnableRopCallerCheck="true" EnableRopSimExec="true" />
    </AppConfig>
    <AppConfig Executable="INFOPATH.EXE">
        <DEP Enable="true" EmulateAtlThunks="false" />
        <ASLR Enable="true" ForceRelocateImages="true" />
        <Payload EnableExportAddressFilter="true" EnableExportAddressFilterPlus="true" EnableImportAddressFilter="true" EnableRopStackPivot="true" EnableRopCallerCheck="true" EnableRopSimExec="true" />
    </AppConfig>
    <AppConfig Executable="java.exe">
        <DEP Enable="true" EmulateAtlThunks="false" />
        <Payload EnableExportAddressFilter="true" EnableExportAddressFilterPlus="true" EnableImportAddressFilter="true" EnableRopStackPivot="true" EnableRopCallerCheck="true" EnableRopSimExec="true" />
    </AppConfig>
    <AppConfig Executable="javaw.exe">
        <DEP Enable="true" EmulateAtlThunks="false" />
        <Payload EnableExportAddressFilter="true" EnableExportAddressFilterPlus="true" EnableImportAddressFilter="true" EnableRopStackPivot="true" EnableRopCallerCheck="true" EnableRopSimExec="true" />
    </AppConfig>
    <AppConfig Executable="javaws.exe">
        <DEP Enable="true" EmulateAtlThunks="false" />
        <Payload EnableExportAddressFilter="true" EnableExportAddressFilterPlus="true" EnableImportAddressFilter="true" EnableRopStackPivot="true" EnableRopCallerCheck="true" EnableRopSimExec="true" />
    </AppConfig>
    <AppConfig Executable="LYNC.EXE">
        <DEP Enable="true" EmulateAtlThunks="false" />
        <ASLR Enable="true" ForceRelocateImages="true" />
        <Payload EnableExportAddressFilter="true" EnableExportAddressFilterPlus="true" EnableImportAddressFilter="true" EnableRopStackPivot="true" EnableRopCallerCheck="true" EnableRopSimExec="true" />
    </AppConfig>
    <AppConfig Executable="MSACCESS.EXE">
        <DEP Enable="true" EmulateAtlThunks="false" />
        <ASLR Enable="true" ForceRelocateImages="true" />
        <Payload EnableExportAddressFilter="true" EnableExportAddressFilterPlus="true" EnableImportAddressFilter="true" EnableRopStackPivot="true" EnableRopCallerCheck="true" EnableRopSimExec="true" />
    </AppConfig>
    <AppConfig Executable="MSPUB.EXE">
        <DEP Enable="true" EmulateAtlThunks="false" />
        <ASLR Enable="true" ForceRelocateImages="true" />
        <Payload EnableExportAddressFilter="true" EnableExportAddressFilterPlus="true" EnableImportAddressFilter="true" EnableRopStackPivot="true" EnableRopCallerCheck="true" EnableRopSimExec="true" />
    </AppConfig>
    <AppConfig Executable="OIS.EXE">
        <DEP Enable="true" EmulateAtlThunks="false" />
        <Payload EnableExportAddressFilter="true" EnableExportAddressFilterPlus="true" EnableImportAddressFilter="true" EnableRopStackPivot="true" EnableRopCallerCheck="true" EnableRopSimExec="true" />
    </AppConfig>
    <AppConfig Executable="OUTLOOK.EXE">
        <DEP Enable="true" EmulateAtlThunks="false" />
        <ASLR Enable="true" ForceRelocateImages="true" />
        <Payload EnableExportAddressFilter="true" EnableExportAddressFilterPlus="true" EnableImportAddressFilter="true" EnableRopStackPivot="true" EnableRopCallerCheck="true" EnableRopSimExec="true" />
    </AppConfig>
    <AppConfig Executable="plugin-container.exe">
        <DEP Enable="true" EmulateAtlThunks="false" />
        <Payload EnableExportAddressFilter="true" EnableExportAddressFilterPlus="true" EnableImportAddressFilter="true" EnableRopStackPivot="true" EnableRopCallerCheck="true" EnableRopSimExec="true" />
    </AppConfig>
    <AppConfig Executable="POWERPNT.EXE">
        <DEP Enable="true" EmulateAtlThunks="false" />
        <ASLR Enable="true" ForceRelocateImages="true" />
        <Payload EnableExportAddressFilter="true" EnableExportAddressFilterPlus="true" EnableImportAddressFilter="true" EnableRopStackPivot="true" EnableRopCallerCheck="true" EnableRopSimExec="true" />
    </AppConfig>
    <AppConfig Executable="PPTVIEW.EXE">
        <DEP Enable="true" EmulateAtlThunks="false" />
        <ASLR Enable="true" ForceRelocateImages="true" />
        <Payload EnableExportAddressFilter="true" EnableExportAddressFilterPlus="true" EnableImportAddressFilter="true" EnableRopStackPivot="true" EnableRopCallerCheck="true" EnableRopSimExec="true" />
    </AppConfig>
    <AppConfig Executable="VISIO.EXE">
        <DEP Enable="true" EmulateAtlThunks="false" />
        <ASLR Enable="true" ForceRelocateImages="true" />
        <Payload EnableExportAddressFilter="true" EnableExportAddressFilterPlus="true" EnableImportAddressFilter="true" EnableRopStackPivot="true" EnableRopCallerCheck="true" EnableRopSimExec="true" />
    </AppConfig>
    <AppConfig Executable="VPREVIEW.EXE">
        <DEP Enable="true" EmulateAtlThunks="false" />
        <ASLR Enable="true" ForceRelocateImages="true" />
        <Payload EnableExportAddressFilter="true" EnableExportAddressFilterPlus="true" EnableImportAddressFilter="true" EnableRopStackPivot="true" EnableRopCallerCheck="true" EnableRopSimExec="true" />
    </AppConfig>
    <AppConfig Executable="WINWORD.EXE">
        <DEP Enable="true" EmulateAtlThunks="false" />
        <ASLR Enable="true" ForceRelocateImages="true" />
        <Payload EnableExportAddressFilter="true" EnableExportAddressFilterPlus="true" EnableImportAddressFilter="true" EnableRopStackPivot="true" EnableRopCallerCheck="true" EnableRopSimExec="true" />
    </AppConfig>
    <AppConfig Executable="wmplayer.exe">
        <DEP Enable="true" EmulateAtlThunks="false" />
        <Payload EnableExportAddressFilter="false" EnableExportAddressFilterPlus="false" EnableImportAddressFilter="false" EnableRopStackPivot="true" EnableRopCallerCheck="true" EnableRopSimExec="true" />
    </AppConfig>
    <AppConfig Executable="wordpad.exe">
        <DEP Enable="true" EmulateAtlThunks="false" />
        <Payload EnableExportAddressFilter="true" EnableExportAddressFilterPlus="true" EnableImportAddressFilter="true" EnableRopStackPivot="true" EnableRopCallerCheck="true" EnableRopSimExec="true" />
    </AppConfig>
</MitigationPolicy>
```

  * File Explorer
    * Block data execution prevention: `Disabled`
    * Block heap termination on corruption: `Disabled`
  * Firewall
    * Firewall profile domain: `Configure`
    * Inbound connections blocked: `Yes`
    * Outbound connections required: `Yes`
    * Inbound notifications blocked: `Yes`
    * Firewall enabled: `Allowed`
    * Firewall profile private: `Configure`
    * Inbound connections blocked: `Yes`
    * Outbound connections required: `Yes`
    * Inbound notifications blocked: `Yes`
    * Firewall enabled: `Allowed`
    * Firewall profile public: `Configure`
    * Inbound connections blocked: `Yes`
    * Outbound connections required: `Yes`
    * Inbound notifications blocked: `Yes`
    * Firewall enabled: `Allowed`
    * Connection security rules from group policy not merged: `Yes`
    * Policy rules from group policy not merged: `Yes`
  * Internet Explorer
    * Internet Explorer encryption support: `TLS v1.1`, `TLS v1.2``
    * Internet Explorer prevent managing smart screen filter: `Enable`
    * Internet Explorer restricted zone script Active X controls marked safe for scripting: `Disable`
    * Internet Explorer restricted zone file downloads: `Disable`
    * Internet Explorer certificate address mismatch warning: `Enabled`
    * Internet Explorer enhanced protected mode: `Enabled`
    * Internet Explorer fallback to SSL3: `No sites`
    * Internet Explorer software when signature is invalid: `Disabled`
    * Internet Explorer check server certificate revocation: `Enabled`
    * Internet Explorer check signatures on downloaded programs: `Enabled`
    * Internet Explorer processes consistent MIME handling: `Enabled`
    * Internet Explorer bypass smart screen warnings: `Disabled`
    * Internet Explorer bypass smart screen warnings about uncommon files: `Disabled`
    * Internet Explorer crash detection: `Disabled`
    * Internet Explorer download enclosures: `Disabled`
    * Internet Explorer ignore certificate errors: `Disabled`
    * Internet Explorer disable processes in enhanced protected mode: `Enabled`
    * Internet Explorer security settings check: `Enabled`
    * Internet Explorer Active X controls in protected mode: `Disabled`
    * Internet Explorer users adding sites: `Disabled`
    * Internet Explorer users changing policies: `Disabled`
    * Internet Explorer block outdated Active X controls: `Enabled`
    * Internet Explorer include all network paths: `Disabled`
    * Internet Explorer internet zone access to data sources: `Disable`
    * Internet Explorer internet zone automatic prompt for file downloads: `Disabled`
    * Internet Explorer internet zone copy and paste via script: `Disable`
    * Internet Explorer internet zone drag and drop or copy and paste files: `Disable`
    * Internet Explorer internet zone less privileged sites: `Disable`
    * Internet Explorer internet zone loading of XAML files: `Disable`
    * Internet Explorer internet zone .NET Framework reliant components: `Disable`
    * Internet Explorer internet zone allow only approved domains to use ActiveX controls: `Enabled`
    * Internet Explorer internet zone allow only approved domains to use tdc ActiveX controls: `Enabled`
    * Internet Explorer internet zone scripting of web browser controls: `Disabled`
    * Internet Explorer internet zone script initiated windows: `Disabled`
    * Internet Explorer internet zone scriptlets: `Disable`
    * Internet Explorer internet zone smart screen: `Enabled`
    * Internet Explorer internet zone updates to status bar via script: `Disabled`
    * Internet Explorer internet zone user data persistence: `Disabled`
    * Internet Explorer internet zone allow VBscript to run: `Disable`
    * Internet Explorer internet zone do not run antimalware against ActiveX controls: `Disabled`
    * Internet Explorer internet zone download signed ActiveX controls: `Disable`
    * Internet Explorer internet zone download unsigned ActiveX controls: `Disable`
    * Internet Explorer internet zone cross site scripting filter: `Enabled`
    * Internet Explorer internet zone drag content from different domains across windows: `Disabled`
    * Internet Explorer internet zone drag content from different domains within windows: `Disabled`
    * Internet Explorer internet zone protected mode: `Enable`
    * Internet Explorer internet zone include local path when uploading files to server: `Disabled`
    * Internet Explorer internet zone initialize and script Active X controls not marked as safe: `Disable`
    * Internet Explorer internet zone java permissions: `Disable java`
    * Internet Explorer internet zone launch applications and files in an iframe: `Disable`
    * Internet Explorer internet zone logon options: `Prompt`
    * Internet Explorer internet zone navigate windows and frames across different domains: `Disable`
    * Internet Explorer internet zone run .NET Framework reliant components signed with Authenticode: `Disable`
    * Internet Explorer internet zone security warning for potentially unsafe files: `Prompt`
    * Internet Explorer internet zone popup blocker: `Enable`
    * Internet Explorer intranet zone do not run antimalware against Active X controls: `Disabled`
    * Internet Explorer intranet zone initialize and script Active X controls not marked as safe: `Disable`
    * Internet Explorer intranet zone java permissions: `High safety`
    * Internet Explorer local machine zone do not run antimalware against Active X controls: `Disabled`
    * Internet Explorer local machine zone java permissions: `Disable java`
    * Internet Explorer locked down internet zone smart screen: `: `Enabled`
    * Internet Explorer locked down intranet zone java permissions: `Disable java`
    * Internet Explorer locked down local machine zone java permissions: `Disable java`
    * Internet Explorer locked down restricted zone smart screen: `Enabled`
    * Internet Explorer locked down restricted zone java permissions: `Disable java`
    * Internet Explorer locked down trusted zone java permissions: `Disable java`
    * Internet Explorer processes MIME sniffing safety feature: `Enabled`
    * Internet Explorer processes MK protocol security restriction: `Enabled`
    * Internet Explorer processes notification bar: `Enabled`
    * Internet Explorer prevent per user installation of Active X controls: `Enabled`
    * Internet Explorer processes protection from zone elevation: `Enabled`
    * Internet Explorer remove run this time button for outdated Active X controls: `Enabled`
    * Internet Explorer processes restrict Active X install: `Enabled`
    * Internet Explorer restricted zone access to data sources: `Disable`
    * Internet Explorer restricted zone active scripting: `Disable`
    * Internet Explorer restricted zone automatic prompt for file downloads: `Disabled`
    * Internet Explorer restricted zone binary and script behaviors: `Disable`
    * Internet Explorer restricted zone copy and paste via script: `Disable`
    * Internet Explorer restricted zone drag and drop or copy and paste files: `Disable`
    * Internet Explorer restricted zone less privileged sites: `Disable`
    * Internet Explorer restricted zone loading of XAML files: `Disable`
    * Internet Explorer restricted zone meta refresh: `Disabled`
    * Internet Explorer restricted zone .NET Framework reliant components: `Disable`
    * Internet Explorer restricted zone allow only approved domains to use Active X controls: `Enabled`
    * Internet Explorer restricted zone allow only approved domains to use tdc Active X controls: `Enabled`
    * Internet Explorer restricted zone scripting of web browser controls: `Disabled`
    * Internet Explorer restricted zone script initiated windows: `Disabled`
    * Internet Explorer restricted zone scriptlets: `Disabled`
    * Internet Explorer restricted zone smart screen: `Enabled`
    * Internet Explorer restricted zone updates to status bar via script: `Disabled`
    * Internet Explorer restricted zone user data persistence: `Disabled`
    * Internet Explorer restricted zone allow vbscript to run: `Disable`
    * Internet Explorer restricted zone do not run antimalware against Active X controls: `Disabled`
    * Internet Explorer restricted zone download signed Active X controls: `Disable`
    * Internet Explorer restricted zone download unsigned Active X controls: `Disable`
    * Internet Explorer restricted zone cross site scripting filter: `Enabled`
    * Internet Explorer restricted zone drag content from different domains across windows: `Disabled`
    * Internet Explorer restricted zone drag content from different domains within windows: `Disabled`
    * Internet Explorer restricted zone include local path when uploading files to server: `Disabled`
    * Internet Explorer restricted zone initialize and script Active X controls not marked as safe: `Disable`
    * Internet Explorer restricted zone java permissions: `Disable java`
    * Internet Explorer restricted zone launch applications and files in an iFrame: `Disable`
    * Internet Explorer restricted zone logon options: `Anonymous`
    * Internet Explorer restricted zone navigate windows and frames across different domains: `Disable`
    * Internet Explorer restricted zone run Active X controls and plugins: `Disable`
    * Internet Explorer restricted zone run .NET Framework reliant components signed with Authenticode: `Disable`
    * Internet Explorer restricted zone scripting of java applets: `Disable`
    * Internet Explorer restricted zone security warning for potentially unsafe files: `Disable`
    * Internet Explorer restricted zone protected mode: `Enable`
    * Internet Explorer restricted zone popup blocker: `Enable`
    * Internet Explorer processes restrict file download: `Enabled`
    * Internet Explorer processes scripted window security restrictions: `Enabled`
    * Internet Explorer security zones use only machine settings: `Enabled`
    * Internet Explorer use Active X installer service: `Enabled`
    * Internet Explorer trusted zone do not run antimalware against Active X controls: `Disabled`
    * Internet Explorer trusted zone initialize and script Active X controls not marked as safe: `Disable`
    * Internet Explorer trusted zone java permissions: `High safety`
    * Internet Explorer auto complete: `Disabled`
  * Local Policies Security Options
    * Block remote logon with blank password: `Yes`
    * Minutes of lock screen inactivity until screen saver activates: `15`
    * Smart card removal behavior: `Lock workstation`
    * Require client to always digitally sign communications: `Yes`
    * Prevent clients from sending unencrypted passwords to third party SMB servers: `Yes`
    * Require server digitally signing communications always: `Yes`
    * Prevent anonymous enumeration of SAM accounts: `Yes`
    * Block anonymous enumeration of SAM accounts and shares: `Yes`
    * Restrict anonymous access to named pipes and shares: `Yes`
    * Allow remote calls to security accounts manager: `O:BAG:BAD:(A;;RC;;;BA)`
    * Prevent storing LAN manager hash value on next password change: `Yes`
    * Authentication level: `Send NTLMv2 response only. Refuse LM and NTLM`
    * Minimum session security for NTLM SSP based clients: `Require NTLM V2 and 128 encrpytion`
    * Minimum session security for NTLM SSP based servers: `Require NTLM V2 and 128 bit encryption`
    * Administrator elevation prompt behavior: `Prompt for credentials on the secure desktop`
    * Standard user elevation prompt behavior: `Prompt for credentials on the secure desktop`
    * Detect application installations and prompt for elevation: `Yes`
    * Only allow UI access applications for secure locations: `Yes`
    * Require admin approval mode for administrators: `Yes`
    * Use admin approval mode: `Yes`
    * Virtualize file and registry write failures to per user locations: `Yes`
  * Microsoft Defender
    * Block Adobe Reader from creating child processes: `Enable`
    * Block Office communication apps from creating child processes: `Enable`
    * Enter how often (0-24 hours) to check for security intelligence updates: `4`
    * Defender schedule scan day: `No scheduled scan`
    * Cloud-delivered protection level: `Not configured`
    * Scan network files: `Yes`
    * Turn on real-time protection: `Yes`
    * Scan archive files: `Yes`
    * Turn on behavior monitoring: `Yes`
    * Turn on cloud-delivered protection: `Yes`
    * Scan incoming email messages: `Yes`
    * Scan removable drives during full scan: `Yes`
    * Block Office applications from injecting code into other processes: `Block`
    * Block Office applications from creating executable content: `Block`
    * Block all Office applications from creating child processes: `Block`
    * Block Win32 API calls from Office macro: `Block`
    * Block execution of potentially obfuscated scripts (js/vbs/ps): `Block`
    * Block JavaScript or VBScript from launching downloaded executable content: `Block`
    * Block executable content download from email and webmail clients: `Block`
    * Block credential stealing from the Windows local security authority subsystem (lsass.exe): `Block`
    * Defender potentially unwanted app action: `Block`
    * Block untrusted and unsigned processes that run from USB: `Block`
    * Enable network protection: `Enable`
    * Defender sample submission consent: `Send safe samples automatically`
  * MS Security Guide
    * SMB v1 client driver start configuration: `Disable`
    * Apply UAC restrictions to local accounts on network logon: `Enabled`
    * Structured exception handling overwrite protection: `Enabled`
    * SMB v1 server: `Disabled`
    * Digest authentication: `Disabled`
  * MSS Legacy
    * Network IPv6 source routing protection level: `Highest protection`
    * Network IP source routing protection level: `Highest protection`
    * Network ignore NetBIOS name release requests except from WINS servers: `Enabled`
    * Network ICMP redirects override OSPF generated routes: `Disabled`
  * Power
    * Require password on wake while on battery: `Enabled`
    * Require password on wake while plugged in: `Enabled`
    * Standby states when sleeping while on battery: `Disabled`
    * Standby states when sleeping while plugged in: `Disabled`
  * Remote Assistance
    * Remote Assistance solicited: `Disable Remote Assistance`
  * Remote Desktop Services
    * Remote desktop services client connection encryption level: `High`
    * Block drive redirection: `Enabled`
    * Block password saving: `Enabled`
    * Prompt for password upon connection: `Enabled`
    * Secure RPC communication: `Enabled`
  * Remote Management
    * Block client digest authentication: `Enabled`
    * Block storing run as credentials: `Enabled`
    * Client basic authentication: `Disabled`
    * Basic authentication: `Enabled`
    * Client unencrypted traffic: `Disabled`
    * Unencrypted traffic: `Disabled`
  * Remote Procedure Call
    * RPC unauthenticated client options: `Authenticated`
  * Search
    * Disable indexing encrypted items: `Yes`
  * Smart Screen
    * Turn on Windows SmartScreen: `Yes`
    * Block users from ignoring SmartScreen warnings: `Yes`
  * System
    * System boot start driver initialization: `Good and unknown`
  * Wi-Fi
    * Block Automatically connecting to Wi-Fi hotspots: `Yes`
    * Block Internet sharing: `Yes`
  * Windows Connection Manager
    * Block connection to non-domain networks: `Enabled`
  * Windows Hello for Business
    * Block Windows Hello for Business: `Enabled`
  * Windows Ink Workspace
    * Ink Workspace: `Enabled`
  * Windows PowerShell
    * PowerShell script block logging: `Enabled`



* Scope tags: `Default`
* Assignments
  * Included groups: `rol-Agency-Administrators`
  * Excluded groups: -
