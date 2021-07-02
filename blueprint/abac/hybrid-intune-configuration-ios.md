---
layout: page
title: Hybrid - Microsoft Endpoint Manager - Intune configuration for iOS devices
menu: abac
---

The ABAC settings for the Agency profiles can be found below. This includes iOS Device Restrictions and iOS Device Feature Policy settings. If a setting is not mentioned in the below, it should be assumed to have been left at its default setting.

The following can be found at `Microsoft Endpoint Manager > Devices > Configuration profiles`

## iOS device restrictions

* Name: `iOS Device Restrictions`
* Description: -
* Type: `iOS/iPadOS`
* Profile Type: `Device restrictions`
* Configuration settings
  * General
    * Block sending diagnostic and usage data to Apple: `Yes`
    * Block screenshots and screen recording: `Yes`
    * Block untrusted TLS certificates: `Yes`
    * Block over-the-air PKI updates: `Yes`
    * Force limited ad tracking: `Yes`
    * Block trusting new enterprise app authors: `Yes`
    * Block app clips: `Yes`
    * Limit Apple personalized advertising: `Yes`
    * Block modification of account settings: `Block`
    * Block Screen Time: `Yes`
    * Block use of erase all content and settings: `Yes`
    * Block modification of device name: `Yes`
    * Block modification of notifications settings: `Not configured`
    * Block modification of Wallpaper: `Yes`
    * Block configuration profile changes: `Yes`
    * Allow activation lock: `Yes`
    * Block removing apps: `Not configured`
    * Allow USB accessories while device is locked: `Not configured`
    * Force automatic date and time: `Yes`
    * Require teacher permission to leave Classroom app unmanaged classes: `Not configured`
    * Allow Classroom to lock to an app and lock the device without prompting: `Not configured`
    * Allow students to automatically join Classroom classes without prompting: `Not configured`
    * Block VPN creation: `Not configured`
    * Block modification of eSIM settings: `Yes`
    * Defer software updates: `Not configured`
  * Password
    * Require Password: `Yes`
    * Block Simple passwords: `Yes`
    * Required password type: `Alphanumeric`
    * Number of non-alphanumerics characters in password: `1`
    * Minimum password length: `14`
    * Number of sign-in failures before wiping the device: `11`
    * Maximum minutes after screen lock before password is required: `Immediately`
    * Maximum minutes of inactivity until screen locks: `2 Minutes`
    * Password expiration (days): `365`
    * Prevent reuse of previous passwords: `5`
    * Block Touch ID and Face ID unlock: `Yes`
  * Automated device enrollment
    * Block Passcode modification: `Not configured`
    * Block modification of Touch and Face ID faces: `Not configured`
    * Block password AutoFill: `Yes`
    * Block password proximity requests: `Yes`
    * Block password sharing: `Yes`
    * Require Touch ID and Face ID authentication for password or credit card information AutoFill: `Yes`
  * Lock Screen Experience
    * Block Control Center access while device locked: `Not configured`
    * Block Notification Center access while device locked: `Yes`
    * Block Today view while device locked: `Yes`
    * Block Wallet notifications while device locked: `Block`
  * App Store, Doc Viewing, Gaming
    * Block viewing corporate documents in unmanaged apps: `Yes`
    * Allow unmanaged apps to read from managed contacts accounts: `Not configured`
    * Treat AirDrop as an unmanaged destination: `Yes`
    * Viewing non-corporate documents in corporate apps: `Block`
    * Require iTunes Store password for all purchases: `Not configured`
    * Block In-app purchases: `Yes`
    * Block download of explicit sexual content in Apple Books: `Yes`
    * Allow managed apps to write contacts to unmanaged contacts accounts: `Not configured`
    * Rating Region: `No region configured`
    * Block App store: `Yes`
    * Block installing apps using App Store: `Not configured`
    * Block automatic app downloads: `Not configured`
    * Block playback of explicit iTunes music, podcast, or news content: `Yes`
    * Block adding Game Center friends: `Yes`
    * Block game Center: `Yes`
    * Block multiplayer gaming: `Yes`
    * Block Access to network drive in Files app: `Yes`
  * Built-in Apps
    * Block Siri: `Yes`
    * Block Siri while device is locked: `N/A`
    * Require Safari fraud warnings: `Yes`
    * Block spotlight search to return results from internet: `Yes`
    * Safari cookies: `Block all cookies`
    * Block Safari JavaScript: `Yes`
    * Block Safari Pop-ups: `Yes`
    * Block Camera: `Not configured`
    * Block FaceTime: `Not configured`
    * Require Siri proximity filter: `N/A`
    * Block user-generated content in Siri: `N/A`
    * Block Apple News: `Not configured`
    * Block Apple Book: `Yes`
    * Block iMessage: `Not configured`
    * Block Podcasts: `Yes`
    * Music service: `Yes`
    * Block iTunes Radio service: `Yes`
    * Block iTunes Store: `Yes`
    * Block Find My iPhone: `Not configured`
    * Block Find My Friends: `Yes`
    * Block user modification to the Find My Friends settings: `Yes`
    * Block removal of system apps from device: `Not configured`
    * Block Safari: `Not configured`
    * Block Safari Autofill: `Yes`
  * Restricted Apps
    * Type of restricted app list: `Approved apps`

    App store URL | App bundle ID | App Name | Publisher
    --- | --- | ---
    https://apps.apple.com/au/app/adobe-acrobat-reader-for-pdf/id469337564 | com.adobe.Adobe-Reader | Adobe Acrobat Reader for PDF | Adobe Inc
    https://apps.apple.com/us/app/microsoft-authenticator/id983156458 | com.microsoft.azureauthenticator | Microsoft Authenticator | Microsoft Corporation
    https://apps.apple.com/us/app/microsoft-edge/id1288723196 | com.microsoft.msedge | Microsoft Edge | Microsoft Corporation
    https://apps.apple.com/us/app/microsoft-edge/id1288723196 | com.microsoft.Office.Excel | Microsoft Excel | Microsoft Corporation
    https://apps.apple.com/us/app/microsoft-onedrive/id477537958 | com.microsoft.skydrive | Microsoft OneDrive | Microsoft Corporation
    https://apps.apple.com/au/app/microsoft-onenote/id410395246 | com.microsoft.onenote | Microsoft OneNote | Microsoft Corporation
    https://apps.apple.com/au/app/microsoft-powerpoint/id586449534 | com.microsoft.Office.Powerpoint | Microsoft PowerPoint | Microsoft Corporation
    https://apps.apple.com/us/app/microsoft-outlook/id951937596 | com.microsoft.Office.Outlook | Microsoft Outlook | Microsoft Corporation
    https://apps.apple.com/au/app/microsoft-sharepoint/id1091505266 | com.microsoft.sharepoint | Microsoft SharePoint | Microsoft Corporation
    https://apps.apple.com/us/app/microsoft-teams/id1113153706 | com.microsoft.skype.teams | Microsoft Teams | Microsoft Corporation
    https://apps.apple.com/us/app/microsoft-word/id586447913 | com.microsoft.Office.Word | Microsoft Word | Microsoft Corporation
    https://apps.apple.com/au/app/power-apps/id1047318566 | com.microsoft.msapps | PowerApps | Microsoft Corporation

  * Show or Hide Apps: 
    * Type of app list: `Not configured`
  * Wireless
    * Block data roaming: `Not configured`
    * Block global background fetch while roaming: `Not configured`
    * Block voice dialing while device is locked: `Yes`
    * Block voice roaming: `Not configured`
    * Block personal Hotspot: `Not configured`
    * Block use of cellular data: `Not configured`
    * Block use of cellular data when roaming: `Not configured`
    * Block changes to app cellular data usage settings: `Not configured`
    * Block changes to cellular plan settings: `Not configured`
    * Block modification of Personal Hotspot: `Not configured`
    * Require joining Wi-Fi networks only using configuration profiles: `Yes`
    * Require Wi-Fi always turned on: `Not configured`
  * Connected Devices
    * Force wrist detection for paired Apple Watch: `Yes`
    * Require AirPlay outgoing requests pairing password: `Yes`
    * Block AirDrop: `Yes`
    * Block Apple Watch pairing: `Yes`
    * Block modifying Bluetooth settings: `Not configured`
    * Block pairing with non-Configurator hosts: `Yes`
    * Block AirPrint: `Block`
    * Block storage of AirPrint credentials in Keychain: `N/A`
    * Require AirPrint to destinations with trusted certificates: `N/A`
    * Block iBeacon discovery of AirPrint printers: `N/A`
    * Block access to USB drive in Files app: `Yes`
    * Disable near-field communication (NFC): `Yes`
  * Keyboard and Dictionary
    * Block Word definition lookup: `Not configured`
    * Block Predictive keyboards: `Not configured`
    * Block Auto-correction: `Not configured`
    * Block Keyboard spell-check: `Not configured`
    * Block Keyboard shortcuts: `Not configured`
    * Block Dictation: `Not configured`
    * Block QuickPath: `Not configured`
  * Cloud and Storage
    * Force Encrypted backup: `Yes`
    * Block managed apps from storing data in iCloud: `Yes`
    * Block Enterprise Book Backup: `Yes`
    * Block notes and highlights sync for enterprise books: `Yes`
    * Block iCloud Photos sync: `Yes`
    * Block iCloud Photo Library: `Yes`
    * Block My photo stream: `Yes`
    * Block Handoff: `Yes`
    * Backup to iCloud: `Yes`
    * Block iCloud document and data sync: `Yes`
    * Block iCloud Keychain sync: `Yes`
  * Autonomous Single App Mode
    * App name: `Not configured`
    * App Bundle ID: `Not configured`
  * Kiosk
    * App to run in kiosk mode: `Not configured`
  * Domains
    * Unmarked email domains: `Not configured`
    * Managed web domains: `Not configured`
    * Safari password auto fill domains: `Not configured`
  * Shared iPad
    * Block Shared iPad temporary sessions: `Yes`
* Scope tags: `Default`
* Assignments
  * Included groups: 
```
rol-Agency-Users
Dev Admins
rol-Agency-Administrators
Office365_Grant_AzureAD_Join 
```
  * Excluded groups: -

## iOS device feature policy

* Name: `iOS Device Feature Policy`
* Description: -
* Type: `iOS/iPadOS`
* Profile Type: `Device Features`
* Configuration settings
  * Lock Screen Message	
    * "If Lost, Return to..." Message: `If lost return to the <Agency Name>`
  * Single Sign On
    * Renewal certificate: `None selected`
  * Wallpaper
    * Wallpaper Display Location: `Lock Screen and Home Screen`
    * Wallpaper Image: `Agency wallpaper`
* Scope tags: `Default`
* Assignments
  * Included groups: 
```
rol-Agency-Users
Dev Admins
rol-Agency-Administrators
Office365_Grant_AzureAD_Join 
```
  * Excluded groups: -

## iOS email configuration

* Name: `iOS email configuration`
* Description: -
* Type: `iOS/iPadOS` 
* Profile Type: `Email`
* Configuration settings
  * Email server: `outlook.office365.com`
  * Account name: `Corporate Email`
  * Username attribute from AAD: `User Principal Name`
  * Email address attribute from AAD: `Primary SMTP Address`
  * Authentication method: `Username and password`
  * SSL: `Enable`
  * OAuth: `Enable`
  * Exchange data to sync: `All data`
  * Allow users to change sync settings: `Yes`
  * S/MIME: `Disable S/MIME`
  * Amount of email to synchronize: `One Month`
  * Allow messages to be moved to other email accounts: `Disable`
  * Allow email to be sent from third party applications: `Disable`
  * Synchronize recently used email addresses : `Enable`
  * VPN profile for per account VPN: ``
* Scope tags: `Default`
* Assignments
  * Included groups: `rol-Agency-Administrators`, `rol-Agency-Users`
  * Excluded groups: `grp-Windows-10-Devices`

## Wifi

* Name: `iOS WiFi SSID`
* Description: -
* Type: `iOS/iPadOS`
* Profile Type: `Wi-Fi`
* Configuration settings
  * Wi-Fi type: `Enterprise`
  * Wi-Fi name (SSID): `WiFi SSID`
  * Connection Name: `WiFi SSID`
  * Connect automatically: `Enable`
  * Hidden network: `Disable`
  * Wireless Security Type: `WPA/WPA2-Enterprise`
  * EAP type: `EAP - TLS`
  * Certificate server names: `********`
  * Client authentication
    * Authentication method: `Certificates`
  * Identity privacy (outer identity): `Anonymous`
  * Proxy settings: `None`
  * Disable MAC address randomization: `Yes`
* Scope tags: `Default`
* Assignments
  * Included groups: `rol-Agency-Administrators`, `rol-Agency-Users`
  * Excluded groups: `grp-Windows-10-Devices`

  ## Per app VPN

* Name: `iOS per app VPN`
* Description: -
* Type: `iOS/iPadOS` 
* Profile Type: `VPN`
* Configuration settings
  * Connection Type: `VPN Provider`
  * Connection name: `PerApp VPN`
  * VPN Address: `XXX.XXX.XXX.XXX`
  * Authentication method: `username and password`
  * Split tunnelling: `Disabled`
  * Type of automatic VPN: `Per-app VPN`
  * Block users from disabling automatic VPN: `Yes`
* Assignments
  * Included groups: `rol-Agency-Administrators`, `rol-Agency-Users`
  * Excluded groups: `grp-Windows-10-Devices`