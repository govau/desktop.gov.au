---
layout: page
title: Hybrid - Microsoft Endpoint Manager - Intune applications for iOS devices
menu: abac
---

## Apple Business Manager Applications

### Apple Business Manager VPP licenses

For each of the following apps complete the license purchase in the Apple Business Manager Portal:
* Applications: 
  * Microsoft Company Portal
  * Adobe Acrobat Reader
  * Microsoft PowerApps
  * Microsoft Edge
  * Microsoft Excel
  * Microsoft Outlook
  * Microsoft PowerPoint
  * Microsoft Word
  * Microsoft Office
  * Microsoft OneNote
  * Microsoft Planner
  * Microsoft Power BI
  * Azure Information Protection
  * Microsoft SharePoint
  * Microsoft StaffHub
  * Microsoft OneDrive
  * Microsoft Teams
  * Microsoft Stream
  * Microsoft Visio Viewer
  * Microsoft Whiteboard
* Assign to: `Intune MDM`
* Quantity: `Number of Agency Devices`


### Apple Business Manager location token

`Apple Business Manager > Settings > Apps and Books`

* Location Token: `Download Location Token`

`Microsoft Endpoint Manager > Tenant Administration > Connectors and tokens > Apple VPP tokens`

* Token Name: `<Agency Acronym> VPP token`
* Apple ID: `<Agency ID>@<Agency>.gov.au`
* VPP token file: `Downloaded from Apple Business Manager above`
* Settings:
  * Take control of token from another MDM: `Yes`
  * Country/Region: `Australia`
  * Type of VPP account: `Business`
  * Automatic updates: `Yes`
  * I grant Microsoft permission to send both user and device information to Apple: `I agree`
* Scope tags: `Default`

## Intune applications

The following Microsoft Endpoint Manager - Intune (Intune) applications can be found in the Azure Portal at `Microsoft Endpoint Manager > Apps > All apps`

### Adobe Acrobat Reader for PDF

* Name: `Adobe Acrobat Reader for PDF`
* Description: `The free Adobe Acrobat Reader PDF app is trusted by millions of people around the world. Adobe Acrobat Reader is the free global standard for reliably viewing, signing, and commenting on PDF documents.`
* Type: `iOS store app`
* App information
  * Publisher: `Adobe Inc.`
  * Appstore URL: `https://apps.apple.com/au/app/adobe-acrobat-reader-for-pdf/id469337564?uo=4`
  * Minimum operating system: `iOS 14.0`
  * Applicable device type: `iPad`, `iPhone and iPod`
  * Category: -
  * Show this as a featured app in the Company Portal: `Yes`
  * Information URL: -
  * Privacy URL: -
  * Developer: -
  * Owner: -
  * Notes: -
  * Logo: ![Adobe Acrobat Reader for PDF](/assets/images/abac/adobe-acrobat-reader.png)
* Scope tags: `Default`
* Assignments
  * Required: -
  * Available for enrolled devices: `rol-Agency-Administrators`, `rol-Agency-Users`
  * Available with or without enrollment: -
  * Uninstall: -
  * License Type: `Device licensing`

### Microsoft Authenticator

* Name: `Microsoft Authenticator`
* Description: `Sign in is easy, convenient, and secure when you use Microsoft Authenticator. Use your phone, not your password, to log into your Microsoft account. You simply have to enter your username, then approve the notification sent to your phone. Your fingerprint, face ID, or PIN will provide a second layer of security in this two step verification process. After, you'll have access to all your Microsoft products`
* Type: `iOS store app`
* App information
  * Publisher: `Microsoft Corporation`
  * Appstore URL: `https://apps.apple.com/us/app/microsoft-authenticator/id983156458?uo=4`
  * Minimum operating system: `iOS 14.0`
  * Applicable device type: `iPad`, `iPhone and iPod`
  * Category: -
  * Show this as a featured app in the Company Portal: `Yes`
  * Information URL: -
  * Privacy URL: -
  * Developer: -
  * Owner: -
  * Notes: -
  * Logo: ![Microsoft Authenticator](/assets/images/abac/microsoft-authenticator.png)
* Scope tags: `Default`
* Assignments
  * Required: `rol-Agency-Administrators`, `rol-Agency-Users`
  * Available for enrolled devices: -
  * Available with or without enrollment: -
  * Uninstall: -
  * License Type: `Device licensing`

### Microsoft Edge

* Name: `Microsoft Edge`
* Description: `Introducing the new Microsoft Edge. It’s time to expect more. More privacy. More control. More productivity. More value. Browse anywhere with one continuous experience from your phone to your computer and other signed-in devices. Microsoft Edge gives you the tools to protect your privacy online with features like tracking prevention, AdBlock Plus and InPrivate mode. Organise the web in a way that cuts through the clutter, making it easier to find, view and manage your content on-the-go. With world class compatibility, performance and new features, the new Microsoft Edge is the only browser you’ll ever need.`
* Type: `iOS store app`
* App information
  * Publisher: `Microsoft Corporation`
  * Appstore URL: `https://apps.apple.com/us/app/microsoft-edge/id1288723196?uo=4`
  * Minimum operating system: `iOS 14.0`
  * Applicable device type: `iPad`, `iPhone and iPod`
  * Category: -
  * Show this as a featured app in the Company Portal: `Yes`
  * Information URL: -
  * Privacy URL: -
  * Developer: -
  * Owner: -
  * Notes: -
  * Logo: ![Microsoft Edge](/assets/images/abac/microsoft-edge.png)
* Scope tags: `Default`
* Assignments
  * Required: `grp-iOS-Dynamic`
  * Available for enrolled devices: -
  * Available with or without enrollment: -
  * Uninstall: -
  * License Type: `Device licensing`

### Microsoft Excel

* Name: `Microsoft Excel`
* Description: `Microsoft Excel, the spreadsheet app, lets you create, view, edit, and share your files quickly and easily. Manage spreadsheets, tables and workbooks attached to email messages from your phone with this powerful productivity app from Microsoft.`
* Type: `iOS store app`
* App information
  * Publisher: `Microsoft Corporation`
  * Appstore URL: `https://apps.apple.com/us/app/microsoft-excel/id586683407?uo=4`
  * Minimum operating system: `iOS 14.0`
  * Applicable device type: `iPad`, `iPhone and iPod`
  * Category: -
  * Show this as a featured app in the Company Portal: `Yes`
  * Information URL: -
  * Privacy URL: -
  * Developer: -
  * Owner: -
  * Notes: -
  * Logo: ![Microsoft Excel](/assets/images/abac/microsoft-excel.png)
* Scope tags: `Default`
* Assignments
  * Required: `grp-iOS-Dynamic`
  * Available for enrolled devices: -
  * Available with or without enrollment: -
  * Uninstall: -
  * License Type: `Device licensing`

### Microsoft OneDrive

* Name: `Microsoft OneDrive`
* Description: `Do more wherever you go with Microsoft OneDrive. Get to and share your documents, photos, and other files from your iOS device, computer (PC or Mac), and any other devices you use. Use the Office mobile apps to stay productive and work together, no matter where you are. The OneDrive app for iOS lets you easily work with your personal and work files when you're on the go.`
* Type: `iOS store app`
* App information
  * Publisher: `Microsoft Corporation`
  * Appstore URL: `https://apps.apple.com/us/app/microsoft-onedrive/id477537958?uo=4`
  * Minimum operating system: `iOS 14.0`
  * Applicable device type: `iPad`, `iPhone and iPod`
  * Category: -
  * Show this as a featured app in the Company Portal: `Yes`
  * Information URL: -
  * Privacy URL: -
  * Developer: -
  * Owner: -
  * Notes: -
  * Logo: ![Microsoft OneDrive](/assets/images/abac/microsoft-onedrive.png)
* Scope tags: `Default`
* Assignments
  * Required: `grp-iOS-Dynamic`
  * Available for enrolled devices: -
  * Available with or without enrollment: -
  * Uninstall: -
  * License Type: `Device licensing`

### Microsoft OneNote

* Name: `Microsoft OneNote`
* Description: `With OneNote, you can plan that big event, seize that moment of inspiration to create something new, and track that list of errands that are too important to forget. Write memos, make a digital sketchbook and save notes right on your phone.`
* Type: `iOS store app`
* App information
  * Publisher: `Microsoft Corporation`
  * Appstore URL: `https://apps.apple.com/au/app/microsoft-onenote/id410395246?uo=4`
  * Minimum operating system: `iOS 14.0`
  * Applicable device type: `iPad`, `iPhone and iPod`
  * Category: -
  * Show this as a featured app in the Company Portal: `Yes`
  * Information URL: -
  * Privacy URL: -
  * Developer: -
  * Owner: -
  * Notes: -
  * Logo: ![Microsoft OneNote](/assets/images/abac/microsoft-onenote.png)
* Scope tags: `Default`
* Assignments
  * Required: -
  * Available for enrolled devices: `grp-iOS-Dynamic`
  * Available with or without enrollment: -
  * Uninstall: -
  * License Type: `Device licensing`

### Microsoft Outlook

* Name: `Microsoft Outlook`
* Description: `Outlook lets you bring all your email accounts and calendars in one convenient spot. Whether it's staying on top of your inbox or scheduling the next big thing, we make it easy to be your most productive, organized, and connected self.`
* Type: `iOS store app`
* App information
  * Publisher: `Microsoft Corporation`
  * Appstore URL: `https://apps.apple.com/us/app/microsoft-outlook/id951937596?uo=4`
  * Minimum operating system: `iOS 14.0`
  * Applicable device type: `iPad`, `iPhone and iPod`
  * Category: `Productivity`
  * Show this as a featured app in the Company Portal: `Yes`
  * Information URL: -
  * Privacy URL: -
  * Developer: -
  * Owner: -
  * Notes: -
  * Logo: ![Microsoft Outlook](/assets/images/abac/microsoft-outlook.png)
* Scope tags: `Default`
* Assignments
  * Required: `grp-iOS-Dynamic`
  * Available for enrolled devices: -
  * Available with or without enrollment: -
  * Uninstall: -
  * License Type: `Device licensing`

### Microsoft PowerPoint

* Name: `Microsoft PowerPoint`
* Description: `The PowerPoint app gives you access to the familiar slideshow maker tool you already know. Create, edit, view, present, or share presentations quickly and easily from anywhere. Need to access your PowerPoint files quickly while on the go? PowerPoint provides a quick view of your recent slides and presentations for easy access on any device.`
* Type: `iOS store app`
* App information
  * Publisher: `Microsoft Corporation`
  * Appstore URL: `https://apps.apple.com/au/app/microsoft-powerpoint/id586449534?uo=4`
  * Minimum operating system: `iOS 14.0`
  * Applicable device type: `iPad`, `iPhone and iPod`
  * Category: `Productivity`
  * Show this as a featured app in the Company Portal: `Yes`
  * Information URL: -
  * Privacy URL: -
  * Developer: -
  * Owner: -
  * Notes: -
  * Logo: ![Microsoft PowerPoint](/assets/images/abac/microsoft-powerpoint.png)
* Scope tags: `Default`
* Assignments
  * Required: -
  * Available for enrolled devices: `grp-iOS-Dynamic`
  * Available with or without enrollment: -
  * Uninstall: -
  * License Type: `Device licensing`

### Microsoft SharePoint

* Name: `Microsoft SharePoint`
* Description: `Find files, news, sites and people - personalized for you and your organization. SharePoint infuses AI into the mobile app experience to keep you focused and productive while on the go. AI builds an understanding of what you work on, how you work, and how your colleagues' work relates to you. When you need expertise, content, or resources when you're on the go, SharePoint is a great place to start.`
* Type: `iOS store app`
* App information
  * Publisher: `Microsoft Corporation`
  * Appstore URL: `https://apps.apple.com/au/app/microsoft-sharepoint/id1091505266?uo=4`
  * Minimum operating system: `iOS 14.0`
  * Applicable device type: `iPad`, `iPhone and iPod`
  * Category: `Productivity`
  * Show this as a featured app in the Company Portal: `Yes`
  * Information URL: -
  * Privacy URL: -
  * Developer: -
  * Owner: -
  * Notes: -
  * Logo: ![Microsoft SharePoint](/assets/images/abac/microsoft-sharepoint.png)
* Scope tags: `Default`
* Assignments
  * Required: -
  * Available for enrolled devices: `grp-iOS-Dynamic`
  * Available with or without enrollment: -
  * Uninstall: -
  * License Type: `Device licensing`

### Microsoft Teams

* Name: `Microsoft Teams`
* Description: `Microsoft Teams is your hub for teamwork in Office 365. All your team conversations, files, meetings, and apps live together in a single shared workspace, and you can take it with you on your favourite mobile device. Whether you’re sprinting towards a deadline or sharing your next big idea, Teams can help you achieve more.`
* Type: `iOS store app`
* App information
  * Publisher: `Microsoft Corporation`
  * Appstore URL: `https://apps.apple.com/us/app/microsoft-teams/id1113153706?uo=4`
  * Minimum operating system: `iOS 14.0`
  * Applicable device type: `iPad`, `iPhone and iPod`
  * Category: -
  * Show this as a featured app in the Company Portal: `Yes`
  * Information URL: -
  * Privacy URL: -
  * Developer: -
  * Owner: -
  * Notes: -
  * Logo: ![Microsoft Teams](/assets/images/abac/microsoft-teams.png)
* Scope tags: `Default`
* Assignments
  * Required: `grp-iOS-Dynamic`
  * Available for enrolled devices: -
  * Available with or without enrollment: -
  * Uninstall: -
  * License Type: `Device licensing`

### Microsoft Word

* Name: `Microsoft Word`
* Description: `The trusted Word app lets you create, edit, view, and share your files with others quickly and easily. Send, view and edit Office docs attached to emails from your phone with this powerful word processing app from Microsoft.`
* Type: `iOS store app`
* App information
  * Publisher: `Microsoft Corporation`
  * Appstore URL: `https://apps.apple.com/us/app/microsoft-word/id586447913?uo=4`
  * Minimum operating system: `iOS 14.0`
  * Applicable device type: `iPad`, `iPhone and iPod`
  * Category: -
  * Show this as a featured app in the Company Portal: `Yes`
  * Information URL: -
  * Privacy URL: -
  * Developer: -
  * Owner: -
  * Notes: -
  * Logo: ![Microsoft Word](/assets/images/abac/microsoft-word.png)
* Scope tags: `Default`
* Assignments
  * Required: `grp-iOS-Dynamic`
  * Available for enrolled devices: -
  * Available with or without enrollment: -
  * Uninstall: -
  * License Type: `Device licensing`

### Power Apps

* Name: `Power Apps`
* Description: `Get Power Apps to use business apps made for your team and organization. Install Power Apps for iPhone and iPad to: Use custom Power Apps that were shared with you; Get work done from anywhere and on any device; Automate tasks and accomplish more • Access, capture, and share business data.`
* Type: `iOS store app`
* App information
  * Publisher: `Microsoft Corporation`
  * Appstore URL: `https://apps.apple.com/au/app/power-apps/id1047318566?uo=4`
  * Minimum operating system: `iOS 14.0`
  * Applicable device type: `iPad`, `iPhone and iPod`
  * Category: -
  * Show this as a featured app in the Company Portal: `Yes`
  * Information URL: -
  * Privacy URL: -
  * Developer: -
  * Owner: -
  * Notes: -
  * Logo: ![PowerApps](/assets/images/abac/microsoft-power-apps.png)
* Scope tags: `Default`
* Assignments
  * Required: `grp-iOS-Dynamic`
  * Available for enrolled devices: -
  * Available with or without enrollment: -
  * Uninstall: -
  * License Type: `Device licensing`

### Microsoft Whiteboard

* Name: `Microsoft Whiteboard`
* Description: `Microsoft Whiteboard provides a freeform intelligent canvas where teams can ideate, create and collaborate visually via the cloud. Designed for pen and touch, it lets you write or draw as smoothly as you would with ink, while automatically recognising and transforming shapes and tables as you draw. It enhances teamwork by allowing all team members to edit and comment directly on the canvas in real time, no matter where they are. And all your work stays safe in the cloud, ready to be picked back up from another location or device.`
* Type: `iOS store app`
* App information
  * Publisher: `Microsoft Corporation`
  * Appstore URL: `https://apps.apple.com/au/app/microsoft-whiteboard/id1352499399?uo=4`
  * Minimum operating system: `iOS 14.0`
  * Applicable device type: `iPad`, `iPhone and iPod`
  * Category: -
  * Show this as a featured app in the Company Portal: `Yes`
  * Information URL: -
  * Privacy URL: -
  * Developer: -
  * Owner: -
  * Notes: -
  * Logo: ![Microsoft Whiteboard](/assets/images/abac/Whiteboard.png)
* Scope tags: `Default`
* Assignments
  * Required: `grp-iOS-Dynamic`
  * Available for enrolled devices: -
  * Available with or without enrollment: -
  * Uninstall: -
  * License Type: `Device licensing`

  ### Microsoft Planner

* Name: `Microsoft Planner`
* Description: `Planner provides a simple, visual way to organize teamwork. Planner makes it easy for your team to create new plans, organize and assign tasks, share files, chat about what you’re working on, and get updates on progress.`
* Type: `iOS store app`
* App information
  * Publisher: `Microsoft Corporation`
  * Appstore URL: `https://apps.apple.com/au/app/microsoft-planner/id1219301037?uo=4`
  * Minimum operating system: `iOS 14.0`
  * Applicable device type: `iPad`, `iPhone and iPod`
  * Category: -
  * Show this as a featured app in the Company Portal: `Yes`
  * Information URL: -
  * Privacy URL: -
  * Developer: -
  * Owner: -
  * Notes: -
  * Logo: ![Microsoft Planner](/assets/images/abac/Planner.png)
* Scope tags: `Default`
* Assignments
  * Required: `grp-iOS-Dynamic`
  * Available for enrolled devices: -
  * Available with or without enrollment: -
  * Uninstall: -
  * License Type: `Device licensing`

## iOS app protection policy

`Microsoft Endpoint Manager > Apps > App protection policies`

* Name: `iOS App Protection Policy`
* Description: -
* Platform: `iOS/iPadOS`
* Apps
  * Target to apps on all device types: `Yes`
  * Device types: -
  * Public apps:
```
Adobe Acrobat Reader
Microsoft PowerApps
Microsoft Edge
Microsoft Excel
Microsoft Outlook
Microsoft PowerPoint
Microsoft Word
Microsoft Office
Microsoft OneNote
Microsoft Planner
Microsoft Power BI
Azure Information Protection
Microsoft SharePoint
Microsoft StaffHub
Microsoft OneDrive
Microsoft Teams
Microsoft Stream
Microsoft Visio Viewer
Microsoft Whiteboard
```
* Data protection
  * Data Transfer
    * Backup org data to iTunes and iCloud backups: `Block`
    * Send org data to other apps: `Policy managed apps`
    * Select apps to exempt:
      * Name 1: `Default`
      * Value 1: `tel;telprompt;skype;app-settings;calshow;itms;itmss;itms-apps;itms-appss;itms-services;`
      * Name 2: `Apple Maps`
      * Value 2: `maps`
      * Name 3: `Google Maps`
      * Value 3: `comgooglemaps`
    * Save copies of org data: `Block`
    * Allow user to save copies to selected services: `OneDrive for Business`, `SharePoint`
    * Receive data from other apps: `Policy managed apps`
    * Restrict cut, copy, and paste between other apps: `Policy managed apps`
    * Cut and copy character limit for any app: `0`
    * Third party keyboards: `Block`
  * Encryption
    * Encrypt org data: `Require`
  * Functionality
    * Sync app with native contacts app: `Block`
    * Printing org data: `Block`
    * Restrict web content transfer with other apps: `Microsoft Edge`
    * Org data notifications: `Allow`
* Access requirements
  * PIN for access: `Not required`
  * PIN type: `Numeric`
  * Select minimum PIN length: `4`
  * Touch ID instead of PIN for access (iOS 8+/iPadOS): `Block`
  * Override biometrics with PIN after timeout: `Require`
  * Timeout (minutes of inactivity): `30`
  * Face ID instead of PIN for access (iOS 11+/iPadOS): `Block`
  * PIN reset after number of days: `Yes`
  * Number of days: `365`
  * App PIN when device PIN is set: `Require`
  * Work or school account credentials for access: `Require`
  * Recheck the access requirements after (minutes of inactivity): `30`
* Conditional launch

Setting | Value | Action
--- | --- | ---
Offline grace period | 720 | Block access (minutes)
Offline grace period | 90 | Wipe data (days)
Jailbroken/rooted devices | | Block access
Min OS version | 14.0 | Block access

* Assignments
  * Included groups: `rol-Agency-Administrators`, `rol-Agency-Users`
  * Excluded groups: `grp-Windows-10-Devices`
* Scope tags: `Default`

## iOS app configuration

`Microsoft Endpoint Manager > Apps > App configuration policies`

* Name: `iOS App Configuration`
* Description: -
* Device enrollment type: `Managed apps`
* Public Apps: 
```
Microsoft Edge
Microsoft Outlook
```
* Custom apps: -
* Edge configuration settings
  * Application proxy redirection: `Enable`
  * Homepage shortcut URL: `https://www.<Agency>.gov.au/`
  * Managed bookmarks: -
  * Allowed URLs: -
  * Blocked URLs: -
  * Redirect restricted sites to personal context: `Enable`
* Outlook configuration settings
  * Focused Inbox: `Yes`
  * Require Biometrics to Access App: `No`
  * Allow user to change setting: `No`
  * Save Contacts: `No`
  * Allow user to change setting: `No`
  * External Recipients MailTip: `Not configured`
  * Block External Images: `Not configured`
  * Allow user to change setting: `Yes`
  * Default App Signature: `No`
  * Suggested Replies: `Not configured`
  * Allow user to change setting: `Yes`
  * Organize mail by thread: `Not configured`
  * Discover Feed: `Not configured`
  * Play My Emails: `Not configured`
  * Data Protection configuration
    * Org Data on Wearables: `No`
    * Calendar Notifications: `Not configured`
  * Sync contact fields to native contacts app
    * Address: `No`
    * Birthday: `No`
    * Company: `No`
    * Department: `No`
    * Email Address: `No`
    * Instant Messaging Address: `No`
    * Job Title: `No`
    * Name Prefix/Suffix: `No`
    * Nickname: `No`
    * Notes: `No`
    * Phone Home Number: `No`
    * Phone Home Fax Number: `No`
    * Phone Mobile Number: `No`
    * Phone Other Number: `No`
    * Phone Pager Number: `No`
    * Phone Work Number: `No`
    * Phone Work Fax Number: `No`
* Assignments
  * Included groups: `rol-Agency-Administrators`, `rol-Agency-Users`
  * Excluded groups: -
* Scope tags: `Default`

## Company Portal

`Microsoft Endpoint Manager > Tenant administration > Customization`

* Branding
  * Organization name: `<Agency Name>`
  * Theme color: `#512876`
  * Show in header: `Organization logo only`
  * Upload logo for theme color background: Agency supplied
  * Upload logo for theme color light background: Agency supplied
  * Upload brand image: Agency supplied
* Support information
  * Contact name: `<Agency Acronym> IT Service Desk`
  * Phone number: `<Agency Support Telephone Number>`
  * Email address: `ITServiceDesk@<Agency>.gov.au`
  * Website name: -
  * Website URL: -
  * Additional information: -
* Configuration
  * Device enrollment: `Available, with prompts`
  * Privacy messages in Company Portal for iOS/iPadOS: `Default`
  * Privacy statement URL: - 
  * Send a push notification to users when their device ownership type changes from personal to corporate (Android and iOS/iPadOS only): `No`
  * Hide remove button on corporate Windows devices: `Yes`
  * Hide reset button on corporate Windows devices: `No`
  * Hide remove button on corporate iOS/iPadOS devices: `No`
  * Hide reset button on corporate iOS/iPadOS devices: `No`
