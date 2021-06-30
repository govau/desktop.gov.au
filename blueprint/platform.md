---
layout: page
title: Platform
---

The blueprint includes guidance for cloud native and hybrid deployments of Microsoft 365 technologies, configured to meet PROTECTED standards. It is designed to also be used for staged deployments that leverage hybrid configurations as a transition step to cloud native transformation.

Each section of the document provides a description of the relevant technology component, including considerations, decisions and their applicability to cloud native implementations, hybrid configurations, or both.

It is important to consider that even if a product is licenced for use by Microsoft, it may not be included in this blueprint if it is not required for all agencies. Additionally, an organisation may have requirements that will need to be considered outside of this blueprint.

This document covers the following topics.

Section | Description
--- | ---
Identity and Access Management | The Identity and Access Management section includes the authentication, authorisation methods and Conditional Access policies used within the blueprint for Cloud and Hybrid solutions.
Security | The Security section details several cloud-based security components available within the Microsoft 365 suite to detect and monitor suspicious behaviour for Cloud and Hybrid solutions.
Client Configuration | The Client Configuration section details the Microsoft Endpoint Manager - Intune (Intune) management methods and design decisions for the client configuration.
Backup and Operational Management | The Backup and Operation Management section details the backup design decisions including RPO, RTO and Data Availability.
System Administration | The System Administration section details how the solution will be managed, the administrative consoles that will be used to administrator the various components, and how Role Based Access Control (RBAC) is implemented to control access.

For each component within the document there is a brief description of the contents of the section, a commentary on the things that have been considered in determining the decisions and the design decisions themselves.

## Scope

The following tables in this section lists the components that are in scope for the blueprint and the relevant design document that contains them.

### Platform design

Component | Inclusions
--- | ---
Azure Active Directory | Domains<br>User Accounts<br>Agency Collaboration<br>Multifactor Authentication<br>Conditional Access
Active Directory | On premises for Hybrid solutions only
Security | Microsoft Cloud App Security<br>Azure Advance Threat Protection<br>Microsoft Defender Advanced Threat Protection<br>Security Information and Event Management<br>Monitoring
Client Configuration | Intune<br>Microsoft Co-Management<br>Group Policy<br>Printing
Backup | Microsoft 365 Backup
System Administration | Administrative Consoles<br>Role bases Access Control

### Microsoft 365 design

Component | Inclusions
--- | ---
Microsoft 365 Organisation | Residency<br>License<br>Self Service Purchase<br>Themes<br>Office 365 Services and Add-Ins<br>Role Based Access Control<br>Customer Lockbox
Microsoft 365 Connectivity | Mail Flow and Gateway<br>Optimisation<br>Exchange Hybrid<br>Mail Exchange Records<br>Mail Connectors<br>Autodiscover<br>SPF, DMARC, DKIM<br>Accepted Domains<br>Remote Domains<br>Certificates
Exchange Online | Mail Migration<br>User Mailbox Configuration<br>Authentication Policies<br>Outlook on the Web Policies<br>Mailbox Archive<br>Journaling<br>Litigation Hold<br>Shared Mailboxes<br>Resource Mailboxes<br>Distribution Lists<br>Microsoft 365 groups<br>Address Book / Address List
SharePoint Online | SharePoint Sites<br>SharePoint Hybrid<br>Application Management<br>Web Parts<br>Sharing and Access Controls<br>Legacy Features
OneDrive for Business | Sharing<br>Storage and Synchronisation<br>Notifications<br>Content Migration
Microsoft Teams | Access<br>Dynamic Security Group<br>Organisation Wide Configuration<br>Policies & Settings<br>Unified Communication<br>Voice Calling
Power Platform | Power Apps and Power Automate<br>Power BI
Security & Compliance | Alerts<br>Classification Labels<br>Retention Policies<br>Data Loss Prevention<br>Audit and Logging
Exchange Online Protection | Connection Filtering<br>Anti-malware<br>Policy Filtering<br>Content Filtering
Microsoft Defender for Office 365 | Safe Links<br>Safe Attachments<br>Anti-Phishing
Microsoft Whiteboard | Diagnostic Data<br>Enablement<br>Connected Experience<br>Easy Sharing
Microsoft Planner | iCalender Publishing
Microsoft Forms | External sharing<br>Phishing Protection<br>Bing and YouTube embedding

### Client devices

Component | Inclusions
--- | ---
Windows 10 | Hardware<br>Deployment<br>Customisations<br>Security
iOS | Enrolment<br>Security<br>Remote Wipe

## Assumptions

* Azure Multifactor Authentication (MFA) natively supports the OATH (Open Authentication) standard for selected hardware tokens. To use Azure MFA with OATH support, and to achieve an Essential 8 Maturity level of 3, hard tokens are required to be procured and deployed to all users. This blueprint and associated security documentation assume the use of soft tokens and a level 2 maturity in this aspect of the Essential 8. 
* Microsoft 365 and Microsoft Azure solutions hold audit data for a period based on the service and the license level of the organisation. The time for most services is under 2 years. For organisations with a requirement to hold audit data past this period, Security Information and Event Management (SIEM) integration should be considered. Service audit data within the Microsoft 365 and Azure clouds is often housed in discrete systems and the opportunities to bring the data under a single pane is limited. Azure Monitor or Azure Sentinel are two Microsoft offerings which could be leveraged for this purpose however a holistic solution should be considered to ensure any legislative requirements are met.
* The blueprint has been designed to cater for government organisations allowing end user devices internet access from anywhere (head office, regional office or home) direct connected and via proxy servers, VPN servers or Security Internet Gateways (SIGs). Where connected through a proxy server, rules will be configured to allow direct connection for some Microsoft 365 services. Mobile users will access Microsoft 365 services directly. These users will be subject to Conditional Access policies to reduce unauthorised access risk.
* The Intune Console is the preferred method to manage all settings regardless of Cloud native or Hybrid. Although a combination of the Microsoft Endpoint Configuration Manager (MECM) Console and Group Policy Objects (GPOs) would be able to achieve the same settings in a hybrid environment, this blueprint does not include MECM and GPOs example configurations due to the level of dissimilarities and per agency customisation in existing MECM and GPOs configurations across Commonwealth entities.

## Identity and access management

A directory service is responsible for the storage of identity information. Directories expose the identity information using network protocols such as the Lightweight Directory Access Protocol (LDAP). To ensure a seamless user experience and minimize potential identity conflicts, each identity should have a single point of truth / source. Changes should be replicated to but not managed by other directories.

Identity and Access Management (IAM) is the framework upon which digital identities and access to resources are managed. Within a hybrid solution this framework needs to encompass both the on-premises and cloud components.

### Azure Active Directory

Azure Active Directory (Azure AD) is a cloud-based directory service which stores identity information and offers Information and Access Management (IAM) for Microsoft cloud products, custom developed applications, and third-party applications. The identities within this directory service can be either cloud based or synchronised from an on-premises AD domain via the Azure AD Connect client.

Azure AD allows users to sign in and access resources like Microsoft Office 365, the Azure management portal, and other SaaS applications. Azure AD also provides control over the following directory activities:

* Registration of applications – The registration of application controls whether users can grant permissions to applications and register them within Azure AD.
* Restriction of the Azure AD administrative portal – The restriction of the Azure AD portal controls who can viewing of the contents of the Azure AD. The contents include user identity data.
* LinkedIn account connection – LinkedIn account connection allows users to link their work account to LinkedIn.
* External user invitations – External user invitation controls who can be invited by users to collaborate within the tenant.
* Azure AD preview features – Azure AD preview features control how new self-service features are made available to users.

Azure IAM is configured using:

* Enterprise Applications - The registration of Microsoft and Third-party enterprise applications. The registration requires information regarding the name, publisher, permissions, authentication configuration and Redirect URIs (Uniform Resource Identifier) to be provided.
* App Registrations – The registration of custom-built enterprise applications. The registration requires information regarding the name, Application Identifier (APP ID), permissions, authentication configuration and Redirect URIs (Uniform Resource Identifier) to be provided.

Where an Agency has already determined that a HYBRID solution is required, [additional considerations](https://docs.microsoft.com/en-us/azure/active-directory/hybrid/plan-hybrid-identity-design-considerations-overview) will apply and these should also be reviewed.

Azure AD Design Decisions for all agencies and implementation types.

Decision Point | Design Decision | Justification
--- | --- | ---
Cloud Based Service Accounts | Configured | Break glass accounts are required to be cloud based to ensure access to the tenant if there are issues with authentication.
Allow the registration of applications by users | Disabled | Only administrators can register applications.
Restrict access to the Azure AD administrative portal | Enabled | Only administrators have access to the portal.
Allow LinkedIn connections | Disabled | To meet the Agency's requirements not to share information with third party Agencies without approval.
External Collaboration | Configured | As required by the Agency, provided the external Agencies are at the same classification.
Group Management | Configured | Naming policies and expiration as required by the Agency.
Azure Active Directory RBAC | Configured | For ease of administration, segregation and delegation of roles. Users and administrators will be assigned only the roles they need through PIM (Privileged Identity Management).
Enterprise Applications | Not Configured  | No enterprise applications have been identified.
Device registration | Not Configured | Default settings.
Identity Governance | Not Configured | Default settings.
Application Proxy | Not Configured | No requirement has been identified.
Licences | Configured | Configured to assign Microsoft 365 licences to user groups. Ensures consistent configuration.
Custom Domain Names | Configured | {agency}.onmicrosoft.com<br>{agency}.gov.au 
Mobility (MDM and MAM) | Not Configured | Default settings.
Company Branding | Configured | Agency specific logos will be required.

Additional Azure AD Design Decisions for cloud native implementations

Decision Point | Design Decision | Justification
--- | --- | ---
Identity Source | Azure AD | As this is a cloud only implementation Azure AD will be the source of identity.
Password Reset | Configured | For self-service password reset, users will need to provide an alternate email address, mobile app and phone number during registration. To reset their password, they will need to provide two methods of verification.
Identity Format | Configured | Usernames will conform to firstname.lastname{sequence number}<br>Note: The sequence number is only required if duplicate names would be created.

Additional Azure AD Design Decisions for hybrid implementations

Decision Point | Design Decision | Justification
--- | --- | ---
Identity Source | Active Directory | As this is a hybrid implementation Active Directory will be the source of identity.
Synchronize to Active Directory | Configured | Cloud identities or Synchronised or Federated in accordance with agency specific requirements.
Azure AD Connect | Configured | See Azure AD Connect section for details.
Identity Format | Not Configured | Usernames will be synchronised from the on-premises Active Directory and will inherit naming convention.

### Microsoft 365 groups

Microsoft 365 Groups are an extension on the traditional mail Distribution Lists, Mail-enabled Security groups and Shared Mailboxes. They are managed within Azure Active Directory and also are used within Office 365 to control access to items such as a Team or Plan.

Microsoft 365 Groups allow members to collaborate with a group email, shared a workspace for conversations, files, calendar events, and a Planner. Unlike Shared Mailboxes, Microsoft 365 groups can be accessed via mobile applications. Microsoft 365 groups are also integrated with Microsoft Teams and Microsoft Planner and are created when a Team or Plan is created.

Membership of an Microsoft 365 Group can be dynamically updated using user attributes available in Azure AD. This removes some of the management overhead involved with managing the traditional group structures.

Management of Microsoft 365 Groups can be streamlined through the enforcement of a Naming Policy, Office 365 group expiry, and creation restrictions. An Office 365 Group Naming Policy allows the enforcement of a consistent naming strategy across Microsoft 365 Groups. It consists of two parts:

* Prefix-Suffix Naming Policy – Setting of prefixes or suffixes for groups names. The prefixes/suffixes can be either fixed strings or user attributes; and
* Custom Blocked Words – Blocking of words in the name based on a custom list.

In conjunction with the Naming Policy, Microsoft 365 groups can also be given expiration dates. This assists with unused group clean-up activities. The expiration period commences on group creation and can be renewed at the end of the period (The owner or contact for groups with no owners has 30 days to renew the group). When a group expires, it is soft deleted for 30 days. Retention policies will however hold the data for the period of the retention policy. An expiration policy can be applied globally to all groups or to specific groups.

Microsoft 365 Groups, by default can be created by any user. This can be restricted to Administrators and members of a security group. This restriction prevents the needless creation of groups. It is advisable to develop a workflow to control the provisioning process.

Microsoft 365 Group Design Decisions for all agencies and implementation types.

Decision Point | Design Decision | Justification
--- | --- | ---
Microsoft 365 Group creation restrictions | Configured <br>Only administrators and select users can create/configure Microsoft 365 groups. | This will ensure that groups are approved before being created, ensuring all groups have a purpose.<br>This setting also affects Exchange, SharePoint, Microsoft Planner and Teams.
Naming Policy | `grp-AgencyName-SecurityGroup-Role` | Exchange groups will be named like the following AgencyName-Agency e.g. `grp-DTA-ExchangeMailbox-ITHelpdesk`
Group Expiration  | All groups - annually | Group Expiration is required to simplify the management overhead associated with groups and to limit Azure AD clutter.

### Emergency access admin accounts

Emergency access or 'break-glass' accounts are accounts used to restore access to an environment or tenant. Break-glass accounts are an option of last resort and should only be used when normal administrative accounts cannot be used to restore access. Example of this is where conditional access has malfunctioned and both users and administrative staff are unable to authenticate into the environment, or all privileged administrator accounts are comprised in another capacity.

[Microsoft best practice](https://docs.microsoft.com/en-us/azure/active-directory/users-groups-roles/directory-emergency-access) recommends that two break-glass accounts are configured. These accounts should be stored according to the standard break-glass procedures. Best practice is for these accounts to meet the following minimum requirements:

* They are not to be associated with any individual user.
* They will use a different authorisation mechanism than standard administrative accounts.
* They are cloud only accounts that use the '*.onmicrosoft.com' domain.
* The passwords to these accounts are set to never expire or be cleaned up or removed due to inactivity.
* The accounts are to be given the Global Administrator role assigned permanently.
* At least one of the accounts is to be excluded from Multi Factor Authentication (MFA).
* At least one of the accounts is to be completely excluded from all Conditional Access policies.
* The accounts are stored on paper, in two or three separate parts, in secure, fireproof safes that are in disparate locations.
* Use of these accounts is monitored and only used in genuine emergencies.

Emergency access account Design Decisions for all agencies and implementation types.

Decision Point | Design Decision | Justification
--- | --- | ---
Emergency Access accounts | Configured | Two emergency access accounts will be configured in alignment to Microsoft and security best practice.

Emergency access account configuration for all agencies and implementation types

Configuration | Value | Description
--- | --- | ---
Username | Any value that is not associated to a specific user. | Accounts are not to be associated with any individual user.
Account type | Accounts are cloud only accounts that use the '*.onmicrosoft.com' domain. | Only *.onmicrosoft.com accounts should be used as per Microsoft best practice.
Password Expiry | Passwords are set to never expire. | Ensures the passwords for these accounts are valid in an emergency.
Roles | Emergency Access accounts will be assigned the Global Administrator role. | The accounts are to be given the Global Administrator role assigned permanently.
MFA | Both Emergency Access accounts will be excluded from MFA. | Multi Factor Authentication (MFA) device may not be available when the emergency access account is required.
Conditional Access | At least one of the accounts is to be completely excluded from all Conditional Access policies. | The emergency access account may need access to fix an issue and it would not be beneficial if a policy were to block access.
Physical access to account details | Account details will be stored on paper in an appropriate location. | It is strongly recommended that the accounts are stored on paper, in two or three separate parts, in secure, fireproof safes that are in disparate locations.
Monitoring of accounts | Account usage will be monitored via MCAS. | Use of these accounts is monitored and only used in genuine emergencies.

### Azure Active Directory identity protection

Azure AD Identity Protection is the function of provisioning access rights to a resource. Azure AD Identity Protection can take the form of an access policy. An access policy defines the business rules on whether authenticated user is granted or denied access to a resource. Azure AD utilises Conditional Access to define the access policies for Office 365 data. Azure AD using Identity Protection utilises analytics further to minimise the risk that access is provisioned to a compromised authenticated user.

Azure AD Identity Protection enables configuration of automated responses to suspicious activities and actions related to user identities. With Azure AD Identity Protection, risk-based policies can be configured that automatically respond to detected issues when a specified risk level has been reached.

These policies, in addition to other conditional access controls provided by Azure AD, can either automatically block, Smart Lockout, or initiate adaptive remediation actions including password resets and MFA enforcement.

Azure AD Identity Protection uses the following mechanisms to detect anomalous activity within the environment:

* Vulnerabilities – Azure AD Identity Protection analyses identity configuration and detects vulnerabilities that can have an impact on user identities. Vulnerabilities can include items such as unmanaged cloud applications.
* Risk Events – Azure AD uses adaptive machine learning algorithms and heuristics to detect suspicious actions that are related to the user's identities. The system creates a record for each detected suspicious action. These records are also known as risk events and include activities such as Sign-ins from anonymous IP addresses, sign-ins from IP addresses previously detected as exhibiting suspicious activity, or unfamiliar locations.

Azure AD Identity Protection provides mechanisms for logging and reporting functionality that simplify investigation activities.

Azure AD Identity Protection Design Decisions for all agencies and implementation types.

Decision Point | Design Decision | Justification
--- | --- | ---
Azure AD Identity Protection | Enable the sign-in risk policy and user risk policy within the Azure AD tenant. | Provide reporting of detected suspicious sign-in activity based on defined MFA, sign-in risk and user risk policies for increased security.
User risk policy | Enabled | The user risk policy detects the probability that a user account has been compromised by detecting risk events that are a typical of a user's behaviour, alerting when high risk behaviour is detected.
Sign-in risk policy | Enabled | Azure AD analyses each sign-in of a user. The objective of the analysis is to detect suspicious actions that come along with the sign-in. Automated actions will be configured to be taken when high risk behaviour is detected.

### Azure AD multifactor authentication

Authentication is a primary security control to protect both information assets - ranging from logging into a Windows device, to sending an email or collaborating on a document. When deploying Microsoft 365, the identity for each individual staff member is either in the cloud or both in the cloud and on-premises. Employing multiple authentication factors present a significant challenge for attackers gaining access to a system. Traditional authentication methods rely purely on something the user knows, such as a password. Note, a username does not technically count as an authentication factor. A user enters a username to only claim an identity, then must provide an authentication factor to validate this claim (identification vs authentication). Multi-factor authentication (MFA) is recommended by the ACSC for all users to prove a user's identity before being granted access.

Multi-factor authentication is any combination of two or more authentication sources from the following categories. Biometrics are yet to be accepted as part of the industry standard definition of an authentication factor.

* Something a user knows (such as a password or PIN).
* Something a user has (such as a specific hardened device).
* Something a user is (such as biometric trait, for example a fingerprint).

Azure Multifactor Authentication provides additional security by requiring a second form of authentication and delivers strong authentication via a range of easy to use authentication methods.

Azure MFA provides multiple verification methods, such as:

* Call to phone – Call to phone places an automated voice call to a phone number defined by the user.
* Verification code from mobile app – The Microsoft Authenticator app generates a new verification code every 30 seconds. The user enters the verification code into the sign-in interface.
* Notification through mobile app – Sends a push notification to a user's phone or registered device using the Microsoft Authenticator app. The user views the notification and selects "Approve" to complete the verification process.
* Text message to phone – Sends a text message that contains a verification code that is used as the authentication token. The user is prompted to enter the verification code into the sign-in interface. This process is called one-way SMS.
* OAuth hardware token verification code – OATH is an open standard that specifies how one-time password (OTP) codes are generated. Various vendor tokens are supported.

Azure MFA integrates with Azure AD Conditional Access polices, or the Trusted IP ranges feature to determine under what circumstances and user's physical location a challenge for additional authentication is required . Conditional Access polices are the recommended method to determine MFA conditions.

Azure AD Multifactor Authentication Design Decisions for all agencies and implementation types.

Decision Point | Design Decision | Justification
--- | --- | ---
MFA | Configured – Mobile App – soft token code | Native Azure MFA will be configured to secure access to applications and desktops from outside of the environment, and any system administration functions. Use of a mobile app for verification instead of SMS message or phone call reduces any possibility of hack by cloning or swapping a sim card.<br> The ACSC recommends implementing soft tokens without push notifications.
Hardware Token Support | Allowed (supported OATH tokens only)  | The default method will be to use soft tokens which will meet maturity level 2 of the essential 8, although hardware tokens will be allowed. Hardware token support is required to support some use cases. Some working locations may not allow mobile phones, or users may have a specific physical token, biometrics or smartcard justification. Having hard tokens is a requirement to achieve Essential 8, level 3 maturity for multifactor authentication.
Trusted IP | Not configured | Conditional Access policies will be used in place of the legacy 'Trusted IP' feature. Trusted egress IP addresses will be defined by Conditional Access.
MFA for Administration | Configured | Administration through the Azure Portal and other Cloud Apps will require MFA.
MFA for User Apps | Configured | MFA is required.

Note: OATH tokens are to be purchased separately if required.

### Conditional access

Conditional Access provides access controls that can be applied to user login requests. These access controls provide an extra level of security to help protect corporate data and information.

When a user attempts to access an application or system from any device, one or more conditions must be met before authentication is successful. Conditional Access offers the following types of access controls:

* User and location based – User and location based Conditional Access limits or blocks user access based on their geo-location or IP address.
* Device based - Device based Conditional Access ensures only enrolled and approved devices can access corporate data.
* Application based - Application based Conditional Access policies provide the ability to allow or block an application based on policy configuration.
* Risk based - Risk based Conditional Access protects corporate data from malicious hackers based on a user's Sign-In risk. The sign-in risk is an indicator for the likelihood (high, medium, or low) that a sign-in attempt was not performed by the legitimate owner of a user account. Azure AD calculates the sign-in risk level during the sign-in of a user.
* Session based – Session based Conditional Access policies enables the control of user sessions by redirecting the user through a reverse proxy instead of directly to the app. From then on, user requests and responses go through Cloud App Security rather than directly to the app.

Based on the above conditions, the user will either be allowed, prompted for multi-factor authentication, or blocked.

Conditional Access Design Decisions for all agencies and implementation types.

Decision Point | Design Decision | Justification
--- | --- | ---
Conditional Access Enabled| Device Based | To meet security and business requirements. This allows only approved and agency issued devices access to the Agency's resources.

Additional Conditional Access Design Decisions for hybrid implementations

Configuration | Description
--- | ---
BLOCK - Legacy Authentication | This global policy blocks all connections from insecure legacy protocols like ActiveSync, IMAP, POP3, etc.
BLOCK - High-Risk Sign-Ins | This global policy blocks all high-risk authentications (requires Azure AD Premium P2).
BLOCK - Countries not Allowed | This global policy blocks all connections from countries not in the Allowed countries list.
GRANT - Terms of Use | This global policy forces Terms of Use on all authentications.
GRANT - Browser Access | General browser access policy that grants authentication from a browser on any device with MFA requirement.
SESSION - Block Unmanaged Browser File Downloads | Browsers on unmanaged devices can never download files and attachments from SharePoint Online and Exchange Online.
GRANT - Intune Enrolment | Devices can authenticate to Intune for enrolment.
GRANT - Mobile Device Access | Grants access to managed mobile devices that are enrolled and compliant in Intune. An approved Microsoft app is required.
GRANT - Windows Device Access | Grants access to managed Windows devices that are Hybrid Azure AD Joined (joined to on-prem AD and Azure AD).
GRANT - Guest Access (B2B) | Approved apps that guest users can access (requires MFA).
BLOCK - Guest Access (B2B) | Blocked apps that guest users can never access.

###  Active Directory

Active Directory is an on-premises directory service which stores identity information in a hierarchical structure. Identities within the service are composed of attributes which describe the object. These attributes include but are not limited to the following:

* First name
* Last name
* Password
* Email Address
* Proxy Address

Additional features and capabilities are available within Active Directory when the Forest and Domain Functional levels are raised. The Functional level of Active Directory is linked to several items including the operating system of the domain controllers. To access all features and capabilities the highest functional level is required. For hybrid identity a minimum Forest Functional level of 2003 is required. To use additional hybrid identity features such as password writeback a minimum Forest Functional level of 2008 R2 is required.

Active Directory Design Decisions for hybrid implementations

Decision Point | Design Decision | Justification
--- | --- | ---
User Identity Source | On-premises Active Directory | The on-premises Active Directory will be used as the source for user identity, with identities being synchronised to Azure Active Directory. The directory functional level must meet the minimum requirements for Azure AD Connect synchronisation.

### Domain

When a new Azure AD tenant is created it is assigned a default domain name. This domain is internet routable and aligns to "{tenant name}.onmicrosoft.com". A second internet routable domain name is also provisioned if Exchange Online is activated within the tenant. The second internet routable domain aligns to "{tenant name}.mail.onmicrosoft.com".

On-premises domains can also be registered to the tenant to be used by Azure and Exchange Online. These domains can then be used for receiving email and/or be utilised as the User Principal Name (UPN).

Domain Design Decisions for hybrid implementations

Decision Point | Design Decision | Justification
--- | --- | ---
Registration of on-premises Domains | Configured | Required to utilise the on-premises identities.
Azure AD Primary domain | Configured - {agency}.gov.au | Required to ensure that Azure created resources contain the Agency's Primary Domain.

### Collaboration

Within Azure and Office 365 the ability to collaborate with other tenants exists through the External sharing, B2B (Business-to-Business) and B2C (Business-to-Customer) services. These are key features for any external or inter-Agency collaboration.

Utilising the blueprints, an Agency can configure collaboration with other organisations where:

* A business requirement exists.
* Both organisations choose to collaborate.
* The organisations trust each other.
* Organisations staff possess the appropriate clearance levels.
* Risk assessments have been completed; and,
* Collaboration is based on the lowest classification level of any involved Organisation or Agency.

Collaboration between organisations assessed and operating at the same security level is relatively straightforward, while collaboration between organisations operating on networks that have been assessed at different security levels presents additional considerations and risk. The additional risks and considerations are similar to those that already exist for organisations today, with activities such as personnel clearances, physical security requirements and the secure creation, storage and destruction of physical artefacts. These considerations will need to be assessed on a case by case basis and risks accepted by the Chief Information Security Officers (CISO).

ACSC provides guidance on connecting networks with differing security classifications available at [Fundamentals of Cross Domain solutions](https://www.cyber.gov.au/acsc/view-all-content/publications/fundamentals-cross-domain-solutions). At the time of writing, there are no automated options for external collaboration from a PROTECTED environment and user validation for external collaboration remains a manual process. This is particularly the case should an Agency adopting this service seek to collaborate from a PROTECTED environment to an environment that is operating at a lower classification. The ISM stipulates that all users of a PROTECTED environment must have a valid security clearance. It is recommended that users of the higher classification Agency collaborate into the lower classification Agency, and that unified labelling be configured as per the blueprints with PROTECTED content restricted to external sharing.

For Agencies operating at a PROTECTED level seeking to collaborate, B2C collaboration is not suitable as it allows authentication through publicly accessible domains such as Google and Facebook. Azure AD Business-to-Business (B2B) allows authentication between Azure tenants which provides a higher level of assurance.

B2B allows the most secure sharing of organisation applications, services, and data with external guest users from other organisations, while maintaining maximal control over corporate data. The collaboration options between two or more organisations can use the following platforms:

* Teams
* Planner
* SharePoint Online
* OneDrive for Business

Azure AD supports several B2B access scenarios to enable users within external organisations to collaborate with a host organisation. Users will be authenticated using an external identity source (e.g., Azure AD tenant credentials) which then generates a linked guest account within the host Azure AD tenant.

When an external user is invited to collaborate, the following items are checked:

* Is collaboration with the external domain allowed by B2B at the Azure AD level?
* Is guest access allowed by the application?
* Is external access with the external domain allowed by the application?

When the above are all true, the external user can be invited generating an invitation email. The user must accept the invitation by clicking on the link contained within the email causing a linked guest account to be created in the hosting Azure AD tenant. When the guest account has been created it is available for use by any of the applications that are configured to allow guest access.

B2B only requires a small amount of user information (name, and email), however it is recommended that CISOs consuming this document creates a process outside of technology that ensures Agency identity requirements are met. The identity requirements should include the properties listed in the table below and the external user's nationality and clearances held.

The following table describes the identity properties that should be a minimum requirement before collaboration is enabled for all agencies and implementation types.

Field | Example | Justification
--- | --- | ---
FirstName | John | Search and identify the user.
LastName | Smith | Search and identify the user.
UserName (UPN) = EmailAddress | john.smith3@{agency}.gov.au | Users agency and contract address.
UserName | john.smith3 | Identity in Microsoft.
EmailAddress | john.smith3@desktopa.gov.au<br>Users email contact.
OfficePhone | 61411 2999 | Users phone contact.
MobilePhone | 0411 123 456 | Users phone contact.
JobProfile | Finance | Users job description in identifying appropriate contact.
Agency | Digital Transformation Agency | Users agency.
Manager | Julie Citizen | Users manager for further consultation.
Photo | ID.JPEG | Viewing and identifying the user.

In addition to the above, Conditional Access policies should be enforced requiring external individuals to use MFA, block legacy authentication, and block from disallowed locations.

The following table describes the minimal conditional access policies that should be applied by the partner agency for all agencies and implementation types.

Decision Point | Design Decision | Justification
--- | --- | ---
Conditional Access Policies | **BLOCK - Legacy Authentication**:<br>This global policy blocks all connections from unsecure legacy protocols like ActiveSync, IMAP, PO3, etc.<br>**BLOCK - Countries not Allowed:**<br>This global policy blocks all connections from countries not in the Allowed countries list. | Minimises the risk of the user in the partner organisation using credentials that have been compromised.

Azure AD design decisions in relation to inter-Agency collaboration for all agencies and implementation types

Decision Point | Design Decision | Justification
--- | --- | ---
B2B Relationships | Not Configured | The Agency is pursuing a PROTECTED-level certification for the solution. To maintain a robust PROTECTED platform for the Agency, B2B collaboration will not be configured.<br>In a future state, specified partner tenants can be listed assuming they are also rated at a PROTECTED level and approved by appropriate Agency personnel. Only users of these tenants would be allowed collaborative access.
B2C Relationships | Not Configured | Client collaboration other than partner organisations is not required.
External Sharing | Disabled | The Agency is pursuing a PROTECTED-level certification for the solution. To maintain a robust PROTECTED platform for the Agency, B2B collaboration will not be configured.<br>In a future state, specified partner tenants can be listed assuming they are also rated at a PROTECTED level and approved by appropriate Agency personnel. Only users of these tenants would be allowed collaborative access.

### Azure AD Connect

Azure Active Directory Connect (AAD Connect) is a product designed to synchronise directory objects and changes between Active Directory and Azure AD. AAD Connect enables the on-premises directory service to be the source of truth for identities within the environment and ensures that changes are replicated to Azure AD.

AAD Connect can be deployed in several patterns. These patterns follow the guiding principles of:

* Only one AAD Connect instance can be actively synchronising to an Azure AD tenant.
* On-premises AD can only be synchronised to one Azure tenant unless directory synchronisation and Microsoft Identity Manager (MIM) are leveraged.

As only one AAD Connect instance can be actively synchronising at a time high availability is not possible. A warm standby can be configured using a second AAD Connect server in Staging mode.

The following illustrates the user identity synchronisation between the Agency's on-premises AD to Azure AD.

![AD Connect Identity Synchronisation](/assets/images/platform-ad-connect-id-sync.png)

Within the AAD Connect client the synchronisation process can be customised in several ways including:

* Group Filtering – Group filtering limits the scope of the synchronisation to the members of a group within the on-premises directory.
* Organisational Unit (OU) Filtering – OU filtering limits the scope of the synchronisation to the objects in one or more OUs within the directory.
* Attribute Filtering – Attribute filtering controls which attributes from an object are synchronised to the cloud.
* Azure AD App Filtering – Azure AD app filtering assists in limiting the number of attributes synchronised to the cloud based on which Office 365 services are in use.

Each of the above customisations provide control over what directory information is synchronised to the cloud from the on-premises directory service. The AAD Connect client can also be leveraged to configure Single Sign-On (SSO) and Exchange Hybrid. AAD Connect must run on Windows Server 2012 or later and can synchronise many Active Directory objects to Azure AD and hence there is a range of hardware requirements to consider based on the number of objects in Active Directory that will be synchronised (refer to the [Azure AD Connect Prerequisites](https://docs.microsoft.com/en-us/azure/active-directory/hybrid/how-to-connect-install-prerequisites)).

When AAD Connect is leveraged, a user would be created within the on-premises directory service (Active Directory) and then synchronised via the AAD Connect client into Azure AD (the cloud-based directory service).

The following illustrates a typical user account provisioning workflow for a hybrid configuration.

![User Provisioning Workflow](/assets/images/platform-user-provisioning-workflow.png)

Firewall rules will be implemented for AAD Connect. Further details on firewall configuration can be found in the Network Configuration ABAC.

AAD Connect Design Decisions for hybrid implementations

Decision Point | Design Decision | Justification
--- | --- | ---
AAD Connect | Configured | Identity synchronisation is a critical requirement and must be implemented based on ACSC guidance. The source of truth for account information will be on-premises Active Directory.
Attribute used for login | User ID | This attribute is commonly used for logins as it will ensure that the same credentials are maintained for on-premises and in-cloud authentication.
Organisational unit filtering | Configured | To target only the required identities for synchronisation. Whole directory synchronisations are not recommended.
Single Sign On | Configured | Single Sign-on.
Staging Server | Configured | Best practice dictates a secondary staging server be in place to be used in disaster recovery scenarios.
Password writeback | Not Configured | Password writeback is not recommended by the ACSC.
Self Service Password Reset | Not configured | The Self-Service Password Reset feature requires activation of password writeback in the AAD Connect configuration which is not recommended by the ACSC. Thus, this feature is not configured.
Azure AD App and attribute filtering | Configured | All Azure AD App and attribute filtering will be synchronised as recommended by Microsoft .
Exchange Hybrid | Configured | Exchange will be used in a hybrid configuration with Exchange Online, therefore this setting is required to be set as Configured.
Exchange Mail Public Folders | Not Configured | The Agency does not leverage Public folders currently, therefore this setting is not required.
Directory extension attribute synchronisation | Not Configured | Not required for this solution.

AAD Connect configuration applicable to agencies leveraging a hybrid implementation.

Configuration | Value | Description
--- | --- | ---
Installation Mode | Custom | The type of installation – Default or Custom. The Default install does not allow customisation of the filtering.
SQL Mode | Local DB | The location of the AAD Connect database. Local DB is the default configuration and the simplest to manage.
Directory to Connect to | {Agency}.gov.au | Azure AD Tenant of the Agency.
On-premises attribute to use for Azure AD (used for logging in to Azure AD) | User ID  | This attribute is commonly used for logins as it will ensure that the same credentials are maintained for on-premises and in-cloud authentication.
Alternate ID | Not required | This is required in scenarios where primary ID may be duplicated between users in the organisation.
OU Filtering | Enabled <br>{TBA by the Agency} | OU filtering will be used to ensure that specific OUs containing entities such as service accounts are not synchronised with Azure AD. OU filtering will be finalised during deployment and documented in As-Built-As-Configured documentation.
Uniquely Identifying Users | Users are represented only once across all directories.<br>Let Azure manage the source anchor (ms-DS-ConsistencyGuid) | Default configuration. As users are not duplicated within the environment, this setting meets the solution requirements. The ms-DS-ConsistencyGuid is used when Azure manages the source anchor.
Azure AD Attributes | Default – All attributes | Default configuration. All attributes to be synchronised.
Synchronisation Interval | 30 minutes | Default synchronisation interval.<br>Note: Password resets and new accounts are synchronised immediately.

### Authentication method

Authentication is the process of verifying one's identity. Active Directory allows for the authentication of directory objects within the corporate network using a number of protocols such as LDAP and Kerberos.

In a hybrid scenario, authentication support is required outside the corporate network. In a hybrid scenario, credential authentication support is required outside the corporate network. This can be achieved using either:

* Cloud authentication – Cloud authentication utilises credentials stored within the cloud to authenticate users. The credentials can belong to cloud only accounts.
* Password hash synchronisation (PHS) – PHS synchronises a hash of the hash of a user's on-premises password which has undergone a salting process before it is sent to Azure AD. If the user's hashed password matches the stored password, the user is then authenticated. This means the authentication method will be handled in the cloud.
* Pass-through authentication (PTA) – This feature allows users to login to Azure services including Office 365 using their on-premises credentials. When authenticating, the user enters their credentials into an Azure authentication service. The service encrypts the credentials and place the request in a queue. The on-premises PTA agents read the queue and perform the decryption and validation against Active Directory. The outcome of the validation is sent via the PTA agent to the Azure authentication service to complete the user's authentication request. Through the process no credentials are stored within Azure AD.
* Federation integration (AD FS) – Active Directory Federation Services (AD FS) allows users to login to Office 365 services using the organisations existing federation infrastructure. A federation trust is established between the corporate network and Azure AD with the authentication method being handled on-premises.

The following illustrates the pass-through authentication method and how it communicates across the network.

![Authentication Architecture](/assets/images/platform-authentication-architecture.png)

The following table describes the ports and protocols used in a hybrid configuration between on the PTA Agent and Azure AD. The hardware requirements for PTA agents are available in Appendix 1: AAD Connect and PTA agent hardware requirements.

Protocol | Port | Description
--- | --- | ---
HTTP | 80 (TCP/UDP) | Enable outbound HTTP traffic for security validation such as SSL. Also needed for the connector auto-update capability to function properly.

Authentication Method design considerations and decisions apply to the HYBRID solution.

Decision Point | Design Decision | Justification
--- | --- | ---
Authentication method | PTA | To ensure passwords are controlled by on-premises Active Directory and are not synchronised and stored in the cloud.
Number of PTA agents | 3 | To ensure high availability 3 PTA agents will service authentication requests. The AAD Connect server and Staging server will form two of the agents.
HTTPS | 443 (TCP/UDP) | Enable outbound HTTPS traffic for operations such as enabling and disabling of the feature, registering connectors, downloading connector updates, and handling all user sign-in requests.

## Security

Information Technology (IT) Security refers to protection of networks, servers, intranets, data systems, data and computer systems. To protect these items, Microsoft Azure contains several security features and products which together:

* Secure the Platform – Microsoft Azure and Office 365, through their features and products to enable security in depth.
* Provide Risk Assessments – Azure Identity Protection, Azure ATP and Microsoft Defender ATP utilise analytics and machine learning to detect and flag unusual/risky behaviour.
* Provide Visibility into User Behaviour – Microsoft Cloud App Security (MCAS) provides security operations dashboards which provide visibility into the activities being undertaken within the environment.
* Control Data Exfiltration –Data Loss Prevention policies and MCAS session policies control the flow and protection of information inside and outside of the environment.

### Microsoft Cloud App Security

MCAS is part of Microsoft's Enterprise Mobility + Security (EM+S) suite of capabilities, providing CASB functionality to reduce the risk of leveraging cloud services, including those offered by Microsoft and third-party providers such as Google, Amazon and Dropbox. To manage the risks presented using cloud services, Microsoft has defined a [cloud app security framework](https://docs.microsoft.com/en-us/cloud-app-security/what-is-cloud-app-security#the-cloud-app-security-framework) which MCAS implements:

* Discover and control the use of Shadow IT – Shadow IT includes cloud services that are in use by users but not assessed and approved by security, including Software-as-a-Service (SaaS), Platform-as-a-Service (PaaS), and Infrastructure-as-a-Service (IaaS) offerings. To protect users and their data these services must be identified so that their risk can be determined, and management controls can be implemented. MCAS enables administrators to assess an extensive library of apps against a wide range of risks.
* Protect your sensitive information anywhere in the cloud – Once data is uploaded to a cloud service it is harder to control and manage compared to traditional on-premises storage. MCAS enables controls to be applied to data regardless of where it is stored leveraging automated processes and inbuilt policies to both classify and protect information.
* Protect against cyberthreats and anomalies – Due to the public nature of cloud apps they are exposed to potential malicious activity from external actors. MCAS monitors both user behaviour and app activity to identity anomalies and perform automatic remediation to ensure the confidentiality of data stored in the cloud. This includes identifying indications that a user's account credentials have been compromised.
* Assess the compliance of your cloud apps – Performing security assessments of cloud apps and services is both complex and expensive. MCAS provides an overview of the industry and regulatory standards that each identified cloud app has been assessed against to simplify the approval process.

#### Product architecture

The architecture of MCAS includes several integrated components to address each of the cloud app security framework requirements. The components include log collection and analysis capabilities to detect cloud apps, Application Programming Interface (API) connectors to interface with and control cloud app activity, and reserve proxy capability to enforce conditional access app control policies for authentication to cloud apps.

An overview of these components and how they combine in MCAS is illustrated below in Figure 5. Figure reproduced from [https://docs.microsoft.com/en-us/cloud-app-security/what-is-cloud-app-security#architecture](https://docs.microsoft.com/en-us/cloud-app-security/what-is-cloud-app-security#architecture)

![Figure 5 - MCAS Architecture](/assets/images/platform-product-architecture.png)

Further details including configuration of each of these components is presented later in this document.

#### Data location

At the time of writing MCAS is hosted from Azure data centres in the United States (US) and Europe ([https://docs.microsoft.com/en-us/cloud-app-security/cas-compliance-trust#data-location](https://docs.microsoft.com/en-us/cloud-app-security/cas-compliance-trust#data-location)). An MCAS tenant account is automatically created in the closest 'Geo'. For Azure tenants located in Australia, MCAS will use the US Geo.

#### Data retention

The data retention period for information stored within MCAS varies depending on the specific type of data.  The [four data types](https://docs.microsoft.com/en-us/cloud-app-security/cas-compliance-trust#data-retention) and their respective periods are listed below:

* Activity log – 180 days
* Discovery data – 90 days
* Alerts – 180 days
* Governance log – 120 days

Note, all user activity and security alert information can be exported from MCAS in Comma-Separated Values (CSV) format if longer data retention is required.

#### Administration

MCAS leverages Azure Active Directory (Azure AD) to provide Role-Based Access Control (RBAC) for administration via the web portal. By default, only the Global Administrator and Security Administrator roles have full access to MCAS. Other standard Azure AD roles that have at least read-only access to the portal include Compliance Administrator, Compliance Data Administrator, Security Operator, Security Reader, and Global Reader. 

In addition to the standard Azure AD roles, MCAS also has its own service-specific roles that provide finer grained RBAC [https://docs.microsoft.com/en-us/cloud-app-security/manage-admins](https://docs.microsoft.com/en-us/cloud-app-security/manage-admins). If required, Global and Security Administrators can also grant access to specific users within the MCAS portal.

### MCAS - Cloud discovery

The MCAS Cloud Discovery design decisions can be found below. MCAS Cloud Discovery components are made up of Log Collectors, Microsoft Defender ATP Integration, Cloud Discovery Enrichment, User Data Anonymisation, Custom Apps and App Filters & Queries.

The cloud discovery component of MCAS enables the detection of cloud apps by analysing logs that are uploaded to it.

There are two types of cloud discovery reports that are generated by MCAS, depending on the specific log source:

* Snapshot reports – generated by manually uploading a log export from a proxy or firewall device, provides on demand analysis at the time the log is uploaded.
* Continuous reports – leverage native first and third-party integrations to provide ongoing data ingest and analysis without the need for user interaction.

Continuous reports can be generated a few ways such as configuring one or more of the following [automated log upload capabilities](https://docs.microsoft.com/en-us/cloud-app-security/set-up-cloud-discovery):

* Log collector – centralisation of logs from one or more proxy or firewall devices to a Docker-powered collector using Syslog and/or File Transfer Protocol (FTP).
* Microsoft Defender ATP integration – native integration with Defender ATP logs directly from onboarded endpoint devices running, regardless of whether they connect to cloud services via a managed gateway or directly via the internet.

MCAS supports a wide range of [popular proxy and firewall vendors and products](https://docs.microsoft.com/en-us/cloud-app-security/set-up-cloud-discovery#supported-firewalls-and-proxies-) for both snapshot and continuous reports (via log collectors)  . A custom parser can also be configured for unsupported devices allowing manual attribute mapping.

Once a cloud app has been discovered and its usage reviewed it can be either sanctioned (approved) or unsanctioned (prohibited) via the Discovered Apps tab. Tagging a cloud app as unsanctioned does not block access directly but allows for the generation of a block script that can be downloaded from MCAS and imported into a proxy of firewall appliance.

Note, if MCAS is integrated with Defender ATP, or other options then unsanctioned apps will be blocked automatically without the need to generate block scripts.

Cloud Discovery Design Decisions for all agencies and implementation types.

Decision Point | Design Decision | Justification
--- | --- | ---
Cloud Discovery report type | Continuous reports | To provide continuous visibility while minimising management overhead
Log collector | Will be deployed to collect logs from the Agency's existing proxy or firewalls and upload them to MCAS | To provide automatic upload of logs
Microsoft Defender ATP integration | Enabled | To provide additional visibility from agency endpoints that have been onboarded into the Defender ATP.
List of sanctioned and unsanctioned cloud apps | To be developed during build with the Agency's Cyber Intelligence team | Provides visibility within the Agency as to what cloud applications are in use and by which department within the Agency.

### MCAS - log collector

A log collector receives logs from supported firewall and proxy devices, providing processing and compression before uploading to MCAS. The compression typically results in outbound traffic from the log collectors being 10% the size of received traffic. Configure automatic log upload for continuous reports at [https://docs.microsoft.com/en-us/cloud-app-security/discovery-docker](https://docs.microsoft.com/en-us/cloud-app-security/discovery-docker)

A log collector can receive logs via FTP - including FTP over Transport Layer Security (TLS) - and Syslog.

There are two supported deployment modes for log collectors:

* Docker container – a Microsoft-provided Docker image for both Windows and Linux operating systems.
* Virtual appliance – a Microsoft-provided Virtual Machine (VM) image for Hyper-V and VMware hypervisors. Note, the virtual appliance deployment mode is now deprecated.

The Docker container is supported on Windows 10 and Windows Server version 1709 and later, Ubuntu versions 14.04, 16.04 and 18.04, and Red Hat Enterprise Linux (RHEL) and CentOS 7.2 or later. The image can be deployed on VMs either hosted on-premises or within Azure, provided suitable network connectivity from the proxy/firewall devices is available.

Log Collector Design Decisions for all agencies and implementation types.

Decision Point | Design Decision | Justification
--- | --- | ---
Number of log collectors | One | An increase in the number of log collectors may be necessary if there is excessive traffic.
Log collector deployment mode | Docker container | Maximise support life as virtual appliance has been deprecated.
Log collector location | Within the Agency Gateway zone | Minimise number of firewall ports to be opened between the existing proxies and the log collector.
Log collector operating system | Agency's discretion of supported operating system for MCAS Log Collector. | MCAS supports Windows and Linux (Ubuntu, RHEL, CentOS) operating systems for the Log Collector.

### MCAS - Microsoft Defender ATP integration

Defender ATP integration enables cloud app and service traffic to be sent from supported Windows 10 devices (1709 or later) to MCAS to provide additional data for continuous reporting.

This capability is enabled from within the Advanced Features settings within Microsoft Defender ATP portal, as shown below in Figure 6. Figure reproduced from [https://docs.microsoft.com/en-us/cloud-app-security/wdatp-integration](https://docs.microsoft.com/en-us/cloud-app-security/wdatp-integration)

![Figure 6 - Defender ATP and MCAS Integration](/assets/images/platform-defender-mcas.png)

Microsoft Defender ATP integration Design Decisions for all agencies and implementation types.

Decision Point | Design Decision | Justification
--- | --- | ---
Microsoft Defender ATP portal configuration | Microsoft Cloud App Security enabled | To enable Defender ATP integration with MCAS. Important Note: Microsoft Defender ATP should be configured prior to enabling this feature.

### MCAS - cloud discovery enrichment

To further enrich cloud discovery data MCAS can integrate with Azure AD to replace users identified with Azure AD usernames.

This simplifies identification and investigation of user activity, as well as allowing correlation with API collected activities.

Cloud Discovery Enrichment Design Decisions for all agencies and implementation types.

Decision Point | Design Decision | Justification
--- | --- | ---
User enrichment | Enabled | To identify users by Azure AD username.

### MCAS - user data anonymisation

To protect the privacy of users MCAS supports anonymisation of usernames when logs are uploaded. This is done by encrypting the usernames included in logs using Advanced Encryption Standard (AES) with a 128-bit key which is unique to each tenant.

Security analysts can resolve encrypted usernames on demand to assist in investigations, and each username conversion is recorded in the Governance log. In addition to usernames, MCAS can also anonymise Windows 10 machine names.

User Data Anonymisation Design Decisions for all agencies and implementation types.

Decision Point | Design Decision | Justification
--- | --- | ---
Username anonymisation | Not enabled | Reduced investigation effort by not requiring security analysts to decrypt usernames on demand. If the Agency identifies a requirement this can be enabled.
Machine name anonymisation | Not enabled | Reduced investigation effort by not requiring security analysts to decrypt machine names on demand. If the Agency identifies a requirement this can be enabled.

### MCAS - custom apps

In addition to the extensive library of cloud apps that are natively available in Cloud Discovery, custom cloud apps – for example internally developed Line of Business (LOB) applications – can be added to provide visibility in their use.

This allows Cloud Discovery to identify these custom cloud apps from uploaded firewall and proxy logs and enables security analysts to filter on them specifically.

Custom Apps Design Decisions for all agencies and implementation types.

Decision Point | Design Decision | Justification
--- | --- | ---
Custom Apps | Configured on demand | Custom apps will be added to Cloud Discovery as they are identified by the Agency.

### MCAS - app filters and queries 

MCAS supports the creation and assignment of custom app tags from within the portal. Security analysts can create and apply custom tags to allow them to filter and query results specific to an area of interest. For example, a custom tag may be created for a business unit and that tag applied to only the applications that are approved for their use (i.e. Procurement). An analyst can then quickly filter based on this tag to review the use of these applications.

App Filters and Queries Design Decisions for all agencies and implementation types.

Decision Point | Design Decision | Justification
--- | --- | ---
Custom tags | Configured on demand | Custom tags will be added to Cloud Discovery as they are identified by the Agency.

### MCAS - app connectors

The MCAS App Connector design decisions can be found below. MCAS App Connector components are made up of MCAS -Azure and MCAS – Office 365.

App connectors enable MCAS to see and reach inside connected cloud apps, providing both visibility into, and control of, the information stored by them. App connectors use APIs provided by the cloud app vendors, with capabilities varying between third-party cloud app vendors.

To support multiple instances of a single cloud app, for example different teams owning and managing their own subscriptions, MCAS supports connecting multiple instances of the same cloud app. However, multi-instance support is only provided for API connected apps and is not available for cloud discovered and proxy connected apps. Additionally, multi-instance support is not available for Azure and Office 365.

At the time of writing, the following API app connectors are [available in MCAS](https://docs.microsoft.com/en-us/cloud-app-security/enable-instant-visibility-protection-and-governance-actions-for-your-apps):

* Azure
* Amazon Web Services (AWS)
* Box
* Dropbox
* G Suite
* Google Cloud Platform (GCP)
* Office 365
* Okta
* Salesforce
* ServiceNow
* WebEx
* Workday

To connect to each cloud app via API MCAS requires an account within that app that has administrative privileges with full access to all objects stored within it. The name of this specific privilege level is specific to each cloud app, e.g. Global Admin for Office 365 and Super Admin for G Suite. It is recommended that a dedicated account is used for integration with MCAS for each connected app.

App Connectors Design Decisions for all agencies and implementation types.

Decision Point | Design Decision | Justification
--- | --- | ---
Use of app connectors | Preferred for all supported cloud apps | Provides the greatest available level of visibility and connect of the connected apps
API administrator accounts | Dedicated account for MCAS for each connected app that requires one | Microsoft best practice to manage connected apps
List of connected apps | Azure<br>Office 365 | All approved cloud apps that are supported will be connected to MCAS via API

### MCAS - Office 365

At the time of writing, the Office 365 app connector supports the following [Office 365 apps](https://docs.microsoft.com/en-us/cloud-app-security/connect-office-365-to-microsoft-cloud-app-security):

* Office 365
* Dynamics 365 CRM
* Exchange
* OneDrive
* Power BI
* SharePoint
* Teams

Note, that Exchange and Power BI require that auditing has been enabled in the Office Security and Compliance Centre.

Office 365 App Connectors Design Decisions for all agencies and implementation types.

Decision Point | Design Decision | Justification
--- | --- | ---
Microsoft Office 365 Connector Configuration | Selected Components:<br>Azure AD Users and Groups<br>Azure AD Management events<br>Azure AD Sign-in events<br>Azure AD Apps<br>Office 365 activities<br>Office 365 files | All components of Office 365 on which Cloud App Security can obtain information

### MCAS - policies

MCAS supports a range of policy types to address the various risks associated with leveraging cloud apps, ranging from the detection of new and therefore unsanctioned cloud apps to identifying anomalous user activities that are outside the pattern of normal behaviour.

Up to seven policy types are available in MCAS depending on the data sources that have been enabled. These include:

* Access policies – providing real-time monitoring and control of user logins to identified cloud apps.
* Activity policies – leveraging integration with each cloud app's API, provides monitoring and control of activities within those applications (specific activities dependent on each vendor's API capabilities).
* Anomaly detection policy – detecting anomalous activities within connected cloud apps based on specific risk factors compared with a pre-determined baseline.
* App discovery policy – detecting new (unsanctioned) cloud apps and provides associated alerts.
* Cloud Discovery anomaly detection policy – reviewing the logs provided to Cloud Discovery specifically (as previously described) to detect anomalous behaviour.
* File policy – enabling scanning of connected cloud apps to detect and apply restrictions to specific sensitive files and content.
* Session policy – providing real-time monitoring and control of user activities within authenticated sessions to identified cloud apps.

Note, Access and Sessions policies are covered in additional detail in the Conditional Access App Control section later in this document.

In addition to the policy types listed above, MCAS provides pre-configured policy templates that can be used to streamline policy development and enforcement. Custom policies can also be created by specifying a trigger 

Policies Design Decisions for all agencies and implementation types.

Decision Point | Design Decision | Justification
--- | --- | ---
Use of policies | Use of policies is agency-specific and would require further development with internal Cyber Security Teams. However it is recommended that a policy be enabled to monitor and detect access from anonymous IP addresses. | Provides visibility within the Agency of suspicious behaviour and application use. 

### MCAS - Threat Protection

The MCAS Threat Protection design decisions can be found below. Threat Protection components include Azure Advanced Threat Protection.

### MCAS – Azure Advanced Threat Protection Integration

Azure ATP provides User Entity Behavioural Analytics (UEBA) by monitoring authentication requests to on-premises Active Directory (AD) Domain Controllers (DCs).

Integrating Azure ATP with MCAS extends this capability to hybrid environments and presents all Azure ATP Suspicious Activity (SA) alerts to the MCAS dashboard, reducing the need for security analysts to monitor multiple consoles. To connect Azure ATP to MCAS the user enabling the setting must be an Azure AD Global Admin. Integration is enabled in the MCAS console and does not require configuration from the Azure ATP console.

Azure ATP integration Design Decisions for all agencies and implementation types.

Decision Point | Design Decision | Justification
--- | --- | ---
Azure ATP data integration | Enabled | Azure ATP will be deployed within the Agency's environment.

### MCAS - Information Protection

The MCAS Information Protection design decisions can be found below. Information Protection components include Admin Quarantine, Azure Information Protection, Azure Security and Files.

### MCAS - Admin Quarantine

The admin quarantine is used to store files for administrative review that have been matched against an MCAS file policy. Examples include identifying and removing files from cloud apps that include sensitive content, such as financial and Personally Identifiable Information (PII) that should only be stored on-premises and not shared to external collaborators. 

Note, files that are detected as malware and are stored in either SharePoint Online or OneDrive for Business are not quarantined by MCAS.

Before a file policy can be configured to use the admin quarantine a folder location for it must first be identified. Only a single admin quarantine folder location is configured for each MCAS instance and is used by all file policies that leverage this capability. A user notification can also be configured to provide an explanation to users when they attempt to access a quarantined file.

Admin quarantine Design Decisions for all agencies and implementation types.

Decision Point | Design Decision | Justification
--- | --- | ---
Admin quarantine folder location | Configured | Folder location is agency-specific and will be determined via an agency's internal decision processes.
User notification | Default text – `This file was quarantined because it might conflict with your Agency's security and compliance policies. Contact your IT administrator for more information.` | Notify user that file has been quarantined using default text. Agency to determine whether default text meets IT security requirements.

### MCAS - Azure Information Protection

Azure Information Protection (AIP) provides document and email classification labelling, and protections based on those labels, across hybrid environments. MCAS can be configured to scan for AIP classification labels and content inspection warning when new files are detected in connected apps. Additionally, MCAS can be configured to only scan labels and warning originating from its tenant, therefore ignoring labels from external tenants.

To enable MCAS to inspect the content of files that have been protected by AIP it must be granted that permission in Azure AD. This is done via a guided activity initiated from the MCAS portal.

Azure Information Protection Design Decisions for all agencies and implementation types.

Decision Point | Design Decision | Justification
--- | --- | ---
Automatically scan new files for AIP classification labels and content inspection warnings | Enabled | MCAS is enabled to scan new files for AIP classifications as the blueprint will leverage Information Protection classification.
Only scan files for AIP classification labels and content inspection warnings from this tenant | Enabled | MCAS is enabled to scan files for AIP classification from the tenant as the blueprint will leverage Information Protection classification.
Inspect protected files | Enabled | MCAS is enabled to inspect protected files as the blueprint will leverage Information Protection classification.

### MCAS - Azure security

MCAS can be configured to monitor all activities generated within those subscriptions.

Similar to Azure ATP integration, this centralises alerts into the MCAS dashboard, reducing the need for security analysts to monitor multiple consoles. The Azure Security Centre standard tier license must be acquired and enabled for one or more subscriptions within the tenant.

Azure Security Design Decisions for all agencies and implementation types.

Decision Point | Design Decision | Justification
--- | --- | ---
Azure Security Centre integration | Enabled | Provides Azure Security alerts and activities to the MCAS dashboard simplifying security operations

### MCAS - files

As previously described, file policies can be used to manage documents stored in cloud apps. To enable this capability MCAS must be allowed to monitor files stored in these apps.

There are no design considerations.

MCAS Files Design Decisions for all agencies and implementation types.

Decision Point | Design Decision | Justification
--- | --- | ---
File monitoring | Enabled | Allow MCAS to monitor files stored in connected cloud apps.

### MCAS - conditional access app control protection

The Conditional Access App Control Protection design decisions can be found below. Conditional Access App Control components include Default Behaviour, User Monitoring, Device Identification and App Onboarding/Maintenance.

The Conditional Access App Control capability of MCAS leverages Azure AD Conditional Access to enforce actions (such as blocking accessing) based on specific conditions (such as device compliance) by using a reverse proxy architecture. Users' cloud app sessions, including authentication, are proxied by MCAS instead of going directly to the app. MCAS does this by replacing the app's Uniform Resource Locators (URLs) and cookies, therefore not requiring an agent to be installed on the endpoints.

Examples of the Conditional Access App Control policies that can be configured to monitor and control user app access and sessions in real time are listed below:

* Prevent data exfiltration – block copy and cut clipboard actions, printing and downloading of sensitive information and documents.
* Protect on download – allows sensitive documents to be downloaded by leveraging AIP applies a classification label and protection to it.
* Prevent upload of unlabelled files – blocks documents from being uploaded until a user applies an AIP label to them based on the sensitivity of the information they contain.
* Monitor user sessions for compliance – identify risky users based on their behaviour during a session.
* Block access – based on specific risk factors can prevent users and/or devices from accessing specific resources across one or all connected apps.
* Block custom activities – application-specific events can be blocked if they increase the risk of data leakage or exfiltration.

Note, to use Conditional Access App Control an Azure AD Premium P1 license is required in addition to the MCAS license.

Conditional Access App Control Protection Design Decisions for all agencies and implementation types.

Decision Point | Design Decision | Justification
--- | --- | ---
Access control policies | Use of policies is agency-specific and would need further development with internal Cyber Security Teams. | Provides a greater security posture for the Agency applications.
Session control policies | Use of policies is agency-specific and would need further development with internal Cyber Security Teams. | Provides a greater security posture for users within the Agency.

### MCAS - default behaviour

In the event of a system outage or downtime MCAS can be configured with a Default Behaviour.

This is a deployment wide setting which can be configured to either prevent users from accessing an app if normal policy cannot be enforced, or to provide unrestricted access if this occurs.

Default Behaviour Design Decisions for all agencies and implementation types.

Decision Point | Design Decision | Justification
--- | --- | ---
Default behaviour configuration | Allow access | To prevent interruption to business functions in the event of an outage or downtime.

### MCAS - user monitoring

When using Conditional Access App Control to manage access and sessions, MCAS provides the option to notify users that their activities are being monitored.

If user notifications are enabled administrators can either use a Microsoft-provided default message or provide their own, which will include their organisational logo (if it has been uploaded to MCAS).

User Monitoring Design Decisions for all agencies and implementation types.

Decision Point | Design Decision | Justification
--- | --- | ---
User monitoring notifications | Not enabled | There is no default requirement to notify users. Agencies can enable User Monitoring as appropriate to meet operational requirements without impacting the cyber security posture of the environment.

### MCAS - device identification

MCAS and Conditional Access App Control can be utilised to identify managed devices within the organisation.

MCAS and Conditional Access App Control support three methods to identified managed devices:

* Microsoft Intune (specifically devices that are identified as Compliant).
* Hybrid Azure AD joined devices.
* The presence of client certificates that are part of a trusted chain.

A combination of multiple of these methods can be configured if required to identify devices within a diverse environment. Devices that are present in Intune, as well as those that are registered to Azure AD (hybrid joined), are automatically synchronised to MCAS. To use client certificates to identify devices either a trusted root or intermediate certificate must be uploaded to MCAS in Privacy-Enhanced Mail (PEM) format.

Device Identification Design Decisions for all agencies and implementation types.

Decision Point | Design Decision | Justification
--- | --- | ---
Use of client certificates for device identification | Not configured | No requirement to use client certificates has been identified as the Agency will leverage Intune for compliant and Hybrid Azure AD devices.

### MCAS - app onboarding/maintenance

Specific users can be identified that can enable currently unsupported cloud apps to be onboarded to Conditional Access App Control for development and testing purposes.

These users can be identified by either email or User Principal Name (UPN) and must be configured within the MCAS console. Once an app is onboarded by a specified user a feedback bar will be presented as part of the application to enable developers and testers to provide feedback directly back to Microsoft's Cloud App Security team.

App Onboarding/Maintenance Design Decisions for all agencies and implementation types.

Decision Point | Design Decision | Justification
--- | --- | ---
Included users | To be developed during build with the Agency's Cyber Intelligence team. | Agency to determine if a requirement for users and non-supported apps exist within the environment.

### MCAS - security extensions

To integrate with other security solutions, for example Security Information and Event Management (SIEM) products, MCAS supports a range of security extensions. These include:

* API tokens – to provide access to MCAS Representational State Transfer (REST) API endpoints for read and update operations.
* SIEM agents – to centralise alerts and activities from MCAS to a SIEM.
* External Data Loss Prevention (DLP) – to connect MCAS to external DLP solutions.
* Playbooks – to leverage Microsoft Flow playbooks for automation.

Each of these security extension capabilities is described in the following sections.

### MCAS - API tokens

By creating API tokens within the MCAS portal external applications can connect to the REST API endpoints and perform a range of read and update operations.

A common use-case for this is to generate block scripts from a third-party network appliance, or to resolve alerts that have been investigated through a connected SIEM product.

API token Design Decisions for all agencies and implementation types.

Decision Point | Design Decision | Justification
--- | --- | ---
API tokens | Not configured | An agency can determine and identify whether use cases exist for the creation of API tokens in their specific environments however for a base implementation of the blueprint, API tokens are not required.

### MCAS - SIEM agents

To enable MCAS alerts and activities to be integrated into existing security analyst workflows that have been developed in SIEM products MCAS provides SIEM agents.

The MCAS SIEM agent runs on a server – either on-premises or as IaaS – and acts as a forwarder between MCAS and the SIEM. The SIEM agent makes an outgoing request to MCAS over an encrypted Hypertext Transfer Protocol Secure (HTTPS) channel using port 443, leveraging the MCAS RESTful APIs. Once the data has been downloaded it then forwards it to the SIEM as syslog messages on a configurable Transmission Control Protocol (TCP) or User Datagram Protocol (UDP) port.  

Note, if both Azure ATP and MCAS are configured to send alerts to the same SIEM duplicate alerts will be received with different alert IDs. It is recommended to only send these alerts from one source.

At the time of writing the SIEM agent only supports Micro Focus ArcSight and generic [Common Event Format (CEF)](https://docs.microsoft.com/en-us/cloud-app-security/siem). Supported time formats include Request for Comment (RFC) 5424, 3164 and 3164 with year. The agent can be installed on either Windows or Linux operating systems and requires Java 8.

In addition to the SIEM agent, MCAS supports native integration with Azure Sentinel and the Microsoft Security Graph API. Azure Sentinel is Microsoft's cloud native SIEM offering, while the Security Graph API provides additional partner integration solutions, e.g. the Microsoft Graph Security API Add-On for Splunk.

SIEM agents Design Decisions for cloud native implementations

Decision Point | Design Decision | Justification
--- | --- | ---
Azure Sentinel integration | Configured | To support integration between MCAS and Microsoft cloud native SIEM solution.
Azure Sentinel license | Yes | To enable Azure Sentinel integration an Azure Sentinel license is required.

SIEM agents Design Decisions for hybrid implementations

Decision Point | Design Decision | Justification
--- | --- | ---
Use of SIEM agent | Yes | To support integration between MCAS and the Agency's existing SIEM solution.
SIEM agent install location | Agency's discretion of supported operating system for MCAS SIEM agents. | MCAS supports Windows and Linux operating systems for the SIEM agents.
Microsoft Security Graph API integration | Not configured | Agency to determine whether requirement exists based on existing SIEM solution.

### MCAS - external Data Loss Prevention (DLP)

In addition to integration with third-party SIEMs, MCAS also supports integration with third-party DLP providers, enabling existing investments in on-premises solutions to be extended into the cloud.

Integration between external DLP solution and MCAS is performed via the Internet Content Adaptation Protocol (ICAP) protocol which is tunnelled over TLS. This allows MCAS to use the external DLP engine to scan content identified in connected cloud apps for policy violations.

External DLP Design Decisions for all agencies and implementation types.

Decision Point | Design Decision | Justification
--- | --- | ---
External DLP | Not configured | Not configured by default however an Agency can determine whether a DLP solution will be leveraged and if there is a requirement to be integrated with MCAS.

### MCAS - playbooks

MCAS supports playbook-based automation by integrating with Microsoft Flow (also known as Power Automate).

Specific MCAS alerts can be configured as playbook triggers, automating pre-approved responses and minimising security analyst intervention. Playbooks are created in Microsoft Flow and made available to MCAS via the Cloud App Security connector. The use of playbooks requires a Microsoft Flow plan.

Playbooks Design Decisions for all agencies and implementation types.

Decision Point | Design Decision | Justification
--- | --- | ---
Playbooks | Not configured | Not configured by default however an Agency can determine whether Playbooks will be leveraged to assist in automating pre-approved and consistent responses reducing analyst intervention. Use of Playbooks is subject to licensing.

### MCAS - firewall

Microsoft Cloud App Security is a cloud-based service that helps keep the organisation secure from threats by collecting data from a Cloud based tenant and Hybrid on-premises infrastructure and analysing this data. To achieve this firewall rules and proxy allow lists needs to be in place to allow communication between all the components.

Firewall rules and proxy allow lists will be implemented as part of the Microsoft Cloud App Security solution. 

Further details on firewall configuration and proxy allow lists for the solution can be found in the Network Configuration ABAC.

### Azure Advanced Threat Protection

Microsoft Azure Advanced Threat Protection (Azure ATP) is an Azure-hosted security capability designed to monitor AD traffic and alert on suspicious authentication-related activities. Azure ATP leverages Machine Learning (ML) to analyse user, device, and resource authentication behaviours to build a baseline, then alert security analysts when an authentication attempt occurs that is not consistent with the baseline. Traffic corresponding to known authentication-based attacked, such as Pass-the-Hash (PtH) and Pass-the-Ticket (PtT), is also recognised by Azure ATP and specific alerts generated.

The Azure ATP architecture is composed of a Azure ATP cloud service with the following components:

* Azure ATP cloud service – Is hosted on Azure infrastructure and at time of writing Azure ATP cloud service is deployed in the US, Europe, and Asia data centres.
* Azure ATP portal – Management interface where the Azure ATP instance can be created, displays data collected by Azure ATP sensors and is the central console for monitoring, managing and investigating threats.
* Azure ATP sensor – Sensors are installed on all domain controllers which monitor and collect domain controller traffic that is feed back to the Azure ATP portal for analysis.

A high-level illustration of Azure ATP architecture is shown below in Figure 7. Figure reproduced from [https://docs.microsoft.com/en-us/azure-advanced-threat-protection/atp-prerequisites](https://docs.microsoft.com/en-us/azure-advanced-threat-protection/atp-prerequisites)

![Figure 7 - Azure ATP Architecture](/assets/images/platform-azure-atp.png)

A single Azure ATP instance can monitor multiple AD DS forests and their respective domains.

Azure ATP relies on Network Name Resolution (NNR) to correlate between raw activities (containing IP addresses) and the relevant devices involved in the activities to be able to generate security alerts for suspicious activities. To correlate IP addresses to device names, Azure ATP sensors lookup the IP addresses using these methods:

* NTLM over RPC (TCP Port 135).
* NetBIOS (UDP port 137).
* RDP (TCP port 3389) – only the first packet of Client hello.
* Queries the DNS server using reverse DNS lookup of the IP address (UDP 53).

Microsoft recommends using all the above methods, however if this is not possible the Agency should use the DNS lookup method and at least one of the other methods available at [https://docs.microsoft.com/en-us/azure-advanced-threat-protection/atp-nnr-policy](https://docs.microsoft.com/en-us/azure-advanced-threat-protection/atp-nnr-policy).

Azure ATP Design Decisions for hybrid implementations

Decision Point | Design Decision | Justification
--- | --- | ---
License | Enterprise Mobility + Security 5 (EMS E5) | An Enterprise Mobility + Security 5 (EMS E5) license is required for Azure ATP.
Number of Azure ATP instances | One | A single Azure ATP instance can monitor multiple AD DS forests.
Azure ATP instance name | {agency-instance-name}.atp.azure.com | The Azure ATP cloud service will be given an Azure ATP instance name which will be used to access the Azure ATP portal.
Forests and domains to be monitored by Azure ATP | {agency}.gov.au | Nominated agency forests and domains.
Azure ATP sensor deployment | To all DC's within the identified forests and domains. | Best practice to ensure all authentication traffic is monitored by Azure ATP.
Internet connectivity | Domain controllers must have internet connectivity | Domain controllers which will have Azure ATP sensors installed, must have internet connectivity to the Azure ATP Cloud Service. <br>Azure ATP sensors support the use of a web proxy / WPAD for internet connectivity.
Directory service accounts | A standard AD user account & password<br><br>A group Managed Service Account (gMSA) | If the Agency environment consists of Windows Server 2008 R2 SP1 domain controllers a standard AD user account and password is required with read access to all objects in the monitored domains.<br><br>If Agency environment consists of Windows Server 2012 or above domain controllers than a group Managed Service Account (gMSA) is required with read access to all objects in the monitored domain.<br><br>If the Agency environment consists of a mixture of domain controller operating system versions, then a combination of group Managed Service Account (gMSA) and Standard user account is required. See [ATP prerequisites](https://docs.microsoft.com/en-us/azure-advanced-threat-protection/atp-prerequisites).
Network Name Resolution (NNR) | Reverse DNS lookup and one other method (listed above) | This is the minimum NNR requirement for Azure ATP. Microsoft recommends using all of the above-mentioned resolution methods available within Agency environment.
Deleted Objects container permissions | Read-only | Microsoft recommends users should have read-only permissions assigned on the [Deleted objects container](https://docs.microsoft.com/en-us/azure-advanced-threat-protection/atp-prerequisites#before-you-start) to allow Azure ATP to detect user deletions from the Agencies Active Directory.

### Azure ATP – role groups

Azure ATP provides security groups to allow the implementation of a Role-Based Access Control (RBAC) model.

Azure AD provides the basis for the Azure ATP role groups. When Azure ATP is enabled for the first time it automatically creates the three security groups in Azure AD. The three Azure ATP security groups are:

* Azure ATP {Instance Name} Administrators – Provides full administrative access to the specific Azure ATP instance including all configuration settings.
* Azure ATP {Instance Name} Users – Able to modify configurations relating to suspicious activities (i.e. change status and add exclusions), alerts and scheduled reports, but not the configuration of Azure ATP sensors or data sources.
* Azure ATP {Instance Name} Viewers – Access to log into the Azure ATP console to view suspicious activities and download reports. Not able to modify the status of a suspicious activity or add any exclusions.

Note, in addition to the role groups described above, any tenant Global and Security Admins can login to the Azure ATP portal.

Role group Design Decisions for hybrid implementations

Decision Point | Design Decision | Justification
--- | --- | ---
Members of the Azure ATP Administrators group | Specific users in the Agency's Cyber Security team only | Only specific users that require administrative access should be added to this group.
Members of the Azure ATP Users group | {agency_domain_name} \<br>Admin_CyberIntelligence | Includes membership of the whole Cyber Intelligence team that are responsible for the day-to-day use and management of Azure ATP.
Members of the Azure ATP Viewers group | {agency_domain_name} \<br>Admin_AZ_TSG_PR_ Assurance_Comp | Includes membership of the Assurance team to enable auditing of Azure ATP.

### Azure ATP – notifications

In addition to creating alerts in the Azure ATP console whenever a new suspicious activity or health issue is detected, Azure ATP can also be configured to send email and syslog notifications. This enables security analysts to monitor existing mailboxes and leverage toolsets such as a Security Information and Event Management (SIEM) product rather than constantly having to login and view the Azure ATP portal.

Azure ATP does not require a Simple Mail Transfer Protocol (SMTP) server to be configured to send email alerts. However, the syslog configuration of a SIEM is required to send alerts to it. The required settings include:

* Fully Qualified Domain Name (FQDN) / Internet Protocol (IP) address.
* Port that that SIEM is listening on for syslog alerts.
* Transport protocol, either Transport Control Protocol (TCP), User Datagram Protocol (UDP) or Transport Layer Security (TLS) / Secured Syslog.
* Request For Comments (RFC) 3164 or 5424 format.

Notification Design Decisions for hybrid implementations

Decision Point | Design Decision | Justification
--- | --- | ---
Mail notifications | Enabled for both suspicious activities and health alerts | Notify relevant security teams when unwarranted and unauthorised activities occur.
Syslog notifications | Enabled for both suspicious activities and health alerts | If the Agency has an existing SIEM solution enable the SIEM to gather all possible security-related events.

### Azure ATP – integration with Defender ATP

Azure ATP supports native integration with Defender Advanced Threat Protection (Defender ATP).

The purpose of this is to combine Azure ATP's monitoring of AD and DCs specifically, with Defender ATP's monitoring of general endpoints, to provide a single interface that combines events and alerts from both.

Integration with Defender ATP Design Decisions for hybrid implementations

Decision Point | Design Decision | Justification
--- | --- | ---
Integration with Defender ATP | Enabled | To integrate data feeds and alerts from both products.

### Azure ATP – firewall

As Azure ATP is reliant upon the Azure ATP portal and the Azure ATP sensor firewall ports are required to be opened to allow communication between infrastructure (Domain Controllers and Standalone servers) and Azure ATP. These port configurations are updated frequently and are available online from Microsoft ([Azure ATP Configure Proxy](https://docs.microsoft.com/en-us/azure-advanced-threat-protection/configure-proxy), [Azure ATP Sensor Firewall ports](https://docs.microsoft.com/en-us/azure-advanced-threat-protection/atp-prerequisites)).

It is important to note the traffic between the client and the Azure ATP portal offering is TLS1.2 encrypted. Configuration will occur on the proxy, external gateway and local infrastructure servers of the Agency as required.

Firewall rules and proxy allow lists will be implemented as part of the Azure ATP solution.

* Azure ATP service location – As mentioned previously Azure ATP data centres are deployed in the United States, Europe, and Asia. The Agency's Azure ATP instance when created will be in the data centre that is geographically closest to the Agency's Azure Active Directory service. This will assist in determining which service location to utilise for generating allow lists.
* Maximal security and data privacy – Azure ATP cloud services use certificate based mutual authentication for communication between Azure ATP cloud backend and all Azure ATP sensors. To make the authentication process seamless as possible if the Agency's environment utilises SSL inspection then the inspection should be configured for mutual authentication.

Further details on the firewall configuration for the solution can be found in the Network Configuration ABAC.

### Azure ATP – integration with MCAS

MCAS is a Cloud Access Security Broker that provide visibility, control over data travel and powerful analytics to identify and deal with cyberthreats. Integrating Azure ATP with MCAS extends this capability to hybrid environments and presents all Azure ATP Suspicious Activity (SA) alerts to the MCAS dashboard, reducing the need for security analysts to monitor multiple consoles.

To connect Azure ATP to MCAS the user enabling the setting must be an Azure AD Global Admin. Integration is enabled in the MCAS console and does not require configuration from the Azure ATP console. MCAS allows Agencies to access the Azure ATP data within a single monitoring and management portal reducing the number of monitoring consoles required, however the following needs to be considered:

* Alerts – MCAS can display Azure ATP alerts within the Alerts queue. MCAS also provides additional alert filtering not available within Azure ATP.
* Alerts management – Management of alerts can be performed in both MCAS and Azure ATP portals. Closing alerts in one portal will not necessarily close the same alert in the other portal. It is recommended to choose which portal will be used to manage and remediate alerts to avoid duplicate effort.
* SIEM notification – both Azure ATP and MCAS can be configured to send alert notification to a SIEM. In the event this is configured duplicate SIEM notifications for the same alerts will be visible within SIEM under different alert ID's. To avoid this situation, it is recommended to choose which portal will be used to perform alert management and then only this portal is to be configured to send alert notification to a SIEM.
* Activities – MCAS displays Azure ATP alerts also in the activity log. MCAS provides additional activity filtering not available within Azure ATP.

Integration with MCAS Design Decisions for hybrid implementations

Decision Point | Design Decision | Justification
--- | --- | ---
Integration with MCAS | Enabled | To integrate data feeds and alerts from both products.

### Microsoft Defender Advanced Threat Protection

Microsoft Defender ATP (formally Windows Defender Advanced Threat Protection) extends the standard Microsoft Defender capabilities to provide additional reporting, pre-breach protection, post-breach detection, automation, and response. Microsoft Defender ATP does not require an agent on the endpoint or any on-premises infrastructure, instead it leverages Microsoft's cloud platform. A single dashboard allows administrators to monitor the compliance and security of all ATP-enabled devices, as well as providing ISO27001 certified Endpoint Detection and Response (EDR) functionality.

Microsoft Defender Advanced Threat Protection can be configured with the following options:

* Data Retention Period - Data Retention Period defines how long gathered telemetry data is stored and available for use in online reporting.
* Alert Notifications - Alert Notifications are configurable rulesets that allow a person or group of people to receive a notification on the occurrence of a pre-set event.
* Secure Score Baseline - Secure Score Baseline configures the product baselines for calculating the score of Microsoft Defender security controls on the secure score dashboard. If third-party solutions are in use the corresponding controls should be excluded from the calculations.
* Administration Roles and Machine Groups - Administration roles provide the ability to configure role-based access and granular options for regulating permissions to portal features and data. Machine groups enabled machines to be organised into groups and apply configured automated remediation levels and assigned administrators.

Microsoft Defender ATP Design Decisions for all agencies and implementation types.

Decision Point | Design Decision | Justification
--- | --- | ---
Microsoft Defender ATP | Configured | To provide increased security and meet the requirements of this document.
Sample Collection | Enabled | Required configuration to enable<br>Deep Analysis on files when required. 
Data storage location | US | As of June 2019, the available Azure data centres to host Windows Defender ATP are located in the US, UK and Europe. All data used by Windows Defender ATP is protected at minimum by Advanced Encryption Standard (AES) 256-bit encryption, both at rest and inflight.  The US has been selected due to policy alignment under the Cloud Act. 
Data Retention Period | 180 Days | Default configuration and suitable for the organisation's requirements.
Alert Notifications | Send Information, Low, Medium, High to Security team. | Alerts will be sent to agency's Cyber Intelligence team for action.
Secure Score Baseline | Windows Defender Antivirus<br>Windows Defender Application Control<br>Windows Defender Exploit Guard<br>Windows Defender Application Guard<br>Windows Defender SmartScreen<br>Windows Defender Firewall<br>Windows Defender Credential Guard<br>Windows Defender Attack Surface Reduction | Meets the requirements of this design
Administration Roles | Full Administrator:<br>Admin_{agency}-securityadmin | Administrative roles will be segregated as per the ACSC Restricting Administrative Privileges (April 2019) guide.
Machine Groups | All Clients | Machines will be segregated into groups with automated remediation levels assigned the administrators that monitor these groups. Groups will be developed with the Agency and documented in the As-Built-As-Configured documentation.
Machine onboarding and Configuration | Configured | Onboarding and configuration will be performed by Intune.
SIEM Integration | Configured | To meet the security logging requirements of this solution.

### Log Analytics

Log Analytics is a component of the Azure Monitor solution and also forms the storage location for the data analysed by Azure Sentinel. It is utilised for log ingestion and querying. Logs can be ingested into Log Analytics in several ways including via:

* Diagnostic Settings
* Sentinel Connectors
* HTTP Post

Log data stored in Log Analytics data can be consumed in various ways:

* Azure Portal -- Azure Portal allows you to create Log queries and analyse the results.
* Azure Monitor Alert rules -- An alert rule is a search that is automatically run at regular intervals. The results are inspected to determine if an alert in Azure Monitor should be generated.
* Azure Dashboards -- Dashboards can be used per Azure user to visualise data gathered from Log Analytics, these dashboards can be shared amongst Azure administrators.
* Export -- Data from Azure Monitor can be imported into Excel or Power BI for further visualisation.
* PowerShell -- PowerShell from a command line or using Azure Automation, can programmatically retrieve data for various use-cases.
* Azure Monitor Logs API -- The native API, uses REST to retrieve log data from the workspace.
* Azure Sentinel -- Azure Sentinel allows dashboarding, alerting, and analysis of the logs.

Log Analytics is billed per gigabyte (GB) of data ingested and retained into the service. When ingesting into a SIEM, data retention periods can be shortened.

Log Analytics Design Decisions for all agencies and implementation types.

Decision Point | Design Decision | Justification
--- | --- | ---
Log Analytics Workspace | Deployed | The Log Analytics workspace will primarily be used to store log data for Microsoft Endpoint Manager managed workloads and Azure AD sign in logs.
Pricing mode | Per GB | Log Analytics pricing is based on data consumed.
Incurs Subscription Cost? | Yes | Log Analytics pricing is based on data consumed. Data Volume could be reduced to 90 days if the Agency has an existing SIEM for further custom log analysis.

Log Analytics configuration for all agencies and implementation types

Configuration | Value | Description
--- | --- | ---
Workspace Name | agency-log-workspace | Log workspace name to be confirmed by the Agency
Azure Subscription | Agency subscription | Configured by Office 365
Region | Australia Central | Closest location of Log Analytics to the Agency
Log retention | Retention Period: 1 year<br>Data Volume Cap: Off | One year aligns with other data retention periods in this solution and meets the system requirements
Enabled Diagnostic Settings | Microsoft Endpoint Manager, Azure AD | Ensures logs are ingested by log analytics. 
Log Analytics Contributor Group | rol-agency-log-admin | Log Analytics Contributor group name to be confirmed by the Agency

### Security Information and Event Management

SIEM is a combination of tools and services that provide insights into a network. The tools and services are classed as either Security Information Management (SIM) or Security Event Management (SEM). SIEM tools gather log files from devices for analysis and reporting. Through this process security threats and events can be identified. SIEM tools provide real-time analysis of log and event data to alert administrators to potential issues like security threats. When combined into a SIEM the Agency is provided with:

* Real-time visibility for the Agency's systems.
* Centralised event log management meaning data is consolidated from multiple sources across the network.
* Correlation of events gathered from different logs and security sources.
* Automated security event notification for administrators.

In a hybrid environment the SIEM can be located either on-premises or in the cloud. In either location, all logs from the environment should be sent to one SIEM. This ensures maximum insight and creates a single pane of glass. To ensure that all logs from the cloud can be ingested by the SIEM compatibility of the SIEM product with Microsoft Office 365, Microsoft Azure, and Azure Monitor should be investigated.

SIEM Design Decisions for all agencies and implementation types.

Decision Point | Design Decision | Justification
--- | --- | ---
SIEM Solution | Not Configured | SIEM Solution configuration is custom to each agency based on its specific requirements. This blueprint does not specify a SIEM and as such does not offer configuration guidance, however Agencies should consider their operational requirements in this area. This blueprint provides guidance on Azure logs, Defender ATP and Office ATP which audit most Azure, Defender, and Office logs for up to two years. These technologies send alert emails to Global Administrators and selected Office 365 administrators.
Azure log ingestion | Not Configured | SIEM Solution configuration is custom to each agency based on its specific requirements. This blueprint does not specify a SIEM and as such does not offer configuration guidance, however Agencies should consider their operational requirements in this area. This blueprint provides guidance on Azure logs, Defender ATP and Office ATP which audit most Azure, Defender, and Office logs for up to two years. These technologies send alert emails to Global Administrators and selected Office 365 administrators.
Office 365 log ingestion | Not Configured | SIEM Solution configuration is custom to each agency based on its specific requirements. This blueprint does not specify a SIEM and as such does not offer configuration guidance, however Agencies should consider their operational requirements in this area. This blueprint provides guidance on Azure logs, Defender ATP and Office ATP which audit most Azure, Defender, and Office logs for up to two years. These technologies send alert emails to Global Administrators and selected Office 365 administrators.

## Client configuration

### Microsoft Endpoint Manager - Intune

Microsoft Endpoint Manager - Intune (Intune) is an Microsoft 365 service that provides Mobile Device Management (MDM) and Mobile Application Management (MAM) capabilities for Apple iOS, Google Android and Microsoft Windows devices to enhance security and protection.

Intune manages which devices can access corporate data, protects company information by controlling the way data is shared, and enforces device configuration to ensure security requirements are met. It does this via:

* Device Enrolment Profiles – Prior to managing devices in Intune they must be enrolled as either Personal or Corporate devices. These can either be self-enrolled or automatically enrolled.
* Device Compliance Policies – Device Compliance Policies are rules, such as device PIN length or encryption requirements, that can be applied to devices. These rules must be met before a device is considered compliant. Device Compliance can then be used by services such as Conditional Access.
* Device Configuration Profiles - Device Configuration Profiles provide the ability to control settings and features on supported endpoints. These include, device and user settings, browser settings and hardware settings. Device Configuration Profiles can be deployed to specific users or devices in Azure AD groups.
* Device Security Baselines – Security baselines are pre-configured groups of Windows settings that are recommended by Microsoft security teams. The security baselines are templates and are used to create a profile that is specific to the environment for deployment.
* Client Applications – Client applications can be delivered to devices registered in Intune based on device type and group membership. Application types that can be distributed include store apps, MS Office suite, MS Edge browser, web links, line of business and Win32 applications. Monitoring of application distribution is provided.
* Software Updates – Software update policies store the configuration of updates without the updates themselves. This prevents the need to approve individual updates allowing for a faster turnaround time. Individual policies can be created and targeted to different groups of devices.

When devices are enrolled into Intune, authorised administrators are able to view hardware details, how the device is used, and what compliance levels currently are for the device's software, hardware, and operating system.

Additionally, Intune can present a customised Company Portal to end users which can be used to install and launch applications or websites via single sign-on (SSO) authentication.

Intune is a component of EMS and integrates with other EMS components such as Azure AD and Azure Information Protection (AIP) natively. This allows for total granular visibility of all endpoints within the Enterprise Mobility Management sphere and simplifies the approach for management.

To complement this visibility, an Intune Data Warehouse can be deployed to capture and create custom reports from Intune data using a reporting service. This can assist in gaining insight into which users are using Intune, what licences are being used, operating system and device breakdowns, and compliance trends. The Data Warehouse also has the capability to export directly to Power BI and create interactive & dynamic reports.

Intune can also configure Windows Information Protection (WIP) polices. WIP can be deployed to:

* Protect against potential data leakage – WIP protects against potential data leakage without any impact to user functionality.
* Protect enterprise applications and data - WIP protects against accidental data leakage on enterprise-owned and personal devices. This can occur without changes to the corporate environment or applications.

Within WIP, Network boundaries are created as a network perimeter that controls what applications can be accessed on the network.

Clients managed by Intune are configured to refresh their status on an 8-hour interval. During this refresh. their policy compliance, configuration profile, and app assignments are checked. If the client is recently enrolled then the compliance, non-compliance, and configuration check-in runs more frequently.

Intune Design Decisions for cloud native implementations

Decision Point | Design Decision | Justification
--- | --- | ---
Co-management | Disabled | Co-Management is disabled as this is not a function that is used in a cloud only solution.
Enrolled Device Types | Windows 10: 10.0.17134 (minimum)<br>iOS: 14.0 | The use of Windows 10 on designated hardware is mandatory.<br>The following platforms will be disabled:<br>macOS<br>Android
Device Compliance | Enabled | Device Compliance is enabled. All devices will be Intune enrolled and have a custom set of compliance policies applied.
Device Enrolment | Enabled | All users must be enrolled to ensure device compliance.
Company Portal | Enabled | The Company Portal is enabled for application deployment. Applications to be deployed will be set by requirements.
Conditional Access | Enabled | Conditional Access is enabled. It will leverage device & user compliance to allow or disallow access to the corporate environment.
Mobile Device Management (MDM) | Enabled | MDM will be used to control what a user can and cannot do on their mobile device defined by policies set by administrators.
Mobile Application Management (MAM) | Enabled | MAM will be used to ensure that users have access to the apps they need to do their work.
Windows Information Protection mode | Configured | Default settings prevent copying and pasting of data between 'work' locations and other 'personal' locations.
Network Boundaries | Cloud resources | Network boundaries create a list of resources that are considered to be on the enterprise network. These boundaries are used to apply policies that reside in these locations.
Cloud Resources Protected via Network Boundaries  | SharePoint<br>Office 365 | Different policies will be created depending on the network location of the client.
Intune Data Warehouse | Not enabled | While this feature is available, it will not be deployed for the solution. 
**Self Service Group Management** | | |
Owners can manage group membership requests in the Access Panel | No | Group creation and modification is to be locked down and controlled by authorised personnel, such as service desk staff, or Administrators.
Restrict access to Groups in the Access Panel | No | Accessing groups is an Administrative function and has been locked down to Administrators.
**Security Groups** | | |
Users can create security groups in Azure portals | No | Group creation and modification is to be locked down and controlled by authorised personnel, such as service desk staff, or Administrators.
**Microsoft 365 groups** | | |
Users can create Microsoft 365 groups in the Azure portals | No | Group creation and modification is to be locked down and controlled by authorised personnel, such as service desk staff, or Administrators.
**Directory-wide Groups** | | |
Enable an "All Users" group in the directory | No | This group is not required. All users to be a member of a controlled group.

### Intune - mobile application management

Mobile Application Management (MAM) allows the management and protection of an Agency's data within an application. It controls data flows into and out of the application container which houses corporate data.

Using MAM, a corporate app that contains sensitive data can be managed on a wide variety of devices. Many productivity apps, such as the Microsoft Office apps, can be managed by Intune MAM. MAM can protect data within the application container using authentication methods and copy/paste controls, but these controls must be balanced with any MDM controls of the device to maintain usability of the solution.

When deploying a hybrid solution, the management of Windows devices should be considered when choosing to implement MAM for clients. Management solutions such as Group Policy and MECM can provide functionality to control applications which negates the use of MAM on Windows machines.

Mobile Application Management Design Decisions for cloud native implementations

Decision Point | Design Decision | Justification
--- | --- | ---
Mobile Application Management Method | Windows 10 – Intune<br>iOS - Intune | Mobile applications (Windows 10 and iOS) will be deployed via Intune.
Applications Managed | Microsoft Azure Information Protection<br>Microsoft Corporate Portal<br>Adobe Reader<br>Microsoft Suite - <br>Outlook<br>Word<br>Excel<br>Teams<br>PowerPoint<br>OneNote<br>OneDrive | These core Microsoft business applications will be managed via Intune as they will be deployed to all Windows 10 and iOS users.

Mobile Application Management Design Decisions for hybrid implementations

Decision Point | Design Decision | Justification
--- | --- | ---
Mobile Application Management Method | Windows 10 – Not Configured<br>iOS - Intune | Agencies operating in hybrid environments can elect to have Windows 10 applications managed by an existing management solution such as MECM and Group Policy, or to manage Windows 10 applications via Intune. This blueprint offers an example of using MECM as the Windows10 Management tool for illustrative purposes however agencies can elect to have Intune as the primary MAM method for both platforms without affecting cyber security postures.<br>Mobile applications (iOS) will be deployed via Intune.
Applications Managed | Microsoft Azure Information Protection<br>Microsoft Corporate Portal<br>Adobe Reader<br>Microsoft Suite - <br>Outlook<br>Word<br>Excel<br>Teams<br>PowerPoint<br>OneNote<br>OneDrive | Agencies operating in hybrid environments can elect to have Windows 10 applications managed by an existing management solution such as MECM and Group Policy, or to manage Windows 10 applications via Intune. This blueprint offers an example of using MECM as the Windows10 Management tool for illustrative purposes however agencies can elect to have Intune as the primary MAM method for both platforms without affecting cyber security postures.<br>Mobile applications (iOS) will be deployed via Intune.

### Intune - enrolment

Device enrolment registers the Windows 10 and iOS devices into the corporate device management solution and ensures the device is then able to be managed by administrators.

Microsoft Intune provides a mechanism for enrolling devices into Azure AD. Once registered the device is populated into Intune policy groups using dynamic membership. This ensures that the device meets the compliance policy, monitored, and secured to the Agency's security requirements. 

Microsoft Intune provides three separate experience in enrolling the iOS devices into the Agency's Azure Active directory. The enrolment experiences are:

* Device Enrolment Program (DEP) – Device Enrolment Program is a managed device enrolment process. The devices serial number is registered with Apple Business Manager allows Intune to bypass Assisted Setup by preconfigure device settings. The user's account will be assigned to the device. The device will be marked as a Supervised device.
* Device Enrolment Manager (DEM) – Device Enrolment Manager assigns a single Azure Active Directory account as the owner of the device. The end users cannot administer or purchase any apps on the device.
* User Enrolment – User enrolment process requires users set up the iOS device and manually install Company Portal to register the device as Intune enrolled device. The device will be marked as a BYOD device.

Intune Enrolment Design Decisions for cloud native implementations

Decision Point | Design Decision | Justification
--- | --- | ---
Windows Enrolment | Configured | Windows 10 devices must be enrolled in Intune prior to management of the device.
iOS Enrolment | Configured | iOS devices must be enrolled in Intune prior to management of the device.

### Intune - co-management

Co-management provides the ability to manage devices via MECM and Intune. 

For a deployment to be enabled for co-management, devices must be Azure AD joined, be enrolled in Intune and have the MECM client installed.

Once co-management is enabled, management tasks such as compliance policies, Windows Update policies, resource access policies, and endpoint protection, can be moved from MECM management over to Intune as required.

Microsoft cloud-hosted services offer the benefit of maintaining cadence with the latest technology updates from Microsoft with reduced effort required by IT BAU teams. Microsoft Intune and Microsoft's co-management strategy is constantly evolving with additional services published regularly.

Intune deploys and manages first-party Microsoft applications in a simple manner but does not allow for a large amount of customisation of update schedule, granular application deployment or application add-ons. Intune does not provide the ability to deploy and update third-party applications in a simple manner at time of writing.

Intune also provides a patching mechanism which simplifies the deployment of Microsoft first-party updates for applications and Windows 10 but does not allow granular control over patches.

The following figure provides an overview of co-management. Figure reproduced from https://docs.microsoft.com/en-us/mem/configmgr/comanage/overview

![Co-management overview](/assets/images/platform-intune-comgmt.png)

Intune Co-Management design considerations and decisions apply to the HYBRID solution.

Decision Point | Design Decision | Justification
--- | --- | ---
Co-management | Enabled | The Microsoft co-management approach will enable Agency to strategically move device management from on-premises to the cloud in a staged manner.
Compliance policies controlled by | Intune preferred | Compliance and remediation policies to controlled via Intune. Staged migration to be completed from MECM if previously in use.
Device Configuration policies controlled by | Intune preferred | Device configuration policies to be controlled via Intune. Staged migration to be completed from MECM if previously in use.
Endpoint Protection policies controlled by | Intune preferred | Endpoint protection, including the Windows Defender products and features are controlled via Intune policies. Staged migration to be completed from MECM if previously in use.
Resource Access policies controlled by | Intune preferred | Resources in this instance refers to VPN profiles, Wi-Fi profiles, certificate profiles, etc. are controlled via Intune policies. Staged migration to be completed from MECM if previously in use.
Office Click-to-Run policies controlled by | Intune preferred | Office Click-to-Run application deployment and updates to be managed through Intune. Staged migration to be completed from MECM if previously in use.
Windows Update policies controlled by | Intune preferred | Windows 10 updates will be managed via Intune update rings. Staged migration to be completed from MECM if previously in use.
MECM minimum version | At least MECM update 1802 | Compatible with co-management and determined by the Agency.
Enrolled Device Types | Windows 10: 10.0.17134 (minimum) | The use of Windows 10 on designated hardware is mandatory.<br>The following platforms will be disabled:<br>macOS<br>Android<br>iOS<br>Note: iOS is permitted but controlled by Intune only
Device Compliance | Enabled | Device Compliance is enabled. All devices will be Intune enrolled and have a custom set of compliance policies applied.
User Enrolment | Enabled | All users must be enrolled to ensure device compliance.
Company Portal | Enabled | The Company Portal is enabled for application deployment. Applications to be deployed will be set by Agency requirements.
Conditional Access | Enabled | Conditional Access is enabled. It will leverage device & user compliance to allow or disallow access to the corporate environment.
Mobile Device Management (MDM) | Enabled | MECM will be the MDM authority for the solution, with Intune inspecting compliance.
Mobile Application Management (MAM) | Disabled | Not required as Group Policy will configure application controls.

### Intune - Windows AutoPilot

Windows Autopilot provides the ability to set up and pre-configure new devices without the need for on premises infrastructure. It is also possible to use Windows Autopilot to reset, repurpose and recover devices.

Windows Autopilot provides the ability to:

* Automatically join devices – Azure Active Directory (Azure AD).
* Auto-enrol devices – Auto-enrol MDM services, such as Microsoft Intune.
* Restrict the Administrator – Restrict administrator account creation.
* Create and auto-assign devices – Auto assign to configuration groups based on a device's profile.

![Autopilot Deployment](/assets/images/platform-autopilot.png)

Intune Windows AutoPilot Design Decisions for cloud native implementations

Decision Point | Design Decision | Justification
--- | --- | ---
Automatically Join Devices | Azure Active Directory (Azure AD) | Devices will automatically joint the Azure Active Directory.
Auto-enrol devices | Configured | Enrolled automatically into Intune MDM.
Restrict the Local Administrator Account | Configured | Aligns with the ACSC Hardening Microsoft Windows 10 1709 Workstations.
Create and auto-assign devices | Configured | For ease of management and enrolment for devices within Agency.
Deployment profile | Refer to DTA – Intune Enrolment -ABAC document | Deployment profile will ensure that all workstations are configured in accordance with the Agency standards with no user intervention.

### Intune - device compliance

Device Compliance Policies are rules, such as device PIN length or encryption requirements, that are applied to devices. These policies must be met before a device is considered compliant, the device compliance status can then be used by services such as Conditional Access to grant or disallow access to applications or services.

Microsoft Intune can control access to resources by interrogating endpoints and determining whether they meet a minimum list of features and are judged as "compliant". Compliance can be assigned a grace period where a device which is not judged as compliant can still access resources for a period or be blocked immediately.

Each compliance policy can be edited to ensure that devices are tested before being allowed access to corporate resources.

Device Compliance Profiles deployed ensure a strong security posture for the entire Windows 10 and iOS fleet. Compliance Policies allow the Agency to ensure that baselines are met prior to access being granted to any corporate applications or data. The Windows 10 compliance policy settings include:

* Device Health – This includes BitLocker status and whether code integrity is enabled.
* Device Properties – Including a minimum and maximum Operating System version.
* Configuration Manager Compliance – Whether the endpoint is compliance will all Configuration Manager evaluations. This is especially applicable in a co-managed scenario such as this deployment.
* System Security – Password compliance, standards, length, and complexity. This also includes device level firewall, TPM, Antivirus, Antispyware, and Microsoft Defender Antimalware settings.
* Microsoft Defender ATP – Configures the maximum allowed machine risk score, if exceeded the device is marked as noncompliant.

Device Compliance Design Decisions for all agencies and implementation types.

Decision Point | Design Decision | Justification
--- | --- | ---
Compliance Assessment | Configured | Since mobile devices routinely leave the office environment, and the protection it affords, it is important that organisations develop a mobile device usage policy governing their use.

### Intune - Windows Hello for Business

Windows Hello for Business provides the ability to replace passwords with strong two-factor authentication for Windows 10 devices. Two-factor authentication is achieved with Windows Hello for Business by the following process:

* The device creates as a key tied to the Trusted Platform Module (TPM) (something users have). Should the device not have TPM 2.0, software shall be used.
* A user input is then provided in the form of a PIN (something users know) or biometric input (something users are) to verify.

Agencies are to align their password policy to insure that the Windows Hello for business PIN does not match any other passwords that the user may use.

For Hybrid deployments agencies will need to refer the [Planning a Windows Hello for Business Deployment](https://docs.microsoft.com/en-us/windows/security/identity-protection/hello-for-business/hello-planning-guide) and [Windows Hello for Business Deployment Prerequisite Overview](https://docs.microsoft.com/en-us/windows/security/identity-protection/hello-for-business/hello-identity-verification) documentation to determine the deployment type required based on its environment and requirements.

Windows Hello for Business Design Decisions for hybrid implementations

Decision Point | Design Decision | Justification
--- | --- | ---
Trust type | Configured | Defines how each Windows Hello for Business client authenticates to the on-premises Active Directory. Trust types available are key trust and certificate trust.
Management | Configured | Provides how agencies manage users and devices. The three ways Windows Hello for Business can be managed include Group Policy, Modern Management, and mixed

### Intune - device configuration

Device Configuration Profiles provide the ability to control settings and features on supported endpoints. These include, device and user settings, browser settings, and hardware settings. Device Configuration Profiles can be deployed to specific users or devices by using Azure AD groups.

There are many supported platforms, each of which have several profile sub-types that they offer configuration for, at the time of writing, the following platforms are supported:

* Android device administrator
* Android Enterprise
* iOS/iPadOS
* macOS
* Windows Phone 8.1
* Windows 8.1 and later
* Windows 10 and later

Within each platform there are number of profile types allowing many settings to be configured. The profile types and settings that are configurable vary depending on the platform.

In general terms, configuration profiles either configure the device for use by the user or secure the device.

Custom profiles can be created for a platform although this should be considered a last resort if the settings are not available in any other way.

In a co-managed state, these settings may be superfluous to existing Group Policies and SOE settings.

Device Configuration Design Decisions for all agencies and implementation types.

Decision Point | Design Decision | Justification
--- | --- | ---
iOS policies | Configured | Intune policies are applied easing management.
Device security policies | Configured by exception | Security baselines as discussed below provide a better option when the settings are available.

Additional Device Configuration Design Decisions for cloud native implementations

Decision Point | Design Decision | Justification
--- | --- | ---
Windows 10 and later polices | Configured | Intune policies are applied easing management.

Additional Device Configuration Design Decisions for hybrid implementations

Decision Point | Design Decision | Justification
--- | --- | ---
Windows 10 and later polices | Not Configured | Management solution such as MECM and Group Policy are applied to manage settings.

### Intune - security baselines

Security Baselines are pre-configured groups of Windows settings that are recommended by Microsoft. The security baselines are templates and are used to create a profile that is specific to the environment for deployment and applied to enrolled devices.

Within Intune, pre-configured Security Baselines profiles can be associated to devices to align them with Microsoft security best practices. These profiles contain multiple device specific configuration profiles and control several security related settings such as, but not limited to:

* App Runtime
* Autoplay
* BitLocker

These baselines provide robust security guidelines and are generated by Microsoft.

In the case of a co-managed / hybrid scenario, security baselines should only be used when they do not conflict with settings deployed through MECM and Group Policy.

Security Baselines Design Decisions for cloud native implementations

Decision Point | Design Decision | Justification
--- | --- | ---
Windows 10 Security Baselines | Configured | System configuration is managed via Intune.
Microsoft Defender ATP Baselines | Configured | System configuration is managed via Intune.
Microsoft Edge Baseline | Configured | System configuration is managed via Intune.

Security Baselines Design Decisions for hybrid implementations

Decision Point | Design Decision | Justification
--- | --- | ---
Windows 10 Security Baselines | Configured | Agencies operating in hybrid environments can elect to have system configuration managed by an existing management solution such as MECM and Group Policy, or to manage system configuration via Intune. This blueprint offers an example of using Intune as the system configuration Management tool for illustrative purposes however agencies can elect to have MECM and Group Policy as the primary system configuration method without affecting cyber security postures.<br>System configuration will be deployed via Intune.
Microsoft Defender ATP Baselines | Configured | Agencies operating in hybrid environments can elect to have system configuration managed by an existing management solution such as MECM and Group Policy, or to manage system configuration via Intune. This blueprint offers an example of using Intune as the system configuration Management tool for illustrative purposes however agencies can elect to have MECM and Group Policy as the primary system configuration method without affecting cyber security postures.<br>System configuration will be deployed via Intune.
Microsoft Edge Baseline | Configured | Agencies operating in hybrid environments can elect to have system configuration managed by an existing management solution such as MECM and Group Policy, or to manage system configuration via Intune. This blueprint offers an example of using Intune as the system configuration Management tool for illustrative purposes however agencies can elect to have MECM and Group Policy as the primary system configuration method without affecting cyber security postures.<br>System configuration will be deployed via Intune.

### Intune - information protection

Application protection policies are rules that ensure an Agency's data remains safe or contained in a managed application.

An application protection policy can be a rule that is enforced when the user attempts to access or move "corporate" data, or a set of actions that are prohibited or monitored when the user is inside the app.

Information Protection Design Decisions for all agencies and implementation types.

Decision Point | Design Decision | Justification
--- | --- | ---
MAM or MDM policies | MDM will be used to apply application protection policies. | MAM based policy is not able to manage non-enlightened line of business applications. (Non-Microsoft Office apps).
Desktop Protected Apps | All Microsoft Office desktop applications will be protected. Detailed settings are in the DTA – Platform – ABAC document. | No additional desktop applications are included in this blueprint.
Mobile Apps | Default set will be protected on mobile devices. Detailed settings are in the DTA – Platform – ABAC document. | Default set of mobile apps covers all of the apps in this blueprint.
Network Boundary – Cloud Resources | Default SharePoint URLs will be protected. Detailed settings are in the DTA – Platform – ABAC document. | If additional URLs are identified these can also be added to the Cloud Resources scope.
Network Boundary – Network Domain | Production domain will be protected. Detailed settings are in the DTA – Platform – ABAC document. | If additional network subnets are identified these can also be added to the Network Domain scope.

### Intune - software updates

Windows Update for Business uses Intune to manage the installation of updates and features from Microsoft Windows Update servers. There is no requirement for on-premises servers or storage of update files.

Intune stores the update policy assignments not the updates themselves. No requirement for on-premises infrastructure.

There is no requirement or ability to selectively enable or disable a particular update.

Fast and slow update rings can be configured and assigned to different groups or users or devices allow early adopters to provide a level of validation before all users are provided with updates.

When deploying a hybrid solution, the software and patch updates of Windows devices should be considered. Other management solutions such as MECM and Windows Server Update Service (WSUS) may be servicing the Windows devices for the updates hence duplicating processes.

Software Updates Design Decisions for all agencies and implementation types.

Decision Point | Design Decision | Justification
--- | --- | ---
Servicing Channel | Semi-Annual Channel | Aligns with ACSC guidance for Operating System updates.
Microsoft Product updates | Allow | Aligns with ACSC guidance for product updates.
Windows Drivers | Allow | Aligns with ACSC guidance for driver updates.
Quality Deferral period | 0 days | Aligns with general ACSC guidance for updates.
Feature Deferral | 0 days | Aligns with general ACSC guidance for updates.
Feature Update uninstall period | 10 days | Allows reversal for a short period of time in the event of breaking change updates.

### Intune - iOS

iOS devices will be enrolled with the Intune Agency Portal to gain secure access to Agency data.

After devices are enrolled, they become managed. Agencies can assign policies and apps to the device through a mobile device management (MDM) provider, such as Intune.

Intune iOS Design Decisions for all agencies and implementation types.

Decision Point | Design Decision | Justification
--- | --- | ---
iOS Enrolment | Configured | iOS is commonly deployed across the Commonwealth and can be hardened in line with the ACSC hardening guide for iOS devices
iOS Configuration | Configurations will follow the ACSC hardening guide for iOS devices as much as possible using Intune. Refer to DTA – Intune Configuration - ABAC document. | Aligns with the ACSC Security Configuration Guide Apple iOS 14.

### Registry settings

Registry settings are applied to the Windows registry to modify the underlying operating system. Registry settings are typically changed in a client operating system to configure the system or increase the security of system.

There are several tools available to apply registry settings such as:

* Group Policy
* Intune
* Configuration Manager (MECM)

The ACSC provides the Microsoft Windows and Office 365 hardening guides that defines group policy settings along with other recommendations to significantly reduce the attack surface available to malicious attacks.

Registry settings Design Decisions for cloud native implementations

Decision Point | Design Decision | Justification
--- | --- | ---
Registry Setting Method | Intune | The Agency will use Intune to implement and modify user and computer registry settings to comply with ACSC Windows and Office 365 Pro Plus hardening guides.

Registry settings Design Decisions for hybrid implementations

Decision Point | Design Decision | Justification
--- | --- | ---
Registry Setting Method | Group Policy Objects & MECM | The Agency may utilise management solutions such as Group Policy Objects and MECM to implement and modify user and computer registry settings to comply with ACSC Windows and Office 365 Pro Plus hardening guides.

### Applications

The lifecycle of applications can be managed using Intune. Applications can be deployed, configured, protected and removed.

Managed applications can be provisioned to the following platforms:

* Android
* iOS
* Windows Phone
* Windows 8.1
* Windows 10 and later

Applications types that can be managed include:

* Store Apps (Android, iOS, Windows Phone, Microsoft Store and Google Play)
* The Microsoft Office suite
* Microsoft Edge
* Microsoft Defender ATP
* Web links
* Built-In applications
* Line of Business applications
* Win32 applications
* Android Enterprise system applications

When deploying a hybrid solution, the application lifecycle method should be considered as other management solutions such as MECM may be performing the same service.

Applications Design Decisions for all agencies and implementation types.

Decision Point | Design Decision | Justification
--- | --- | ---
Application Deployment | Configured | Deployment and monitoring of the deployment can be assigned to users or devices.
Application Configuration | Configured | Store applications are easily updated while Win32 applications will need some packaging.
Application Protection | Configured | In combination with conditional access and network boundaries, applications are limited with respect to the copy, paste, forwarding, printing capabilities.
Application Removal | Configured | When applications (or versions of applications) are no longer required they are removed via Agency's nominated management solution.

### Printing

Printing is a legitimate method of data transfer out of an environment. Printing allows users to physically export data from a network and hence also it can be leveraged by malicious actors for data exfiltration. To minimize the risks associated with printing, the location where printing is allowed should be controlled.

Intune can be leveraged to control what printers are available within a device and whether a user is able to add additional local printers.

For a user to leverage an available printer, connectivity and a device driver is often required. The drivers can be delivered and updated using Intune and/or MECM. Connectivity depends on the connected network(s) of the client. The options include:

* Corporate Network printing – In the workplace, the domain joined computers can connect to the print servers and send jobs to the queue.
* External Network printing via Hybrid Cloud Print – Without network connectivity via Citrix, a VPN, or Microsoft Hybrid Cloud Print, direct print server connectivity is not available. Microsoft Hybrid Cloud Print utilises a reverse proxy to communicate with the print servers located within the work network.
* External Network printing via VPN – When direct printer connectivity is not available from external networks, a VPN such as Windows 10 Always-On VPN can allow clients to function as if they were part of the corporate network.

When deploying a hybrid solution, the allocation of printers to users should be considered. Other management solutions such as Group Policy and MECM may be servicing the allocation of printers to devices.

Printing Design Decisions for cloud native implementations

Decision Point | Design Decision | Justification
--- | --- | ---
Printer addition restrictions | Configured | Configured using scripts deployed via Intune. Printers must be supported out of the box in Windows 10.
Unsecure location Printing | Configured | Out of office printing is to be restricted as adequate controls for the creation, storage and destruction of classified content cannot be implemented 

Printing Design Decisions for hybrid implementations

Decision Point | Design Decision | Justification
--- | --- | ---
Printer addition restrictions | Configured | Management tools such as Group Policy can be used to configure printer configurations. Printers will need to be supported out of the box in Windows 10.
Unsecure location Printing | Configured | Out of office printing is restricted as adequate controls cannot be implemented to prevent the creation of classified content on untrusted print device.

## Backup and operational management

### RPO, RTO and retention periods

As with an on-premises environment, backups play an important part of an overall cloud solution capability. It is important that critical information is backed up to enable recovery for scenarios such as accidental deletion or corruption.

To ensure a successful backup, configuration of the following items should be taken into consideration:

* What to backup - understanding what configuration, files and mailboxes that need to be backed up is important. If only a partial configuration is backed up, successful restoration may not be possible.
* Recovery Point Objective (RPO) - RPO defines an acceptable loss of data (in time) for a data type in a data-loss event. RPOs are expressed in hours / days and directly influence the backup approach used, and how backups are performed with sufficient frequency to meet the defined RPO. For example, if an RPO of 12 hours was defined for a given data type, backups of this data type could not be scheduled further than 12 hours apart.
* Recovery Time Objective (RTO) - RTO is used to define the acceptable level of service interruption (in time) between a data loss event and the recovery of the data to a point at which normal service is resumed. When determining RTOs for a given data type, consideration must also be given to any additional recovery process that are undertaken after the restoration of data. The RTO directly influences the type of backups performed and may dictate additional protection mechanisms outside of the backup platform for data types where a very short RTO is defined.
* Legislative Requirements – The essential 8 details that backups of important information, software and configuration settings are performed. More detail on these controls are listed in the [Protective Security Policy Framework](https://www.protectivesecurity.gov.au/sites/default/files/2019-11/pspf-infosec-10-safeguarding-information-cyber-threats.pdf) and the [Essential Eight Maturity Model](https://www.cyber.gov.au/acsc/view-all-content/publications/essential-eight-maturity-model).

It is important that prior to defining the backup and restore policies, RTO and RPO objectives for each data type hosted the environment are defined in line with business requirements and Service Level Agreements (SLA).

There are several enterprise Backup software solutions which can backup data on-premises or in the cloud. Depending on the requirements, a backup solution can cover the following scenarios:

* Backup local data directly to on-premises infrastructure from on-premises.
* Backup local data to on-premises infrastructure and to the Azure storage blob from on-premises.
* Backup cloud data directly from the cloud.

Using the native Office 365 tools only, in combination with recycle bins the following data recovery options and retention periods are available:

* What to backup - understanding what configuration, files and mailboxes that need to be backed up is important. If only a partial configuration is backed up, successful restoration may not be possible.
* Documents, Desktops and Pictures for each user is redirected from the Windows client device to OneDrive using Windows Known Folders providing a backup of data to the cloud.
* OneDrive includes recycle bins allowing recovery of data for up to 93 days.
* SharePoint Online data includes recycle bins allowing recovery of data for up to 93 days.
* Teams chat, channel and files data retained indefinitely by default unless retention policies have been implemented.
* Exchange Online has a recover deleted items from server option allowing recovery of data for up to 30 days.

Retention policies are created that ensure that data is retained forever for:

* Exchange
* SharePoint
* OneDrive
* Microsoft 365 groups
* Skype for Business
* Exchange Public Folders
* Teams channel messages
* Teams chats

Workstation configuration is stored in Intune. (AutoPilot rebuild).

When deploying a hybrid solution, the cloud based native Office 365 tools will not meet an Agencies requirement for on-premises and cloud backups. Agencies will need to investigate third party backup solutions which can backup data on-premises or in the cloud.

RPO, RTO and Retention Periods Design Decisions for all agencies and implementation types.

Decision Point | Design Decision | Justification
--- | --- | ---
Online Exchange Mailboxes | RPO – 24 hours from backup or better<br>RTO - < 48 hours or better | RPO and RTO in relation to cloud backups is for guidance only. Agencies are required to measure these against the business, application, regulatory and security requirements.
Online Exchange Mail Items | RPO – 24 hours from backup or better<br>RTO - < 48 hours or better | RPO and RTO in relation to cloud backups is for guidance only. Agencies are required to measure these against the business, application, regulatory and security requirements.
Online SharePoint | RPO – 24 hours from backup or better<br>RTO - < 48 hours or better | RPO and RTO in relation to cloud backups is for guidance only. Agencies are required to measure these against the business, application, regulatory and security requirements.
Teams | RPO – 24 hours from backup or better<br>RTO - < 48 hours or better | RPO and RTO in relation to cloud backups is for guidance only. Agencies are required to measure these against the business, application, regulatory and security requirements.
OneDrive for Business | RPO – 24 hours from backup or better<br>RTO - < 48 hours or better | RPO and RTO in relation to cloud backups is for guidance only. Agencies are required to measure these against the business, application, regulatory and security requirements.

Additional RPO, RTO and Retention Periods Design Decisions for cloud native implementations

Decision Point | Design Decision | Justification
--- | --- | ---
Restoration tools | Microsoft backup and restoration tools | The Agency will leverage Microsoft Office 365 native tools in the first instance to recover user data.
Items to Backup | Exchange Online<br>SharePoint Online<br>Microsoft Teams<br>OneDrive for Business<br>Microsoft 365 groups | Backups must cover the Microsoft suite of tools at a minimum.
Retention Policies | Up to maximum allowable days per Microsoft Office 365 application | For guidance only. Agencies are required to measure these against the business, application, regulatory and security requirements.

Additional RPO, RTO and Retention Periods Design Decisions for hybrid implementations

Decision Point | Design Decision | Justification
--- | --- | ---
Restoration tools | Third party backup and restoration tools | Agencies should investigate third-party backup tools to backup and restore data on-premises and within the cloud.
Items to Backup | Exchange Online<br>SharePoint Online<br>Microsoft Teams<br>OneDrive for Business<br>Microsoft 365 groups<br>On-premises Exchange<br>On-premises SharePoint | Backups must cover the Microsoft suite of tools at a minimum.
Retention Policies | At discretion of Agency | Retention policies for the backups should be determined by the Agency and measured against the business, application, regulatory and security requirements.

### Data availability

Microsoft Azure services are available globally and provides geographical, regional, data centre, virtual infrastructure, and application resiliency. This allows the Microsoft Azure platform and Office 365 to combat and minimise potential disasters such as customers loss of connectivity to data or loss of data.

Data availability is an important part of making sure that end users have access to the data when they require it. The cloud-based service of Microsoft Office 365 will replicate and store Agency's data in multiple data centres which are geographically dispersed  (see [Exchange data resiliency](https://docs.microsoft.com/en-us/office365/enterprise/office-365-exchange-data-resiliency) and [Office 365 data resiliency](https://docs.microsoft.com/en-us/office365/Enterprise/office-365-data-resiliency-overview)). The Office 365 applications that provide this data availability are:

* Exchange
* SharePoint
* OneDrive
* Teams

The data availability and resiliency of Office 365 cloud service is in-built and managed by Microsoft.

Data Availability Design Decisions for all agencies and implementation types.

Decision Point | Design Decision | Justification
--- | --- | ---
Data Availability | Configured | Microsoft have in-built data availability into the Office 365 cloud services.

## System administration

System Administration is the process of managing, troubleshooting, and maintaining the solution. To complete this, system administrators are granted permissions over the solution. The allocation of permissions to administrators should align with the administrator's role within the organisation and the principle of least privileged access. The allocation of permission to the administrator's role is captured within the Role Based Access Control (RBAC) policy.

### Administrative consoles

To manage and configure the solution, administrators will use various administrative consoles. These consoles are a mixture of server based and web-based consoles that exist internally or in the cloud.

Web based administrative consoles are provided by Microsoft however the urls for these consoles constantly change (refer to [Microsoft Security Portals](https://docs.microsoft.com/en-us/microsoft-365/security/mtp/portals?view=o365-worldwide)). The consoles listed below are correct at the time of writing.

Administration consoles Design Decisions for all agencies and implementation types.

Decision Point | Design Decision | Justification
--- | --- | ---
Azure Portal | Available from web console | The console is available from any managed device using a standard Web browser with internet access. The FQDN used for access will be [https://portal.azure.com](https://portal.azure.com/).<br>Standard users do not have access to the portal.
Microsoft 365 Admin Center | Available from web console | The console is available from any managed device using a standard Web browser with internet access. The FQDN used for access will be [https://admin.microsoft.com/](https://admin.microsoft.com/)
Microsoft Defender ATP Portal | Available from web console | The console is available from any managed device using a standard Web browser with internet access. The FQDN used for access will be [https://securitycenter.windows.com/](https://securitycenter.windows.com/)
MCAS Portal | Available from web console | The console is available from any managed device using a standard Web browser with internet access. The FQDN used for access will be [https://portal.cloudappsecurity.com/](https://portal.cloudappsecurity.com/)
Microsoft 365 Compliance Center | Available from web console | The console is available from any managed device using a standard Web browser with internet access. The FQDN used for access will be [https://compliance.microsoft.com/](https://compliance.microsoft.com/)
Microsoft Endpoint Manager Admin Center | Available from web console | The console is available from any managed device using a standard Web browser with internet access. The FQDN used for access will be [https://endpoint.microsoft.com/](https://endpoint.microsoft.com/)
Microsoft 365 Security Center | Available from web console | The console is available from any managed device using a standard Web browser with internet access. The FQDN used for access will be [https://security.microsoft.com/](https://security.microsoft.com/)
Azure ATP Portal | Available from web console | The console is available from any managed device using a standard Web browser with internet access. The FQDN used for access will be [https://portal.atp.azure.com/](https://portal.atp.azure.com/)
Microsoft Defender Security Intelligence Portal | Available from web console | The console is available from any managed device using a standard Web browser with internet access. The FQDN used for access will be [https://microsoft.com/wdsi/](https://microsoft.com/wdsi/)

### Role Based Access Control (RBAC)

RBAC defines what an end user or administrator can do. In relation to system administration, RBAC provides various roles each of which can only perform certain tasks. For example, help desk staff may be able to only view certain resources, whereas system administrators could view, create, and delete those resources.

When deploying the RBAC model in Azure, there are two scopes where access can be granted. The use of the scopes can be used separately or together depending on the services activated. The scopes include:

* Tenant Scope - Roles within the Tenant Scope allow access to perform tasks at the Tenant and Office 365 administration level. By default, there are 51 built-in RBAC roles that can be assigned at this level to ensure least privilege access is implemented.
* Subscription Scope – Roles within the Subscription Scope allow access to perform tasks within a subscription. Subscription roles do not have permissions at the Tenant Scope level.

Privileged Identity Management (PIM) can be leveraged to enhance the Azure RBAC model. PIM is an implementation of Just-in-time (JIT) access. JIT access ensures that an administrative account only has privileges when required to complete a function. JIT aligns to the principal of Zero Standing Privilege.

Each PIM role assignment can have the following attributes:

* Activation Duration - the Activation Duration attribute specifies the duration to allow the access request, the maximum is 72 hours.
* Approver – the Approver attribute specifies the person or people who can approve role activation requests.
* Notification – the Notification attribute specifies that a pending request is awaiting approval via email.
* Incident Request Ticket – the Incident Request Ticket attribute specifies that the approver add an incident ticket number to the approval request.
* Multi-factor Authentication – the Multi-factor Authentication attribute specifies whether MFA is required for activation.

RBAC Design Decisions for all agencies and implementation types.

Decision Point | Design Decision | Justification
--- | --- | ---
Azure AD Role Based Management | Least Privilege, using PIM | PIM will be utilised to provide Just-in-Time role-based management to ensure elevated access is only provided when required.
PIM Roles | Authentication Administrator<br>Azure Information Protection Administrator<br>Global Administrator<br>Exchange Service Administrator<br>Helpdesk Administrator<br>Intune Service Administrator<br>Office Apps Administrator<br>Power BI Service Administrator<br>Power Platform Administrator<br>Privileged Role Administrator<br>Security Administrator<br>Security Operator<br>SharePoint Service Administrator<br>Teams Communications Administrator<br>Teams Communications Support Engineer<br>Teams Communications Support Specialist<br>Teams Service Administrator<br>User Account Administrator | The configured PIM roles align to the services utilised within the solution.
PIM approval | Automatic approval for all roles except for Global Administrator | Approval will only be required for Global Administrators.
Activation duration | 8 hours | The activation duration will be one workday to ensure that administrative actions are not impeded.
