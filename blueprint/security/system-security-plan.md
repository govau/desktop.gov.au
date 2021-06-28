---
layout: page
title: System security plan
menu: security
download: true
---

## Introduction

### System name

CloudSystem.

### System overview

The CloudSystem leverages the Information Security Registered Assessors Program (IRAP) assessed Microsoft Azure and Microsoft 365 platforms and their associated services. The CloudSystem includes the following components to improve the security posture of a target Agency:

* Cloud identity – Azure Active Directory (Azure AD) configuration including conditional access allowing log in from anywhere and appropriate security policies to be applied.
* Microsoft 365 – Configuration of Exchange Online, SharePoint Online, OneDrive for Business, Microsoft Teams, Microsoft Forms, Microsoft Whiteboard and Microsoft Planner allowing cloud-based file storage and collaboration.
* Device management – Management of security and configuration profiles for enrolled devices including the testing against security baselines and confirmation of security compliance.
* Applications – Delivery and configuration of applications appropriate to the user.
* Security stack – Security configuration of Microsoft 365 and endpoint devices to maximise compliance and minimise risk.
* Autopilot deployment – Configuration of Autopilot to allow for automated deployment (and redeployment when required) of devices with no user interaction.
* Support – A flexible support model where system administration and Role Based Access Control (RBAC) is provided regardless of whether the support is carried out by in house staff, third party contractors or a managed service provider.

### System classification

The CloudSystem is designed to be able to achieve and maintain security accreditation up to PROTECTED.

### System purpose and scope

The CloudSystem is intended to achieve a PROTECTED standard and raise Australian Government agencies cyber security posture. The CloudSystem details technology and configuration settings to deploy a cloud-only Microsoft 365 solution.

Note: The Microsoft suite includes multiple products including Windows 10, Microsoft 365 and Enterprise Mobility + Security (EM+S).

### System boundary

The boundary of the system is the Subscription level of the Microsoft 365 implementation. The tenancy and all Microsoft components (including both Azure and Microsoft 365 hosted services), the transport of data between the endpoint devices and cloud services along with the endpoint devices themselves are included within the system. Network components are not considered to be part of the system.

As shown in Figure 1 the system therefore includes:

* Azure (including Azure AD)
* Multi-Factor Authentication (MFA)
* Subscription and its management
* Tenancy and its management
* Endpoints including the hardware and Basic Input/Output System (BIOS) and the management of the endpoints
* Transport of data between the endpoints and the cloud components

Note: This means that Transport Layer Security (TLS) would be included within the system boundary, but the network devices and mail gateway would not be included in the system boundary from a security perspective. Those items outside of the system boundary will be consumed and the existing security documentation will be utilised.

![Figure 1 Cloud-Only Security Boundary](/assets/images/security/security-boundary.png)

### Document purpose and scope

The purpose of this System Security Plan (SSP) is to describe the security implementation of the CloudSystem, including the underlying Azure and Microsoft 365 components that are leveraged in its deployment. This document is designed to comply with the Australian Government Information Security Manual (ISM) documentation requirements for system authorisation.

This document is deliberately written using descriptive and explanatory language to assist an Agency to understand how the CloudSystem operates securely, the security controls it provides, and the residual controls that must be addressed by an Agency.

For detailed information on how the CloudSystem addresses specific controls in the ISM (June 2021 update), refer to the ‘DTA - Cloud-Only Blueprint - System Security Plan Annex (June 2021)’.

### Overarching security policies

The security policies that the CloudSystem has been designed to comply with are listed below:

* The Australian Government ISM (June 2021) controls.
* The Australian Cyber Security Centre (ACSC) Strategies to Mitigate Cyber Security Incidents, including the Essential Eight Maturity Model.
* The ACSC Security Configuration Guide - Apple iOS 14 Devices (February 2020).
* The Protected Security Policy Framework.
* Hardening Microsoft Windows 10 version 1909 Workstation (June 2020).

### Related security documentation

In accordance with the requirements of the ISM, the following security documentation has been developed for the CloudSystem:

* DTA – Blueprint – Solution Overview (August 2020)
* DTA – Blueprint – Client Devices Design (August 2020)
* DTA – Blueprint – Platform Design (August 2020)
* DTA – Blueprint – Office 365 Design (August 2020)
* DTA – Cloud-Native Blueprint – System Security Plan (June 2021) (this document)
* DTA – Cloud- Native Blueprint – System Security Plan Annex (June 2021)
* DTA – Cloud- Native Blueprint – Security Risk Management Plan (October 2020)
* DTA – Cloud- Native Blueprint – Security Standard Operating Procedures (October 2020)
* DTA – Cloud- Native Blueprint – Incident Response Plan (October 2020)

The suite of documentation produced to support ACSC’s certification of Azure and Microsoft 365 for PROTECTED have also been leveraged in the development of the CloudSystem, and includes the following :

* 2020 Microsoft Azure IRAP - Azure Security Fundamentals and Cloud Services Assessment
* Office 365 Security Fundamentals and Cloud Services IRAP Assessment Report 2020
* 2019 Microsoft Office 365 IRAP Assessment Report

These documents are available from the [Microsoft Service Trust Portal](https://servicetrust.microsoft.com/ViewPage/Australia).

### Risk assessment

The results of the threat and risk assessment undertaken on the CloudSystem are documented in the ‘DTA – Cloud-Only Blueprint – Security Risk Management Plan (August 2020)’ (SRMP). This document describes the reduction in risk to the confidentiality, integrity and availability of system components and information processed and stored by the CloudSystem by the implementation of security controls and mitigations.

## Assessment of services

This section provides details of the security assessment status of each Azure and Microsoft 365 service used by the CloudSystem as listed in their respective IRAP reports. The assessment status of each of the utilised services and any associated mitigations is shown in Table 1.

Table 1 Assessment of Services

Category | Service | Assessment Status | Mitigation
--- | --- | --- | ---
General | Azure Portal | PROTECTED | N/A
Identity Services | Azure AD | PROTECTED | N/A
Identity Services | Conditional Access | Not Assessed | Conditional Access is an Azure AD Premium P1 licenced feature of Azure AD (included in Microsoft 365 E3) that restricts access to cloud resources and management tools beyond just a successful authentication. It includes customisable policies based on location, user, device and more. Conditional Access is an additional security capability that is part of Azure AD, which is PROTECTED certified. 
Identity Services | Azure MFA | PROTECTED | N/A
Identity Services | Azure AD Identity Protection | Not Assessed | Azure AD Identity Protection is an Azure AD Premium P2 licenced feature of Azure AD (included in Microsoft 365 E5) that allows organisations to accomplish three key tasks:<br><br>* Automate the detection and remediation of identity-based risks.<br>* Investigate risks using data in the portal.<br>* Export risk detection data to third-party utilities for further analysis.
Microsoft 365 | Exchange Online, SharePoint Online, Microsoft Teams, Microsoft Forms, Microsoft Whiteboard, Microsoft Planner | PROTECTED | N/A
Monitoring and Compliance | Intune Policies| PROTECTED | Microsoft Endpoint Manager - Intune (Intune) is configured to allow policies to be created and deployed to devices that configure, check for compliance and assess against a security baseline. These policies are applied and reported against in the Intune web console.

## Section definitions

The remaining sections of this document relate specifically to the chapters of the ISM. For each chapter of the ISM there is a corresponding section in this document, which is divided into four sections as detailed below in Table 2.

Table 2 Section Definitions

Section | Description
--- | ---
Applicability to CloudSystem | For each chapter, the applicability relates to whether the CloudSystem provides any technical, process or documentation that need to be assessed.<br><br>The CloudSystem inherits many controls from the underlying Azure and Microsoft 365 platforms, so if a chapter is listed as Not Applicable then the Agency may or may not be required to address the control. The reason the chapter is not applicable is stated in this section and if the Agency is required to address the controls then this is listed in the Residual controls to be addressed by the Agency section.
CloudSystem compliance approach | The compliance approach for the CloudSystem is described in this section to provide:<br><br>* The background and context for how the CloudSystem address the controls in the chapter<br>* To provide the Agency with information to assist in the assessment of the CloudSystem
Security controls provided by the CloudSystem | The specific technical, process or documentation that the CloudSystem provides to address the controls are listed in this section.
Residual controls to be addressed by the Agency | If there are any residual controls that the Agency must address in relation to the operation of the CloudSystem, then they are listed in this section.

## Summary of applicability

A summary of the applicability and responsibility for the controls presented in of each chapter of the for the CloudSystem is listed below in Table 3. Each of these chapters are discussed in further details in this document, and the implementation status of each control is listed in the SSP Annex.

Table 3 Summary of applicability

ISM Chapter | Applicability | Rationale
--- | --- | ---
Guidelines for Cyber Security Roles | Not Applicable | Fulfilling these roles is an Agency responsibility.
Guidelines for Cyber Security Incidents | Not Applicable | The Agency is responsible for identifying, managing and reporting cyber security incidents.
Guidelines for Outsourcing | Applicable  | Shared responsibility between the CloudSystem and the Agency consuming it.
Guidelines for Security Documentation | Applicable | The CloudSystem provides system-specific documentation to be read in conjunction with the Agency’s cyber security strategy.
Guidelines for Physical Security | Not Applicable | The CloudSystem inherits the physical security controls which are implemented by Microsoft for Azure and Microsoft 365 components.
Guidelines for Personnel Security | Not Applicable | The Agency is responsible for the personnel security as it relates to users of the CloudSystem.
Guidelines for Communications Infrastructure | Not Applicable | The Agency is responsible for communications infrastructure leveraged by the CloudSystem.
Guidelines for Communications Systems | Applicable | The CloudSystem includes Microsoft Teams which provides video conferencing functionality.
Guidelines for Enterprise Mobility | Applicable | The CloudSystem includes the management and use of mobile devices.
Guidelines for Evaluated Products | Applicable | The CloudSystem includes Windows 10 which has been evaluated. Additionally, the CloudSystem leverages Microsoft 365 services which include evaluated products.
Guidelines for ICT Equipment Management | Not Applicable | The CloudSystem does not contain any Information and Communications Technology (ICT) Equipment, and the security of the Azure and Microsoft 365 hosting equipment is the responsibility of Microsoft.
Guidelines for Media | Applicable | The CloudSystem is responsible for restricting the use of unapproved media.
Guidelines for System Hardening | Applicable | Hardening of operating systems and applications included in the CloudSystem is applicable.
Guidelines for System Management | Applicable | Management of CloudSystem system components is applicable.
Guidelines for System Monitoring | Applicable | Monitoring of CloudSystem system components is applicable.
Guidelines for Software Development | Not Applicable | The CloudSystem is not designed to support software development activities.
Guidelines for Database Systems | Not Applicable | The CloudSystem does not include the use of databases.
Guidelines for Email | Applicable | The CloudSystem leverages Microsoft 365 to provide email functionality.
Guidelines for Networking | Applicable | The CloudSystem is designed to run using the public internet. All security controls are implemented on the endpoint devices and the Microsoft 365 component. The Microsoft 365 design includes a high-level network diagram showing the components that are considered in scope. Conditional access policies and MCAS to limit access to the environment from anonymity networks. The configuration and fine tuning of these policies is the shared responsibility of the Agency and the CloudSystem.
Guidelines for Cryptography | Applicable | The CloudSystem makes use of cryptography to protect both data at rest and data in transit.
Guidelines for Gateways | Applicable | The CloudSystem leverages Exchange Online Protection and Microsoft 365 ATP for content filtering.
Guidelines for Data Transfers | Applicable | The CloudSystem is responsible for implementing technical controls relating to data transfer.

## Cyber security roles

### Chief Information Security Officer

#### Applicability to CloudSystem

Not applicable as appointing a Chief Information Security Officer (CISO) is an Agency’s responsibility.

#### CloudSystem compliance approach

The CloudSystem is deployed into the Agency’s environment and is under the scope of the Agency’s CISO.

#### Security controls provided by the CloudSystem

Not applicable.

#### Residual controls to be addressed by the Agency

* The Agency must appoint a CISO to provide cyber security leadership and guidance.
* The Agency's CISO is responsible for all duties outlined in the Annex.

### System owners

#### Applicability to CloudSystem

Not applicable as the CloudSystem does not designate a system owner.

#### CloudSystem compliance approach

The deployment of the CloudSystem into an Agency’s environment can be designated as a specific system, or it can form part of a broader system.

By default, the CloudSystem is defined as a system and all documentation, including this SSP, is written in that context.

#### Security controls provided by the CloudSystem

Not Applicable.

#### Residual controls to be addressed by the Agency

* The Agency must designate a System Owner for the CloudSystem.
* The System Owner must perform the relevant duties outlined in the Annex.

## Cyber security incidents

### Detecting cyber security incidents

#### Applicability to CloudSystem

Not applicable to the CloudSystem as the detection of cyber security incidents is the responsibility of the Agency.

#### CloudSystem compliance approach

The CloudSystem implements technical controls and processes to assist the Agency with detecting cyber security incidents related to the system.

#### Security controls provided by the CloudSystem

The CloudSystem utilises Microsoft Defender Advanced Threat Protection (ATP) to assist in the detection of cyber security incidents. The Defender ATP security operations dashboard shows:

* Active alerts
* Machines at risk
* Sensor health
* Service health
* Daily machines reporting
* Active automated investigations
* Users at risk and
* Suspicious activities

#### Residual controls to be addressed by the Agency

* The Agency must develop and implement an Intrusion Detection and Prevention Policy, which can leverage the security controls implemented by the CloudSystem and meets requirements outlined in the Annex.

### Managing cyber security incidents

#### Applicability to CloudSystem

Not applicable to the CloudSystem as the management of cyber security incidents is the responsibility of the Agency.

#### CloudSystem compliance approach

The CloudSystem implements technical controls and processes to assist the Agency with managing cyber security incidents related to the system.

#### Security controls provided by the CloudSystem

* The CloudSystem utilises Microsoft Defender ATP to assist in the management of cyber security incidents. Specific capabilities include the Incident queue and Incident management pane views.

#### Residual controls to be addressed by the Agency

* The Agency should establish a cyber security incident register, cyber security incident communication and response strategy and associated procedures that meet requirements outlined in the Annex.

### Reporting cyber security incidents

#### Applicability to CloudSystem

Not applicable to the CloudSystem as the reporting of cyber security incidents is the responsibility of the Agency.

#### CloudSystem compliance approach

The reporting requirements for cyber security incidents are an Agency responsibility.

#### Security controls provided by the CloudSystem

Not applicable.

#### Residual controls to be addressed by the Agency

* The Agency should establish a process and standard operating procedures for reporting cyber security incidents that meet requirements outlined in the Annex.

## Outsourcing

### Information technology and cloud services

#### Applicability to CloudSystem

Outsourcing is applicable to the CloudSystem as it leverages both Azure and Microsoft 365, which are cloud services.

#### CloudSystem compliance approach

The CloudSystem leverages Microsoft Azure and Microsoft 365, which have been IRAP assessed at PROTECTED by CyberCX. All CloudSystem services have been IRAP assessed.

Azure AD is defined as a non-regional service, but [hosts identity data in Australian datacentres](https://docs.microsoft.com/en-us/azure/active-directory/fundamentals/active-directory-data-storage-australia) for customers that provide an Australian or New Zealand address. This includes Azure AD Directory Management and Authentication functions. Other Azure AD functions, including Azure MFA, store data in global datacentres.

Where possible the CloudSystem leverages services located in Australia, otherwise the United States is selected. Microsoft Cloud App Security is available in the Europe or United States regions. If the Azure AD tenant is in Australia, then the [United States is automatically selected](https://docs.microsoft.com/en-us/cloud-app-security/cas-compliance-trust). [Microsoft Defender Advanced Threat Protection](https://docs.microsoft.com/en-us/windows/security/threat-protection/microsoft-defender-atp/data-storage-privacy) is available in Europe, United Kingdom, or in the United States. The United States is selected as part of the service creation for the CloudSystem.

#### Security controls provided by the CloudSystem

* Australian data locations have been selected where supported by Azure and Microsoft 365 services leveraged by the CloudSystem. Where Australia is not available, the United States is used.
* Microsoft provides a shared responsibility model which outlines how security responsibilities are shared between itself and the Agency.

#### Residual controls to be addressed by the Agency

* The Agency must assess, establish, manage and maintain the commercial and contractual relationship with Microsoft as the provider of the cloud services and review changes to the Microsoft IRAP assessments as they occur.

#### Shared responsibility

When consuming a cloud service, management of some security controls is transferred from the agency to the Cloud Service Provider (CSP), in this case Microsoft. The level of control transferred ultimately depends on the type of services being consumed i.e. cloud native or hybrid deployment and the agreement made with Microsoft.

Whilst responsibility for controls may be shared, agencies must be conscious that security risk is not transferred to the service provider. It is therefore critical that agencies understand how the sharing of responsibilities impacts system risk, and what impact it may have on [Assessing and Authorising the system within their environment](https://www.cyber.gov.au/acsc/view-all-content/publications/anatomy-cloud-assessment-and-authorisation).

In general, [Microsoft defines](https://docs.microsoft.com/en-us/azure/security/fundamentals/shared-responsibility) themselves as being responsible for:

* Ensuring the physical systems and infrastructure required for the operation of a cloud service is secured appropriately.
* Being accountable in the event of an incident relating to the physical systems and infrastructure they manage as required for the operation of a cloud service.
* Assessing, managing and where possible mitigating risks inherent with the physical systems and infrastructure required for the operation of a cloud service.

In the context of Software as a Service (SaaS) protected utility platforms, Microsoft is responsible for:

* Incident response
* Backups
* Physical security
* System hardening
* Vulnerability and patch management
* Software development

When deploying a cloud native model, in the context of Software as a Service (SaaS) protected utility platforms, the agency is responsible for:

* Access management
* System monitoring
* Client devices

Overall, the agency is deemed accountable for any technology platform when in use, with Microsoft responsible for some or all of the platform operational management.

A suggested high-level shared responsibility matrix for the technology stack across the platform, Microsoft 365 and client devices has been tabled below. There are three defined stakeholders who share the responsibility to maintain the agency’s security capabilities.

* **Agency**: Australian government agency adapting and implementing the DTA cloud native blueprint.
* **Microsoft**: Cloud Service Provider who provide and/or manage the defined technology platforms.
* **Product Vendors**: external product vendors (such as Apple for iOS) that provide or manage platforms within the agency’s ecosystem that are not performed by Microsoft.

##### Platform

CATEGORY | ITEM | INCIDENT RESPONSE | BACKUPS | PHYSICAL SECURITY | SYSTEM HARDENING | ACCESS MANAGEMENT | VULNERABILITY & PATCH MANAGEMENT | SYSTEM MONITORING | SOFTWARE DEVELOPMENT
--- | --- | --- | --- | --- | --- | --- | --- | --- | ---
IDENTITY & ACCESS MANAGEMENT | AZURE ACTIVE DIRECTORY | MICROSOFT | MICROSOFT | MICROSOFT | MICROSOFT | AGENCY | MICROSOFT | MICROSOFT | MICROSOFT
SECURITY | MICROSOFT CLOUD APP SECURITY | MICROSOFT | MICROSOFT | MICROSOFT | MICROSOFT | AGENCY | MICROSOFT | MICROSOFT | MICROSOFT
SECURITY | AZURE ADVANCED THREAT PROTECTION | MICROSOFT | MICROSOFT | MICROSOFT | MICROSOFT | AGENCY | MICROSOFT | MICROSOFT | MICROSOFT
SECURITY | MICROSOFT DEFENDER ADVANCED THREAT PROTECTION | MICROSOFT | MICROSOFT | MICROSOFT | MICROSOFT | AGENCY | MICROSOFT | MICROSOFT | MICROSOFT
SECURITY | LOG ANALYTICS | MICROSOFT | MICROSOFT | MICROSOFT | MICROSOFT | AGENCY | MICROSOFT | MICROSOFT | MICROSOFT
SECURITY | SECURITY INFORMATION & EVENT MANAGEMENT | AGENCY | AGENCY | AGENCY | AGENCY | AGENCY | AGENCY | AGENCY | AGENCY
CLIENT CONFIGURATION | MICROSOFT ENDPOINT MANAGER - INTUNE | AGENCY  | AGENCY | MICROSOFT | MICROSOFT | AGENCY | MICROSOFT | MICROSOFT | MICROSOFT
CLIENT CONFIGURATION | PRINTING | AGENCY | AGENCY | AGENCY | AGENCY | AGENCY | AGENCY | AGENCY | AGENCY
BACKUP & OPERATIONAL MANAGEMENT | Microsoft 365 BACKUP | MICROSOFT | MICROSOFT | MICROSOFT | MICROSOFT | AGENCY | MICROSOFT | MICROSOFT | MICROSOFT
SYSTEM ADMINISTRATION | ADMINISTRATIVE CONSOLES | MICROSOFT | MICROSOFT | MICROSOFT | AGENCY | AGENCY | MICROSOFT | MICROSOFT | MICROSOFT
SYSTEM ADMINISTRATION | PRIVILEGED IDENTITY MANAGEMENT | MICROSOFT | MICROSOFT | MICROSOFT | AGENCY | AGENCY | MICROSOFT | MICROSOFT | MICROSOFT

##### Microsoft 365

ITEM | INCIDENT RESPONSE | BACKUPS | PHYSICAL SECURITY | SYSTEM HARDENING | ACCESS MANAGEMENT | VULNERABILITY & PATCH MANAGEMENT | SYSTEM MONITORING | SOFTWARE DEVELOPMENT
--- | --- | --- | --- | --- | --- | --- | --- | ---
EXCHANGE ONLINE | MICROSOFT | MICROSOFT | MICROSOFT | MICROSOFT | AGENCY | MICROSOFT | AGENCY | MICROSOFT
SHAREPOINT ONLINE | MICROSOFT | MICROSOFT | MICROSOFT | MICROSOFT | AGENCY | MICROSOFT | AGENCY | MICROSOFT
ONEDRIVE FOR BUSINESS | MICROSOFT | MICROSOFT | MICROSOFT | MICROSOFT | AGENCY | MICROSOFT | AGENCY | MICROSOFT
MICROSOFT TEAMS | MICROSOFT | MICROSOFT | MICROSOFT | MICROSOFT | AGENCY | MICROSOFT | AGENCY | MICROSOFT
POWER BI | MICROSOFT | MICROSOFT | MICROSOFT | MICROSOFT | AGENCY | MICROSOFT | AGENCY | MICROSOFT
SECURITY & COMPLIANCE PLATFORMS | MICROSOFT | MICROSOFT | MICROSOFT | MICROSOFT | AGENCY | MICROSOFT | AGENCY | MICROSOFT
EXCHANGE ONLINE PROTECTION | MICROSOFT | MICROSOFT | MICROSOFT | MICROSOFT | AGENCY | MICROSOFT | AGENCY | MICROSOFT
MICROSOFT 365 ADVANCED THREAT PROTECTION | MICROSOFT | MICROSOFT | MICROSOFT | MICROSOFT | AGENCY | MICROSOFT | AGENCY | MICROSOFT
MICROSOFT FORMS | MICROSOFT | MICROSOFT | MICROSOFT | MICROSOFT | AGENCY | MICROSOFT | AGENCY | MICROSOFT
MICROSOFT WHITEBOARD | MICROSOFT | MICROSOFT | MICROSOFT | MICROSOFT | AGENCY | MICROSOFT | AGENCY | MICROSOFT
MICROSOFT PLANNER | MICROSOFT | MICROSOFT | MICROSOFT | MICROSOFT | AGENCY | MICROSOFT | AGENCY | MICROSOFT

##### Client devices

ITEM | INCIDENT RESPONSE | BACKUPS | PHYSICAL SECURITY | SYSTEM HARDENING | ACCESS MANAGEMENT | VULNERABILITY & PATCH MANAGEMENT | SYSTEM MONITORING | SOFTWARE DEVELOPMENT
--- | --- | --- | --- | --- | --- | --- | --- | ---
WINDOWS 10 – INTUNE MANAGED | AGENCY | AGENCY | AGENCY | AGENCY | AGENCY | AGENCY | AGENCY | MICROSOFT
IOS – INTUNE MANAGED | AGENCY | AGENCY | AGENCY | AGENCY | AGENCY | AGENCY | AGENCY | APPLE

## Security documentation

### Development and maintenance of security documentation

#### Applicability to CloudSystem

Development and maintenance of security documentation is applicable to the CloudSystem.

#### CloudSystem compliance approach

The CloudSystem provides security documentation that an Agency can review, approve and incorporate into the broader Agency-level security documentation.

Cloud-Only provides an SSP (this document), SSP Annex (formerly the Statement of Applicability (SoA)), SRMP, Incident Response Plan (IRP), Standard Operating Procedures (SOPs) and other operational documentation to assist in the understanding of the Cloud-Only system and the security controls included.

#### Security controls provided by the CloudSystem

* The CloudSystem provides SOPs for administrators and support staff to understand, monitor and operate the CloudSystem-provided security controls.
* The CloudSystem includes detailed design, configuration, operational and support documentation.
* The CloudSystem provides security documentation for input into an Agency’s security processes.
* The CloudSystem's security documentation and notification of subsequent changes is communicated by DTA to Agencies who have implemented the CloudSystem.

#### Residual controls to be addressed by the Agency

* The Agency must develop a cyber security strategy.
* The Agency CISO or equivalent should approve all security documentation and ensure the documentation is reviewed annually.
* The Agency should communicate their security documentation to stakeholders of the CloudSystem and ensure stakeholders are notified of subsequent changes.

### System-specific security documentation

#### Applicability to CloudSystem

System-specific security documentation is applicable to the CloudSystem.

#### CloudSystem compliance approach

The CloudSystem includes a suite of security and operational documentation that are logically connected and consistent.

The CloudSystem provides an SSP (this document), SSP Annex, SRMP, IRP, SOPs and other operational documentation to assist in the understanding of the system and the security controls included.

#### Security controls provided by the CloudSystem

* The CloudSystem provides SOPs for administrators and support staff to understand, monitor and operate the CloudSystem-provided security controls.
* The CloudSystem includes detailed design, configuration, operational and support documentation.
* The CloudSystem provides security documentation for input into an Agency’s security processes.

#### Residual controls to be addressed by the Agency

* The Agency should incorporate the ‘DTA – Cloud-Native Blueprint - Incident Response Plan (August 2020)’ that applies to the CloudSystem into their Agency-wide IRP.
* The Agency is responsible for developing a continuous monitoring plan.
* The Agency is responsible for developing a security assessment report including a plan of action post security assessment.

## Physical security

This section does not include specific subsections as the information is the same for all subsections of this chapter.

### Applicability to CloudSystem

Not applicable as the CloudSystem does not contain any physical components, and the security of the Azure and Microsoft 365 hosting equipment is the responsibility of Microsoft and is addressed in the respective IRAP reports for each service.

### CloudSystem compliance approach

The CloudSystem inherits physical security controls from the underlying Azure and Microsoft 365 platforms.

### Security controls provided by the CloudSystem

Not applicable.

### Residual controls to be addressed by the Agency

* The Agency is responsible for the physical security of all Agency owned equipment, such as network devices and endpoint devices, that are utilised to connect to Azure and Microsoft 365.

## Personnel security

This section does not include specific subsections as the information is the same for all subsections of this chapter.

### Applicability to CloudSystem

Not applicable to the CloudSystem as this is an Agency’s responsibility. An Agency’s implementation of personnel security controls should cover the CloudSystem system.

### CloudSystem compliance approach

The CloudSystem provides a role-based access control implementation and associated operations guide to enable an Agency to easily and securely control access, including privileged and emergency access, to Azure and Microsoft 365 services.

### Security controls provided by the CloudSystem

* The CloudSystem provides a framework for identity and access management for Azure and Microsoft 365 resources.
* The CloudSystem provides emergency access or ‘break-glass’ accounts to be used in emergency situations to restore access to an environment or tenant.
* The CloudSystem provides Microsoft Cloud App Security (MCAS) policy monitoring to monitor the activity of the break-glass accounts.

### Residual controls to be addressed by the Agency

* The Agency is responsible for ensuring that personnel undergo pre-employment checks and hold the appropriate level of security clearance, as well as providing cyber security awareness training to staff and contractors.
* The Agency is responsible establishing processes for the creation, maintenance and remediation of counts created within the system in accordance with the controls within the annex.

## Communications infrastructure

This section does not include specific subsections as the information is the same for all subsections of this chapter.

### Applicability to CloudSystem

Not applicable as the CloudSystem does not contain any communications infrastructure, and the security of the Azure and Microsoft 365 hosting equipment is the responsibility of Microsoft and is addressed in the respective IRAP reports for Azure and Microsoft 365.

### CloudSystem compliance approach

The CloudSystem inherits communications infrastructure controls from the underlying Azure and Microsoft 365 platforms.

### Security controls provided by the CloudSystem

Not applicable.

### Residual controls to be addressed by the Agency

* The Agency is responsible for the Agency-owned communication infrastructure utilised to connect to Azure and Microsoft 365.

## Communications systems

### Telephone Systems

#### Applicability to CloudSystem

This section is not applicable as the CloudSystem does not include telephone systems.

#### CloudSystem compliance approach

Not Applicable.

#### Security controls provided by the CloudSystem

Not Applicable.

#### Residual controls to be addressed by the Agency

Not applicable.

### Video Conferencing and IP telephony

#### Applicability to CloudSystem

This section is applicable as the CloudSystem contains Microsoft Teams which provides video conferencing functionality.

#### CloudSystem compliance approach

The CloudSystem inherits the security controls Microsoft have implemented for Microsoft Teams as assessed in the Microsoft 365 IRAP report.

The Annex specifies the controls associated with the self-contained use of Microsoft Teams up to the level of PROTECTED. As per the HybridSystem solution design, agencies have the ability to connect Teams to the Telstra Calling for Microsoft 365 service to allow calling between Teams and traditional telephones. Agencies wishing to use Telstra Calling for Microsoft 365 or another similar services should undertake a security assessment to ensure that the product addresses their security requirements.

#### Security controls provided by the CloudSystem

* Microsoft Teams signalling data is encrypted.
* Secure signalling and data protocols are used by Microsoft Teams including Session Initiation Protocol (SIP) and Secure Real Time Protocol (SRTP).
* Microsoft Teams leverages Azure AD for authentication.
* Microsoft Teams has a dedicated Virtual Local Area Network (VLAN) within the Microsoft cloud.
* Microsoft Teams leverages Azure's Distributed Denial of Service (DDoS) protection capabilities.

#### Residual controls to be addressed by the Agency

* The Agency is responsible for all gateway configurations.

### Fax machines and multifunction devices

#### Applicability to CloudSystem

This section is not applicable as the CloudSystem does not include fax machines or multifunction devices.

#### CloudSystem compliance approach

Not Applicable.

#### Security controls provided by the CloudSystem

Not Applicable.

#### Residual controls to be addressed by the Agency

* The Agency is responsible for the use and management of any fax machines and Multifunction Devices (MFDs) that are used with the CloudSystem.

## Enterprise mobility

### Mobile device management

#### Applicability to CloudSystem

This section is applicable as the CloudSystem includes mobile devices.

#### CloudSystem compliance approach

The CloudSystem leverages Microsoft Endpoint Manager - Intune (Intune) to provide both Mobile Device Management (MDM) Mobile Application Management (MAM) controls to protect mobile devices and data stored on them. Both Windows laptops and iOS devices will be enrolled within Intune and tagged as Corporate devices, allowing policies to be centrally managed and deployed. This includes configuring storage encryption, disabling unneeded features and controlling application behaviour.

iOS devices are configured to align to the intent of the ACSC ‘[Security Configuration Guide - Apple iOS 14 Devices](https://www.cyber.gov.au/acsc/view-all-content/publications/security-configuration-guide-apple-ios-14-devices)’ and the Information Security Manual (ISM). iOS devices within the blueprint limit user access and some usability in order to reach a higher security and compliance level.

Note: Agencies should conduct a threat and risk assessment prior to changing the described configuration of mobile devices.

Bluetooth is enabled as it allows users to pair devices they may required to perform their duties (e.g. conference calls or online meetings).

Users can reset certain security settings in Personal Hotspot and Passcode for situations where the passcode/password may have been compromised.

All iOS applications must be deployed using Microsoft Endpoint Manager with volume licensing. Users are not able to access the App store to update or download applications. All updates are managed via Intune update policies. The agency can track, and monitor apps found on managed devices that have been enrolled in Intune. Details on how to view and report on discovered apps can be found in the ‘DTA - Cloud-Native Blueprint - Security Standard Operating Procedures (August 2020)’.

The risk of supervised iOS devices is addressed in the [DTA - Cloud-Native Blueprint - Security Risk Management Plan (June 2021)](security-risk-management-plan.html) at [R17](security-risk-management-plan.html#r17-mobile-phone-compromised-due-to-loss-or-theft) and [R20](security-risk-management-plan.html#r20-mobile-phone-exploited)

#### Security controls provided by the CloudSystem

* Microsoft Intune provides MDM and MAM capability.
* The CloudSystem does not include the use of privately-owned mobile devices. Only Agency-owned devices are enrolled and allowed to access data.
* The CloudSystem provides Windows 10 for laptops which is hardened in accordance with ACSC guidance. The CloudSystem also provides MDM for iOS but does not fully implement ACSC's guidance for PROTECTED.
* Microsoft BitLocker provides full disk encryption of CloudSystem mobile devices, implementing Advanced Encryption Standard (AES)-256. Additionally, iOS devices implement AES-256 encryption by default.
* All information transmitted to and from mobile devices and Microsoft 365 is encrypted.
* Bluetooth is disabled on Windows 10 devices. Bluetooth is not managed for iOS devices.
* CloudSystem standard users do not have sufficient permissions to install or uninstall applications on Windows 10 devices. Standard users do not have sufficient permissions to install or uninstall applications on iOS devices. Applications within iOS are centrally managed. 
* Intune will monitor and report installed iOS applications on any company-owned device.
* CloudSystem standard users do not have sufficient permissions to modify security functions on Windows 10 devices. Standard users can modify security functions on iOS devices.
* Apple provides timely security updates for iOS devices.
* The CloudSystem specifies the use of per-application and per-account VPN configurations for iOS devices. Agencies are responsible for configuring these with their chosen VPN solution and configuration.

#### Residual controls to be addressed by the Agency

* The Agency is responsible for developing a mobile device management policy in relation to the CloudSystem that meets requirements outlined in the Annex.

### Mobile device usage

#### Applicability to CloudSystem

This section is applicable as the CloudSystem may contain mobile devices.

#### CloudSystem compliance approach

The CloudSystem is reliant on the Agency to development and enforce a mobile device usage policy which include mobiles devices that are enrolled into the CloudSystem.

#### Security controls provided by the CloudSystem

Not Applicable.

#### Residual controls to be addressed by the Agency

* The Agency is responsible for developing and enforcing a mobile device usage policy in relation to the CloudSystem that meets requirements outlined in the Annex.

## Evaluated products

This section does not include specific subsections as the information is the same for all subsections of this chapter.

### Applicability to CloudSystem

The CloudSystem includes Windows 10 which has been evaluated and therefore the controls relating to evaluated products are applicable to the CloudSystem. No high assurance products are used by the CloudSystem, Azure or Microsoft 365.

### CloudSystem compliance approach

A Protection Profile (PP) evaluation has been performed on Windows 10 and Microsoft publish deployment and administration guides for each [evaluated operating system](https://docs.microsoft.com/en-us/windows/security/threat-protection/windows-platform-common-criteria). The CloudSystem implements the recommendations for the latest evaluated version of Windows 10 (May 2019 Update). This includes sourcing installation media directly from Microsoft and implementing configuration hardening.

### Security controls provided by the CloudSystem

* The CloudSystem includes Windows 10 which has been evaluated against the relevant Protection Profile.
* Windows 10 installation media is sourced directly from Microsoft in accordance with the evaluated delivery procedures.
* Windows 10 is managed by Microsoft Endpoint Manager -  Intune (Intune) in accordance with the published guidance from Microsoft as well the ACSC's hardening guide for Windows 10.

### Residual controls to be addressed by the Agency

* The Agency is responsible for any evaluated products if they are implemented as part of network connectivity to Azure and Microsoft 365.

## ICT equipment management

This section does not include specific subsections as the information is the same for all subsections of this chapter.

### Applicability to CloudSystem

The security of the Azure and Microsoft 365 hosting equipment is the responsibility of Microsoft and is addressed in the respective IRAP reports for Azure and Microsoft 365.

### CloudSystem compliance approach

The CloudSystem inherits ICT equipment controls from the underlying Azure and Microsoft 365 platforms.

### Security controls provided by the CloudSystem

Not applicable.

### Residual controls to be addressed by the Agency

* The Agency is responsible for the implementation of controls relating to ICT equipment management based on their deployment of the CloudSystem.

## Media management

### Media usage

#### Applicability to CloudSystem

This section is applicable as removable media may be connected to CloudSystem endpoints.

#### CloudSystem compliance approach

The CloudSystem implements technical controls to restrict access to removeable media devices that may be connected to CloudSystem endpoints.

#### Security controls provided by the CloudSystem

* Autorun is disabled for removable media via Microsoft Endpoint Manager -  Intune (Intune) policies.
* Only authorised devices that are permitted in Intune policies can be connected to Cloud-Only endpoints. Unauthorised devices will not be mounted to the operating system.
* External connections relying on Direct Memory Access (DMA) will be disabled via Intune policies
* Removable media is encrypted via BitLocker using AES-256.

#### Residual controls to be addressed by the Agency

* The Agency is responsible for implementing controls relating to media management if media is connected to the CloudSystem.

### Media sanitisation

#### Applicability to CloudSystem

The controls relating to the sanitisation of media are not applicable to the CloudSystem and are instead the responsibility of the Agency.

#### CloudSystem compliance approach

Not applicable.

#### Security controls provided by the CloudSystem

Not applicable.

#### Residual controls to be addressed by the Agency

* The Agency is responsible for the management, including sanitisation, of media connected to CloudSystem endpoints.

### Media destruction

#### Applicability to CloudSystem

The controls relating to the destruction of media are not applicable to the CloudSystem and are instead the responsibility of the Agency.

#### CloudSystem compliance approach

Not applicable.

#### Security controls provided by the CloudSystem

Not applicable.

#### Residual controls to be addressed by the Agency

* The Agency is responsible for the management, including destruction, of media connected to CloudSystem endpoints.

### Media disposal

#### Applicability to CloudSystem

The controls relating to the disposal of media are not applicable to the CloudSystem and are instead the responsibility of the Agency.

#### CloudSystem compliance approach

Not applicable.

#### Security controls provided by the CloudSystem

Not applicable.

#### Residual controls to be addressed by the Agency

* The Agency is responsible for the management, including disposal, of media connected to CloudSystem endpoints.

## System hardening

### Operating system hardening

#### Applicability to CloudSystem

Operating system hardening is applicable to the CloudSystem.

#### CloudSystem compliance approach

The CloudSystem will utilise Windows 10 as the endpoint operating system, provided by the Original Equipment Manufacturer (OEM), and then use the Software Updates component of Microsoft Endpoint Manager - Intune (Intune) to maintain the latest version of the operating system.

The CloudSystem will harden the operating system configuration using a combination of Intune policies to implement ACSC and vendor guidance. These Intune policies achieve the results that would traditionally be performed by group policies. Local administrator accounts and guest accounts will be disabled and renamed via Intune policy.

The potential attack surface will be minimised by only including required components and apps, removing and disabling the components that aren’t needed. Standard users will be prevented from running all script execution engines. The CloudSystem will install applications via Intune and not allow standard users the ability to install applications.

The CloudSystem will use Windows Defender Application Control (WDAC) to perform application control. WDAC is the latest system from Microsoft for application control and works in a very similar manner to AppLocker. In addition to performing all functions of AppLocker, WDAC is also able to control plug-ins, add-ins, modules and code at the kernel level as well as the user level.

Enhanced Mitigation Experience Toolkit (EMET) is not supported by the latest release of Windows 10 and all functionality of EMET has been incorporated into Windows Defender Exploit Guard which is fully configured.

Windows Defender Firewall is enabled as part of the CloudSystem Windows 10 Standard Operating Environment (SOE) and configured by Intune policies. Windows Defender Antivirus and Microsoft Defender ATP provide antivirus including signature, reputation and heuristic-based detection. 

Scanning frequency for both quick scans and full scans is determined by the policies and occurs for fixed and removable drives.

Endpoint Device Control will be configured by Intune policies restricting usage to only permitted devices.

#### Security controls provided by the CloudSystem

* Windows 10 Semi-Annual Channel (SAC) is used as the SOE for the CloudSystem.
* The 64-bit version of Windows 10 is used as the SOE for the CloudSystem.
* The Windows 10 SOE has been hardened in accordance with ACSC guidance where possible using Intune.
* The default administrator and guest accounts have been disabled and renamed.
* Additional administrators can be added to endpoints via Azure AD groups, with configuration changes pulled to devices via Intune.
* Intune and Microsoft Endpoint Configuration Manager (MECM) policies prevent standard users from running cmd.exe however can only restrict the PowerShell execution policy to RemoteSigned.
* RBAC policy defines separate domain and local administrator roles. Standard users do not have permissions to install or uninstall software.
* WDAC provides application control functionality. A combination of hash, publisher certificate and path rules will be used by WDAC for control of applications. Both publisher and product names are used by WDAC for control of applications. WDAC writes to the local event log. Standard users cannot disable application control within the system.
* File permissions prevent standard users from writing to locations that are whitelisted using path rules.
* [Microsoft's recommended block rules](https://docs.microsoft.com/en-us/windows/security/threat-protection/windows-defender-application-control/microsoft-recommended-block-rules) to prevent known WDAC bypasses are implemented.
* The 'Exploit protection' feature is enabled as part of the CloudSystem Windows 10 SOE.
* Windows Defender Exploit Guard and Defender ATP provide HIPS functionality as part of the CloudSystem Windows 10 SOE.
* Windows Defender Firewall is enabled as part of the CloudSystem Windows 10 SOE.
* Defender Antivirus and Defender ATP provide antivirus including signature and heuristic-based detection. Reputation rating features are enabled.
* Intune provides device access control by DeviceID or Device Class.
* Windows Defender Firewall has been configured via Microsoft Endpoint Manager - Intune configuration profiles to block all non-approved inbound or outbound traffic from Windows 10 endpoints.

#### Residual controls to be addressed by the Agency

* Where Agencies utilise SOE developed by third parties, the Agency must ensure that the SOE is scanned for malicious content and configurations before being used and that the design is reviewed and updated at least annually.
* The Agency must validate cryptographic hash rules, publisher certificate rules and path rules used for application control at least annually.

### Application hardening

#### Applicability to CloudSystem

Application hardening is applicable to the CloudSystem.

#### CloudSystem compliance approach

The CloudSystem compliance approach is to select native cloud capabilities that are hardened and maintained as part of the service.

The CloudSystem utilises the Monthly Targeted Channel of Microsoft to ensure the latest versions of software are used. Where third party applications are used these are also targeted at the most recent versions. Software update policies are configured to update plugins, browsers and applications regularly ensuring endpoints are using most recent versions.

ACSC guidance has been incorporated into the applications to harden the configuration and remove unneeded features.

Web browsers are configured to block Flash content and Java content by the Intune policies and Java can be selected by exception for non-internet sites. The CloudSystem does not include a web advertisements blocker to reduce the reliance on third party applications.

Microsoft 365 macros sourced from the internet are blocked and only signed macros will be allowed to execute. Additional macro controls are configured in the Attack Surface Rules configuration controlled by Intune. Users are not able to change macro settings.

#### Security controls provided by the CloudSystem

* All applications are supplied by Microsoft which has made a commitment to secure development. The CloudSystem does not include any third-party applications.
* The latest version of Microsoft 365 is installed.
* ACSC guidance has been implemented to harden Microsoft and built-in web browsers.
* Flash is blocked in both Edge and Internet Explorer.
* Flash and Java-based web advertisements are blocked in Edge and Internet Explorer.
* Java is blocked in both Edge and Internet Explorer.
* Support for Flash content is disabled by default.
* Object Linking and Embedding (OLE) is blocked for Microsoft 365.
* Unrequired functionality, such as Microsoft Access, has been removed.
* The use of add-ons is restricted to Microsoft-provided add-ons only.
* Microsoft's Attack Surface Reduction Rules are implemented in the configuration controlled by Intune.
* Only signed macros are enabled.
* All macros downloaded from the internet are disabled.
* Users cannot change macro settings.

#### Residual controls to be addressed by the Agency

* The Agency is responsible for hardening any third-party browsers (e.g. Google Chrome) that are deployed to CloudSystem endpoints. The [United Kingdom Government provides guidance on hardening Chrome](https://assets.publishing.service.gov.uk/government/uploads/system/uploads/attachment_data/file/380011/Browser_Security_Guidance_-_Google_Chrome.pdf) specifically which Agencies may choose to follow.

### Authentication hardening

#### Applicability to CloudSystem

The authentication hardening section is applicable to the CloudSystem.

#### CloudSystem compliance approach

The CloudSystem leverages Azure AD for controlling system access. All technical capabilities that the CloudSystem performs are completed through Application and Service Principal objects in Azure AD, which utilise certificate-based authentication.

The CloudSystem utilises RBAC, automation and policy controls to restrict access to modify any of the functional capabilities provided by the CloudSystem. The CloudSystem also provides auditing and alerting on attempted or successful modifications of CloudSystem capabilities. 

The CloudSystem provides security controls and an identity management framework that can be utilised to manage system access for systems deployed within the CloudSystem. The CloudSystem enforces multi-factor authentication through conditional access policies, creates recovery accounts for maintaining access to resources and enforces password policies for accounts created directly in Azure AD. The CloudSystem uses a soft-token to reduce the need for purchase, distribution and management of hard-tokens.

The CloudSystem utilises Azure AD to store groups utilised for RBAC and provides process and administration documentation for managing access to Azure resources.

#### Security controls provided by the CloudSystem

* Azure AD is configured to require all users to be authenticated before granting access.
* Azure MFA is enforced for all standard and privileged users.
* MFA requires complex password and One Time Password (OTP) from Microsoft Authenticator App (soft token).
* Azure AD password complexity enforces a minimum character length of 14 characters.
* None of the authentication factors on their own can be used for single-factor authentication to another system.
* Azure MFA is enforced for all users accessing Microsoft 365 content.
* Azure AD Smart Lockout is configured to lock account after five failed logon attempts.
* Azure AD self-service password reset requires users to verify their identity before resetting their password in accordance with password complexity requirements.
* Local Area Network (LAN) Manager is not used by the CloudSystem, however New Technology LAN Manager (NTLM) is used within the system.
* Credentials are stored within Azure AD. Azure AD Identity Protection is enabled to detected leaked passwords.
* The CloudSystem Windows 10 SOE is configured with a screen saver after 15 minutes which requires users to re-authenticate.
* The CloudSystem Windows 10 SOE is configured with a logon banner provided by the Agency.

#### Residual controls to be addressed by the Agency

* The Agency is responsible for investigating repeated account lockouts.
* The Agency is responsible for managing passwords/passphrases.
* The Agency is responsible for procedures involving provisioning user passwords.
* The Agency is responsible for terminating user sessions and rebooting workstations outside of business hours.

### Virtualisation hardening

#### Applicability to CloudSystem

This section is not applicable as the CloudSystem does not include servers which are not managed by Microsoft.

#### CloudSystem compliance approach

Not Applicable.

#### Security controls provided by the CloudSystem

* All in scope servers are provided by Microsoft and secured in line with the controls outlined in the Microsoft 365 and Azure IRAP Assessments.

#### Residual controls to be addressed by the Agency

Not applicable.

## System management

### System administration

#### Applicability to CloudSystem

The system administration section is applicable to the CloudSystem in the context of the operations and management of the controls that the CloudSystem provides.

#### CloudSystem compliance approach

Privileged Access Workstations (PAWs) and admin jump servers are not used in the CloudSystem due to the limited size of the expected agencies and all administrative access to the Microsoft portals is with Azure AD accounts using MFA. The risk of not implementing these controls is addressed in the CloudSystem SRMP.

All administration of the CloudSystem is performed through a web browser to a number of Microsoft 365 portals as listed in Table 4.

Table 4 Microsoft Management Portals

Portal | URL
--- | ---
Microsoft Defender ATP portal | [https://securitycenter.windows.com](https://securitycenter.windows.com)
Cloud App Security portal | [https://portal.cloudappsecurity.com](https://portal.cloudappsecurity.com)
Azure portal (including Azure AD) | [https://portal.azure.com](https://portal.azure.com)
Microsoft 365 Compliance Center | [https://compliance.microsoft.com](https://compliance.microsoft.com)
Microsoft 365 Security Center | [https://security.microsoft.com](https://security.microsoft.com)
Microsoft 365 homepage | [https://portal.office.com](https://portal.office.com)

The CloudSystem protects access to these portals through authentication via Azure AD and enforcement of MFA and location-based policies through Conditional Access. Privileges within the CloudSystem are controlled through the CloudSystem RBAC model.

The Conditional Access policies and RBAC model also extend to the administration of endpoint devices that are deployed as part of the CloudSystem.

#### Security controls provided by the CloudSystem

* The CloudSystem includes a system administration SOP.
* Azure MFA is required for all privileged user access.

18.1.4 Residual controls to be addressed by the Agency

* The Agency is responsible for provisioning, managing and decommissioning administrative accounts to be used for CloudSystem administration.

### System patching

#### Applicability to CloudSystem

System patching of Microsoft 365 and Azure AD are not applicable as these cloud components are a Microsoft responsibility.

System patching of endpoint devices is required, and this is accomplished via Intune policies setting the frequency, installation options and reporting values.

#### CloudSystem compliance approach

The CloudSystem compliance approach is to primarily utilise native cloud capabilities that are patched as part of the service. 

For patching endpoint devices deployed by the Agency, the CloudSystem provides configuration of which patches to apply, deferral periods, update behaviour and reporting.

#### Security controls provided by the CloudSystem

* The CloudSystem provides the types of updates that are applied to endpoints.
* The CloudSystem configures the intervals for checking for new updates.
* The CloudSystem provides the reporting of device status to determine which devices have received updates.
* The CloudSystem includes a system administration SOP which specifically references patching.
* All configurations are included in the relevant ABAC.
* Application and driver patches will be automatically deployed via Intune for the CloudSystem Windows 10 SOE.
* Operating system patches will be automatically deployed via Intune for the CloudSystem Windows 10 SOE. 
* Intune provides a centralised and managed approach to patching.
* Windows Update verifies the integrity of patches before installing them.

#### Residual controls to be addressed by the Agency

* The Agency is responsible for system patching for all systems deployed within the CloudSystem.
* The Agency is responsible for any firmware patching dependent on the specific hardware model chosen by them.
* The Agency is responsible for viewing the reporting and alerts and rectification of faults as they occur.

### Change management

#### Applicability to CloudSystem

Change management is not applicable as the ongoing management and maintenance of the CloudSystem utilises the Agency’s change management process.

#### CloudSystem compliance approach

The CloudSystem integrates with an Agency’s existing change management process. 

#### Security controls provided by the CloudSystem

Not applicable.

#### Residual controls to be addressed by the Agency

* The Agency is responsible for all change management processes.

### Data backups

#### Applicability to CloudSystem

Data backups are not applicable to the CloudSystem as they are the responsibility of the Agency to implement in accordance with their data preservation strategy.

#### CloudSystem compliance approach

The Agency is responsible for backup and restoration of data and configurations stored in the CloudSystem. 

#### Security controls provided by the CloudSystem

Not applicable.

#### Residual controls to be addressed by the Agency

* The Agency is responsible for data backups data and configurations stored in the CloudSystem.

## System monitoring

### Event logging and auditing

#### Applicability to CloudSystem

The controls relating to the logging and auditing of events for components included in the CloudSystem are applicable. The CloudSystem does not include web applications, databases, Domain Name System (DNS) or proxy services, and therefore the controls relating to these components are not applicable.

#### CloudSystem compliance approach

The CloudSystem provides extensive event logging and auditing for Azure and Microsoft 365 resources that can be incorporated into an Agency’s event logging strategy. Logs are stored in Log Analytics for two years which is the maximum available period for Log Analytics. 

All logs relevant to the operation and integrity of the CloudSystem are stored in a centralised storage account. The CloudSystem protects the integrity of logs through policy enforcement, automation and RBAC.

Local event logs on Windows 10 devices will be lost when endpoints are rebuilt as the local event logs are not centralised.

#### Security controls provided by the CloudSystem

* Microsoft Defender ATP and Microsoft 365 ATP centralise logs relating to the security of devices and Microsoft services respectively.
* Windows Time is used as the time source for all CloudSystem components.
* Azure AD sign-in and audit logs are centralised by Log Analytics.
* Update Management and Security Center logs are centralised by Log Analytics.
* The following events are logged to the local event log on each Windows 10 endpoint:
  * access to important data and processes
  * application crashes and any error messages
  * attempts to use special privileges
  * changes to accounts
  * changes to security policy
  * changes to system configurations
  * DNS and Hypertext Transfer Protocol requests
  * failed attempts to access data and system resources
  * service failures and restarts
  * system startup and shutdown
  * transfer of data to external media
  * user or group management
  * use of special privileges.
* Logs include the date and time of the event, the relevant user or process, the event description, and the ICT equipment involved are recorded.
* Logs stored in Log Analytics are protected from unauthorised access, modification and deletion by the Azure AD RBAC model. Standard Windows 10 users don’t have access to modify the local event logs.

#### Residual controls to be addressed by the Agency

* The Agency is responsible for developing and implementing an event logging policy. 
* The Agency is responsible for centralising local event logs from Windows 10 endpoints if required by their event logging policy.

## Software development

This section does not include specific subsections as the information is the same for all subsections of this chapter.

### Applicability to CloudSystem

Not applicable as the CloudSystem is not designed to support software development activities.

#### CloudSystem compliance approach

Not applicable.

##### Security controls provided by the CloudSystem

Not applicable.

#### Residual controls to be addressed by the Agency

* The Agency is responsible for all application development controls but can leverage the CloudSystem security controls detailed in this document.

## Database systems management

This section does not include specific subsections as the information is the same for all subsections of this chapter.

### Applicability to CloudSystem

Not applicable as the CloudSystem does not include any database servers, database management system software or databases.

### CloudSystem compliance approach

Not applicable.

### Security controls provided by the CloudSystem

Not applicable.

### Residual controls to be addressed by the Agency

Not applicable.

## Email management

### Email usage

#### Applicability to CloudSystem

The controls relating to email usage are applicable to the CloudSystem as it provides an email capability of its users.

#### CloudSystem compliance approach

The CloudSystem provides the capability for users to apply protective markings to emails based on their classification. If required, users have the ability to lower the classification of an email, but are required to provide a text-based justification that is included in the audit log. 

The CloudSystem will leverage an Agency's Secure Internet Gateway (SIG) for proxy and mail services.

#### Security controls provided by the CloudSystem

* The CloudSystem applies protective markings in accordance with the Protective Security Policy Framework (PSPF) based on the classification of the content of emails, including attachments.
* Users are required to select the classification of emails to apply protective markings.
* Only appropriate classification options will be presented to CloudSystem users.
* Microsoft 365 ATP will notify users and administrators of blocked emails.

#### Residual controls to be addressed by the Agency

* The Agency is responsible for developing and implementing an email usage policy.
* The Agency is responsible for ensuring their email gateway blocks, logs and reports on emails with inappropriate protective markings.
* The Agency is responsible for conducting a risk assessment if the Agency chooses not to use a SIG.

### Email gateways and servers

#### Applicability to CloudSystem

The controls relating to email gateways and servers are applicable to the CloudSystem as it leverages Exchange Online.

#### CloudSystem compliance approach

The CloudSystem leverages Exchange Online, part of Microsoft 365, to provide email capability without the need to deploy traditional email servers and gateways. Native Exchange Online security capabilities are enabled to prevent against email-related threats such as spoofing and phishing. Exchange Online is configured to route through the Agency’s existing email gateway.

The advanced features of Microsoft 365 ATP, including Safe Attachments and Safe Links which provide sandboxing of attachments and inspective of hyperlinks respectively, are enabled by the CloudSystem. This provides email content filtering and expands on the default protections offered by Exchange Online Protection (EOP).

#### Security controls provided by the CloudSystem

* Exchange Online is configured to route through the Agency's existing email gateway.
* Email traffic between external users and Exchange Online is encrypted with TLS 1.2. Exchange Online then forwards emails to the Agency's existing email gateway via an Exchange connector.
* Exchange Online is not configured to act as an open relay.
* Sender Policy Framework (SPF) is configured in Exchange Online using a hard fail record. SPF blocks are visible to the recipients.
* DomainKeys Identified Mail (DKIM) is configured in Exchange Online and DKIM signatures on received emails are verified.
* Domain-based Message Authentication, Reporting and Conformance (DMARC) records are configured in Exchange Online.
* Microsoft 365 ATP provides content filtering including sandboxing of attachments (Smart Attachments) and inspection of links (Smart Links).

#### Residual controls to be addressed by the Agency

* The Agency is responsible for controls implemented by their existing email gateway.
* The Agency is responsible for any backup or alternative email gateways.

## Network management

### Network design and configuration

#### Applicability to CloudSystem

The majority of the controls relating to network design and configuration are not directly applicable to the CloudSystem and are instead the responsibility of the Agency to implement. This is due to the CloudSystem not including network devices within its scope.

#### CloudSystem compliance approach

The CloudSystem leverages the Microsoft backbone to provide networking for the Microsoft 365 and Azure services. LAN design and configuration is the responsibility of the Agency and may reuse existing capabilities. The CloudSystem designs the interfaces between endpoints and services, including how data traverses’ public networks such as the internet.

#### Security controls provided by the CloudSystem

* The Microsoft 365 design which includes the high-level network design has a document control table listing the last update date.
* Conditional access policies are used to restrict inbound network traffic to specific geographic regions.
* MCAS policies are used to analyse sign in logs to identify and subsequently notify administrators when users are identified as originating from anonymous proxy IP addresses.

#### Residual controls to be addressed by the Agency

* The Agency is responsible for the management of network devices used in relation to the CloudSystem.
* The Agency is responsible for implementing security controls within their email gateway.
* The Agency is responsible for ensuring that they segregate their network from that of service providers.
* The Agency is responsible for tuning conditional access policy to limit inbound network connections from anonymity networks.
* The Agency is responsible for tuning MCAS policies to meet their requirements. Further, the Agency must implement appropriate procedures around these alerts.

### Wireless networks

#### Applicability to CloudSystem

The controls relating to wireless networks are not applicable as the CloudSystem does not include any wireless networks.

#### CloudSystem compliance approach

Not applicable.

#### Security controls provided by the CloudSystem

Not applicable.

#### Residual controls to be addressed by the Agency

* The Agency is responsible for securing any wireless networks that they provide to enable connectivity between CloudSystem endpoints and Azure/Microsoft 365 services.

### Service continuity for online services

#### Applicability to CloudSystem

The majority controls relating to service continuity for online services controls are not applicable to the CloudSystem as it does not host online services. As the CloudSystem is dependent on online services hosted by Microsoft, some controls are applicable.

#### CloudSystem compliance approach

Microsoft is the service provider for the online services used by the CloudSystem, specifically Azure and Microsoft 365. The CloudSystem inherits Microsoft’s implementation of controls to mitigate this risk of Denial of Service (DoS) events targeting their services. 

#### Security controls provided by the CloudSystem

Not applicable.

#### Residual controls to be addressed by the Agency

* The CloudSystem does not host online services. The Agency is responsible for the procurement and management of online services.
* The Agency is responsible for documenting the functionality and quality of services, how to maintain such functionality, and what functionality can be lived without during a denial-of-service attack in relation to the Microsoft services used by the CloudSystem.
* The Agency is responsible for discussing denial-of-service attack prevention and mitigation strategies with Microsoft as the service provider for Azure and

## Using cryptography

### Cryptographic fundamentals

#### Applicability to CloudSystem

The controls relating to cryptographic fundamentals are applicable to the CloudSystem.

#### CloudSystem compliance approach

The CloudSystem leverages cryptography provided by Microsoft to encrypt both data at rest and data in transit. This includes the use of Microsoft BitLocker to encrypt mobile devices using an Australian Signals Directorate (ASD) Approved Cryptographic Algorithm (AACA), namely AES. Note, that the CloudSystem does not use encryption for the purposes of reducing the handling requirements for endpoints.

Microsoft’s implementation of cryptography, including TLS 1.2 which is an ASD Approved Cryptographic Protocol (AACP), has been assessed as part of the IRAP assessments for Azure and Microsoft 365. However, an ASD Cryptographic Evaluation (ACE) has not been performed on Microsoft’s cryptographic software.

At the time of writing Microsoft does not support the latest version of TLS – version 1.3. Microsoft have previously stated that versions 1.0 and 1.1 are not supported and were to become deprecated for Microsoft 365 services from June 2020, however this has since been delayed due to world events. See [Preparing for TLS 1.2 in Office 365 and Office 365 GCC](https://docs.microsoft.com/en-us/office365/troubleshoot/security/prepare-tls-1.2-in-office-365).

#### Security controls provided by the CloudSystem

* The CloudSystem uses Microsoft BitLocker for encryption leveraging AES which is an AACA.
* Microsoft BitLocker provides full disk encryption of CloudSystem mobile devices, implementing AES-256. BitLocker recovery keys are stored in Azure AD.
* TLS with AES is used to protect traffic to and from Azure and Microsoft 365 servers over the internet.

#### Residual controls to be addressed by the Agency

* The Agency is responsible for informing users of their responsibilities in relation to the management encrypted devices.

### ASD approved cryptographic algorithms

#### Applicability to CloudSystem

The controls relating to AACAs are applicable to the CloudSystem.

#### CloudSystem compliance approach

The CloudSystem leverage’s Microsoft’s implementation of AACAs in Azure and Microsoft 365.

#### Security controls provided by the CloudSystem

* Microsoft Azure and Microsoft 365 services implement AACAs where possible.
* Microsoft Azure and Microsoft 365 services implement Elliptic Curve Diffie-Hellman Ephemeral (ECDHE) as the preferred algorithm.
* Microsoft Azure and Microsoft 365 services do not use Diffie-Hellman (DH).
* Microsoft Azure and Microsoft 365 services do not use Digital Signature Algorithm (DSA).
* Microsoft Azure and Microsoft 365 services implement National Institute of Standards and Technology (NIST) P-256 and P-384.
* Microsoft Azure and Microsoft 365 services use a 256-bit key where possible for Elliptic Curve Diffie-Hellman (ECDH).
* Microsoft Azure and Microsoft 365 services use a 2048-bit key for Rivest–Shamir–Adleman (RSA).
* Microsoft Azure and Microsoft 365 services use separate RSA key pairs for these purposes.
* Microsoft Azure and Microsoft 365 services use Secure Hash Algorithm (SHA)-256 for hashing.
* Microsoft Azure and Microsoft 365 services do not use Electronic Codebook Mode (ECM).
* Microsoft Azure and Microsoft 365 services do not use Triple Data Encryption Standard (3DES).

#### Residual controls to be addressed by the Agency

Not applicable.

### ASD approved cryptographic protocols

#### Applicability to CloudSystem

The controls relating to AACPs are applicable to the CloudSystem.

#### CloudSystem compliance approach

The CloudSystem leverage’s Microsoft’s implementation of AACPs in Azure and Microsoft 365.

#### Security controls provided by the CloudSystem

* Microsoft Azure and Microsoft 365 services implement AACAs where possible.

#### Residual controls to be addressed by the Agency

Not applicable.

### Transport layer security

#### Applicability to CloudSystem

The controls relating to TLS are applicable to the CloudSystem.

#### CloudSystem compliance approach

The CloudSystem leverage’s Microsoft’s implementation of TLS in Azure and Microsoft 365.

#### Security controls provided by the CloudSystem

* Microsoft Azure and Microsoft 365 services implement TLS versions 1.2 and 1.3.
* Microsoft Azure and Microsoft 365 services implement AES in Galois Counter Mode (GCM).
* Microsoft Azure and Microsoft 365 services implement secure renegotiation.
* Microsoft Azure and Microsoft 365 services implement ECDHE as the preferred algorithm.
* Microsoft Azure and Microsoft 365 services use SHA-2-based certificates.
* Microsoft Azure and Microsoft 365 services use SHA-2 as part of the Message Authentication Code and Pseudo-Random Function.
* Microsoft Azure and Microsoft 365 services disable TLS compression.
* Microsoft Azure and Microsoft 365 services implement Perfect Forward Secrecy (PFS).

#### Residual controls to be addressed by the Agency

Not applicable.

### Secure shell

#### Applicability to CloudSystem

The controls relating to the use of Secure Shell (SSH) are not applicable to the CloudSystem as it does not utilise SSH. 

#### CloudSystem compliance approach

Not applicable.

#### Security controls provided by the CloudSystem

Not applicable.

#### Residual controls to be addressed by the Agency

Not applicable.

### Secure/multipurpose internet mail extension

#### Applicability to CloudSystem

The controls relating to the use of Secure/Multipurpose Internet Mail Extension (S/MIME) are not applicable to the CloudSystem as it does not utilise S/MIME. 

#### CloudSystem compliance approach

Not applicable.

#### Security controls provided by the CloudSystem

Not applicable.

#### Residual controls to be addressed by the Agency

Not applicable.

### Internet protocol security

#### Applicability to CloudSystem

The controls relating to the use of Internet Protocol Security (IPsec) are not applicable to the CloudSystem as it does not utilise IPsec. 

#### CloudSystem compliance approach

Not applicable.

#### Security controls provided by the CloudSystem

Not applicable.

#### Residual controls to be addressed by the Agency

Not applicable.

### Cryptographic system management

#### Applicability to CloudSystem

The controls relating to cryptographic system management is not applicable to the CloudSystem as the CloudSystem does not include the use of Commercial Grade Cryptographic Equipment (CGCE) equipment.

#### CloudSystem compliance approach

Not applicable.

#### Security controls provided by the CloudSystem

Not applicable.

#### Residual controls to be addressed by the Agency

* The Agency is responsible for the management of any CGCE used in relation to the CloudSystem.

## Gateway management

### Gateways

#### Applicability to CloudSystem

The controls relating to gateways are applicable to the CloudSystem as the solution is designed to integrate with a SIG provided by the Agency.

#### CloudSystem compliance approach

The CloudSystem leverages the Agency’s SIG capability where required.

#### Security controls provided by the CloudSystem

Not applicable.

#### Residual controls to be addressed by the Agency

* The Agency is responsible for the implementation of security controls relating to their internet and email gateway if integrated with the CloudSystem.

### Cross domain solutions

#### Applicability to CloudSystem

The controls relating to cross-domain solutions are not applicable as the CloudSystem does not include any cross-domain solutions.

#### CloudSystem compliance approach

Not applicable.

#### Security controls provided by the CloudSystem

Not applicable.

#### Residual controls to be addressed by the Agency

Not applicable.

### Firewalls

#### Applicability to CloudSystem

The controls relating to firewalls are not applicable to the CloudSystem as the CloudSystem does not include firewalls for the purpose of separating official/classified and public networks. 

#### CloudSystem compliance approach

Not applicable.

#### Security controls provided by the CloudSystem

Not applicable.

#### Residual controls to be addressed by the Agency

* The Agency is responsible for the implementation of security controls relating to their email gateway if integrated with the CloudSystem.

### Diodes

#### Applicability to CloudSystem

The controls relating to diodes are not applicable to the CloudSystem as the CloudSystem does not include any diodes or unidirectional gateways.

#### CloudSystem compliance approach

Not applicable.

#### Security controls provided by the CloudSystem

Not applicable.

#### Residual controls to be addressed by the Agency

Not applicable.

### Web proxies

#### Applicability to CloudSystem

The controls relating to web proxies are applicable to the CloudSystem as the solution leverages the Agency’s SIG for proxy services. 

#### CloudSystem compliance approach

The CloudSystem does not include a web proxy service.  If an Agency's risk profile requires a web proxy service, the DTA recommend the use of a certified SIG provider.

#### Security controls provided by the CloudSystem

Not applicable.

#### Residual controls to be addressed by the Agency

* The Agency is responsible for the development and implementation of a web usage policy.
* The Agency is responsible for controls relating to web proxies as required by their risk profile.

### Web content filters

#### Applicability to CloudSystem

The controls relating to web content filters are applicable to the CloudSystem as the solution leverages an Agency’s SIG for web content filtering. 

#### CloudSystem compliance approach

The CloudSystem does not include a web content filtering service.  If an Agency's risk profile requires a web content filtering service, the DTA recommend the use of a certified SIG provider.

#### Security controls provided by the CloudSystem

Not applicable.

#### Residual controls to be addressed by the Agency

* The Agency is responsible for controls relating to web content filtering as required by their risk profile.

### Content filtering

#### Applicability to CloudSystem

The controls relating to content filtering are applicable to the CloudSystem in relation to the filtering of email content. 

#### CloudSystem compliance approach

The CloudSystem leverages Microsoft 365 capabilities including Microsoft 365 ATP and EOP to inspect and manage email traffic. Content validation is not performed.

#### Security controls provided by the CloudSystem

* Exchange Online Protection and Microsoft 365 ATP prevent specific file types from entering the system via email.
* Microsoft 365 ATP provides content filtering including sandboxing of attachments (Smart Attachments) and inspection of links (Smart Links).
* Multiple scanning engines are provided by Exchange Online Protection, Microsoft 365 ATP and Defender ATP.
* Archives are scanned for malware.
* Microsoft 365 ATP alerts are configured.
* Integrity of patches is verified before installation.
* Microsoft 365 ATP provides content filtering including sandboxing of attachments (Smart Attachments) and inspection of links (Smart Links).

#### Residual controls to be addressed by the Agency

* The Agency is responsible for any additional content filtering controls required based on their risk profile.

### Peripheral switches

#### Applicability to CloudSystem

The controls relating to peripheral switches are not applicable to the CloudSystem as the CloudSystem does not include any peripheral switches.

#### CloudSystem compliance approach

Not applicable.

#### Security controls provided by the CloudSystem

Not applicable.

#### Residual controls to be addressed by the Agency

Not applicable.

## Data transfers

### Applicability to CloudSystem

The controls relating to data transfers are applicable to the CloudSystem as it is expected users will transfer data to and from the solution.

### CloudSystem compliance approach

The CloudSystem includes Microsoft Defender ATP to assist with the inspection and auditing of data transfer to and from CloudSystem endpoints. Event logs are generated when data is transferred to external media from a Windows 10 endpoint.

### Security controls provided by the CloudSystem

* Defender ATP will scan all data copied onto CloudSystem Windows 10 devices.
* Event logs are generated when data is transferred to external media from a Windows 10 endpoint. 

### Residual controls to be addressed by the Agency

* The Agency is responsible for the development and implementation of a data transfer policy.
* The Agency is responsible for auditing data transfer logs.
