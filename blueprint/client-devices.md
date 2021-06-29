---
layout: page
title: Client devices
---

This document covers the following topics.

Section | Description
--- | ---
Windows 10 Hardware | The Hardware Platform section includes the physical hardware, firmware drivers, and peripherals. 
Windows 10 Deployment & Management | The Deployment & Management section shows the typical deployment options for cloud & hybrid as well as the management options that can be used in the application of the SOE and security.
Windows 10 Standard Operating Environment (SOE) | The SOE section defines all the operating system components that are installed on the physical hardware. It includes the operating system and core services.
Windows 10 Security | The Windows Security section describes the configuration and methods of locking down the configuration to align with Microsoft security best practices and ACSC guidance for Windows 10 clients.
iOS | The iOS section covers enrolment and security of the mobile devices.

For each component within the document there is a brief description of the contents of the section, a commentary on the things that have been considered in determining the decisions and the design decisions themselves.

## Assumptions

* ACSC Windows 10 hardening guidelines have been broadly implemented as outlined within the SSP and SSP Annex.
* The Microsoft Endpoint Manager Console is the preferred method to manage all settings regardless of a cloud native or hybrid implementation. Although a combination of the Microsoft Endpoint Configuration Manager (MECM) Console and Group Policy Objects (GPOs) would  be able to achieve the same settings in a hybrid environment, this blueprint does not include MECM and GPOs example configurations due to the level of dissimilarities and per agency customisation in existing MECM and GPOs configurations across Commonwealth entities.
* Minimum version of MECM 1710 is required for co-management, recommended at least 1910.

## Windows 10 hardware

While most modern hardware will be capable of running Windows 10, making sure the right choice of hardware is made will ensure that the Standard Operating Environment runs efficiently. This section focuses on the technical requirements to support Windows 10 features described in this design.

The style of hardware will also need to be considered, whether a typical desktop model which will be permanently located on a desk or a laptop model to facilitate a more mobile workforce. The DTA recommends that, within operational constraints, agencies consider a laptop deployment to increase the flexibility and mobility of an agency workforce. 

### Hardware requirements

The hardware platform chosen to support the Standard Operating Environment (SOE) is key to its stability and provides the components that can be configured by the operating system and applications.

The selected processor architecture and associated firmware capability directly influence the supportability of applications and security features of an operating system. The minimum hardware requirements listed below will ensure that the system runs reliably and is supported by the vendor, Microsoft.

Hardware Requirements Design Decisions for all agencies and implementation types.

Decision Point | Design Decision | Justification
--- | --- | ---
Laptop Model | Any device that meets the below requirements and the Agency procurement processes | To meet the Agency requirements.
Desktop Model | Any device that meets the below requirements and the Agency procurement processes | To meet the Agency requirements.

Minimum hardware configuration for the Windows 10 SOE applicable to all agencies and implementation types.

Component | Description | Justification
--- | --- | ---
Architecture | x64 | Required to Support more than 4GB RAM
Processor | At least 4 logical processors, VT-x (Intel) or AMD-V CPU extensions, 2 GHz or higher with Second Level Address Translation (SLAT) support | SLAT is required to support Windows Defender Application Guard as required by ACSC hardening guide for Windows 10 1709
RAM | 8GB | To meet design specifications.
Graphics Card | DirectX 9 WDDM 1.0 | To meet design specifications. Integrated or dedicated. 
Input Device(s) | Keyboard<br>Mouse<br>Multi-touch display screen to enable Windows 10 touch screen features (optional) | Keyboard and mouse may be built into a laptop, but touch screens are optional.
Minimum HDD Space | 128GB | To meet design specifications.
Microphone | Required for speech recognition (optional) | Speech recognition is not required to be enabled but may be needed for agencies with accessibility requirements.
BIOS | Minimum UEFI 2.3.1  | Required to support Secure Boot, Windows Defender Device Guard, Windows Defender Credential Guard, Windows Defender Exploit Guard and Kernel DMA Protection
TPM | Minimum version 2.0  | Required to support Microsoft Endpoint Manager - Intune (Intune) Autopilot and MECM

### Drivers and peripherals

Drivers allow hardware and software to function within a SOE. Drivers are essentially written code that allows Windows to recognise physical components of a computer such as printers, keyboards, mouse, graphics cards and peripherals. It is critical these drivers are supported on the Operating System version and are deployed at the right time.

Drivers that are essential to the hardware platform can be deployed in the base reference image, during device deployment task sequence through MECM, via Microsoft Intune or later by Microsoft Windows Update. Drivers such as network drivers are critical during the deployment phase, whereas a microphone driver is not. The more generic a reference image, the lower the deployment and maintenance costs.

Other drivers like printer drivers can be deployed after the end user has logged onto the device using either a "Follow Me Print" or "Defined print queue list" selected by the end user.

Drivers and Peripherals Design Decisions for all agencies and implementation types.

Decision Point | Design Decision | Justification
--- | --- | ---
Driver Integration | Configured | Deployed via Microsoft Windows Update which aligns with the ACSC guidance
Approved Peripheral Devices | Configured | The SOE will enforce a list of approved peripheral devices using Intune
Unapproved Peripheral Devices | Blocked | The SOE will block installation of unapproved peripheral devices using Intune
Signed Device Driver Store | Configured | Deployed via Microsoft Windows Update which aligns with the ACSC guidance
Peripheral Drivers | Configured | Deployed via Microsoft Windows Update which aligns with the ACSC guidance
Workstation Device Drivers | Configured | Deployed via Microsoft Windows Update which aligns with the ACSC guidance
Printer Drivers | Configured | Deployed via Microsoft Windows Update which aligns with the ACSC guidance

### Firmware configuration

The firmware is the software that provides the interface between the hardware and the operating system. Firmware configuration and capabilities can directly influence the security features of an operating system.

Two important Firmware capabilities are detailed below:

* UEFI - Unified Extensible Firmware Interface (UEFI) is a replacement for the older Basic Input / Output System (BIOS) firmware interface and the Extensible Firmware Interface (EFI) 1.10 specifications
* Secure Boot - Secure Boot ensures that the device boots using only software that is trusted by the PC manufacturer. When the PC starts, the firmware checks the signature of each piece of boot software, including firmware drivers (Option ROMs) and the operating system. If the signatures are valid, the PC boots, and the firmware gives control to the operating system

Firmware that meets the UEFI 2.3.1 or newer specifications provides the following benefits:

* Faster boot and faster resume times
* Use of security features such as Secure Boot and factory encrypted drives help prevent suspicious code from running before the operating system is loaded
* Able to support 2 terabytes and greater hard drives with more than four partitions
* Some UEFI-based PCs have a Compatibility Support Module (CSM) that can emulate earlier BIOS which provide greater flexibility and compatibility for end users.

Note: Secure Boot must be disabled in order to use CSM.

* Capable of Multicast deployment whereby a PC image from a PC manufacturer can be received by multiple PCs without saturating the network or source image server.
* Support for UEFI firmware drivers, Option ROMs and applications.

UEFI 2.3.1 is a requirement for the use of Device Guard.

Secure Boot is required for the use of Credential Guard.

Firmware Configuration Design Decisions for all agencies and implementation types.

Decision Point | Design Decision | Justification
--- | --- | ---
UEFI version | At least 2.3.1 | This is minimum UEFI version required for Device Guard
Secure Boot | Enabled | Secure Boot is a requirement for the use of Windows Credential Guard and provides greater security protection for users
Secure Boot Configuration Method | Configured via Intune | Meets ACSC Windows 10 1909 hardening guidelines.

### Trusted platform module

A Trusted Platform Module (TPM) is a microchip designed to provide basic security-related functions, primarily involving encryption keys. The TPM is usually installed on the motherboard of a computer or laptop and communicates with the rest of the system using a hardware bus.

With a TPM, private portions of key pairs are kept separate from the memory controlled by the Operating System. Keys can be sealed to the TPM, and certain assurances about the state of a system—that define its "trustworthiness"—can be made before the keys are unsealed and released for use. Because the TPM uses its own internal firmware and logic circuits for processing instructions, it does not rely upon the Operating System and is not exposed to external software vulnerabilities.

A TPM of specific version number is required to support Windows 10 features such as Windows Hello and Bitlocker to help meet ACSC hardening guidelines for the OS.

Trusted Platform Module Design Decisions for all agencies and implementation types.

Decision Point | Design Decision | Justification
--- | --- | ---
TPM | Enabled in BIOS/UEFI from hardware vendor or manually configured | Required for Windows 10 and BitLocker
TPM Version | 2.0 | Meets ACSC Windows 10 1909 hardening guidelines

## Windows 10 deployment & management

The type of deployment and management methods used for the Standard Operating Environment (SOE) will vary depending on the use of either a cloud native or hybrid configuration. Cloud native will typically utilise pre-installed or offline custom images with Autopilot for the deployment method and utilise Intune as the ongoing management method.

Hybrid can benefit from enabling the MECM Co-management feature. Once enabled this allows additional deployment methods which can be utilised to ensure images remain light weight. Co-management provides a more staged approach to moving workloads into the cloud that may assist existing larger environments to complete a more gradual transition.

### Deployment

Windows 10 can be deployed via Intune or MECM, or a combination of both. The configuration of a Windows 10 deployment will depend upon which technologies are available to an agency and whether a hybrid deployment is required.

Windows 10 deployments will be based on either a deployment which is cloud native or hybrid.

* Cloud native – Image on the workstation will be modified using Autopilot and Intune. The OEM image in use should be from a trusted vendor or custom-built by the agency and provided to the OEM vendor for implementation prior to being dispatched to the agency. Alternatively, an offline image will need to be created by the agency and applied to the workstation prior to Autopilot and Intune. The offline image should be light weight consisting of the base Windows image and required base drivers only.
* Hybrid – OEM image or MECM task sequence should be used as the base for Windows 10, with Intune and Autopilot applied over the top of the image for further customisation. The OEM image in use should be from a trusted vendor or custom built by the agency and provided to the OEM vendor for implementation prior to being dispatched to the agency. Alternatively, an agency specific MECM task sequence including the Windows image and base drivers. This image will then be further customised with Intune and Autopilot.

Deployment Design Decisions for cloud native implementations

Decision Point | Design Decision | Justification
--- | --- | ---
Deployment method | Agency light weight base image with Autopilot and Intune deployments applied as during enrolment | An agency specific light weight base image provides the benefit of removing all OEM applications and firmware prior to onboarding to the agency through Intune & Autopilot.

Deployment Design Decisions for hybrid implementations

Decision Point | Design Decision | Justification
--- | --- | ---
Deployment method | Agency light weight base image with MECM deployment applied | An agency specific light weight base image provides the benefit of removing all OEM applications and firmware prior to onboarding to the agency through MECM.

### Management

Windows 10 can be managed via Intune or MECM, or a combination of both. The configuration of Windows 10 management will depend upon which technologies are available to an agency and whether a hybrid deployment is required.

Windows 10 management options will be based on either a deployment which is cloud native or hybrid. This section provides detailed information on the different configuration options for Windows 10 management.

Cloud native deployments provides the agency the immediate benefits of working with Intune and Autopilot while also integrating directly with other cloud services including Microsoft 365 and Azure AD. Using Intune will simplify the overall deployment and management of Windows 10 to a single console which is also shared with the mobile device management of iOS devices.

A hybrid deployment gives the option of co-management which enables the agency to manage Windows 10 by using both MECM and Intune. Enabling co-management within MECM allows the agency to utilise their investment in MECM and take advantage of additional cloud capabilities. This allows the agency additional flexibility to use the technology solution that works best for them and facilitates a more gradual move to cloud native as the agency can pilot test various workloads in Intune first.

Hybrid deployments can choose to enable MECM or Intune for client management depending on the cloud maturity level of the agency. It is not a requirement of agencies undertaking hybrid implementations to use MECM. This blueprint provides guidance on integration between MECM and Intune for hybrid deployments however agencies with existing infrastructure may alternatively elect to migrate device management from MECM to Intune, which will not affect cyber security postures.

Management methods that can be used to manage Windows 10 in a Microsoft 365 environment.

Microsoft 365 Implementation | MECM Implementation | Management Method for Windows 10 | Benefits
--- | --- | --- | ---
Cloud native | MECM is not possible with cloud native | Intune | No on-premises infrastructure required for management. Well suited for Azure AD joined workstations for a full cloud solution.
Hybrid with Intune management | Suitable for:<br> * No MECM or<br>* MECM with co-management enabled and Workloads set to Intune | Intune | Existing on-premises MECM infrastructure with Hybrid Azure AD joined workstation. Well suited for an agency that does not have MECM or does not want to use MECM to manage workstations.
Hybrid with MECM management | MECM with co-management enabled and Workloads set to MECM | MECM | Existing on-premises MECM infrastructure with Hybrid Azure AD joined workstations. Well suited for an agency that has a significant investment in MECM and requires a more gradual migration to Intune. Individual Workloads can be targeted and pilot-tested in Intune.

The following image displays an overview diagram of MECM Co-Management.

![MECM Co-Management diagram](/assets/images/cd-sccm-comgmt.png)

With co-management enabled, the agency can choose which workloads remain on-premises and which workloads are offloaded to Intune. The workloads are:

* Compliance Policies – Compliance policies define the rules and settings that a device must comply with to be considered compliant by conditional access policies. Compliance policies will also monitor and remediate compliance issues with devices. 
* Device Configuration – The device configuration workload includes settings that you manage for devices in your organization. Switching this workload also moves the Endpoint Protection and Resource Access workloads.
* Endpoint Protection – The Endpoint Protection workload includes the Windows Defender suite of antimalware protection features including Defender Advanced Threat Protection.
* Resource Access Policies – Resource access policies configure VPN, Wi-Fi, email, and certificate settings on devices.
* Client Apps – Use Intune to manage client apps and PowerShell scripts on co-managed Windows 10 devices. After transitioning this workload, any available apps deployed from Intune will be available in the Company Portal. Apps that are deployed from Configuration Manager are available in Software Centre.
* Office Click-to-Run Apps – This workload manages Office 365 apps on co-managed devices.
* Windows Update Policies – Windows Update for Business policies allow deferral policies for Windows 10 feature updates or quality updates for Windows 10 devices managed directly by Windows Update for Business.

With co-management disabled and no cloud integration, the agency will rely on on-premises management of the Windows 10 workstations. These could include utilising MECM task sequencing, MECM Compliance and Endpoint Protection policies, various scripting methods and Group Policy Preferences. 

There are many benefits to going cloud native or hybrid co-management utilising workloads weighted to Intune. The workstations can be managed from any internet-connected location whether that be in the office or a remote location (home, client site etc).

Management Design Decisions for cloud native implementations

Decision Point | Design Decision | Justification
--- | --- | ---
Management method | Cloud native Intune management | Cloud native implementation offers a simple and efficient implementation. Single console to manage both Windows 10 and iOS.

Management Design Decisions for hybrid implementations

Decision Point | Design Decision | Justification
--- | --- | ---
Management method | Hybrid with MECM co-management enabled. | Intune integration is required to enable features such as Conditional Access. <br>MECM co-management offers flexibility for the agency to take advantage of cloud services immediately or pilot and move individual workloads across when ready while leveraging existing configuration on-premises.
Management tool | Agency preference for Intune or MECM managing endpoints in co-management. | Each agency has a different level of investment and different maturity in MECM and cloud products. Co-management has several options to meet the unique requirements of each agency.

## Windows 10 standard operating environment

The Standard Operating Environment (SOE) consists of the customisations to Windows 10 that will vary between environments. The customisations are largely cosmetic and functional in nature to ensure that end users can operate efficiently.

### Operating system

The operating system allows software application to interface with the hardware. The operating system manages input and output device components like the mouse, keyboard, network and storage.

Windows 10 is available in several editions these include:

* Home – minimal management and deployment features and cannot be joined to either an on-premises or Azure AD domain. It is targeted from home use only
* Professional – this edition includes management and deployment features and can be joined to both an on-premises and Azure AD domain
* Enterprise – this edition has additional enterprise security features as well as the UE-V and App-V clients built in and only distributable through Microsoft's Volume Licensing Program

Servicing of Windows 10 falls into three distinct channels or rings:

* Windows Insider Program – Windows Insider Program receive feature updates immediately allowing pilot machines to evaluate early builds than the semi-annual channel. A business must opt-in for this service and install a specific Windows Insider Program for Business Preview build.
* Semi-Annual Channel – Semi-Annual Channel receives feature update releases twice per year and is designed for the broad population of general-purpose devices within an organisation.
* Long-Term Servicing Channel – Long-Term Servicing Channel (LTSC) receives releases much more gradually (expected every 2 - 3 years) and is designed for special purpose devices such as those used in Point of Sale (POS) systems or controlling factory or medical equipment, and those machines without Microsoft Office. Additionally, a number of applications are not supported on LTSC Windows devices, for example Microsoft Edge, Microsoft Store, and Microsoft Mail, amongst others.

Operating System Design Decisions for all agencies and implementation types.

Decision Point | Design Decision | Justification
--- | --- | ---
Windows 10 Edition | Windows 10 Enterprise 64-bit | Enterprise is required to support BitLocker.<br>The 64-bit edition of Windows is required to support security such as BitLocker and Windows Defender Application Control (WDAC) as specified by ACSC Windows 10 hardening guide.
Windows 10 Servicing Channels | Semi-Annual Channel | Semi-Annual Channel is the recommended ring to deploy to most enterprise clients, especially those with Office 365. 
Windows 10 Build | 1909 | At the time of writing build 1909 is the latest Semi-annual Channel release and recommended by Microsoft (refer to [technet.microsoft.com/en-us/windows/release-info.aspx](https://technet.microsoft.com/en-us/windows/release-info.aspx)). At the time of writing build 1909 remains the most recent version to have undergone formal evaluation (refer to [Common Criteria evaluations](https://www.commoncriteriaportal.org/products/)).

### Activation and licencing

License keys and activation processes are leveraged by Microsoft to ensure that the device or user is eligible to use the feature or run the product (i.e. Windows 10).

Windows 10 licensing has evolved significantly since the initial release by Microsoft. In addition to the traditional activation methods for on premises networks (KMS, MAK and AD Based Activation) it is also possible to use Windows 10 Subscription Activation. The evolution of Windows 10 activation is described below:

* Windows 10, version 1909 updates Windows 10 Subscription Activation to enable step up from Windows 10 Pro to Windows 10 Enterprise for those with a qualifying Windows 10 or Microsoft 365 subscription
* Azure Active Directory (Azure AD) available for identity management

Office 365 products require licensing to enable full functionality and support. The available activation methods are:

* Office 365 based activation - Office 365 is Microsoft's productivity solution in the cloud. Office 365 has two sets of suites: one for the small and medium business segment and one for the enterprise segment. These suites are sold across different channels and programs designed to meet each segment's needs. Products are assigned to users and then activated through the online Microsoft Office 365 licensing service

For agencies looking to alternate licensing arrangements, at a minimum, Microsoft recommends the following licensing in addition to the VSA 4 Common Cloud Commitment:

* Microsoft 365 E5 Security
* Microsoft 365 E5 Compliance

The VSA 4 Common Cloud Commitment consists of:

* Windows 10 E3
* Office 365 E3
* Enterprise Mobility and Security E3
* Productivity server licences (Exchange Server, SharePoint Server, Lync/Skype Server)
* Office Device licences

These recommendations require a minimum subscription requirement of Microsoft 365 E3, Enterprise Mobility and Security E5 and Microsoft 365 E5 Compliance with one licence required for each user.

This alternate configuration will not provide access to Azure Active Directory Premium P2 licencing meaning that the following components would need to be removed from the blueprint:

* Azure Active Directory Identity Protection
* Azure Active Directory Privileged Identity Management (PIM)

While this meets the Microsoft minimum guidance and will comply with the requirements of the ISM, the exclusion of these two features reduces the effectiveness of the security controls. Specifically, the Just-In-Time administrative access provided by PIM and the automated responses to detected suspicious activities will not be available.

Activation and Licencing Design Decisions for all agencies and implementation types.

Decision Point | Design Decision | Justification
--- | --- | ---
Licensing for Microsoft Windows 10 Enterprise 1909<br>Microsoft Office 365 E5 | Microsoft 365 E5 which includes Windows 10 E5, Office 365 E5 and EM+S E5 per user | For agencies to meet their obligations under the ISM, PSPF, and ACSC cloud guidance as they relate to PROTECTED security classification. It is recommended in this design that agencies purchase a Microsoft 365 E5 licence for each user.
Windows Activation Method | Windows 10 Subscription | All devices will meet the requirements for Subscription Activation (1909 and internet access), and this is the simplest solution to implement and manage.
Office Activation Method | Office 365 Subscription | All devices meet the requirement for Office 365 activation (internet access) and this is the simplest solution to implement and manage.

### Windows features

Windows 10 incorporates optional features that can be enabled to offer additional functionality.

When deploying a Windows 10 SOE, removing unnecessary features from the standard installation creates a simpler image to maintain. In Windows if a feature is not required or used within an environment, its removal means a faster deployment and a simpler user experience.

Windows Features Design Decisions for all agencies and implementation types.

Decision Point | Design Decision | Justification
--- | --- | ---
.Net Framework 3.5 | Enabled | This is to meet legacy .net applications that relies on .Net Framework 3.5 and earlier
.Net Framework 4.8 | Enabled | This is to allow new .net applications in the Windows 10 device.
Windows Media Features | Enabled | Support media content functionality
Windows 10 Ink | Enabled | Required to support full functionality of devices to be deployed, including Handwriting support
Print and Document Services - Windows Fax and Scan | Enabled | Support scanning functionality
Printing | Enabled | Printing enabled for office use only. Printer drivers must be supported by Windows 10
Microsoft Print to PDF | Enabled | Enables user support for Print to PDF
Microsoft XPS Doc Writer | Enabled | Enables user support for Microsoft XPS Doc Writer functionality
Remote Differential Compression Application Programming Interface (API) Support | Enabled | Required for application compatibility
Windows PowerShell | Enabled | Support administration scripting activities.

### Universal Windows Platform applications

Universal Windows Platform (UWP) applications are applications that run on Windows 10 and newer devices. Developers can build line of business Windows Store apps using standard programming languages. The Windows Runtime (WinRT) supports C#, C++, JavaScript and Visual Basic.

UWP applications cannot access user resources unless the application specifically declares a need to use those resources. This ensures a clear connection between apps and the types of resources the app has access to.

Universal Windows Platform Applications Design Decisions for all agencies and implementation types.

Decision Point | Design Decision | Justification
--- | --- | ---
Windows 10 SOE Footprint | Reduce | The Agency will reduce the size of the Windows 10 SOE footprint by configuring the settings in the next table unless the Agency has a specific requirement for these applications to be enabled.

Universal Windows Platform Application configuration applicable to all agencies and implementation types

Configuration | Value | Description
--- | --- | ---
Alarms and Clock | Removed | A versatile combination of alarm clock app, world clock, timer, and stopwatch.
Bing | Removed | Weather and News
Calculator | Provisioned | A calculator that includes standard, scientific, and programmer modes, as well as a unit converter.
Camera | Removed | The redesigned Camera is faster and simpler than ever before.
Mail and Calendar | Removed | The Mail and Calendar apps provides access to a user's email, schedule, and contacts. This access will be granted through Outlook.
Maps | Removed | Provides search functionality for places to get directions, contact numbers, business info, and reviews.
Microsoft OneDrive | OneDrive personal removed. OneDrive for Business will be used. | OneDrive is a cloud storage, file hosting service that allows a user to synchronise files and later access them from a web browser or mobile device.
Microsoft Solitaire Collection | Removed | Microsoft Solitaire Collection on Windows 10.
Microsoft Video | Removed | The Movies & TV app brings a user the latest entertainment in one simple, fast, and elegant app on Windows. 
Mixed Reality | Removed | 3D Viewer, Print 3D, Mixed Reality Portal
Mobile | Removed | Your Phone, Mobile Plans, Connect App
OfficeHub | Removed | MyOffice
OneNote | Provisioned | Microsoft OneNote Application
Paint3D | Provisioned | Microsoft Paint3d Application
People | Removed | The People app in Windows is a modern take on the flat contact lists of the past. It is built for the way people communicate today and is connected to cloud services. 
Photos | Removed | The best place to enjoy, organise, edit, and share digital memories.
Snip and Sketch | Provisioned | Capture a specific area of the screen.
MS Paint | Provisioned | Creative paint and drawing tool.
Sticky Notes | Provisioned | Sticky Notes
Store | Microsoft Store for Business will be used. | Shopfront for purchasing and downloading applications.
Microsoft Xbox | Removed | The Xbox experience on Windows 10. The Xbox app brings together friends, games, and accomplishments across Xbox One and Windows 10 devices.
Zune | Removed | Groove Music and Movies

### Microsoft Store

The Microsoft Store is an online store for applications available for Windows 8 and newer operating systems. The Microsoft Store has been designed to be used in both public and enterprise scenarios depending on whether the Microsoft Public Store or Microsoft Store for Business is configured.

The Microsoft Public Store is the central location for browsing the library of available Windows UWP Applications that can be installed on Windows 10. The Microsoft Public Store includes both free and paid applications. Applications published by Microsoft and other developers are available.

The Microsoft Store for Business allows organisations to purchase applications in larger volumes and customise which applications are available to users. Applications which are made available can either be distributed directly from the store or through a managed distribution approach. Applications which have been developed within the organisation can also be added and distributed as required.

Licensing can also be managed through the Microsoft Store for Business and administrators can reclaim and reuse application licenses.

Microsoft Store Design Decisions for all agencies and implementation types.

Decision Point | Design Decision | Justification
--- | --- | ---
Microsoft Public Store | Disabled | Meets ACSC Windows 10 1909 hardening guidelines and aligns with security and compliance requirements
Microsoft Store for Business | Disabled | Meets ACSC Windows 10 1909 hardening guidelines and aligns with security and compliance requirements

### Enterprise applications

Enterprise applications provide organisations and end users the functionality they require to perform day to day activities.

Applications can be delivered to the user's desktop by one of the following methods:

* Intune – Ideally suited for Windows 10 only deployments. The only option for delivering application to iOS clients.
* MECM – Ideally suited where an agency has a large existing investment of packaged applications. The only option for delivering application to servers. Only option for delivering virtual applications. 

Self-Service applications are requested by users directly. They can be delivered via Software Centre which is installed as part of the MECM client or Company Portal which is available as part of Intune and the Microsoft 365 tenant. As of MECM version 1802 "user-available" apps now appear in Software Centre under the applications tab where they were previously available in the Application Catalogue

Packaging methodology should be inherited from existing Agency procedures as each application has unique requirements. It is possible to repacked existing applications into an msix format which is compatible with both Intune and MECM delivery.

Enterprise Applications Design Decisions for cloud native implementations

Decision Point | Design Decision | Justification
--- | --- | ---
Application Delivery Technologies | Deployed via Intune | Applications deployed via Intune and will be installed during the build deployment.
Self Service | Company Portal | Allow users to install the apps needed while ensuring the SOE remains as light weight as possible.
Packaging Methodology | Agency preference | Applications will be packaged according to agency internal packaging standards and delivered via Intune

Enterprise Applications Design Decisions for hybrid implementations

Decision Point | Design Decision | Justification
--- | --- | ---
Application Delivery Technologies | Deployed via Intune or MECM | Applications deployed via Intune and will be installed during the build deployment.<br>MECM can continue to be used for existing applications, however consideration should be made to migrate these to Intune in future.
Self Service | Company Portal or MECM Software Center | Allow users to request install of specific apps while ensuring the SOE remains as light weight as possible.
Packaging Methodology | Agency preference | Applications will be packaged according to agency internal packaging standards and delivered via MECM or Intune.

5.7 Power Management

The power settings in Windows 10 can be fully managed by Intune, MECM, or Group Policy. Individual settings can be enforced or set as defaults that can then be changed by the user as desired.

Users can adjust power and performance options via the system tray power slider icon to:

* Better Battery / Recommended - Better Battery / Recommended provides extended battery life than the default settings on previous versions of Windows
* Better Performance - Better Performance is the default slider mode that slightly favours performance over battery life and is appropriate for users who want to trade-off power for better performance of applications
* Best Performance - Best Performance prioritizes performance over battery life

Power Management Design Decisions for all agencies and implementation types.

Decision Point | Design Decision | Justification
--- | --- | ---
Power Management technology | Intune, MECM or Group Policy | Agency preference of technology to configure power options. <br>If using MECM or GPO, consideration should be made to migrate these to Intune in future.
Default Power Option Battery | Balanced | Default setting, no requirement to change has been identified
Default Power Option Powered | Better Performance | Default setting, no requirement to change has been identified
Allow standby states when sleeping (on battery)  | Disabled | Meets ACSC Windows 10 1909 hardening guidelines
Allow standby states when sleeping (plugged in) | Disabled | Meets ACSC Windows 10 1909 hardening guidelines
Require a password when a computer wake (on battery) | Enabled | Meets ACSC Windows 10 1909 hardening guidelines
Require a password when a computer wakes (plugged in) | Enabled | Meets ACSC Windows 10 1909 hardening guidelines
Specify system hibernate timeout (on battery) | Enabled<br>System Hibernate Timeout (seconds): 0 | Meets ACSC Windows 10 1909 hardening guidelines
Specify system hibernate timeout (plugged in) | Enabled<br>System Hibernate Timeout (seconds): 0 | Meets ACSC Windows 10 1909 hardening guidelines
Specify system sleep timeout (on battery) | Enabled<br>System Sleep Timeout (seconds): 0 | Meets ACSC Windows 10 1909 hardening guidelines
Specify system sleep timeout (plugged in) | Enabled<br>System Sleep Timeout (seconds): 0 | Meets ACSC Windows 10 1909 hardening guidelines
Specify the unattended sleep timeout (on battery) | Enabled<br>Unattended Sleep Timeout (seconds): 0 | Meets ACSC Windows 10 1909 hardening guidelines
Specify the unattended sleep timeout (plugged in) | Enabled<br>Unattended Sleep Timeout (seconds): 0 | Meets ACSC Windows 10 1909 hardening guidelines
Turned off hybrid sleep (on battery) | Enabled | Meets ACSC Windows 10 1909 hardening guidelines
Turned off hybrid sleep (plugged in) | Enabled | Meets ACSC Windows 10 1909 hardening guidelines
Show hibernate in the power options menu | Disabled | Meets ACSC Windows 10 1909 hardening guidelines
Show sleep in the power options menu | Disabled | Meets ACSC Windows 10 1909 hardening guidelines

### Windows Search and Cortana

The Windows Search feature of Windows 10 provides indexing capability of the operating and file system allowing rapid searching for content stored on an attached hard disk. Once indexed, a file can be searched using either the file name or the content contained within the file.

Cortana is a voice search capability of Windows 10. Cortana's features include being able to set reminders, recognise natural voice without the user having to input a predefined series of commands, and answer questions using information from Bing (like current weather and traffic conditions, sports scores, and biographies).

Cortana can be used to perform tasks like setting a reminder, asking a question, or launching the app.

Configuration of Cortana features can be managed by GPO or modern management (such as Microsoft Intune).

Windows Search is text-based and is built into the local operating system.

Windows Search and Cortana Design Decisions for all agencies and implementation types.

Decision Point | Design Decision | Justification
--- | --- | ---
Cortana  | Disabled | As per the ACSC hardening guidelines the Cortana feature will be disabled to comply with security requirements.
Windows Search  | Enabled and configured for local content | Windows search will be limited to local items only to prevent data leakage.

### Internet browser

The internet browser is a software application used for accessing web pages. This browser may be built into the operating system or installed later.

Microsoft Edge Chromium is a web browser for Windows 10. It has been developed to modern standards and provides greater performance, security, and reliability. It also provides additional features such as Web Note and Reading View.

Alternate browsers may also be deployed to support specific business needs or requirements.

Internet Browser Design Decisions for all agencies and implementation types.

Decision Point | Design Decision | Justification
--- | --- | ---
Default Browser | Microsoft Edge Chromium – Stable edition | To ensure compatibility and performance with modern web pages. 
Alternate Browsers | Internet Explorer 11 | To ensure compatibility with legacy web pages. 

### Tablet mode

Tablet Mode is an adaptive user experience feature in Windows 10 that optimises the look and behaviour of applications and the Windows shell for the physical form factor and end-user's usage preferences.

Tablet Mode is a feature that switches a device experience from tablet mode to desktop mode and back. The primary way for an end-user to enter and exit "tablet mode" is manually through the Action Centre. In addition, Original Equipment Manufacturers (OEMs) can report hardware transitions (for example, transformation of 2-in-1 device from clamshell to tablet and vice versa), enabling automatic switching between the two modes.

Tablet Mode Design Decisions for all agencies and implementation types.

Decision Point | Design Decision | Justification
--- | --- | ---
Tablet Mode | Enabled by default on devices that support it. | To provide the option to manipulate Tablet Mode behaviour through the Action Centre on supported devices.

### Fast user switching

Fast User Switching allows more than one concurrent connection to a Windows 10 device, however only one session can be active at a time.

Fast User Switching creates potential security risks around session-jacking and credential breaches. If one user reboots or shuts down the computer while another user is logged on, the other user may lose work as applications may not automatically save documents.

Fast User Switching Design Decisions for all agencies and implementation types.

Decision Point | Design Decision | Justification
--- | --- | ---
Fast User Switching | Disabled | Meets ACSC Windows 10 1909 hardening guidelines

### Corporate branding

Organisational branding enables a consistent corporate user experience.

Windows 10 permits the image displayed at the lock screen, logon screen, and desktop wallpaper to be customised and support various resolutions. The appropriate resolution is selected based on an image file name. Windows will automatically select the appropriate image based on the current screen resolution. If a file matching the screen resolution cannot be found, a default image file is used, and the picture stretched to fit the screen.

Custom themes can be deployed to workstations either enforcing the theme or allowing a user to customise it after the initial SOE deployment. Each client agency would be required to provide information necessary to customise the branding.

Although a system is capable of being assessed as PROTECTED, banners and backgrounds should not be set to PROTECTED in the SOE or desktop background until an IRAP assessment has been completed.

Corporate Branding Design Decisions for all agencies and implementation types.

Decision Point | Design Decision | Justification
--- | --- | ---
Lock Screen | Custom<br>Corporate wallpaper | Customised to the Agency
Logon Screen | Custom<br>Corporate wallpaper | Customised to the Agency
Wallpaper | Custom<br>Corporate image | Customised to the Agency
Account Picture  | User account picture must correspond to the user security pass | Customised to the Agency
Theme | Custom | Customised to the Agency to set default wallpaper
Theme Colour | Default | Default theme is suitable for purposes
Windows Colour | Default  | Default theme is suitable for purposes
Corporate Account Picture | Default | The default Windows 10 user icon will be displayed as the user account picture
User Ability to Change Account Picture | Disabled | Disabled to provide a consistent configuration for all agency users.

### System properties

The System Properties window can be customised in several ways. Within the System Properties window, the Manufacturer and Model values can be displayed.

Support information can also be populated which includes:

* Support phone number
* Support hours
* Support website

A custom OEM logo can also be displayed below the Windows logo.

The system Computer Description can also be used to display the build date, time and SOE version.

The Manufacturer value is used in the title string displayed in the support section, being "Manufacturer support". If the actual computer manufacturer were to be populated, then the support section heading would be "HP support", for example, which would be misleading for users. Setting the Manufacturer value to "Agency" would set the support section heading to "Agency support".

System Properties Design Decisions for all agencies and implementation types.

Decision Point | Design Decision | Justification
--- | --- | ---
Company Name | Not Configured | Not required to support deployments
OEM Logo | Configured – Australian Government crest | To identify that the equipment is under Australian Government jurisdiction
Manufacturer Value | Configured – the Agency Name | To identify the Agency as the device owner
Model Value | Configured – Asset Number | To identify the device via asset label
Support Hours Value | Configured – Support hours of internal ICT support | To simplify Agency desktop support
Support Phone Value | Configured | To simplify Agency desktop support
Support URL Value | Configured | To simplify Agency desktop support
Computer Description | Configured – Asset type and model | To simplify Agency desktop support

5.14 Start Menu

The Windows 10 Start Menu contains tiles that represent different programs that a user can launch by clicking on the tile. 

The default Start Menu layout can be configured for all users that use the device. Applications can be pinned to the start menu by administrators to ensure a consistent user experience across the environment. There are three levels of enforcement which are possible within the start menu:

* Full enforcement – The start menu is deployed centrally and cannot be altered by users in any way. Applications dictated by administrators are the only items which are present in the start menu. 
* Partial enforcement – The start menu is deployed centrally and can be altered by users. An application group is deployed for corporate shortcuts and users can then add applications to the start menu in a separate application group. The centrally deployed application group cannot be altered.
* No enforcement – No start menu is defined centrally, and users can make any modification to the start menu.

Start Menu Design Decisions for all agencies and implementation types.

Decision Point | Design Decision | Justification
--- | --- | ---
Start Menu Layout Enforced | Partially | Corporate application group will be enforced, with the rest of the application group able to be customised by users. End-users can customise the Start Menu to suit specific needs, including the ability to resize, reorganise and choose whether to list most recent shortcuts

### Screen saver

The screen saver was originally designed prevent burn-in on Cathode Ray Tube (CRT) and plasma screens. Modern usage of the screen saver allows the operating system to detect a period of inactivity and lock or blank the screen reducing power usage.

Screensavers should be applied at regular intervals in instances that a user may walk away from their endpoint and leave their workstation unlocked. Screensavers can also be used in some circumstances as a communication mechanism to users. Microsoft does not recommend enabling a screen saver on devices. Instead, Microsoft recommends using automatic power plans to dim or turn off the screen as this can help reduce system power consumption. 

Configuration can be applied to restrict the end-user ability to configure or change the screen saver settings.

Screen Saver Design Decisions for all agencies and implementation types.

Decision Point | Design Decision | Justification
--- | --- | ---
Screen Saver | Disabled | Not required, the device will be configured to sleep after 15 minutes
Machine Inactivity | Configured – 900 seconds | This is defined as per ACSC 1909 Windows 10 Hardening guide
Users Can Configure the Screen Saver | No | Disable the ability for users to configure the screen saver for all Windows 10 SOE devices
Require Password on Wake | Configured | Users will be required to enter their password on machine wake up. This is defined as per ACSC 1909 Windows 10 Hardening guide

### Profiles, personalization, and folder redirection

Profiles are a collection of data and settings for each user of a Windows computer. Examples of data captured as part of a user's profile are user settings, desktop shortcuts, and application settings.

Profile configuration values are specific to a single user and are stored in a single folder known as the 'User Profile'. These configuration parameters (themes, window colour, wallpapers, and application settings) determine the look and feel of the operating environment for a specific user.

Microsoft includes several standard options for user profiles. Alternatively, technologies such as Microsoft UE-V or FSLogix can be used to address user profile and personalisation requirements. If no user profile is configured, a desktop local profile is used, which does not backup options but performs well.

Microsoft provide the following profile management solutions:

* Local Profiles – Local user profiles are stored on the workstation. When the user logs on for the first time, a local user profile is created for the user and stored by default in `C:\Users\%USERNAME%`. Whenever a user logs on to the workstation, the user's local user profile is loaded. When the user logs off the workstation, any configuration changes made to the user's profile are saved in the user's profile
* Mandatory Profiles – Mandatory profiles are a profile that does not save profile changes and are enforced at each logon
* Roaming Profiles – Roaming user profiles are stored in a central location on the network, which is generally a shared folder on a server. When the user logs on to a workstation, the roaming user profile is downloaded from the network location and loaded onto the workstation. When the user logs off the workstation, any profile changes are saved to the network share. In addition to maintaining a copy of the roaming profile on the network share, Windows also keeps a locally cached copy of the roaming profile on each workstation that the user logs on. 

Windows 10 provides two main roaming profile technologies in User Experience Virtualization (UE-V) and FSLogix. FSLogix is now the preferred Roaming Profile option as it provides a consistently higher performance than UE-V and can provide a cloud-based roaming profile when configured with suitable Azure cloud storage blobs.

Profiles, Personalization, and Folder Redirection Design Decisions for all agencies and implementation types.

Decision Point | Design Decision | Justification
--- | --- | ---
Folder Redirection | Redirect Windows Known Folders | Users can continue using the folders they are familiar with. Files are automatically backed up to the users OneDrive folder in the cloud

Table 73 Additional Profiles, Personalization, and Folder Redirection Design Decisions for cloud native implementations

Decision Point | Design Decision | Justification
--- | --- | ---
Profile Type | Local Profiles | Local profiles will be configured to support end-user assigned devices. This configuration assumes that users will not share devices and do not require profile backups.<br>Enterprise State Roaming will be enabled for key backup of key user settings.
Known Folder Redirection Configuration  | Configured as listed below in Table 75 | To enable user personalisation and provide backup of essential user data.

Table 74 Additional Profiles, Personalization, and Folder Redirection Design Decisions for hybrid implementations

Decision Point | Design Decision | Justification
--- | --- | ---
Profile Type | Roaming Profiles | Roaming profiles will be configured to support end-user assigned devices. A roaming profile management product is recommended by the vendor, Microsoft.<br>This configuration assumes that users can share devices.
Known Folder Redirection Configuration  | Configured as listed below in Table 76 | To enable user personalisation and backup of essential user data

Table 75 Known Folder Redirection Configuration applicable to agencies leveraging a cloud native implementation

Folder | Path
--- | ---
AppData | Not Configured
Contacts | Not Configured
Desktop | `C:\Users\%username%\OneDrive\Desktop`
Documents | `C:\Users\%username%\OneDrive\Documents`
Downloads | Not Configured
Favourites | Not Configured
Links | Not Configured
Searches | Not Configured
Music | Not Configured
Pictures | `C:\Users\%username%\OneDrive\Pictures`
Videos | Not Configured

Known Folder Redirection Configuration applicable to agencies leveraging a hybrid implementation

Folder | Path
--- | ---
AppData | Not Configured
Contacts | Not Configured
Desktop | `\\\\server\share\Users\%username%\OneDrive\Desktop`
Documents | `\\\\server\share\Users\%username%\OneDrive\Documents`
Downloads | Not Configured
Favourites | Not Configured
Links | Not Configured
Searches | Not Configured
Music | Not Configured
Pictures | `\\\\server\share\Users\%username%\OneDrive\Pictures`
Videos | Not Configured

### Operational support

Windows 10 and supporting management tools offer various SOE support features to allow support personnel to access a machine remotely or provide users with the option to perform automated repairs.

The following support components are available to support Windows 10:

* Intune – Intune can remotely wipe, reset and remove a device from Azure AD. These functions are controlled by role-based administration, permitting only certain administrators to control these settings.
* Windows Remote Management (WinRM) – WinRM is the Microsoft implementation of the WS-Management Protocol, a standard Simple Object Access Protocol (SOAP) based, firewall-friendly protocol that allows hardware and Operating Systems from different vendors to interoperate
* WS-Management protocol - The WS-Management protocol specification provides a common way for systems to access and exchange management information across an IT infrastructure. WinRM and Intelligent Platform Management Interface (IPMI), along with the Event Collector are components of the Windows Hardware Management features
* Windows Remote Assistance – Windows Remote Assistance in Windows 10 uses the Remote Desktop Protocol (RDP) protocol to provide a remote desktop connection that is interactive between the locally logged on user and a remote user
* Remote Desktop – Remote Desktop enables a user to remotely logon interactively to a workstation from another computer with a supported Remote Desktop client
* Remote Control – Remote control options are limited to the following:
  * TeamViewer which is a paid service that fully integrates in Intune
  * Remote Control within MECM is this is configured in a hybrid deployment
  * Microsoft Teams assuming the user can share the desktop

Operational Support Design Decisions for all agencies and implementation types.

Decision Point | Design Decision | Justification
--- | --- | ---
Intune | Enabled | Intune management functions cannot be disabled when a device is enrolled in Intune and Azure AD.
WinRM | Enabled | To meet operating support requirements for the Agency.<br>Consideration should be made to harden the use of WinRM in the agency environment to increase the security of Windows 10 endpoints.
Windows Remote Assistance | Disabled | Aligns with Windows 10 1909 hardening guide
Remote Desktop | Enabled | To meet operating support requirements for the Agency.<br>Access granted via AD Groups
Remote Desktop Client | Enabled | Remote Desktop will be enabled for Windows 10 devices
Remote Control via Teams | Enabled | Users can share desktop within the Microsoft Teams application

### Windows update and patching

Many updates released for operating systems and application contain bug fixes and security updates. Vulnerabilities can be exploited by malicious code or hackers and need to be patched as soon as possible.

A risk assessment of a vulnerability is essential in determining the timeframe for applying patches. There are many different sources and indicators that will help with this assessment, for example if the vendor releases a patch outside of their normal patching cycle and its marked as a critical update then it is worth immediate investigation and deployment to see how it could affect an organisation.

It is vital to have a robust and reliable patch management solution based on industry best practices.

For Microsoft Windows environments the primary patching technologies are:

* Windows Server Update Service – WSUS enables administrators to deploy the most recent Microsoft updates. A WSUS server connects directly to Microsoft Update or an "upstream" WSUS server. This allows administrators to control what updates are applied and when, rather than having every computer on the network going to the Internet and installing every available update immediately
* Microsoft Endpoint Configuration Manager (MECM) – MECM integrates with a WSUS server to deliver patch management. WSUS obtains updates from the internet and MECM is used to approve and deploy the updates. Using MECM to deploy software updates allows for more control over many aspects of the process such as targeting, maintenance windows, scheduling, and reporting
* Microsoft Intune – Windows Update for Business provides management policies for several types of updates to Windows 10 devices
  * Feature updates: feature updates contain security and quality revisions, significant feature additions and changes; they are released semi-annually in March and September
  * Quality updates: these are traditional operating system updates, typically released the second Tuesday of each month (though they can be released at any time). These include security, critical, and driver updates. Windows Update for Business also treats non-Windows updates (such as those for Microsoft Office or Visual Studio) as quality updates. These non-Windows Updates are known as "Microsoft updates" and can configure devices to receive or not receive such updates along with their Windows updates
  * Driver updates: these are non-Microsoft drivers that are applicable to the devices. Driver updates can be turned off by using Windows Update for Business policies
  * Microsoft product updates: these are updates for other Microsoft products, such as Office. These updates can be enabled or disabled by using Windows Update for Business policy

Update rings are policies that are assigned to groups of devices. Intune can define update rings that specify how and when service updates are deployed to Windows 10 devices. By using update rings, it is possible to create an update strategy that mirrors business needs

To deploy patches to endpoints as quickly as possible, client-side settings should not restrict or delay the installation of patches where it does not interfere with critical operation or cause loss of data due to unexpected reboots.

Windows Update and Patching Design Decisions for all agencies and implementation types.

Decision Point | Design Decision | Justification
--- | --- | ---
Patching technology | Agency preference of Intune, MECM or WSUS  | For cloud native deployments, Intune is the only option available.<br><br>For hybrid deployments, all three options are available for implementation. Intune provides a simpler approach to patching whilst MECM and WSUS provide more granular control of patch deployment.
Patching Testing Method | Pilot and Production  | Allows early deployment and test of Windows updates to selected users prior to the full release of updates to the remaining users.<br><br>The Pilot group will be a select number of users who will actively provide feedback on updates before rollout to all users
Feature Updates | Enabled | Meets ACSC hardening guidance for Windows 10 1909
Quality Updates | Enabled | Meets ACSC hardening guidance for Windows 10 1909
Driver Updates | Enabled | Meets ACSC hardening guidance for Windows 10 1909
Microsoft Product Updates | Enabled | Meets ACSC hardening guidance for Windows 10 1909
Patching Frequency | Existing Agency patch scheduling based on patch criticality | The Agencies existing patch schedule should reflect:<br><br>* **Extreme risk**: within 2 days<br>* **High risk or lower**: within 7-14 days<br><br>Meets ACSC assessing security vulnerabilities and applying patches assessment guidance

### Networking

Windows 10 contains networking technologies built into the operating system. These features allow Windows to communicate with other networked devices including those on the Internet.

IPv6 can be enabled or disabled within Windows 10 depending on the network to which the device will be connected. IPv6 should be disabled unless it is exclusively used throughout the network. 

Windows 10 provides support for several wireless networking technologies that allow devices to connect to a wireless network. The two most popular technologies supported in Windows currently are Wi-Fi and Mobile Broadband networking.

802.1x ensures that only appropriate users or devices can connect to a protected network and that data is secure at the radio transmission level. The Single Sign-On (SSO) feature executes Layer 2 network authentication at the appropriate time given the network security configuration, integrating with the user's Windows logon experience.

Networking Design Decisions for all agencies and implementation types.

Decision Point | Design Decision | Justification
--- | --- | ---
IPv6 | Disabled | As per ACSC hardening guidance for Windows 10 1909 IPv6 will be disabled.<br>Exceptions to this rule are if IPv6 is wholly deployed within an Agency network with no IPv4.
Wireless | Enabled | Where applicable, wireless capable devices will have Wi-Fi enabled to allow use case of mobile working.
Wireless Configuration | Refer to Table 86 for wireless configuration recommendations. | As per ACSC hardening guidance for Windows 10 1909 and security compliance requirements. 
Broadband | Not Configured | If Agency devices have Subscriber Identity Module (SIM) capability this can be enabled without affecting an agencies cyber security posture.
Network Bridging | Disabled | As per ACSC hardening guidance for Windows 10 1909 and security and compliance requirements.
Wake on LAN (WoL) | Configured via existing MECM solution if in use | Wake on LAN configured to allow existing MECM management tasks to operate on computers regardless of power status.

Table 86 Wireless Configuration applicable to all agencies and implementation types

Configuration | Value | Description
--- | --- | ---
Connect to Wireless Hotspots | Enabled | Allows users to connect to wireless hotspots when working remotely
Automatically Connect to Suggested Open Hotspots | Disabled | Meets ACSC hardening guidance for Windows 10 1909 and aligns with security and compliance requirements
Prohibit installation and configuration of Network Bridge | Enabled | Meets ACSC hardening guidance for Windows 10 1909 and aligns with security and compliance requirements
Single Sign On 802.1x | Enabled | Meets ACSC hardening guidance for Windows 10 1909 and aligns with security and compliance requirements
Wireless Profile Configuration | Configured | Will be configured depending on the Agency requirements

### Delivery optimisation

Delivery optimisation makes use of configurable peer-to-peer or caching technologies to decrease the internet bandwidth consumed by patches and updates.

Within an organisation application deployments and patch updates can occur daily to keep an organisation secure and provide the end users the capabilities to perform their work. These application deployments and patch updates can consume high levels of bandwidth, increasing the cost to an organisation. To allow organisations to decrease bandwidth use and cost organisations can implement Delivery Optimisation solutions within their main and remote offices. Delivery Optimisation is essentially a peer-to-peer client service that allows end clients to source installation packages and updates from other clients/servers within the network as opposed to always obtaining them from the source servers or the internet. There are two types of configuration options for Delivery Optimisation:

The following options apply to cloud native and Hybrid (configured in Intune):

* HTTP only, no peering:  Delivery Optimisation cloud service used to provide metadata only. Content to be retrieved from the downloads original source with peer-to-peer disabled. 
* HTTP blended with peering behind same NAT: Peer-to-peer sharing within the same network. Delivery Optimisation cloud service locates peers that connect to the internet using the same public IP and attempts to connect peers using their private subnet IP. 
* HTTP blended with peering across private group: Peers are automatically grouped by using the Active Directory Domain Services site or domain they reside in. Custom groups can also be used with peers being either manually or dynamically added to the group. Peers within the same group are connected.
* HTTP blended with internet peering: Enabled Internet peer sources for Delivery Optimisation
* Simple download mode with no peering:  Delivery Optimisation cloud services are disabled. This mode is automatically used with the Delivery Optimisation cloud service is not accessible or when the content file size is less than 10 MB.
* Bypass mode:  Microsoft BITS will be used instead and should only be used if WSUS is in use and the preference is for BranchCache. 

The following options apply to Hybrid (configured in MECM):

* BranchCache: Allows systems within the same subnet and separated from a content source to share downloaded content locally instead of traversing a latent network link back to the MECM content source. 

BranchCache provides two modes of operation being:

* Distributed Cache Mode - Content cache at a remote office is distributed among client systems
* Hosted Cache Mode - Content cache at a remote office is hosted on one or more hosted cache servers
* Peer Cache: Is a built-in Configuration Manager solution that allows the clients to share content with other clients directly from their local cache providing Delivery Optimisation. 
* Microsoft Connected Cache: With MECM version 1910 a MECM distribution point can be configured as a Microsoft Connected Cache server allowing it to act as an on-demand cache for content downloaded by Delivery Optimisation. 

Delivery Optimisation Design Decisions for cloud native implementations

Decision Point | Design Decision | Justification
--- | --- | ---
Delivery Optimisation Method / Feature - single subnet per physical office | HTTP blended with peering behind same NAT | Good for single subnet per office as content will be peered per office only. This will not saturate WAN connections
Delivery Optimisation Method / Feature - single subnet spanning multiple physical offices | HTTP blended with peering across private group | Provides the ability to group workstations so only the groupings can peer content

Delivery Optimisation Design Decisions for hybrid implementations where MECM is the update delivery mechanism

Decision Point | Design Decision | Justification
--- | --- | ---
Delivery Optimisation Method / Feature | BranchCache | Existing MECM solution exists with BranchCache Delivery Optimisation feature enabled
Mode of operation | Distributed Cache Mode | Existing MECM solution exists with BranchCache configured in Distribute Cache Mode

### Microsoft Office edition

Microsoft Office is available in two release cycles and within those release cycles there are multiple editions.

Microsoft Office has two release cycles:

* Office 365 – Office 365 combines the Microsoft Office desktop suite with cloud-based versions of Microsoft's communications and collaboration services—including Microsoft Exchange Online, Microsoft SharePoint Online, Office Online, and Microsoft Teams. Office 365 is upgraded with new features on a regular basis
* Traditional Office – Traditional Office is sold as a one-time purchase and provides Office applications for a single computer. There are no upgrade options which means to upgrade to the next major release, another copy of Office will have to be procured. Traditional Office is not upgraded with new features for the life of the release

Within these release cycles, you can choose the architecture of 32-bit or 64-bit. Microsoft Office provides 32-bit or 64-bit version to be installed on Windows 10 devices. Microsoft 64-bit version of Office will be automatically chosen to be installed, unless 32-bit version is installed. The 64-bit version of Office provides ability working with larger datasets and files. However, 64-bit version of Office does not support legacy macros, and COM Add-In.

Pro Plus is an optional configuration which installed Office 365 locally on a client machine 

Microsoft Office Edition Design Decisions for all agencies and implementation types.

Decision Point | Design Decision | Justification
--- | --- | ---
Microsoft Office Version | Office 365 Pro Plus 64-bit | Aligns with modernisation vision and provides access to the latest and most updated features.

### Office features

The Office 365 ProPlus features include the application set that will be provided to the users.

The Microsoft Office feature section includes the details of the following components:

* Microsoft Access
* Microsoft Excel
* Microsoft Teams
* Microsoft Office OneNote
* Microsoft Outlook
* Microsoft Publisher
* Microsoft PowerPoint
* Microsoft Word
* Microsoft OneDrive Desktop

Office Features Design Decisions for all agencies and implementation types.

Decision Point | Design Decision | Justification
--- | --- | ---
Microsoft Office Word | Enabled | Required to enable user productivity
Microsoft Office Excel | Enabled | Required to enable user productivity
Microsoft Office Outlook | Enabled | Required to enable user productivity
Microsoft Office PowerPoint | Enabled | Required to enable user productivity
Microsoft Office OneNote | Enabled | Required to enable user productivity
Microsoft Office Publisher | Enabled | Required to enable user productivity
Microsoft Teams | Enabled  | Required to enable user productivity
Microsoft Office Access | Disabled | Not installed by default

### Office language pack

Language packs add additional display, help, and proofing tools to Microsoft Office. Multiple language packs can be installed to support specific user requirements.

If additional language packs are installed it is also likely that keyboards other than US will be required.

Office Language Pack Design Decisions for all agencies and implementation types.

Decision Point | Design Decision | Justification
--- | --- | ---
Default Language | English (UK) – AU Default | Required to support the Microsoft Office deployment and allow user productivity.<br>English (US) language pack is removed from the SOE as part of the English (UK) install. English (UK) contains the AU region language pack which is then set as default
Additional Language | Not Configured | Additional languages will be installed if required to meet agency requirements

### OneDrive for Business

OneDrive for Business provides a robust cloud storage platform for government agencies.

This OneDrive for Business section considers the client component only. The configuration of the server component of OneDrive for Business is contained in the Office 365 Solution Overview document.

OneDrive enables the secure sharing of files inside and outside an organisation, and on any compatible device. Compliance settings allow administrators to control the level of security within the application.

OneDrive also facilitates collaboration between users. Co-authoring is available via Office web apps, Office mobile apps, and Office desktop apps, helping users maintain a single working version of any file and edit concurrently.

The OneDrive for Business client has access to two distinct primary rings and an additional preview ring:

* Production Ring – The Production ring provides new features and improvements as soon as they are released by Microsoft
* Enterprise Ring – The enterprise ring rolls out changes after validated in the Production ring, reducing the risk of issues. This ring enables administrators to deploy updates from an internal network location and control the timing of the deployment (within a 60-day window). This is the recommended update ring for most large scale or high-risk organisations
* Insiders Ring – Insider ring users will receive builds that let them preview new features coming to OneDrive

The Windows Known Folder feature of OneDrive for Business enables administrators to easily move files in a users' Desktop, Documents, and Pictures folders to OneDrive.

OneDrive Files On-Demand enables users to view, search for, and interact with files stored in OneDrive from within File Explorer without downloading them all to the local device. The feature delivers a unified look and feel for both OneDrive and local files whilst saving on space normally taken up on the local hard drive.

OneDrive allows users to work on content that is stored online in the cloud as well as store and synchronise content locally through the OneDrive for Business client. Synchronising this content between the client device and OneDrive storage can consume a considerable amount of bandwidth especially if the internet connection is slow. The OneDrive for Business client provides the capability to shape upload and download rates for synchronising content hence reducing the impact on the network. If the internet connection is too slow the end user can continue to work on the content locally and the content will be automatically synchronised by OneDrive for Business client when the internet connection speed improves.

OneDrive for Business Design Decisions for all agencies and implementation types.

Decision Point | Design Decision | Justification
--- | --- | ---
OneDrive for Business | Enabled and silently configured | OneDrive will be configured without user intervention on sign in
Sync Client Update Ring | Enterprise | As per Microsoft recommendations for large environments
OneDrive Personal Account | Disabled | Personal accounts will be disabled. OneDrive for Business accounts will be used
Default Location | `%userprofile%` | Default OneDrive folder location is suitable for the Windows 10 SOE
Allow Changing Default Location | Disabled | As per Microsoft recommendation for shared devices users will be prevented from changing the default OneDrive folder location
Files On-Demand | Enabled | Files On-Demand will be configured to save storage space on users' computers and minimize the network impact of sync
Backup - Sync Windows Known Folders | Enabled | Syncing Windows known folders to OneDrive for Business will be configured for the Windows 10 SOE. This will enable the users Documents, Pictures and Desktop folders to be saved in OneDrive automatically
Network settings – Upload | Do not limit | Allow dynamic network configuration to provide best performance
Network settings – Download | Do not limit | Allow dynamic network configuration to provide best performance
File Collaboration Policy | Enabled | File collaboration will be enabled to allow users to work collaboratively and increase productivity
Sync Conflict Policy | Let me choose to merge changes or keep copies | The OneDrive sync conflict policy will be configured to allow the user to choose

## Windows 10 security

Security settings are applied to the Standard Operating Environment largely to slow down and prevent malicious adversaries and payloads from causing harm to an Agency. The security settings should not prevent legitimate users from conducting work and should provide them with the correct amount of access to the environment to allow them to operate without impeding the work.

### Microsoft Defender

Microsoft delivers several threat protection and mitigation capabilities in Windows 10 Enterprise devices delivered through Microsoft Defender.

These capabilities do not require additional agents and are manageable via Intune Endpoint Protection Profiles.

The following details the Microsoft Defender capabilities:

* Microsoft Defender Antivirus – Provides anti-malware and spyware protection including always-on scanning, dedicated protection updates and cloud-delivered protection. Integration with Internet Explorer and Microsoft Edge browsers enable real time scanning of files as they are downloaded to detect malicious software
* Microsoft Defender Exploit Guard – Provides Host-based Intrusion Protection System (HIPS) capabilities and replaces the Microsoft Enhanced Mitigation Experience Toolkit (EMET)
* Microsoft Defender Application Guard – Provides hardware isolation of Microsoft Edge to protect against malicious websites. Protection is provided through the use of Hyper-V enabled containers isolated from the host operating system for opening untrusted websites
* Microsoft Defender Credential Guard – Provides virtualisation-based security to isolate credentials to protect against identity theft attacks. Much like Device Guard, Credential Guard uses Virtual Secure Mode (VSM) to isolate processes, in this case the Local Security Authority (LSA). The LSA performs various security operations, including the storage and management of user and system credentials. Unauthorised access to the LSA can lead to credential theft attacks, such as Pass-the-Hash or Pass-The-Ticket
* Microsoft Defender Firewall – Provides stateful packet inspection and blocking of network traffic. Windows Defender Firewall blocks unauthorized network traffic flowing into and out of the client endpoint reducing the attack surface of the device.
* Microsoft Defender SmartScreen – Provides malware and phishing website protection including downloaded files. SmartScreen protects users by performing the following
  * Analysing webpages for signs of distrustful behaviour and shows a warning page if it identifies suspicious activity.
  * Validates sites against a dynamic list of known phishing and malicious software sites and shows a warning page if it identifies page
  * Validates downloaded files against a list of known software sites and programs and shows a warning page if it identifies the site or program may be malicious
  * Validates downloaded files against a list of files that are known and used by a large number of windows users. If not found on the list SmartScreen shows a warning

Microsoft Defender Exploit guard comprises of the below features:

* Exploit protection – Exploit protection applies exploit mitigation mechanisms to applications. Works with third-party antivirus solutions and Windows Defender Antivirus
* Attack surface reduction – Attack Surface Reduction (ASR) rules reduce the attack surface of applications with rules that stop the vectors used by Office, script, and mail-based malware
* Network protection – Network protection extends the malware and social engineering protection offered by Microsoft Defender SmartScreen in Microsoft Edge to cover network traffic and connectivity on agency devices
* Controlled Folder Access – Controlled folder access protects files in key system folders from changes made by malicious and suspicious apps

Microsoft Defender Design Decisions for all agencies and implementation types.

Decision Point | Design Decision | Justification
--- | --- | ---
Microsoft Defender | Enabled | Microsoft Defender will be enabled to align with ACSC guidance
Microsoft Defender Capabilities Enabled in the SOE | Components:<br>Microsoft Defender Antivirus<br>Microsoft Defender Exploit Guard<br>Microsoft Defender Application Control<br>Microsoft Defender SmartScreen<br>Microsoft Defender Application Guard<br>Microsoft Defender Credential Guard<br>Microsoft Defender Firewall | Provides required security controls for the SOE
Microsoft Defender Antivirus Exclusions | Enabled and configured | Meets ACSC Windows 10 1909 hardening guidelines and aligns with security and compliance requirements
Microsoft Defender Exploit Guard Configuration | Enabled and configured | Meets ACSC Windows 10 1909 hardening guidelines and aligns with security and compliance requirements
Microsoft Defender Application Control Configuration | Enabled and configured | Meets ACSC Windows 10 1909 hardening guidelines and aligns with security and compliance requirements
Microsoft Defender Smart Screen Configuration | Enabled and configured | Meets ACSC Windows 10 1909 hardening guidelines and aligns with security and compliance requirements
Microsoft Defender Credential Guard Configuration | Enabled and configured | Aligns with security and compliance requirements. Enabled without lock allows Microsoft Defender Credential Guard to be managed remotely
Microsoft Defender Firewall Configuration | Enabled and configured | Default blocking of incoming and outbound connections as per the ACSC hardening guidance. Agencies should add additional firewalls rules as required for application and management connectivity. Further information on creating firewall rules can be referred to [Best practices for configuring Windows Defender Firewall](https://docs.microsoft.com/en-us/windows/security/threat-protection/windows-firewall/best-practices-configuring)

### Windows 10 security baselines

The Windows 10 security baseline settings support Windows 10 version 1809 and later.

The Windows 10 security baseline configures Windows settings as advised by the relevant Microsoft security teams. This baseline configures default settings in accordance with vendor best practice and increases the overall security of the client device whilst still allowing it to function correctly.

Windows 10 Security Baselines Design Decisions for all agencies and implementation types.

Decision Point | Design Decision | Justification
--- | --- | ---
Above Lock | Configured | Default configuration, meets security requirements.
App Runtime | Configured | Default configuration, meets security requirements.
Application Management | Configured | Default configuration, meets security requirements.
Auto Play | Configured | Default configuration, meets security requirements.
BitLocker | Configured | Default configuration, meets security requirements.
Browser | Configured | Default configuration, meets security requirements.
Connectivity | Configured | Default configuration, meets security requirements.
Credentials Delegation | Configured | Default configuration, meets security requirements.
Credentials UI | Configured | Default configuration, meets security requirements.
Data Protection | Configured | Default configuration, meets security requirements.
Device Guard | Configured | Default configuration, meets security requirements.
Device Installation | Configured | Default configuration, meets security requirements.
Device Lock | Configured | Prevent use of camera, require password, Disable the lock screen slide show settings, Set password minimum age in days
DMA Guard | Configured | Default configuration, meets security requirements.
Event Log Service | Configured | Event log sizes modified to align with ACSC guidance.
Experience | Configured | Default configuration, meets security requirements.
Exploit Guard | Configured | Default configuration, meets security requirements.
File Explorer | Configured | Default configuration, meets security requirements.
Firewall | Configured | Default configuration, meets security requirements.
Internet Explorer | Configured | Default configuration, meets security requirements.
Local Policies Security Options | Configured | UAC settings have been modified to align with ACSC guidance
Microsoft Defender | Configured | Scheduled scan has been disabled in this baseline. This is set in Defender ATP baseline to avoid conflicts.
MS Security Guide | Configured | Default configuration, meets security requirements.
MSS Legacy | Configured | Default configuration, meets security requirements.
Power | Configured | Default configuration, meets security requirements.
Remote Assistance | Configured | Default configuration, meets security requirements.
Remote Desktop Services | Configured | Default configuration, meets security requirements.
Remote Management | Configured | Default configuration, meets security requirements.
Remote Procedure Call | Configured | Default configuration, meets security requirements.
Search | Configured | Default configuration, meets security requirements.
Smart Screen | Configured | Default configuration, meets security requirements.
System | Configured | System boot start driver initialization modified to align with ACSC guidance.
Wi-Fi | Configured | Default configuration, meets security requirements.
Windows Connection Manager | Configured | Default configuration, meets security requirements.
Windows Hello for Business | Disabled | Does not meet security requirements.
Windows Ink Workspace | Configured | Default configuration, meets security requirements.
Windows PowerShell | Configured | Default configuration, meets security requirements.

### Microsoft Defender ATP baseline

The Microsoft Defender ATP security baseline settings support Windows 10 version 1809 and later.

The Defender ATP security baseline configures Defender settings as advised by the relevant Microsoft security teams. This baseline configures default settings in accordance with vendor best practice and increases the overall security of the client device whilst still allowing it to function correctly.

Defender ATP security baseline Design Decisions for all agencies and implementation types.

Decision Point | Design Decision | Justification
--- | --- | ---
Application Guard | Configured | Application Guard, Windows network isolation policy
BitLocker | Configured | Require device encryption with AES 256-bit XTS, BitLocker removable drive policy, BitLocker fixed drive policy, BitLocker system drive policy
Browser | Configured | Malicious site and file settings for MS Edge
Data Protection | Configured | DMA block
Device Guard | Configured | Credential guard settings
Device Installation | Configured | Manage the installation of hardware devices based off hardware identifiers
DMA Guard | Configured | Additional DMA protection for external DMA capable devices
Endpoint Detection and Response | Configured | Expedite telemetry reporting frequency, Sample sharing for all files
Firewall | Configured | Security association idle time before deletion, File Transfer Protocol, Packet queuing, Firewall profile domain, Firewall profile public, Firewall profile private, Firewall pre shared key encoding method, Certificate revocation list verification
Microsoft Defender  | Configured | Several security settings ranging from Scanning scripts / incoming mail, File Blocking to real-time monitoring
Microsoft Defender Security Center | Configured | Blocking the use of Exploit Guard settings
Smart Screen | Configured | Application protection interface for end users
Windows Hello for Business | Disabled | Does not meet security requirements.

### Microsoft Edge Security baseline

The Microsoft Edge security baseline settings support Edge version 80 and later.

The Microsoft Edge security baseline has pre-configured groups of Windows settings and the default settings as advised by the relevant Microsoft security teams.

Microsoft Edge security baseline Design Decisions for all agencies and implementation types.

Decision Point | Design Decision | Justification
--- | --- | ---
Microsoft Edge Settings | Configured | Default configuration, meets security requirements of this design.

### Windows Defender Application Control

Application control is a crucial line of defence for protecting enterprises given today's threat landscape, and it has an inherent advantage over traditional antivirus solutions. Specifically, application control moves away from the traditional application trust model where all applications are assumed trustworthy by default to one where applications must earn trust in order to run. ASD frequently cite application control as one of the most effective means for addressing the threat of executable file-based malware (.exe, .dll, etc.).

Windows Defender Application Control (WDAC) helps mitigate these types of security threats by restricting the applications that users can run and the code that runs in the System Core (kernel). WDAC policies also:
* Enforce Constrained Language mode for Windows PowerShell
* Enforce the use of drivers signed by Windows Hardware Quality labs and produced by partners who have an Extended Verification certificate.
* Block unsigned and unapproved scripts, MSIs, Universal Windows Store Applications, and .NET applications.

To reduce management overhead, WDAC allows for the use of managed installers; such as Microsoft Endpoint Configuration Manager. When configured, items deployed via the managaged installer are added into the allow list. 

When deploying WDAC it is important that it is deployed utilising audit mode prior to enforcement. 

Application Control Design Decisions for all agencies and implementation types.

Decision Point | Design Decision | Justification
--- | --- | ---
Application Whitelisting Product | WDAC | Microsoft recommended product for [application whitelisting](https://docs.microsoft.com/en-us/windows/security/threat-protection/windows-defender-application-control/windows-defender-application-control#choose-when-to-use-wdac-or-applocker) 
User Mode Code Integrity | Enabled | Restricts both kernel-mode and user-mode binaries. This aligns with the ACSC Windows 10 1909 hardening guidance.
Windows Hardware Quality Labs Signing | Required |  Blocks the execution of legacy drivers and ensures drivers have passed [Windows Hardware Certification Testing](https://docs.microsoft.com/en-us/windows-hardware/drivers/install/whql-release-signature) 
Flight Signing | Disabled  | Restricts the use of non production release binaries. Flightroot-signed binaries will not be trusted.
Unsigned System Integrity Policy | Agency Decision | The use of signed policies prevent administrative tampering and kernel mode exploit access. However, it does increase the administrative overhead associated with management and updating of policies. There is no current ACSC guidance on the configuration of signed integrity polices. 
EV Signers | Required  | Blocks the execution of drivers created by a partner without a Extended Verification (EV) certificate.
Advanced Boot Options Menu | Disabled | Restricts access to the advanced boot options menu. 
Boot Audit on Failure | Enabled | Enables investigation when a driver fails on boot. 
Script Enforcement | Enabled  |  Restricts PowerShell scripts and interactive sessions to constrained language mode. This aligns with the ACSC guidance on hardening PowerShell.
Enforce Store Applications | Enabled | Enforces WDAC policies on Universal Windows applications.
Update Policy No Reboot | Enabled | Ensures new policies can be applied without reboot.
Allow Supplemental Policies | Intune deployed: Enabled; Group Policy deployed: Disabled | Supplemental polices allow for policies to be targeted to users/groups. This is however not supported when policies are deployed by Group Policy.
Dynamic Code Security | Enabled | Enforces WDAC policies on .NET applications and dynamically-loaded libraries.
Managed Installer | Enabled | Allow lists applications deployed using a managed installer such as Microsoft Endpoint Configuration Manager.
Hypervisor-protected code integrity | Enabled | To align with the ACSC Windows 10 1909 hardening guidance.
Application Control method | A combination of publisher certificate and path rules and will be used. | Controlled via Intune for cloud managed devices and Group policy for hybrid devices.
Microsoft Block Rules | Configured | To align with the ACSC Windows 10 1909 hardening guidance. The latest Microsoft recommended block rules for [Windows 10](https://docs.microsoft.com/en-us/windows/security/threat-protection/windows-defender-application-control/microsoft-recommended-block-rules) and [Drivers](https://docs.microsoft.com/en-us/windows/security/threat-protection/windows-defender-application-control/microsoft-recommended-driver-block-rules) are available online at Microsoft website.
Intelligent Security Graph connection | Disabled | The Intelligent Security Graph connection allows applications to be run if they are deemed as good and there is no explicit block rule configured. 

### Identity providers

The identity providers section considers the different methods of logging on to the Windows 10 device. The local administrator account is addressed in a separate section.

Windows 10 provides various user account types or identity providers. This section outlines the identity providers that can be implemented for a Windows 10 device.

* Local Accounts - A local account is an account on a single Windows system. Local accounts are not replicated and cannot access to corporate resources. They allow access to local storage only. It may be desirable to disable, rename and scramble the passwords for the in-built local accounts
* Active Directory Domain - Domain identities are used to grant access to corporate resources and are implemented using Active Directory Domain Services. Administrators manage domain identities and ensure that users have access to the appropriate resources when group policies or any profile management solution is applied to the account. Domain identities are recommended if personalisation data will be stored in a corporate datacentre and will be synchronised to multiple corporate devices
* Azure Active Directory (Azure AD) - Azure AD is Microsoft's cloud directory and identity management service. Azure AD includes a full suite of identity management capabilities. Azure AD is a prerequisite for Microsoft Intune mobile device management including Conditional Access
* Microsoft Account - A Microsoft Account is an email address issued by or linked to a Microsoft authentication service. A Microsoft Account is a public version of an Azure Active Directory account. If this account is disabled certain features such as Windows Store cannot function.

Identity Providers Design Decisions for all agencies and implementation types.

Decision Point | Design Decision | Justification
--- | --- | ---
Guest Account (Local) | Disabled | In line with the ACSC Windows 10 1909 hardening guidelines
Guest Account Name | Renamed | In line with the ACSC Windows 10 1909 hardening guidelines
Microsoft Accounts | Disabled | In line with the ACSC Windows 10 1909 hardening guidelines
Windows Hello for Business | Disabled | Does not meet security requirements.
Windows Hello for Business Configuration Method | Disabled | Does not meet security requirements.

Additional Identity Providers Design Decisions for cloud native implementations

Decision Point | Design Decision | Justification
--- | --- | ---
Azure Active Directory Accounts | Enabled | Machines will be Azure AD Joined
Domain Accounts | Disabled | Machine will be Azure AD Joined

Additional Identity Providers Design Decisions for hybrid implementations

Decision Point | Design Decision | Justification
--- | --- | ---
Azure Active Directory Accounts | Enabled | Machines will be Hybrid Azure AD Joined
Domain Accounts | Enabled | Users will log onto devices using credentials which originate in an on-premises domain.<br>Machines will also be joined to the Agency domain

### Desktop analytics

Desktop Analytics is a service which provides insight and intelligence to an organisation to make informed decisions about the update readiness of the Windows clients. Desktop Analytics is a Microsoft cloud service that integrates with Configuration Manager (MECM). The data collected from the organisation is pooled together with many of other systems that are connected to the Microsoft cloud services.

Desktop Analytics can be used with Configuration Manager to:

* Capture application inventory for an organisation
* Analyse application compatibility with latest Windows 10 feature updates
* Determine compatibility issues and provide potential resolutions
* Analyse the data to provide pilot groups consisting of devices which cover most of the organisation's application set
* Deploy Windows 10 to pilot and production-managed devices

Desktop Analytics is replacing Windows Analytics and Configuration Manager (MECM) is a pre-requisite to enabling Desktop Analytics. Desktop Analytics provides:

* Device and software inventory - collect information such as application information and Windows versions
* Pilot identification - Identifying the devices which have the most coverage of components for the pilot of Windows upgrades and updates
* Issue identification - Potential issues are highlighted with upgrading and patching Windows by analysing captured data from Microsoft clients and organisational data. Potential mitigations can then be recommended
* Configuration Manager integration - Extends the on-premises infrastructure by allowing data to be analysed in order to assist in the deployment and management of Windows devices

Desktop Analytics Design Decisions for hybrid implementations

Decision Point | Design Decision | Justification
--- | --- | ---
Desktop Analytics  | Enable and configure | In line with the ACSC hardening guideline policy recommendations and meets requirements for future Desktop Analytics use
Data Location | United States | The Windows diagnostic data (non-business data) is encrypted using SSL and sent to and processed at a Microsoft-managed secure data centre located in the United States. This service is only hosted in the United States<br><br>The diagnostic data cannot be seen by another customer nor unauthorised Microsoft personnel<br><br>Note: As the diagnostic data is located offshore this may raise data sovereignty questions for the Agency, for which internal decisions might need to be made

### Telemetry collection

Windows 10 and Windows Server include the Connected User Experiences and Telemetry component, which uses Event Tracing for Windows (ETW) trace logging technology that gathers and stores diagnostic data events and data.

The operating system and some Microsoft management solutions, such as MECM use the same logging technology.

Windows uses telemetry information to analyse and fix software problems. It also helps Microsoft improve its software and provide updates that enhance the security and reliability of devices within organisations.

Telemetry level options are:

* Off – Disable telemetry data collection
* Security – Information that is required to help keep Windows secure, including info about telemetry client settings, the Malicious Software Removal Tool, and Windows Defender. This level is available only on Windows 10 Enterprise and Windows 10 Education, and Windows 10 IoT Core
* Basic – Basic device info, including quality-related info, application compatibility, and info from the Security level
* Enhanced – Additional insights, including how Windows and Windows apps are used, how they perform, advanced reliability info, and info from both the Basic and the Security levels
* Full – All info necessary to identify and help to fix problems, plus info from the Security, Basic, and Enhanced levels

The following image shows the information in each of the different Telemetry Collection levels.

![Telemetry Options](/assets/images/cd-telemetry.png)

Telemetry Collection Design Decisions for all agencies and implementation types.

Decision Point | Design Decision | Justification
--- | --- | ---
Allow Telemetry  | Enabled | In line with the ACSC hardening guideline policy recommendations and meets requirements for future Desktop Analytics use.
Telemetry Level | 2 – Enhanced | Microsoft recommend Enhanced Limited for Desktop Analytics

### Office macro hardening

Microsoft Office files can include Visual Basic for Applications (VBA) programming code (macro) embedded into the document.

A macro can comprise of several repeatable actions that can be coded or recorded and rerun later to automate repetitive tasks. Macros are powerful tools that can be easily created by novice users to greatly improve their productivity. However, an adversary can also create macros to perform a variety of malicious activities, such as assisting in the compromise of workstations to exfiltrate or deny access to sensitive information.

The ACSC provides guidelines in securing systems against malicious macros and recommend they are implemented in all Windows environments. The ACSC recommends that one of the following approaches is implemented:

* All macros are disabled
* Only macros from trusted locations are enabled
* Only digitally signed macros are enabled

Email and web content filtering is recommended to be implemented by the ACSC to ensure that macros are not delivered to users via email by malicious actors.

Office Macro Hardening Design Decisions for all agencies and implementation types.

Decision Point | Design Decision | Justification
--- | --- | ---
Implementation approach | Only digitally signed macros are enabled | In line with the ACSC Microsoft Office Macro security policy recommendation
Email and Web Content Filtering | Enabled | In line with the ACSC Microsoft Office Macro security policy recommendation
Configuration Method | Agency preference | Macro hardening can be configured via the Agencies existing Group Policies or Intune as well as Attack Surface Reduction in Windows Defender Exploit Guard

### OLE Hardening

Object Linking and Embedding (OLE) is a functionality within Microsoft Office which allows for the embedding and linking to documents and other objects. OLE is utilised to seamlessly integrate several types of data or components within a Microsoft Office file. 

Adversaries have leveraged the OLE functionality to enable and download malicious content or execute a malicious payload.  Within Microsoft Office 365 and Microsoft Office 2019 clients, the activation of objects that link to extensions that are considered high risk are blocked from executing. The list of high risk extensions can be configured through the use of Intune or GPOs.

The ACSC provides guidance around securing systems against malicious OLE packages and recommend they are implemented in all Windows environments. The guidance is to block all OLE packages from executing in Word, PowerPoint, and Excel.

Office OLE Hardening Design Decisions for all agencies and implementation types.

Decision Point | Design Decision | Justification
--- | --- | ---
OLE configuration | Block all | To meet the essential 8 requirements and to align to ACSC guidance

### Local administrator

The default local Administrator account is a highly privileged user account found on every Windows operating system. The Administrator account is the first account that is created during the installation for all Windows client operating systems.

The Administrator account can be used to create local users and assign user rights and access control permissions. It can also be used take control of local resources at any time simply by changing the user rights and permissions.

The default Administrator account cannot be deleted or locked out, but it can be renamed and / or disabled. It is Microsoft best practice and an ACSC hardening guideline recommendation to leave the Administrator account disabled and renamed.

If there is a requirement to utilise the local Administrator account in an on-premises environment, Microsoft provides Local Administrator Password Solution (LAPS), an Active Directory integrated Access Control List (ACL) protected password management tool.

LAPS allows system administrators the ability to set a different, random password for the common local administrator account on each computer in the domain and store the password for the computer's local administrator account in Active Directory, secured in a confidential attribute in the computer's corresponding Active Directory object.

Local Administrator Design Decisions for all agencies and implementation types.

Decision Point | Design Decision | Justification
--- | --- | ---
Local Administrator Account | Disabled | The local administrator account will be disabled in line with the ACSC Windows 10 1909 hardening guideline policy recommendations
Local Administrator Account Name | Renamed | The local administrator account will be renamed during the image deployment<br>In line with [Microsoft recommendations for securing the local administrator account](https://docs.microsoft.com/en-us/windows/security/threat-protection/security-policy-settings/accounts-rename-administrator-account)
Local Administrator Account Password | Randomised | In line with the ACSC Windows 10 1909 hardening guideline policy recommendations
Additional Local Administrator Accounts | Not Configured | Additional administrator accounts will not be created during the image deployment
LAPS | Not Configured | Not required for the solution. The local Administrator account will be disabled and renamed

## iOS

ACSC guidance for mobile devices currently only covers iOS devices. Due to this, the recommendation is to utilise iOS for Agency devices to ensure that the environment is following security guidelines.

### iOS devices

iOS follows a yearly major release cycle. With every major release of iOS, older iPhone devices are deprecated from support, hence security updates will not be available to these devices. 

Update policies control the update frequency  of managed devices. Intune can define update policies that specify how and when service updates are deployed to iOS devices. By using update rings, it is possible to create an update strategy that mirrors business needs.

iOS devices should be configured to adhere to the ACSC hardening requirements to provide secure access to corporate information. ACSC recommends that the latest version of iOS is used in order to have the most secure operating system present on the device.

iOS Devices Design Decisions for all agencies and implementation types.

Decision Point | Design Decision | Justification
--- | --- | ---
iOS version | iOS 14 or above | To align with the ACSC Security Configuration guide – iOS version enforcement of n or n-1 will allow for testing of patches and internal applications before deploying operating system updates.<br><br>Apple applies the n-1 rule to incremental updates, all other versions are no longer signed. <br><br>Once a version is not signed a device cannot be restored to it. iOS 14 has not undergone an ASD Cryptographic Evaluation (ACE) whereas iOS 13 has undergone an ACE. Agencies will need to evaluate whether iOS 14 fits their requirements from an encryption standpoint.
iOS Devices | iPhone XS and above | iPhone X and older are all vulnerable to the exploit Checkm8 and should be avoided.
Update Policies | Configured | To align with the ACSC Security Configuration guide
Jailbroken/rooted devices | Blocked | Prevents jail broken devices from accessing Agency information. 

### Enrolling iOS devices

Device enrolment registers the iOS devices into the corporate device management solution and ensures the device is then able to be managed by administrators.

Microsoft Intune provides a mechanism for enrolling devices into Azure AD. Once registered the device is populated into Intune policy groups using dynamic membership. This ensures that the device meets the compliance policy, monitored, and secured to the Agencies security requirements. 

Microsoft Intune provides three separate experience in enrolling the iOS devices into the Agencies Azure Active directory. The enrolment experiences are: 

* Automated device enrollment (ADE) (formerly Device Enrolment Program (DEP)) – Device Enrolment Program is a managed device enrolment process. The devices serial number is registered with Apple Business Manager allows Intune to bypass Assisted Setup by preconfigure device settings. The user's account will be assigned to the device. The device will be marked as a Supervised device.
* Device Enrolment Manager (DEM) – Device Enrolment Manager assigns a single Azure Active Directory account as the owner of the device. The end users cannot administer or purchase any apps on the device
* User Enrolment – User enrolment process requires users set up the iOS device and manually install Company Portal to register the device as Intune enrolled device. The device will be marked as a BYOD device

Enrolling iOS Design Decisions for all agencies and implementation types.

Decision Point | Design Decision | Justification
--- | --- | ---
Enrolment method | Automated device enrollment | Devices will be pre-configured before the device is handed over to the end user. This in line with the ACSC iOS Secure Configuration Hardening guide for PROTECTED devices.

### Securing iOS devices

Intune provides ability to configure iOS configuration settings for securing, configuring and applications on iOS devices. These configurations are managed via Mobile Device Management (MDM) as an Intune policy. MDM allows the Agency to deploy consistent configuration on enrolled iOS devices.

MDM provides the capability to configure iOS devices. These devices must be configured to meet ACSC iOS Secure Hardening guide to ensure the device can access and store the Agencies data. These configurations can be categories as:

* Security – Ensure device has up to date and secure authentication policies and encryption devices that meets ACSC Secure iOS guide. 
* Branding – The Agencies branding for lock screen, wallpapers, and reporting if the device is lost can be configured.
* Device features – Configures device features, for example, AirDrop and Bluetooth pairing, within iOS devices.

Securing iOS devices Design Decisions for all agencies and implementation types.

Decision Point | Design Decision | Justification
--- | --- | ---
Mobile Device Management | Mobile devices will be managed using Intune | Leveraging the capabilities already available in the licensing agreement. Intune and Apple Business Manager will be adopted to manage mobile devices.
Security policies and hardening requirements | Security policies will be enforced on all mobile devices managed by the Agency | Security policies will be configured in line with the ACSC Security Configuration Guide – Apple iOS 14 Devices.
Device Features | Configured | Device features configured in line with ACSC guidance. 

Mobile Device Management Configuration applicable to all agencies and implementation types

Configuration | Value | Description
--- | --- | ---
**Security** | | |
Supervised Mode | Enable | Agency iPhones are managed devices. This is in line with the ACSC iOS Secure Configuration Hardening guide for PROTECTED devices.
Device passcode | Device passcode of 14 characters or above. Alphanumic in nature and must contain a minimum of 1 special character | iOS devices, by default, is encrypted once a passcode is provided to the device. Configured in line with ISM requirements on password length.
Biometrics | Disable | This is in line with the ACSC iOS Secure Configuration Hardening guide for PROTECTED devices.
Mobile Device Management | Enable | Mobile Device Management provides the Agency better auditing tools on the device. In line with ACSC iOS Secure Configuration Hardening guide for PROTECTED devices
Maximum Auto-Lock  | 2 minutes | Auto Lock will lock a device if it is inactive for specified time
Virtual Private Network (VPN)  | Configured | Per-app VPN will be set up to secure communication between the device and the Agencies data. This is in line with ACSC iOS Secure Configuration Hardening guide
**Branding** | | |
Lock Screen and background branding | Configured | Agency branding will be applied to endpoints. It is recommended that agencies include the contact information of the relevant IT Support in the event that a device is lost.
**Device Features** | | |
Disable Screenshot and screen recording | Disable | Disablement of screenshot and screen recording, disable users from taking a snapshot of protected documents
Allow iCloud backup, document data and Keychain | Disable | The Agencies data on mobile devices must be uploaded into the Agencies tenant. This is done to prevent accidental information being backed up to iCloud or another unsecure data store
Managed Open-In | Enable  | Managed Open-In is enable segregates between corporate managed and personal application in an iOS device. This in line with the ACSC iOS Secure Configuration Hardening guide for PROTECTED devices.
Allow documents from managed sources in unmanaged destination | Disable | The Agencies data cannot be moved between managed and unmanaged application destination. This is to prevent PROTECTED from being transferred to an unmanaged application or location
Treat AirDrop as unmanaged destination | Disable | AirDrop provides the ability to wirelessly transfer documents between Apple devices. Setting AirDrop as an unmanaged destination prevents users from accidentally transferring Agency data to unsecure applications or locations
Restricted Application List | Configured | Unapproved application installs will be alerted upon. 

### Securing iOS applications

Mobile Application Management (MAM) in Intune allows configuration of managed applications within an iOS device. Managed applications enclose Agency applications within an application bubble. This bubble prevents accidental data spillage by preventing cutting and pasting, as well as allowing data sharing within the application bubble.

MAM provides the capability to configure iOS device applications. These configurations include:

* Managed Applications – List of Agency business applications
* Managed Application configuration – Configure and secure managed application configuration within the device. These configurations allow and isolate managed applications to resides next to unmanaged applications.
* Per-app VPN - Secure communication between applications on devices, and the Office 365 tenant. This will require the Agencies VPN device setup to accept communication from the VPN connection from managed apps.

Securing iOS Design Decisions for all agencies and implementation types.

Decision Point | Design Decision | Justification
--- | --- | ---
Managed Applications | Microsoft Corporate Portal<br>Microsoft Azure Information Protection<br>Microsoft Word<br>Microsoft Excel<br>Microsoft Outlook<br>Microsoft PowerPoint<br>Microsoft Teams<br>Microsoft Edge<br>Adobe Reader<br>Microsoft OneDrive<br>Microsoft OneNote<br>Microsoft Whiteboard<br>Microsoft Planner<br>PowerApps | This is to meet the Agencies requirement for managed applications
Application VPN configuration | Configured for Managed Applications | Managed applications will use VPN to secure its connection to the Agencies Office 365 tenant
Disable organisation data to be backed up to iCloud | Disable | PROTECTED must be stored within the Agencies environment / corporate data store
Encrypt organisation data in mobile device | Configured | To ensure encryption requirements are met based on ACSC hardening requirements. 
Send organisation data to unmanaged apps | Policy managed apps with Open In/Share Filtering | Prevents data to be shared between managed application stated above and other unmanaged application on the device
Save copies of organisation data | SharePoint and OneDrive for Business only | Ensure all data is saved within the Agencies tenant
Organisation data notification | Block organisation Data | Prevents Agency information being displayed on the lock screen
Microsoft Edge Configuration | Configured.<br>Set Microsoft Edge proxy and homepage URL to Agencies Intranet | Configured so Microsoft Edge is able to access Agencies internal websites
Microsoft Outlook | Configured.<br>Ensure Contact list is added into Outlook Contact list rather than device | Configured so Agencies contact list is maintained within managed application rather than the phone's contact details

### Remote wipe iOS devices

iOS devices can be used to access information while away from the office. The device is used to access the Agencies intranet, documents and for collaboration purpose. 

Intune provides the ability to reset the device (factory reset) or remove managed application data from the device. The Factory reset option remotely wipes and resets the iOS device. The Intune manage application wipe does not clear the device of non-managed application data which allow for potential security issues.

The Agency has indicated that the device must be able to be remotely wipe in case it is lost. This is to ensure information cannot be recovered in any circumstances.

Remote wipe iOS devices Design Decisions for all agencies and implementation types.

Decision Point | Design Decision | Justification
--- | --- | ---
Device Lost Mode | Enabled | Lost mode sends notification to the device's lock screen.
Device remote wipe | Device will be remote wiped of corporate data (Factory Reset). | To minimize the security if the device is lost or not return to the Agency.<br>Intune remotely wipe and reset the iOS device when a user is off-boarded. This allows the device to be reassign to other users in the Agency. 
