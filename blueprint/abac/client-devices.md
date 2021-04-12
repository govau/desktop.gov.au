---
layout: page
title: Client Devices
menu: abac
---

The ABAC settings for the Agency workstation configuration can be found below. This includes the Windows Defender Application Control configuration (WDAC). 

Please note, if a setting is not mentioned in the below, it should be assumed to have been left at its default setting.

## Application Control

WDAC utilise one or more policies to defined what drivers and files are whitelisted to run on a Windows 10 devices. Multiple policies can only be leveraged when the policies are deployed utilising Microsoft Endpoint Manager and the Application Control Configuration Service Provider (CSP). Multiple policies will not work on machines pre 1903. When multiple policy files are leveraged the fall into one of the following scenarios:
* **Enforce and Audit Side-by-side** - A base policy configured to enforce and a second base policy configured to audit. This is used to test a new base policy prior to enforcement.
* **Multiple Base Policies Enforced** - Two or more base policies configured in enforce mode. For applications to run they must be whitelisted in both.
* **Supplementary Policies** - A base policy and one or more supplementary policies in enforce mode. For applications to run they must only be whitelisted in one of the policies.  

These WDAC policies can be signed to ensure that they cannot be tampered with. Details on signing policy can be found in the [WDAC Policy - Policy Signing](/blueprint/abac/client-devices.html#wdac-policy---policy-signing) section.

Once implemented, the procedure to remove one or more WDAC policies can be found in the [WDAC Policy - Removal](/blueprint/abac/client-devices.html#wdac-policy---removal) section.

### WDAC Policy - Baseline Policy

The base policy contains the whitelist for the operating system, base applications, and drivers. It can be generated based upon an existing image (i.e a gold image) or leveraging a predefined Microsoft baseline.  

![WDAC Base policy](/assets/images/WDAC_base.png)

#### Gold Image Base

A base policy generated from a gold image machine should only be utlised where all Windows 10 machines in the environment utilise the same image. The procedure to generate the policy is as follows:

1. Deploy the image to a standard organisational Windows 10 device.
2. Ensure all standard software is installed (e.g. Microsoft Office).
3. Verify all drivers are signed (This can be completed utilising sigverif)
4. Using an administrative PowerShell session run
```powershell
$PolicyPath=$env:userprofile+"\Desktop\"
$PolicyName="GoldImagePolicy"
$WDACPolicy=$PolicyPath+$PolicyName+".xml"
New-CIPolicy -Level PcaCertificate -FilePath $WDACPolicy –UserPEs 3> CIPolicyLog.txt -Fallback hash
```
5. Review the CIPolicy.txt file for any items which could not be whitelisted.
6. Using an administrative PowerShell session run
```powershell
Set-RuleOption -FilePath $WDACPolicy -Option 0 # Enabled UMCI
Set-RuleOption -FilePath $WDACPolicy -Option 1 # Enable Boot Menu Protection
Set-RuleOption -FilePath $WDACPolicy -Option 2 # Require WHQL
Set-RuleOption -FilePath $WDACPolicy -Option 3 # Enable Audit Mode
Set-RuleOption -FilePath $WDACPolicy -Option 4 # Disable Flight Signing
Set-RuleOption -FilePath $WDACPolicy -Option 6 # Enable Unsigned Policy
Set-RuleOption -FilePath $WDACPolicy -Option 8 # Require EV Signers
Set-RuleOption -FilePath $WDACPolicy -Option 10 # Enable Boot Audit on Failure
Set-RuleOption -FilePath $WDACPolicy -Option 12 # Enable Enforce Store Apps
Set-RuleOption -FilePath $WDACPolicy -Option 13 # Enable Managed Installer
Set-RuleOption -FilePath $WDACPolicy -Option 16 # Enable No Reboot
Set-RuleOption -FilePath $WDACPolicy -Option 17 # Enable Allow Supplemental
Set-RuleOption -FilePath $WDACPolicy -Option 19 # Enable Dynamic Code Security
```
7. Deploy the file locally in audit mode and validate no additional files require whitelisting
8. Using an administrative PowerShell session run the following to switch the policy into enforce mode
```powershell
Set-RuleOption -FilePath $WDACPolicy -Option 3 -Delete
```
9. Deploy the file locally in enforce mode and validate no additional files require whitelisting
10. [Deploy the file globally](/blueprint/abac/client-devices.html#wdac-policy---deployment) 


#### Predefined Microsoft Base 

Microsoft have developed a number of base policies which provide the whitelisting rules for a standard Windows 10 image. These base policies are located on any Windows 10 machine under `%windir%\schemas\CodeIntegrity\ExamplePolicies`. Within that directory, the policy titled `DefaultWindows_Audit.xml` should be leveraged as the base policy. It includes the rules necessary to ensure that Windows, 3rd party hardware and software kernel drivers, and Windows Store apps will run.  

The procedure to leverage this policy is as follows:
1. Make a copy of the policy and place it into a working directory (i.e. My documents)
2. Open the policy in a XML editor such as Visual Studio code or notepad.
3. Remove all flighting related signers (note they will also need to be removed from the signing scenarios)
4. Using an administrative PowerShell session run
```powershell
$WDACPolicy = path to policy file
Set-RuleOption -FilePath $WDACPolicy -Option 0 # Enabled UMCI
Set-RuleOption -FilePath $WDACPolicy -Option 1 # Enable Boot Menu Protection
Set-RuleOption -FilePath $WDACPolicy -Option 2 # Require WHQL
Set-RuleOption -FilePath $WDACPolicy -Option 3 # Enable Audit Mode
Set-RuleOption -FilePath $WDACPolicy -Option 4 # Disable Flight Signing
Set-RuleOption -FilePath $WDACPolicy -Option 6 # Enable Unsigned Policy
Set-RuleOption -FilePath $WDACPolicy -Option 8 # Require EV Signers
Set-RuleOption -FilePath $WDACPolicy -Option 10 # Enable Boot Audit on Failure
Set-RuleOption -FilePath $WDACPolicy -Option 12 # Enable Enforce Store Apps
Set-RuleOption -FilePath $WDACPolicy -Option 13 # Enable Managed Installer
Set-RuleOption -FilePath $WDACPolicy -Option 16 # Enable No Reboot
Set-RuleOption -FilePath $WDACPolicy -Option 17 # Enable Allow Supplemental
Set-RuleOption -FilePath $WDACPolicy -Option 19 # Enable Dynamic Code Security
```
5. Deploy the file locally in audit mode and validate no additional files require whitelisting
6. Using an administrative PowerShell session run the following to switch the policy into enforce mode
```powershell
Set-RuleOption -FilePath $WDACPolicy -Option 3 -Delete
```
7. Deploy the file locally in enforce mode and validate no additional files require whitelisting
8. [Deploy the file globally](/blueprint/abac/client-devices.html#wdac-policy---deployment) 


### WDAC Policy - Supplementary Policy

Supplementary policies expand on the base policy and allow for whitelisting to be targeted to users and/or groups. Supplementary pollcies can contain both allow and deny rules. A supplementary policy is a base policy until it is linked as a supplementary policy to another base policy. 

Supplementary policies will not work on machines pre 1903. If you are deploying to pre 1903 machines they must be merged into the base policy. [Merge policy](/blueprint/abac/hybrd-client-devices.html#wdac-policy---whitelisting-an-application) instructions are available in the hybrid client whitelisting section.

#### Whitelisting an application

Whitelisting of an application utilising WDAC can be completed utilising a number of methods including Hash (ACSC recommended), file path, and certificate based. Depending on the application, the chosen method of whitelisting may change. Hash whitelisting offers the highest degree of security however it increases the management overhead associated with whitelisting. 

If the managed installer option within the base policy is enabled, whitelisting of applications deployed via the managed installer is not required. 

![WDAC Whitelisting Application policy](/assets/images/WDAC_Intune.png)

Post identification of the appropriate whitelisting method, the following procedure is used to whitelist the application:

1.  Within an administrative PowerShell session navigate to the install directory of the application
2.  Run the following command
```powershell
$whitelistlevel = the chosen method of whitelist e.g. hash
$Outputlocation = the path to output the policy file
$fallbacklevel = the backup method of whitelist e.g. publisher
$directory = the path to be scanned
New-CIPolicy -Level $whitelistlevel -FilePath $Outputlocation -Fallback $fallbacklevel  -ScanPath $directory -UserPEs
```
3. Convert the new base policy to a supplementary policy.

#### Whitelisting all errors in the event log

Whitelisting of an applications blocked or audited by WDAC can be completed utilising PowerShell. This is normally conducted when testing policies as it is important to test a sample of individual application functionality. 

Post identification of the appropriate whitelisting method, the following procedure is used to whitelist the applications identified in the event log:

1.  Within an administrative PowerShell session run the following command
```powershell
$whitelistlevel = the chosen method of whitelist e.g. hash
$Outputlocation = the path to output the policy file
$fallbacklevel = the backup method of whitelist e.g. publisher
New-CIPolicy -Level $whitelistlevel -FilePath $Outputlocation -Fallback $fallbacklevel  -audit -UserPEs
```
2. Convert the new base policy to a supplementary policy.


#### Linking to the base policy

To convert a base policy to a supplementary policy of another base policy they must be linked. Upon linking the policyID of the suppplementary policy will be set to a new GUID. This new guid is required when deploying the supplementary policy via Microsoft Endpoint Manager. 

The procedure to link the policies is as follows:
1. Using an administrative PowerShell session run
```powershell
$SupWDACPolicy = path to the supplementary policy file
$SupWDACPolicyName = Name of the supplementary policy
$WDACPolicy = path to the policy file
$WDACPolicyID = base policy id available within the base policy PolicyID tags
Set-CIPolicyIdInfo -FilePath $SupWDACPolicy -PolicyName $SupWDACPolicyName -SupplementsBasePolicyID $WDACPolicyID -BasePolicyToSupplementPath $WDACPolicy
```

### WDAC Policy - Policy Signing

Prior to deployment of the WDAC policy it can be signed using an internal Certificate Authority code signing certificate or a purchased code signing certificate.

The procedure to sign the policy is as follows:
* Add-SignerRule -FilePath `Path to the XML policy file` -CertificatePath `Path to exported .cer certificate` -Kernel -User –Update
* ConvertFrom-CIPolicy -XmlFilePath `Path to the XML policy file` -BinaryFilePath `Binary output location`
* `Path to signtool.exe` sign -v /n `Certificate Subject name` -p7 . -p7co 1.3.6.1.4.1.311.79.1 -fd sha256 `Binary policy location`

### WDAC Policy - Deployment

The above policies are deployed through Microsoft Endpoint Manager/Intune. The deployment configuration is available within the [Intune Configuration](/blueprint/abac/intune-configuration.html#agency-wdacenablement) section.

### WDAC Policy - Removal

#### Signed Policies

The procedure to remove signed policies is as follows:
1. Create a new signed policy with `6 Enabled: Unsigned System Integrity Policy` enabled.
2. Deploy the policy and rempve the previous policy from deployment.
3. Reboot the machine.
4. Disable the WDAC deployment method.
5. On the machine verify the following files have been deleted (if they have not then delete them)
    * OS Volume\Windows\System32\CodeIntegrity\ `Policy GUID`.cip
    * EFI System Partition\Microsoft\Boot\ `Policy GUID`.cip
6. Reboot the machine.

#### Unsigned Policies

The procedure to remove unsigned policies is as follows:
1. Disable the WDAC deployment method. 
2. On the machine verify the following files have been deleted (if they have not then delete them)
    * OS Volume\Windows\System32\CodeIntegrity\ `Policy GUID`.cip
    * EFI System Partition\Microsoft\Boot\ `Policy GUID`.cip
3. Reboot the machine.

### Appendix 1: Example Microsoft Recommended Block Rules Supplemental Policy

```xml
<?xml version="1.0" encoding="utf-8"?>
<SiPolicy xmlns="urn:schemas-microsoft-com:sipolicy" PolicyType="Supplemental Policy">
  <VersionEx>10.0.0.0</VersionEx>
  <PlatformID>{2E07F7E4-194C-4D20-B7C9-6F44A6C5A234}</PlatformID>
  <Rules>
    <Rule>
      <Option>Enabled:UMCI</Option>
    </Rule>
    <Rule>
      <Option>Enabled:Update Policy No Reboot</Option>
    </Rule>
    <Rule>
      <Option>Required:WHQL</Option>
    </Rule>
    <Rule>
      <Option>Disabled:Flight Signing</Option>
    </Rule>
    <Rule>
      <Option>Required:EV Signers</Option>
    </Rule>
    <Rule>
      <Option>Enabled:Boot Audit On Failure</Option>
    </Rule>
    <Rule>
      <Option>Required:Enforce Store Applications</Option>
    </Rule>
    <Rule>
      <Option>Enabled:Managed Installer</Option>
    </Rule>
    <Rule>
      <Option>Enabled:Dynamic Code Security</Option>
    </Rule>
    <Rule>
      <Option>Enabled:Unsigned System Integrity Policy</Option>
    </Rule>
  </Rules>
  <!--EKUS-->
  <EKUs>
    <EKU ID="ID_EKU_STORE" Value="010a2b0601040182374c0301" FriendlyName="Windows Store EKU - 1.3.6.1.4.1.311.76.3.1 Windows Store" />
    <EKU ID="ID_EKU_WINDOWS" Value="010A2B0601040182370A0306" FriendlyName="" />
    <EKU ID="ID_EKU_ELAM" Value="010A2B0601040182373D0401" FriendlyName="" />
    <EKU ID="ID_EKU_HAL_EXT" Value="010a2b0601040182373d0501" FriendlyName="" />
    <EKU ID="ID_EKU_WHQL" Value="010A2B0601040182370A0305" FriendlyName="" />
  </EKUs>
  <!--File Rules-->
  <FileRules>
    <Deny ID="ID_DENY_KD_KMCI_1_1" FriendlyName="kd.exe" FileName="kd.Exe" MinimumFileVersion="65535.65535.65535.65535" />
    <FileAttrib ID="ID_FILEATTRIB_F_5_1_0_1" FriendlyName="C:\Program Files (x86)\Adobe\Acrobat Reader DC\Reader\AcroRd32.exe FileAttribute" FileName="AcroRd32.exe" MinimumFileVersion="21.1.20145.32109" />
    <Allow ID="ID_ALLOW_A_1_0_0_1" FriendlyName="C:\Program Files (x86)\Adobe\Acrobat Reader DC\Reader\AcroRd32.exe Hash Sha1" Hash="29A96CBC5E001E519E98AB43352E6A134FDE68CA" />
    <Allow ID="ID_ALLOW_A_2_0_0_1" FriendlyName="C:\Program Files (x86)\Adobe\Acrobat Reader DC\Reader\AcroRd32.exe Hash Sha256" Hash="79D1B4BD9AB7C11F063FF49DC5E1263B1763D1E8E0EDFA4AEE95091E825DB6B5" />
    <Allow ID="ID_ALLOW_A_3_0_0_1" FriendlyName="C:\Program Files (x86)\Adobe\Acrobat Reader DC\Reader\AcroRd32.exe Hash Page Sha1" Hash="94C1F7183797A9D0A508362FEAC947932A082E0F" />
    <Allow ID="ID_ALLOW_A_4_0_0_1" FriendlyName="C:\Program Files (x86)\Adobe\Acrobat Reader DC\Reader\AcroRd32.exe Hash Page Sha256" Hash="7C2A0E432B65FA9F9CD605C05CB6A949D9020F6A30D1AB45692B43551515C94A" />
    <Deny ID="ID_DENY_ADDINPROCESS_1_1" FriendlyName="AddInProcess.exe" FileName="AddInProcess.exe" MinimumFileVersion="65535.65535.65535.65535" />
    <Deny ID="ID_DENY_ADDINPROCESS32_1_1" FriendlyName="AddInProcess32.exe" FileName="AddInProcess32.exe" MinimumFileVersion="65535.65535.65535.65535" />
    <Deny ID="ID_DENY_ADDINUTIL_1_1" FriendlyName="AddInUtil.exe" FileName="AddInUtil.exe" MinimumFileVersion="65535.65535.65535.65535" />
    <Deny ID="ID_DENY_ASPNET_1_1" FriendlyName="aspnet_compiler.exe" FileName="aspnet_compiler.exe" MinimumFileVersion="65535.65535.65535.65535" />
    <Deny ID="ID_DENY_BASH_1_1" FriendlyName="bash.exe" FileName="bash.exe" MinimumFileVersion="65535.65535.65535.65535" />
    <Deny ID="ID_DENY_BGINFO_1_1" FriendlyName="bginfo.exe" FileName="BGINFO.Exe" MinimumFileVersion="4.21.0.0" />
    <Deny ID="ID_DENY_CBD_1_1" FriendlyName="cdb.exe" FileName="CDB.Exe" MinimumFileVersion="65535.65535.65535.65535" />
    <Deny ID="ID_DENY_CSI_1_1" FriendlyName="csi.exe" FileName="csi.Exe" MinimumFileVersion="65535.65535.65535.65535" />
    <Deny ID="ID_DENY_DBGHOST_1_1" FriendlyName="dbghost.exe" FileName="DBGHOST.Exe" MinimumFileVersion="2.3.0.0" />
    <Deny ID="ID_DENY_DBGSVC_1_1" FriendlyName="dbgsvc.exe" FileName="DBGSVC.Exe" MinimumFileVersion="2.3.0.0" />
    <Deny ID="ID_DENY_DNX_1_1" FriendlyName="dnx.exe" FileName="dnx.Exe" MinimumFileVersion="65535.65535.65535.65535" />
    <Deny ID="ID_DENY_DOTNET_1_1" FriendlyName="dotnet.exe" FileName="dotnet.exe" MinimumFileVersion="65535.65535.65535.65535" />
    <Deny ID="ID_DENY_FSI_1_1" FriendlyName="fsi.exe" FileName="fsi.exe" MinimumFileVersion="65535.65535.65535.65535" />
    <Deny ID="ID_DENY_FSI_ANYCPU_1_1" FriendlyName="fsiAnyCpu.exe" FileName="fsiAnyCpu.exe" MinimumFileVersion="65535.65535.65535.65535" />
    <Deny ID="ID_DENY_INFINSTALL_1_1" FriendlyName="infdefaultinstall.exe" FileName="infdefaultinstall.exe" MinimumFileVersion="65535.65535.65535.65535" />
    <Deny ID="ID_DENY_KD_1_1" FriendlyName="kd.exe" FileName="kd.Exe" MinimumFileVersion="65535.65535.65535.65535" />
    <Deny ID="ID_DENY_KILL_1_1" FriendlyName="kill.exe" FileName="kill.exe" MinimumFileVersion="65535.65535.65535.65535" />
    <Deny ID="ID_DENY_LXSS_1_1" FriendlyName="LxssManager.dll" FileName="LxssManager.dll" MinimumFileVersion="65535.65535.65535.65535" />
    <Deny ID="ID_DENY_LXRUN_1_1" FriendlyName="lxrun.exe" FileName="lxrun.exe" MinimumFileVersion="65535.65535.65535.65535" />
    <Deny ID="ID_DENY_MFC40_1_1" FriendlyName="mfc40.dll" FileName="mfc40.dll" MinimumFileVersion="65535.65535.65535.65535" />
    <Deny ID="ID_DENY_MS_BUILD_1_1" FriendlyName="Microsoft.Build.dll" FileName="Microsoft.Build.dll" MinimumFileVersion="65535.65535.65535.65535" />
    <Deny ID="ID_DENY_MS_BUILD_FMWK_1_1" FriendlyName="Microsoft.Build.Framework.dll" FileName="Microsoft.Build.Framework.dll" MinimumFileVersion="65535.65535.65535.65535" />
    <Deny ID="ID_DENY_MWFC_1_1" FriendlyName="Microsoft.Workflow.Compiler.exe" FileName="Microsoft.Workflow.Compiler.exe" MinimumFileVersion="65535.65535.65535.65535" />
    <Deny ID="ID_DENY_MSBUILD_1_1" FriendlyName="MSBuild.exe" FileName="MSBuild.Exe" MinimumFileVersion="65535.65535.65535.65535" />
    <Deny ID="ID_DENY_MSBUILD_DLL_1_1" FriendlyName="MSBuild.dll" FileName="MSBuild.dll" MinimumFileVersion="65535.65535.65535.65535" />
    <Deny ID="ID_DENY_MSHTA_1_1" FriendlyName="mshta.exe" FileName="mshta.exe" MinimumFileVersion="65535.65535.65535.65535" />
    <Deny ID="ID_DENY_NTKD_1_1" FriendlyName="ntkd.exe" FileName="ntkd.Exe" MinimumFileVersion="65535.65535.65535.65535" />
    <Deny ID="ID_DENY_NTSD_1_1" FriendlyName="ntsd.exe" FileName="ntsd.Exe" MinimumFileVersion="65535.65535.65535.65535" />
    <Deny ID="ID_DENY_PWRSHLCUSTOMHOST_1_1" FriendlyName="powershellcustomhost.exe" FileName="powershellcustomhost.exe" MinimumFileVersion="65535.65535.65535.65535" />
    <Deny ID="ID_DENY_RCSI_1_1" FriendlyName="rcsi.exe" FileName="rcsi.Exe" MinimumFileVersion="65535.65535.65535.65535" />
    <Deny ID="ID_DENY_RUNSCRIPTHELPER_1_1" FriendlyName="runscripthelper.exe" FileName="runscripthelper.exe" MinimumFileVersion="65535.65535.65535.65535" />
    <Deny ID="ID_DENY_TEXTTRANSFORM_1_1" FriendlyName="texttransform.exe" FileName="texttransform.exe" MinimumFileVersion="65535.65535.65535.65535" />
    <Deny ID="ID_DENY_VISUALUIAVERIFY_1_1" FriendlyName="visualuiaverifynative.exe" FileName="visualuiaverifynative.exe" MinimumFileVersion="65535.65535.65535.65535" />
    <Deny ID="ID_DENY_WFC_1_1" FriendlyName="WFC.exe" FileName="wfc.exe" MinimumFileVersion="65535.65535.65535.65535" />
    <Deny ID="ID_DENY_WINDBG_1_1" FriendlyName="windbg.exe" FileName="windbg.Exe" MinimumFileVersion="65535.65535.65535.65535" />
    <Deny ID="ID_DENY_WMIC_1_1" FriendlyName="wmic.exe" FileName="wmic.exe" MinimumFileVersion="65535.65535.65535.65535" />
    <Deny ID="ID_DENY_WSL_1_1" FriendlyName="wsl.exe" FileName="wsl.exe" MinimumFileVersion="65535.65535.65535.65535" />
    <Deny ID="ID_DENY_WSLCONFIG_1_1" FriendlyName="wslconfig.exe" FileName="wslconfig.exe" MinimumFileVersion="65535.65535.65535.65535" />
    <Deny ID="ID_DENY_WSLHOST_1_1" FriendlyName="wslhost.exe" FileName="wslhost.exe" MinimumFileVersion="65535.65535.65535.65535" />
    <Deny ID="ID_DENY_D_1_1_1" FriendlyName="Powershell 1" Hash="02BE82F63EE962BCD4B8303E60F806F6613759C6" />
    <Deny ID="ID_DENY_D_2_1_1" FriendlyName="Powershell 2" Hash="13765D9A16CC46B2113766822627F026A68431DF" />
    <Deny ID="ID_DENY_D_3_1_1" FriendlyName="Powershell 3" Hash="148972F670E18790D62D753E01ED8D22B351A57E45544D88ACE380FEDAF24A40" />
    <Deny ID="ID_DENY_D_4_1_1" FriendlyName="Powershell 4" Hash="29DF1D593D0D7AB365F02645E7EF4BCCA060763A" />
    <Deny ID="ID_DENY_D_5_1_1" FriendlyName="Powershell 5" Hash="2E3C47BBE1BA99842EE187F756CA616EFED61B94" />
    <Deny ID="ID_DENY_D_6_1_1" FriendlyName="Powershell 6" Hash="38DC1956313B160696A172074C6F5DA9852BF508F55AFB7FA079B98F2849AFB5" />
    <Deny ID="ID_DENY_D_7_1_1" FriendlyName="Powershell 7" Hash="513B625EA507ED9CE83E2FB2ED4F3D586C2AA379" />
    <Deny ID="ID_DENY_D_8_1_1" FriendlyName="Powershell 8" Hash="71FC552E66327EDAA72D72C362846BD80CB65EECFAE95C4D790C9A2330D95EE6" />
    <Deny ID="ID_DENY_D_9_1_1" FriendlyName="Powershell 9" Hash="72E4EC687CFE357F3E681A7500B6FF009717A2E9538956908D3B52B9C865C189" />
    <Deny ID="ID_DENY_D_10_1_1" FriendlyName="Powershell 10" Hash="74E207F539C4EAC648A5507EB158AEE9F6EA401E51808E83E73709CFA0820FDD" />
    <Deny ID="ID_DENY_D_11_1_1" FriendlyName="Powershell 11" Hash="75288A0CF0806A68D8DA721538E64038D755BBE74B52F4B63FEE5049AE868AC0" />
    <Deny ID="ID_DENY_D_12_1_1" FriendlyName="Powershell 12" Hash="7DB3AD53985C455990DD9847DE15BDB271E0C8D1" />
    <Deny ID="ID_DENY_D_13_1_1" FriendlyName="Powershell 13" Hash="84BB081141DA50B3839CD275FF34854F53AECB96CA9AEB8BCD24355C33C1E73E" />
    <Deny ID="ID_DENY_D_14_1_1" FriendlyName="Powershell 14" Hash="86DADE56A1DBAB6DDC2769839F89244693D319C6" />
    <Deny ID="ID_DENY_D_15_1_1" FriendlyName="Powershell 15" Hash="BD3139CE7553AC7003C96304F08EAEC2CDB2CC6A869D36D6F1E478DA02D3AA16" />
    <Deny ID="ID_DENY_D_16_1_1" FriendlyName="Powershell 16" Hash="BE3FFE10CDE8B62C3E8FD4D8198F272B6BD15364A33362BB07A0AFF6731DABA1" />
    <Deny ID="ID_DENY_D_17_1_1" FriendlyName="Powershell 17" Hash="C1196433541B87D22CE2DD19AAAF133C9C13037A" />
    <Deny ID="ID_DENY_D_18_1_1" FriendlyName="Powershell 18" Hash="C6C073A80A8E76DC13E724B5E66FE4035A19CCA0C1AF3FABBC18E5185D1B66CB" />
    <Deny ID="ID_DENY_D_19_1_1" FriendlyName="Powershell 19" Hash="CE5EA2D29F9DD3F15CF3682564B0E765ED3A8FE1" />
    <Deny ID="ID_DENY_D_20_1_1" FriendlyName="Powershell 20" Hash="D027E09D9D9828A87701288EFC91D240C0DEC2C3" />
    <Deny ID="ID_DENY_D_21_1_1" FriendlyName="Powershell 21" Hash="D2CFC8F6729E510AE5BA9BECCF37E0B49DDF5E31" />
    <Deny ID="ID_DENY_D_22_1_1" FriendlyName="Powershell 22" Hash="DED853481A176999723413685A79B36DD0F120F9" />
    <Deny ID="ID_DENY_D_23_1_1" FriendlyName="Powershell 23" Hash="DFCD10EAA2A22884E0A41C4D9E6E8DA265321870" />
    <Deny ID="ID_DENY_D_24_1_1" FriendlyName="Powershell 24" Hash="F16E605B55774CDFFDB0EB99FAFF43A40622ED2AB1C011D1195878F4B20030BC" />
    <Deny ID="ID_DENY_D_25_1_1" FriendlyName="Powershell 25" Hash="F29A958287788A6EEDE6035D49EF5CB85EEC40D214FDDE5A0C6CAA65AFC00EEC" />
    <Deny ID="ID_DENY_D_26_1_1" FriendlyName="Powershell 26" Hash="F875E43E12685ECE0BA2D42D55A13798CE9F1FFDE3CAE253D2529F4304811A52" />
    <Deny ID="ID_DENY_D_27_1_1" FriendlyName="PowerShell 27" Hash="720D826A84284E18E0003526A0CD9B7FF0C4A98A" />
    <Deny ID="ID_DENY_D_28_1_1" FriendlyName="PowerShell 28" Hash="CB5DF9D0D25571948C3D257882E07C7FA5E768448E0DEBF637E110F9FF575808" />
    <Deny ID="ID_DENY_D_29_1_1" FriendlyName="PowerShell 29" Hash="3C7265C3393C585D32E509B2D2EC048C73AC5EE6" />
    <Deny ID="ID_DENY_D_30_1_1" FriendlyName="PowerShell 30" Hash="7F1E03E956CA38CC0C491CB958D6E61A52491269CDB363BC488B525F80C56424" />
    <Deny ID="ID_DENY_D_31_1_1" FriendlyName="PowerShell 31" Hash="27D86C9B54E1A97399A6DC9C9DF9AE030CB734C8" />
    <Deny ID="ID_DENY_D_32_1_1" FriendlyName="PowerShell 32" Hash="917BD10E82C6E932F9C63B9BDCCC1D9BF04510CD8491B005CFFD273B48B5CD1E" />
    <Deny ID="ID_DENY_D_33_1_1" FriendlyName="PowerShell 33" Hash="B3BB2D75AECB34ED316CE54C6D513420186E4950" />
    <Deny ID="ID_DENY_D_34_1_1" FriendlyName="PowerShell 34" Hash="B734F6269A6738861E1DF98EE0E4E7377FAED10B82AAA9731DA0BB1CB366FCCE" />
    <Deny ID="ID_DENY_D_35_1_1" FriendlyName="PowerShell 35" Hash="FF378B465F2C8A87B4092F7C1F96399C0156CEEB" />
    <Deny ID="ID_DENY_D_36_1_1" FriendlyName="PowerShell 36" Hash="9B884CFE78F921042B003574AE30D9E86EE3DCC11E7110A1C92927F13C3F47E6" />
    <Deny ID="ID_DENY_D_37_1_1" FriendlyName="PowerShell 37" Hash="C7B99E8B59182112A3A14BD39880BDCDDD5C724F" />
    <Deny ID="ID_DENY_D_38_1_1" FriendlyName="PowerShell 38" Hash="6E585890C7369D6D8DA85C8B6B7411463BAA1ACAE9CE4197E033A46C897B35E5" />
    <Deny ID="ID_DENY_D_39_1_1" FriendlyName="PowerShell 39" Hash="BA4B3A92123FBCE66398020AFBCC0BCA1D1AAAD7" />
    <Deny ID="ID_DENY_D_40_1_1" FriendlyName="PowerShell 40" Hash="D8D361E3690676C7FDC483003BFC5C0C39FB16B42DFC881FB8D42A1064740B0B" />
    <Deny ID="ID_DENY_D_41_1_1" FriendlyName="PowerShell 41" Hash="1EA5104AE1A7A53F9421E0193B749F310B9261D1" />
    <Deny ID="ID_DENY_D_42_1_1" FriendlyName="PowerShell 42" Hash="66C1B8569019512ACDDC145DA6D348A68DE008BE7C05930AD0EC6927C26061AD" />
    <Deny ID="ID_DENY_D_43_1_1" FriendlyName="PowerShell 43" Hash="4EB2C3A4B551FC028E00F2E7DA9D0F1E38728571" />
    <Deny ID="ID_DENY_D_44_1_1" FriendlyName="PowerShell 44" Hash="30EAC589069FB79D540080B04B7FDBB8A9B1DF4E96B9D7C98519E49A1ED56851" />
    <Deny ID="ID_DENY_D_45_1_1" FriendlyName="PowerShell 45" Hash="E55505B609DD7A22F55C4BA9EDAD5627ECA6A8E8" />
    <Deny ID="ID_DENY_D_46_1_1" FriendlyName="PowerShell 46" Hash="ABDDA9C1EDA9F2344FB5B79890B7FD854D0E3D28BEC26AE33AAD196948AB642D" />
    <Deny ID="ID_DENY_D_47_1_1" FriendlyName="PowerShell 47" Hash="A15964475D213FB752B42E7DCDDBF4B14D623D14" />
    <Deny ID="ID_DENY_D_48_1_1" FriendlyName="PowerShell 48" Hash="61A68B436D828193E0C7B44D2AF83D22A9CB557B90186E4E6AC998CE5E3BFE8A" />
    <Deny ID="ID_DENY_D_49_1_1" FriendlyName="PowerShell 49" Hash="DB0C4B5CA1CBC3B117AB0439C5937B6A263DFD87" />
    <Deny ID="ID_DENY_D_50_1_1" FriendlyName="PowerShell 50" Hash="6D4FB385328CA01700092E1CDF75A97123A95120D5F8A9877FFB4D5A8531380B" />
    <Deny ID="ID_DENY_D_51_1_1" FriendlyName="PowerShell 51" Hash="72F9DCDA6ECDD6906A2538DFE795A2E2CA787BBC" />
    <Deny ID="ID_DENY_D_52_1_1" FriendlyName="PowerShell 52" Hash="F98FEC4A0306BD398F7FB7F611679B7797D32D54D1F2B35D728C0C7A058153ED" />
    <Deny ID="ID_DENY_D_53_1_1" FriendlyName="PowerShell 53" Hash="C980B65B86F780AC93B9458E9657291083CFEDA8" />
    <Deny ID="ID_DENY_D_54_1_1" FriendlyName="PowerShell 54" Hash="F9473493FF53274B8E75EC7E517F324AA0C5644C6F8045D3EF3A1B9A669ECF78" />
    <Deny ID="ID_DENY_D_55_1_1" FriendlyName="PowerShell 55" Hash="C30355B5E6FA3F793A3CC0A649945829723DD85C" />
    <Deny ID="ID_DENY_D_56_1_1" FriendlyName="PowerShell 56" Hash="4EB14099165177F0F3A1FACE32E72CF2DD221DB44155E73AFF94CB7DA195EF22" />
    <Deny ID="ID_DENY_D_57_1_1" FriendlyName="PowerShell 57" Hash="5C6CC1903D3DA2054ECD9A295EEE26F5561E152A" />
    <Deny ID="ID_DENY_D_58_1_1" FriendlyName="PowerShell 58" Hash="0BF8CAB75DAB712FC848DE7CC7DC5C8A10D666515E7535F89146F45AAAF9EF54" />
    <Deny ID="ID_DENY_D_59_1_1" FriendlyName="PowerShell 59" Hash="1443E8F56DEE11EEF5B746E3657C2F953FD4F6EA" />
    <Deny ID="ID_DENY_D_60_1_1" FriendlyName="PowerShell 60" Hash="487CB42795046E885303FC96EA54C3234E1B2072DAEB4F9218C21CC6C39A3223" />
    <Deny ID="ID_DENY_D_61_1_1" FriendlyName="PowerShell 61" Hash="072D4E33D1478C863DBAB20BF5DFF1A0FB5A9D53" />
    <Deny ID="ID_DENY_D_62_1_1" FriendlyName="PowerShell 62" Hash="631E091AE7AD2C543EE5755BC9D8DB34683C41E20D9A6CD41C8F07827156D6DB" />
    <Deny ID="ID_DENY_D_63_1_1" FriendlyName="PowerShell 63" Hash="FD15A313B890369B7D8E26C13B2070AE044FB4D8" />
    <Deny ID="ID_DENY_D_64_1_1" FriendlyName="PowerShell 64" Hash="AB9886A0993F87C2A39BC7822EE44FD4B4751C530ACF292ACD0319C967FB4F3B" />
    <Deny ID="ID_DENY_D_65_1_1" FriendlyName="PowerShell 65" Hash="4BAFD867B59328E7BB853148FE6D16B9411D7A12" />
    <Deny ID="ID_DENY_D_66_1_1" FriendlyName="PowerShell 66" Hash="D1F22B37902C2DD53FA27438436D9D236A196C10C8E492A8F4A14768644592D3" />
    <Deny ID="ID_DENY_D_67_1_1" FriendlyName="PowerShell 67" Hash="AC53AE4C8AB56D84393D67D820BEBDC3218739D3" />
    <Deny ID="ID_DENY_D_68_1_1" FriendlyName="PowerShell 68" Hash="49580C9459C3917E6F982C8E0D753D293DFA2E4FD1152F78FF7C73CF8B422507" />
    <Deny ID="ID_DENY_D_69_1_1" FriendlyName="PowerShell 69" Hash="333678A44D4BEBE9BEA3041FFDA9E2B55B58F1B5" />
    <Deny ID="ID_DENY_D_70_1_1" FriendlyName="PowerShell 70" Hash="94CBBC3970F01280D98C951BD0C4158D4B09A2BE21B8A27790D9F127B78C6F3F" />
    <Deny ID="ID_DENY_D_71_1_1" FriendlyName="PowerShell 71" Hash="5F5620DC049FE1F1C2DBAC077A59BA69CF2FF72C" />
    <Deny ID="ID_DENY_D_72_1_1" FriendlyName="PowerShell 72" Hash="A32C0769F36CAE0B6A7A1B8CCB6B7A75AA8BEB7F49815E96B4E120BFD7527E0A" />
    <Deny ID="ID_DENY_D_73_1_1" FriendlyName="PowerShell 73" Hash="BDBE541D269EC8235563842D024F9E37883DFB57" />
    <Deny ID="ID_DENY_D_74_1_1" FriendlyName="PowerShell 74" Hash="441076C7FD0AD481E6AC3198F08BE80EA9EB2926CA81D733F798D03DBEFD683E" />
    <Deny ID="ID_DENY_D_75_1_1" FriendlyName="PowerShell 75" Hash="FD6FE9143A46F4EBB46E6B46332FA7171002EBF0" />
    <Deny ID="ID_DENY_D_76_1_1" FriendlyName="PowerShell 76" Hash="85399D84601207AB92C8CA4D7D6E58CB1B0B0B57ED94FA7E5A1191FA1810E223" />
    <Deny ID="ID_DENY_D_77_1_1" FriendlyName="PowerShell 77" Hash="98FD94A89DCF92A7BEDB51C72BAD1A67650DD6E5" />
    <Deny ID="ID_DENY_D_78_1_1" FriendlyName="PowerShell 78" Hash="5CE4B042E986DAFEB7E2D2ABFB80376C4DEC325DB23B584B76039EEA6E1A74B1" />
    <Deny ID="ID_DENY_D_79_1_1" FriendlyName="PowerShell 79" Hash="6BC1E70F0EA84E88AC28BEAF74C10F3ABDF99209" />
    <Deny ID="ID_DENY_D_80_1_1" FriendlyName="PowerShell 80" Hash="93CB3907D1A9473E8A90593250C4A95EAE3A7066E9D8A57535CBDF82AA4AD4C2" />
    <Deny ID="ID_DENY_D_81_1_1" FriendlyName="PowerShell 81" Hash="7FCE82DBBC0FE45AFBE3927C323349C32D5A463A" />
    <Deny ID="ID_DENY_D_82_1_1" FriendlyName="PowerShell 82" Hash="2EDA8CA129E30CB5522C4DCD1E5AFDCA1E9C6447DD7053DACEF18DCDCCF3E2BC" />
    <Deny ID="ID_DENY_D_83_1_1" FriendlyName="PowerShell 83" Hash="BDB3DAC80667A0B931835D5D658C08F236B413D1" />
    <Deny ID="ID_DENY_D_84_1_1" FriendlyName="PowerShell 84" Hash="51287BACB692AAC5A8659774D982B304DC0C0B4A4D8F41CBCCD47D69796786DE" />
    <Deny ID="ID_DENY_D_85_1_1" FriendlyName="PowerShell 85" Hash="9633529CACE25ACCB29EBC5941DE1874903C0297" />
    <Deny ID="ID_DENY_D_86_1_1" FriendlyName="PowerShell 86" Hash="483A3997D5DA69A51DC7EA368A36C3CA4A5BD56CB08BFD9912BE799005156C18" />
    <Deny ID="ID_DENY_D_87_1_1" FriendlyName="PowerShell 87" Hash="B3493E30A2C347B550331C86529BDC288EAF8186" />
    <Deny ID="ID_DENY_D_88_1_1" FriendlyName="PowerShell 88" Hash="9371E2333906441715DE15FEE8A9AA03C4D076CA3C04D9A7AB0CC32189DA66ED" />
    <Deny ID="ID_DENY_D_89_1_1" FriendlyName="PowerShell 89" Hash="5D4B0794EB973D61CF74A700F11BE84E527E0E51" />
    <Deny ID="ID_DENY_D_90_1_1" FriendlyName="PowerShell 90" Hash="537DE34A1F4B3F8345D02F5BBA2B063F070A42FC1581AAC2AA91C1D071B14521" />
    <Deny ID="ID_DENY_D_91_1_1" FriendlyName="PowerShell 91" Hash="F3C75F35F42C1C5B3B4ED888187D6AB4035F994C" />
    <Deny ID="ID_DENY_D_92_1_1" FriendlyName="PowerShell 92" Hash="AD5678ED0734281973465DD728281A6C0EA146620FF2106A4EEFC7E94622B92F" />
    <Deny ID="ID_DENY_D_93_1_1" FriendlyName="PowerShell 93" Hash="91C0F76798A9679188C7D93FDEBAF797BDBE41B2" />
    <Deny ID="ID_DENY_D_94_1_1" FriendlyName="PowerShell 94" Hash="1D9244EAFEDFBFC02E13822E24A476C36FFD362B9D18F6CD195B654A34F946FF" />
    <Deny ID="ID_DENY_D_95_1_1" FriendlyName="PowerShell 95" Hash="7FCB424E67DDAC49413B45D7DCD636AD70E23B41" />
    <Deny ID="ID_DENY_D_96_1_1" FriendlyName="PowerShell 96" Hash="7E6F9A738520F78D1E9D0D0883FB07DD9188408CBE7C2937BDE1590F90C61753" />
    <Deny ID="ID_DENY_D_97_1_1" FriendlyName="PowerShell 97" Hash="A9745E20419EC1C90B23FE965D3C2DF028AF39DC" />
    <Deny ID="ID_DENY_D_98_1_1" FriendlyName="PowerShell 98" Hash="71B5B58EAA0C90397BC9546BCCA8C657500499CD2087CD7D7E1753D54C07E71D" />
    <Deny ID="ID_DENY_D_99_1_1" FriendlyName="PowerShell 99" Hash="3E5294910C59394DA93962128968E6C23016A028" />
    <Deny ID="ID_DENY_D_100_1_1" FriendlyName="PowerShell 100" Hash="DA700D4F58BCEA1D5A9CAD4F20AC725C6A354F9DA40E4F8F95E1C3DC7B84F550" />
    <Deny ID="ID_DENY_D_101_1_1" FriendlyName="PowerShell 101" Hash="266896FD257AD8EE9FC73B3A50306A573714EA8A" />
    <Deny ID="ID_DENY_D_102_1_1" FriendlyName="PowerShell 102" Hash="8E36BD08084C73AF674F2DAD568EE3BA2C85769FA7B3400CB62F7A7BD028BE9A" />
    <Deny ID="ID_DENY_D_103_1_1" FriendlyName="PowerShell 103" Hash="2CB781B3BD79FD277D92332ACA22C04430F9D692" />
    <Deny ID="ID_DENY_D_104_1_1" FriendlyName="PowerShell 104" Hash="92AE03F0090C0A5DF329B4B3FFEDBA622B0521BA699FA303C24120A30ED4C9E6" />
    <Deny ID="ID_DENY_D_105_1_1" FriendlyName="PowerShell 105" Hash="D82583F7D5EA477C94630AC5AAEB771C85BD4B0A" />
    <Deny ID="ID_DENY_D_106_1_1" FriendlyName="PowerShell 106" Hash="9B0F39AB233628A971ACEC53029C9B608CAB99868F1A1C5ABE20BC1BD1C2B70E" />
    <Deny ID="ID_DENY_D_107_1_1" FriendlyName="PowerShell 107" Hash="2DF4350DE3C97C9D4FD2973F8C5EA8AE621D22A8" />
    <Deny ID="ID_DENY_D_108_1_1" FriendlyName="PowerShell 108" Hash="015CE571E8503A353E2250D4D0DA19493B3311F3437527E6DDD2D2B6439FA2EB" />
    <Deny ID="ID_DENY_D_109_1_1" FriendlyName="PowerShell 109" Hash="080DEC3B15AD5AFE9BF3B0943A36285E92BAF469" />
    <Deny ID="ID_DENY_D_110_1_1" FriendlyName="PowerShell 110" Hash="F1391E78F17EA6097906B99C6F4F0AE8DD2E519856F837A3BCC58FBB87DAAE62" />
    <Deny ID="ID_DENY_D_111_1_1" FriendlyName="PowerShell 111" Hash="F87C726CCB5E64C6F363C21255935D5FEA9E4A0E" />
    <Deny ID="ID_DENY_D_112_1_1" FriendlyName="PowerShell 112" Hash="B7B42C3C8C61FD2616C16BBCF36EA15EC26A67536E94764D72A91CE04B89AAA4" />
    <Deny ID="ID_DENY_D_113_1_1" FriendlyName="PowerShell 113" Hash="25F52340199A0EA352C8B1A7014BCB610B232523" />
    <Deny ID="ID_DENY_D_114_1_1" FriendlyName="PowerShell 114" Hash="64D6D1F3A053908C5635BD6BDA36BC8E72D518C7ECE8DA761C0DDE70C50BB632" />
    <Deny ID="ID_DENY_D_115_1_1" FriendlyName="PowerShell 115" Hash="029198F05598109037A0E9E332EC052317E834DA" />
    <Deny ID="ID_DENY_D_116_1_1" FriendlyName="PowerShell 116" Hash="70B4BB6C2B7E9237FB14ABBC94955012285E2CAA74F91455EE52809CDAD4E7FC" />
    <Deny ID="ID_DENY_D_117_1_1" FriendlyName="PowerShell 117" Hash="A4390EF2D77F76DC4EFE55FF74EE1D06C303FDAE" />
    <Deny ID="ID_DENY_D_118_1_1" FriendlyName="PowerShell 118" Hash="3246A0CB329B030DA104E04B1A0728DE83724B08C724FD0238CE4578A0245576" />
    <Deny ID="ID_DENY_D_119_1_1" FriendlyName="PowerShell 119" Hash="89CEAB6518DA4E7F75B3C75BC04A112D3637B737" />
    <Deny ID="ID_DENY_D_120_1_1" FriendlyName="PowerShell 120" Hash="6581E491FBFF954A1A4B9CEA69B63951D67EB56DF871ED8B055193595F042B0D" />
    <Deny ID="ID_DENY_D_121_1_1" FriendlyName="PowerShell 121" Hash="00419E981EDC8613E600C939677F7B460855BF7E" />
    <Deny ID="ID_DENY_D_122_1_1" FriendlyName="PowerShell 122" Hash="61B724BCFC3DA1CC1583DB0BC42EFE166E92D8D3CE91E58A29F7AEBEFAE2149F" />
    <Deny ID="ID_DENY_D_123_1_1" FriendlyName="PowerShell 123" Hash="272EF88BBA9B4B54D242FFE1E96D07DBF53497A0" />
    <Deny ID="ID_DENY_D_124_1_1" FriendlyName="PowerShell 124" Hash="AFC0968EDCE9E5FC1BC392382833EBEF3265B32D3ECBB529D89A1DF33A31E9BD" />
    <Deny ID="ID_DENY_D_125_1_1" FriendlyName="PowerShell 125" Hash="CD9D9789B3B31562C4BE44B6BEEA8815C5EDAE1F" />
    <Deny ID="ID_DENY_D_126_1_1" FriendlyName="PowerShell 126" Hash="FCAF8DC3C7A5D3B29B19A9C5F89324BF65B50C440AC0316B08532CEA2F1FF9B0" />
    <Deny ID="ID_DENY_D_127_1_1" FriendlyName="PowerShell 127" Hash="941D0FD47887035A04E17F46DE6C4004D7FD8871" />
    <Deny ID="ID_DENY_D_128_1_1" FriendlyName="PowerShell 128" Hash="4AD6DC7FF0A2E776CE7F27B4E3D3C1C380CA3548DFED565429D88C3BBE61DD0F" />
    <Deny ID="ID_DENY_D_129_1_1" FriendlyName="PowerShell 129" Hash="421D1142105358B8360454E43FD15767DA111DBA" />
    <Deny ID="ID_DENY_D_130_1_1" FriendlyName="PowerShell 130" Hash="692CABD40C1EDFCB6DC50591F31FAE30848E579D6EF4D2CA0811D06B086CF8BE" />
    <Deny ID="ID_DENY_D_131_1_1" FriendlyName="PowerShell 131" Hash="AC9F095DD4AE80B124F55541761AA1F35E49A575" />
    <Deny ID="ID_DENY_D_132_1_1" FriendlyName="PowerShell 132" Hash="0D8A0FB3BF3CF80D44ED20D9F1E7292E9EE5A49ABCE68592DED55A71B0ACAECE" />
    <Deny ID="ID_DENY_D_133_1_1" FriendlyName="PowerShell 133" Hash="B1CF2A18B281F73FE6685B5CE74D1BA50BE9AFE5" />
    <Deny ID="ID_DENY_D_134_1_1" FriendlyName="PowerShell 134" Hash="095B79953F9E3E2FB721693FBFAD5841112D592B6CA7EB2055B262DEB7C7008A" />
    <Deny ID="ID_DENY_D_135_1_1" FriendlyName="PowerShell 135" Hash="128D7D03E4B85DBF95427D72EFF833DAB5E92C33" />
    <Deny ID="ID_DENY_D_136_1_1" FriendlyName="PowerShell 136" Hash="EACFC615FDE29BD858088AF42E0917E4B4CA5991EFB4394FB3129735D7299235" />
    <Deny ID="ID_DENY_D_137_1_1" FriendlyName="PowerShell 137" Hash="47D2F87F2D2D516D712A156421F0C2BD285200E9" />
    <Deny ID="ID_DENY_D_138_1_1" FriendlyName="PowerShell 138" Hash="8CACA1828E7770DADF21D558976D415AC7BDA16D58926308FD5E9D5087F4B0E6" />
    <Deny ID="ID_DENY_D_139_1_1" FriendlyName="PowerShell 139" Hash="CD9D70B0107801567EEADC4ECD74511A1A6FF4FE" />
    <Deny ID="ID_DENY_D_140_1_1" FriendlyName="PowerShell 140" Hash="9C96396EFCC9DC09F119DE8695CB3372F82DB46D23A1B7A88BD86CBE814233E1" />
    <Deny ID="ID_DENY_D_141_1_1" FriendlyName="PowerShell 141" Hash="233E3B5108A43239C6C13292043DED0567281AF9" />
    <Deny ID="ID_DENY_D_142_1_1" FriendlyName="PowerShell 142" Hash="6EDF19CC53EA2064CE108957343EB3505359CF05BD6955C7502AF565BD761702" />
    <Deny ID="ID_DENY_D_143_1_1" FriendlyName="PowerShell 143" Hash="CD725B606888E5C5426FEAB44E2CC7722DFE5411" />
    <Deny ID="ID_DENY_D_144_1_1" FriendlyName="PowerShell 144" Hash="B20C4F36AE6A3AC323759C81173FACE1B1C112FA5B701C65DCD7313D7CE59907" />
    <Deny ID="ID_DENY_D_145_1_1" FriendlyName="PowerShell 145" Hash="E5212F1081B5777B88F5C41174ADEDB35B4258CF" />
    <Deny ID="ID_DENY_D_146_1_1" FriendlyName="PowerShell 146" Hash="F4DE5B5395701F8C94D65D732E4D212E1879C9C84345B46A941965B094F75017" />
    <Deny ID="ID_DENY_D_147_1_1" FriendlyName="PowerShell 147" Hash="EC41A3FB8D6E3B0F55F6583C14C45B6238753019" />
    <Deny ID="ID_DENY_D_148_1_1" FriendlyName="PowerShell 148" Hash="76CA6B396796351685198D6189E865AFD7FB9E6C5CEFA9EA0B5F0A9F1FC98D57" />
    <Deny ID="ID_DENY_D_149_1_1" FriendlyName="PowerShell 149" Hash="3B2B7042A84033CA846AFE472912524F7BAD57E5" />
    <Deny ID="ID_DENY_D_150_1_1" FriendlyName="PowerShell 150" Hash="2DF95ABEB23DAA0377DFA6360976B69D3CEE7325A9B7571F331D569809FAED8B" />
    <Deny ID="ID_DENY_D_151_1_1" FriendlyName="PowerShell 151" Hash="7BED2F9C0ADF1597C7EBB79163BDA21D8D7D28CA" />
    <Deny ID="ID_DENY_D_152_1_1" FriendlyName="PowerShell 152" Hash="44BDD2DADB13E7A8FF6AFCF4AE3E2CC830506D9475B4C2C71D319E169977998F" />
    <Deny ID="ID_DENY_D_153_1_1" FriendlyName="PowerShell 153" Hash="A1251FA30162B13456A4687495726FF793D511BE" />
    <Deny ID="ID_DENY_D_154_1_1" FriendlyName="PowerShell 154" Hash="9C15E4DE10DE47ACD393359D523211AD8596C61FE54F2C0664D48E1D249231CE" />
    <Deny ID="ID_DENY_D_155_1_1" FriendlyName="PowerShell 155" Hash="D835947C84CFBA652B553A77A90475E02291AA5F" />
    <Deny ID="ID_DENY_D_156_1_1" FriendlyName="PowerShell 156" Hash="B4D6DAA10398D5DA192DFDD75010F428D24762D432934F0E2030D39610D43E12" />
    <Deny ID="ID_DENY_D_157_1_1" FriendlyName="PowerShell 157" Hash="1F85BBEC1DFC5785B91735A7C561E664F7FE1E94" />
    <Deny ID="ID_DENY_D_158_1_1" FriendlyName="PowerShell 158" Hash="828F05BFF829019EC0F3082323FEA859C0D71CCE14B5B75C07E7D418EF354269" />
    <Deny ID="ID_DENY_D_159_1_1" FriendlyName="PowerShell 159" Hash="FC0E23771620B41E6920F2463F49B84307D8BA91" />
    <Deny ID="ID_DENY_D_160_1_1" FriendlyName="PowerShell 160" Hash="C4FA568C852A46316308A660B80D83A11D41071F1CF4A79847A3F56714CC47AF" />
    <Deny ID="ID_DENY_D_161_1_1" FriendlyName="PowerShell 161" Hash="D18240AEE8B9B964F6B9CDFC5AFB6C343C286636" />
    <Deny ID="ID_DENY_D_162_1_1" FriendlyName="PowerShell 162" Hash="7B4C39285569F14AA9799332C542A0796717C5EF9D636BD11B2841450BC6399D" />
    <Deny ID="ID_DENY_D_163_1_1" FriendlyName="PowerShell 163" Hash="1A16008D330330182AA555B1D3E9BE0B2D6BECBF" />
    <Deny ID="ID_DENY_D_164_1_1" FriendlyName="PowerShell 164" Hash="D7685E259D0328937487856A3AB68B6D9D420DD4E02541F4D71164DFA65B4644" />
    <Deny ID="ID_DENY_D_165_1_1" FriendlyName="PowerShell 165" Hash="FBA274406B503B464B349805149E6AA722909CC9" />
    <Deny ID="ID_DENY_D_166_1_1" FriendlyName="PowerShell 166" Hash="FEBC97ED819C79E54157895457DBA755F182D6330A5103E0663AFA07E01E5CF8" />
    <Deny ID="ID_DENY_D_167_1_1" FriendlyName="PowerShell 167" Hash="293AF426A39282770387F5EE25CA719A91419A18" />
    <Deny ID="ID_DENY_D_168_1_1" FriendlyName="PowerShell 168" Hash="A9E655A96A124BC361D9CC5C7663FC033AA6F6609916EFAA76B6A6E9713A0D32" />
    <Deny ID="ID_DENY_D_169_1_1" FriendlyName="PowerShell 169" Hash="AEBFE7497F4A1947B5CB32650843CA0F85BD56D0" />
    <Deny ID="ID_DENY_D_170_1_1" FriendlyName="PowerShell 170" Hash="8C385B2C16136C097C96701D2140E014BF454CFA7297BE0C28431DED15339C0F" />
    <Deny ID="ID_DENY_D_171_1_1" FriendlyName="PowerShell 171" Hash="8FB604CD72701B83BC265D87F52B36C6F14E5DBE" />
    <Deny ID="ID_DENY_D_172_1_1" FriendlyName="PowerShell 172" Hash="B35AFBA7A897CB882C14A08AFB36A8EC938BDA14DF070234A2CCBDBA8F7DF91C" />
    <Deny ID="ID_DENY_D_173_1_1" FriendlyName="PowerShell 173" Hash="CE70309DB83C9202F45028EBEC252747F4936E6F" />
    <Deny ID="ID_DENY_D_174_1_1" FriendlyName="PowerShell 174" Hash="1F6D74FDA1F9EE6BBAC72E7E717A01B9FFC29822561D11175F6809D12215B4ED" />
    <Deny ID="ID_DENY_D_175_1_1" FriendlyName="PowerShell 175" Hash="9D71AD914DBB2FDF793742AA63AEEF4E4A430790" />
    <Deny ID="ID_DENY_D_176_1_1" FriendlyName="PowerShell 176" Hash="8CC1B5FA9A9609AC811F6505FA9B68E85A87BAE1EF676EFFE1BE438EACBDF3E1" />
    <Deny ID="ID_DENY_D_177_1_1" FriendlyName="PowerShell 177" Hash="7484FD78A9298DBA24AC5C882D16DB6146E53712" />
    <Deny ID="ID_DENY_D_178_1_1" FriendlyName="PowerShell 178" Hash="A79A74BFB768312E8EE089060C5C3238D59EF0C044A450FEB97DCA26815ECB34" />
    <Deny ID="ID_DENY_D_179_1_1" FriendlyName="PowerShell 179" Hash="78C3C6AEF52A6A5392C55F1EC98AF18053B3087D" />
    <Deny ID="ID_DENY_D_180_1_1" FriendlyName="PowerShell 180" Hash="493B620FCAD8A91D1FD7C726697E09358CA90822E8D6E021DF56E70B46F7C346" />
    <Deny ID="ID_DENY_D_181_1_1" FriendlyName="PowerShell 181" Hash="783FFB771F08BCF55C2EA474B5460EB65EA9444C" />
    <Deny ID="ID_DENY_D_182_1_1" FriendlyName="PowerShell 182" Hash="09DA1592B8457F860297821EB7FAA7F3BB71FC1916ED5DEE6D85044953640D5C" />
    <Deny ID="ID_DENY_D_183_1_1" FriendlyName="PowerShell 183" Hash="B303D1689ED99613E4F52CE6E5F96AAEBC3A45C3" />
    <Deny ID="ID_DENY_D_184_1_1" FriendlyName="PowerShell 184" Hash="82AB406FD78DCF58F65DC14D6FDDD72840015F3FE5B554428969BECA0325CD9C" />
    <Deny ID="ID_DENY_D_185_1_1" FriendlyName="PowerShell 185" Hash="DB5C6CB23C23BA6A3CD4FD4EC0A4DAEE3FC66500" />
    <Deny ID="ID_DENY_D_186_1_1" FriendlyName="PowerShell 186" Hash="9A46C16C5151D97A0EFA3EA503249E31A6D5D8D25E4F07CD4E5E077A574713FB" />
    <Deny ID="ID_DENY_D_187_1_1" FriendlyName="PowerShell 187" Hash="C1E08AD32F680100C51F138C6C095139E7230C3B" />
    <Deny ID="ID_DENY_D_188_1_1" FriendlyName="PowerShell 188" Hash="A5D5C1F79CD26216194D4C72DBAA3E48CB4A143D9E1F78819E52E9FEB2AD0AE3" />
    <Deny ID="ID_DENY_D_189_1_1" FriendlyName="PowerShell 189" Hash="BACA825D0852E2D8F3D92381D112B99B5DD56D9F" />
    <Deny ID="ID_DENY_D_190_1_1" FriendlyName="PowerShell 190" Hash="ABA28E0FC251E1D7FE5E264E1B36EC5E482D70AA434E75A756356F23F0C1F2F4" />
    <Deny ID="ID_DENY_D_191_1_1" FriendlyName="PowerShell 191" Hash="E89C29D38F554F6CB73B5FD3D0A783CC12FFEBC3" />
    <Deny ID="ID_DENY_D_192_1_1" FriendlyName="PowerShell 192" Hash="4C93CBDCF4328D27681453D8DFD7495955A07EE6A0EFB9A593853A86990CF528" />
    <Deny ID="ID_DENY_D_193_1_1" FriendlyName="PowerShell 193" Hash="5B5E7942233D7C8A325A429FC4F4AE281325E8F9" />
    <Deny ID="ID_DENY_D_194_1_1" FriendlyName="PowerShell 194" Hash="40DA20086ED76A5EA5F62901D110216EE206E7EEB2F2BFF02F61D0BE85B0BB5A" />
    <Deny ID="ID_DENY_D_195_1_1" FriendlyName="PowerShell 195" Hash="926DCACC6983F85A8ABBCB5EE13F3C756705A1D5" />
    <Deny ID="ID_DENY_D_196_1_1" FriendlyName="PowerShell 196" Hash="A22761E2BF18F02BB630962E3C5E32738770AAEA77F8EDA233E77792EB480072" />
    <Deny ID="ID_DENY_D_197_1_1" FriendlyName="PowerShell 197" Hash="6FE6723A355DEB4BC6B8637A634D1B43AFA64112" />
    <Deny ID="ID_DENY_D_198_1_1" FriendlyName="PowerShell 198" Hash="9BCC55A97A275F7D81110877F1BB5B41F86A848EA02B4EE1E1E6A44D927A488F" />
    <Deny ID="ID_DENY_D_199_1_1" FriendlyName="PowerShell 199" Hash="8D5599B34BED4A660DACC0922F6C2F112F264758" />
    <Deny ID="ID_DENY_D_200_1_1" FriendlyName="PowerShell 200" Hash="F375014915E5E027F697B29201362B56F2D9E598247C96F86ABADCC6FF42F034" />
    <Deny ID="ID_DENY_D_201_1_1" FriendlyName="PowerShell 201" Hash="CCFB247A3BCA9C64D82F647F3D30A3172E645F13" />
    <Deny ID="ID_DENY_D_202_1_1" FriendlyName="PowerShell 202" Hash="5E52ABBC051368315F078D31F01B0C1B904C1DDB6D1C1E4A91BE276BDF44C66F" />
    <Deny ID="ID_DENY_D_203_1_1" FriendlyName="PowerShell 203" Hash="E8EB859531F426CC45A3CB9118F399C92054563E" />
    <Deny ID="ID_DENY_D_204_1_1" FriendlyName="PowerShell 204" Hash="CD9E1D41F8D982F4AA6C610A2EFEAEBA5B0CDD883DF4A86FA0180ACD333CAA86" />
    <Deny ID="ID_DENY_D_205_1_1" FriendlyName="PowerShell 205" Hash="C92D4EAC917EE4842A437C54F96D87F003199DE8" />
    <Deny ID="ID_DENY_D_206_1_1" FriendlyName="PowerShell 206" Hash="3A270242EB49E06405FD654FA4954B166297BBC886891C64B4424134C39872DB" />
    <Deny ID="ID_DENY_D_207_1_1" FriendlyName="PowerShell 207" Hash="66681D9171981216B31996429695931DA2A638B9" />
    <Deny ID="ID_DENY_D_208_1_1" FriendlyName="PowerShell 208" Hash="7A2DF7D56912CB4EB5B36D071496EDC97661086B0E4C9CC5D9C61779A5A7DAAA" />
    <Deny ID="ID_DENY_D_209_1_1" FriendlyName="PowerShell 209" Hash="9DCA54C85E4C645CB296FE3055E90255B6506A95" />
    <Deny ID="ID_DENY_D_210_1_1" FriendlyName="PowerShell 210" Hash="8C9C58AD12FE61CBF021634EC6A4B3094750FC002DA224423E0BCEB01ECF292A" />
    <Deny ID="ID_DENY_D_211_1_1" FriendlyName="PowerShell 211" Hash="3AF2587E8B62F88DC363D7F5308EE4C1A6147338" />
    <Deny ID="ID_DENY_D_212_1_1" FriendlyName="PowerShell 212" Hash="D32D88F158FD341E32708CCADD48C426D227D0EC8465FF4304C7B7EAC2C6A93E" />
    <Deny ID="ID_DENY_D_213_1_1" FriendlyName="PowerShell 213" Hash="D3D453EBC368DF7CC2200474035E5898B58D93F1" />
    <Deny ID="ID_DENY_D_214_1_1" FriendlyName="PowerShell 214" Hash="BBE569BCC282B3AF682C1528D4E3BC53C1A0C6B5905FA34ADB4305160967B64A" />
    <Deny ID="ID_DENY_D_215_1_1" FriendlyName="PowerShell 215" Hash="D147CE5C7E7037D1BE3C0AF67EDB6F528C77DB0A" />
    <Deny ID="ID_DENY_D_216_1_1" FriendlyName="PowerShell 216" Hash="11F936112832738AD9B3A1C67537D5542DE8E86856CF2A5893C4D26CF3A2C558" />
    <Deny ID="ID_DENY_D_217_1_1" FriendlyName="PowerShell 217" Hash="7DBB41B87FAA887DE456C8E6A72E09D2839FA1E7" />
    <Deny ID="ID_DENY_D_218_1_1" FriendlyName="PowerShell 218" Hash="3741F3D2F264E047339C95A66085599A49766DEF1C5BD0C32237CE87FA0B41FB" />
    <Deny ID="ID_DENY_D_219_1_1" FriendlyName="PowerShell 219" Hash="5F3AECC89BAF094EAFA3C25E6B883EE68A6F00B0" />
    <Deny ID="ID_DENY_D_220_1_1" FriendlyName="PowerShell 220" Hash="AA085BE6498D2E3F527F3D72A5D1C604508133F0CDC05AD404BB49E8E3FB1A1B" />
    <Deny ID="ID_DENY_D_221_1_1" FriendlyName="PowerShell 221" Hash="DDE4D9A08514347CDE706C42920F43523FC74DEA" />
    <Deny ID="ID_DENY_D_222_1_1" FriendlyName="PowerShell 222" Hash="81835C6294B96282A4D7D70383BBF797C2E4E7CEF99648F85DDA50F7F41B02F6" />
    <Deny ID="ID_DENY_D_223_1_1" FriendlyName="PowerShell 223" Hash="48092864C96C4BF9B68B5006EAEDAB8B57B3738C" />
    <Deny ID="ID_DENY_D_224_1_1" FriendlyName="PowerShell 224" Hash="36EF3BED9A5D0D563BCB354BFDD2931F6256759D1D905BA5DC21CDA496F2FEB7" />
    <Deny ID="ID_DENY_D_225_1_1" FriendlyName="PowerShell 225" Hash="7F6725BA8CCD2DAEEFD0C9590A5DF9D98642CCEA" />
    <Deny ID="ID_DENY_D_226_1_1" FriendlyName="PowerShell 226" Hash="DB68DB3AE32A8A662AA6EE16CF459124D2701719D019B614CE9BF115F5F9C904" />
    <Deny ID="ID_DENY_D_227_1_1" FriendlyName="PowerShell 227" Hash="FF205856A3209227D571EAD4B8C1E611E7FF9924" />
    <Deny ID="ID_DENY_D_228_1_1" FriendlyName="PowerShell 228" Hash="A63B38CE17DA60C4C431FC42C4507A0B7C19B384AC9E121E2988AD026E71ED63" />
    <Deny ID="ID_DENY_D_229_1_1" FriendlyName="PowerShell 229" Hash="479C9429691314D3E21E4F4CA8B95D5BD2BDDEDA" />
    <Deny ID="ID_DENY_D_230_1_1" FriendlyName="PowerShell 230" Hash="2BA4E369D267A9ABDEBA50DA2CB5FC56A8EE4382C5BCFCFFD121350B88A6F0E1" />
    <Deny ID="ID_DENY_D_231_1_1" FriendlyName="PowerShell 231" Hash="C7D70B96440D215173F35412D56CF9329886D8D3" />
    <Deny ID="ID_DENY_D_232_1_1" FriendlyName="PowerShell 232" Hash="B00C54F1AA77D88335675EAF07ED834E68FD96DD7606914C2867F9C506AB0A56" />
    <Deny ID="ID_DENY_D_233_1_1" FriendlyName="PowerShell 233" Hash="2AB804E1FF982AE0EDB591BC61AA909CF32E99C5" />
    <Deny ID="ID_DENY_D_234_1_1" FriendlyName="PowerShell 234" Hash="253120422B0DD987C293CAF5928FA820414C0A01622FD0EAF304A750FC5AEEFE" />
    <Deny ID="ID_DENY_D_235_1_1" FriendlyName="PowerShell 235" Hash="8DAB1D74CAEDBAA8D17805CF00D64A44F5831C12" />
    <Deny ID="ID_DENY_D_236_1_1" FriendlyName="PowerShell 236" Hash="AC1CE3AA9023E23F2F63D5A3536294B914686057336402E059DEF6559D1CE723" />
    <Deny ID="ID_DENY_D_237_1_1" FriendlyName="PowerShell 237" Hash="993425279D204D1D14C3EB989DEB4805ADC558CF" />
    <Deny ID="ID_DENY_D_238_1_1" FriendlyName="PowerShell 238" Hash="BDADDD710E47EB8D24B78E542F3996B0EA2CA577ABD515785819302DB15839DD" />
    <Deny ID="ID_DENY_D_239_1_1" FriendlyName="PowerShell 239" Hash="F4DB0CDF3A3FD163A9B90789CC6D14D326AD609C" />
    <Deny ID="ID_DENY_D_240_1_1" FriendlyName="PowerShell 240" Hash="5D249D8366077713024552CA8D08F164E975AFF89E8909E35A43F02B0DC66F70" />
    <Deny ID="ID_DENY_D_241_1_1" FriendlyName="PowerShell 241" Hash="5B8E45EECA32C2F0968C2252229D768B0DB796A0" />
    <Deny ID="ID_DENY_D_242_1_1" FriendlyName="PowerShell 242" Hash="B4D336B32C27E3D3FEBE4B06252DDE9683814E7E903C98448972AAB7389DFC02" />
    <Deny ID="ID_DENY_D_243_1_1" FriendlyName="PowerShell 243" Hash="4F5D66B449C4D2FDEA532F9B5DBECA5ACA8195EF" />
    <Deny ID="ID_DENY_D_244_1_1" FriendlyName="PowerShell 244" Hash="39F2F19A5C6708CE8CE4E1ABBEBA8D3D1A6220391CA86B2D319E347B46005C97" />
    <Deny ID="ID_DENY_D_245_1_1" FriendlyName="PowerShell 245" Hash="4BFB3F95CA1B79DA3C6B0A2ECB432059E686F967" />
    <Deny ID="ID_DENY_D_246_1_1" FriendlyName="PowerShell 246" Hash="0C4688AACD02829850DE0F792AC06D3C87895412A910EA76F7F9BF31B3B4A3E9" />
    <Deny ID="ID_DENY_D_247_1_1" FriendlyName="PowerShell 247" Hash="6DC048AFA50B5B1B0AD7DD3125AC83D46FED730A" />
    <Deny ID="ID_DENY_D_248_1_1" FriendlyName="PowerShell 248" Hash="432F666CCE8CD222484E263AE02F63E0038143DD6AD07B3EB1633CD3C498C13D" />
    <Deny ID="ID_DENY_D_249_1_1" FriendlyName="PubPrn 249" Hash="68E96BE23748AA680D5E1E557778901F332ED5D3" />
    <Deny ID="ID_DENY_D_250_1_1" FriendlyName="PubPrn 250" Hash="8FA30B5931806565C2058E565C06AD5F1C5A48CDBE609975EB31207C25214063" />
    <Deny ID="ID_DENY_D_251_1_1" FriendlyName="PubPrn 251" Hash="32C4B29FE428B1DF473F3F4FECF519D285E93521" />
    <Deny ID="ID_DENY_D_252_1_1" FriendlyName="PubPrn 252" Hash="D44FB563198D60DFDC91608949FE2FADAD6161854D084EB1968C558AA36513C7" />
    <Deny ID="ID_DENY_D_253_1_1" FriendlyName="PubPrn 253" Hash="9EDBEF086D350863F29175F5AB5178B88B142C75" />
    <Deny ID="ID_DENY_D_254_1_1" FriendlyName="PubPrn 254" Hash="9B22C98351F2B6DEDDCED0D805C65F5B166FF519A8DF41EB242CB909471892EB" />
    <Deny ID="ID_DENY_D_255_1_1" FriendlyName="PubPrn 255" Hash="8A3B30F345C43246B3500721CFEEADBAC6B9D9C6" />
    <Deny ID="ID_DENY_D_256_1_1" FriendlyName="PubPrn 256" Hash="37C20BF20A2BBACE50957F8D0AB3FD16174BC005E79D47E51E899AFD9E4B7724" />
    <Deny ID="ID_DENY_D_257_1_1" FriendlyName="PubPrn 257" Hash="C659DAD2B37375781E2D584E16AAE2A10B5A1156" />
    <Deny ID="ID_DENY_D_258_1_1" FriendlyName="PubPRn 258" Hash="EBDACA86F10AC0446D60CC75628EC7A370B1E2236E6D20F22372F91033B6D429" />
    <Deny ID="ID_DENY_D_259_1_1" FriendlyName="PubPrn 259" Hash="C9D6394BBFF8CD9C6590F08C54EC6AFDEB5CFFB4" />
    <Deny ID="ID_DENY_D_260_1_1" FriendlyName="PubPrn 260" Hash="518E4EA7A2B70713E1AEC6E7E75A488C39384B625C5F2779073E9294CBF2BD9F" />
    <Deny ID="ID_DENY_D_263_1_1" FriendlyName="PubPrn 263" Hash="763A652217A1E30F2D288B7F44E08346949A02CD" />
    <Deny ID="ID_DENY_D_264_1_1" FriendlyName="PubPrn 264" Hash="FCDDA212B06602F642B29FC05316EF75E4EE9975E6E8A9526E842BE2EA237C5D" />
    <Deny ID="ID_DENY_D_267_1_1" FriendlyName="PubPrn 267" Hash="60FD28D770B23A0477679311D247DA4D5C61074C" />
    <Deny ID="ID_DENY_D_268_1_1" FriendlyName="PubPrn 268" Hash="D09A4B2EA611CDFDC6DCA44314289B622B2A5EDA09716EF4A16B91EC90BFBA8F" />
    <Deny ID="ID_DENY_D_271_1_1" FriendlyName="PubPrn 271" Hash="47CBE201ED224BF3F5C322F7A49EF64469AF2E1A" />
    <Deny ID="ID_DENY_D_272_1_1" FriendlyName="PubPrn 272" Hash="24855B9CC420719D5AB93F4F1589CE09E4063E4FC98681BD91A1D18A3C8ACB43" />
    <Deny ID="ID_DENY_D_275_1_1" FriendlyName="PubPrn 275" Hash="663D8E25BAE20510A882F6692BE2620FBABFB94E" />
    <Deny ID="ID_DENY_D_276_1_1" FriendlyName="PubPrn 276" Hash="649A9E5A4867A28C7D0934793F33B545F9441EA23872715C84826D80CC8EC576" />
    <Deny ID="ID_DENY_D_277_1_1" FriendlyName="PubPrn 277" Hash="226ABB2FBAEFC5A7E2A819D9D708F826C00FD215" />
    <Deny ID="ID_DENY_D_278_1_1" FriendlyName="PubPrn 278" Hash="AC6B35C904D388FD12C07C2F6A1A07F337D31895713BF01DCCE7A7F187D7F4D9" />
    <Deny ID="ID_DENY_D_279_1_1" FriendlyName="PubPrn 279" Hash="071D7849941E43144839988971255FE34690A747" />
    <Deny ID="ID_DENY_D_280_1_1" FriendlyName="PubPrn 280" Hash="5AF75895BDC11A6B68C816A8677D7CF9692BF25A95C4378A43FBDE740B18EEB1" />
    <Deny ID="ID_DENY_D_281_1_1" FriendlyName="PubPrn 281" Hash="9FBFF074C201BFEBE37710CB453EFF9A14AE3BFF" />
    <Deny ID="ID_DENY_D_282_1_1" FriendlyName="PubPrn 282" Hash="A0C71A925850D2D481C7E520F5D5A83305EC169EEA4C5B8DC20C8D8AFCD8A512" />
    <Deny ID="ID_DENY_D_283_1_1" FriendlyName="PSWorkflowUtility 283" Hash="4FBC9A72C5D5246F34994F13076A5AD98A1A844E" />
    <Deny ID="ID_DENY_D_284_1_1" FriendlyName="PSWorkflowUtility 284" Hash="7BF44433D3A606104778F64B11B92C52FC99C4BA570C50B70438275D0B587B8E" />
    <Deny ID="ID_DENY_D_285_1_1" FriendlyName="PSWorkflowUtility 285" Hash="99382ED8FA3577DFD903C01478A79D6D90681406" />
    <Deny ID="ID_DENY_D_286_1_1" FriendlyName="PSWorkflowUtility 286" Hash="C3A5DAB20947CA8FD092E75C25177E7BAE7884CA58710F14827144C09EA1F94B" />
    <Deny ID="ID_DENY_D_287_1_1" FriendlyName="PowerShellShell 287" Hash="2B45C165F5E0BFD932397B18980BA680E2E82BD1" />
    <Deny ID="ID_DENY_D_288_1_1" FriendlyName="PowerShellShell 288" Hash="1DD0AD6B85DAEBAE7555DC37EA6C160EA38F75E3D4847176F77562A59025660A" />
    <Deny ID="ID_DENY_D_289_1_1" FriendlyName="PowerShellShell 289" Hash="A8C9E28F25C9C5F479691F2F49339F4448747638" />
    <Deny ID="ID_DENY_D_290_1_1" FriendlyName="PowerShellShell 290" Hash="F8FA17038CD532BF5D0D6D3AC55CE34E45EB690637D38D399CAB14B09807EB6C" />
    <Deny ID="ID_DENY_D_293_1_1" FriendlyName="PowerShellShell 293" Hash="3BA0605C08935B340BEFDC83C0D92B1CE52B8348" />
    <Deny ID="ID_DENY_D_294_1_1" FriendlyName="PowerShellShell 294" Hash="B794B01CE561F2791D4ED3EADE523D03D2BE7B4CEFE9AAFC685ECE8ACF515ED2" />
    <Deny ID="ID_DENY_D_295_1_1" FriendlyName="PowerShellShell 295" Hash="8B74A22710A532A71532E4F0B1C60AABDCAA29AB" />
    <Deny ID="ID_DENY_D_296_1_1" FriendlyName="PowerShellShell 296" Hash="EB335007DF9897BCD2ED5C647BA724F07658E8597E73E353479201000CF2EF79" />
    <Deny ID="ID_DENY_D_297_1_1" FriendlyName="PowerShellShell 297" Hash="10E2CD3A2CFA0549590F740139F464626DEE2092" />
    <Deny ID="ID_DENY_D_298_1_1" FriendlyName="PowerShellShell 298" Hash="61DEC96B91F3F152DFDA84B28EBB184808A21C4C183CC0584C66AC7E20F0DDB6" />
    <Deny ID="ID_DENY_D_299_1_1" FriendlyName="PowerShellShell 299" Hash="98E84F46B3EB3AD7420C9715722145AFB0C065A7" />
    <Deny ID="ID_DENY_D_300_1_1" FriendlyName="PowerShellShell 300" Hash="67398990D42DFF84F8BE33B486BF492EBAF61671820BB9DCF039D1F8738EC5A4" />
    <Deny ID="ID_DENY_D_301_1_1" FriendlyName="PowerShellShell 301" Hash="58F399EC75708720E722FBD038F0EC089BF5A8C0" />
    <Deny ID="ID_DENY_D_302_1_1" FriendlyName="PowerShellShell 302" Hash="C523FFF884C44251337470870E0B158230961845FC1E953F877D515668524F2E" />
    <Deny ID="ID_DENY_D_303_1_1" FriendlyName="PowerShellShell 303" Hash="41EE8E9559FC0E772FC26EBA87ED4D77E60DC76C" />
    <Deny ID="ID_DENY_D_304_1_1" FriendlyName="PowerShellShell 304" Hash="219AD97976987C614B00C0CD1229B4245F2F1453F5AF90B907664D0BF6ADFE78" />
    <Deny ID="ID_DENY_D_305_1_1" FriendlyName="PowerShellShell 305" Hash="7F7E646892FCEB8D6A19647F00C1153014955C45" />
    <Deny ID="ID_DENY_D_306_1_1" FriendlyName="PowerShellShell 306" Hash="5825FF16398F12B4999B9A12849A757DD0884F9908220FB33E720F170DA288D5" />
    <Deny ID="ID_DENY_D_307_1_1" FriendlyName="PowerShellShell 307" Hash="7EA8A590583008446583F0AE7D66537FAD63619D" />
    <Deny ID="ID_DENY_D_308_1_1" FriendlyName="PowerShellShell 308" Hash="26DD094717B15B3D39600D909A9CAEBCF5C616C6277933BCC01326E8C475A128" />
    <Deny ID="ID_DENY_D_309_1_1" FriendlyName="PowerShellShell 309" Hash="5F6CDF52C1E184B080B89EB234DE179C19F110BA" />
    <Deny ID="ID_DENY_D_310_1_1" FriendlyName="PowerShellShell 310" Hash="41FB90606E3C66D21C703D84C943F8CB35772030B689D9A9895CB3EF7C863FB2" />
    <Deny ID="ID_DENY_D_311_1_1" FriendlyName="PowerShellShell 311" Hash="91C1DACBD6773BFC7F9305418A6683B8311949CF" />
    <Deny ID="ID_DENY_D_312_1_1" FriendlyName="PowerShellShell 312" Hash="EB678387D01938D88E6F2F46712269D54D845EB6A8AAC3FCA256DC2160D42975" />
    <Deny ID="ID_DENY_D_313_1_1" FriendlyName="PowerShellShell 313" Hash="A05294D23A4A7DC91692013C0EC4373598A28B21" />
    <Deny ID="ID_DENY_D_314_1_1" FriendlyName="PowerShellShell 314" Hash="ABEEA4903403D2C07489436E59955ECFEEF893C63D1FDBED234343F6A6D472B1" />
    <Deny ID="ID_DENY_D_315_1_1" FriendlyName="PowerShellShell 315" Hash="B155C278617845EC6318E4009E4CED6639FAB951" />
    <Deny ID="ID_DENY_D_316_1_1" FriendlyName="PowerShellShell 316" Hash="59549FEEB4D64BA3AF50F925FECC8107422D3F54AF6106E5B0152B2F50912980" />
    <Deny ID="ID_DENY_D_317_1_1" FriendlyName="PowerShellShell 317" Hash="465D848F11CECE4452E831D248D326360B73A319" />
    <Deny ID="ID_DENY_D_318_1_1" FriendlyName="PowerShellShell 318" Hash="B9C9F208C6E50AABF91D234227D09D7C6CAB2FDB229163103E7C1F541F71C213" />
    <Deny ID="ID_DENY_D_319_1_1" FriendlyName="PowerShellShell 319" Hash="F0B9D75B53A268C0AC30584738C3A5EC33420A2E" />
    <Deny ID="ID_DENY_D_320_1_1" FriendlyName="PowerShellShell 320" Hash="365A7812DFC448B1FE9CEA83CF55BC62189C4E72BAD84276BD5F1DAB47CB3EFF" />
    <Deny ID="ID_DENY_D_321_1_1" FriendlyName="PowerShellShell 321" Hash="8ADCDD18EB178B6A43CF5E11EC73212C90B91988" />
    <Deny ID="ID_DENY_D_322_1_1" FriendlyName="PowerShellShell 322" Hash="51BD119BE2FBEFEC560F618DBBBB8203A251F455B1DF825F37B1DFFDBE120DF2" />
    <Deny ID="ID_DENY_D_323_1_1" FriendlyName="PowerShellShell 323" Hash="D2011097B6038D8507B26B7618FF07DA0FF01234" />
    <Deny ID="ID_DENY_D_324_1_1" FriendlyName="PowerShellShell 324" Hash="BA3D20A577F355612E53428D573767C48A091AE965FCB30CC348619F1CB85A02" />
    <Deny ID="ID_DENY_D_325_1_1" FriendlyName="PowerShellShell 325" Hash="57ABBC8E2FE88E04C57CDDD13D58C9CE03455D25" />
    <Deny ID="ID_DENY_D_326_1_1" FriendlyName="PowerShellShell 326" Hash="0280C4714BC806BFC1863BE9E84D38F203942DD35C6AF2EB96958FD011E4D23D" />
    <Deny ID="ID_DENY_D_327_1_1" FriendlyName="PowerShellShell 327" Hash="DEB07053D6059B56109DFF885720D5721EB0F55C" />
    <Deny ID="ID_DENY_D_328_1_1" FriendlyName="PowerShellShell 328" Hash="E374A14871C35DB57D6D67281C16F5F9EF77ABE248DE92C1A937C6526133FA36" />
    <Deny ID="ID_DENY_D_329_1_1" FriendlyName="PowerShellShell 329" Hash="AC33BA432B35A662E2D9D015D6283308FD046251" />
    <Deny ID="ID_DENY_D_330_1_1" FriendlyName="PowerShellShell 330" Hash="93B22B0D5369327247DF491AABD3CE78421D0D68FE8A3931E0CDDF5F858D3AA7" />
    <Deny ID="ID_DENY_D_331_1_1" FriendlyName="PowerShellShell 331" Hash="05126413310F4A1BA2F7D2AD3305E2E3B6A1B00D" />
    <Deny ID="ID_DENY_D_332_1_1" FriendlyName="PowerShellShell 332" Hash="108A73F4AE78786C9955ED71EFD916465A36175F8DC85FD82DDA6410FBFCDB52" />
    <Deny ID="ID_DENY_D_333_1_1" FriendlyName="PowerShellShell 333" Hash="B976F316FB5EE6E5A325320E7EE5FBF487DA9CE5" />
    <Deny ID="ID_DENY_D_334_1_1" FriendlyName="PowerShellShell 334" Hash="D54CCD405D3E904CAECA3A6F7BE1737A9ACE20F7593D0F6192B811EF17744DD6" />
    <Deny ID="ID_DENY_D_335_1_1" FriendlyName="PowerShellShell 335" Hash="F3471DBF534995307AEA230D228BADFDCA9E4021" />
    <Deny ID="ID_DENY_D_336_1_1" FriendlyName="PowerShellShell 336" Hash="2048F33CCD924D224154307C28DDC6AC1C35A1859F118AB2B6536FB954FC44EF" />
    <Deny ID="ID_DENY_D_337_1_1" FriendlyName="PowerShellShell 337" Hash="1FAC9087885C2FEBD7F57CC9AACE8AF94294C8FB" />
    <Deny ID="ID_DENY_D_338_1_1" FriendlyName="PowerShellShell 338" Hash="942E0D0BA5ECBF64A3B2D0EA1E08C793712A4C89BC1BC3B6C32A419AE38FACC1" />
    <Deny ID="ID_DENY_D_339_1_1" FriendlyName="PowerShellShell 339" Hash="5B67EE19AA7E4B42E58127A63520D44A0679C6CE" />
    <Deny ID="ID_DENY_D_340_1_1" FriendlyName="PowerShellShell 340" Hash="2B6A59053953737D345B97FA1AFB23C379809D1532BAF31E710E48ED7FA2D735" />
    <Deny ID="ID_DENY_D_341_1_1" FriendlyName="PowerShellShell 341" Hash="1ABC67650B169E7C437853922805706D488EEEA2" />
    <Deny ID="ID_DENY_D_342_1_1" FriendlyName="PowerShellShell 342" Hash="754CA97A95464F1A1687C83AE3ECC6670B80A50503067DEBF6135077C886BCF4" />
    <Deny ID="ID_DENY_D_343_1_1" FriendlyName="PowerShellShell 343" Hash="0E280FF775F406836985ECA66BAA9BA17D12E38B" />
    <Deny ID="ID_DENY_D_344_1_1" FriendlyName="PowerShellShell 344" Hash="19C9A6D1AE90AEA163E35930FAB1B57D3EC78CA5FE192D6E510CED2DAB5DD03B" />
    <Deny ID="ID_DENY_D_345_1_1" FriendlyName="PowerShellShell 345" Hash="4E6081C3BBB2809C417E2D03412E29FF7317DA54" />
    <Deny ID="ID_DENY_D_346_1_1" FriendlyName="PowerShellShell 346" Hash="3AE4505A552EA04C7664C610E81172CA329981BF53ECC6758C03357EB653F5D1" />
    <Deny ID="ID_DENY_D_347_1_1" FriendlyName="PowerShellShell 347" Hash="61BED1C7CD54B2F60923D26CD2F6E48C063AFED5" />
    <Deny ID="ID_DENY_D_348_1_1" FriendlyName="PowerShellShell 348" Hash="9405CBE91B7519290F90577DCCF5796C514746DE6390322C1624BA258D284EE9" />
    <Deny ID="ID_DENY_D_349_1_1" FriendlyName="PowerShellShell 349" Hash="63AA55C3B46EFAFC8625F8D5562AB504E4CBB78F" />
    <Deny ID="ID_DENY_D_350_1_1" FriendlyName="PowerShellShell 350" Hash="FF54885D30A13008D60F6D0B96CE802209C89A2A7D9D86A85804E66B6DE29A5D" />
    <Deny ID="ID_DENY_D_351_1_1" FriendlyName="PowerShellShell 351" Hash="20845E4440DA2D9AB3559D4B6890691CACD0E93E" />
    <Deny ID="ID_DENY_D_352_1_1" FriendlyName="PowerShellShell 352" Hash="3C9098C4BFD818CE8CFA130F6E6C90876B97D57ABBEAFABB565C487F1DD33ECC" />
    <Deny ID="ID_DENY_D_353_1_1" FriendlyName="PowerShellShell 353" Hash="4A473F14012EB9BF7DCEA80B86C2612A6D9D914E" />
    <Deny ID="ID_DENY_D_354_1_1" FriendlyName="PowerShellShell 354" Hash="1C6914B58F70A9860F67311C32258CD9072A367BF30203DA9D8C48188D888E65" />
    <Deny ID="ID_DENY_D_355_1_1" FriendlyName="PowerShellShell 355" Hash="641871FD5D9875DB75BFC58B7B53672D2C645F01" />
    <Deny ID="ID_DENY_D_356_1_1" FriendlyName="PowerShellShell 356" Hash="C115A974DD2C56574E93A4800247A23B98B9495F6EF41460D1EC139266A2484D" />
    <Deny ID="ID_DENY_D_357_1_1" FriendlyName="PowerShellShell 357" Hash="A21E254C18D3D53B832AD381FF58B36E6737FFB6" />
    <Deny ID="ID_DENY_D_358_1_1" FriendlyName="PowerShellShell 358" Hash="D214AF2AD9204118EB670D08D80D4CB9FFD74A978726240360C35AD5A57F8E7D" />
    <Deny ID="ID_DENY_D_359_1_1" FriendlyName="PowerShellShell 359" Hash="102B072F29122BC3A89B924987A7BF1AC3C598DB" />
    <Deny ID="ID_DENY_D_360_1_1" FriendlyName="PowerShellShell 360" Hash="DA444773FE7AD8309FA9A0ABCDD63B302E6FC91E750903843FBA2A7F370DB0C0" />
    <Deny ID="ID_DENY_D_361_1_1" FriendlyName="PowerShellShell 361" Hash="EAD58EBB00001E678B9698A209308CC7406E1BCC" />
    <Deny ID="ID_DENY_D_362_1_1" FriendlyName="PowerShellShell 362" Hash="34A5F48629F9FDAEBAB9468EF7F1683EFA856AAD32E3C0CC0F92B5641D722EDC" />
    <Deny ID="ID_DENY_D_363_1_1" FriendlyName="PowerShellShell 363" Hash="727EDB00C15DC5D3C14368D88023FDD5A74C0B06" />
    <Deny ID="ID_DENY_D_364_1_1" FriendlyName="PowerShellShell 364" Hash="5720BEE5CBE7D724B67E07C53E22FB869F8F9B1EB95C4F71D61D240A1ED8D8AD" />
    <Deny ID="ID_DENY_D_365_1_1" FriendlyName="PowerShellShell 365" Hash="A43137EC82721A81C3E05DC5DE74F0549DE6A130" />
    <Deny ID="ID_DENY_D_366_1_1" FriendlyName="PowerShellShell 366" Hash="1731118D97F278C18E2C6922A016DA7C55970C6C4C5441710D1B0464EED6EAEB" />
    <Deny ID="ID_DENY_D_367_1_1" FriendlyName="PowerShellShell 367" Hash="17EC94CB9BF98E605F9352987CA33DCE8F5733CD" />
    <Deny ID="ID_DENY_D_368_1_1" FriendlyName="PowerShellShell 368" Hash="AFE0CC143108BBDBE60771B6894406785C471BA5730F06EE8185D0A71617B583" />
    <Deny ID="ID_DENY_D_369_1_1" FriendlyName="PowerShellShell 369" Hash="F6E9C098737F0905E53B92D4AD49C199EC76D24B" />
    <Deny ID="ID_DENY_D_370_1_1" FriendlyName="PowerShellShell 370" Hash="50A57BFCD20380DDEFD2A717D7937D49380D4D5931CC6CC403C904139546CB1D" />
    <Deny ID="ID_DENY_D_371_1_1" FriendlyName="PowerShellShell 371" Hash="2118ACC512464EE95946F064560C15C58341B80C" />
    <Deny ID="ID_DENY_D_372_1_1" FriendlyName="PowerShellShell 372" Hash="005990EE785C1CA7EAEC82DA29F5B363049DC117A18823D83C10B86B5E8D0A5F" />
    <Deny ID="ID_DENY_D_373_1_1" FriendlyName="PowerShellShell 373" Hash="54FAE3A389FDD2F5C21293D2317E87766AF0473D" />
    <Deny ID="ID_DENY_D_374_1_1" FriendlyName="PowerShellShell 374" Hash="70F4E503D7484DF5B5F73D9A753E585BFADB8B8EBA42EB482B6A66DB17C87881" />
    <Deny ID="ID_DENY_D_375_1_1" FriendlyName="PowerShellShell 375" Hash="B4831AF4B25527EF0C172DAA5E4CA26DE105D30B" />
    <Deny ID="ID_DENY_D_376_1_1" FriendlyName="PowerShellShell 376" Hash="D410A37042A2DC53AD1801EBB2EF507B4AE475870522A298567B79DA61C3E9C8" />
    <Deny ID="ID_DENY_D_377_1_1" FriendlyName="PowerShellShell 377" Hash="85BBC0CDC34BD5A56113B0DCB6795BCEBADE63FA" />
    <Deny ID="ID_DENY_D_378_1_1" FriendlyName="PowerShellShell 378" Hash="C6F8E3A3F2C513CEDD2F21D486BF0116BAF2E2EE4D631A9BE4760860B1161848" />
    <Deny ID="ID_DENY_D_379_1_1" FriendlyName="PowerShellShell 379" Hash="46105ACE7ABEC3A6E6226183F2F7F8E90E3639A5" />
    <Deny ID="ID_DENY_D_380_1_1" FriendlyName="PowerShellShell 380" Hash="F60BE088F226CA1E2308099C3B1C2A54DB4C41D2BE678504D03547B9E1E023F6" />
    <Deny ID="ID_DENY_D_381_1_1" FriendlyName="PowerShellShell 381" Hash="C9478352ACE4BE6D6B70BBE710C2E2128FEFC7FE" />
    <Deny ID="ID_DENY_D_382_1_1" FriendlyName="PowerShellShell 382" Hash="F4A81E7D4BD3B8762FAED760047877E06E40EC991D968BD6A6929B848804C1A4" />
    <Deny ID="ID_DENY_D_383_1_1" FriendlyName="PowerShellShell 383" Hash="9E56E910919FF65BCCF5D60A8F9D3EBE27EF1381" />
    <Deny ID="ID_DENY_D_384_1_1" FriendlyName="PowerShellShell 384" Hash="34887B225444A18158B632CAEA4FEF6E7D691FEA3E36C12D4152AFAB260668EB" />
    <Deny ID="ID_DENY_D_385_1_1" FriendlyName="PowerShellShell 385" Hash="1FD04D4BD5F9E41FA8278F3F9B05FE8702ADB4C8" />
    <Deny ID="ID_DENY_D_386_1_1" FriendlyName="PowerShellShell 386" Hash="6586176AEBE8307829A1E03D878EF6F500E8C5032E50198DF66F54D3B56EA718" />
    <Deny ID="ID_DENY_D_387_1_1" FriendlyName="PowerShellShell 387" Hash="DEBC3DE2AD99FC5E885A358A6994E6BD39DABCB0" />
    <Deny ID="ID_DENY_D_388_1_1" FriendlyName="PowerShellShell 388" Hash="FDF54A4A3089062FFFA4A41FEBF38F0ABC9D502B57749348DF6E78EA2A33DDEA" />
    <Deny ID="ID_DENY_D_389_1_1" FriendlyName="PowerShellShell 389" Hash="6AA06D07D9DE8FE7E13B66EDFA07232B56F7E21D" />
    <Deny ID="ID_DENY_D_390_1_1" FriendlyName="PowerShellShell 390" Hash="DD3E74CFB8ED64FA5BE9136C305584CD2E529D92B360651DD06A6DC629E23449" />
    <Deny ID="ID_DENY_D_391_1_1" FriendlyName="PowerShellShell 391" Hash="5C858042246FDDDB281C1BFD2FEFC9BAABC3F7AD" />
    <Deny ID="ID_DENY_D_392_1_1" FriendlyName="PowerShellShell 392" Hash="20E65B1BE06A99507412FC0E75D158EE1D9D43AE5F492BE4A87E3AA29A148310" />
    <Deny ID="ID_DENY_D_393_1_1" FriendlyName="PowerShellShell 393" Hash="2ABCD0525D31D4BB2D0131364FBE1D94A02A3E2A" />
    <Deny ID="ID_DENY_D_394_1_1" FriendlyName="PowerShellShell 394" Hash="806EC87F1EFA428627989318C882CD695F55F60A1E865C621C9F2B14E4E1FC2E" />
    <Deny ID="ID_DENY_D_395_1_1" FriendlyName="PowerShellShell 395" Hash="E2967D755D0F79FA8EA7A8585106926CA87F89CB" />
    <Deny ID="ID_DENY_D_396_1_1" FriendlyName="PowerShellShell 396" Hash="07382BE9D8ACBAFDA953C842BAAE600A82A69183D6B63F91B061671C4AF9434B" />
    <Deny ID="ID_DENY_D_397_1_1" FriendlyName="PowerShellShell 397" Hash="75EF6F0B78098FB1766DCC853E004476033499CF" />
    <Deny ID="ID_DENY_D_398_1_1" FriendlyName="PowerShellShell 398" Hash="699A9D17E1247F05767E82BFAFBD96DBE07AE521E23D39613D4A39C3F8CF4971" />
    <Deny ID="ID_DENY_D_399_1_1" FriendlyName="PowerShellShell 399" Hash="E73178C487AF6B9F182B2CCA25774127B0303093" />
    <Deny ID="ID_DENY_D_400_1_1" FriendlyName="PowerShellShell 400" Hash="0BD1FE62BE97032ADDAAB41B445D00103302D3CE8A03A798A36FEAA0F89939FF" />
    <Deny ID="ID_DENY_D_401_1_1" FriendlyName="PowerShellShell 401" Hash="EBF20FEECA95F83B9F5C22B97EB44DD7EB2C7B5F" />
    <Deny ID="ID_DENY_D_402_1_1" FriendlyName="PowerShellShell 402" Hash="B5AE0EAA5AF4245AD9B37C8C1FC5220081B92A13950C54D82E824D2D3B840A7C" />
    <Deny ID="ID_DENY_D_403_1_1" FriendlyName="PowerShellShell 403" Hash="5E53A4235DC549D0195A9DDF607288CEDE7BF115" />
    <Deny ID="ID_DENY_D_404_1_1" FriendlyName="PowerShellShell 404" Hash="FE57195757977E4485BF5E5D72A24EA65E33F8EAA7245381453960D5646FAF58" />
    <Deny ID="ID_DENY_D_405_1_1" FriendlyName="PowerShellShell 405" Hash="014BC30E1FC12F270824F01DC7C934497A573124" />
    <Deny ID="ID_DENY_D_406_1_1" FriendlyName="PowerShellShell 406" Hash="65B3B357C356DAE26E5B036820C193989C0F9E8E08131B3186F9443FF9A511E4" />
    <Deny ID="ID_DENY_D_411_1_1" FriendlyName="PowerShellShell 411" Hash="8287B536E8E63F024DE1248D0FE3E6A759E9ACEE" />
    <Deny ID="ID_DENY_D_412_1_1" FriendlyName="PowerShellShell 412" Hash="B714D4A700A56BC1D4B3F59DFC1F5835CB97CBEF3927523BF71AF96B00F0FFA4" />
    <Deny ID="ID_DENY_D_427_1_1" FriendlyName="PowerShellShell 427" Hash="EA157E01147629D1F59503D8335FB6EBC688B2C1" />
    <Deny ID="ID_DENY_D_428_1_1" FriendlyName="PowerShellShell 428" Hash="14C160DF95736EC1D7C6C55B9D0F81832E8FE0DB6C5931B23E45A559995A1000" />
    <Deny ID="ID_DENY_D_435_1_1" FriendlyName="PowerShellShell 435" Hash="6792915D3C837A39BD04AD169488009BB1EA372C" />
    <Deny ID="ID_DENY_D_436_1_1" FriendlyName="PowerShellShell 436" Hash="23B10EC5FC7EAEB9F8D147163463299328FAED4B973BB862ECD3F28D6794DA9D" />
    <Deny ID="ID_DENY_D_441_1_1" FriendlyName="PowerShellShell 441" Hash="24F9CF6C5E9671A295AD0DEED74737FB6E9146DE" />
    <Deny ID="ID_DENY_D_442_1_1" FriendlyName="PowerShellShell 442" Hash="C2E862CC578F54A53496EEE2DCB534A106AFD55C7288362AF6499B45F8D8755E" />
    <Deny ID="ID_DENY_D_457_1_1" FriendlyName="PowerShellShell 457" Hash="D5A9460A941FB5B49EAFDD57575CFB23F27779D3" />
    <Deny ID="ID_DENY_D_458_1_1" FriendlyName="PowerShellShell 458" Hash="4BDAAC1654328E4D37B6ED89DA351155438E558F51458F2129AFFAC5B596CD61" />
    <Deny ID="ID_DENY_D_463_1_1" FriendlyName="PowerShellShell 463" Hash="C647D17850941CFB5B9C8AF49A48569B52230274" />
    <Deny ID="ID_DENY_D_464_1_1" FriendlyName="PowerShellShell 464" Hash="0BCBDE8791E3D6D7A7C8FC6F25E14383014E6B43D9720A04AF0BD4BDC37F79E0" />
    <Deny ID="ID_DENY_D_465_1_1" FriendlyName="PowerShellShell 465" Hash="CA6E0BAB6B28E1592D0FC5940023C7A81E2568F8" />
    <Deny ID="ID_DENY_D_466_1_1" FriendlyName="PowerShellShell 466" Hash="366E00E2F517D4D404133AEFEF6F917DFA156E3E46D350A8CBBE59BE1FB877A2" />
    <Deny ID="ID_DENY_D_467_1_1" FriendlyName="PowerShellShell 467" Hash="7D9FFFA86DDCD227A3B4863D995456308BAC2403" />
    <Deny ID="ID_DENY_D_468_1_1" FriendlyName="PowerShellShell 468" Hash="4439BBF61DC012AFC8190199AF5722C3AE26F365DEE618D0D945D75FD1AABF3C" />
    <Deny ID="ID_DENY_D_469_1_1" FriendlyName="PowerShellShell 469" Hash="8FFDD4576F2B6D4999326CFAF67727BFB471FA21" />
    <Deny ID="ID_DENY_D_470_1_1" FriendlyName="PowerShellShell 470" Hash="94630AB6F60A7193A6E27E312AF9B71DA265D42AD49465F4EEA11EBF134BA54A" />
    <Deny ID="ID_DENY_D_471_1_1" FriendlyName="PowerShellShell 471" Hash="78B8454F78E216B629E43B4E40765F73BFE0D6C6" />
    <Deny ID="ID_DENY_D_472_1_1" FriendlyName="PowerShellShell 472" Hash="498BB1688410EE243D61FB5C7B37457FA6C0A9A32D136AF70FAD43D5F37D7A81" />
    <Deny ID="ID_DENY_D_475_1_1" FriendlyName="PowerShellShell 475" Hash="8AF579DE1D7E590A13BD1DAE5BFDB39476068A05" />
    <Deny ID="ID_DENY_D_476_1_1" FriendlyName="PowerShellShell 476" Hash="9917A3055D194F47AB295FA3F917E4BD2F08DDF45C04C65C591A020E1507A573" />
    <Deny ID="ID_DENY_D_477_1_1" FriendlyName="PowerShellShell 477" Hash="DD64046BAB221CF4110FF230FA5060310A4D9610" />
    <Deny ID="ID_DENY_D_478_1_1" FriendlyName="PowerShellShell 478" Hash="A55AF37229D7E249C8CAFED3432E595AA77FAF8B62990C07938220E957679081" />
    <Deny ID="ID_DENY_D_483_1_1" FriendlyName="PowerShellShell 483" Hash="2F587293F16DFCD06F3BF8B8348FF68827ECD307" />
    <Deny ID="ID_DENY_D_484_1_1" FriendlyName="PowerShellShell 484" Hash="B2F4A5FE21D5961F464CAB3E88C0ED88154B0C1A422629474AD5C9EDC11880B6" />
    <Deny ID="ID_DENY_D_493_1_1" FriendlyName="PowerShellShell 493" Hash="E180486F0CC90AF4FB8283ADCF571884894513C8" />
    <Deny ID="ID_DENY_D_494_1_1" FriendlyName="PowerShellShell 494" Hash="3800E38275E6BB3B4645CDAD14CD756239BB9A87EF261DC1B68072B6DB2850C0" />
    <Deny ID="ID_DENY_D_503_1_1" FriendlyName="PowerShellShell 503" Hash="231A02EAB7EB192638BC89AB61A5077346FF22B9" />
    <Deny ID="ID_DENY_D_504_1_1" FriendlyName="PowerShellShell 504" Hash="4D544170DE5D9916678EA43A7C6F796FC02EFA9197C6E0C01A1D832BF554F748" />
    <Deny ID="ID_DENY_D_507_1_1" FriendlyName="PowerShellShell 507" Hash="15EF1F7DBC474732E122A0147640ACBD9DA1775C" />
    <Deny ID="ID_DENY_D_508_1_1" FriendlyName="PowerShellShell 508" Hash="04724BF232D5F169FBB0DB6821E35D772619FB4F24069BE0EC571BA622ACC4D2" />
    <Deny ID="ID_DENY_D_509_1_1" FriendlyName="PowerShellShell 509" Hash="7959AB2B34A5F490AD54782D135BF155592DF13F" />
    <Deny ID="ID_DENY_D_510_1_1" FriendlyName="PowerShellShell 510" Hash="DD03CD6B5655B4EB9DD259F26E1585389804C23DB39C10122B6BC0E8886B4C2A" />
    <Deny ID="ID_DENY_D_511_1_1" FriendlyName="PowerShellShell 511" Hash="CCA8C8FB699496BD50AE296B20CC9ADC3496DECE" />
    <Deny ID="ID_DENY_D_512_1_1" FriendlyName="PowerShellShell 512" Hash="75E6C2DD81FE2664DF466C9C2EB0F923B0C6D992FF653B673793A896D8860957" />
    <Deny ID="ID_DENY_D_515_1_1" FriendlyName="PowerShellShell 515" Hash="B3B7A653DD1A10EE9A3D35C818D227E2E3C3B5FB" />
    <Deny ID="ID_DENY_D_516_1_1" FriendlyName="PowerShellShell 516" Hash="43E2D91C0C6A8473BE178F1793E5E34966D700F71362297ECF4B5D46239603E3" />
    <Deny ID="ID_DENY_D_519_1_1" FriendlyName="PowerShellShell 519" Hash="AAE22FD137E8B7217222974DCE60B9AD4AF2A512" />
    <Deny ID="ID_DENY_D_520_1_1" FriendlyName="PowerShellShell 520" Hash="DAC9E963A3897D7F7AB2B4FEBBD4894A15441246639CE3E8EE74B0228F312742" />
    <Deny ID="ID_DENY_D_527_1_1" FriendlyName="PowerShellShell 527" Hash="25CA971D7EDFAA7A48FA19B8399301853809D7CC" />
    <Deny ID="ID_DENY_D_528_1_1" FriendlyName="PowerShellShell 528" Hash="0A10C71CB5CC8A801F84F2CCD8041D13DB55711435388D9500C53D122688D4E5" />
    <Deny ID="ID_DENY_D_529_1_1" FriendlyName="PowerShellShell 529" Hash="46E05FD4D62451C1DCB0287B32B3D77AD41544EA" />
    <Deny ID="ID_DENY_D_530_1_1" FriendlyName="PowerShellShell 530" Hash="D86F930445F0715D0D7E4C3B089399280FBA2ACE0E4125BA5D3DAB9FAC1A6D3A" />
    <Deny ID="ID_DENY_D_537_1_1" FriendlyName="PowerShellShell 537" Hash="46936F4F0AFE4C87D2E55595F74DDDFFC9AD94EE" />
    <Deny ID="ID_DENY_D_538_1_1" FriendlyName="PowerShellShell 538" Hash="9843DC862BC7491A279A09EFD8FF122EB23C57CA" />
    <Deny ID="ID_DENY_D_539_1_1" FriendlyName="PowerShellShell 539" Hash="11F11FB1E57F299383A615D6A28436E02A1C1A83" />
    <Deny ID="ID_DENY_D_540_1_1" FriendlyName="PowerShellShell 540" Hash="C593ABE79DFFB1504CFCDB1A6AD65D24996E7B97" />
    <Deny ID="ID_DENY_D_541_1_1" FriendlyName="PowerShellShell 541" Hash="93E22F2BA6C8B1C09F100F9C0E3B06FAF2D1DDB6" />
    <Deny ID="ID_DENY_D_542_1_1" FriendlyName="PowerShellShell 542" Hash="5A8D9712CF7893C335FFB7414748625D524227FE" />
    <Deny ID="ID_DENY_D_543_1_1" FriendlyName="PowerShellShell 543" Hash="B5FFFEE20F25691A59F3894644AEF088B4845761" />
    <Deny ID="ID_DENY_D_544_1_1" FriendlyName="PowerShellShell 544" Hash="3334059FF4484C43A5D08CEC3E43E2D27EDB927B" />
    <Deny ID="ID_DENY_D_545_1_1" FriendlyName="PowerShellShell 545" Hash="00B6993F59990C3DFEA33584BDB050F91313B17A" />
    <Deny ID="ID_DENY_D_546_1_1" FriendlyName="PowerShellShell 546" Hash="7518F60A0B33011D19873908559961F96A9B4FC0" />
    <Deny ID="ID_DENY_D_547_1_1" FriendlyName="PowerShellShell 547" Hash="A1D1AF7675C2596D0DF977F57B54372298A56EE0F3E1FF2D974D387D7F69DD4E" />
    <Deny ID="ID_DENY_D_548_1_1" FriendlyName="PowerShellShell 548" Hash="3C1743CBC43B80F5AF5B17239B03A8727B4BE81F14052BDE37685E2D54214071" />
    <Deny ID="ID_DENY_D_549_1_1" FriendlyName="PowerShellShell 549" Hash="C7DC8B00F0BDA000D1F3CF0FBC7AB32D443C377C0130BB5153A0390E712DDDE5" />
    <Deny ID="ID_DENY_D_550_1_1" FriendlyName="PowerShellShell 550" Hash="ED5A4747C8AEEB1AC2F4FDB8EB0B9BFC240F2B3C00BF7C6CDB372BFFEC0F8ABE" />
    <Deny ID="ID_DENY_D_551_1_1" FriendlyName="PowerShellShell 551" Hash="939C291D4A2592209EC7664EC832670FA0AC1009F974F47489D866751F4B862F" />
    <Deny ID="ID_DENY_D_552_1_1" FriendlyName="PowerShellShell 552" Hash="497A2D4207B2AE6EF09424591624A86A64A2C8E451389ED9A3256E6274556A7B" />
    <Deny ID="ID_DENY_D_553_1_1" FriendlyName="PowerShellShell 553" Hash="732BC385B191C8436B42CD1441DC234FFDD5EC1BD18A32894F093EECA3DD8FBC" />
    <Deny ID="ID_DENY_D_554_1_1" FriendlyName="PowerShellShell 554" Hash="CBD19FDB6338DB02299A3F3FFBBEBF216B18013B3377D1D31E51491C0C5F074C" />
    <Deny ID="ID_DENY_D_555_1_1" FriendlyName="PowerShellShell 555" Hash="3A316A0A470744EB7D18339B76E786564D1E96130766A9895B2222C4066CE820" />
    <Deny ID="ID_DENY_D_556_1_1" FriendlyName="PowerShellShell 556" Hash="68A4A1E8F4E1B903408ECD24608659B390B9E7154EB380D94ADE7FEB5EA470E7" />
    <Deny ID="ID_DENY_D_557_1_1" FriendlyName="PowerShellShell 557" Hash="45F948AF27F4E698A8546027717901B5F70368EE" />
    <Deny ID="ID_DENY_D_558_1_1" FriendlyName="PowerShellShell 558" Hash="2D63C337961C6CF2660C5DB906D9070CA38BCE828584874680EC4F5097B82E30" />
    <Deny ID="ID_DENY_D_559_1_1" FriendlyName="PowerShellShell 559" Hash="DA4CD4B0158B774CE55721718F77ED91E3A42EB3" />
    <Deny ID="ID_DENY_D_560_1_1" FriendlyName="PowerShellShell 560" Hash="7D181BB7A4A0755FF687CCE34949FC6BD6FBC377E6D4883698E8B45DCCBEA140" />
    <Deny ID="ID_DENY_D_561_1_1" FriendlyName="PowerShellShell 561" Hash="C67D7B12BBFFD5FBD15FBD892955EA48E6F4B408" />
    <Deny ID="ID_DENY_D_562_1_1" FriendlyName="PowerShellShell 562" Hash="1DCAD0BBCC036B85875CC0BAF1B65027933624C1A29BE336C79BCDB00FD5467A" />
    <Deny ID="ID_DENY_D_563_1_1" FriendlyName="PowerShellShell 563" Hash="7D8CAB8D9663926E29CB810B42C5152E8A1E947E" />
    <Deny ID="ID_DENY_D_564_1_1" FriendlyName="PowerShellShell 564" Hash="2E0203370E6E5437CE2CE1C20895919F806B4E5FEBCBE31F16CB06FC5934F010" />
    <Deny ID="ID_DENY_D_565_1_1" FriendlyName="PowerShellShell 565" Hash="20E7156E348912C20D35BD4BE2D52C996BF5535E" />
    <Deny ID="ID_DENY_D_566_1_1" FriendlyName="PowerShellShell 566" Hash="EB26078544BDAA34733AA660A1A2ADE98523DAFD9D58B3995919C0E524F2FFC3" />
    <Deny ID="ID_DENY_D_567_1_1" FriendlyName="PowerShellShell 567" Hash="B9DD16FC0D02EA34613B086307C9DBEAC30546AF" />
    <Deny ID="ID_DENY_D_568_1_1" FriendlyName="PowerShellShell 568" Hash="DE5B012C4DC3FE3DD432AF9339C36EFB8D54E8864493EA2BA151F0ADBF3E338C" />
    <Deny ID="ID_DENY_D_569_1_1" FriendlyName="PowerShellShell 569" Hash="6397AB5D664CDB84A867BC7E22ED0789060C6276" />
    <Deny ID="ID_DENY_D_570_1_1" FriendlyName="PowerShellShell 570" Hash="B660F6CA0788DA18375602537095C378990E8229B11B57B092AC8A550E9C61E8" />
    <Deny ID="ID_DENY_D_571_1_1" FriendlyName="PowerShellShell 571" Hash="3BF717645AC3986AAD0B4EA9D196B18D05199DA9" />
    <Deny ID="ID_DENY_D_572_1_1" FriendlyName="PowerShellShell 572" Hash="364C227F9E57C72F9BFA652B8C1DE738AB4747D0DB68A7B899CA3EE51D802439" />
    <Deny ID="ID_DENY_D_573_1_1" FriendlyName="PowerShellShell 573" Hash="3A1B06680F119C03C60D12BAC682853ABE430D21" />
    <Deny ID="ID_DENY_D_574_1_1" FriendlyName="PowerShellShell 574" Hash="850759BCE4B66997CF84E84683A2C1980D4B498821A8AB9C3568EB298B824AE3" />
    <Deny ID="ID_DENY_D_575_1_1" FriendlyName="PowerShellShell 575" Hash="654C54AA3F2C74FBEB55B961FB1924A7B2737E61" />
    <Deny ID="ID_DENY_D_576_1_1" FriendlyName="PowerShellShell 576" Hash="B7EA81960C6EECFD2FF385890F158F5B1CB3D1E100C7157AB161B3D23DCA0389" />
    <Deny ID="ID_DENY_D_577_1_1" FriendlyName="PowerShellShell 577" Hash="496F793112B6BCF4B6EA16E8B2F8C3F5C1FEEB52" />
    <Deny ID="ID_DENY_D_578_1_1" FriendlyName="PowerShellShell 578" Hash="E430485B577774825CEF53E5125B618A2608F7BE3657BB28383E9A34FCA162FA" />
    <Deny ID="ID_DENY_D_579_1_1" FriendlyName="PowerShellShell 579" Hash="6EA8CEEA0D2879989854E8C86CECA26EF79F7B19" />
    <Deny ID="ID_DENY_D_580_1_1" FriendlyName="PowerShellShell 580" Hash="8838FE3D8E2505F3D3D8B98C64739115838A0B443BBBBFB487342F1EE7801360" />
    <Deny ID="ID_DENY_D_581_1_1" FriendlyName="PowerShellShell 581" Hash="28C5E53DE197E872F7E4772BF40F728F56FE3ACC" />
    <Deny ID="ID_DENY_D_582_1_1" FriendlyName="PowerShellShell 582" Hash="3493DAEC6EC03E56ECC4A15432C750735F75F9CB38D8779C7783B4DA956BF037" />
    <Deny ID="ID_DENY_D_583_1_1" FriendlyName="Winrm 583" Hash="3FA2D2963CBF47FFD5F7F5A9B4576F34ED42E552" />
    <Deny ID="ID_DENY_D_584_1_1" FriendlyName="Winrm 584" Hash="6C96E976DC47E0C99B77814E560E0DC63161C463C75FA15B7A7CA83C11720E82" />
    <Deny ID="ID_DENY_D_585_1_1" FriendlyName="PowerShellShell 585" Hash="DBB5A6F5388C574A3B5B63E65F7810AB271E9A77" />
    <Deny ID="ID_DENY_D_586_1_1" FriendlyName="PowerShellShell 586" Hash="6DB24D174CCF06C9138B5A9320AE4261CA0CF305357DEF1B7054DD84758E92AB" />
    <Deny ID="ID_DENY_D_587_1_1" FriendlyName="PowerShellShell 587" Hash="757626CF5D444F5A4AF79EDE38E9EF65FA2C9802" />
    <Deny ID="ID_DENY_D_588_1_1" FriendlyName="PowerShellShell 588" Hash="1E17D036EBB5E82BF2FD5BDC3ABAB08B5EA9E4504D989D2BAAAA0B6047988996" />
    <Deny ID="ID_DENY_D_589_1_1" FriendlyName="PowerShellShell 589" Hash="2965DC840B8F5F7ED2AEC979F21EADA664E3CB70" />
    <Deny ID="ID_DENY_D_590_1_1" FriendlyName="PowerShellShell 590" Hash="5449560095D020687C268BD34D9425E7A2739E1B9BFBC0886142519293E02B9D" />
    <Deny ID="ID_DENY_D_591_1_1" FriendlyName="PowerShellShell 591" Hash="BB47C1251866F87723A7EDEC9A01D3B955BAB846" />
    <Deny ID="ID_DENY_D_592_1_1" FriendlyName="PowerShellShell 592" Hash="B05F3BE23DE6AE2557D6661C6FE35E114E8A69B326A3C855023B7AC5CE9FC31B" />
    <Deny ID="ID_DENY_D_593_1_1" FriendlyName="PowerShellShell 593" Hash="2F3D30827E02D5FEF051E54C74ECA6AD4CC4BAD2" />
    <Deny ID="ID_DENY_D_594_1_1" FriendlyName="PowerShellShell 594" Hash="F074589A1FAA76A751B05AD61B968683134F3FFC10DE3077FBCEE4E263EAEB0D" />
    <Deny ID="ID_DENY_D_595_1_1" FriendlyName="PowerShellShell 595" Hash="10096BD0A359142A13F2B8023A341C79A4A97975" />
    <Deny ID="ID_DENY_D_596_1_1" FriendlyName="PowerShellShell 596" Hash="A271D72CDE48F69EB694B753BF9417CD6A72F7DA06C52E47BAB40EC2BD9DD819" />
    <Deny ID="ID_DENY_D_597_1_1" FriendlyName="PowerShellShell 597" Hash="F8E803E1623BA66EA2EE0751A648834130B8BE5D" />
    <Deny ID="ID_DENY_D_598_1_1" FriendlyName="PowerShellShell 598" Hash="E70DB033B773FE01B1D4464CAC112AF41C09E75D25FEA25AE8DAE67ED941E797" />
    <Deny ID="ID_DENY_D_599_1_1" FriendlyName="PowerShellShell 599" Hash="665BE52329F9CECEC1CD548A1B4924C9B1F79BD8" />
    <Deny ID="ID_DENY_D_600_1_1" FriendlyName="PowerShellShell 600" Hash="24CC5B946D9469A39CF892DD4E92117E0E144DC7C6FAA65E71643DEAB87B2A91" />
    <Deny ID="ID_DENY_D_601_1_1" FriendlyName="PowerShellShell 601" Hash="C4627F2CF69A8575D7BF7065ADF5354D96707DFD" />
    <Deny ID="ID_DENY_D_602_1_1" FriendlyName="PowerShellShell 602" Hash="7F1DF759C050E0EF4F9F96FF43904B418C674D4830FE61818B60CC68629F5ABA" />
    <Deny ID="ID_DENY_D_603_1_1" FriendlyName="PowerShellShell 603" Hash="4126DD5947E63DB50AD5C135AC39856B6ED4BF33" />
    <Deny ID="ID_DENY_D_604_1_1" FriendlyName="PowerShellShell 604" Hash="B38E1198F82E7C2B3123984C017417F2A48BDFF5B6DBAD20B2438D7B65F6E39F" />
    <Deny ID="ID_DENY_D_605_1_1" FriendlyName="PowerShellShell 605" Hash="DE16A6B93178B6C6FC33FBF3E9A86CFF070DA6D3" />
    <Deny ID="ID_DENY_D_606_1_1" FriendlyName="PowerShellShell 606" Hash="A3EF9A95D1E859958DEBE44C033B4562EBB9B4C6E32005CA5C07B2E07A42E2BE" />
    <FileAttrib ID="ID_FILEATTRIB_LIBNICM_DRIVER_2" FriendlyName="" FileName="libnicm.sys" MinimumFileVersion="0.0.0.0" MaximumFileVersion="3.1.12.0" />
    <FileAttrib ID="ID_FILEATTRIB_NICM_DRIVER_2" FriendlyName="" FileName="NICM.SYS" MinimumFileVersion="0.0.0.0" MaximumFileVersion="3.1.12.0" />
    <FileAttrib ID="ID_FILEATTRIB_NSCM_DRIVER_2" FriendlyName="" FileName="nscm.sys" MinimumFileVersion="0.0.0.0" MaximumFileVersion="3.1.12.0" />
    <FileAttrib ID="ID_FILEATTRIB_SANDRA_DRIVER_2" FriendlyName="" FileName="sandra.sys" MinimumFileVersion="0.0.0.0" MaximumFileVersion="10.12.0.0" />
    <FileAttrib ID="ID_FILEATTRIB_CPUZ_DRIVER_2" FriendlyName="" FileName="cpuz.sys" MinimumFileVersion="0.0.0.0" MaximumFileVersion="1.0.4.3" />
    <FileAttrib ID="ID_FILEATTRIB_ELBY_DRIVER_2" FriendlyName="" FileName="ElbyCDIO.sys" MinimumFileVersion="0.0.0.0" MaximumFileVersion="6.0.3.2" />
    <FileAttrib ID="ID_FILEATTRIB_RTKIO64_DRIVER_2" FriendlyName="" FileName="rtkio64.sys " MinimumFileVersion="65535.65535.65535.65535" />
    <FileAttrib ID="ID_FILEATTRIB_RTKIOW10X64_DRIVER_2" FriendlyName="" FileName="rtkiow10x64.sys " MinimumFileVersion="65535.65535.65535.65535" />
    <FileAttrib ID="ID_FILEATTRIB_RTKIOW8X64_DRIVER_2" FriendlyName="" FileName="rtkiow8x64.sys " MinimumFileVersion="65535.65535.65535.65535" />
    <FileAttrib ID="ID_FILEATTRIB_MTCBSV64_2" FriendlyName="mtcBSv64.sys FileAttribute" FileName="mtcBSv64.sys" MinimumFileVersion="0.0.0.0" MaximumFileVersion="21.2.0.0" />
    <FileAttrib ID="ID_FILEATTRIB_BS_HWMIO64_2" FriendlyName="" FileName="BS_HWMIO64_W10.sys" MinimumFileVersion="0.0.0.0" MaximumFileVersion="10.0.1806.2200" />
    <FileAttrib ID="ID_FILEATTRIB_BSMI_2" FriendlyName="" FileName="BSMI.sys" MinimumFileVersion="0.0.0.0" MaximumFileVersion="1.0.0.3" />
    <FileAttrib ID="ID_FILEATTRIB_BS_I2CIO_2" FriendlyName="" FileName="BS_I2cIo.sys" MinimumFileVersion="0.0.0.0" MaximumFileVersion="1.1.0.0" />
    <FileAttrib ID="ID_FILEATTRIB_NTIOLIB_2" FriendlyName="" FileName="NTIOLib.sys" MinimumFileVersion="0.0.0.0" MaximumFileVersion="1.0.0.0" />
    <FileAttrib ID="ID_FILEATTRIB_NCHGBIOS2X64_2" FriendlyName="" FileName="NCHGBIOS2x64.SYS" MinimumFileVersion="0.0.0.0" MaximumFileVersion="4.2.4.0" />
    <FileAttrib ID="ID_FILEATTRIB_SEGWINDRVX64_2" FriendlyName="segwindrvx64.sys FileAttribute" FileName="segwindrvx64.sys" MinimumFileVersion="0.0.0.0" MaximumFileVersion="100.0.7.2" />
    <Allow ID="ID_ALLOW_ALL_1_2" FriendlyName="" FileName="*" />
    <Deny ID="ID_DENY_BANDAI_SHA1_2" FriendlyName="bandai.sys Hash Sha1" Hash="0F780B7ADA5DD8464D9F2CC537D973F5AC804E9C" />
    <Deny ID="ID_DENY_BANDAI_SHA256_2" FriendlyName="bandai.sys Hash Sha256" Hash="7FD788358585E0B863328475898BB4400ED8D478466D1B7F5CC0252671456CC8" />
    <Deny ID="ID_DENY_BANDAI_SHA1_PAGE_2" FriendlyName="bandai.sys Hash Page Sha1" Hash="EA360A9F23BB7CF67F08B88E6A185A699F0C5410" />
    <Deny ID="ID_DENY_BANDAI_SHA256_PAGE_2" FriendlyName="bandai.sys Hash Page Sha256" Hash="BB83738210650E09307CE869ACA9BFA251024D3C47B1006B94FCE2846313F56E" />
    <Deny ID="ID_DENY_CAPCOM_SHA1_2" FriendlyName="capcom.sys Hash Sha1" Hash="1D1CAFC73C97C6BCD2331F8777D90FDCA57125A3" />
    <Deny ID="ID_DENY_CAPCOM_SHA256_2" FriendlyName="capcom.sys Hash Sha256" Hash="FAA08CB609A5B7BE6BFDB61F1E4A5E8ADF2F5A1D2492F262483DF7326934F5D4" />
    <Deny ID="ID_DENY_CAPCOM_SHA1_PAGE_2" FriendlyName="capcom.sys Hash Page Sha1" Hash="69006FBBD1B150FB9404867A5BCDC04FE0FC1BAD" />
    <Deny ID="ID_DENY_CAPCOM_SHA256_PAGE_2" FriendlyName="capcom.sys Hash Page Sha256" Hash="42589C7CE89941060465096C4661654B43E38C1F9D05D66239825E8FCCF52705" />
    <Deny ID="ID_DENY_FIDDRV_SHA1_2" FriendlyName="fiddrv.sys Hash Sha1" Hash="8CC8974A05E81678E3D28ACFE434E7804ABD019C" />
    <Deny ID="ID_DENY_FIDDRV_SHA256_2" FriendlyName="fiddrv.sys Hash Sha256" Hash="97B976F7E7E5DF7AF0781BBBB33CB5F3F7A59EFDD07995253B31DE8123352A67" />
    <Deny ID="ID_DENY_FIDDRV_SHA1_PAGE_2" FriendlyName="fiddrv.sys Hash Page Sha1" Hash="282BB241BDA5C4C1B8EB9BF56D018896649CA0E1" />
    <Deny ID="ID_DENY_FIDDRV_SHA256_PAGE_2" FriendlyName="fiddrv.sys Hash Page Sha256" Hash="1ED9DA2DA2539284404E0701E6BA3C9EB37BE10353E826F425A194D247B8B7CE" />
    <Deny ID="ID_DENY_FIDDRV64_SHA1_2" FriendlyName="fiddrv64.sys Hash Sha1" Hash="10E15BA8FF8ED926DDD3636CEC66A0F08C9860A4" />
    <Deny ID="ID_DENY_FIDDRV64_SHA256_2" FriendlyName="fiddrv64.sys Hash Sha256" Hash="FEEF191064D18B6FB63B7299415D1B1E2EC8FCDD742854AA96268D0EC4A0F7B6" />
    <Deny ID="ID_DENY_FIDDRV64_SHA1_PAGE_2" FriendlyName="fiddrv64.sys Hash Page Sha1" Hash="E4436C8C42BA5FFABD58A3B2256F6E86CCC907AB" />
    <Deny ID="ID_DENY_FIDDRV64_SHA256_PAGE_2" FriendlyName="fiddrv64.sys Hash Page Sha256" Hash="2D48414647A7F9DEA30F19074EBF8F17E55E9031B8604794CEB88369C8C52532" />
    <Deny ID="ID_DENY_FIDPCIDRV_SHA1_2" FriendlyName="fidpcidrv.sys Hash Sha1" Hash="08596732304351B311970FF96B21F451F23B1E25" />
    <Deny ID="ID_DENY_FIDPCIDRV_SHA256_2" FriendlyName="fidpcidrv.sys Hash Sha256" Hash="7B7E0E1453E733050B586A6FAC91883DBB85AE0775C84C4CEB967CFC9B4EFD10" />
    <Deny ID="ID_DENY_FIDPCIDRV_SHA1_PAGE_2" FriendlyName="fidpcidrv.sys Hash Page Sha1" Hash="7838FB56FDAB816BC1900A4720EEA2FC9972EF7A" />
    <Deny ID="ID_DENY_FIDPCIDRV_SHA256_PAGE_2" FriendlyName="fidpcidrv.sys Hash Page Sha256" Hash="0893E186E236315FE78A7EF41ED71617E75D90D2D14FE93911E0D9344BEAF69F" />
    <Deny ID="ID_DENY_FIDPCIDRV64_SHA1_2" FriendlyName="fidpcidrv64.sys Hash Sha1" Hash="4789B910023A667BEE70FF1F1A8F369CFFB10FE8" />
    <Deny ID="ID_DENY_FIDPCIDRV64_SHA256_2" FriendlyName="fidpcidrv64.sys Hash Sha256" Hash="7FB0F6FC5BDD22D53F8532CB19DA666A77A66FFB1CF3919A2E22B66C13B415B7" />
    <Deny ID="ID_DENY_FIDPCIDRV64_SHA1_PAGE_2" FriendlyName="fidpcidrv64.sys Hash Page Sha1" Hash="EEFF4EC4EBC12C6ACD2C930DC2EAAF877CFEC7EC" />
    <Deny ID="ID_DENY_FIDPCIDRV64_SHA256_PAGE_2" FriendlyName="fidpcidrv64.sys Hash Page Sha256" Hash="B98E008DFEA10EC74C89D08F12F31C12F52234BE6FFFF06B6B9E749BFEA6CBED" />
    <Deny ID="ID_DENY_GDRV_2" FriendlyName="gdrv.sys" FileName="gdrv.sys" />
    <Deny ID="ID_DENY_GLCKIO2_SHA1_2" FriendlyName="GLCKIO2.sys Hash Sha1" Hash="D99B80B3269D735CAC43AF5E43483E64CA7961C3" />
    <Deny ID="ID_DENY_GLCKIO2_SHA256_2" FriendlyName="GLCKIO2.sys Hash Sha256" Hash="47DBA240967FD0088BE618163672DFBDDF0138178CCCD45B54037F622B221220" />
    <Deny ID="ID_DENY_GLCKIO2_SHA1_PAGE_2" FriendlyName="GLCKIO2.sys Hash Page Sha1" Hash="51E0740AAEE5AE76B0095C92908C97B817DB8BEA" />
    <Deny ID="ID_DENY_GLCKIO2_SHA256_PAGE_2" FriendlyName="GLCKIO2.sys Hash Page Sha256" Hash="E7F011E9857C7DB5AACBD424612CD7E3D12C363FDC8F072DDFAF9E2E5C85F5F3" />
    <Deny ID="ID_DENY_GVCIDRV64_SHA1_2" FriendlyName="GVCIDrv64.sys Hash Sha1" Hash="4EAE38E9DC262EB7B6EDE4B3D3F4AD068933845E" />
    <Deny ID="ID_DENY_GVCIDRV64_SHA256_2" FriendlyName="GVCIDrv64.sys Hash Sha256" Hash="2FF09BB919A9909068166C30322C4E904BEFEBA5429E9A11D011297FB8A73C07" />
    <Deny ID="ID_DENY_GVCIDRV64_SHA1_PAGE_2" FriendlyName="GVCIDrv64.sys Hash Page Sha1" Hash="6980122AEF4E2D5D7A6DDDB6DA76A166C460E0A1" />
    <Deny ID="ID_DENY_GVCIDRV64_SHA256_PAGE_2" FriendlyName="GVCIDrv64.sys Hash Page Sha256" Hash="A69247025DD32DC15E06FEE362B494BCC6105D34B8D7091F7EC3D9000BD71501" />
    <Deny ID="ID_DENY_WINFLASH64_SHA1_2" FriendlyName="WinFlash64.sys Hash Sha1" Hash="DA21F5889F8374C3961856D681ADEC3D663D2964" />
    <Deny ID="ID_DENY_WINFLASH64_SHA256_2" FriendlyName="WinFlash64.sys Hash Sha256" Hash="F2B51FBEEAD17F5EE34D5B4A3A83C848FB76F8F0E80769212E137A7AA539A3BC" />
    <Deny ID="ID_DENY_WINFLASH64_SHA1_PAGE_2" FriendlyName="WinFlash64.sys Hash Page Sha1" Hash="C5057A4FD3C9B58F4C9AB9FE356081DF8804BF98" />
    <Deny ID="ID_DENY_WINFLASH64_SHA256_PAGE_2" FriendlyName="WinFlash64.sys Hash Page Sha256" Hash="C8FA1EC3D03050FBC1AA677F2C0348690521291219E8D2E94F0EA9E9174B9156" />
    <Deny ID="ID_DENY_AMIFLDRV64_SHA1_2" FriendlyName="amifldrv64.sys Hash Sha1" Hash="B0EC7D971DA8AE84C0ED8F88A5D46B23996E636C" />
    <Deny ID="ID_DENY_AMIFLDRV64_SHA256C_2" FriendlyName="amifldrv64.sys Hash Sha256" Hash="038F39558035292F1D794B7CF49F8E751E8633DAEC31454FE85CCCBEA83BA3FB" />
    <Deny ID="ID_DENY_AMIFLDRV64_SHA1_PAGE_2" FriendlyName="amifldrv64.sys Hash Page Sha1" Hash="C9CC3779ED67755220DBF9592EC2AC0E1DE363DC" />
    <Deny ID="ID_DENY_AMIFLDRV64_SHA256_PAGE_2" FriendlyName="amifldrv64.sys Hash Page Sha256" Hash="AA594D977312A944B14351C075634E7C59B42687928FBCDA8E2C4CEA46686DD9" />
    <Deny ID="ID_DENY_ASUPIO64_SHA1F_2" FriendlyName="AsUpIO64.sys Hash Sha1" Hash="2A95F882DD9BAFCC57F144A2708A7EC67DD7844C" />
    <Deny ID="ID_DENY_ASUPIO64_SHA256_2" FriendlyName="AsUpIO64.sys Hash Sha256" Hash="7F75D91844B0C162EEB24D14BCF63B7F230E111DAA7B0A26EAA489EEB22D9057" />
    <Deny ID="ID_DENY_ASUPIO64_SHA1_PAGE_2" FriendlyName="AsUpIO64.sys Hash Page Sha1" Hash="316E7872A227F0EAD483D244805E9FF4D3569F6F" />
    <Deny ID="ID_DENY_ASUPIO64_SHA256_PAGE_2" FriendlyName="AsUpIO64.sys Hash Page Sha256" Hash="5958CBE6CF7170C4B66893777BDE66343F5536A98610BD188E10D47DB84BC04C" />
    <Deny ID="ID_DENY_BSFLASH64_SHA1_2" FriendlyName="BS_Flash64.sys Hash Sha1" Hash="5107438A02164E1BCEDD556A786F37F59CD04231" />
    <Deny ID="ID_DENY_BSFLASH64_SHA256_2" FriendlyName="BS_Flash64.sys Hash Sha256" Hash="543C3F024E4AFFD0AAFA3A229FA19DBE7A70972BB18ED6347D3492DD174EDAC5" />
    <Deny ID="ID_DENY_BSFLASH64_SHA1_PAGE_2" FriendlyName="BS_Flash64.sys Hash Page Sha1" Hash="26C398B86FD33B3E6C4348F780C4CF758C99C8FD" />
    <Deny ID="ID_DENY_BSFLASH64_SHA256_PAGE_2" FriendlyName="BS_Flash64.sys Hash Page Sha256" Hash="8BF958AFA751D7AB66EBB1FAE25679E6F0FDE72078AEFC09F1824EEFA526005E" />
    <Deny ID="ID_DENY_BSHWMIO64_SHA1_2" FriendlyName="BS_HWMIo64.sys Hash Sha1" Hash="3281135748C9C7A9DDACE55C648C720AF810475F" />
    <Deny ID="ID_DENY_BSHWMIO64_SHA256_2" FriendlyName="BS_HWMIo64.sys Hash Sha256" Hash="3DE51A3102DB7297D96B4DE5B60ACA5F3A07E8577BBBED7F755F1DE9A9C38E75" />
    <Deny ID="ID_DENY_BSHWMIO64_SHA1_PAGE_2" FriendlyName="BS_HWMIo64.sys Hash Page Sha1" Hash="FC5F231383FE72E298893010A9A3714B205C4110" />
    <Deny ID="ID_DENY_BSHWMIO64_SHA256_PAGE_2" FriendlyName="BS_HWMIo64.sys Hash Page Sha256" Hash="6AD3624CA1DC38ECEEC75234E50934B1BAD7C72621DC57DEAB09044D0135877D" />
    <Deny ID="ID_DENY_MSIO64_SHA1_2" FriendlyName="MsIo64.sys Hash Sha1" Hash="7E732ACB7CFAD9BA043A9350CDEFF25D742BECB8" />
    <Deny ID="ID_DENY_MSIO64_SHA256_2" FriendlyName="MsIo64.sys Hash Sha256" Hash="7018D515A6C781EA6097CA71D0F0603AD0D689F7EC99DB27FCACD492A9E86027" />
    <Deny ID="ID_DENY_MSIO64_SHA1_PAGE_2" FriendlyName="MsIo64.sys Hash Page Sha1" Hash="CDE1A50E1DF7870F8E4AFD8631E45A847C714C0A" />
    <Deny ID="ID_DENY_MSIO64_SHA256_PAGE_2" FriendlyName="MsIo64.sys Hash Page Sha256" Hash="05736AB8B48DF84D81CB2CC0FBDC9D3DA34C22DB67A3E71C6F4B6B3923740DD5" />
    <Deny ID="ID_DENY_PIDDRV_SHA1_2" FriendlyName="piddrv.sys Hash Sha1" Hash="877C6C36A155109888FE1F9797B93CB30B4957EF" />
    <Deny ID="ID_DENY_PIDDRV_SHA256_2" FriendlyName="piddrv.sys Hash Sha256" Hash="4E19D4CE649C28DD947424483796BEACE3656284FB0379D97DDDD320AA602BBC" />
    <Deny ID="ID_DENY_PIDDRV_SHA1_PAGE_2" FriendlyName="piddrv.sys Hash Page Sha1" Hash="A7D827A41B2C4B7638495CD1D77926F1BA902978" />
    <Deny ID="ID_DENY_PIDDRV_SHA256_PAGE_2" FriendlyName="piddrv.sys Hash Page Sha256" Hash="EAC7316089DBAF7DF79A531355547BBDA22FA0921E31BBA0D27BCC88234E9ED3" />
    <Deny ID="ID_DENY_PIDDRV64_SHA1_2" FriendlyName="piddrv64.sys Hash Sha1" Hash="0C2599D738D01A82EC91725F499ACEBBCFB47CC9" />
    <Deny ID="ID_DENY_PIDDRV64_SHA256_2" FriendlyName="piddrv64.sys Hash Sha256" Hash="B97F870C501714FA453CF18AE8A30C87D08FF1E6D784AFDBB0121AEA3DA2DC28" />
    <Deny ID="ID_DENY_PIDDRV64_SHA1_PAGE_2" FriendlyName="piddrv64.sys Hash Page Sha1" Hash="C978063E678233C5EFB8F002FEF000FD479CC632" />
    <Deny ID="ID_DENY_PIDDRV64_SHA256_PAGE_2" FriendlyName="piddrv64.sys Hash Page Sha256" Hash="1081CCD57FD35998634103AE1E736638D82351092ACD30FE75084EA6A08CA0F7" />
    <Deny ID="ID_DENY_SEMAV6MSR64_SHA1_2" FriendlyName="semav6msr64.sys Hash Sha1" Hash="E3DBE2AA03847DF621591A4CAD69A5609DE5C237" />
    <Deny ID="ID_DENY_SEMAV6MSR64_SHA256_2" FriendlyName="semav6msr64.sys Hash Sha256" Hash="EB71A8ECEF692E74AE356E8CB734029B233185EE5C2CCB6CC87CC6B36BEA65CF" />
    <Deny ID="ID_DENY_SEMAV6MSR64_SHA1_PAGE_2" FriendlyName="semav6msr64.sys Hash Page Sha1" Hash="F3821EC0AEF270F749DF9F44FBA91AFA5C8C38E8" />
    <Deny ID="ID_DENY_SEMAV6MSR64_SHA256_PAGE_2" FriendlyName="semav6msr64.sys Hash Page Sha256" Hash="4F12EE563E7496E7105D67BF64AF6B436902BE4332033AF0B5A242B206372CB7" />
    <Allow ID="ID_ALLOW_ALL_2_2" FriendlyName="" FileName="*" />
  </FileRules>
  <!--Signers-->
  <Signers>
    <Signer ID="ID_SIGNER_MICROSOFT_PRODUCT_1997_0" Name="MincryptKnownRootMicrosoftProductRoot1997">
      <CertRoot Type="Wellknown" Value="04" />
    </Signer>
    <Signer ID="ID_SIGNER_MICROSOFT_PRODUCT_2001_0" Name="MincryptKnownRootMicrosoftProductRoot2001">
      <CertRoot Type="Wellknown" Value="05" />
    </Signer>
    <Signer ID="ID_SIGNER_MICROSOFT_PRODUCT_2010_0" Name="MincryptKnownRootMicrosoftProductRoot2010">
      <CertRoot Type="Wellknown" Value="06" />
    </Signer>
    <Signer ID="ID_SIGNER_MICROSOFT_STANDARD_2011_0" Name="MincryptKnownRootMicrosoftStandardRoot2011">
      <CertRoot Type="Wellknown" Value="07" />
    </Signer>
    <Signer ID="ID_SIGNER_MICROSOFT_CODEVERIFICATION_2006_0" Name="MincryptKnownRootMicrosoftCodeVerificationRoot2006">
      <CertRoot Type="Wellknown" Value="08" />
    </Signer>
    <Signer ID="ID_SIGNER_DRM_0" Name="MincryptKnownRootMicrosoftDMDRoot2005">
      <CertRoot Type="Wellknown" Value="0C" />
    </Signer>
    <Signer ID="ID_SIGNER_MICROSOFT_FLIGHT_2014_0" Name="MincryptKnownRootMicrosoftFlightRoot2014">
      <CertRoot Type="Wellknown" Value="0E" />
    </Signer>
    <Signer ID="ID_SIGNER_TEST2010_0" Name="MincryptKnownRootMicrosoftTestRoot2010">
      <CertRoot Type="Wellknown" Value="0A" />
    </Signer>
    <Signer ID="ID_SIGNER_MICROSOFT_PRODUCT_1997_UMCI_0" Name="MincryptKnownRootMicrosoftProductRoot1997">
      <CertRoot Type="Wellknown" Value="04" />
    </Signer>
    <Signer ID="ID_SIGNER_MICROSOFT_PRODUCT_2001_UMCI_0" Name="MincryptKnownRootMicrosoftProductRoot2001">
      <CertRoot Type="Wellknown" Value="05" />
    </Signer>
    <Signer ID="ID_SIGNER_MICROSOFT_PRODUCT_2010_UMCI_0" Name="MincryptKnownRootMicrosoftProductRoot2010">
      <CertRoot Type="Wellknown" Value="06" />
    </Signer>
    <Signer ID="ID_SIGNER_MICROSOFT_STANDARD_2011_UMCI_0" Name="MincryptKnownRootMicrosoftStandardRoot2011">
      <CertRoot Type="Wellknown" Value="07" />
    </Signer>
    <Signer ID="ID_SIGNER_MICROSOFT_CODEVERIFICATION_2006_UMCI_0" Name="MincryptKnownRootMicrosoftCodeVerificationRoot2006">
      <CertRoot Type="Wellknown" Value="08" />
    </Signer>
    <Signer ID="ID_SIGNER_DRM_UMCI_0" Name="MincryptKnownRootMicrosoftDMDRoot2005">
      <CertRoot Type="Wellknown" Value="0C" />
    </Signer>
    <Signer ID="ID_SIGNER_MICROSOFT_FLIGHT_2014_UMCI_0" Name="MincryptKnownRootMicrosoftFlightRoot2014">
      <CertRoot Type="Wellknown" Value="0E" />
    </Signer>
    <Signer ID="ID_SIGNER_STORE_0" Name="Microsoft MarketPlace PCA 2011">
      <CertRoot Type="TBS" Value="FC9EDE3DCCA09186B2D3BF9B738A2050CB1A554DA2DCADB55F3F72EE17721378" />
      <CertEKU ID="ID_EKU_STORE" />
    </Signer>
    <Signer ID="ID_SIGNER_TEST2010_UMCI_0" Name="MincryptKnownRootMicrosoftTestRoot2010">
      <CertRoot Type="Wellknown" Value="0A" />
    </Signer>
    <Signer ID="ID_SIGNER_WINDOWS_PRODUCTION_0_0_2_1" Name="Microsoft Product Root 2010 Windows EKU">
      <CertRoot Type="Wellknown" Value="06" />
      <CertEKU ID="ID_EKU_WINDOWS" />
    </Signer>
    <Signer ID="ID_SIGNER_ELAM_PRODUCTION_0_0_2_1" Name="Microsoft Product Root 2010 ELAM EKU">
      <CertRoot Type="Wellknown" Value="06" />
      <CertEKU ID="ID_EKU_ELAM" />
    </Signer>
    <Signer ID="ID_SIGNER_HAL_PRODUCTION_0_0_2_1" Name="Microsoft Product Root 2010 HAL EKU">
      <CertRoot Type="Wellknown" Value="06" />
      <CertEKU ID="ID_EKU_HAL_EXT" />
    </Signer>
    <Signer ID="ID_SIGNER_WHQL_SHA2_0_0_2_1" Name="Microsoft Product Root 2010 WHQL EKU">
      <CertRoot Type="Wellknown" Value="06" />
      <CertEKU ID="ID_EKU_WHQL" />
    </Signer>
    <Signer ID="ID_SIGNER_WHQL_SHA1_0_0_2_1" Name="Microsoft Product Root WHQL EKU SHA1">
      <CertRoot Type="Wellknown" Value="05" />
      <CertEKU ID="ID_EKU_WHQL" />
    </Signer>
    <Signer ID="ID_SIGNER_WHQL_MD5_0_0_2_1" Name="Microsoft Product Root WHQL EKU MD5">
      <CertRoot Type="Wellknown" Value="04" />
      <CertEKU ID="ID_EKU_WHQL" />
    </Signer>
    <Signer ID="ID_SIGNER_WINDOWS_FLIGHT_ROOT_0_0_2_1" Name="Microsoft Flighting Root 2014 Windows EKU">
      <CertRoot Type="Wellknown" Value="0E" />
      <CertEKU ID="ID_EKU_WINDOWS" />
    </Signer>
    <Signer ID="ID_SIGNER_ELAM_FLIGHT_0_0_2_1" Name="Microsoft Flighting Root 2014 ELAM EKU">
      <CertRoot Type="Wellknown" Value="0E" />
      <CertEKU ID="ID_EKU_ELAM" />
    </Signer>
    <Signer ID="ID_SIGNER_HAL_FLIGHT_0_0_2_1" Name="Microsoft Flighting Root 2014 HAL EKU">
      <CertRoot Type="Wellknown" Value="0E" />
      <CertEKU ID="ID_EKU_HAL_EXT" />
    </Signer>
    <Signer ID="ID_SIGNER_WHQL_FLIGHT_SHA2_0_0_2_1" Name="Microsoft Flighting Root 2014 WHQL EKU">
      <CertRoot Type="Wellknown" Value="0E" />
      <CertEKU ID="ID_EKU_WHQL" />
    </Signer>
    <Signer ID="ID_SIGNER_F_1_1_0_1" Name="DigiCert EV Code Signing CA (SHA2)">
      <CertRoot Type="TBS" Value="EEC58131DC11CD7F512501B15FDBC6074C603B68CA91F7162D5A042054EDB0CF" />
      <CertPublisher Value="Adobe Inc." />
      <FileAttribRef RuleID="ID_FILEATTRIB_F_5_1_0_1" />
    </Signer>
    <Signer ID="ID_SIGNER_AUTHROOT_0_0_2_1" Name="Authroot Dummy WellKnown Value">
      <CertRoot Type="Wellknown" Value="14" />
    </Signer>
    <Signer ID="ID_SIGNER_S_18C_1_2_0_2_1" Name="SecureSign RootCA2">
      <CertRoot Type="TBS" Value="648DBE77ABD3178D82EEAE00D2AB02C3F39C3605" />
    </Signer>
    <Signer ID="ID_SIGNER_S_18D_1_2_0_2_1" Name="Hellenic Academic and Research Institutions RootCA 2015">
      <CertRoot Type="TBS" Value="08F31221356AFBF68BDC93A1F74FB377D6473D8FAAB7BF405B4709719082266F" />
    </Signer>
    <Signer ID="ID_SIGNER_S_18E_1_2_0_2_1" Name="Microsoft Time Stamp Root Certificate Authority 2014">
      <CertRoot Type="TBS" Value="E4A2F6FE9CA7F18A2BEBA96161308BAA8880B013161DDD8532D4259E27E50570" />
    </Signer>
    <Signer ID="ID_SIGNER_S_18F_1_2_0_2_1" Name="NetLock Minositett Kozjegyzoi (Class QA) Tanusitvanykiado">
      <CertRoot Type="TBS" Value="46B853FC5247E913FF84FC233C0484F0E065107B" />
    </Signer>
    <Signer ID="ID_SIGNER_S_190_1_2_0_2_1" Name="KISA RootCA 1">
      <CertRoot Type="TBS" Value="50102F4FD88E98430B17C94FDD13C35FAEF550EE" />
    </Signer>
    <Signer ID="ID_SIGNER_S_191_1_2_0_2_1" Name="AddTrust External CA Root">
      <CertRoot Type="TBS" Value="09B9105C5BBA24343CA7F341C624E183F6EE7C1B" />
    </Signer>
    <Signer ID="ID_SIGNER_S_192_1_2_0_2_1" Name="GeoTrust Primary Certification Authority - G3">
      <CertRoot Type="TBS" Value="C135754AD3671E5EBA095F7A1A2E00D8D4012BDF4B3653CA6CEFE3C4F3C09B22" />
    </Signer>
    <Signer ID="ID_SIGNER_S_193_1_2_0_2_1" Name="Halcom CA FO">
      <CertRoot Type="TBS" Value="63DB5900B1066727D731D35D7375C2189841E38D" />
    </Signer>
    <Signer ID="ID_SIGNER_S_194_1_2_0_2_1" Name="O=ACNLB, C=SI">
      <CertRoot Type="TBS" Value="62E76B5FC689D75EC27F690A55691934058ABD98" />
    </Signer>
    <Signer ID="ID_SIGNER_S_195_1_2_0_2_1" Name="UTN-USERFirst-Hardware">
      <CertRoot Type="TBS" Value="89B5351EC11451D06E2F95B5F89722D527A897B9" />
    </Signer>
    <Signer ID="ID_SIGNER_S_196_1_2_0_2_1" Name="Certipost E-Trust TOP Root CA">
      <CertRoot Type="TBS" Value="2CA327DF4CD34154076DF97F6C0F894DAA43D724" />
    </Signer>
    <Signer ID="ID_SIGNER_S_197_1_2_0_2_1" Name="DigiCert Assured ID Root CA">
      <CertRoot Type="TBS" Value="6DCA5BD00DCF1C0F327059D374B29CA6E3C50AA6" />
    </Signer>
    <Signer ID="ID_SIGNER_S_198_1_2_0_2_1" Name="NetLock Arany (Class Gold) Főtanúsítvány">
      <CertRoot Type="TBS" Value="5B61C31792D86EA5B286E1E68F552CF8D07629F8712E2C470CEB2349AE698297" />
    </Signer>
    <Signer ID="ID_SIGNER_S_199_1_2_0_2_1" Name="Macao Post eSignTrust Root Certification Authority (G02)">
      <CertRoot Type="TBS" Value="AEB9FB5712D7A84CA429A5EC5258EEBD798DE019" />
    </Signer>
    <Signer ID="ID_SIGNER_S_19A_1_2_0_2_1" Name="Microsoft ECC Product Root Certificate Authority 2018">
      <CertRoot Type="TBS" Value="32991981BF1575A1A5303BB93A381723EA346B9EC130FDB596A75BA1D7CE0B0A06570BB985D25841E23BE944E8FF118F" />
    </Signer>
    <Signer ID="ID_SIGNER_S_19B_1_2_0_2_1" Name="Certum Trusted Network CA">
      <CertRoot Type="TBS" Value="A8569CCD21EF9CC5737C7A12DF608C2CBC545DF1" />
    </Signer>
    <Signer ID="ID_SIGNER_S_19C_1_2_0_2_1" Name="VAS Latvijas Pasts SSI(RCA)">
      <CertRoot Type="TBS" Value="A669E06A306A09C81E3F49E4A4B55FFD988C5D77" />
    </Signer>
    <Signer ID="ID_SIGNER_S_19D_1_2_0_2_1" Name="QuoVadis Root CA 2 G3">
      <CertRoot Type="TBS" Value="BC9C578712090A1C04397CA4A528D202B145B32D9A9FD76743F632A6636ABAAF" />
    </Signer>
    <Signer ID="ID_SIGNER_S_19E_1_2_0_2_1" Name="Netrust Root CA 2">
      <CertRoot Type="TBS" Value="355E8880A37089FAD04C948F7533BD1891274BD5D8C9F267629AACCD9A4F9D2B" />
    </Signer>
    <Signer ID="ID_SIGNER_S_19F_1_2_0_2_1" Name="SwissSign Gold Root CA - G3">
      <CertRoot Type="TBS" Value="0E861345FBFD676439E639CBF8B4C0B42649FE2120D43101280EBA4C2685ACF7" />
    </Signer>
    <Signer ID="ID_SIGNER_S_1A0_1_2_0_2_1" Name="UCA Global Root">
      <CertRoot Type="TBS" Value="B20FEA7B11777661FC7E32BD525CE1E01B040FC9" />
    </Signer>
    <Signer ID="ID_SIGNER_S_1A1_1_2_0_2_1" Name="SSC GDL CA Root A">
      <CertRoot Type="TBS" Value="9BDD84BD6E82921D6343F83C2873A4158ADEC11FDAD35822E45AB4874A8D2CA0" />
    </Signer>
    <Signer ID="ID_SIGNER_S_1A2_1_2_0_2_1" Name="ANCERT Corporaciones de Derecho Publico">
      <CertRoot Type="TBS" Value="B45A4BF86006A177F34E208B789F0AE37AE0F41C" />
    </Signer>
    <Signer ID="ID_SIGNER_S_1A3_1_2_0_2_1" Name="Amazon Root CA 3">
      <CertRoot Type="TBS" Value="6D29DBED0025D7540E14E4110AEFA547C48FC75C85E2180B6038F18E126CB74F" />
    </Signer>
    <Signer ID="ID_SIGNER_S_1A4_1_2_0_2_1" Name="GDCA TrustAUTH R5 ROOT">
      <CertRoot Type="TBS" Value="D6008F4627A91D824182456930A573ECA8F8D46FA62EF9B62DB9ADC6475CB4B8" />
    </Signer>
    <Signer ID="ID_SIGNER_S_1A5_1_2_0_2_1" Name="OISTE WISeKey Global Root GB CA">
      <CertRoot Type="TBS" Value="04524E82755B1E36393B942C01DEE51978C032D7D4519F7DA6C964ABF89C5EA9" />
    </Signer>
    <Signer ID="ID_SIGNER_S_1A6_1_2_0_2_1" Name="Staat der Nederlanden Root CA">
      <CertRoot Type="TBS" Value="3C54CF8D129C47DB965858447396F3105762DBC1" />
    </Signer>
    <Signer ID="ID_SIGNER_S_1A7_1_2_0_2_1" Name="GLOBAL CHAMBERSIGN ROOT - 2016">
      <CertRoot Type="TBS" Value="1AB6CBCEF365C714E07B3F5DAAEC5A05503C2CEEFB58AE786689B139C37A8442" />
    </Signer>
    <Signer ID="ID_SIGNER_S_1A8_1_2_0_2_1" Name="ANCERT Certificados CGN">
      <CertRoot Type="TBS" Value="5E05CA07CA332F64B4D6779E3A17A2A89942BE95" />
    </Signer>
    <Signer ID="ID_SIGNER_S_1A9_1_2_0_2_1" Name="VeriSign Class 3 Public Primary Certification Authority - G3">
      <CertRoot Type="TBS" Value="DF243244279C8EB88633DAB7F89E9BE55C94492E" />
    </Signer>
    <Signer ID="ID_SIGNER_S_1AA_1_2_0_2_1" Name="Entrust Root Certification Authority - G4">
      <CertRoot Type="TBS" Value="812F815EEE3D160225EBDC544669E3C0DAA07CFFF503B0D1728A2134B094B2E9" />
    </Signer>
    <Signer ID="ID_SIGNER_S_1AB_1_2_0_2_1" Name="CA DATEV STD 01">
      <CertRoot Type="TBS" Value="9E3DFE15F6D922797B41F7B357B3080322EE46F5" />
    </Signer>
    <Signer ID="ID_SIGNER_S_1AC_1_2_0_2_1" Name="CA 沃通根证书">
      <CertRoot Type="TBS" Value="D5B12BB9975454057E6E629D9507A5A033EF1C11AF5E2A567DC3C939DC2057B1" />
    </Signer>
    <Signer ID="ID_SIGNER_S_1AD_1_2_0_2_1" Name="SITHS CA v3">
      <CertRoot Type="TBS" Value="04A0DEBF159B8ED5839978F37F8DE44DCCDE2335" />
    </Signer>
    <Signer ID="ID_SIGNER_S_1AE_1_2_0_2_1" Name="OU=&quot;NO LIABILITY ACCEPTED, (c)97 VeriSign, Inc.&quot;, OU=VeriSign Time Stamping Service Root, OU=&quot;VeriSign, Inc.&quot;, O=VeriSign Trust Network">
      <CertRoot Type="TBS" Value="65FC47520F66383962EC0B7B88A0821D" />
    </Signer>
    <Signer ID="ID_SIGNER_S_1AF_1_2_0_2_1" Name="IGC/A AC racine Etat francais">
      <CertRoot Type="TBS" Value="5E873CAF65994C56D4516C228671A26225F8F2B087B682FA56F5608A02ABE6C6" />
    </Signer>
    <Signer ID="ID_SIGNER_S_1B0_1_2_0_2_1" Name="S-TRUST Universal Root CA">
      <CertRoot Type="TBS" Value="F792A153F5588D4430B0B0A8A5CD567F6F73CAC01CA29B9EF419CAC230185F80" />
    </Signer>
    <Signer ID="ID_SIGNER_S_1B1_1_2_0_2_1" Name="TÜBİTAK UEKAE Kök Sertifika Hizmet Sağlayıcısı - Sürüm 3">
      <CertRoot Type="TBS" Value="E2F4764AB337EB1CCF45DB5C6970B3B716D5E5C9" />
    </Signer>
    <Signer ID="ID_SIGNER_S_1B2_1_2_0_2_1" Name="QuoVadis Root CA 1 G3">
      <CertRoot Type="TBS" Value="F982E236A51FAA61379BA4F357D2A938B9E1971EE78DE115CAC46E53CDDDFD0B" />
    </Signer>
    <Signer ID="ID_SIGNER_S_1B3_1_2_0_2_1" Name="SSL.com EV Root Certification Authority RSA">
      <CertRoot Type="TBS" Value="1CAF6E8C1A82D5C814626183F7C7688DF951181EBB26ADE00E53C879AD368CD8" />
    </Signer>
    <Signer ID="ID_SIGNER_S_1B4_1_2_0_2_1" Name="LuxTrust Global Root 2">
      <CertRoot Type="TBS" Value="CA090B0194970E75A8DB0A8636F7BE96AD83B46A03F0E5FAF2CF35854A82B760" />
    </Signer>
    <Signer ID="ID_SIGNER_S_1B5_1_2_0_2_1" Name="emSign Root CA - G2">
      <CertRoot Type="TBS" Value="F2340A5C1F8B39F4228936DBB4F3F3DD0A464DD5E9D11AC13304C150C2E24486A0AC61EAF298B230867C16F158ED90E7" />
    </Signer>
    <Signer ID="ID_SIGNER_S_1B6_1_2_0_2_1" Name="GlobalSign">
      <CertRoot Type="TBS" Value="F019C7BA12795DACD6EF1A767657DEB8E41060A2AD1A6C66900A6EC16870FC06EECBAFFDD3E2C986659578E7E85E535A" />
    </Signer>
    <Signer ID="ID_SIGNER_S_1B7_1_2_0_2_1" Name="Notarius Root Certificate Authority">
      <CertRoot Type="TBS" Value="7D0967214719912471E2F36B16B251A226F7733AA2317615126B73E7D311C689" />
    </Signer>
    <Signer ID="ID_SIGNER_S_1B8_1_2_0_2_1" Name="QuoVadis Root CA 3">
      <CertRoot Type="TBS" Value="93C45418230486C6BC515DD0CDC552AC78C36B64" />
    </Signer>
    <Signer ID="ID_SIGNER_S_1B9_1_2_0_2_1" Name="VeriSign Class 1 Public Primary Certification Authority - G3">
      <CertRoot Type="TBS" Value="6AD6CBA0916142FC5F3C64F342C18D2CCFC66EC4" />
    </Signer>
    <Signer ID="ID_SIGNER_S_1BA_1_2_0_2_1" Name="SAPO Class 4 Root CA">
      <CertRoot Type="TBS" Value="D417D171DAB0CDEFAFC9BDA6526404E04E12FD4E" />
    </Signer>
    <Signer ID="ID_SIGNER_S_1BB_1_2_0_2_1" Name="Entrust Root Certification Authority - EC1">
      <CertRoot Type="TBS" Value="CAA26D95667D69D7E73DBE7989B34A1CE4E218495E2522B1362E6150575F984102A4B12F59DA094252AC2DC857C070D2" />
    </Signer>
    <Signer ID="ID_SIGNER_S_1BC_1_2_0_2_1" Name="Registradores de España - CA Raíz">
      <CertRoot Type="TBS" Value="AD658A9D9A3E497ECC30000FB83B34B0527764A8" />
    </Signer>
    <Signer ID="ID_SIGNER_S_1BD_1_2_0_2_1" Name="Class 3P Primary CA">
      <CertRoot Type="TBS" Value="4B9F28DDC3513CF1AA4CBBA1A18C5DBFE3D0D593" />
    </Signer>
    <Signer ID="ID_SIGNER_S_1BE_1_2_0_2_1" Name="Application CA G4 Root">
      <CertRoot Type="TBS" Value="84FB8B115BDA655F4B70708FCB8CACC4D7DC04AAE8965B506D871598A8ADCA2D" />
    </Signer>
    <Signer ID="ID_SIGNER_S_1BF_1_2_0_2_1" Name="VeriSign Class 3 Public Primary Certification Authority - G4">
      <CertRoot Type="TBS" Value="28AC5AD930278CBD276EED75214DBA04EC8B1E19E63E30324FA1BDA0E9E83BF8E0786EB8792BE4A75B20A693B9F621EF" />
    </Signer>
    <Signer ID="ID_SIGNER_S_1C0_1_2_0_2_1" Name="Certplus Root CA G1">
      <CertRoot Type="TBS" Value="69BA57C3D88EE48557638992FE45439C84154E8798BAAACD7CE08E31D02629F3E727000A64C081BE9051B6FE30A860018C5DE51F1A4A465DBB92C1133E3699AF" />
    </Signer>
    <Signer ID="ID_SIGNER_S_1C1_1_2_0_2_1" Name="Microsec e-Szigno Root CA">
      <CertRoot Type="TBS" Value="E1AED0AB1F0A6C26B10CE19C4144CA8E7A3A857F" />
    </Signer>
    <Signer ID="ID_SIGNER_S_1C2_1_2_0_2_1" Name="Halcom Root Certificate Authority">
      <CertRoot Type="TBS" Value="1E8DC5CE2C0841080D9C09A919494B26D67B24C6D7FD3A28E2E683DC1D50F372" />
    </Signer>
    <Signer ID="ID_SIGNER_S_1C3_1_2_0_2_1" Name="Thawte Server CA">
      <CertRoot Type="TBS" Value="1015676C3B5DEDEC330183A43E1FCCA2" />
    </Signer>
    <Signer ID="ID_SIGNER_S_1C4_1_2_0_2_1" Name="SSC Root CA C">
      <CertRoot Type="TBS" Value="7AA4D6C39DF6DAD545DC03596E3399531995781B" />
    </Signer>
    <Signer ID="ID_SIGNER_S_1C6_1_2_0_2_1" Name="Admin-Root-CA">
      <CertRoot Type="TBS" Value="9E4B504421B07EDE743FD2E37DFC9565BD47B568" />
    </Signer>
    <Signer ID="ID_SIGNER_S_1C7_1_2_0_2_1" Name="Symantec Class 3 Public Primary Certification Authority - G6">
      <CertRoot Type="TBS" Value="139537D16C2BE16BDB77B9D3C8838C473741F095A93F2A4DCD829051704FCB8AF256EA95928DE2D214DA8DB68278948F" />
    </Signer>
    <Signer ID="ID_SIGNER_S_1C8_1_2_0_2_1" Name="OU=certSIGN ROOT CA G2, O=CERTSIGN SA, C=RO">
      <CertRoot Type="TBS" Value="0B286A252F3C55FFA750C15F0F109C875854382DE678AB674FCEDDE9D2A13445" />
    </Signer>
    <Signer ID="ID_SIGNER_S_1C9_1_2_0_2_1" Name="OU=Go Daddy Class 2 Certification Authority, O=&quot;The Go Daddy Group, Inc.&quot;, C=US">
      <CertRoot Type="TBS" Value="5D82ADB90D5DD3C7E3524F56F787EC5372618776" />
    </Signer>
    <Signer ID="ID_SIGNER_S_1CA_1_2_0_2_1" Name="CA DATEV STD 03">
      <CertRoot Type="TBS" Value="90EA27425E745A6E2AB5EE41DAC08231CB180EDE" />
    </Signer>
    <Signer ID="ID_SIGNER_S_1CB_1_2_0_2_1" Name="EC-ACC">
      <CertRoot Type="TBS" Value="1B8B713E8748912A4B073DB0C8E9E3E5C0962D98" />
    </Signer>
    <Signer ID="ID_SIGNER_S_1CC_1_2_0_2_1" Name="UCA Global G2 Root">
      <CertRoot Type="TBS" Value="8D7782C727E5BB96DF1EF260DE5D0558916B47EBE45423F2D7D624DF4751205C" />
    </Signer>
    <Signer ID="ID_SIGNER_S_1CD_1_2_0_2_1" Name="AffirmTrust Networking">
      <CertRoot Type="TBS" Value="2110A6E8DA67CEE9D90CCBF913117C60EC31C914" />
    </Signer>
    <Signer ID="ID_SIGNER_S_1CE_1_2_0_2_1" Name="GTS Root R4">
      <CertRoot Type="TBS" Value="DF03EE17776FAE07203AE956F6094206455C833A06297419E38793A34C4E010E8E0DD06107E0CD574F970FB35FB7C04E" />
    </Signer>
    <Signer ID="ID_SIGNER_S_1CF_1_2_0_2_1" Name="CA Disig">
      <CertRoot Type="TBS" Value="8DE82C8B78D72F2D4E46D162F96A212C1903269F" />
    </Signer>
    <Signer ID="ID_SIGNER_S_1D0_1_2_0_2_1" Name="USERTrust RSA Certification Authority">
      <CertRoot Type="TBS" Value="66B764A96581128168CF208E374DDA479D54E311F32457F4AEE0DBD2A6C8D171D531289E1CD22BFDBBD4CFD979625483" />
    </Signer>
    <Signer ID="ID_SIGNER_S_1D1_1_2_0_2_1" Name="Atos TrustedRoot 2011">
      <CertRoot Type="TBS" Value="B81BC142AB6A29EAF7A0FB26C1A362AAB3631D0F684F803B649BAA176A6E65BD" />
    </Signer>
    <Signer ID="ID_SIGNER_S_1D2_1_2_0_2_1" Name="Cisco RXC-R2">
      <CertRoot Type="TBS" Value="50A98E6D324323B9DB9BAB390EB7266EFE490583EBDB29E86251F49BAB6D81C1" />
    </Signer>
    <Signer ID="ID_SIGNER_S_1D3_1_2_0_2_1" Name="Certigna Root CA">
      <CertRoot Type="TBS" Value="9668D6C44B5F62EE4A56423640D93D45A2C772C6D42ED178978AF5ADDB15FDAE" />
    </Signer>
    <Signer ID="ID_SIGNER_S_1D4_1_2_0_2_1" Name="CHAMBERS OF COMMERCE ROOT - 2016">
      <CertRoot Type="TBS" Value="C2CAFDB9E75DDEF680CC07675B09DA57628D1889AEFD4139EEA0D6A560B41F5A" />
    </Signer>
    <Signer ID="ID_SIGNER_S_1D5_1_2_0_2_1" Name="Certinomis - Autorité Racine">
      <CertRoot Type="TBS" Value="A2C028101D5D53DDA69EA4CBA4103E45D2A40C5E" />
    </Signer>
    <Signer ID="ID_SIGNER_S_1D6_1_2_0_2_1" Name="A-Trust-Root-05">
      <CertRoot Type="TBS" Value="852B800D36C31C7F38A1FF1EF55C32E38F8EA56C94E2BB6A561843B74B872390" />
    </Signer>
    <Signer ID="ID_SIGNER_S_1D7_1_2_0_2_1" Name="Trustwave Global Certification Authority">
      <CertRoot Type="TBS" Value="53F5B139376A52678853BEB4B5841E155E864D2EA83BB089519D95FF0266F4B3" />
    </Signer>
    <Signer ID="ID_SIGNER_S_1D8_1_2_0_2_1" Name="emSign ECC Root CA - G3">
      <CertRoot Type="TBS" Value="E466C65CCF00EB61B38A4EDAFD8098C0B50D0CDDEB6206BB3E28AECC20CDF02078F39DF0A612E728D3BEF467E8323ED2" />
    </Signer>
    <Signer ID="ID_SIGNER_S_1D9_1_2_0_2_1" Name="Izenpe.com">
      <CertRoot Type="TBS" Value="A16D1FAA61F7277CD9ABC31C9C893ED7B41EFB19" />
    </Signer>
    <Signer ID="ID_SIGNER_S_1DA_1_2_0_2_1" Name="GTS Root R3">
      <CertRoot Type="TBS" Value="A17CEA0667930C32F4D668AEE1AF63A67A46B74D3DE8965AE147E7FD56A9B946038347B354491F4E51B90FD60916DE1D" />
    </Signer>
    <Signer ID="ID_SIGNER_S_1DB_1_2_0_2_1" Name="PosDigicert Class 2 Root CA G2">
      <CertRoot Type="TBS" Value="4C095B520D4F33A9277C9038C729DFA356729A5C7AAD05C75F98BF2CDAB38222B9DEE0343FDBE4A162F423EDA9E9EE63BE1B98EBCBEE5D12BD01FFFCA25BA93F" />
    </Signer>
    <Signer ID="ID_SIGNER_S_1DC_1_2_0_2_1" Name="TUBITAK Kamu SM SSL Kok Sertifikasi - Surum 1">
      <CertRoot Type="TBS" Value="ADCFF054AD813E9C69C3D02CDEC2C8B6ADBD41BC8AFB22C16901D3807111DA4E" />
    </Signer>
    <Signer ID="ID_SIGNER_S_1DD_1_2_0_2_1" Name="StartCom Certification Authority G2">
      <CertRoot Type="TBS" Value="BE918DAD689ACE92CD42F3BF9C463343C2EAF23A8D2F5A7356C72E48E279BD52" />
    </Signer>
    <Signer ID="ID_SIGNER_S_1DE_1_2_0_2_1" Name="Microsoft ECC TS Root Certificate Authority 2018">
      <CertRoot Type="TBS" Value="03D1C76765EDA88BC8E0875E6091D060432543D180BCB86C064936ADB941C42163780B8289921A94FEBB7F9E47EDAC12" />
    </Signer>
    <Signer ID="ID_SIGNER_S_1DF_1_2_0_2_1" Name="GeoTrust Primary Certification Authority">
      <CertRoot Type="TBS" Value="6C0485E5A990E6E0A0C7D325312AB03362763A54" />
    </Signer>
    <Signer ID="ID_SIGNER_S_1E0_1_2_0_2_1" Name="Swedish Government Root Authority v2">
      <CertRoot Type="TBS" Value="B08B83F04ED9EBF8C472375F80B81544E655534A4775AB1C06297A01477AFE89" />
    </Signer>
    <Signer ID="ID_SIGNER_S_1E1_1_2_0_2_1" Name="LAWtrust Root Certification Authority 2048">
      <CertRoot Type="TBS" Value="3FB61ADC6AFCF927BE15930221AB596449478CB7" />
    </Signer>
    <Signer ID="ID_SIGNER_S_1E2_1_2_0_2_1" Name="Global Chambersign Root">
      <CertRoot Type="TBS" Value="B02F56CCEC49800699C31FA897A9E0EAC24F2890" />
    </Signer>
    <Signer ID="ID_SIGNER_S_1E3_1_2_0_2_1" Name="GLOBALTRUST">
      <CertRoot Type="TBS" Value="5EE282A798FC9B8E81395D845C85E3188CC38B91" />
    </Signer>
    <Signer ID="ID_SIGNER_S_1E4_1_2_0_2_1" Name="Autoridad Certificadora Raiz de la Secretaria de Economia">
      <CertRoot Type="TBS" Value="9A92DE6FBF1A70BE211189361D24CA183DDC15A1" />
    </Signer>
    <Signer ID="ID_SIGNER_S_1E5_1_2_0_2_1" Name="VeriSign Universal Root Certification Authority">
      <CertRoot Type="TBS" Value="17FE16F394EC70A5BB0C6784CAB40B1E61025AE9D50ECAA0531D6B4D997BBC59" />
    </Signer>
    <Signer ID="ID_SIGNER_S_1E6_1_2_0_2_1" Name="OU=Security Communication RootCA1, O=SECOM Trust.net, C=JP">
      <CertRoot Type="TBS" Value="9A680178CFA26912AB0DFA84D27D892AA0235881" />
    </Signer>
    <Signer ID="ID_SIGNER_S_1E7_1_2_0_2_1" Name="ZETES TSP ROOT CA 001">
      <CertRoot Type="TBS" Value="4D7F5ED9CDB2142EF6F0633C2487824092A267DFD5203232D148EDA8EFDE4D8D" />
    </Signer>
    <Signer ID="ID_SIGNER_S_1E8_1_2_0_2_1" Name="GeoTrust Universal CA 2">
      <CertRoot Type="TBS" Value="052F8BA40B64117FAAEB1589D7FCCA213E2ED851" />
    </Signer>
    <Signer ID="ID_SIGNER_S_1E9_1_2_0_2_1" Name="Sonera Class2 CA">
      <CertRoot Type="TBS" Value="C9B272311F5EF70451235B16F49983ECA12ED2B3" />
    </Signer>
    <Signer ID="ID_SIGNER_S_1EA_1_2_0_2_1" Name="SAPO Class 3 Root CA">
      <CertRoot Type="TBS" Value="91A46A6A731BE490C0C44AB462BC50F9ECFA7D8D" />
    </Signer>
    <Signer ID="ID_SIGNER_S_1EB_1_2_0_2_1" Name="ECRaizEstado">
      <CertRoot Type="TBS" Value="DD5705DF22DD3FE2A68A8C8DA89743C7C0280542" />
    </Signer>
    <Signer ID="ID_SIGNER_S_1EC_1_2_0_2_1" Name="America Online Root Certification Authority 1">
      <CertRoot Type="TBS" Value="B31DA18FBA765C5ED3DF3080593871DB9E0DCC29" />
    </Signer>
    <Signer ID="ID_SIGNER_S_1ED_1_2_0_2_1" Name="CA DATEV BT 02">
      <CertRoot Type="TBS" Value="E45CF2AC438BEE26EFA260F44F8872F907C049F7" />
    </Signer>
    <Signer ID="ID_SIGNER_S_1EE_1_2_0_2_1" Name="Autoridad de Certificacion Raiz del Estado Venezolano">
      <CertRoot Type="TBS" Value="1CFB7757E07253C1FF9C950EFBC48592528E8FADBC9D41DFC93DB3EBEDC65859921E0C042CD812FAD94D1A6B9CD87DD5" />
    </Signer>
    <Signer ID="ID_SIGNER_S_1EF_1_2_0_2_1" Name="Secure Global CA">
      <CertRoot Type="TBS" Value="BDCB3FC55040776BF7A9460CFFB252A13CF1989F" />
    </Signer>
    <Signer ID="ID_SIGNER_S_1F0_1_2_0_2_1" Name="SI-TRUST Root">
      <CertRoot Type="TBS" Value="84C1EC23D7B65005DA6428FD70F5F7A151F5525D2E7F4A9D58F7F91CE760741F" />
    </Signer>
    <Signer ID="ID_SIGNER_S_1F1_1_2_0_2_1" Name="Microsoft EV RSA Root Certificate Authority 2017">
      <CertRoot Type="TBS" Value="0B94EC93356997EC26556D14594A239CD79E1DC03D74CFCBA30DB0FF8BE4C9EB7CC0A69BEF3EB2FD274939571C24CD3E" />
    </Signer>
    <Signer ID="ID_SIGNER_S_1F2_1_2_0_2_1" Name="Microsoft Root Certificate Authority 2010">
      <CertRoot Type="TBS" Value="08FBA831C08544208F5208686B991CA1B2CFC510E7301784DDF1EB5BF0393239" />
    </Signer>
    <Signer ID="ID_SIGNER_S_1F3_1_2_0_2_1" Name="OU=Trustis FPS Root CA, O=Trustis Limited, C=GB">
      <CertRoot Type="TBS" Value="EDE0EF707AFAD0055DADDF528A402C78D68F716F" />
    </Signer>
    <Signer ID="ID_SIGNER_S_1F4_1_2_0_2_1" Name="SecureSign RootCA11">
      <CertRoot Type="TBS" Value="C7CD1252642EE271A0AC60F8AAAC886558012B49" />
    </Signer>
    <Signer ID="ID_SIGNER_S_1F5_1_2_0_2_1" Name="CCA India 2015 SPL">
      <CertRoot Type="TBS" Value="B778B86E7A68A8F8467AA66E569B9E09C709D5580F06A4846EA7C0A61981EAAC" />
    </Signer>
    <Signer ID="ID_SIGNER_S_1F6_1_2_0_2_1" Name="CA DATEV BT 03">
      <CertRoot Type="TBS" Value="ED46C3E70D171CA459F19E407C06E711AFC4A74E" />
    </Signer>
    <Signer ID="ID_SIGNER_S_1F7_1_2_0_2_1" Name="StartCom Certification Authority">
      <CertRoot Type="TBS" Value="84E608DD4CC47C78E2DE0F831405996C467FC35D" />
    </Signer>
    <Signer ID="ID_SIGNER_S_1F8_1_2_0_2_1" Name="OU=sigen-ca, O=state-institutions, C=si">
      <CertRoot Type="TBS" Value="34E0FB3431584524E1E81AEE7CE15ECF9D2F4799" />
    </Signer>
    <Signer ID="ID_SIGNER_S_1F9_1_2_0_2_1" Name="SSC Root CA B">
      <CertRoot Type="TBS" Value="4736D9ED7A737F12989D009B15E7E6FCEC3153F4" />
    </Signer>
    <Signer ID="ID_SIGNER_S_1FA_1_2_0_2_1" Name="BYTE Root Certification Authority 001">
      <CertRoot Type="TBS" Value="AB026BD5E6127A6925DA084F948A86BE43A0245881BB2477BF9585A033534D13" />
    </Signer>
    <Signer ID="ID_SIGNER_S_1FB_1_2_0_2_1" Name="Juur-SK">
      <CertRoot Type="TBS" Value="2317BA5922D1774E7782247DFC9FA79D7383D511" />
    </Signer>
    <Signer ID="ID_SIGNER_S_1FC_1_2_0_2_1" Name="Symantec Class 2 Public Primary Certification Authority - G6">
      <CertRoot Type="TBS" Value="15B9C784C3881D85CCB9E50C313D629BBE68C1952C5439749600242633EA4400" />
    </Signer>
    <Signer ID="ID_SIGNER_S_1FD_1_2_0_2_1" Name="A-Trust-Qual-03">
      <CertRoot Type="TBS" Value="17C360168C5470A0C2FD8E621B3E04FB5B0C51B2" />
    </Signer>
    <Signer ID="ID_SIGNER_S_1FE_1_2_0_2_1" Name="TeliaSonera Root CA v1">
      <CertRoot Type="TBS" Value="6D705B892247D3BD3EEC85A6632A78376C4445D4" />
    </Signer>
    <Signer ID="ID_SIGNER_S_1FF_1_2_0_2_1" Name="PersonalID Trustworthy RootCA 2011">
      <CertRoot Type="TBS" Value="E734C178E26C69FC2A524066B3192AE076199B81A020D41131F22C0809D4284E" />
    </Signer>
    <Signer ID="ID_SIGNER_S_200_1_2_0_2_1" Name="OU=FNMT Clase 2 CA, O=FNMT, C=ES">
      <CertRoot Type="TBS" Value="5DB2E9A3DA334BE97DD5F0BD3DD83D17515B0ADE" />
    </Signer>
    <Signer ID="ID_SIGNER_S_201_1_2_0_2_1" Name="GLOBALTRUST 2015">
      <CertRoot Type="TBS" Value="F28EC8C3F8878B6D2CD11CEDF8327EAE56C6BD78F515C8ED28271EC073E9D008" />
    </Signer>
    <Signer ID="ID_SIGNER_S_202_1_2_0_2_1" Name="MULTICERT Root Certification Authority 01">
      <CertRoot Type="TBS" Value="9D1B3E2667103237A1B2248D012A359E52EBDE1B06759B6F9719BB171255B573" />
    </Signer>
    <Signer ID="ID_SIGNER_S_203_1_2_0_2_1" Name="Go Daddy Root Certificate Authority - G2">
      <CertRoot Type="TBS" Value="3560E45B41E46B8F36537025D1D5BC02D9652A10645B0EFF69E8B6A52191F335" />
    </Signer>
    <Signer ID="ID_SIGNER_S_204_1_2_0_2_1" Name="QuoVadis Root CA 3 G3">
      <CertRoot Type="TBS" Value="D09AACD3381F4D4FF6F99F9D2D7BFB522B5B969A2D573348EE029A01DF64E316" />
    </Signer>
    <Signer ID="ID_SIGNER_S_205_1_2_0_2_1" Name="Buypass Class 2 Root CA">
      <CertRoot Type="TBS" Value="56DB6D3C33811B6420936A9B42F80EABDB96C6F17C128EBEC63E70DB0B6CB77D" />
    </Signer>
    <Signer ID="ID_SIGNER_S_206_1_2_0_2_1" Name="D-TRUST Root Class 2 CA 2007">
      <CertRoot Type="TBS" Value="AF0B816B34511D678AE0C37749E13522EB606AB5" />
    </Signer>
    <Signer ID="ID_SIGNER_S_207_1_2_0_2_1" Name="Izenpe.com">
      <CertRoot Type="TBS" Value="CCA62043CB686EC7520381F684FB544AEB3A79BF" />
    </Signer>
    <Signer ID="ID_SIGNER_S_208_1_2_0_2_1" Name="Global Chambersign Root - 2008">
      <CertRoot Type="TBS" Value="6036808A80BCFF19F1AECC97FBAAF2FF618CBA82" />
    </Signer>
    <Signer ID="ID_SIGNER_S_209_1_2_0_2_1" Name="Autoridade Certificadora Raiz Brasileira v5">
      <CertRoot Type="TBS" Value="15CCEC4C310FFC5B8BE9167546E70649C51BECEF601755815C6EBF46B03144553E1FCA0C6183136462BC985ADED3A30C2AA4520EDDB3708A85741749795DEB61" />
    </Signer>
    <Signer ID="ID_SIGNER_S_20A_1_2_0_2_1" Name="OATI WebCARES Root CA">
      <CertRoot Type="TBS" Value="D018B5A72EF8C81B338C34AFF170AB30CD1401B0" />
    </Signer>
    <Signer ID="ID_SIGNER_S_20B_1_2_0_2_1" Name="A-Trust-nQual-03">
      <CertRoot Type="TBS" Value="C473CC52C457092D939B5A73BE349C487782714B" />
    </Signer>
    <Signer ID="ID_SIGNER_S_20C_1_2_0_2_1" Name="SSL.com EV Root Certification Authority ECC">
      <CertRoot Type="TBS" Value="2B3CBCDEB5AFCC4D44496BD72B688B5789EDFCF6C49B33FEE10C5572D64E212A" />
    </Signer>
    <Signer ID="ID_SIGNER_S_20D_1_2_0_2_1" Name="VeriSign Class 3 Public Primary Certification Authority - G5">
      <CertRoot Type="TBS" Value="E91E1E972B8F467AB4E0598FA92285387DEE94C9" />
    </Signer>
    <Signer ID="ID_SIGNER_S_20E_1_2_0_2_1" Name="Certplus Root CA G2">
      <CertRoot Type="TBS" Value="8CDA597FF092F3BFDEB8C9D11419D7E9E6CC243EA9704E47590CD7AE9AC226E65D1999138168857C3F5833B7A6495425" />
    </Signer>
    <Signer ID="ID_SIGNER_S_20F_1_2_0_2_1" Name="China Internet Network Information Center EV Certificates Root">
      <CertRoot Type="TBS" Value="4BD93261E9BF30D7B0C4D9DCC50C328E4C6282AF" />
    </Signer>
    <Signer ID="ID_SIGNER_S_210_1_2_0_2_1" Name="Entrust.net Certification Authority (2048)">
      <CertRoot Type="TBS" Value="327FC447408DE9BF596F83D4B2FA4B8E3E7097D8" />
    </Signer>
    <Signer ID="ID_SIGNER_S_211_1_2_0_2_1" Name="Hotspot 2.0 Trust Root CA - 03">
      <CertRoot Type="TBS" Value="BE52E461B17DD6252771251B45E98F1232CAA12512DC79118D0C5FCE73A54D95" />
    </Signer>
    <Signer ID="ID_SIGNER_S_212_1_2_0_2_1" Name="Symantec Class 1 Public Primary Certification Authority - G6">
      <CertRoot Type="TBS" Value="8BC0D6FAAF266CC1D2125552BA8825FBF0B053E82F9B1BE7295BCE0EC7EDD056" />
    </Signer>
    <Signer ID="ID_SIGNER_S_213_1_2_0_2_1" Name="E-Tugra Certification Authority">
      <CertRoot Type="TBS" Value="AE284D570FF1601F3D9E2067F8B5D44E58B49D5142A2D888235926E44B49A1EB" />
    </Signer>
    <Signer ID="ID_SIGNER_S_214_1_2_0_2_1" Name="CA DATEV INT 01">
      <CertRoot Type="TBS" Value="915845D5B4205B663F2A03E7FA9FC0E0CA0885C1" />
    </Signer>
    <Signer ID="ID_SIGNER_S_215_1_2_0_2_1" Name="Halcom Root CA">
      <CertRoot Type="TBS" Value="3A787B25C0D197B712867FDB44F923CEF02295C0647A647857205A735F52B2A038C6B0A11D3ADAB86988CD285B87B51BAF03358C88FC4FFF03137DCB3AD25DC5" />
    </Signer>
    <Signer ID="ID_SIGNER_S_216_1_2_0_2_1" Name="AC Raíz Certicámara S.A.">
      <CertRoot Type="TBS" Value="ACFF6E0191EDDA67BEF1B1D069926F0B0EE7FF0B91AC5429786C6ADA065535CB" />
    </Signer>
    <Signer ID="ID_SIGNER_S_217_1_2_0_2_1" Name="CAEDICOM Root">
      <CertRoot Type="TBS" Value="F2BD37A2F2CB1E5AFF2C289901C2933941AE9B95227B87710DC5BB7A33129DAB" />
    </Signer>
    <Signer ID="ID_SIGNER_S_218_1_2_0_2_1" Name="T-TeleSec GlobalRoot Class 3">
      <CertRoot Type="TBS" Value="6DB8909F229EEED57BF28DADF024C2B27A554913CDE84B514B46FD532714DB5D" />
    </Signer>
    <Signer ID="ID_SIGNER_S_219_1_2_0_2_1" Name="OU=Netrust CA1, O=Netrust Certificate Authority 1, C=SG">
      <CertRoot Type="TBS" Value="2A5E5E85FA395EB3191E099F9C1D0B5D2C17F339" />
    </Signer>
    <Signer ID="ID_SIGNER_S_21A_1_2_0_2_1" Name="SwissSign Platinum CA - G2">
      <CertRoot Type="TBS" Value="4DEE3796A33CABE8A7D969E3ED7E7D15E16D4AF7" />
    </Signer>
    <Signer ID="ID_SIGNER_S_21B_1_2_0_2_1" Name="SITHS Root CA v1">
      <CertRoot Type="TBS" Value="9FA32E105FFD66CD861E896EB643B705073618CE" />
    </Signer>
    <Signer ID="ID_SIGNER_S_21C_1_2_0_2_1" Name="Hongkong Post Root CA 3">
      <CertRoot Type="TBS" Value="D6ED17A5F51972C262E2D3A8677577857C6A85700A2D22E0A4F87948D6834F63" />
    </Signer>
    <Signer ID="ID_SIGNER_S_21D_1_2_0_2_1" Name="TrustCor ECA-1">
      <CertRoot Type="TBS" Value="0295C62AC6347BD1271BAD1CF73CB2C65F119AA54DD8760CAF900D9CC991A4EA" />
    </Signer>
    <Signer ID="ID_SIGNER_S_21E_1_2_0_2_1" Name="Symantec Class 3 Public Primary Certification Authority - G4">
      <CertRoot Type="TBS" Value="7C76E562326C019EBBA09BE6B57CEAB86AE5C08B5578F892B635C46A16770FDE686CE2BFAD84545BBE1AFA9A989A5733" />
    </Signer>
    <Signer ID="ID_SIGNER_S_21F_1_2_0_2_1" Name="D-TRUST Root Class 3 CA 2 2009">
      <CertRoot Type="TBS" Value="09793E87B5D73EFDDFAFA2600A3E107A138F2D821B18868E61535CD1435FBBC1" />
    </Signer>
    <Signer ID="ID_SIGNER_S_220_1_2_0_2_1" Name="T-TeleSec GlobalRoot Class 2">
      <CertRoot Type="TBS" Value="568B40EB6F006B46507DA6367D046B6B3E317C83414595E786E48407F4126F3E" />
    </Signer>
    <Signer ID="ID_SIGNER_S_221_1_2_0_2_1" Name="OISTE WISeKey Global Root GA CA">
      <CertRoot Type="TBS" Value="FDBAD1AFBCBB45744A7DEC7B998BD6266412640C" />
    </Signer>
    <Signer ID="ID_SIGNER_S_222_1_2_0_2_1" Name="Staat der Nederlanden Root CA - G2">
      <CertRoot Type="TBS" Value="16541BB1EE4F9A8F7694BC8CECB786B3B200BC21A9740A20D64F58B87046E7C2" />
    </Signer>
    <Signer ID="ID_SIGNER_S_223_1_2_0_2_1" Name="Visa Information Delivery Root CA">
      <CertRoot Type="TBS" Value="7F61704A69779E272D209AC1D4D1BF9E43AEBBB2" />
    </Signer>
    <Signer ID="ID_SIGNER_S_224_1_2_0_2_1" Name="SSC Root CA A">
      <CertRoot Type="TBS" Value="3AAD417EA7C0E93CD0FC830BF416CFA61985498A" />
    </Signer>
    <Signer ID="ID_SIGNER_S_225_1_2_0_2_1" Name="Amazon Root CA 2">
      <CertRoot Type="TBS" Value="D746A5BF1663A495FB88BBE77DBCE6A325C994A696299331EF4C5AFA26C00970BACDD3D3B49DB055B6582B5D1A54B7AF" />
    </Signer>
    <Signer ID="ID_SIGNER_S_226_1_2_0_2_1" Name="ANF Global Root CA">
      <CertRoot Type="TBS" Value="A1A2056F4CBCFFF4537CEB98D04AD5C578E318E4" />
    </Signer>
    <Signer ID="ID_SIGNER_S_227_1_2_0_2_1" Name="TRUST2408 OCES Primary CA">
      <CertRoot Type="TBS" Value="C56609BC78CAD27B7ABA102132DAEFECACE344FA8E249880CB354BCCB6CFABFE" />
    </Signer>
    <Signer ID="ID_SIGNER_S_228_1_2_0_2_1" Name="Starfield Services Root Certificate Authority">
      <CertRoot Type="TBS" Value="EF4B92510CF5214F96C19FE5DAC82FE416B1167B" />
    </Signer>
    <Signer ID="ID_SIGNER_S_229_1_2_0_2_1" Name="Swisscom Root CA 1">
      <CertRoot Type="TBS" Value="A80FB311312188FD9DEEDF37FB087479BD6B47C8" />
    </Signer>
    <Signer ID="ID_SIGNER_S_22A_1_2_0_2_1" Name="OU=Security Communication RootCA2, O=&quot;SECOM Trust Systems CO.,LTD.&quot;, C=JP">
      <CertRoot Type="TBS" Value="453ECC5C2C07CCC737ABCA4F06054723F20169FCE993F86657343DB97515C000" />
    </Signer>
    <Signer ID="ID_SIGNER_S_22B_1_2_0_2_1" Name="Cybertrust Global Root">
      <CertRoot Type="TBS" Value="6776D660D7996923FAB7139628F9BE199FB81197" />
    </Signer>
    <Signer ID="ID_SIGNER_S_22C_1_2_0_2_1" Name="DigiCert High Assurance EV Root CA">
      <CertRoot Type="TBS" Value="E35EF08D884F0A0ADE2F75E96301CE6230F213A8" />
    </Signer>
    <Signer ID="ID_SIGNER_S_22D_1_2_0_2_1" Name="IGC/A">
      <CertRoot Type="TBS" Value="A5188DD70D6AAA1E23CD334ECB5A3D912C70ED79" />
    </Signer>
    <Signer ID="ID_SIGNER_S_22E_1_2_0_2_1" Name="VeriSign Class 2 Public Primary Certification Authority - G3">
      <CertRoot Type="TBS" Value="1BFA4731FCBB41F57F97A7B7D2B5C24CFB759190" />
    </Signer>
    <Signer ID="ID_SIGNER_S_22F_1_2_0_2_1" Name="Fina Root CA">
      <CertRoot Type="TBS" Value="A5E3CBD4540D4193A585609ED2E1E497C6A8175A8DD402220C9974B76D932994" />
    </Signer>
    <Signer ID="ID_SIGNER_S_230_1_2_0_2_1" Name="Certum CA">
      <CertRoot Type="TBS" Value="1E427A3639CCE4C27E94B1777964CA289A722CAD" />
    </Signer>
    <Signer ID="ID_SIGNER_S_231_1_2_0_2_1" Name="Thawte Premium Server CA">
      <CertRoot Type="TBS" Value="5F3D1AA6F471A760663EB7EF254281EF" />
    </Signer>
    <Signer ID="ID_SIGNER_S_232_1_2_0_2_1" Name="I.CA - Qualified root certificate">
      <CertRoot Type="TBS" Value="83349E55D81C5EA2AA084845F04BCC1673242CAB" />
    </Signer>
    <Signer ID="ID_SIGNER_S_233_1_2_0_2_1" Name="Thailand National Root Certification Authority - G1">
      <CertRoot Type="TBS" Value="FE6CEEFD64122FB5E18728C22B4C9925BECF008B9FA84FB535E2CFFFF0EFF562D13FC0EED8F3B5F4B75DAA18C5F0793FD628411D869231EACD607065F39FB9AD" />
    </Signer>
    <Signer ID="ID_SIGNER_S_234_1_2_0_2_1" Name="Symantec Class 2 Public Primary Certification Authority - G4">
      <CertRoot Type="TBS" Value="1466611C6127F71CA624D38E0679ADF23E767E7EB6ED9DFE0B3AEA8194D55FA9F6C83CB3491C45908ACEDE8DB9585A6F" />
    </Signer>
    <Signer ID="ID_SIGNER_S_235_1_2_0_2_1" Name="OU=ePKI Root Certification Authority, O=&quot;Chunghwa Telecom Co., Ltd.&quot;, C=TW">
      <CertRoot Type="TBS" Value="E2D1E7E0391A13E13A9759961938A4FAAB8DEA65" />
    </Signer>
    <Signer ID="ID_SIGNER_S_236_1_2_0_2_1" Name="GlobalSign">
      <CertRoot Type="TBS" Value="2C01409799ECEB34F3892A4E50B9F27560DF1F40110626BAA1D302A7CF1266F0" />
    </Signer>
    <Signer ID="ID_SIGNER_S_237_1_2_0_2_1" Name="Class 1 Primary CA">
      <CertRoot Type="TBS" Value="844757B10EB29FC369DDC36074BF656354A415D4" />
    </Signer>
    <Signer ID="ID_SIGNER_S_238_1_2_0_2_1" Name="AC1 RAIZ MTIN">
      <CertRoot Type="TBS" Value="BCF718DE863199B9E44E4B973AE5573330105194" />
    </Signer>
    <Signer ID="ID_SIGNER_S_239_1_2_0_2_1" Name="AdminCA-CD-T01">
      <CertRoot Type="TBS" Value="7BBFBFDB889B0F0A5F155A227FF35801F1FC781A" />
    </Signer>
    <Signer ID="ID_SIGNER_S_23A_1_2_0_2_1" Name="D-TRUST Root CA 3 2013">
      <CertRoot Type="TBS" Value="E317B9DCA152A5DA6786DA2DF515318802C7B0A35FCD0300150F7BE78A5376E3" />
    </Signer>
    <Signer ID="ID_SIGNER_S_23B_1_2_0_2_1" Name="OpenTrust Root CA G3">
      <CertRoot Type="TBS" Value="9B61A95E91560681EDDB84E14DEE37FC294291B5F8DB9BCF7A07433BCBAF71A0DEBF695AB6525AE319D4784474F3E5BD" />
    </Signer>
    <Signer ID="ID_SIGNER_S_23C_1_2_0_2_1" Name="Chambers of Commerce Root">
      <CertRoot Type="TBS" Value="A2630B78C797830810A1F842F5312B7E93711BE1" />
    </Signer>
    <Signer ID="ID_SIGNER_S_23D_1_2_0_2_1" Name="Application CA G3 Root">
      <CertRoot Type="TBS" Value="AD67CAD8DEAD47B3858232BFB0914AD599786C2ED63B6F8867A033622FC87404" />
    </Signer>
    <Signer ID="ID_SIGNER_S_23E_1_2_0_2_1" Name="ANCERT Certificados Notariales V2">
      <CertRoot Type="TBS" Value="4B15989ECCDB465CD3129674A71B34EF46B37AC331FEBF98FFFE13999F1C95F8" />
    </Signer>
    <Signer ID="ID_SIGNER_S_23F_1_2_0_2_1" Name="Visa eCommerce Root">
      <CertRoot Type="TBS" Value="86BF41E1724E80DCFC53946F9BEB7803E0086659" />
    </Signer>
    <Signer ID="ID_SIGNER_S_240_1_2_0_2_1" Name="Autoridade Certificadora Raiz Brasileira v1">
      <CertRoot Type="TBS" Value="C08A9924E73226ADA01CA23A723296658F73B3D8" />
    </Signer>
    <Signer ID="ID_SIGNER_S_241_1_2_0_2_1" Name="Network Solutions Certificate Authority">
      <CertRoot Type="TBS" Value="0695B2EA03E9C9FC9D51EACF7FEDBE9392CCC012" />
    </Signer>
    <Signer ID="ID_SIGNER_S_242_1_2_0_2_1" Name="Class 2 Primary CA">
      <CertRoot Type="TBS" Value="83446093BF635DDBA22D70E25CFD1EB12BEB43B0" />
    </Signer>
    <Signer ID="ID_SIGNER_S_244_1_2_0_2_1" Name="Certipost E-Trust Primary Qualified CA">
      <CertRoot Type="TBS" Value="066BE0457F0C833814AFEF69F2945A7D7B41533B" />
    </Signer>
    <Signer ID="ID_SIGNER_S_245_1_2_0_2_1" Name="SSL.com EV Root Certification Authority RSA R2">
      <CertRoot Type="TBS" Value="CF1152C4BFBD96B6D381B61E0C66E7A74B1185518120396449ABE53AAADF640B" />
    </Signer>
    <Signer ID="ID_SIGNER_S_246_1_2_0_2_1" Name="Swedish Government Root Authority v3">
      <CertRoot Type="TBS" Value="CC71B065232D8A942E94369464A31525C7E961F0834E686F9F3F561D52038192" />
    </Signer>
    <Signer ID="ID_SIGNER_S_247_1_2_0_2_1" Name="GlobalSign">
      <CertRoot Type="TBS" Value="BF4D2C390BBF0AA3A2B7EA2DC751011BF5FD422E" />
    </Signer>
    <Signer ID="ID_SIGNER_S_248_1_2_0_2_1" Name="GPKIRootCA1">
      <CertRoot Type="TBS" Value="BB7369D7F66FEE4EDAEE4F95B7FACF13DE1DFFF59F54FCFE4BB4DCCBFFE818A3" />
    </Signer>
    <Signer ID="ID_SIGNER_S_249_1_2_0_2_1" Name="Staat der Nederlanden EV Root CA">
      <CertRoot Type="TBS" Value="ED29FF01D8B91513FF35BC90F379EB8774C01D3070A3A40D9C5114E18C72F315" />
    </Signer>
    <Signer ID="ID_SIGNER_S_24A_1_2_0_2_1" Name="Swisscom Root CA 2">
      <CertRoot Type="TBS" Value="9914C19BCEF248F5F1474AB39A7AF0717C28683443739F872C5DC2221C48B75D" />
    </Signer>
    <Signer ID="ID_SIGNER_S_24B_1_2_0_2_1" Name="Chambers of Commerce Root - 2008">
      <CertRoot Type="TBS" Value="2E32BF12551A50A4F43C45178F6B8F03022CCF15" />
    </Signer>
    <Signer ID="ID_SIGNER_S_24C_1_2_0_2_1" Name="OpenTrust Root CA G2">
      <CertRoot Type="TBS" Value="7B1C4FB63CA701248C7D417D5EAA14A9563D366F3E2CA76F5CCF02D88E3808B262EB819F5F45F2228AA402C650F46953EF278AD8DFE60C6D4EE51B68B93C58E4" />
    </Signer>
    <Signer ID="ID_SIGNER_S_24D_1_2_0_2_1" Name="OpenTrust Root CA G1">
      <CertRoot Type="TBS" Value="D4BB7F384F3706F0396D4F1941FEFC93F5F845005B97645672937373DA6FFC46" />
    </Signer>
    <Signer ID="ID_SIGNER_S_24E_1_2_0_2_1" Name="Autoridad Certificadora Raíz Nacional de Uruguay">
      <CertRoot Type="TBS" Value="BEA16705BC065CCFD88BF6D104411F070C6C6E28D4049DF81F2FDBC5F8977CE0" />
    </Signer>
    <Signer ID="ID_SIGNER_S_24F_1_2_0_2_1" Name="Microsoft ECC Root Certificate Authority 2017">
      <CertRoot Type="TBS" Value="65C745E97E3D1F6911FB89172C3A29BB283EBBC5538C8CCE1BB1A6E5BC254AC93810DE49AD96B918CEE21F024C7EF6BA" />
    </Signer>
    <Signer ID="ID_SIGNER_S_250_1_2_0_2_1" Name="DigiCert Global Root G3">
      <CertRoot Type="TBS" Value="82C80199397722B57AD473EA266B93D47FFC77FE07F09388345F20DAB6ADDD087672F988B4BBFD154C4B133C70C9ECFF" />
    </Signer>
    <Signer ID="ID_SIGNER_S_251_1_2_0_2_1" Name="Equifax Secure Global eBusiness CA-1">
      <CertRoot Type="TBS" Value="824BAE7C7CB3A15CE851A396760574A3" />
    </Signer>
    <Signer ID="ID_SIGNER_S_252_1_2_0_2_1" Name="ANCERT Certificados CGN V2">
      <CertRoot Type="TBS" Value="783ABC375A298BE5D950432D9F45CB33D39F67AC05D8A4819E9EB11AE8BEFFC9" />
    </Signer>
    <Signer ID="ID_SIGNER_S_253_1_2_0_2_1" Name="Autoridad de Certificacion de la Abogacia">
      <CertRoot Type="TBS" Value="D2677B195F105BAA77F96EA15096B3835608570F" />
    </Signer>
    <Signer ID="ID_SIGNER_S_254_1_2_0_2_1" Name="OU=ApplicationCA, O=Japanese Government, C=JP">
      <CertRoot Type="TBS" Value="4F93ED8D9BD4132787B380CA55CB029B84E12259" />
    </Signer>
    <Signer ID="ID_SIGNER_S_255_1_2_0_2_1" Name="OU=sigov-ca, O=state-institutions, C=si">
      <CertRoot Type="TBS" Value="E1507773865E0142009C2F71B55F41A65C3E1277" />
    </Signer>
    <Signer ID="ID_SIGNER_S_256_1_2_0_2_1" Name="Halcom CA PO 2">
      <CertRoot Type="TBS" Value="011DE67748C57F5F432CE1E3DE54EF5DE044345C" />
    </Signer>
    <Signer ID="ID_SIGNER_S_257_1_2_0_2_1" Name="TC TrustCenter Class 3 CA II">
      <CertRoot Type="TBS" Value="2BEEE4C82F7F140FE80EE5269DA60BA6D3D2C2AE" />
    </Signer>
    <Signer ID="ID_SIGNER_S_258_1_2_0_2_1" Name="GlobalSign">
      <CertRoot Type="TBS" Value="EA09C51D4C3A334CE4ACD2BC08C6A9BE352E334F45C4FCCFCAB63EDB9F82DC87D4BD2ED2FADAE11163FB954809984FF1" />
    </Signer>
    <Signer ID="ID_SIGNER_S_259_1_2_0_2_1" Name="ComSign Advanced Security CA">
      <CertRoot Type="TBS" Value="AD966B5E91D6B1DB36C146B55C16C1FBF72BC127" />
    </Signer>
    <Signer ID="ID_SIGNER_S_25A_1_2_0_2_1" Name="Network Solutions ECC Certificate Authority">
      <CertRoot Type="TBS" Value="24871A42CB63ED0CC642CA54AFC18BD4604F27615D9D843D5520EFB2E499084717AD04E1FC4BE6CC14CC71EFD8888F70" />
    </Signer>
    <Signer ID="ID_SIGNER_S_25B_1_2_0_2_1" Name="ePKI EV SSL Certification Authority - G1">
      <CertRoot Type="TBS" Value="EBD5DA3CC4DE2B88F7205D0BD9324EE5E280FC350DB38C12C76FB5C087302323" />
    </Signer>
    <Signer ID="ID_SIGNER_S_25C_1_2_0_2_1" Name="UCA Root">
      <CertRoot Type="TBS" Value="3AF0AB038DB60A82DC39CA93DAB69D9610C82007" />
    </Signer>
    <Signer ID="ID_SIGNER_S_25D_1_2_0_2_1" Name="OU=Saudi National Root CA, O=National Center for Digital Certification, C=SA">
      <CertRoot Type="TBS" Value="B7AE4BD0F154501E54EA75D2161A7D5A1AD29DDCB82AFD3BBF3628FBD96766EC" />
    </Signer>
    <Signer ID="ID_SIGNER_S_25E_1_2_0_2_1" Name="ADOCA02">
      <CertRoot Type="TBS" Value="8E86A675CA65F6943CEFD26A395D4B9CCCD35DA2" />
    </Signer>
    <Signer ID="ID_SIGNER_S_25F_1_2_0_2_1" Name="Symantec Class 1 Public Primary Certification Authority - G4">
      <CertRoot Type="TBS" Value="701FB48F98FC1287D2C93066D72F28290D85FDD7BD356DECC800721F981C04433503B31276ABD53A1ADE353DA0C3A004" />
    </Signer>
    <Signer ID="ID_SIGNER_S_260_1_2_0_2_1" Name="OU=VeriSign Trust Network, OU=&quot;(c) 1998 VeriSign, Inc. - For authorized use only&quot;, OU=Class 3 Public Primary Certification Authority - G2, O=&quot;VeriSign, Inc.&quot;, C=US">
      <CertRoot Type="TBS" Value="D95944F5BD92127092218F9F02C719C42386B499" />
    </Signer>
    <Signer ID="ID_SIGNER_S_261_1_2_0_2_1" Name="Deutsche Telekom Root CA 2">
      <CertRoot Type="TBS" Value="FF99B1116ECA7B69F516900DEA2D12202453B511" />
    </Signer>
    <Signer ID="ID_SIGNER_S_262_1_2_0_2_1" Name="SecureTrust CA">
      <CertRoot Type="TBS" Value="31D254C62674C351D6E6212F6E53175AADE3175C" />
    </Signer>
    <Signer ID="ID_SIGNER_S_263_1_2_0_2_1" Name="ATHEX Root CA G2">
      <CertRoot Type="TBS" Value="6E8D5E45F0B7674A6D2665E79A06000B3A018BA6ED5FACBE2E637A9CE1706F4FA092EEFDCE86CC43DFA16E38400C6499" />
    </Signer>
    <Signer ID="ID_SIGNER_S_264_1_2_0_2_1" Name="Microsec e-Szigno Root CA 2009">
      <CertRoot Type="TBS" Value="FDD54FC45D67057BF0B0207C68C0E4E03F6CC052E52007BA2C01CF01C5AB8E50" />
    </Signer>
    <Signer ID="ID_SIGNER_S_265_1_2_0_2_1" Name="TÜRKTRUST Elektronik Sertifika Hizmet Sağlayıcısı H6">
      <CertRoot Type="TBS" Value="4D888C963311646262C40425E3A28E37C9EFFCCA6D0A4B4CC103935A1D4DAF21" />
    </Signer>
    <Signer ID="ID_SIGNER_S_266_1_2_0_2_1" Name="emSign Root CA - G1">
      <CertRoot Type="TBS" Value="B175FD701FE8281F2A7F64D8CAE89EB4C2AAEEA3C9993EE095A1B8237EC07A6E" />
    </Signer>
    <Signer ID="ID_SIGNER_S_267_1_2_0_2_1" Name="CNNIC ROOT">
      <CertRoot Type="TBS" Value="531CA532A3AC4B15FCF156612D003AB94141553F" />
    </Signer>
    <Signer ID="ID_SIGNER_S_268_1_2_0_2_1" Name="EBG Elektronik Sertifika Hizmet Sağlayıcısı">
      <CertRoot Type="TBS" Value="8D6DCA2B955DDE143E9DFDBC0BD2EA6E507B683E" />
    </Signer>
    <Signer ID="ID_SIGNER_S_269_1_2_0_2_1" Name="Entrust Root Certification Authority - G2">
      <CertRoot Type="TBS" Value="FDE5F2D9CE2026E1E10064C0A468C9F355B90ACF85BAF5CE6F52D4016837FD94" />
    </Signer>
    <Signer ID="ID_SIGNER_S_26A_1_2_0_2_1" Name="SwissSign Silver Root CA - G3">
      <CertRoot Type="TBS" Value="0917AB8B127C8509B9702A0122CF720E5E990013FF44622C21C322137440D2FD" />
    </Signer>
    <Signer ID="ID_SIGNER_S_26B_1_2_0_2_1" Name="GeoTrust Primary Certification Authority - G2">
      <CertRoot Type="TBS" Value="C65F6AC17E26D2DB60DB2D0BF6BFEBDBD7A03A5AADDB8587A4750AB4F9898AB2C59DD3AEA188374FAF38D33E58863329" />
    </Signer>
    <Signer ID="ID_SIGNER_S_26C_1_2_0_2_1" Name="Amazon Root CA 1">
      <CertRoot Type="TBS" Value="6FC4B8AC3D2B52C08BAF56255E43D22C762962E4FACAB01ACE16D48EC008BE0A" />
    </Signer>
    <Signer ID="ID_SIGNER_S_26D_1_2_0_2_1" Name="CA Disig Root R1">
      <CertRoot Type="TBS" Value="4A7E09189BE16F8CB7BD4C2B643EBD6300950138" />
    </Signer>
    <Signer ID="ID_SIGNER_S_26E_1_2_0_2_1" Name="Network Solutions RSA Certificate Authority">
      <CertRoot Type="TBS" Value="67B7CF8DA28ECA382B949BEE856006D03DB841DDB7333B2BF0E5181027A11DDD2C3D952A0C7ADF1B58E002909E167620" />
    </Signer>
    <Signer ID="ID_SIGNER_S_26F_1_2_0_2_1" Name="SecureSign RootCA3">
      <CertRoot Type="TBS" Value="8C0B5A20BF9B6F371A37BB15E562C82F486BD8C0" />
    </Signer>
    <Signer ID="ID_SIGNER_S_270_1_2_0_2_1" Name="Microsoft Root Certificate Authority 2011">
      <CertRoot Type="TBS" Value="279CD652C4E252BFBE5217AC722205D7729BA409148CFA9E6D9E5B1CB94EAFF1" />
    </Signer>
    <Signer ID="ID_SIGNER_S_271_1_2_0_2_1" Name="NAVER Global Root Certification Authority">
      <CertRoot Type="TBS" Value="CB0ED5782058CAE101E7B5B54CA3E71B832869B12D6C06A32B5DCA7412034D75E263575032C3274799190408202CBF99" />
    </Signer>
    <Signer ID="ID_SIGNER_S_272_1_2_0_2_1" Name="Federal Common Policy CA">
      <CertRoot Type="TBS" Value="375DC361146CFBDD26F82CBF8A4C1A173C9B6A11AC61DFE4C28CAC281888ED22" />
    </Signer>
    <Signer ID="ID_SIGNER_S_273_1_2_0_2_1" Name="&quot;I.CA - Standard Certification Authority">
      <CertRoot Type="TBS" Value="ACEDD41B63C16FE7E0FA120026FD8CF58518C928CE179678FA29B324DFFE137B" />
    </Signer>
    <Signer ID="ID_SIGNER_S_274_1_2_0_2_1" Name="Verizon Global Root CA">
      <CertRoot Type="TBS" Value="28E178AAFED6448838C3EF00179709DA1F7CC09C54EF5ABAB65093FB804E562A" />
    </Signer>
    <Signer ID="ID_SIGNER_S_275_1_2_0_2_1" Name="Actalis Authentication CA G1">
      <CertRoot Type="TBS" Value="9AB638B1A08F6EE35C35B8695AE46789F70E9141" />
    </Signer>
    <Signer ID="ID_SIGNER_S_276_1_2_0_2_1" Name="thawte Primary Root CA">
      <CertRoot Type="TBS" Value="85FEF11B4F47FE3952F98301C9F98976FEFEE0CE" />
    </Signer>
    <Signer ID="ID_SIGNER_S_277_1_2_0_2_1" Name="CA DATEV INT 03">
      <CertRoot Type="TBS" Value="CF93552EC735FB40229B29FAD119CA95D2711F15" />
    </Signer>
    <Signer ID="ID_SIGNER_S_278_1_2_0_2_1" Name="Starfield Services Root Certificate Authority - G2">
      <CertRoot Type="TBS" Value="1504593902EC8A0BAB29F03BF35C3058B5FD1807A74DAB92CB61ED4A9908AFA4" />
    </Signer>
    <Signer ID="ID_SIGNER_S_279_1_2_0_2_1" Name="Symantec Enterprise Mobile Root for Microsoft">
      <CertRoot Type="TBS" Value="5753D57D68F332262C4CC2E5EF76848E03DDC8212C34C757087C2AA7E320A946" />
    </Signer>
    <Signer ID="ID_SIGNER_S_27A_1_2_0_2_1" Name="ACCVRAIZ1">
      <CertRoot Type="TBS" Value="DF0ADAA6D1F05AD803AC447EBEF1DEEECB9483CB" />
    </Signer>
    <Signer ID="ID_SIGNER_S_27B_1_2_0_2_1" Name="CA DATEV INT 02">
      <CertRoot Type="TBS" Value="B873695FDDE03353A9A0CD2AFDE74A4D9AE93B3D" />
    </Signer>
    <Signer ID="ID_SIGNER_S_27C_1_2_0_2_1" Name="Tunisian Root Certificate Authority - TunRootCA2">
      <CertRoot Type="TBS" Value="84148155A10B1BF6CE118BDF7E40F2F83B4B518230A26C6BCCBFD3B8D21F5814" />
    </Signer>
    <Signer ID="ID_SIGNER_S_27D_1_2_0_2_1" Name="OU=Application CA G2, O=LGPKI, C=JP">
      <CertRoot Type="TBS" Value="E93D57C681D74A9474FCDEFC001C60DB6543F813" />
    </Signer>
    <Signer ID="ID_SIGNER_S_27E_1_2_0_2_1" Name="D-TRUST Root Class 3 CA 2 EV 2009">
      <CertRoot Type="TBS" Value="D6785F75F0E8CEA41BBEDF55A99B6B02F9F893D8E387FCF7EC1E14C17EF51160" />
    </Signer>
    <Signer ID="ID_SIGNER_S_27F_1_2_0_2_1" Name="VI Registru Centras RCSC (RootCA)">
      <CertRoot Type="TBS" Value="38EB94E886AED1DEFAEF6F2FEFBF8A5278FFF295" />
    </Signer>
    <Signer ID="ID_SIGNER_S_280_1_2_0_2_1" Name="GTE CyberTrust Global Root">
      <CertRoot Type="TBS" Value="E1B34A19374FC710C61667B82E8F1C2C" />
    </Signer>
    <Signer ID="ID_SIGNER_S_281_1_2_0_2_1" Name="TM Applied Business Root Certificate">
      <CertRoot Type="TBS" Value="1FC031BEFFCE2D18E11100C1FAB1C4C6652F32193F54391A4129A5CFFC12A76C" />
    </Signer>
    <Signer ID="ID_SIGNER_S_282_1_2_0_2_1" Name="I.CA Root CA/RSA">
      <CertRoot Type="TBS" Value="01D87750A41CCB2CB65C3E50D655E8BA8AD8A74A4B3A59510FF419CFC7A96D8135A6E943BE6AAA92C0207512BB6C391C997DFC143F9BC52D7058889EF3100978" />
    </Signer>
    <Signer ID="ID_SIGNER_S_283_1_2_0_2_1" Name="SwissSign Silver CA - G2">
      <CertRoot Type="TBS" Value="526AAA5D52A07C057AD6E17522FB678A3E154558" />
    </Signer>
    <Signer ID="ID_SIGNER_S_284_1_2_0_2_1" Name="KEYNECTIS ROOT CA">
      <CertRoot Type="TBS" Value="B96093E75DE4A329035ADD216B0D5A0947DE172220174DB9C4F323CB380EA205" />
    </Signer>
    <Signer ID="ID_SIGNER_S_285_1_2_0_2_1" Name="TWCA Global Root CA">
      <CertRoot Type="TBS" Value="1D1B3D16AE2539E340C3E18FB7124E710DD9DAF1E756D2C7BC206404705CE833" />
    </Signer>
    <Signer ID="ID_SIGNER_S_286_1_2_0_2_1" Name="Certinomis - Root CA">
      <CertRoot Type="TBS" Value="C3C592DC85FC87FD0283612F4B07612F97AD45156516A06AAD07D54F5AFDC93F" />
    </Signer>
    <Signer ID="ID_SIGNER_S_287_1_2_0_2_1" Name="COMODO ECC Certification Authority">
      <CertRoot Type="TBS" Value="2D1C163DE15BBD92733712539A2F98928EE16D87B615A917EB040B9A7F3BC19557AB4EFA3E1B8856651DAA4856656ED7" />
    </Signer>
    <Signer ID="ID_SIGNER_S_288_1_2_0_2_1" Name="esignit.org">
      <CertRoot Type="TBS" Value="A1989169CF167AA73BD4DAC0E16CE6ADD1AA99194F262CC6CB74F657925B72BF2EC4D9BE4C509AAFB66A18A020A0A319D9DD5DD871FCBD89EDA646BCC0F73B1C" />
    </Signer>
    <Signer ID="ID_SIGNER_S_289_1_2_0_2_1" Name="Hellenic Academic and Research Institutions ECC RootCA 2015">
      <CertRoot Type="TBS" Value="8BDAD1A79FEC14D9E71A70DC6E9B8D4569C2C8B0E9CB7A5D1C68030CB97B2FA7" />
    </Signer>
    <Signer ID="ID_SIGNER_S_28A_1_2_0_2_1" Name="Root CA Generalitat Valenciana">
      <CertRoot Type="TBS" Value="89E8BAE9373E686E9F915F98B78107A9F4F9AB6D" />
    </Signer>
    <Signer ID="ID_SIGNER_S_28B_1_2_0_2_1" Name="PostSignum Root QCA 2">
      <CertRoot Type="TBS" Value="A65E94296B4431F286AD83B1710D08E9FE9928205FED315250ADB2C442C158B0" />
    </Signer>
    <Signer ID="ID_SIGNER_S_28C_1_2_0_2_1" Name="DigiCert Assured ID Root G2">
      <CertRoot Type="TBS" Value="55F6FDB37438EF60921041652435BD675053196D206537FEF13BDFAE8AF2B525" />
    </Signer>
    <Signer ID="ID_SIGNER_S_28D_1_2_0_2_1" Name="Swiss Government Root CA I">
      <CertRoot Type="TBS" Value="B25882961F75684B845E860C0E1C7800CCD36C2DA5233E5980D2BDEBAA4B414A" />
    </Signer>
    <Signer ID="ID_SIGNER_S_28E_1_2_0_2_1" Name="SwissSign Platinum Root CA - G3">
      <CertRoot Type="TBS" Value="11FE9305E2689501EA9E9BD8BFD7FD88149336AE9C8F7C96F7E6D8E89CB6EAFF" />
    </Signer>
    <Signer ID="ID_SIGNER_S_28F_1_2_0_2_1" Name="CCA India 2014">
      <CertRoot Type="TBS" Value="B941D1DAAC35F3FE6F86601FCF4813BD91BE31B34B822D07F2E98706D1062010" />
    </Signer>
    <Signer ID="ID_SIGNER_S_290_1_2_0_2_1" Name="UCA Extended Validation Root">
      <CertRoot Type="TBS" Value="F7264AA07ED1280ACA6E666E4F85FCA8E3E6AC6983FBF2071F30EB2CFB0B7383" />
    </Signer>
    <Signer ID="ID_SIGNER_S_291_1_2_0_2_1" Name="Microsoft Root Authority">
      <CertRoot Type="TBS" Value="8B3C3087B7056F5EC5DDBA91A1B901F0" />
    </Signer>
    <Signer ID="ID_SIGNER_S_292_1_2_0_2_1" Name="Certipost E-Trust Primary Normalised CA">
      <CertRoot Type="TBS" Value="E2EDAFD4D1AB0EF7F3D77637E04A51D6AC4AFE1B" />
    </Signer>
    <Signer ID="ID_SIGNER_S_293_1_2_0_2_1" Name="CISRCA1">
      <CertRoot Type="TBS" Value="BFD4ACA2F874A6B89CC4ABA787F3C9BD911BD3FAEF546C29705061B837F0CB5A" />
    </Signer>
    <Signer ID="ID_SIGNER_S_294_1_2_0_2_1" Name="DigiCert Global Root CA">
      <CertRoot Type="TBS" Value="B34DDD372ED92E8F2ABFBB9E20A9D31F204F194B" />
    </Signer>
    <Signer ID="ID_SIGNER_S_295_1_2_0_2_1" Name="Autoridade Certificadora Raiz Brasileira v2">
      <CertRoot Type="TBS" Value="EDE6A61CA180765034086DBE66A2F90FEF463D9F44AAEBE48029F0D555947D9D3075769F62D1C03707A585E269197D4B669CDD643DFE6C80636291A253C00270" />
    </Signer>
    <Signer ID="ID_SIGNER_S_296_1_2_0_2_1" Name="Australian Defence Public Root CA">
      <CertRoot Type="TBS" Value="CFAB614EB8FF5EE18E8B74AF5A6CDB29FD0C3F530F4E4FB42EE7618CA3C75C65" />
    </Signer>
    <Signer ID="ID_SIGNER_S_297_1_2_0_2_1" Name="GeoTrust Global CA 2">
      <CertRoot Type="TBS" Value="C445B30C7D0B3759925C490FAFDA0C20B0D03730" />
    </Signer>
    <Signer ID="ID_SIGNER_S_298_1_2_0_2_1" Name="PostSignum Root QCA 4">
      <CertRoot Type="TBS" Value="C79F846E770BDB37FFFCF37AF3E214EB6BE85696F78E05115B6F76DB107E139568258026A6AE467D16A30D93355715148C18F0C041B7F9CE5132D9A003A46670" />
    </Signer>
    <Signer ID="ID_SIGNER_S_299_1_2_0_2_1" Name="thawte Primary Root CA - G2">
      <CertRoot Type="TBS" Value="051BE0BFA627DEA0599AF7B016E65FCA2011BA03E693B0A830582BDF934C7F23987B08F91C9F061168DC7A63DD799C31" />
    </Signer>
    <Signer ID="ID_SIGNER_S_29A_1_2_0_2_1" Name="I.CA - Standard root certificate">
      <CertRoot Type="TBS" Value="0E4201093B69155F763F4ABC860C7EB93CE0E342" />
    </Signer>
    <Signer ID="ID_SIGNER_S_29B_1_2_0_2_1" Name="CA DATEV STD 02">
      <CertRoot Type="TBS" Value="EB7A8BDFE52D2D4051AB0ECD7FF9505F29A01708" />
    </Signer>
    <Signer ID="ID_SIGNER_S_29C_1_2_0_2_1" Name="NetLock Kozjegyzoi (Class A) Tanusitvanykiado">
      <CertRoot Type="TBS" Value="6FE5D7A8F9A30400C9444FD2F7844B09" />
    </Signer>
    <Signer ID="ID_SIGNER_S_29D_1_2_0_2_1" Name="OU=Starfield Class 2 Certification Authority, O=&quot;Starfield Technologies, Inc.&quot;, C=US">
      <CertRoot Type="TBS" Value="0F6AAD4C3FE04619CDC8B2BD655AA1A26042E650" />
    </Signer>
    <Signer ID="ID_SIGNER_S_29E_1_2_0_2_1" Name="ComSign Global Root CA">
      <CertRoot Type="TBS" Value="ED9F175EF9120E87F1E9EB48B767EFB4E302C3F71F54943233321F81951EF2E6" />
    </Signer>
    <Signer ID="ID_SIGNER_S_29F_1_2_0_2_1" Name="Autoridad de Certificacion Firmaprofesional CIF A62634068">
      <CertRoot Type="TBS" Value="C4914746CF1E0D8D665EFFD226C7C5127878BE00" />
    </Signer>
    <Signer ID="ID_SIGNER_S_2A0_1_2_0_2_1" Name="COMODO RSA Certification Authority">
      <CertRoot Type="TBS" Value="761613F4CD8607508C3D520FBEFE68773735FC73746F42A9FD6254BA3B72F0047994E5AF57677CF6D2C1965984965DF1" />
    </Signer>
    <Signer ID="ID_SIGNER_S_2A1_1_2_0_2_1" Name="O=Government Root Certification Authority, C=TW">
      <CertRoot Type="TBS" Value="FC71D060ACACCF336C07F960D87BB36FA02AFD8921E57DDE40F08CEEE2769018" />
    </Signer>
    <Signer ID="ID_SIGNER_S_2A2_1_2_0_2_1" Name="Certigna">
      <CertRoot Type="TBS" Value="D49BA8CA0DB5E6C661B57B56F33B4F05163FF8F2" />
    </Signer>
    <Signer ID="ID_SIGNER_S_2A3_1_2_0_2_1" Name="UTN-USERFirst-Client Authentication and Email">
      <CertRoot Type="TBS" Value="BBA770661F7C955307EC67E2369D3C3991E21FDC" />
    </Signer>
    <Signer ID="ID_SIGNER_S_2A4_1_2_0_2_1" Name="GlobalSign Root CA">
      <CertRoot Type="TBS" Value="5A6D07B6371D966A2FB6BA92828CE5512A49513D" />
    </Signer>
    <Signer ID="ID_SIGNER_S_2A5_1_2_0_2_1" Name="OU=POSTArCA, O=POSTA, C=SI">
      <CertRoot Type="TBS" Value="511C12311CE4701DF4DD37BBC27914B3DDA86CB0" />
    </Signer>
    <Signer ID="ID_SIGNER_S_2A6_1_2_0_2_1" Name="Signet Root CA">
      <CertRoot Type="TBS" Value="C3A06444454737EF1E54818BB6BAB6C9A0A5AD371845D852C13C432FF6D6FAB4" />
    </Signer>
    <Signer ID="ID_SIGNER_S_2A7_1_2_0_2_1" Name="Entrust Root Certification Authority">
      <CertRoot Type="TBS" Value="254F527930383ECBE8B1B23D4940698C516F5A7C" />
    </Signer>
    <Signer ID="ID_SIGNER_S_2A8_1_2_0_2_1" Name="AC RAIZ DNIE">
      <CertRoot Type="TBS" Value="EF8B3F0C89F556961EFDE20191F5E0E3AE1CA048" />
    </Signer>
    <Signer ID="ID_SIGNER_S_2A9_1_2_0_2_1" Name="Trustwave Global ECC P256 Certification Authority">
      <CertRoot Type="TBS" Value="EA21AAA3F7594717EA87C8F62B56D5A7E3AABD721677319803D3F8FE6282F141" />
    </Signer>
    <Signer ID="ID_SIGNER_S_2AA_1_2_0_2_1" Name="Starfield Root Certificate Authority - G2">
      <CertRoot Type="TBS" Value="71B437F087F3700FFD4E2FA46F42B6B810D7BF19ADFEDF951C023EDD65B50B05" />
    </Signer>
    <Signer ID="ID_SIGNER_S_2AB_1_2_0_2_1" Name="CA Disig Root R2">
      <CertRoot Type="TBS" Value="D7C07DF8AC609D732E922AD8B6207CAA61B345382F6015502640DFB4775C0FA3" />
    </Signer>
    <Signer ID="ID_SIGNER_S_2AC_1_2_0_2_1" Name="emSign ECC Root CA - C3">
      <CertRoot Type="TBS" Value="477BAAA25FCB9ED820D0C432C8D6118E1FD3AA7A66EC06B736D601B1575B25E8CF0A7E3D0402A94622E698B0CC0CA4B2" />
    </Signer>
    <Signer ID="ID_SIGNER_S_2AD_1_2_0_2_1" Name="SSL.com Root Certification Authority RSA">
      <CertRoot Type="TBS" Value="489FF6233F3D3C5DA77604BE230745657FE488CB05257DA551BFD64C1F179E72" />
    </Signer>
    <Signer ID="ID_SIGNER_S_2AE_1_2_0_2_1" Name="XRamp Global Certification Authority">
      <CertRoot Type="TBS" Value="7DC305DFEECBC204AD650E5BCAF99C84AC749C3F" />
    </Signer>
    <Signer ID="ID_SIGNER_S_2AF_1_2_0_2_1" Name="Microsoft EV ECC Root Certificate Authority 2017">
      <CertRoot Type="TBS" Value="2E98146A2374DA82479AFA1806B058654F8CC45C8F27815C62F24AF57C9C6A2BD7ACC6592AB42743884183DB5921E6E1" />
    </Signer>
    <Signer ID="ID_SIGNER_S_2B0_1_2_0_2_1" Name="Security Communication ECC RootCA1">
      <CertRoot Type="TBS" Value="7E359060547600864BECF834D75FEF83FB468FFEDE373404B76D9EFBB2A2CCE8E55ABEA11CDB0F71685E7015355E51AD" />
    </Signer>
    <Signer ID="ID_SIGNER_S_2B1_1_2_0_2_1" Name="AffirmTrust Premium ECC">
      <CertRoot Type="TBS" Value="0C6AB563667BABBC67A05050C5F8FE6EBEB23DB5CF48A88D9C4D686376710A56A658A42F9469ED94622E9097CBD2062E" />
    </Signer>
    <Signer ID="ID_SIGNER_S_2B2_1_2_0_2_1" Name="OU=AC RAIZ FNMT-RCM, O=FNMT-RCM, C=ES">
      <CertRoot Type="TBS" Value="86A09A310E3CED34E66470FA1F5C7FAAB27D9D34" />
    </Signer>
    <Signer ID="ID_SIGNER_S_2B3_1_2_0_2_1" Name="TrustCor RootCert CA-2">
      <CertRoot Type="TBS" Value="4F47F02AF39948DA80E8F581968AB3C7B1FFF2101BE05F8113E59190025BDB7C" />
    </Signer>
    <Signer ID="ID_SIGNER_S_2B4_1_2_0_2_1" Name="Certification Authority of WoSign">
      <CertRoot Type="TBS" Value="44CB4357ECB773B9AC3A3B0B1E45AB6BC45C2F1C" />
    </Signer>
    <Signer ID="ID_SIGNER_S_2B5_1_2_0_2_1" Name="IdenTrust Public Sector Root CA 1">
      <CertRoot Type="TBS" Value="C06D5B5B4BBEB48D56852B7F29575C85A40FCFBB28D7A621BB6557CEC418DC53" />
    </Signer>
    <Signer ID="ID_SIGNER_S_2B6_1_2_0_2_1" Name="emSign Root CA - C2">
      <CertRoot Type="TBS" Value="EC7ED68B075468A437449838865649C8E2D1BFDD2F058F803EB2C757C5B9E41BA1F6C3649D98084A78E45CFD48D7F943" />
    </Signer>
    <Signer ID="ID_SIGNER_S_2B7_1_2_0_2_1" Name="Thawte Timestamping CA">
      <CertRoot Type="TBS" Value="E8A598BE84828EFEAE701115013576B2" />
    </Signer>
    <Signer ID="ID_SIGNER_S_2B8_1_2_0_2_1" Name="S-TRUST Authentication and Encryption Root CA 2005:PN">
      <CertRoot Type="TBS" Value="24B99E7F6E1D8BFF793C51872A6EAC878BDA7050" />
    </Signer>
    <Signer ID="ID_SIGNER_S_2B9_1_2_0_2_1" Name="CCA India 2011">
      <CertRoot Type="TBS" Value="CF0DAB61B7E4AE5E0534FF59BB97E29C3C81630CF1C6B5AA74E70BD9EC729B80" />
    </Signer>
    <Signer ID="ID_SIGNER_S_2BA_1_2_0_2_1" Name="ANCERT Certificados Notariales">
      <CertRoot Type="TBS" Value="4FF4AFD3B64CEE484FCEDA64C403C7DE1AC294F4" />
    </Signer>
    <Signer ID="ID_SIGNER_S_2BB_1_2_0_2_1" Name="Security Communication RootCA3">
      <CertRoot Type="TBS" Value="E4B4F27BA8E2645D5A9272F876698071787EEBD37328BF36BABE2B0ECE4058BE9C1CFB0D7A21337F2C9AB12E45BD85CE" />
    </Signer>
    <Signer ID="ID_SIGNER_S_2BC_1_2_0_2_1" Name="SSL.com Root Certification Authority ECC">
      <CertRoot Type="TBS" Value="0CF408E20830AC839F2A12143CE4B3E53AC6A27B2338A504F200FA3A154C4FD8" />
    </Signer>
    <Signer ID="ID_SIGNER_S_2BD_1_2_0_2_1" Name="TÜRKTRUST Elektronik Sertifika Hizmet Sağlayıcısı H5">
      <CertRoot Type="TBS" Value="D8523A20072B133D13EF9D542F56853C9A85432F157799774008E199B8374284" />
    </Signer>
    <Signer ID="ID_SIGNER_S_2BE_1_2_0_2_1" Name="Swiss Government Root CA II">
      <CertRoot Type="TBS" Value="ED64C4050EA75C93519C6214A0DF30975134EF6A89996C7E91E7C51599B5EA38" />
    </Signer>
    <Signer ID="ID_SIGNER_S_2BF_1_2_0_2_1" Name="SSC GDL CA Root B">
      <CertRoot Type="TBS" Value="7AECE32EACD35B7EE5B171F6239F075088C77F6E63393C7C3E4D76580C95E96C" />
    </Signer>
    <Signer ID="ID_SIGNER_S_2C0_1_2_0_2_1" Name="E-ME SSI (RCA)">
      <CertRoot Type="TBS" Value="DADE06769A15956E024D13665B3ABE47FE28A196" />
    </Signer>
    <Signer ID="ID_SIGNER_S_2C1_1_2_0_2_1" Name="LuxTrust Global Root">
      <CertRoot Type="TBS" Value="087129B41285EFAEF659C86EFB8E161FD007F36E1E2F3369C45A4260236495CE" />
    </Signer>
    <Signer ID="ID_SIGNER_S_2C2_1_2_0_2_1" Name="EE Certification Centre Root CA">
      <CertRoot Type="TBS" Value="3FD9A3751E2081CB6BF65CCEBD588623D20D9A61" />
    </Signer>
    <Signer ID="ID_SIGNER_S_2C3_1_2_0_2_1" Name="QuoVadis Root CA 2">
      <CertRoot Type="TBS" Value="C8F8A3C6BF401D34E6F1D8F8E1DDD08BBB934626" />
    </Signer>
    <Signer ID="ID_SIGNER_S_2C4_1_2_0_2_1" Name="SecureSign RootCA1">
      <CertRoot Type="TBS" Value="729D4C00652FDAC523063A9456B6183E82EA2CA7" />
    </Signer>
    <Signer ID="ID_SIGNER_S_2C5_1_2_0_2_1" Name="ISRG Root X1">
      <CertRoot Type="TBS" Value="3F0411EDE9C4477057D57E57883B1F205B20CDC0F3263129B1EE0269A2678F63" />
    </Signer>
    <Signer ID="ID_SIGNER_S_2C6_1_2_0_2_1" Name="Common Policy">
      <CertRoot Type="TBS" Value="0E5FCF764C89AD384697340970CC4E3EFA6B3521" />
    </Signer>
    <Signer ID="ID_SIGNER_S_2C7_1_2_0_2_1" Name="Echoworx Root CA2">
      <CertRoot Type="TBS" Value="AAD20751F2F1F96E8BC48FF5A14E174EC4CE709F" />
    </Signer>
    <Signer ID="ID_SIGNER_S_2C8_1_2_0_2_1" Name="AC Raíz Certicámara S.A.">
      <CertRoot Type="TBS" Value="FB8068C0FFCEEF5F965C173454E6A3EDF83BB062" />
    </Signer>
    <Signer ID="ID_SIGNER_S_2C9_1_2_0_2_1" Name="Swiss Government Root CA III">
      <CertRoot Type="TBS" Value="BD5CB444DE26A4547A4350131863A51CED150593DA5A214C6BE8FDB336FAD237" />
    </Signer>
    <Signer ID="ID_SIGNER_S_2CA_1_2_0_2_1" Name="A-Trust-Qual-02">
      <CertRoot Type="TBS" Value="1DDC260DB1ED23CDFD6EA50F84678A67EE60DDA8" />
    </Signer>
    <Signer ID="ID_SIGNER_S_2CB_1_2_0_2_1" Name="Microsoft Root Certificate Authority">
      <CertRoot Type="TBS" Value="391BE92883D52509155BFEAE27B9BD340170B76B" />
    </Signer>
    <Signer ID="ID_SIGNER_S_2CC_1_2_0_2_1" Name="ANF Server CA">
      <CertRoot Type="TBS" Value="C4D90E9489ADD808A5DB10CB59895DB17D02E47F" />
    </Signer>
    <Signer ID="ID_SIGNER_S_2CD_1_2_0_2_1" Name="TWCA Root Certification Authority">
      <CertRoot Type="TBS" Value="F61C40F715933F23BDF4A2CF75DF6777ABA7376B" />
    </Signer>
    <Signer ID="ID_SIGNER_S_2CE_1_2_0_2_1" Name="USERTrust ECC Certification Authority">
      <CertRoot Type="TBS" Value="0B043572C899DEC43EFD590CFCE610CF443A6315925EBFE589F7506907E44824608489581C7CA0E041458514CF157614" />
    </Signer>
    <Signer ID="ID_SIGNER_S_2CF_1_2_0_2_1" Name="AAA Certificate Services">
      <CertRoot Type="TBS" Value="3E8E6487F8FD27D322A269A71EDAAC5D57811286" />
    </Signer>
    <Signer ID="ID_SIGNER_S_2D0_1_2_0_2_1" Name="OU=Equifax Secure Certificate Authority, O=Equifax, C=US">
      <CertRoot Type="TBS" Value="FFA3AC0084DA1673B5A031EBB2156B3E8FBBF6D8" />
    </Signer>
    <Signer ID="ID_SIGNER_S_2D1_1_2_0_2_1" Name="&quot;I.CA - Qualified Certification Authority">
      <CertRoot Type="TBS" Value="08B3513257F11859D4F76E36128068ABEECAE94FEB873C423959C514458D3868" />
    </Signer>
    <Signer ID="ID_SIGNER_S_2D2_1_2_0_2_1" Name="SSC GDL CA VS Root">
      <CertRoot Type="TBS" Value="3F0690CD403B94AD7502B8ABBF1EE002D55BBF896533BB6382F8CFAF6F3C2879" />
    </Signer>
    <Signer ID="ID_SIGNER_S_2D3_1_2_0_2_1" Name="GTS Root R2">
      <CertRoot Type="TBS" Value="41E2F7E5F10535B1307965E1B82B42F29E7FA2F298DF3508B9790818DB73BB2D4007F68D70C2D5112DC450E34F387EC7" />
    </Signer>
    <Signer ID="ID_SIGNER_S_2D4_1_2_0_2_1" Name="CA WoSign ECC Root">
      <CertRoot Type="TBS" Value="238971A442835C4AE38596EA145E9C637D91DA7B97B1CA167471C562A8182985F87E2B76F3139B1FC47CBBD5E0502B7E" />
    </Signer>
    <Signer ID="ID_SIGNER_S_2D5_1_2_0_2_1" Name="Class 3 Primary CA">
      <CertRoot Type="TBS" Value="294098C8DE6D334AD9B6D49B046E95BA86236771" />
    </Signer>
    <Signer ID="ID_SIGNER_S_2D6_1_2_0_2_1" Name="Certum Trusted Network CA 2">
      <CertRoot Type="TBS" Value="5CF58DC4429325FB69E9498383333ACBF76EDDFD5845BB9D29FDB935B2652C9184295565157A1D83335F9B67E3E2B67D6C01238CE81ADECBF3D75E98B3E99D79" />
    </Signer>
    <Signer ID="ID_SIGNER_S_2D7_1_2_0_2_1" Name="SZAFIR ROOT CA">
      <CertRoot Type="TBS" Value="B12270C4258F2CF2F009C383035503E876348AA9" />
    </Signer>
    <Signer ID="ID_SIGNER_S_2D8_1_2_0_2_1" Name="ACA ROOT">
      <CertRoot Type="TBS" Value="43E085204EA02203DE44F8CC2998A26E304F631FF1296B873143D9B78BCBD914" />
    </Signer>
    <Signer ID="ID_SIGNER_S_2D9_1_2_0_2_1" Name="Baltimore CyberTrust Root">
      <CertRoot Type="TBS" Value="CE0E658AA3E847E467A147B3049191093D055E6F" />
    </Signer>
    <Signer ID="ID_SIGNER_S_2DA_1_2_0_2_1" Name="GlobalSign">
      <CertRoot Type="TBS" Value="5229BA15B31B0C6F4CCA89C2985177974327D1B689A3B935A0BD975532AF22AB" />
    </Signer>
    <Signer ID="ID_SIGNER_S_2DB_1_2_0_2_1" Name="Posta CA Root">
      <CertRoot Type="TBS" Value="E32638C0C98734D2C1E568FAB93443B1AE37675B" />
    </Signer>
    <Signer ID="ID_SIGNER_S_2DC_1_2_0_2_1" Name="Hongkong Post Root CA 1">
      <CertRoot Type="TBS" Value="1AF28EF7ECF42E4E8F197F460D7556C7039F4D2E" />
    </Signer>
    <Signer ID="ID_SIGNER_S_2DD_1_2_0_2_1" Name="AffirmTrust Premium">
      <CertRoot Type="TBS" Value="4C4C09E8300419A64DEC94200477F0F059C033FC793B04E4FB0CAF29CBC262B5815396DF53648FC80310795238AB8BCD" />
    </Signer>
    <Signer ID="ID_SIGNER_S_2DE_1_2_0_2_1" Name="SwissSign Gold CA - G2">
      <CertRoot Type="TBS" Value="27C3A4D3AAFB96F69E1FC9FA5977CE9D831934AE" />
    </Signer>
    <Signer ID="ID_SIGNER_S_2DF_1_2_0_2_1" Name="Staat der Nederlanden Root CA - G3">
      <CertRoot Type="TBS" Value="9928E650F05E868B02EFEA00E860FB2904DFEE6BBC35395D3258833E59B58D98" />
    </Signer>
    <Signer ID="ID_SIGNER_S_2E0_1_2_0_2_1" Name="ePKI Root Certification Authority - G2">
      <CertRoot Type="TBS" Value="63122EBB44D6B2FA64EDDD452BFAD472453751CEFDA9F645E23999FF25AB7D67" />
    </Signer>
    <Signer ID="ID_SIGNER_S_2E1_1_2_0_2_1" Name="CA DATEV BT 01">
      <CertRoot Type="TBS" Value="D8B06556A3498819007FD65007E68D3291281F0B" />
    </Signer>
    <Signer ID="ID_SIGNER_S_2E2_1_2_0_2_1" Name="DST Root CA X3">
      <CertRoot Type="TBS" Value="5BCAA1C2780F0BCB5A90770451D96F38963F012D" />
    </Signer>
    <Signer ID="ID_SIGNER_S_2E3_1_2_0_2_1" Name="Buypass Class 3 Root CA">
      <CertRoot Type="TBS" Value="B6F1FB471CC89480980B2A8BB76F75BB60A29E057236A5E5274C1EA60FA9AFF8" />
    </Signer>
    <Signer ID="ID_SIGNER_S_2E4_1_2_0_2_1" Name="ATHEX Root CA">
      <CertRoot Type="TBS" Value="7A6E62C9F85260AD9E83932B8C6EBD5890323834" />
    </Signer>
    <Signer ID="ID_SIGNER_S_2E5_1_2_0_2_1" Name="Autoridad de Certificacion Raiz del Estado Venezolano">
      <CertRoot Type="TBS" Value="B8F3F915D28678831BAFAD9B47AC5111462128BA" />
    </Signer>
    <Signer ID="ID_SIGNER_S_2E6_1_2_0_2_1" Name="e-Guven Kok Elektronik Sertifika Hizmet Saglayicisi">
      <CertRoot Type="TBS" Value="F55BCC78926D88127610037F48B59B94F3FA6101" />
    </Signer>
    <Signer ID="ID_SIGNER_S_2E7_1_2_0_2_1" Name="DigiCert Trusted Root G4">
      <CertRoot Type="TBS" Value="4EA1B34B10B982A96A38915843507820AD632C6AAD8343E337B34D660CD8366FA154544AE80668AE1FDF3931D57E1996" />
    </Signer>
    <Signer ID="ID_SIGNER_S_2E8_1_2_0_2_1" Name="Hongkong Post Root CA 2">
      <CertRoot Type="TBS" Value="24ED0E35CBBC5D8A92186E5EC3EFD91C13BA632777BF013E20751B70F97886DF" />
    </Signer>
    <Signer ID="ID_SIGNER_S_2E9_1_2_0_2_1" Name="GeoTrust Global CA">
      <CertRoot Type="TBS" Value="F6D2EF0341D7F1BAA1DE015024AF0D8221EF716E" />
    </Signer>
    <Signer ID="ID_SIGNER_S_2EA_1_2_0_2_1" Name="QuoVadis Root Certification Authority">
      <CertRoot Type="TBS" Value="8A2375A87E6675F9FB223BE8FECE422D4573BC91" />
    </Signer>
    <Signer ID="ID_SIGNER_S_2EB_1_2_0_2_1" Name="Cisco Root CA 2048">
      <CertRoot Type="TBS" Value="210B7771CF36A78E846392B77D2F078AF48B13F7" />
    </Signer>
    <Signer ID="ID_SIGNER_S_2EC_1_2_0_2_1" Name="DigiCert Global Root G2">
      <CertRoot Type="TBS" Value="4B4EB4B074298B828B5C003095A10B4523FB951C0C88348B09C53E5BABA408A3" />
    </Signer>
    <Signer ID="ID_SIGNER_S_2ED_1_2_0_2_1" Name="TWCA Root Certification Authority">
      <CertRoot Type="TBS" Value="CA6FAF52BF1A102FB5C0C4474C1A60A7F959FE5670ADD4AA4DA718A41B3A221B" />
    </Signer>
    <Signer ID="ID_SIGNER_S_2EE_1_2_0_2_1" Name="IdenTrust Commercial Root CA 1">
      <CertRoot Type="TBS" Value="7A9BC7FFECF427111C5A2E5BF589FFFF1EE95FEF12B3CC42764D7C907A3F6959" />
    </Signer>
    <Signer ID="ID_SIGNER_S_2EF_1_2_0_2_1" Name="OISTE WISeKey Global Root GC CA">
      <CertRoot Type="TBS" Value="836D9489191F18A2959E1C1FD1C0EF276C268788B1F10506F1893C6A299199131076EB746B7009D92FD65008E113D1CE" />
    </Signer>
    <Signer ID="ID_SIGNER_S_2F0_1_2_0_2_1" Name="ACEDICOM Root">
      <CertRoot Type="TBS" Value="DB7C911BF8FE225C8AB812CD16072D8EDA9395BF" />
    </Signer>
    <Signer ID="ID_SIGNER_S_2F1_1_2_0_2_1" Name="UTN-USERFirst-Object">
      <CertRoot Type="TBS" Value="F45A0858C9CD920E647BAD539AB9F1CFC77F24CB" />
    </Signer>
    <Signer ID="ID_SIGNER_S_2F2_1_2_0_2_1" Name="ComSign CA">
      <CertRoot Type="TBS" Value="0437EFCE6C464D17FBE4DA5DB1552A44D69DBF1F" />
    </Signer>
    <Signer ID="ID_SIGNER_S_2F3_1_2_0_2_1" Name="GTS Root R1">
      <CertRoot Type="TBS" Value="E4C58A0A499480862DB093ADA2B299298D57D1C586BEE12C4B74D5E13DD4BCBDA6D57BE981EEE012E984E6B83D0B4C7B" />
    </Signer>
    <Signer ID="ID_SIGNER_S_2F4_1_2_0_2_1" Name="SZAFIR ROOT CA2">
      <CertRoot Type="TBS" Value="8F65AB514D193E1BC2C69D82520F73C4E3255744356064E9859107F26C0EFD5C" />
    </Signer>
    <Signer ID="ID_SIGNER_S_2F5_1_2_0_2_1" Name="CFCA EV ROOT">
      <CertRoot Type="TBS" Value="47452C0A07D5C464094763E532740F28E2B3034FF0FCBFABE2D0F22650F57D2A" />
    </Signer>
    <Signer ID="ID_SIGNER_S_2F6_1_2_0_2_1" Name="SAPO Class 2 Root CA">
      <CertRoot Type="TBS" Value="D074382C394C27DE0249FCB555A8EF835B889769" />
    </Signer>
    <Signer ID="ID_SIGNER_S_2F7_1_2_0_2_1" Name="GeoTrust Universal CA">
      <CertRoot Type="TBS" Value="3C574C8C3C5896F06FE67795885AFB18FEC4FF8D" />
    </Signer>
    <Signer ID="ID_SIGNER_S_2F8_1_2_0_2_1" Name="emSign Root CA - C1">
      <CertRoot Type="TBS" Value="281D590E2000FAF72E950EAC38010962188DB670446B1623ABBF2855D16BD6D3" />
    </Signer>
    <Signer ID="ID_SIGNER_S_2F9_1_2_0_2_1" Name="Swisscom Root EV CA 2">
      <CertRoot Type="TBS" Value="8B1FD6F6775FF5B9550DF544854C8FCEE94764C386EA9C1991E73C7FB94103DA" />
    </Signer>
    <Signer ID="ID_SIGNER_S_2FA_1_2_0_2_1" Name="Trustwave Global ECC P384 Certification Authority">
      <CertRoot Type="TBS" Value="224B13F7DB2BA006B545E060DC3F6D3BA7D8BC1AD643B31C8513F7F3C1280B27D561DCEC7A7AF11CE026A9B51BB660B2" />
    </Signer>
    <Signer ID="ID_SIGNER_S_2FB_1_2_0_2_1" Name="O=CFCA GT CA, C=CN">
      <CertRoot Type="TBS" Value="B09805544D023A57EA249858C090F5EC398A4118" />
    </Signer>
    <Signer ID="ID_SIGNER_S_2FC_1_2_0_2_1" Name="OU=AC RAIZ FNMT-RCM, O=FNMT-RCM, C=ES">
      <CertRoot Type="TBS" Value="FCA6451D4B87C5B079334C184D9D28CE7E61D6F1DB1A209F6E4B468970FF4E23" />
    </Signer>
    <Signer ID="ID_SIGNER_S_2FD_1_2_0_2_1" Name="NetLock Platina (Class Platinum) Főtanúsítvány">
      <CertRoot Type="TBS" Value="F908C630A787EA72D2E2A66B3569A32EEB9194D35C3D0F6E2C2F32FA16A6E6EE" />
    </Signer>
    <Signer ID="ID_SIGNER_S_2FE_1_2_0_2_1" Name="Microsoft RSA Root Certificate Authority 2017">
      <CertRoot Type="TBS" Value="69ED5A79811138471B0367AA2EDBE202F8F2CAA02D3AF05BDCF3617F00AE980994682DD398DEF59DC334914B3854A1C4" />
    </Signer>
    <Signer ID="ID_SIGNER_S_2FF_1_2_0_2_1" Name="COMODO Certification Authority">
      <CertRoot Type="TBS" Value="A69C493C75FAC1DA819B515A29BB219B63E31BA2" />
    </Signer>
    <Signer ID="ID_SIGNER_S_300_1_2_0_2_1" Name="ApplicationCA2 Root">
      <CertRoot Type="TBS" Value="A4431B6889FBE2AEF05D5E8797468083518DEEEEDAE06D078BC50E4D73ECD870" />
    </Signer>
    <Signer ID="ID_SIGNER_S_301_1_2_0_2_1" Name="CFCA Identity CA">
      <CertRoot Type="TBS" Value="C965FC3CB55836E91A329927F355C8297F8B2F402C0BD593AAEBF9945439CE16" />
    </Signer>
    <Signer ID="ID_SIGNER_S_302_1_2_0_2_1" Name="Digidentity L3 Root CA - G2">
      <CertRoot Type="TBS" Value="C4317DB709496F89FF93419EBD8D954253F4BA9D5B33A50FEBDB9D76CE48BC54" />
    </Signer>
    <Signer ID="ID_SIGNER_S_303_1_2_0_2_1" Name="thawte Primary Root CA - G3">
      <CertRoot Type="TBS" Value="9F0A5682A992F8534CE24CE23B6C04A1A258B5E42B2B8196B24DA0690F2240F8" />
    </Signer>
    <Signer ID="ID_SIGNER_S_304_1_2_0_2_1" Name="Actalis Authentication Root CA">
      <CertRoot Type="TBS" Value="927824E958A132AFBCADD9E12357A0F9788AB99C5669E1EC3825E1EB5F6F5454" />
    </Signer>
    <Signer ID="ID_SIGNER_S_305_1_2_0_2_1" Name="VRK Gov. Root CA - G2">
      <CertRoot Type="TBS" Value="57E9594F2EBFD3FBCD4B7DB6434BA18E94CADFA10B89A846A4EFDBEC772D09772876BFB155E5D4E0073A2C3F5C80958C9268F503104EB2F06319C364033AAE0C" />
    </Signer>
    <Signer ID="ID_SIGNER_S_306_1_2_0_2_1" Name="Class 3TS Primary CA">
      <CertRoot Type="TBS" Value="01A8CC75ED5DE358B06BFB26E58B857C72AD9577" />
    </Signer>
    <Signer ID="ID_SIGNER_S_307_1_2_0_2_1" Name="O=Government Root Certification Authority, C=TW">
      <CertRoot Type="TBS" Value="6576BCF08464F0A055B87AFF04A585F4308A4707" />
    </Signer>
    <Signer ID="ID_SIGNER_S_308_1_2_0_2_1" Name="DigiCert Assured ID Root G3">
      <CertRoot Type="TBS" Value="FA95CF8212A28D5983C7C9B5891589555DA7B46A4BA83D5ECCB3D6F4B9C4B2DB4FC23358C57C9CBE55EFD567D2521D03" />
    </Signer>
    <Signer ID="ID_SIGNER_S_309_1_2_0_2_1" Name="Amazon Root CA 4">
      <CertRoot Type="TBS" Value="E0DA58676E3A50DE9D8CB3AA5FFEFFDAE691BA9705B3ABE41A09270D63A3284F58247CE20D354B579EB548755912E833" />
    </Signer>
    <Signer ID="ID_SIGNER_S_30A_1_2_0_2_1" Name="AffirmTrust Commercial">
      <CertRoot Type="TBS" Value="1E2A4EF9C6208AEA146FE6074589F80FB1C7A73D2D20511B6F47D0E61874EB6A" />
    </Signer>
    <Signer ID="ID_SIGNER_S_30B_1_2_0_2_1" Name="ComSign Secured CA">
      <CertRoot Type="TBS" Value="3C0BB60D9259E5F84E6DA4DD217992DB6F1D39EF" />
    </Signer>
    <Signer ID="ID_SIGNER_S_30C_1_2_0_2_1" Name="Correo Uruguayo - Root CA">
      <CertRoot Type="TBS" Value="D7D56CCEB4FC8EDDB3C2BB0F4924DAA3D9AA9C8A" />
    </Signer>
    <Signer ID="ID_SIGNER_S_30D_1_2_0_2_1" Name="Certeurope Root CA 2">
      <CertRoot Type="TBS" Value="A6D522610F3DE3265F9E64E8EB7E0E26A1AA7B42" />
    </Signer>
    <Signer ID="ID_SIGNER_S_30E_1_2_0_2_1" Name="VRK Gov. Root CA">
      <CertRoot Type="TBS" Value="A1D0B0B2415CED2D6CAE1FBA721B5FC4089ECE8F" />
    </Signer>
    <Signer ID="ID_SIGNER_S_30F_1_2_0_2_1" Name="OU=certSIGN ROOT CA, O=certSIGN, C=RO">
      <CertRoot Type="TBS" Value="FB3C6AB2DE9A3F0B74E5C372ED86A52B25385A6C" />
    </Signer>
    <Signer ID="ID_SIGNER_S_310_1_2_0_2_1" Name="Certification Authority of WoSign G2">
      <CertRoot Type="TBS" Value="63B2F71DF462D039ECBE4D03E68B1F1C6B005547D4E5EDFB57E9232EBEA34E2E" />
    </Signer>
    <Signer ID="ID_SIGNER_S_311_1_2_0_2_1" Name="D-TRUST Root Class 3 CA 2007">
      <CertRoot Type="TBS" Value="1BD29F89F6A79A0F673E9187C11BD86985B03260" />
    </Signer>
    <Signer ID="ID_SIGNER_S_312_1_2_0_2_1" Name="RCSC RootCA">
      <CertRoot Type="TBS" Value="6945FF71C59580E108533160A7D990B06C089ECEF8715CE8CF858EC29D2143C6" />
    </Signer>
    <Signer ID="ID_SIGNER_S_313_1_2_0_2_1" Name="Hellenic Academic and Research Institutions RootCA 2011">
      <CertRoot Type="TBS" Value="953FBDAEE03FD0A1F4527440A14940A3FD2CBB6C" />
    </Signer>
    <Signer ID="ID_SIGNER_S_314_1_2_0_2_1" Name="OU=Security Communication EV RootCA1, O=&quot;SECOM Trust Systems CO.,LTD.&quot;, C=JP">
      <CertRoot Type="TBS" Value="CF5C65CD58EBB3D31A6618C3D42BE79BB5DE72E8" />
    </Signer>
    <Signer ID="ID_SIGNER_S_315_1_2_0_2_1" Name="Főtanúsítványkiadó - Kormányzati Hitelesítés Szolgáltató">
      <CertRoot Type="TBS" Value="4E3F084ED3C4B0D20C5D5E2E1D33F6096C65162DE9D58F866D8BC6B79EDC39FB" />
    </Signer>
    <Signer ID="ID_SIGNER_S_316_1_2_0_2_1" Name="TrustCor RootCert CA-1">
      <CertRoot Type="TBS" Value="91655B012CB919746D6127AA24E54BA90B2F15A01927DC1DC88E54365D126363" />
    </Signer>
    <Signer ID="ID_SIGNER_MIMIKATZ_KERNEL_2" Name="GlobalSign CodeSigning CA - G2">
      <CertRoot Type="TBS" Value="589A7D4DF869395601BA7538A65AFAE8C4616385" />
      <CertPublisher Value="Benjamin Delpy" />
    </Signer>
    <Signer ID="ID_SIGNER_MIMIKATZ_USER_2" Name="Certum Code Signing CA SHA2">
      <CertRoot Type="TBS" Value="F7B6EEB3A567223000A61F68C53B458193557C17E5D512D2825BCB13E5FC9BE5" />
      <CertPublisher Value="Open Source Developer, Benjamin Delpy" />
    </Signer>
    <Signer ID="ID_SIGNER_NOVELL_2" Name="VeriSign Class 3 Code Signing 2009-2 CA">
      <CertRoot Type="TBS" Value="4CDC38C800761463749C3CBD94A12F32E49877BF" />
      <CertPublisher Value="Novell, Inc." />
      <FileAttribRef RuleID="ID_FILEATTRIB_LIBNICM_DRIVER_2" />
      <FileAttribRef RuleID="ID_FILEATTRIB_NICM_DRIVER_2" />
      <FileAttribRef RuleID="ID_FILEATTRIB_NSCM_DRIVER_2" />
    </Signer>
    <Signer ID="ID_SIGNER_RWEVERY_2" Name="GlobalSign CodeSigning CA - G2">
      <CertRoot Type="TBS" Value="589A7D4DF869395601BA7538A65AFAE8C4616385" />
      <CertPublisher Value="ChongKim Chan" />
    </Signer>
    <Signer ID="ID_SIGNER_SANDRA_2" Name="GeoTrust TrustCenter CodeSigning CA I">
      <CertRoot Type="TBS" Value="172F39BCA3DDA7C6D5169C96B34A5FE7E96FF0BD" />
      <CertPublisher Value="SiSoftware Ltd" />
      <FileAttribRef RuleID="ID_FILEATTRIB_SANDRA_DRIVER_2" />
    </Signer>
    <Signer ID="ID_SIGNER_SPEEDFAN_2" Name="VeriSign Class 3 Code Signing 2004 CA">
      <CertRoot Type="TBS" Value="C7FC1727F5B75A6421A1F95C73BBDB23580C48E5" />
      <CertPublisher Value="Sokno S.R.L." />
    </Signer>
    <Signer ID="ID_SIGNER_VBOX_2" Name="GlobalSign Primary Object Publishing CA">
      <CertRoot Type="TBS" Value="041750993D7C9E063F02DFE74699598640911AAB" />
      <CertPublisher Value="innotek GmbH" />
    </Signer>
    <Signer ID="ID_SIGNER_CPUZ_2" Name="DigiCert EV Code Signing CA (SHA2)">
      <CertRoot Type="TBS" Value="EEC58131DC11CD7F512501B15FDBC6074C603B68CA91F7162D5A042054EDB0CF" />
      <CertPublisher Value="CPUID" />
      <FileAttribRef RuleID="ID_FILEATTRIB_CPUZ_DRIVER_2" />
    </Signer>
    <Signer ID="ID_SIGNER_ELBY_2" Name="GlobalSign Primary Object Publishing CA">
      <CertRoot Type="TBS" Value="041750993D7C9E063F02DFE74699598640911AAB" />
      <CertPublisher Value="Elaborate Bytes AG" />
      <FileAttribRef RuleID="ID_FILEATTRIB_ELBY_DRIVER_2" />
    </Signer>
    <Signer ID="ID_SIGNER_F_1_2" Name="VeriSign Class 3 Code Signing 2010 CA">
      <CertRoot Type="TBS" Value="4843A82ED3B1F2BFBEE9671960E1940C942F688D" />
      <CertPublisher Value="CPUID" />
      <FileAttribRef RuleID="ID_FILEATTRIB_CPUZ_DRIVER_2" />
    </Signer>
    <Signer ID="ID_SIGNER_REALTEK_2" Name="DigiCert EV Code Signing CA">
      <CertRoot Type="TBS" Value="2D54C16A8F8B69CCDEA48D0603C132F547A5CF75" />
      <CertPublisher Value="Realtek Semiconductor Corp." />
      <FileAttribRef RuleID="ID_FILEATTRIB_RTKIO64_DRIVER_2" />
      <FileAttribRef RuleID="ID_FILEATTRIB_RTKIOW10X64_DRIVER_2" />
      <FileAttribRef RuleID="ID_FILEATTRIB_RTKIOW8X64_DRIVER_2" />
    </Signer>
    <Signer ID="ID_SIGNER_REALTEK_2_2" Name="DigiCert EV Code Signing CA (SHA2)">
      <CertRoot Type="TBS" Value="EEC58131DC11CD7F512501B15FDBC6074C603B68CA91F7162D5A042054EDB0CF" />
      <CertPublisher Value="Realtek Semiconductor Corp." />
      <FileAttribRef RuleID="ID_FILEATTRIB_RTKIO64_DRIVER_2" />
      <FileAttribRef RuleID="ID_FILEATTRIB_RTKIOW10X64_DRIVER_2" />
      <FileAttribRef RuleID="ID_FILEATTRIB_RTKIOW8X64_DRIVER_2" />
    </Signer>
    <Signer ID="ID_SIGNER_VERISIGN_2004_2" Name="VeriSign Class 3 Code Signing 2004 CA">
      <CertRoot Type="TBS" Value="C7FC1727F5B75A6421A1F95C73BBDB23580C48E5" />
      <CertPublisher Value="Mitac Technology Corporation" />
      <FileAttribRef RuleID="ID_FILEATTRIB_MTCBSV64_2" />
    </Signer>
    <Signer ID="ID_SIGNER_F_2_2" Name="Microsoft Windows Third Party Component CA 2014">
      <CertRoot Type="TBS" Value="D8BE9E4D9074088EF818BC6F6FB64955E90378B2754155126FEEBBBD969CF0AE" />
      <CertPublisher Value="Microsoft Windows Hardware Compatibility Publisher" />
      <FileAttribRef RuleID="ID_FILEATTRIB_CPUZ_DRIVER_2" />
      <FileAttribRef RuleID="ID_FILEATTRIB_RTKIOW10X64_DRIVER_2" />
      <FileAttribRef RuleID="ID_FILEATTRIB_BS_HWMIO64_2" />
    </Signer>
    <Signer ID="ID_SIGNER_VERISIGN_2009_2" Name="VeriSign Class 3 Code Signing 2009-2 CA">
      <CertRoot Type="TBS" Value="4CDC38C800761463749C3CBD94A12F32E49877BF" />
      <CertPublisher Value="BIOSTAR MICROTECH INT'L CORP" />
      <FileAttribRef RuleID="ID_FILEATTRIB_BSMI_2" />
    </Signer>
    <Signer ID="ID_SIGNER_VERISIGN_BIOSTAR_2" Name="VeriSign Class 3 Code Signing 2004 CA">
      <CertRoot Type="TBS" Value="C7FC1727F5B75A6421A1F95C73BBDB23580C48E5" />
      <CertPublisher Value="BIOSTAR MICROTECH INT'L CORP" />
      <FileAttribRef RuleID="ID_FILEATTRIB_BS_I2CIO_2" />
    </Signer>
    <Signer ID="ID_SIGNER_GLOBALSIGN_G2_MICROSTAR_2" Name="GlobalSign CodeSigning CA - G2">
      <CertRoot Type="TBS" Value="589A7D4DF869395601BA7538A65AFAE8C4616385" />
      <CertPublisher Value="MICRO-STAR INTERNATIONAL CO., LTD." />
      <FileAttribRef RuleID="ID_FILEATTRIB_NTIOLIB_2" />
    </Signer>
    <Signer ID="ID_SIGNER_VERISIGN_TOSHIBA_2" Name="VeriSign Class 3 Code Signing 2010 CA">
      <CertRoot Type="TBS" Value="4843A82ED3B1F2BFBEE9671960E1940C942F688D" />
      <CertPublisher Value="TOSHIBA CORPORATION" />
      <FileAttribRef RuleID="ID_FILEATTRIB_NCHGBIOS2X64_2" />
    </Signer>
    <Signer ID="ID_SIGNER_GLOBALSIGN_MICROSTAR_2" Name="GlobalSign Primary Object Publishing CA">
      <CertRoot Type="TBS" Value="041750993D7C9E063F02DFE74699598640911AAB" />
      <CertPublisher Value="Micro-Star Int'l Co. Ltd." />
      <FileAttribRef RuleID="ID_FILEATTRIB_NTIOLIB_2" />
    </Signer>
    <Signer ID="ID_SIGNER_VERISIGN_INSYDE_2" Name="VeriSign Class 3 Code Signing 2010 CA">
      <CertRoot Type="TBS" Value="4843A82ED3B1F2BFBEE9671960E1940C942F688D" />
      <CertPublisher Value="Insyde Software Corp." />
      <FileAttribRef RuleID="ID_FILEATTRIB_SEGWINDRVX64_2" />
    </Signer>
  </Signers>
  <!--Driver Signing Scenarios-->
  <SigningScenarios>
    <SigningScenario Value="131" ID="ID_SIGNINGSCENARIO_DRIVERS_1" FriendlyName="Auto generated policy on 04-01-2021">
      <ProductSigners>
        <AllowedSigners>
          <AllowedSigner SignerId="ID_SIGNER_MICROSOFT_PRODUCT_1997_0" />
          <AllowedSigner SignerId="ID_SIGNER_MICROSOFT_PRODUCT_2001_0" />
          <AllowedSigner SignerId="ID_SIGNER_MICROSOFT_PRODUCT_2010_0" />
          <AllowedSigner SignerId="ID_SIGNER_MICROSOFT_STANDARD_2011_0" />
          <AllowedSigner SignerId="ID_SIGNER_MICROSOFT_CODEVERIFICATION_2006_0" />
          <AllowedSigner SignerId="ID_SIGNER_DRM_0" />
          <AllowedSigner SignerId="ID_SIGNER_MICROSOFT_FLIGHT_2014_0" />
          <AllowedSigner SignerId="ID_SIGNER_TEST2010_0" />
          <AllowedSigner SignerId="ID_SIGNER_WINDOWS_PRODUCTION_0_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_ELAM_PRODUCTION_0_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_HAL_PRODUCTION_0_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_WHQL_SHA2_0_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_WHQL_SHA1_0_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_WHQL_MD5_0_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_WINDOWS_FLIGHT_ROOT_0_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_ELAM_FLIGHT_0_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_HAL_FLIGHT_0_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_WHQL_FLIGHT_SHA2_0_0_2_1" />
        </AllowedSigners>
        <FileRulesRef>
          <FileRuleRef RuleID="ID_DENY_KD_KMCI_1_1" />
          <FileRuleRef RuleID="ID_ALLOW_ALL_1_2" />
          <FileRuleRef RuleID="ID_DENY_BANDAI_SHA1_2" />
          <FileRuleRef RuleID="ID_DENY_BANDAI_SHA256_2" />
          <FileRuleRef RuleID="ID_DENY_BANDAI_SHA1_PAGE_2" />
          <FileRuleRef RuleID="ID_DENY_BANDAI_SHA256_PAGE_2" />
          <FileRuleRef RuleID="ID_DENY_CAPCOM_SHA1_2" />
          <FileRuleRef RuleID="ID_DENY_CAPCOM_SHA256_2" />
          <FileRuleRef RuleID="ID_DENY_CAPCOM_SHA1_PAGE_2" />
          <FileRuleRef RuleID="ID_DENY_CAPCOM_SHA256_PAGE_2" />
          <FileRuleRef RuleID="ID_DENY_FIDDRV_SHA1_2" />
          <FileRuleRef RuleID="ID_DENY_FIDDRV_SHA256_2" />
          <FileRuleRef RuleID="ID_DENY_FIDDRV_SHA1_PAGE_2" />
          <FileRuleRef RuleID="ID_DENY_FIDDRV_SHA256_PAGE_2" />
          <FileRuleRef RuleID="ID_DENY_FIDDRV64_SHA1_2" />
          <FileRuleRef RuleID="ID_DENY_FIDDRV64_SHA256_2" />
          <FileRuleRef RuleID="ID_DENY_FIDDRV64_SHA1_PAGE_2" />
          <FileRuleRef RuleID="ID_DENY_FIDDRV64_SHA256_PAGE_2" />
          <FileRuleRef RuleID="ID_DENY_FIDPCIDRV_SHA1_2" />
          <FileRuleRef RuleID="ID_DENY_FIDPCIDRV_SHA256_2" />
          <FileRuleRef RuleID="ID_DENY_FIDPCIDRV_SHA1_PAGE_2" />
          <FileRuleRef RuleID="ID_DENY_FIDPCIDRV_SHA256_PAGE_2" />
          <FileRuleRef RuleID="ID_DENY_FIDPCIDRV64_SHA1_2" />
          <FileRuleRef RuleID="ID_DENY_FIDPCIDRV64_SHA256_2" />
          <FileRuleRef RuleID="ID_DENY_FIDPCIDRV64_SHA1_PAGE_2" />
          <FileRuleRef RuleID="ID_DENY_FIDPCIDRV64_SHA256_PAGE_2" />
          <FileRuleRef RuleID="ID_DENY_GDRV_2" />
          <FileRuleRef RuleID="ID_DENY_GLCKIO2_SHA1_2" />
          <FileRuleRef RuleID="ID_DENY_GLCKIO2_SHA256_2" />
          <FileRuleRef RuleID="ID_DENY_GLCKIO2_SHA1_PAGE_2" />
          <FileRuleRef RuleID="ID_DENY_GLCKIO2_SHA256_PAGE_2" />
          <FileRuleRef RuleID="ID_DENY_GVCIDRV64_SHA1_2" />
          <FileRuleRef RuleID="ID_DENY_GVCIDRV64_SHA256_2" />
          <FileRuleRef RuleID="ID_DENY_GVCIDRV64_SHA1_PAGE_2" />
          <FileRuleRef RuleID="ID_DENY_GVCIDRV64_SHA256_PAGE_2" />
          <FileRuleRef RuleID="ID_DENY_WINFLASH64_SHA1_2" />
          <FileRuleRef RuleID="ID_DENY_WINFLASH64_SHA256_2" />
          <FileRuleRef RuleID="ID_DENY_WINFLASH64_SHA1_PAGE_2" />
          <FileRuleRef RuleID="ID_DENY_WINFLASH64_SHA256_PAGE_2" />
          <FileRuleRef RuleID="ID_DENY_AMIFLDRV64_SHA1_2" />
          <FileRuleRef RuleID="ID_DENY_AMIFLDRV64_SHA256C_2" />
          <FileRuleRef RuleID="ID_DENY_AMIFLDRV64_SHA1_PAGE_2" />
          <FileRuleRef RuleID="ID_DENY_AMIFLDRV64_SHA256_PAGE_2" />
          <FileRuleRef RuleID="ID_DENY_ASUPIO64_SHA1F_2" />
          <FileRuleRef RuleID="ID_DENY_ASUPIO64_SHA256_2" />
          <FileRuleRef RuleID="ID_DENY_ASUPIO64_SHA1_PAGE_2" />
          <FileRuleRef RuleID="ID_DENY_ASUPIO64_SHA256_PAGE_2" />
          <FileRuleRef RuleID="ID_DENY_BSFLASH64_SHA1_2" />
          <FileRuleRef RuleID="ID_DENY_BSFLASH64_SHA256_2" />
          <FileRuleRef RuleID="ID_DENY_BSFLASH64_SHA1_PAGE_2" />
          <FileRuleRef RuleID="ID_DENY_BSFLASH64_SHA256_PAGE_2" />
          <FileRuleRef RuleID="ID_DENY_BSHWMIO64_SHA1_2" />
          <FileRuleRef RuleID="ID_DENY_BSHWMIO64_SHA256_2" />
          <FileRuleRef RuleID="ID_DENY_BSHWMIO64_SHA1_PAGE_2" />
          <FileRuleRef RuleID="ID_DENY_BSHWMIO64_SHA256_PAGE_2" />
          <FileRuleRef RuleID="ID_DENY_MSIO64_SHA1_2" />
          <FileRuleRef RuleID="ID_DENY_MSIO64_SHA256_2" />
          <FileRuleRef RuleID="ID_DENY_MSIO64_SHA1_PAGE_2" />
          <FileRuleRef RuleID="ID_DENY_MSIO64_SHA256_PAGE_2" />
          <FileRuleRef RuleID="ID_DENY_PIDDRV_SHA1_2" />
          <FileRuleRef RuleID="ID_DENY_PIDDRV_SHA256_2" />
          <FileRuleRef RuleID="ID_DENY_PIDDRV_SHA1_PAGE_2" />
          <FileRuleRef RuleID="ID_DENY_PIDDRV_SHA256_PAGE_2" />
          <FileRuleRef RuleID="ID_DENY_PIDDRV64_SHA1_2" />
          <FileRuleRef RuleID="ID_DENY_PIDDRV64_SHA256_2" />
          <FileRuleRef RuleID="ID_DENY_PIDDRV64_SHA1_PAGE_2" />
          <FileRuleRef RuleID="ID_DENY_PIDDRV64_SHA256_PAGE_2" />
          <FileRuleRef RuleID="ID_DENY_SEMAV6MSR64_SHA1_2" />
          <FileRuleRef RuleID="ID_DENY_SEMAV6MSR64_SHA256_2" />
          <FileRuleRef RuleID="ID_DENY_SEMAV6MSR64_SHA1_PAGE_2" />
          <FileRuleRef RuleID="ID_DENY_SEMAV6MSR64_SHA256_PAGE_2" />
        </FileRulesRef>
        <DeniedSigners>
          <DeniedSigner SignerId="ID_SIGNER_MIMIKATZ_KERNEL_2" />
          <DeniedSigner SignerId="ID_SIGNER_MIMIKATZ_USER_2" />
          <DeniedSigner SignerId="ID_SIGNER_NOVELL_2" />
          <DeniedSigner SignerId="ID_SIGNER_RWEVERY_2" />
          <DeniedSigner SignerId="ID_SIGNER_SANDRA_2" />
          <DeniedSigner SignerId="ID_SIGNER_SPEEDFAN_2" />
          <DeniedSigner SignerId="ID_SIGNER_VBOX_2" />
          <DeniedSigner SignerId="ID_SIGNER_CPUZ_2" />
          <DeniedSigner SignerId="ID_SIGNER_ELBY_2" />
          <DeniedSigner SignerId="ID_SIGNER_F_1_2" />
          <DeniedSigner SignerId="ID_SIGNER_REALTEK_2" />
          <DeniedSigner SignerId="ID_SIGNER_REALTEK_2_2" />
          <DeniedSigner SignerId="ID_SIGNER_VERISIGN_2004_2" />
          <DeniedSigner SignerId="ID_SIGNER_F_2_2" />
          <DeniedSigner SignerId="ID_SIGNER_VERISIGN_2009_2" />
          <DeniedSigner SignerId="ID_SIGNER_VERISIGN_BIOSTAR_2" />
          <DeniedSigner SignerId="ID_SIGNER_GLOBALSIGN_G2_MICROSTAR_2" />
          <DeniedSigner SignerId="ID_SIGNER_VERISIGN_TOSHIBA_2" />
          <DeniedSigner SignerId="ID_SIGNER_GLOBALSIGN_MICROSTAR_2" />
          <DeniedSigner SignerId="ID_SIGNER_VERISIGN_INSYDE_2" />
        </DeniedSigners>
      </ProductSigners>
    </SigningScenario>
    <SigningScenario Value="12" ID="ID_SIGNINGSCENARIO_WINDOWS" FriendlyName="Auto generated policy on 04-01-2021">
      <ProductSigners>
        <AllowedSigners>
          <AllowedSigner SignerId="ID_SIGNER_MICROSOFT_PRODUCT_1997_UMCI_0" />
          <AllowedSigner SignerId="ID_SIGNER_MICROSOFT_PRODUCT_2001_UMCI_0" />
          <AllowedSigner SignerId="ID_SIGNER_MICROSOFT_PRODUCT_2010_UMCI_0" />
          <AllowedSigner SignerId="ID_SIGNER_MICROSOFT_STANDARD_2011_UMCI_0" />
          <AllowedSigner SignerId="ID_SIGNER_MICROSOFT_CODEVERIFICATION_2006_UMCI_0" />
          <AllowedSigner SignerId="ID_SIGNER_DRM_UMCI_0" />
          <AllowedSigner SignerId="ID_SIGNER_MICROSOFT_FLIGHT_2014_UMCI_0" />
          <AllowedSigner SignerId="ID_SIGNER_STORE_0" />
          <AllowedSigner SignerId="ID_SIGNER_TEST2010_UMCI_0" />
          <AllowedSigner SignerId="ID_SIGNER_F_1_1_0_1" />
          <AllowedSigner SignerId="ID_SIGNER_AUTHROOT_0_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_18C_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_18D_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_18E_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_18F_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_190_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_191_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_192_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_193_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_194_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_195_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_196_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_197_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_198_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_199_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_19A_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_19B_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_19C_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_19D_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_19E_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_19F_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_1A0_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_1A1_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_1A2_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_1A3_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_1A4_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_1A5_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_1A6_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_1A7_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_1A8_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_1A9_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_1AA_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_1AB_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_1AC_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_1AD_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_1AE_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_1AF_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_1B0_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_1B1_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_1B2_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_1B3_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_1B4_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_1B5_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_1B6_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_1B7_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_1B8_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_1B9_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_1BA_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_1BB_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_1BC_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_1BD_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_1BE_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_1BF_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_1C0_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_1C1_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_1C2_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_1C3_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_1C4_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_1C6_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_1C7_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_1C8_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_1C9_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_1CA_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_1CB_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_1CC_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_1CD_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_1CE_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_1CF_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_1D0_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_1D1_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_1D2_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_1D3_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_1D4_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_1D5_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_1D6_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_1D7_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_1D8_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_1D9_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_1DA_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_1DB_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_1DC_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_1DD_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_1DE_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_1DF_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_1E0_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_1E1_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_1E2_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_1E3_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_1E4_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_1E5_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_1E6_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_1E7_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_1E8_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_1E9_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_1EA_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_1EB_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_1EC_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_1ED_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_1EE_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_1EF_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_1F0_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_1F1_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_1F2_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_1F3_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_1F4_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_1F5_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_1F6_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_1F7_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_1F8_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_1F9_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_1FA_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_1FB_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_1FC_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_1FD_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_1FE_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_1FF_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_200_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_201_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_202_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_203_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_204_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_205_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_206_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_207_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_208_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_209_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_20A_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_20B_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_20C_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_20D_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_20E_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_20F_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_210_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_211_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_212_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_213_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_214_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_215_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_216_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_217_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_218_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_219_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_21A_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_21B_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_21C_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_21D_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_21E_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_21F_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_220_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_221_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_222_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_223_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_224_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_225_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_226_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_227_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_228_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_229_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_22A_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_22B_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_22C_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_22D_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_22E_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_22F_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_230_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_231_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_232_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_233_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_234_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_235_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_236_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_237_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_238_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_239_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_23A_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_23B_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_23C_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_23D_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_23E_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_23F_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_240_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_241_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_242_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_244_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_245_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_246_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_247_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_248_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_249_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_24A_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_24B_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_24C_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_24D_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_24E_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_24F_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_250_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_251_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_252_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_253_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_254_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_255_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_256_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_257_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_258_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_259_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_25A_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_25B_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_25C_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_25D_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_25E_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_25F_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_260_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_261_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_262_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_263_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_264_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_265_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_266_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_267_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_268_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_269_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_26A_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_26B_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_26C_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_26D_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_26E_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_26F_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_270_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_271_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_272_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_273_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_274_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_275_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_276_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_277_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_278_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_279_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_27A_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_27B_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_27C_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_27D_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_27E_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_27F_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_280_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_281_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_282_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_283_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_284_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_285_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_286_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_287_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_288_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_289_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_28A_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_28B_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_28C_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_28D_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_28E_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_28F_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_290_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_291_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_292_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_293_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_294_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_295_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_296_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_297_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_298_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_299_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_29A_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_29B_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_29C_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_29D_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_29E_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_29F_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_2A0_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_2A1_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_2A2_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_2A3_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_2A4_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_2A5_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_2A6_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_2A7_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_2A8_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_2A9_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_2AA_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_2AB_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_2AC_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_2AD_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_2AE_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_2AF_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_2B0_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_2B1_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_2B2_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_2B3_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_2B4_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_2B5_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_2B6_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_2B7_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_2B8_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_2B9_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_2BA_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_2BB_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_2BC_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_2BD_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_2BE_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_2BF_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_2C0_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_2C1_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_2C2_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_2C3_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_2C4_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_2C5_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_2C6_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_2C7_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_2C8_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_2C9_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_2CA_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_2CB_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_2CC_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_2CD_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_2CE_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_2CF_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_2D0_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_2D1_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_2D2_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_2D3_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_2D4_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_2D5_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_2D6_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_2D7_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_2D8_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_2D9_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_2DA_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_2DB_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_2DC_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_2DD_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_2DE_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_2DF_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_2E0_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_2E1_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_2E2_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_2E3_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_2E4_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_2E5_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_2E6_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_2E7_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_2E8_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_2E9_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_2EA_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_2EB_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_2EC_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_2ED_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_2EE_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_2EF_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_2F0_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_2F1_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_2F2_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_2F3_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_2F4_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_2F5_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_2F6_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_2F7_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_2F8_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_2F9_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_2FA_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_2FB_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_2FC_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_2FD_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_2FE_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_2FF_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_300_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_301_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_302_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_303_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_304_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_305_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_306_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_307_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_308_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_309_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_30A_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_30B_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_30C_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_30D_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_30E_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_30F_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_310_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_311_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_312_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_313_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_314_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_315_1_2_0_2_1" />
          <AllowedSigner SignerId="ID_SIGNER_S_316_1_2_0_2_1" />
        </AllowedSigners>
        <FileRulesRef>
          <FileRuleRef RuleID="ID_ALLOW_A_1_0_0_1" />
          <FileRuleRef RuleID="ID_ALLOW_A_2_0_0_1" />
          <FileRuleRef RuleID="ID_ALLOW_A_3_0_0_1" />
          <FileRuleRef RuleID="ID_ALLOW_A_4_0_0_1" />
          <FileRuleRef RuleID="ID_DENY_ADDINPROCESS_1_1" />
          <FileRuleRef RuleID="ID_DENY_ADDINPROCESS32_1_1" />
          <FileRuleRef RuleID="ID_DENY_ADDINUTIL_1_1" />
          <FileRuleRef RuleID="ID_DENY_ASPNET_1_1" />
          <FileRuleRef RuleID="ID_DENY_BASH_1_1" />
          <FileRuleRef RuleID="ID_DENY_BGINFO_1_1" />
          <FileRuleRef RuleID="ID_DENY_CBD_1_1" />
          <FileRuleRef RuleID="ID_DENY_CSI_1_1" />
          <FileRuleRef RuleID="ID_DENY_DBGHOST_1_1" />
          <FileRuleRef RuleID="ID_DENY_DBGSVC_1_1" />
          <FileRuleRef RuleID="ID_DENY_DNX_1_1" />
          <FileRuleRef RuleID="ID_DENY_DOTNET_1_1" />
          <FileRuleRef RuleID="ID_DENY_FSI_1_1" />
          <FileRuleRef RuleID="ID_DENY_FSI_ANYCPU_1_1" />
          <FileRuleRef RuleID="ID_DENY_INFINSTALL_1_1" />
          <FileRuleRef RuleID="ID_DENY_KD_1_1" />
          <FileRuleRef RuleID="ID_DENY_KILL_1_1" />
          <FileRuleRef RuleID="ID_DENY_LXSS_1_1" />
          <FileRuleRef RuleID="ID_DENY_LXRUN_1_1" />
          <FileRuleRef RuleID="ID_DENY_MFC40_1_1" />
          <FileRuleRef RuleID="ID_DENY_MS_BUILD_1_1" />
          <FileRuleRef RuleID="ID_DENY_MS_BUILD_FMWK_1_1" />
          <FileRuleRef RuleID="ID_DENY_MWFC_1_1" />
          <FileRuleRef RuleID="ID_DENY_MSBUILD_1_1" />
          <FileRuleRef RuleID="ID_DENY_MSBUILD_DLL_1_1" />
          <FileRuleRef RuleID="ID_DENY_MSHTA_1_1" />
          <FileRuleRef RuleID="ID_DENY_NTKD_1_1" />
          <FileRuleRef RuleID="ID_DENY_NTSD_1_1" />
          <FileRuleRef RuleID="ID_DENY_PWRSHLCUSTOMHOST_1_1" />
          <FileRuleRef RuleID="ID_DENY_RCSI_1_1" />
          <FileRuleRef RuleID="ID_DENY_RUNSCRIPTHELPER_1_1" />
          <FileRuleRef RuleID="ID_DENY_TEXTTRANSFORM_1_1" />
          <FileRuleRef RuleID="ID_DENY_VISUALUIAVERIFY_1_1" />
          <FileRuleRef RuleID="ID_DENY_WFC_1_1" />
          <FileRuleRef RuleID="ID_DENY_WINDBG_1_1" />
          <FileRuleRef RuleID="ID_DENY_WMIC_1_1" />
          <FileRuleRef RuleID="ID_DENY_WSL_1_1" />
          <FileRuleRef RuleID="ID_DENY_WSLCONFIG_1_1" />
          <FileRuleRef RuleID="ID_DENY_WSLHOST_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_1_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_2_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_3_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_4_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_5_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_6_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_7_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_8_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_9_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_10_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_11_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_12_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_13_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_14_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_15_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_16_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_17_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_18_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_19_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_20_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_21_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_22_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_23_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_24_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_25_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_26_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_27_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_28_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_29_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_30_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_31_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_32_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_33_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_34_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_35_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_36_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_37_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_38_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_39_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_40_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_41_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_42_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_43_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_44_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_45_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_46_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_47_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_48_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_49_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_50_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_51_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_52_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_53_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_54_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_55_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_56_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_57_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_58_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_59_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_60_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_61_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_62_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_63_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_64_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_65_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_66_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_67_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_68_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_69_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_70_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_71_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_72_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_73_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_74_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_75_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_76_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_77_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_78_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_79_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_80_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_81_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_82_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_83_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_84_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_85_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_86_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_87_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_88_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_89_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_90_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_91_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_92_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_93_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_94_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_95_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_96_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_97_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_98_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_99_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_100_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_101_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_102_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_103_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_104_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_105_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_106_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_107_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_108_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_109_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_110_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_111_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_112_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_113_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_114_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_115_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_116_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_117_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_118_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_119_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_120_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_121_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_122_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_123_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_124_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_125_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_126_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_127_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_128_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_129_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_130_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_131_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_132_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_133_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_134_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_135_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_136_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_137_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_138_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_139_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_140_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_141_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_142_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_143_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_144_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_145_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_146_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_147_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_148_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_149_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_150_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_151_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_152_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_153_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_154_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_155_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_156_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_157_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_158_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_159_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_160_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_161_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_162_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_163_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_164_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_165_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_166_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_167_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_168_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_169_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_170_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_171_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_172_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_173_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_174_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_175_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_176_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_177_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_178_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_179_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_180_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_181_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_182_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_183_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_184_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_185_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_186_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_187_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_188_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_189_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_190_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_191_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_192_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_193_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_194_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_195_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_196_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_197_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_198_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_199_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_200_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_201_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_202_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_203_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_204_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_205_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_206_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_207_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_208_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_209_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_210_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_211_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_212_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_213_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_214_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_215_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_216_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_217_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_218_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_219_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_220_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_221_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_222_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_223_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_224_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_225_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_226_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_227_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_228_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_229_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_230_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_231_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_232_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_233_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_234_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_235_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_236_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_237_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_238_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_239_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_240_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_241_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_242_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_243_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_244_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_245_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_246_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_247_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_248_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_249_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_250_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_251_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_252_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_253_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_254_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_255_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_256_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_257_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_258_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_259_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_260_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_263_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_264_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_267_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_268_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_271_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_272_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_275_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_276_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_277_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_278_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_279_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_280_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_281_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_282_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_283_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_284_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_285_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_286_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_287_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_288_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_289_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_290_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_293_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_294_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_295_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_296_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_297_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_298_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_299_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_300_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_301_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_302_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_303_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_304_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_305_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_306_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_307_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_308_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_309_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_310_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_311_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_312_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_313_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_314_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_315_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_316_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_317_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_318_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_319_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_320_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_321_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_322_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_323_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_324_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_325_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_326_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_327_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_328_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_329_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_330_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_331_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_332_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_333_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_334_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_335_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_336_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_337_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_338_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_339_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_340_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_341_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_342_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_343_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_344_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_345_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_346_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_347_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_348_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_349_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_350_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_351_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_352_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_353_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_354_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_355_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_356_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_357_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_358_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_359_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_360_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_361_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_362_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_363_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_364_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_365_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_366_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_367_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_368_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_369_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_370_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_371_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_372_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_373_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_374_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_375_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_376_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_377_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_378_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_379_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_380_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_381_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_382_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_383_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_384_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_385_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_386_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_387_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_388_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_389_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_390_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_391_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_392_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_393_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_394_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_395_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_396_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_397_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_398_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_399_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_400_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_401_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_402_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_403_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_404_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_405_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_406_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_411_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_412_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_427_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_428_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_435_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_436_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_441_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_442_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_457_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_458_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_463_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_464_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_465_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_466_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_467_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_468_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_469_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_470_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_471_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_472_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_475_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_476_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_477_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_478_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_483_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_484_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_493_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_494_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_503_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_504_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_507_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_508_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_509_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_510_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_511_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_512_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_515_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_516_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_519_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_520_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_527_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_528_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_529_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_530_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_537_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_538_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_539_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_540_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_541_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_542_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_543_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_544_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_545_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_546_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_547_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_548_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_549_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_550_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_551_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_552_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_553_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_554_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_555_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_556_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_557_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_558_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_559_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_560_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_561_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_562_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_563_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_564_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_565_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_566_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_567_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_568_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_569_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_570_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_571_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_572_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_573_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_574_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_575_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_576_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_577_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_578_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_579_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_580_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_581_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_582_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_583_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_584_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_585_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_586_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_587_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_588_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_589_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_590_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_591_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_592_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_593_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_594_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_595_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_596_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_597_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_598_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_599_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_600_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_601_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_602_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_603_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_604_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_605_1_1" />
          <FileRuleRef RuleID="ID_DENY_D_606_1_1" />
          <FileRuleRef RuleID="ID_ALLOW_ALL_2_2" />
        </FileRulesRef>
      </ProductSigners>
    </SigningScenario>
  </SigningScenarios>
  <UpdatePolicySigners />
  <CiSigners>
    <CiSigner SignerId="ID_SIGNER_STORE_0" />
    <CiSigner SignerId="ID_SIGNER_F_1_1_0_1" />
    <CiSigner SignerId="ID_SIGNER_S_18C_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_18D_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_18E_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_18F_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_190_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_191_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_192_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_193_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_194_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_195_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_196_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_197_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_198_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_199_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_19A_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_19B_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_19C_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_19D_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_19E_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_19F_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_1A0_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_1A1_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_1A2_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_1A3_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_1A4_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_1A5_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_1A6_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_1A7_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_1A8_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_1A9_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_1AA_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_1AB_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_1AC_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_1AD_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_1AE_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_1AF_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_1B0_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_1B1_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_1B2_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_1B3_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_1B4_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_1B5_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_1B6_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_1B7_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_1B8_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_1B9_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_1BA_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_1BB_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_1BC_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_1BD_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_1BE_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_1BF_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_1C0_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_1C1_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_1C2_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_1C3_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_1C4_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_1C6_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_1C7_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_1C8_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_1C9_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_1CA_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_1CB_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_1CC_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_1CD_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_1CE_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_1CF_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_1D0_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_1D1_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_1D2_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_1D3_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_1D4_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_1D5_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_1D6_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_1D7_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_1D8_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_1D9_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_1DA_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_1DB_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_1DC_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_1DD_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_1DE_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_1DF_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_1E0_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_1E1_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_1E2_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_1E3_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_1E4_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_1E5_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_1E6_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_1E7_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_1E8_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_1E9_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_1EA_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_1EB_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_1EC_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_1ED_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_1EE_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_1EF_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_1F0_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_1F1_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_1F2_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_1F3_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_1F4_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_1F5_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_1F6_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_1F7_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_1F8_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_1F9_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_1FA_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_1FB_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_1FC_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_1FD_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_1FE_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_1FF_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_200_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_201_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_202_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_203_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_204_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_205_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_206_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_207_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_208_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_209_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_20A_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_20B_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_20C_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_20D_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_20E_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_20F_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_210_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_211_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_212_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_213_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_214_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_215_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_216_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_217_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_218_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_219_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_21A_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_21B_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_21C_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_21D_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_21E_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_21F_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_220_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_221_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_222_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_223_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_224_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_225_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_226_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_227_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_228_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_229_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_22A_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_22B_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_22C_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_22D_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_22E_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_22F_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_230_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_231_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_232_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_233_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_234_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_235_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_236_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_237_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_238_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_239_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_23A_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_23B_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_23C_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_23D_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_23E_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_23F_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_240_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_241_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_242_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_244_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_245_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_246_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_247_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_248_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_249_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_24A_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_24B_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_24C_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_24D_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_24E_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_24F_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_250_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_251_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_252_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_253_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_254_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_255_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_256_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_257_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_258_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_259_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_25A_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_25B_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_25C_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_25D_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_25E_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_25F_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_260_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_261_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_262_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_263_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_264_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_265_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_266_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_267_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_268_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_269_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_26A_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_26B_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_26C_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_26D_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_26E_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_26F_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_270_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_271_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_272_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_273_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_274_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_275_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_276_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_277_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_278_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_279_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_27A_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_27B_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_27C_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_27D_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_27E_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_27F_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_280_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_281_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_282_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_283_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_284_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_285_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_286_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_287_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_288_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_289_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_28A_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_28B_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_28C_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_28D_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_28E_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_28F_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_290_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_291_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_292_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_293_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_294_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_295_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_296_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_297_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_298_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_299_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_29A_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_29B_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_29C_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_29D_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_29E_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_29F_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_2A0_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_2A1_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_2A2_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_2A3_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_2A4_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_2A5_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_2A6_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_2A7_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_2A8_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_2A9_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_2AA_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_2AB_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_2AC_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_2AD_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_2AE_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_2AF_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_2B0_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_2B1_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_2B2_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_2B3_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_2B4_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_2B5_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_2B6_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_2B7_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_2B8_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_2B9_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_2BA_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_2BB_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_2BC_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_2BD_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_2BE_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_2BF_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_2C0_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_2C1_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_2C2_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_2C3_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_2C4_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_2C5_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_2C6_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_2C7_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_2C8_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_2C9_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_2CA_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_2CB_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_2CC_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_2CD_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_2CE_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_2CF_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_2D0_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_2D1_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_2D2_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_2D3_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_2D4_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_2D5_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_2D6_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_2D7_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_2D8_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_2D9_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_2DA_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_2DB_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_2DC_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_2DD_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_2DE_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_2DF_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_2E0_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_2E1_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_2E2_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_2E3_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_2E4_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_2E5_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_2E6_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_2E7_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_2E8_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_2E9_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_2EA_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_2EB_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_2EC_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_2ED_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_2EE_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_2EF_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_2F0_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_2F1_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_2F2_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_2F3_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_2F4_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_2F5_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_2F6_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_2F7_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_2F8_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_2F9_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_2FA_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_2FB_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_2FC_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_2FD_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_2FE_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_2FF_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_300_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_301_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_302_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_303_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_304_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_305_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_306_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_307_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_308_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_309_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_30A_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_30B_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_30C_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_30D_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_30E_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_30F_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_310_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_311_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_312_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_313_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_314_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_315_1_2_0_2_1" />
    <CiSigner SignerId="ID_SIGNER_S_316_1_2_0_2_1" />
  </CiSigners>
  <HvciOptions>0</HvciOptions>
  <BasePolicyID>{8B43435D-3FE4-4EE3-9684-843BF21BB922}</BasePolicyID>
  <PolicyID>{DF77AED4-F6AF-4CAC-91F2-A00A904FD96F}</PolicyID>
  <Settings>
    <Setting Provider="PolicyInfo" Key="Information" ValueName="Id">
      <Value>
        <String>040121</String>
      </Value>
    </Setting>
    <Setting Provider="PolicyInfo" Key="Information" ValueName="Name">
      <Value>
        <String>AllowMicrosoft_040121</String>
      </Value>
    </Setting>
  </Settings>
</SiPolicy>
```
### Appendix 2: Example Git whitelisting supplementary policy based on publisher and falling back to hash
```xml
<?xml version="1.0" encoding="utf-8"?>
<SiPolicy xmlns="urn:schemas-microsoft-com:sipolicy" PolicyType="Supplemental Policy">
  <VersionEx>10.0.0.0</VersionEx>
  <PlatformID>{2E07F7E4-194C-4D20-B7C9-6F44A6C5A234}</PlatformID>
  <Rules>
    <Rule>
      <Option>Enabled:Unsigned System Integrity Policy</Option>
    </Rule>
    <Rule>
      <Option>Enabled:Audit Mode</Option>
    </Rule>
    <Rule>
      <Option>Enabled:Advanced Boot Options Menu</Option>
    </Rule>
    <Rule>
      <Option>Required:Enforce Store Applications</Option>
    </Rule>
    <Rule>
      <Option>Enabled:UMCI</Option>
    </Rule>
  </Rules>
  <!--EKUS-->
  <EKUs />
  <!--File Rules-->
  <FileRules>
    <Allow ID="ID_ALLOW_A_B52" FriendlyName="C:\Program Files\git\mingw64\libexec\git-core\Bitbucket.Authentication.dll Hash Sha1" Hash="A24F6CEF94A74D07DB6A93359305CD40AF114DE6" />
    <Allow ID="ID_ALLOW_A_B53" FriendlyName="C:\Program Files\git\mingw64\libexec\git-core\Bitbucket.Authentication.dll Hash Sha256" Hash="C992924DBE2909BEEA9D327685A317D0BDDD423D09BC6FC113FB5123384BA717" />
    <Allow ID="ID_ALLOW_A_B54" FriendlyName="C:\Program Files\git\mingw64\libexec\git-core\Bitbucket.Authentication.dll Hash Page Sha1" Hash="A787208D88B2E3BDCFA9D4D6B234D4BDA3F08470" />
    <Allow ID="ID_ALLOW_A_B55" FriendlyName="C:\Program Files\git\mingw64\libexec\git-core\Bitbucket.Authentication.dll Hash Page Sha256" Hash="5AD460711379DB0891F84BD49AC9995429D5E9185074B6CB01DC7B2560760349" />
    <Allow ID="ID_ALLOW_A_B5E" FriendlyName="C:\Program Files\git\mingw64\libexec\git-core\GitHub.Authentication.exe Hash Sha1" Hash="6581D1AACD55D02FEFCF97A0E0F831CE02F65516" />
    <Allow ID="ID_ALLOW_A_B5F" FriendlyName="C:\Program Files\git\mingw64\libexec\git-core\GitHub.Authentication.exe Hash Sha256" Hash="86F083134A42C98776EA249D615EC8F576C3B1BB8714296CD233BD4B1D41313D" />
    <Allow ID="ID_ALLOW_A_B60" FriendlyName="C:\Program Files\git\mingw64\libexec\git-core\GitHub.Authentication.exe Hash Page Sha1" Hash="C85EE6DEFA296437B3C07107CC047E9AFF15577F" />
    <Allow ID="ID_ALLOW_A_B61" FriendlyName="C:\Program Files\git\mingw64\libexec\git-core\GitHub.Authentication.exe Hash Page Sha256" Hash="36BE51CCD8DB137322F1932ADAF439CE55538D767EACCD0202BBC22025921FF5" />
    <Allow ID="ID_ALLOW_A_48A" FriendlyName="C:\Program Files\git\git-bash.exe Hash Sha1" Hash="4782B6F9791404540D4751BCBB495EFAAC0E7F39" />
    <Allow ID="ID_ALLOW_A_48B" FriendlyName="C:\Program Files\git\git-bash.exe Hash Sha256" Hash="31C4E5814A006E98070D22CF15CF643A3F450D907226E9A17F1F1C6709F9E55A" />
    <Allow ID="ID_ALLOW_A_48C" FriendlyName="C:\Program Files\git\usr\share\terminfo\78\xterm.js Hash Sha1" Hash="2847AB9374626B34E208DF7E6E4ACB85F858F609" />
    <Allow ID="ID_ALLOW_A_48D" FriendlyName="C:\Program Files\git\usr\share\terminfo\78\xterm.js Hash Sha256" Hash="2251E380D6DDD02F330EBF5A99F2AF11FBDEC5646E379951D3BC83711348205B" />
    <Allow ID="ID_ALLOW_A_48E" FriendlyName="C:\Program Files\git\usr\share\terminfo\78\xterm.js Hash Authenticode SIP Sha256" Hash="E584DF9098E251F8F3A7FE68085BD93E7AF6F563A798071ADBFF0EBD73DDE8A2" />
    <Allow ID="ID_ALLOW_A_48F" FriendlyName="C:\Program Files\git\usr\libexec\frcode.exe Hash Sha1" Hash="6F664783830953F9808AB467C93CDB445B77B82B" />
    <Allow ID="ID_ALLOW_A_490" FriendlyName="C:\Program Files\git\usr\libexec\frcode.exe Hash Sha256" Hash="B527795900176C2C470B41B4429CAB654D9CA8E335033CB5528768D194694CD8" />
    <Allow ID="ID_ALLOW_A_491" FriendlyName="C:\Program Files\git\usr\libexec\frcode.exe Hash Page Sha1" Hash="D60246772B4B664A47D4721B1F69CB4EA126AA9D" />
    <Allow ID="ID_ALLOW_A_492" FriendlyName="C:\Program Files\git\usr\libexec\frcode.exe Hash Page Sha256" Hash="B8AA1567268649AAC3C3451D162691F1B9D4564DAF6BDD020FE9A1263179D900" />
    <Allow ID="ID_ALLOW_A_493" FriendlyName="C:\Program Files\git\usr\libexec\getprocaddr32.exe Hash Sha1" Hash="DB4FC70DDEDAAA9C0796015AA6912218BD101D58" />
    <Allow ID="ID_ALLOW_A_494" FriendlyName="C:\Program Files\git\usr\libexec\getprocaddr32.exe Hash Sha256" Hash="ACBF7A645A15899AC2968292B954CD6B5E2FD362795173E2A466881683C423C1" />
    <Allow ID="ID_ALLOW_A_495" FriendlyName="C:\Program Files\git\usr\libexec\getprocaddr32.exe Hash Page Sha1" Hash="CF534CE39A1F9D937779D4F5BB17F5E44F5B83A7" />
    <Allow ID="ID_ALLOW_A_496" FriendlyName="C:\Program Files\git\usr\libexec\getprocaddr32.exe Hash Page Sha256" Hash="593CF286B0366FFED8F086069EDDAA64BE93AFAAFDE9F85E5B1D528BAAF828B8" />
    <Allow ID="ID_ALLOW_A_497" FriendlyName="C:\Program Files\git\usr\libexec\getprocaddr64.exe Hash Sha1" Hash="DF699E70196D019A708A21F306AE0C516C914D77" />
    <Allow ID="ID_ALLOW_A_498" FriendlyName="C:\Program Files\git\usr\libexec\getprocaddr64.exe Hash Sha256" Hash="E246DEFF51949DB3D90724B7878E6893CA5E60EA4FA7BED00E76B6212BF4091D" />
    <Allow ID="ID_ALLOW_A_499" FriendlyName="C:\Program Files\git\usr\libexec\getprocaddr64.exe Hash Page Sha1" Hash="EC11A25A2BE47DE08CF8E546BB64C2842DB77367" />
    <Allow ID="ID_ALLOW_A_49A" FriendlyName="C:\Program Files\git\usr\libexec\getprocaddr64.exe Hash Page Sha256" Hash="1DF077F0C215814DAC71F2BEFBB3E82FC9C53593356E6152B90FDD8362D1E80A" />
    <Allow ID="ID_ALLOW_A_49B" FriendlyName="C:\Program Files\git\usr\libexec\p11-kit\p11-kit-remote.exe Hash Sha1" Hash="D7FA5DE3758D69690A123BA6AA3131FD4BFB00F3" />
    <Allow ID="ID_ALLOW_A_49C" FriendlyName="C:\Program Files\git\usr\libexec\p11-kit\p11-kit-remote.exe Hash Sha256" Hash="29DD1827C44B4FE4A1F40D3F3DE69DDA538AA4B5331DBE9CEB0BF35F592813DA" />
    <Allow ID="ID_ALLOW_A_49D" FriendlyName="C:\Program Files\git\usr\libexec\p11-kit\p11-kit-remote.exe Hash Page Sha1" Hash="FCD5D9E9243F8F05261098CBD0C2A10268CAB5A4" />
    <Allow ID="ID_ALLOW_A_49E" FriendlyName="C:\Program Files\git\usr\libexec\p11-kit\p11-kit-remote.exe Hash Page Sha256" Hash="E0BB4FCC48670ADC953E6451460EF20D78D1B037A843EBC2C10DD109E9D511C9" />
    <Allow ID="ID_ALLOW_A_49F" FriendlyName="C:\Program Files\git\usr\libexec\p11-kit\p11-kit-server.exe Hash Sha1" Hash="77027A9967FE4F0A21F780D5AEB8ACE9318C338B" />
    <Allow ID="ID_ALLOW_A_4A0" FriendlyName="C:\Program Files\git\usr\libexec\p11-kit\p11-kit-server.exe Hash Sha256" Hash="3E80505A7B9290EF23D72F610A4060BCD20FFA27864F68899A0E501A195BDD32" />
    <Allow ID="ID_ALLOW_A_4A1" FriendlyName="C:\Program Files\git\usr\libexec\p11-kit\p11-kit-server.exe Hash Page Sha1" Hash="991C4E7DDBC75CBD5704ABD9B4F4E398D61A588B" />
    <Allow ID="ID_ALLOW_A_4A2" FriendlyName="C:\Program Files\git\usr\libexec\p11-kit\p11-kit-server.exe Hash Page Sha256" Hash="FB554A6FD4622717BDFD99689E6E2AB479B76625A56F4C0E3BDC46064D0A301E" />
    <Allow ID="ID_ALLOW_A_4A3" FriendlyName="C:\Program Files\git\usr\lib\tar\rmt.exe Hash Sha1" Hash="91C5FFF6BD989A554EAC81A1791A124752AB9805" />
    <Allow ID="ID_ALLOW_A_4A4" FriendlyName="C:\Program Files\git\usr\lib\tar\rmt.exe Hash Sha256" Hash="2CA781384D8862821E432F0D00CB40E805A291EF2CE0EAE5EB32CC2F3A4CD7D8" />
    <Allow ID="ID_ALLOW_A_4A5" FriendlyName="C:\Program Files\git\usr\lib\tar\rmt.exe Hash Page Sha1" Hash="F6C9003568FDAF3F69063EE2B71D11F657524638" />
    <Allow ID="ID_ALLOW_A_4A6" FriendlyName="C:\Program Files\git\usr\lib\tar\rmt.exe Hash Page Sha256" Hash="0B441B7BCFDE89A40CD328E1AEC8D1A0368A7D582EA5C2AF3397543C23FBD8CB" />
    <Allow ID="ID_ALLOW_A_4A7" FriendlyName="C:\Program Files\git\usr\lib\ssh\sftp-server.exe Hash Sha1" Hash="3960846F583025A84ED7E17F5B122BDE3C7DF14A" />
    <Allow ID="ID_ALLOW_A_4A8" FriendlyName="C:\Program Files\git\usr\lib\ssh\sftp-server.exe Hash Sha256" Hash="DBAF28CA8D9019AA8DC788C8DF3FB1837001CDFB26F7A8BBD5199D1FEFDF1769" />
    <Allow ID="ID_ALLOW_A_4A9" FriendlyName="C:\Program Files\git\usr\lib\ssh\sftp-server.exe Hash Page Sha1" Hash="1A797431634EA52804623FD805040C5D40B8561B" />
    <Allow ID="ID_ALLOW_A_4AA" FriendlyName="C:\Program Files\git\usr\lib\ssh\sftp-server.exe Hash Page Sha256" Hash="C59A1936A8387E6D8D5039D2C49B654E5102ADE624123A9DC7958BF9533C7881" />
    <Allow ID="ID_ALLOW_A_4AB" FriendlyName="C:\Program Files\git\usr\lib\ssh\ssh-keysign.exe Hash Sha1" Hash="EB64B95FBE7F9914F6C494B85F89513BF58706AB" />
    <Allow ID="ID_ALLOW_A_4AC" FriendlyName="C:\Program Files\git\usr\lib\ssh\ssh-keysign.exe Hash Sha256" Hash="B80E46D986A28017067C64B314AFBFC67A6553113D4FE36B5EDDF80A24117301" />
    <Allow ID="ID_ALLOW_A_4AD" FriendlyName="C:\Program Files\git\usr\lib\ssh\ssh-keysign.exe Hash Page Sha1" Hash="D599FC7B04DB40C52893B80F6386FADF8BA7B6C7" />
    <Allow ID="ID_ALLOW_A_4AE" FriendlyName="C:\Program Files\git\usr\lib\ssh\ssh-keysign.exe Hash Page Sha256" Hash="C98F95ABBC99AFCF3CC1D506E1DB291BAAF0A3020336A980B346D0146ADB617C" />
    <Allow ID="ID_ALLOW_A_4AF" FriendlyName="C:\Program Files\git\usr\lib\ssh\ssh-pkcs11-helper.exe Hash Sha1" Hash="6B1581DE5037B8E587AE45BE63AEB3C5B6457822" />
    <Allow ID="ID_ALLOW_A_4B0" FriendlyName="C:\Program Files\git\usr\lib\ssh\ssh-pkcs11-helper.exe Hash Sha256" Hash="3A8E9E535A4C792BF46EBD733F236D8FE7522EF0178E0E12242C766103940F5A" />
    <Allow ID="ID_ALLOW_A_4B1" FriendlyName="C:\Program Files\git\usr\lib\ssh\ssh-pkcs11-helper.exe Hash Page Sha1" Hash="04CBECC3335867B0B794903DF6C84EC279D9C00E" />
    <Allow ID="ID_ALLOW_A_4B2" FriendlyName="C:\Program Files\git\usr\lib\ssh\ssh-pkcs11-helper.exe Hash Page Sha256" Hash="719B7D7A202D9DAF4892D96931221DF567C1C8C0DFE8A830FD9A9293C68F9A0F" />
    <Allow ID="ID_ALLOW_A_4B3" FriendlyName="C:\Program Files\git\usr\lib\ssh\ssh-sk-helper.exe Hash Sha1" Hash="896707937B23932824028BB39D69FD3000839FC2" />
    <Allow ID="ID_ALLOW_A_4B4" FriendlyName="C:\Program Files\git\usr\lib\ssh\ssh-sk-helper.exe Hash Sha256" Hash="3A92178D3CB818EBD069B54C2DA31A58D10B47519B8D832FE8AF727C58E33747" />
    <Allow ID="ID_ALLOW_A_4B5" FriendlyName="C:\Program Files\git\usr\lib\ssh\ssh-sk-helper.exe Hash Page Sha1" Hash="C7BCC8CB3078AC03AF1E78EB942DA5E50D04C5DA" />
    <Allow ID="ID_ALLOW_A_4B6" FriendlyName="C:\Program Files\git\usr\lib\ssh\ssh-sk-helper.exe Hash Page Sha256" Hash="84B21FFA84ED3EDD7712EF5B62AF50CE18F62DD114B6D6B1394ED14F8F8395AE" />
    <Allow ID="ID_ALLOW_A_4B7" FriendlyName="C:\Program Files\git\usr\lib\sasl2\msys-anonymous-3.dll Hash Sha1" Hash="207BE024F6083C1EC68AD0900956D5EC90F14738" />
    <Allow ID="ID_ALLOW_A_4B8" FriendlyName="C:\Program Files\git\usr\lib\sasl2\msys-anonymous-3.dll Hash Sha256" Hash="7DFE604D3FECE8057B94EB7A8C43FB9A33DA5320440660EC9824A6804C6FE950" />
    <Allow ID="ID_ALLOW_A_4B9" FriendlyName="C:\Program Files\git\usr\lib\sasl2\msys-anonymous-3.dll Hash Page Sha1" Hash="C46D1BB535119F71107204D2C94BAE5F72E8CC52" />
    <Allow ID="ID_ALLOW_A_4BA" FriendlyName="C:\Program Files\git\usr\lib\sasl2\msys-anonymous-3.dll Hash Page Sha256" Hash="E85D5B3905EAB9DD4EDCAB285B4E45B7610645D4C35D56E46A06531E8109D041" />
    <Allow ID="ID_ALLOW_A_4BB" FriendlyName="C:\Program Files\git\usr\lib\sasl2\msys-crammd5-3.dll Hash Sha1" Hash="32BCDEE88BD8012AA964C4FF7F1223392E221486" />
    <Allow ID="ID_ALLOW_A_4BC" FriendlyName="C:\Program Files\git\usr\lib\sasl2\msys-crammd5-3.dll Hash Sha256" Hash="32A1DCB02161C88583B9890CFF1909166F8AD802435330FB0CD88D22BEAE6DA1" />
    <Allow ID="ID_ALLOW_A_4BD" FriendlyName="C:\Program Files\git\usr\lib\sasl2\msys-crammd5-3.dll Hash Page Sha1" Hash="69ADACA7B8F24FD1DAA4694CD258A474E00F2FA3" />
    <Allow ID="ID_ALLOW_A_4BE" FriendlyName="C:\Program Files\git\usr\lib\sasl2\msys-crammd5-3.dll Hash Page Sha256" Hash="356374B64CB53E2709441278D50C500B468FE899B49C78AA2D083E0A37AB47B0" />
    <Allow ID="ID_ALLOW_A_4BF" FriendlyName="C:\Program Files\git\usr\lib\sasl2\msys-digestmd5-3.dll Hash Sha1" Hash="3A099AFDA21C1EE0EAB3FFD9E275150DC31295D4" />
    <Allow ID="ID_ALLOW_A_4C0" FriendlyName="C:\Program Files\git\usr\lib\sasl2\msys-digestmd5-3.dll Hash Sha256" Hash="DB07D95D2F7D83DDECE38BAA9D4B4628E65CD48BA5368EE4CCA3EA5746056E82" />
    <Allow ID="ID_ALLOW_A_4C1" FriendlyName="C:\Program Files\git\usr\lib\sasl2\msys-digestmd5-3.dll Hash Page Sha1" Hash="682B84D9D824AC462411348C0F6A70B7E1F1A20A" />
    <Allow ID="ID_ALLOW_A_4C2" FriendlyName="C:\Program Files\git\usr\lib\sasl2\msys-digestmd5-3.dll Hash Page Sha256" Hash="48F5F25A11D4CD37EA29DAE2816047AC4B968BBBA550E18930B55DC32A574CF7" />
    <Allow ID="ID_ALLOW_A_4C3" FriendlyName="C:\Program Files\git\usr\lib\sasl2\msys-gs2-3.dll Hash Sha1" Hash="BC1077DEAF03802C3926085E516DC61C248DD44D" />
    <Allow ID="ID_ALLOW_A_4C4" FriendlyName="C:\Program Files\git\usr\lib\sasl2\msys-gs2-3.dll Hash Sha256" Hash="E91D6C331056270930B52DF8D41710B0829852D717C042F30002EB56DEEC4CB7" />
    <Allow ID="ID_ALLOW_A_4C5" FriendlyName="C:\Program Files\git\usr\lib\sasl2\msys-gs2-3.dll Hash Page Sha1" Hash="A07C6713482221D9AABC2315CED0E18661BD03F7" />
    <Allow ID="ID_ALLOW_A_4C6" FriendlyName="C:\Program Files\git\usr\lib\sasl2\msys-gs2-3.dll Hash Page Sha256" Hash="39678D96B4C80FC923654BBCF6D1A88A6E883ACE43AE3CBEF5E3BB5A3878FFC7" />
    <Allow ID="ID_ALLOW_A_4C7" FriendlyName="C:\Program Files\git\usr\lib\sasl2\msys-gssapiv2-3.dll Hash Sha1" Hash="088ACF08A149BB1EE83DE82FD7894F0EC52714AE" />
    <Allow ID="ID_ALLOW_A_4C8" FriendlyName="C:\Program Files\git\usr\lib\sasl2\msys-gssapiv2-3.dll Hash Sha256" Hash="C10CE45B41244F9B33C129EA5F0D96AA9D7DD958B7D6560D78178259CB63418F" />
    <Allow ID="ID_ALLOW_A_4C9" FriendlyName="C:\Program Files\git\usr\lib\sasl2\msys-gssapiv2-3.dll Hash Page Sha1" Hash="9480491D470322517674A528D0E9F9E1F056FCA7" />
    <Allow ID="ID_ALLOW_A_4CA" FriendlyName="C:\Program Files\git\usr\lib\sasl2\msys-gssapiv2-3.dll Hash Page Sha256" Hash="E26A9F78DD31B66007571C363FC2FF440C2C5E04B5CA7B3083AFDBBC401CF823" />
    <Allow ID="ID_ALLOW_A_4CB" FriendlyName="C:\Program Files\git\usr\lib\sasl2\msys-otp-3.dll Hash Sha1" Hash="551D028AFA1226F811FE7B3FF9C68D3CE81AED8A" />
    <Allow ID="ID_ALLOW_A_4CC" FriendlyName="C:\Program Files\git\usr\lib\sasl2\msys-otp-3.dll Hash Sha256" Hash="0E6DF1B6E249F55AD0AFF5D81161F9E6A77C3D35C244E4AB60862E607C8CCD5E" />
    <Allow ID="ID_ALLOW_A_4CD" FriendlyName="C:\Program Files\git\usr\lib\sasl2\msys-otp-3.dll Hash Page Sha1" Hash="CDCEB362A1283B8842BEE069EF43A891A99E8C72" />
    <Allow ID="ID_ALLOW_A_4CE" FriendlyName="C:\Program Files\git\usr\lib\sasl2\msys-otp-3.dll Hash Page Sha256" Hash="5FE1922247EFD90F1DD5489D5F9C8E8C305B0C809F58664D771664E9B3A28C75" />
    <Allow ID="ID_ALLOW_A_4CF" FriendlyName="C:\Program Files\git\usr\lib\sasl2\msys-plain-3.dll Hash Sha1" Hash="C151FC104117795E1A6AA1D3CCD9AB8D0412860A" />
    <Allow ID="ID_ALLOW_A_4D0" FriendlyName="C:\Program Files\git\usr\lib\sasl2\msys-plain-3.dll Hash Sha256" Hash="0EF95C5840F0481607933CC215BBC2FEECEB3A28C6278C25AE8BCBA3EEBB014E" />
    <Allow ID="ID_ALLOW_A_4D1" FriendlyName="C:\Program Files\git\usr\lib\sasl2\msys-plain-3.dll Hash Page Sha1" Hash="269F7524D2BCD8A2BB998E1DBA6729DA6C8DAD48" />
    <Allow ID="ID_ALLOW_A_4D2" FriendlyName="C:\Program Files\git\usr\lib\sasl2\msys-plain-3.dll Hash Page Sha256" Hash="6DF3A4C56517EB0463C220480EFAA32426D0057569F83340049B1ABDA2CC5D15" />
    <Allow ID="ID_ALLOW_A_4D3" FriendlyName="C:\Program Files\git\usr\lib\sasl2\msys-scram-3.dll Hash Sha1" Hash="4F2F9C409E49FF8110F3A764032A659C3BD37366" />
    <Allow ID="ID_ALLOW_A_4D4" FriendlyName="C:\Program Files\git\usr\lib\sasl2\msys-scram-3.dll Hash Sha256" Hash="584D4164D4902F02ECF355BD483A7342E51CE50AB5EB4771923FF855E4CFBC3C" />
    <Allow ID="ID_ALLOW_A_4D5" FriendlyName="C:\Program Files\git\usr\lib\sasl2\msys-scram-3.dll Hash Page Sha1" Hash="ED335A092EE901CE300C0DE266D1EF2B4C6348AF" />
    <Allow ID="ID_ALLOW_A_4D6" FriendlyName="C:\Program Files\git\usr\lib\sasl2\msys-scram-3.dll Hash Page Sha256" Hash="2D5FE8FC7E16D242CE507B90C516FEF77FFB97113420A8F7E467667F1022A6A6" />
    <Allow ID="ID_ALLOW_A_4D7" FriendlyName="C:\Program Files\git\usr\lib\pkcs11\p11-kit-trust.dll Hash Sha1" Hash="44EECD3C0BBE39A5D6FA2B704FB1CDD305B984E8" />
    <Allow ID="ID_ALLOW_A_4D8" FriendlyName="C:\Program Files\git\usr\lib\pkcs11\p11-kit-trust.dll Hash Sha256" Hash="5EE8718F691DA7E18543612D29A07FE664B36958B4F31D3A62FDCE88DB71C251" />
    <Allow ID="ID_ALLOW_A_4D9" FriendlyName="C:\Program Files\git\usr\lib\pkcs11\p11-kit-trust.dll Hash Page Sha1" Hash="7D49508709CF11F63E9D357955FF45E5247B467C" />
    <Allow ID="ID_ALLOW_A_4DA" FriendlyName="C:\Program Files\git\usr\lib\pkcs11\p11-kit-trust.dll Hash Page Sha256" Hash="139C970B7A9DD3EBC6A722A1D18647CB7433C3956F5690296A9B0D9072F2A4AD" />
    <Allow ID="ID_ALLOW_A_4DB" FriendlyName="C:\Program Files\git\usr\lib\perl5\vendor_perl\auto\Term\ReadKey\ReadKey.dll Hash Sha1" Hash="D0255B1DE82F2473DB4EDD92BF10663B6C6EAF27" />
    <Allow ID="ID_ALLOW_A_4DC" FriendlyName="C:\Program Files\git\usr\lib\perl5\vendor_perl\auto\Term\ReadKey\ReadKey.dll Hash Sha256" Hash="2AC2594B3AD2D1BC0B0C6B04ECD7B45F4DB08DFB31D583BD661C8CD937763738" />
    <Allow ID="ID_ALLOW_A_4DD" FriendlyName="C:\Program Files\git\usr\lib\perl5\vendor_perl\auto\Term\ReadKey\ReadKey.dll Hash Page Sha1" Hash="58E60F040FC9C1AAE05DFD8CC6774AC40122FE59" />
    <Allow ID="ID_ALLOW_A_4DE" FriendlyName="C:\Program Files\git\usr\lib\perl5\vendor_perl\auto\Term\ReadKey\ReadKey.dll Hash Page Sha256" Hash="A7CCF9DF7D2BE21C6818AE593517959A7FC4B6672945DD34D5D871EF77A5EAE5" />
    <Allow ID="ID_ALLOW_A_4DF" FriendlyName="C:\Program Files\git\usr\lib\perl5\vendor_perl\auto\SVN\_Wc\_Wc.dll Hash Sha1" Hash="B231F686D481EAF59198DA784077F927A1DFE319" />
    <Allow ID="ID_ALLOW_A_4E0" FriendlyName="C:\Program Files\git\usr\lib\perl5\vendor_perl\auto\SVN\_Wc\_Wc.dll Hash Sha256" Hash="D7C509799A335A180D03B7E76A9C5659BEB31639FC5BB542A2E0F48B58334644" />
    <Allow ID="ID_ALLOW_A_4E1" FriendlyName="C:\Program Files\git\usr\lib\perl5\vendor_perl\auto\SVN\_Wc\_Wc.dll Hash Page Sha1" Hash="207069212D41030DE659F3847E2634D5157DA72B" />
    <Allow ID="ID_ALLOW_A_4E2" FriendlyName="C:\Program Files\git\usr\lib\perl5\vendor_perl\auto\SVN\_Wc\_Wc.dll Hash Page Sha256" Hash="E490B7016FBEE94FBA68BD53773571A15C6AEDB681AD0315909049FFCC72CA8A" />
    <Allow ID="ID_ALLOW_A_4E3" FriendlyName="C:\Program Files\git\usr\lib\perl5\vendor_perl\auto\SVN\_Repos\_Repos.dll Hash Sha1" Hash="137878130AAAB198DCE10F51EF6C964EF0CA5F69" />
    <Allow ID="ID_ALLOW_A_4E4" FriendlyName="C:\Program Files\git\usr\lib\perl5\vendor_perl\auto\SVN\_Repos\_Repos.dll Hash Sha256" Hash="59AC0BE5F59AF2185D604594474922DEBC49B47CF10D1596B40A7430112692C5" />
    <Allow ID="ID_ALLOW_A_4E5" FriendlyName="C:\Program Files\git\usr\lib\perl5\vendor_perl\auto\SVN\_Repos\_Repos.dll Hash Page Sha1" Hash="C0C6AD6610B5AF6AB6582E2D9AC4B1A5126787DC" />
    <Allow ID="ID_ALLOW_A_4E6" FriendlyName="C:\Program Files\git\usr\lib\perl5\vendor_perl\auto\SVN\_Repos\_Repos.dll Hash Page Sha256" Hash="C5246D6E58694B27A5D5A663083494F4636FA18D21A6A476775BD501386BFC5B" />
    <Allow ID="ID_ALLOW_A_4E7" FriendlyName="C:\Program Files\git\usr\lib\perl5\vendor_perl\auto\SVN\_Ra\_Ra.dll Hash Sha1" Hash="384F912FE5F6A69D6A478DF494E29B8BC68094B2" />
    <Allow ID="ID_ALLOW_A_4E8" FriendlyName="C:\Program Files\git\usr\lib\perl5\vendor_perl\auto\SVN\_Ra\_Ra.dll Hash Sha256" Hash="F80E726D05D036E106A5745B3ED3A9AAA2599A376B97199C43803AD3736A3931" />
    <Allow ID="ID_ALLOW_A_4E9" FriendlyName="C:\Program Files\git\usr\lib\perl5\vendor_perl\auto\SVN\_Ra\_Ra.dll Hash Page Sha1" Hash="FE59007B4746220A4F4E72020E9A1F8E5F195BB3" />
    <Allow ID="ID_ALLOW_A_4EA" FriendlyName="C:\Program Files\git\usr\lib\perl5\vendor_perl\auto\SVN\_Ra\_Ra.dll Hash Page Sha256" Hash="F519E66DB10341E4E24A233465635799D33259A40B8DFE2BC1AFFFFFF4AD44BD" />
    <Allow ID="ID_ALLOW_A_4EB" FriendlyName="C:\Program Files\git\usr\lib\perl5\vendor_perl\auto\SVN\_Fs\_Fs.dll Hash Sha1" Hash="919520A3255AF61881677D83B422E2097B50A58A" />
    <Allow ID="ID_ALLOW_A_4EC" FriendlyName="C:\Program Files\git\usr\lib\perl5\vendor_perl\auto\SVN\_Fs\_Fs.dll Hash Sha256" Hash="D62615B91514C30BA31B96291A01BFF490F9152B82E2882EB5ED7637D6E56990" />
    <Allow ID="ID_ALLOW_A_4ED" FriendlyName="C:\Program Files\git\usr\lib\perl5\vendor_perl\auto\SVN\_Fs\_Fs.dll Hash Page Sha1" Hash="35371386CE35A714A91D294EBC4E0CE602E3F614" />
    <Allow ID="ID_ALLOW_A_4EE" FriendlyName="C:\Program Files\git\usr\lib\perl5\vendor_perl\auto\SVN\_Fs\_Fs.dll Hash Page Sha256" Hash="333FF7B5145CBC544DF54149388E90D079CF59FFBDC14058B0B908F228E86559" />
    <Allow ID="ID_ALLOW_A_4EF" FriendlyName="C:\Program Files\git\usr\lib\perl5\vendor_perl\auto\SVN\_Delta\_Delta.dll Hash Sha1" Hash="E3EC5434F0FA02DDE0947747EB1C74BD823372E3" />
    <Allow ID="ID_ALLOW_A_4F0" FriendlyName="C:\Program Files\git\usr\lib\perl5\vendor_perl\auto\SVN\_Delta\_Delta.dll Hash Sha256" Hash="91BDE369DC8517A229BCF3F62A734F116ED2E838AE434EE7A0F0EEB84D594AEB" />
    <Allow ID="ID_ALLOW_A_4F1" FriendlyName="C:\Program Files\git\usr\lib\perl5\vendor_perl\auto\SVN\_Delta\_Delta.dll Hash Page Sha1" Hash="B6CB85859A003A1B6D91AB5F192A98706CAAD6ED" />
    <Allow ID="ID_ALLOW_A_4F2" FriendlyName="C:\Program Files\git\usr\lib\perl5\vendor_perl\auto\SVN\_Delta\_Delta.dll Hash Page Sha256" Hash="4DEB7079F6C38A6266CDCA6CF9E989580B5845B90982084E6B0ACFE8742C3B08" />
    <Allow ID="ID_ALLOW_A_4F3" FriendlyName="C:\Program Files\git\usr\lib\perl5\vendor_perl\auto\SVN\_Core\_Core.dll Hash Sha1" Hash="C594A12F52E27392684D1BB861E737584DEEFDEC" />
    <Allow ID="ID_ALLOW_A_4F4" FriendlyName="C:\Program Files\git\usr\lib\perl5\vendor_perl\auto\SVN\_Core\_Core.dll Hash Sha256" Hash="9FB5BA6229849F50B88509ED59FCE9BB507628F6E998ECF9031AFC6ECD1016C4" />
    <Allow ID="ID_ALLOW_A_4F5" FriendlyName="C:\Program Files\git\usr\lib\perl5\vendor_perl\auto\SVN\_Core\_Core.dll Hash Page Sha1" Hash="30ED8A39132D63A90ADD8DFD55477B9B30D6ABC7" />
    <Allow ID="ID_ALLOW_A_4F6" FriendlyName="C:\Program Files\git\usr\lib\perl5\vendor_perl\auto\SVN\_Core\_Core.dll Hash Page Sha256" Hash="9E33399CBD824BF72EDC9FC5650D3C3042760AE7D50A598665378B7C0A5B00D3" />
    <Allow ID="ID_ALLOW_A_4F7" FriendlyName="C:\Program Files\git\usr\lib\perl5\vendor_perl\auto\SVN\_Client\_Client.dll Hash Sha1" Hash="DEAC7777AC55E111826595B342966102D7BAC927" />
    <Allow ID="ID_ALLOW_A_4F8" FriendlyName="C:\Program Files\git\usr\lib\perl5\vendor_perl\auto\SVN\_Client\_Client.dll Hash Sha256" Hash="7CD898712D8EB5B717FB2AB410565DDFF72138C90E04B146BA855841B0483F3B" />
    <Allow ID="ID_ALLOW_A_4F9" FriendlyName="C:\Program Files\git\usr\lib\perl5\vendor_perl\auto\SVN\_Client\_Client.dll Hash Page Sha1" Hash="E9DACBE21CA17A9EC6285E40B623CCEEB74807C9" />
    <Allow ID="ID_ALLOW_A_4FA" FriendlyName="C:\Program Files\git\usr\lib\perl5\vendor_perl\auto\SVN\_Client\_Client.dll Hash Page Sha256" Hash="D7DF1D4FD9A76999211D1A7008EED4EFAB35D8C5148F5FFC435C6F9B8745F5FE" />
    <Allow ID="ID_ALLOW_A_4FB" FriendlyName="C:\Program Files\git\usr\lib\perl5\vendor_perl\auto\Net\SSLeay\SSLeay.dll Hash Sha1" Hash="2F473A670AA297812B8D1E27EF30FB6F24F43CED" />
    <Allow ID="ID_ALLOW_A_4FC" FriendlyName="C:\Program Files\git\usr\lib\perl5\vendor_perl\auto\Net\SSLeay\SSLeay.dll Hash Sha256" Hash="A432152098745590ACA548345BD58FA0E73E3F596ED4F79A0B8755696301CB8B" />
    <Allow ID="ID_ALLOW_A_4FD" FriendlyName="C:\Program Files\git\usr\lib\perl5\vendor_perl\auto\Net\SSLeay\SSLeay.dll Hash Page Sha1" Hash="DFEA5CEB8C217EA9DF942D1B7E99345C772EB88A" />
    <Allow ID="ID_ALLOW_A_4FE" FriendlyName="C:\Program Files\git\usr\lib\perl5\vendor_perl\auto\Net\SSLeay\SSLeay.dll Hash Page Sha256" Hash="77B4BAFD02A072A2B6D5C7AD2E78025749048C48A2FC355FEEF15BE9E313695A" />
    <Allow ID="ID_ALLOW_A_4FF" FriendlyName="C:\Program Files\git\usr\lib\perl5\vendor_perl\auto\HTML\Parser\Parser.dll Hash Sha1" Hash="23E231101D9939FAFCBE8679BB77F847F827BAB1" />
    <Allow ID="ID_ALLOW_A_500" FriendlyName="C:\Program Files\git\usr\lib\perl5\vendor_perl\auto\HTML\Parser\Parser.dll Hash Sha256" Hash="1B9DA322B06CF02D372C3AABAA65DBA2D64C4A48E1D0B6D7A9EC2BFC62E64EEF" />
    <Allow ID="ID_ALLOW_A_501" FriendlyName="C:\Program Files\git\usr\lib\perl5\vendor_perl\auto\HTML\Parser\Parser.dll Hash Page Sha1" Hash="4A0E25AFE0A12E50FB7BDFADF6E9CB63EA8A4B0E" />
    <Allow ID="ID_ALLOW_A_502" FriendlyName="C:\Program Files\git\usr\lib\perl5\vendor_perl\auto\HTML\Parser\Parser.dll Hash Page Sha256" Hash="DAE9BF65311FEF200B35F70229D23FD78275F6C186246862B98366E6D2779775" />
    <Allow ID="ID_ALLOW_A_503" FriendlyName="C:\Program Files\git\usr\lib\perl5\vendor_perl\auto\Clone\Clone.dll Hash Sha1" Hash="E33D682E192F6954C92CB7F19BC25D8F89A12CF5" />
    <Allow ID="ID_ALLOW_A_504" FriendlyName="C:\Program Files\git\usr\lib\perl5\vendor_perl\auto\Clone\Clone.dll Hash Sha256" Hash="4E28B6AFD004B5A71C0252D7E0C4B14B4E4FDA42B8953C4A21800601CD893A62" />
    <Allow ID="ID_ALLOW_A_505" FriendlyName="C:\Program Files\git\usr\lib\perl5\vendor_perl\auto\Clone\Clone.dll Hash Page Sha1" Hash="06F2BF9B3F055ADEA9A912B59220630C7DBA0027" />
    <Allow ID="ID_ALLOW_A_506" FriendlyName="C:\Program Files\git\usr\lib\perl5\vendor_perl\auto\Clone\Clone.dll Hash Page Sha256" Hash="DE5334265B188D330635A4625560861618F9F8BE215EB6F25BC2A557E93B0622" />
    <Allow ID="ID_ALLOW_A_507" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\CORE\msys-perl5_32.dll Hash Sha1" Hash="DF9337A7571E30742F4C5989F14B8940C7E2DA80" />
    <Allow ID="ID_ALLOW_A_508" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\CORE\msys-perl5_32.dll Hash Sha256" Hash="0CD0E5FBBAB3F5E27FC6E77D42E9D63C988EB36AC02700DC51658A4A8FB6E065" />
    <Allow ID="ID_ALLOW_A_509" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\CORE\msys-perl5_32.dll Hash Page Sha1" Hash="26B62410A230E79A02DE371A7AEB9DFE2604A79C" />
    <Allow ID="ID_ALLOW_A_50A" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\CORE\msys-perl5_32.dll Hash Page Sha256" Hash="2988CE8265B33BF5DA0116F49E69EB60E747E39F05A0EEEA35E2DBE3DCE3B9C1" />
    <Allow ID="ID_ALLOW_A_50B" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\XS\Typemap\Typemap.dll Hash Sha1" Hash="38FD61F66D1E03C06E67D6D88316646FFF356239" />
    <Allow ID="ID_ALLOW_A_50C" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\XS\Typemap\Typemap.dll Hash Sha256" Hash="02934A9D97449106867F670A88E89A0A3C5DD5CB1BCC18710F01DCD6AFB96941" />
    <Allow ID="ID_ALLOW_A_50D" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\XS\Typemap\Typemap.dll Hash Page Sha1" Hash="5A1B4984CE0522E9B3CEDFB1FA0FBD410354D78A" />
    <Allow ID="ID_ALLOW_A_50E" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\XS\Typemap\Typemap.dll Hash Page Sha256" Hash="F48B00E677567AE3015D895C61476BC694A1DB0F100FE201AEE76A15C7DB27E1" />
    <Allow ID="ID_ALLOW_A_50F" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\XS\APItest\APItest.dll Hash Sha1" Hash="1B35B39009E270FD3C9FFE6B8AB41CE7CB205742" />
    <Allow ID="ID_ALLOW_A_510" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\XS\APItest\APItest.dll Hash Sha256" Hash="69FFC3F936D3D51AEFE2F17078D1CB74FE2417961858F1B24E225330147DDAD5" />
    <Allow ID="ID_ALLOW_A_511" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\XS\APItest\APItest.dll Hash Page Sha1" Hash="E4C7F05FD6ED611FB1262612A8FB4700394F116E" />
    <Allow ID="ID_ALLOW_A_512" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\XS\APItest\APItest.dll Hash Page Sha256" Hash="DD36B5F1A53F2770BBA23E34B223A6181E4684F2087DCAAB36683CD9F3AD4114" />
    <Allow ID="ID_ALLOW_A_513" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\Win32API\File\File.dll Hash Sha1" Hash="00852AEC4D857A004824EA16BA0E2DCC296DAA24" />
    <Allow ID="ID_ALLOW_A_514" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\Win32API\File\File.dll Hash Sha256" Hash="6654F4FFFA5D7232798DB14EEB8975A42352DA9F199B5A1856BCD754D5C6DA91" />
    <Allow ID="ID_ALLOW_A_515" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\Win32API\File\File.dll Hash Page Sha1" Hash="4477D91448A4E47ACDFEC3E80B32D634B838138F" />
    <Allow ID="ID_ALLOW_A_516" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\Win32API\File\File.dll Hash Page Sha256" Hash="8E808140F2428D2D23F4683BB422FCB964DEA0B212C12B83932AC137B8B1A8EC" />
    <Allow ID="ID_ALLOW_A_517" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\Win32\Win32.dll Hash Sha1" Hash="88EBBECA3F09304543D9AD28BC0112567CD820E5" />
    <Allow ID="ID_ALLOW_A_518" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\Win32\Win32.dll Hash Sha256" Hash="D0CC785ED0967A37861647E491C7FDE9CF8EF1E6B468084C93633C46B8876ADF" />
    <Allow ID="ID_ALLOW_A_519" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\Win32\Win32.dll Hash Page Sha1" Hash="FFF0D4D8DFB27C29493E2A53DB985C253BAAFCD8" />
    <Allow ID="ID_ALLOW_A_51A" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\Win32\Win32.dll Hash Page Sha256" Hash="F003CA67CF344C1CA4C758BA0653B9E44103BD7D3336E237976E9AC49F13F569" />
    <Allow ID="ID_ALLOW_A_51B" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\Unicode\Normalize\Normalize.dll Hash Sha1" Hash="394FC2A07AF995E09C5584D6C677B2F7EF275E70" />
    <Allow ID="ID_ALLOW_A_51C" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\Unicode\Normalize\Normalize.dll Hash Sha256" Hash="0D2036069CA36AD02EE392E3B857382CE09C67398D01364FAB5239247B7A5509" />
    <Allow ID="ID_ALLOW_A_51D" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\Unicode\Normalize\Normalize.dll Hash Page Sha1" Hash="3637E85072E643E3F03FFEB30D8EE21DAE6D3658" />
    <Allow ID="ID_ALLOW_A_51E" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\Unicode\Normalize\Normalize.dll Hash Page Sha256" Hash="D603B78B0FE1735EBA0FC2E1FE77C511B69AA1C575AA23EA07CB7C1529CB79C3" />
    <Allow ID="ID_ALLOW_A_51F" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\Unicode\Collate\Collate.dll Hash Sha1" Hash="86A50D331BAEB04A2B2919162CCF1724ACB086AA" />
    <Allow ID="ID_ALLOW_A_520" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\Unicode\Collate\Collate.dll Hash Sha256" Hash="3C12E3E84D2A686E5CF9A21D67043D2E89706A80D593A280D4A71689DDE7EE04" />
    <Allow ID="ID_ALLOW_A_521" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\Unicode\Collate\Collate.dll Hash Page Sha1" Hash="45CDAD32AF6895EB1741DF4AF8F908E3FA708F74" />
    <Allow ID="ID_ALLOW_A_522" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\Unicode\Collate\Collate.dll Hash Page Sha256" Hash="96BD2A04368DFE48924C5795A2B23D39546A3F2A169FA4D5F438B56CE8925160" />
    <Allow ID="ID_ALLOW_A_523" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\Time\Piece\Piece.dll Hash Sha1" Hash="40E72AA4842FA87A029F5CE5ADA0C82C6BB7EB6D" />
    <Allow ID="ID_ALLOW_A_524" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\Time\Piece\Piece.dll Hash Sha256" Hash="9FC52378D202BB4BDEB970320FA3A17424B2D2BA66C398EF1CD58256B4AC285C" />
    <Allow ID="ID_ALLOW_A_525" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\Time\Piece\Piece.dll Hash Page Sha1" Hash="7054B87F06A1B6D86335B2BE1758EB53A750BDAA" />
    <Allow ID="ID_ALLOW_A_526" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\Time\Piece\Piece.dll Hash Page Sha256" Hash="05BD29584A85D7123822423554CF912B767B95F5AFC5F90C66EFEC4C0BAFEF87" />
    <Allow ID="ID_ALLOW_A_527" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\Time\HiRes\HiRes.dll Hash Sha1" Hash="ED0AF96FDA7503B6DCA870230BF422DA85D2C77B" />
    <Allow ID="ID_ALLOW_A_528" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\Time\HiRes\HiRes.dll Hash Sha256" Hash="8776671EDE75795FFB4E7542A38BBC37A103EA48C8AC7F90786FF38D1BADC42D" />
    <Allow ID="ID_ALLOW_A_529" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\Time\HiRes\HiRes.dll Hash Page Sha1" Hash="03A61DD8CACA08A9CDFA67CEF6226ED912156EBC" />
    <Allow ID="ID_ALLOW_A_52A" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\Time\HiRes\HiRes.dll Hash Page Sha256" Hash="D7C08BBB4371E7B7104076F030DB74BD6A1B0FC023496978A165A3FDA526C642" />
    <Allow ID="ID_ALLOW_A_52B" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\threads\threads.dll Hash Sha1" Hash="A4D987527991CD4F23E6923A961CBC45D7F55F84" />
    <Allow ID="ID_ALLOW_A_52C" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\threads\threads.dll Hash Sha256" Hash="8361A5F9D34087256399F278A0C64038CB32E4FD68670BFFEB0B0628A606F7B9" />
    <Allow ID="ID_ALLOW_A_52D" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\threads\threads.dll Hash Page Sha1" Hash="2796C461468AEE4818BDCE612E58FAEB87FDAD88" />
    <Allow ID="ID_ALLOW_A_52E" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\threads\threads.dll Hash Page Sha256" Hash="516484B63674DEA92ED68B46D8B65819E7777E1F1DD602DF62FA499B89F77328" />
    <Allow ID="ID_ALLOW_A_52F" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\threads\shared\shared.dll Hash Sha1" Hash="F1F5F096A6AD38282D9B0651F348965BA7CBCAC8" />
    <Allow ID="ID_ALLOW_A_530" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\threads\shared\shared.dll Hash Sha256" Hash="E05163F6F97196010799578E569D1F02D816558D0A85331BA1DEA5B7B3B487CC" />
    <Allow ID="ID_ALLOW_A_531" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\threads\shared\shared.dll Hash Page Sha1" Hash="BAB35F8D526AD572CC5CDC57DF1182EBBF5F2DF6" />
    <Allow ID="ID_ALLOW_A_532" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\threads\shared\shared.dll Hash Page Sha256" Hash="EB8003DEE7EB7F7B571AA89112912A24F14CC65EA39BAE03B6F5489703F11B89" />
    <Allow ID="ID_ALLOW_A_533" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\Sys\Syslog\Syslog.dll Hash Sha1" Hash="BCFE42D5EF1B073A0C2894B3AB7DA1C328CAAC16" />
    <Allow ID="ID_ALLOW_A_534" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\Sys\Syslog\Syslog.dll Hash Sha256" Hash="F462156682701F60CA9E0544F6DD45431164641592BD169520D9377429C1D5DA" />
    <Allow ID="ID_ALLOW_A_535" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\Sys\Syslog\Syslog.dll Hash Page Sha1" Hash="A226C7693F91C5AA73D103C3E9571EB713B14BB2" />
    <Allow ID="ID_ALLOW_A_536" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\Sys\Syslog\Syslog.dll Hash Page Sha256" Hash="DA234E9381AEDEC7E8C12E85A617431BC265E63D243D013DA0D51B7FF4836E5D" />
    <Allow ID="ID_ALLOW_A_537" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\Sys\Hostname\Hostname.dll Hash Sha1" Hash="577B0BC8A93F7742DA0BEAFE5DF46CE6A99A23AC" />
    <Allow ID="ID_ALLOW_A_538" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\Sys\Hostname\Hostname.dll Hash Sha256" Hash="F7C586B8A01AD1DB205782954BDB17AD5A6005FA92B9B199627C873682AA5EE7" />
    <Allow ID="ID_ALLOW_A_539" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\Sys\Hostname\Hostname.dll Hash Page Sha1" Hash="70723FD33783BB055301422ECB592FFDCE45997F" />
    <Allow ID="ID_ALLOW_A_53A" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\Sys\Hostname\Hostname.dll Hash Page Sha256" Hash="6166B0177895C4E795DF5906871C8EFAE024A6BCD33963D2E1B8EFCF01DBCDFE" />
    <Allow ID="ID_ALLOW_A_53B" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\Storable\Storable.dll Hash Sha1" Hash="CC5163CD99B85C5E7FD7744581C24F3E19623063" />
    <Allow ID="ID_ALLOW_A_53C" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\Storable\Storable.dll Hash Sha256" Hash="B6F6240EDAB1D4D49D7E6299C1721706A95F7E838D11F95D6A77DC5F59F2256C" />
    <Allow ID="ID_ALLOW_A_53D" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\Storable\Storable.dll Hash Page Sha1" Hash="AC64CA4461C9F7527E3C593406133A10968AF3D5" />
    <Allow ID="ID_ALLOW_A_53E" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\Storable\Storable.dll Hash Page Sha256" Hash="0E03C55F769396C70B4E0F722ECCB40A493E95DEFE07A73C17375DCF49FD4BE1" />
    <Allow ID="ID_ALLOW_A_53F" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\Socket\Socket.dll Hash Sha1" Hash="9892EE2CBE1C9415F4B117B35055F89EE4665A6A" />
    <Allow ID="ID_ALLOW_A_540" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\Socket\Socket.dll Hash Sha256" Hash="9CCDB2ED32B1153C06563004EAFE03C976C1A7131EA2171D59EB9A9184351C2C" />
    <Allow ID="ID_ALLOW_A_541" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\Socket\Socket.dll Hash Page Sha1" Hash="5F1B7BE89BB4C1BA5F7A78A263610C8296E04C45" />
    <Allow ID="ID_ALLOW_A_542" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\Socket\Socket.dll Hash Page Sha256" Hash="8BFD7B44678D27DBB6E74E86449380DCECF1D2544B4A24AB5688734F4BBE3E3B" />
    <Allow ID="ID_ALLOW_A_543" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\SDBM_File\SDBM_File.dll Hash Sha1" Hash="3138F8F0B247E9BA71898CEB2B4E29E348656C9D" />
    <Allow ID="ID_ALLOW_A_544" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\SDBM_File\SDBM_File.dll Hash Sha256" Hash="C1661D8CB16F4F22B3A8CAC8A29B35E53F80297B9A3C3B4C50E7BFBCA3D4BAFB" />
    <Allow ID="ID_ALLOW_A_545" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\SDBM_File\SDBM_File.dll Hash Page Sha1" Hash="FA82594D9EE3B58E868481A1F473439E67D4580B" />
    <Allow ID="ID_ALLOW_A_546" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\SDBM_File\SDBM_File.dll Hash Page Sha256" Hash="9D78A835E0400DC272BD2B1D8C9E5682E550E843D69B4323827876DFA56D91D0" />
    <Allow ID="ID_ALLOW_A_547" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\re\re.dll Hash Sha1" Hash="177EBA744F3DDE6EDD8CFA76EDF36E23222A1BC3" />
    <Allow ID="ID_ALLOW_A_548" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\re\re.dll Hash Sha256" Hash="2439C41164B792B338E4E03393E975C0A0CED54A0B5CD0A4D9AA0A3A0A5A748D" />
    <Allow ID="ID_ALLOW_A_549" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\re\re.dll Hash Page Sha1" Hash="50BE68A47EB5BC4F23A64305D481A0D8C56AE87E" />
    <Allow ID="ID_ALLOW_A_54A" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\re\re.dll Hash Page Sha256" Hash="20159D509BDF1A255CE613C2AC462F1FFF1597C5B23B6D9CDD4EDA7F8AB9D9FA" />
    <Allow ID="ID_ALLOW_A_54B" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\POSIX\POSIX.dll Hash Sha1" Hash="B958667E1B195BB1B43DABB2AA5B6701DD60F657" />
    <Allow ID="ID_ALLOW_A_54C" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\POSIX\POSIX.dll Hash Sha256" Hash="5B4A3D8B7BA6ACE2A2A11B8CA8D561D104B9C5F23398835ACEAE2D88BA0ACA69" />
    <Allow ID="ID_ALLOW_A_54D" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\POSIX\POSIX.dll Hash Page Sha1" Hash="35285F1BF9F2E522320FBFE873184738A10542E7" />
    <Allow ID="ID_ALLOW_A_54E" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\POSIX\POSIX.dll Hash Page Sha256" Hash="10279CAD61FF649A4A2931FB215A87BFBCE794679D3AD02977ADAB546F9DBC68" />
    <Allow ID="ID_ALLOW_A_54F" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\PerlIO\via\via.dll Hash Sha1" Hash="F55B6264C509130011395E488A717C2707AA57C3" />
    <Allow ID="ID_ALLOW_A_550" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\PerlIO\via\via.dll Hash Sha256" Hash="F67E340A1032E491484CF5676027CC9F91CED37BDD1D0AE2A1250AF6EFD9842E" />
    <Allow ID="ID_ALLOW_A_551" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\PerlIO\via\via.dll Hash Page Sha1" Hash="6A121E7CFDB96EB19C5770C5B1C1143426D1E726" />
    <Allow ID="ID_ALLOW_A_552" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\PerlIO\via\via.dll Hash Page Sha256" Hash="7F85F1026C0C8214036EE7C45E6AB5038FDB7AF39E38959501CAE1F4341C76EE" />
    <Allow ID="ID_ALLOW_A_553" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\PerlIO\scalar\scalar.dll Hash Sha1" Hash="24D6989E4979AA2853A62BF813EF237CFC9123AC" />
    <Allow ID="ID_ALLOW_A_554" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\PerlIO\scalar\scalar.dll Hash Sha256" Hash="D089B97C97B5970F24E5FB29CB5ACCDF452CE7AB8BFEADC649DC943168DFC6D9" />
    <Allow ID="ID_ALLOW_A_555" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\PerlIO\scalar\scalar.dll Hash Page Sha1" Hash="B42705138B4F405A4D57C945B98007DCB84CF504" />
    <Allow ID="ID_ALLOW_A_556" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\PerlIO\scalar\scalar.dll Hash Page Sha256" Hash="4C2849080542A69472C3C3DBAAB970D778BA2332F92A9F42F6A41BA7F52E5596" />
    <Allow ID="ID_ALLOW_A_557" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\PerlIO\mmap\mmap.dll Hash Sha1" Hash="847274AD3FF231D8E667E058D896615FAEFF38CC" />
    <Allow ID="ID_ALLOW_A_558" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\PerlIO\mmap\mmap.dll Hash Sha256" Hash="A461F1AF166A83678D40FE916C140191FB3E91EBB441ECF6E4C644771D116006" />
    <Allow ID="ID_ALLOW_A_559" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\PerlIO\mmap\mmap.dll Hash Page Sha1" Hash="63B2FD001FFED796919B1D21B2B66A3CB960AA78" />
    <Allow ID="ID_ALLOW_A_55A" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\PerlIO\mmap\mmap.dll Hash Page Sha256" Hash="037D0CDFE6956E4555F9A0B443FC6242E70AD500AC6468B8A8B277F747473088" />
    <Allow ID="ID_ALLOW_A_55B" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\PerlIO\encoding\encoding.dll Hash Sha1" Hash="1E03BA84AF45528CD234E92B88DE71F625E29D0B" />
    <Allow ID="ID_ALLOW_A_55C" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\PerlIO\encoding\encoding.dll Hash Sha256" Hash="FB2C1C15A08851DF5DD230D6D96B3CF0D6C8FE7F9DF708AD06A42DEF28928BF3" />
    <Allow ID="ID_ALLOW_A_55D" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\PerlIO\encoding\encoding.dll Hash Page Sha1" Hash="71B5D8490C04E1AB1D9DFB599A414CDB3D206BCA" />
    <Allow ID="ID_ALLOW_A_55E" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\PerlIO\encoding\encoding.dll Hash Page Sha256" Hash="138FD390B85346D68EB510335F48B308D0ED64B6372DEF963225B1F16E86F122" />
    <Allow ID="ID_ALLOW_A_55F" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\Opcode\Opcode.dll Hash Sha1" Hash="F124B03986BA76CF82FF88375EE86E565BE4E967" />
    <Allow ID="ID_ALLOW_A_560" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\Opcode\Opcode.dll Hash Sha256" Hash="DC57A4E42D47960DF5F9A563DF6F2BEFA182EC7F4182E9D0A8163E648A5CEA23" />
    <Allow ID="ID_ALLOW_A_561" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\Opcode\Opcode.dll Hash Page Sha1" Hash="3EB1D3ED5CA1E7DB4AA6EBC9FAF6C72EA571C159" />
    <Allow ID="ID_ALLOW_A_562" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\Opcode\Opcode.dll Hash Page Sha256" Hash="5E94E6E262F4AAA4184B3EBCDDB7A1EA334A8162A6B7D190C74B50C2BE6EC004" />
    <Allow ID="ID_ALLOW_A_563" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\NDBM_File\NDBM_File.dll Hash Sha1" Hash="7CFE472301185716D2B40400FFE4DE0967541233" />
    <Allow ID="ID_ALLOW_A_564" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\NDBM_File\NDBM_File.dll Hash Sha256" Hash="5AE8A974CB220BBAE747D5C59A9A6CF9B1DD3E9DBF01D5E1AD03D9C5EFE58234" />
    <Allow ID="ID_ALLOW_A_565" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\NDBM_File\NDBM_File.dll Hash Page Sha1" Hash="22D634FAA9045F50F50317DED67A40DF30EE9456" />
    <Allow ID="ID_ALLOW_A_566" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\NDBM_File\NDBM_File.dll Hash Page Sha256" Hash="22264D5F96BD89359F1B3508E6EDCAF150BB6CDD1C19C06901731FAC36A15594" />
    <Allow ID="ID_ALLOW_A_567" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\mro\mro.dll Hash Sha1" Hash="EC81683B5E6B3622D6B5A8C3365333685D03465E" />
    <Allow ID="ID_ALLOW_A_568" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\mro\mro.dll Hash Sha256" Hash="24325862DC93411FE761AACAB1DC2786044D00CFC341B19C03D6A1E51A8946C1" />
    <Allow ID="ID_ALLOW_A_569" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\mro\mro.dll Hash Page Sha1" Hash="ECDB1AB8CCA9CEA958DFFFB40DFC9AFF95B16101" />
    <Allow ID="ID_ALLOW_A_56A" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\mro\mro.dll Hash Page Sha256" Hash="CFA2BE0235DE1218F039A47D8A1AAA3805169098ECB07C4EE0FEE51DB712CE9B" />
    <Allow ID="ID_ALLOW_A_56B" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\MIME\Base64\Base64.dll Hash Sha1" Hash="99BF636D640DB4450A65D0C0ECA5D20AF3B71BED" />
    <Allow ID="ID_ALLOW_A_56C" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\MIME\Base64\Base64.dll Hash Sha256" Hash="6C5F69CD0F61D5FD6AD31FD39FCE73115E21D4606B74540EDCF73A3BA0CB6ED3" />
    <Allow ID="ID_ALLOW_A_56D" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\MIME\Base64\Base64.dll Hash Page Sha1" Hash="C4F690699A7E7810D7680B5E7A54DA68E555EB67" />
    <Allow ID="ID_ALLOW_A_56E" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\MIME\Base64\Base64.dll Hash Page Sha256" Hash="169BD26C7411B24676930615E9998626212E6A00560E8ADBFF59F3AA9C48EAED" />
    <Allow ID="ID_ALLOW_A_56F" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\Math\BigInt\FastCalc\FastCalc.dll Hash Sha1" Hash="6E4414E716E621A4F3478FAB38B7A9FDA29E2BD8" />
    <Allow ID="ID_ALLOW_A_570" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\Math\BigInt\FastCalc\FastCalc.dll Hash Sha256" Hash="76523EFD2437E1E33C46D76C39AC6991A0BC295AEF9276A6B04720915304EB4E" />
    <Allow ID="ID_ALLOW_A_571" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\Math\BigInt\FastCalc\FastCalc.dll Hash Page Sha1" Hash="32A159334738C2D4E4FFBF38DF388AD2535ACAEB" />
    <Allow ID="ID_ALLOW_A_572" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\Math\BigInt\FastCalc\FastCalc.dll Hash Page Sha256" Hash="4D7B678C65F327E1574EFDADAE907F4FE0B476A96A2FA918E4F9B6BDEBFD655D" />
    <Allow ID="ID_ALLOW_A_573" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\List\Util\Util.dll Hash Sha1" Hash="C4F8BBA76AD547613EC4986579BD3F5D061DD6DD" />
    <Allow ID="ID_ALLOW_A_574" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\List\Util\Util.dll Hash Sha256" Hash="696A75D459191B1FE78031990CB32B9078788B79C3A2648A6AB0B80922A727A4" />
    <Allow ID="ID_ALLOW_A_575" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\List\Util\Util.dll Hash Page Sha1" Hash="E0BF63038F2DB0C14D9CB240E114731575266479" />
    <Allow ID="ID_ALLOW_A_576" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\List\Util\Util.dll Hash Page Sha256" Hash="AC75E507F752F961AF6226345DC37D3A3461F0CCF5BD6670AD6CA5372321E8AB" />
    <Allow ID="ID_ALLOW_A_577" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\IPC\SysV\SysV.dll Hash Sha1" Hash="99814ABA75429E4B2D4AC42E3FEC159F67C627CA" />
    <Allow ID="ID_ALLOW_A_578" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\IPC\SysV\SysV.dll Hash Sha256" Hash="9E37E9C9E023062AFA2D83F320D13CCBC21AD550E05996D794BB1E2CB4A258F8" />
    <Allow ID="ID_ALLOW_A_579" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\IPC\SysV\SysV.dll Hash Page Sha1" Hash="46103BE4657BD50F557A2232CF4187173BF377CD" />
    <Allow ID="ID_ALLOW_A_57A" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\IPC\SysV\SysV.dll Hash Page Sha256" Hash="76B01EACE5603C058B227A0880515481F991DA7F8BAC6CA57D4E6BCFFDAB8790" />
    <Allow ID="ID_ALLOW_A_57B" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\IO\IO.dll Hash Sha1" Hash="609A54ACF0791595A834AB65B7CFF520AD9FC302" />
    <Allow ID="ID_ALLOW_A_57C" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\IO\IO.dll Hash Sha256" Hash="0459BA972219E96328DE320221D6D893CFD1F97390D4B9016456A62CB3C65B12" />
    <Allow ID="ID_ALLOW_A_57D" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\IO\IO.dll Hash Page Sha1" Hash="FF9E411524915EE315E9F3675CDCC7719CEBEFD8" />
    <Allow ID="ID_ALLOW_A_57E" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\IO\IO.dll Hash Page Sha256" Hash="8FEA19CC74001E56709681DE6F9A8D79AF6EA89854289F9ADAA06223A9813F4D" />
    <Allow ID="ID_ALLOW_A_57F" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\I18N\Langinfo\Langinfo.dll Hash Sha1" Hash="41B3F63D84D6B3EBAF5028FBD2CEA49AA985CF7A" />
    <Allow ID="ID_ALLOW_A_580" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\I18N\Langinfo\Langinfo.dll Hash Sha256" Hash="9FD5BD9C3B26194AE9D940787C599A9631F598781A7B18878E996697C79BE339" />
    <Allow ID="ID_ALLOW_A_581" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\I18N\Langinfo\Langinfo.dll Hash Page Sha1" Hash="F0D92D50EC887AFB84ADBE07C28A7A7311F52BD1" />
    <Allow ID="ID_ALLOW_A_582" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\I18N\Langinfo\Langinfo.dll Hash Page Sha256" Hash="AB276D84C45A422A8F19E3367392C78B11CA81061FE24804C5BF1A8024AA99CC" />
    <Allow ID="ID_ALLOW_A_583" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\Hash\Util\Util.dll Hash Sha1" Hash="0B6CB0EA8215985D300D1327D5B049177312C50F" />
    <Allow ID="ID_ALLOW_A_584" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\Hash\Util\Util.dll Hash Sha256" Hash="2E2DBBC6BD4BC6B302B3F378261999B704A63045DA70E796F1BDC24D82C67967" />
    <Allow ID="ID_ALLOW_A_585" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\Hash\Util\Util.dll Hash Page Sha1" Hash="BA7D773F5AD16C61B8F34EC9FFC9E543D215FAC9" />
    <Allow ID="ID_ALLOW_A_586" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\Hash\Util\Util.dll Hash Page Sha256" Hash="942BAEAF9B9C42E6F0E06108AF831C9C39930D8BC7E13C950ECE590DE6D769CC" />
    <Allow ID="ID_ALLOW_A_587" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\Hash\Util\FieldHash\FieldHash.dll Hash Sha1" Hash="F2298FBFF227ED0BF838192D9ADD77EB04861EEF" />
    <Allow ID="ID_ALLOW_A_588" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\Hash\Util\FieldHash\FieldHash.dll Hash Sha256" Hash="3FBF05DFB32D4DAD8CD11A708A5D7E09D366D30DC612332AD2C615F4219F2616" />
    <Allow ID="ID_ALLOW_A_589" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\Hash\Util\FieldHash\FieldHash.dll Hash Page Sha1" Hash="7CBEB960D57C79DCDE44B167D8A6E22765EBF952" />
    <Allow ID="ID_ALLOW_A_58A" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\Hash\Util\FieldHash\FieldHash.dll Hash Page Sha256" Hash="8A413A2DF7ABEA287ABD8887B1001403D110F933F947EAF99533821628A7CCB9" />
    <Allow ID="ID_ALLOW_A_58B" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\Filter\Util\Call\Call.dll Hash Sha1" Hash="0528A341D23F7216FD7F2236199F239C3456D5F5" />
    <Allow ID="ID_ALLOW_A_58C" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\Filter\Util\Call\Call.dll Hash Sha256" Hash="B00C50600C2B4044D07097797B245C41697FB3801928947D8AC30A915193B123" />
    <Allow ID="ID_ALLOW_A_58D" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\Filter\Util\Call\Call.dll Hash Page Sha1" Hash="FFCD4AFD028F334CDF521FBE45D66E8E614DA972" />
    <Allow ID="ID_ALLOW_A_58E" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\Filter\Util\Call\Call.dll Hash Page Sha256" Hash="F8120A6DF58767D9BDED45BD0F439480624FDCE19A57A735E223BC607A000952" />
    <Allow ID="ID_ALLOW_A_58F" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\File\Glob\Glob.dll Hash Sha1" Hash="34D8389919A161342F1041EA63F9AA904326D5B8" />
    <Allow ID="ID_ALLOW_A_590" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\File\Glob\Glob.dll Hash Sha256" Hash="364E89D0CD0C421649D20D3954072E688E0C63C62E3CEC80B182D0417D07EB3F" />
    <Allow ID="ID_ALLOW_A_591" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\File\Glob\Glob.dll Hash Page Sha1" Hash="A4F68E40E333FA767F5787A0F610FD1C5BF095E7" />
    <Allow ID="ID_ALLOW_A_592" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\File\Glob\Glob.dll Hash Page Sha256" Hash="B42B65D7D89372F98E8E9CA9F415C7F5588730A8A14ECCC14FF0E5D8C3B44F68" />
    <Allow ID="ID_ALLOW_A_593" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\File\DosGlob\DosGlob.dll Hash Sha1" Hash="A8530C8597F8971E2302B1CC564814470BFC618A" />
    <Allow ID="ID_ALLOW_A_594" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\File\DosGlob\DosGlob.dll Hash Sha256" Hash="67A7D6C788C99812ACFD9AA820B07FF3A13565C05232978F118496382C99EB75" />
    <Allow ID="ID_ALLOW_A_595" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\File\DosGlob\DosGlob.dll Hash Page Sha1" Hash="E2877DA53389326C925443ECA52EE70F214CF69B" />
    <Allow ID="ID_ALLOW_A_596" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\File\DosGlob\DosGlob.dll Hash Page Sha256" Hash="2E0C61966AA618EF6D53E118096701F67E73FB2ACC04A93576CA6FF7D9864B6E" />
    <Allow ID="ID_ALLOW_A_597" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\Fcntl\Fcntl.dll Hash Sha1" Hash="DE721E4AD7AC745A4E02E2567071D70696CED4F4" />
    <Allow ID="ID_ALLOW_A_598" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\Fcntl\Fcntl.dll Hash Sha256" Hash="B3A37E8F0B4E70ACD8109D0187028AE10DDFA189DAD4F04F9C34B384FC13F60B" />
    <Allow ID="ID_ALLOW_A_599" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\Fcntl\Fcntl.dll Hash Page Sha1" Hash="1AFEF0C410946E91D004CAD8C3EB49A5C2998D86" />
    <Allow ID="ID_ALLOW_A_59A" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\Fcntl\Fcntl.dll Hash Page Sha256" Hash="4A8B4E8D248C54C86B32B7361F560D4789AFCEB804DE194A24F475A1F12A90A9" />
    <Allow ID="ID_ALLOW_A_59B" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\Encode\Encode.dll Hash Sha1" Hash="2223BD17F3C43F2E557FC467E90877C551327C5C" />
    <Allow ID="ID_ALLOW_A_59C" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\Encode\Encode.dll Hash Sha256" Hash="1A6643DAAA1555E0FF870C11DE8321006F9E1677DCBCE4394833F71935429B6E" />
    <Allow ID="ID_ALLOW_A_59D" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\Encode\Encode.dll Hash Page Sha1" Hash="5ADEBC421D5D9AFF471CDCE58B9BE18381F35034" />
    <Allow ID="ID_ALLOW_A_59E" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\Encode\Encode.dll Hash Page Sha256" Hash="B76E6D6426ADCF9C0491FFA6E179F141B7B3FC01D6D4992E907DC7A4BD410E15" />
    <Allow ID="ID_ALLOW_A_59F" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\Encode\Unicode\Unicode.dll Hash Sha1" Hash="07615E512CCDCC82D28E458EBC1F27B8752ED6A9" />
    <Allow ID="ID_ALLOW_A_5A0" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\Encode\Unicode\Unicode.dll Hash Sha256" Hash="70ABB0E7AE533BB42D6D83D3C8C1A08544B12212DBFAA0061804C73328BC52F8" />
    <Allow ID="ID_ALLOW_A_5A1" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\Encode\Unicode\Unicode.dll Hash Page Sha1" Hash="B6EDA34DB40AD7C142D7A88400B02731C03D2E22" />
    <Allow ID="ID_ALLOW_A_5A2" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\Encode\Unicode\Unicode.dll Hash Page Sha256" Hash="18692890BC1F62CC7AFC610DDD2AF1B141D63AFB476A4BC865B1499B8CB555A5" />
    <Allow ID="ID_ALLOW_A_5A3" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\Encode\TW\TW.dll Hash Sha1" Hash="C40609E509ACB63CE46150EDCB1E055A18ADFFBD" />
    <Allow ID="ID_ALLOW_A_5A4" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\Encode\TW\TW.dll Hash Sha256" Hash="4971E88F8329C84F09E22A295949A9CD708580491E1F2E516E780EC9C8A6798E" />
    <Allow ID="ID_ALLOW_A_5A5" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\Encode\TW\TW.dll Hash Page Sha1" Hash="DBC1C68285C749135BB0A1653FD328C0CD54CDDA" />
    <Allow ID="ID_ALLOW_A_5A6" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\Encode\TW\TW.dll Hash Page Sha256" Hash="A1D0391C746260DAA891BF30417F8272C259D2496B95920A0256207108436C82" />
    <Allow ID="ID_ALLOW_A_5A7" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\Encode\Symbol\Symbol.dll Hash Sha1" Hash="327D68E9EFBFDCD45F39F446A6B47B7BAF1DED17" />
    <Allow ID="ID_ALLOW_A_5A8" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\Encode\Symbol\Symbol.dll Hash Sha256" Hash="607F56EAEC9C5FFED149EB83DF444A4D8DAC9D715200F9A878A5036B35B5CA80" />
    <Allow ID="ID_ALLOW_A_5A9" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\Encode\Symbol\Symbol.dll Hash Page Sha1" Hash="D0B4E18C077380B5C0A30215198042606F1A7F8C" />
    <Allow ID="ID_ALLOW_A_5AA" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\Encode\Symbol\Symbol.dll Hash Page Sha256" Hash="F852324755A789653E3698B2BA11AB4193BF3D3ECDCF17AF6FDADCFD5BFD201E" />
    <Allow ID="ID_ALLOW_A_5AB" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\Encode\KR\KR.dll Hash Sha1" Hash="34D3EB3D57AEB1D82F5DA874215FADB4F422BBCD" />
    <Allow ID="ID_ALLOW_A_5AC" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\Encode\KR\KR.dll Hash Sha256" Hash="EAE8060396D719BA2918F80AEAF0C98E43996977F25FF1CC790CB25A42F56EB7" />
    <Allow ID="ID_ALLOW_A_5AD" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\Encode\KR\KR.dll Hash Page Sha1" Hash="61EAD0E810C2BFF57283B9004A0356D8D3FC17D4" />
    <Allow ID="ID_ALLOW_A_5AE" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\Encode\KR\KR.dll Hash Page Sha256" Hash="A2ACA8EECE03CA0D517BA86C8563F8E2AF592B28992004F76AFB8DFDDB574A0A" />
    <Allow ID="ID_ALLOW_A_5AF" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\Encode\JP\JP.dll Hash Sha1" Hash="B5E45F3433A337FCA02CD97F67C519841B6F3E68" />
    <Allow ID="ID_ALLOW_A_5B0" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\Encode\JP\JP.dll Hash Sha256" Hash="C0B3793453E2527157CFC99BDAB9B37B4553EB2402BAE76529666FC65703DD6E" />
    <Allow ID="ID_ALLOW_A_5B1" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\Encode\JP\JP.dll Hash Page Sha1" Hash="7B92FCE4478CAF2CD952584688DA948D508983FE" />
    <Allow ID="ID_ALLOW_A_5B2" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\Encode\JP\JP.dll Hash Page Sha256" Hash="A719632F344CCE2514DA7208B850E6FC213FFD90C5B5D9D469E633CD5A125350" />
    <Allow ID="ID_ALLOW_A_5B3" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\Encode\EBCDIC\EBCDIC.dll Hash Sha1" Hash="ABD21297AC71F178A6D02BC78CFC0D133E0A9519" />
    <Allow ID="ID_ALLOW_A_5B4" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\Encode\EBCDIC\EBCDIC.dll Hash Sha256" Hash="7596CA5C83A253762E579D60B21F46F31A82E7645A6F663C5F9A4B51F4FA29AB" />
    <Allow ID="ID_ALLOW_A_5B5" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\Encode\EBCDIC\EBCDIC.dll Hash Page Sha1" Hash="6C31F4ECD801A9F7D16A85DF11E03039183B31F0" />
    <Allow ID="ID_ALLOW_A_5B6" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\Encode\EBCDIC\EBCDIC.dll Hash Page Sha256" Hash="6E7BF8B5411603B79691BA276B20A54BADEFB3CBB76AAEB1011119033EDD1E24" />
    <Allow ID="ID_ALLOW_A_5B7" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\Encode\CN\CN.dll Hash Sha1" Hash="DCF1E5E5595E09A1F1C83688D0CB5FF88B6E71BB" />
    <Allow ID="ID_ALLOW_A_5B8" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\Encode\CN\CN.dll Hash Sha256" Hash="715B95CDDCC1B89E04D6C3079D9728580108E9638B7D760346408A195AD646B7" />
    <Allow ID="ID_ALLOW_A_5B9" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\Encode\CN\CN.dll Hash Page Sha1" Hash="72577CF09B99ADEE22CA01C531773617DB23DB99" />
    <Allow ID="ID_ALLOW_A_5BA" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\Encode\CN\CN.dll Hash Page Sha256" Hash="4B3D60756D4B1E82DF516E2A9E77C42EC0F368575823FE6F2DC87DA8B819C54A" />
    <Allow ID="ID_ALLOW_A_5BB" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\Encode\Byte\Byte.dll Hash Sha1" Hash="0EF53D0FFFDB03FCD65AA019FF653D07DBDF5718" />
    <Allow ID="ID_ALLOW_A_5BC" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\Encode\Byte\Byte.dll Hash Sha256" Hash="59756EEC12DBF84A51C8E2A1959A39491CBEF8DA34F8D4F8068E6AA66A3F3A3C" />
    <Allow ID="ID_ALLOW_A_5BD" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\Encode\Byte\Byte.dll Hash Page Sha1" Hash="1A4D9A3864C95FBC3F3C0CC7EE59DFC64C8A2E36" />
    <Allow ID="ID_ALLOW_A_5BE" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\Encode\Byte\Byte.dll Hash Page Sha256" Hash="95AE9BF227924C3562126139CDA40C21133BE0E17AF32567207E8E4F27164149" />
    <Allow ID="ID_ALLOW_A_5BF" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\Digest\SHA\SHA.dll Hash Sha1" Hash="3178CDC0CDE20ADA924C534F3BDFD4B89B1B2468" />
    <Allow ID="ID_ALLOW_A_5C0" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\Digest\SHA\SHA.dll Hash Sha256" Hash="39195C0D7E964CFAB137DA0A4C761ABFD9DB13935476C986A9777134F24B98AA" />
    <Allow ID="ID_ALLOW_A_5C1" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\Digest\SHA\SHA.dll Hash Page Sha1" Hash="311EBB948D40158C23CD8B1A7257D69B1D259BFC" />
    <Allow ID="ID_ALLOW_A_5C2" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\Digest\SHA\SHA.dll Hash Page Sha256" Hash="6598B83EF44ACF39412201A2B841A19B59C0519B365FCB5A4FC038DD2F89060A" />
    <Allow ID="ID_ALLOW_A_5C3" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\Digest\MD5\MD5.dll Hash Sha1" Hash="83E0A7328F56D33EF9CCFD295A17960E75B8D854" />
    <Allow ID="ID_ALLOW_A_5C4" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\Digest\MD5\MD5.dll Hash Sha256" Hash="44A9AA8354B5B7CBF950DB4E8778AA074FC543102955185344C5846C0CFE7E30" />
    <Allow ID="ID_ALLOW_A_5C5" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\Digest\MD5\MD5.dll Hash Page Sha1" Hash="E65811AD7FF1E79BB0D8C377424954F4159B80AC" />
    <Allow ID="ID_ALLOW_A_5C6" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\Digest\MD5\MD5.dll Hash Page Sha256" Hash="BA1E24D89D637055D613F0E44B4DFC717D8E0FA3EABF8AC9A94597A800D48D7E" />
    <Allow ID="ID_ALLOW_A_5C7" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\Devel\Peek\Peek.dll Hash Sha1" Hash="FD0349070C0F9C6AE6C57060106E797581432C8D" />
    <Allow ID="ID_ALLOW_A_5C8" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\Devel\Peek\Peek.dll Hash Sha256" Hash="0D56562840796197D812C68EA8C1DEB2CEB2C651865617AF0EECBFDB8C83F19A" />
    <Allow ID="ID_ALLOW_A_5C9" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\Devel\Peek\Peek.dll Hash Page Sha1" Hash="1962E9C2D89C1BC804BF93A8D19AEEAFC9D1A18B" />
    <Allow ID="ID_ALLOW_A_5CA" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\Devel\Peek\Peek.dll Hash Page Sha256" Hash="54CE6F579D9D8F3F3DC511CD99C5952C8E4D421366DDF9EDDCA59B2B6DA5BE1E" />
    <Allow ID="ID_ALLOW_A_5CB" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\Data\Dumper\Dumper.dll Hash Sha1" Hash="2404A59EF280CB5B8AD0AAD4DC2B65B83A7AB67A" />
    <Allow ID="ID_ALLOW_A_5CC" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\Data\Dumper\Dumper.dll Hash Sha256" Hash="1AA1E12227E8A6B0DB610B00A41E041B900DC25208E62A6A04DD8B67F3D001F2" />
    <Allow ID="ID_ALLOW_A_5CD" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\Data\Dumper\Dumper.dll Hash Page Sha1" Hash="D257D859C4E8E12F45CED5CB098C35FD1B06F048" />
    <Allow ID="ID_ALLOW_A_5CE" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\Data\Dumper\Dumper.dll Hash Page Sha256" Hash="BABA08211F5407CE60EC5585BC516BB5878E02E49F5742AA8ACC383951C6A0EB" />
    <Allow ID="ID_ALLOW_A_5CF" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\Cwd\Cwd.dll Hash Sha1" Hash="F850B1EA9054885B194D1ECC6D6FCFA5ECDA7C62" />
    <Allow ID="ID_ALLOW_A_5D0" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\Cwd\Cwd.dll Hash Sha256" Hash="E1E477CF3E3A92D0046ACFA20C2DEDF5F391F2FAE158A81ECDA2EAD136C638D7" />
    <Allow ID="ID_ALLOW_A_5D1" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\Cwd\Cwd.dll Hash Page Sha1" Hash="BEEC0B01D0B56469ECB1BC8776794637F0DD5E85" />
    <Allow ID="ID_ALLOW_A_5D2" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\Cwd\Cwd.dll Hash Page Sha256" Hash="3E8215A985735CBEFF82ADB0F8293560E8F8B5A3621683112E35BC4140EB8E4A" />
    <Allow ID="ID_ALLOW_A_5D3" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\Compress\Raw\Zlib\Zlib.dll Hash Sha1" Hash="5B9387AE6D39B95A981DD3AC5482D729FB40D87C" />
    <Allow ID="ID_ALLOW_A_5D4" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\Compress\Raw\Zlib\Zlib.dll Hash Sha256" Hash="687E531CA6FD6943829BA0E4461B502A0D88134D22CA62936FF66E55BC9E543A" />
    <Allow ID="ID_ALLOW_A_5D5" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\Compress\Raw\Zlib\Zlib.dll Hash Page Sha1" Hash="F9D852F150DB65358C8AE39B97E6C3779C000D60" />
    <Allow ID="ID_ALLOW_A_5D6" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\Compress\Raw\Zlib\Zlib.dll Hash Page Sha256" Hash="D7E8A42FFDCC3D0B8D4C90C1862023E0028506AF8E36A4206D31DA63062F702E" />
    <Allow ID="ID_ALLOW_A_5D7" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\Compress\Raw\Bzip2\Bzip2.dll Hash Sha1" Hash="0248987CA68C5A51D84EA88995DCC0C6FFADF25E" />
    <Allow ID="ID_ALLOW_A_5D8" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\Compress\Raw\Bzip2\Bzip2.dll Hash Sha256" Hash="1C97EA3571245710D8671399B5B59D3B423DBB3B9D4CC52528CC44A3746CC746" />
    <Allow ID="ID_ALLOW_A_5D9" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\Compress\Raw\Bzip2\Bzip2.dll Hash Page Sha1" Hash="F812B031B59B2A14F17BEEA38146B23F49217735" />
    <Allow ID="ID_ALLOW_A_5DA" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\Compress\Raw\Bzip2\Bzip2.dll Hash Page Sha256" Hash="BC84D33880B5F2103644EFFFC97CDFBAA33A9EDD6B8FA3A3060FFD43128FA7B2" />
    <Allow ID="ID_ALLOW_A_5DB" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\B\B.dll Hash Sha1" Hash="4E1ADE7AB625C72B50CDFD8B5957C8AA943947F0" />
    <Allow ID="ID_ALLOW_A_5DC" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\B\B.dll Hash Sha256" Hash="4339DCE2066F235869B99558B854C01D8B5419F36DED51A26D74E685A3622374" />
    <Allow ID="ID_ALLOW_A_5DD" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\B\B.dll Hash Page Sha1" Hash="2080B0229D6C241E3DA80BE733004A78B473164B" />
    <Allow ID="ID_ALLOW_A_5DE" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\B\B.dll Hash Page Sha256" Hash="CEAB1A5DE6C48B9C110CB9718E55AB94518F9AD1DFFD0116F689FA6B7A3A4204" />
    <Allow ID="ID_ALLOW_A_5DF" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\attributes\attributes.dll Hash Sha1" Hash="04A48AD94ABD54CCC09F8AD3CE7B2FBDB6F9B1A2" />
    <Allow ID="ID_ALLOW_A_5E0" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\attributes\attributes.dll Hash Sha256" Hash="760D809BB6A5C1CF644E23069658158ADBC6B222D5D2770D3391A05B5A45B8FC" />
    <Allow ID="ID_ALLOW_A_5E1" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\attributes\attributes.dll Hash Page Sha1" Hash="45EE30FBF2A26F98C67BB010707C60F4ACEB9FE1" />
    <Allow ID="ID_ALLOW_A_5E2" FriendlyName="C:\Program Files\git\usr\lib\perl5\core_perl\auto\attributes\attributes.dll Hash Page Sha256" Hash="624A10FE8017786308E5743E9942298BE316EF531E7079FF56B02244FF6FA94A" />
    <Allow ID="ID_ALLOW_A_5E3" FriendlyName="C:\Program Files\git\usr\lib\openssl\engines-1.1\capi.dll Hash Sha1" Hash="40DCB668B58969950BDA2A0505E73CE4728D10DB" />
    <Allow ID="ID_ALLOW_A_5E4" FriendlyName="C:\Program Files\git\usr\lib\openssl\engines-1.1\capi.dll Hash Sha256" Hash="3187C23A2433BDCB9456E3AAFE042E68EF32F8B48C19A00980B42BF3E54EF1A2" />
    <Allow ID="ID_ALLOW_A_5E5" FriendlyName="C:\Program Files\git\usr\lib\openssl\engines-1.1\capi.dll Hash Page Sha1" Hash="6CF33140892DAF243AC8727B429FC742C9A1A791" />
    <Allow ID="ID_ALLOW_A_5E6" FriendlyName="C:\Program Files\git\usr\lib\openssl\engines-1.1\capi.dll Hash Page Sha256" Hash="EF97E7168E83036B75097C5C9C0B05EB4FEE65219A6BB92CF1BD9B1C718CC68C" />
    <Allow ID="ID_ALLOW_A_5E7" FriendlyName="C:\Program Files\git\usr\lib\openssl\engines-1.1\padlock.dll Hash Sha1" Hash="A57F60F88068EC3FEBF938335A439F04B9E7E53F" />
    <Allow ID="ID_ALLOW_A_5E8" FriendlyName="C:\Program Files\git\usr\lib\openssl\engines-1.1\padlock.dll Hash Sha256" Hash="6E8BAC7B7C67BBE5E6A8B0B28DC8E8A2F40A2E8A4F627F5AD0020422A6330A91" />
    <Allow ID="ID_ALLOW_A_5E9" FriendlyName="C:\Program Files\git\usr\lib\openssl\engines-1.1\padlock.dll Hash Page Sha1" Hash="03306E0CAF8F09B87EAD594A00C2FBE648BA256B" />
    <Allow ID="ID_ALLOW_A_5EA" FriendlyName="C:\Program Files\git\usr\lib\openssl\engines-1.1\padlock.dll Hash Page Sha256" Hash="8366784E5062E812BC5A841092BCF4E9DA1C90F4FF0E54273FB542112C6A27FC" />
    <Allow ID="ID_ALLOW_A_5EB" FriendlyName="C:\Program Files\git\usr\lib\gnupg\gpg-check-pattern.exe Hash Sha1" Hash="F34D453D6D63EF0049558FE6CF393FFE1A92260A" />
    <Allow ID="ID_ALLOW_A_5EC" FriendlyName="C:\Program Files\git\usr\lib\gnupg\gpg-check-pattern.exe Hash Sha256" Hash="988BF7FC1EAA41BA008B21C22F62C6B20681DD3CF78C380CF47C2038F7015E13" />
    <Allow ID="ID_ALLOW_A_5ED" FriendlyName="C:\Program Files\git\usr\lib\gnupg\gpg-check-pattern.exe Hash Page Sha1" Hash="2E05F3375A32EF25539EEFE69485DB2F03BBE304" />
    <Allow ID="ID_ALLOW_A_5EE" FriendlyName="C:\Program Files\git\usr\lib\gnupg\gpg-check-pattern.exe Hash Page Sha256" Hash="19930693FAE7EC4BB3DE4B4418834FC93E78FD316D7C0C6F7D8995088C123123" />
    <Allow ID="ID_ALLOW_A_5EF" FriendlyName="C:\Program Files\git\usr\lib\gnupg\gpg-preset-passphrase.exe Hash Sha1" Hash="8A04A61426A47C0CE54DF1500678A99A9BD00CC8" />
    <Allow ID="ID_ALLOW_A_5F0" FriendlyName="C:\Program Files\git\usr\lib\gnupg\gpg-preset-passphrase.exe Hash Sha256" Hash="06F001F1F72508459011C6A2391FFCD831A52A64DD7263D634B4F4368BFCF5F3" />
    <Allow ID="ID_ALLOW_A_5F1" FriendlyName="C:\Program Files\git\usr\lib\gnupg\gpg-preset-passphrase.exe Hash Page Sha1" Hash="6A16F51C8C591A6271566F506FE1BA8920174CDE" />
    <Allow ID="ID_ALLOW_A_5F2" FriendlyName="C:\Program Files\git\usr\lib\gnupg\gpg-preset-passphrase.exe Hash Page Sha256" Hash="C25C8076BB225D3F50123DF601B25470D1482C1F17B67071F3BE93BCE0B81D42" />
    <Allow ID="ID_ALLOW_A_5F3" FriendlyName="C:\Program Files\git\usr\lib\gnupg\gpg-protect-tool.exe Hash Sha1" Hash="5A961A9BD59808F05AC8392467FECFA267D55231" />
    <Allow ID="ID_ALLOW_A_5F4" FriendlyName="C:\Program Files\git\usr\lib\gnupg\gpg-protect-tool.exe Hash Sha256" Hash="21C4BA9CF8434348F203462FC5E21261ACD868DC785B8526C127E893C6D961AE" />
    <Allow ID="ID_ALLOW_A_5F5" FriendlyName="C:\Program Files\git\usr\lib\gnupg\gpg-protect-tool.exe Hash Page Sha1" Hash="264088B1934C4A6097B7449F88B95DE68C98F1DB" />
    <Allow ID="ID_ALLOW_A_5F6" FriendlyName="C:\Program Files\git\usr\lib\gnupg\gpg-protect-tool.exe Hash Page Sha256" Hash="FD1E99C96539B5D3F58E693BC25FA9DCB9C04ADF4839AA395D58DF47D8AB228E" />
    <Allow ID="ID_ALLOW_A_5F7" FriendlyName="C:\Program Files\git\usr\lib\gnupg\gpg-wks-client.exe Hash Sha1" Hash="A1D02567DD85A7283C25D7A95B9FE4C78754064C" />
    <Allow ID="ID_ALLOW_A_5F8" FriendlyName="C:\Program Files\git\usr\lib\gnupg\gpg-wks-client.exe Hash Sha256" Hash="D28B2D6B29CEEDA5FD3B10D70CB5E85114133F7CA97AE68E851C6AB8A3CA9AFB" />
    <Allow ID="ID_ALLOW_A_5F9" FriendlyName="C:\Program Files\git\usr\lib\gnupg\gpg-wks-client.exe Hash Page Sha1" Hash="48FCE9419B15B4F8F0E8E0E5986955D472E58B8B" />
    <Allow ID="ID_ALLOW_A_5FA" FriendlyName="C:\Program Files\git\usr\lib\gnupg\gpg-wks-client.exe Hash Page Sha256" Hash="06C1E722229668FE989DC6E1C75F4C2AA99E447C5B94FF422B509CD8F008D281" />
    <Allow ID="ID_ALLOW_A_5FB" FriendlyName="C:\Program Files\git\usr\lib\gnupg\scdaemon.exe Hash Sha1" Hash="80CE19CA9FB9E08D99D1F6E4C7371071571CB340" />
    <Allow ID="ID_ALLOW_A_5FC" FriendlyName="C:\Program Files\git\usr\lib\gnupg\scdaemon.exe Hash Sha256" Hash="ADF4CF035BCBC1E56E0E42A87C7F5F8D7AE9A9401BB75753F8BD77BE7868D85F" />
    <Allow ID="ID_ALLOW_A_5FD" FriendlyName="C:\Program Files\git\usr\lib\gnupg\scdaemon.exe Hash Page Sha1" Hash="53A04D2CBE3131CB137F77FDEEBF03FCD27A36E0" />
    <Allow ID="ID_ALLOW_A_5FE" FriendlyName="C:\Program Files\git\usr\lib\gnupg\scdaemon.exe Hash Page Sha256" Hash="9B4B3D2C6C274AC9E1BC10EAC7D271136D9D49875F5CB7F0095CC18445397F01" />
    <Allow ID="ID_ALLOW_A_5FF" FriendlyName="C:\Program Files\git\usr\lib\gettext\cldr-plurals.exe Hash Sha1" Hash="25DBC5DC201AB3E2E6724BD628AAD36D3FF66205" />
    <Allow ID="ID_ALLOW_A_600" FriendlyName="C:\Program Files\git\usr\lib\gettext\cldr-plurals.exe Hash Sha256" Hash="B2A678E253FFABE2410B03DF364F47B1C01E4EA375D65E17ADCF3360879D3E7E" />
    <Allow ID="ID_ALLOW_A_601" FriendlyName="C:\Program Files\git\usr\lib\gettext\cldr-plurals.exe Hash Page Sha1" Hash="4D07681B122A6E96D33ED840BA39218FB509A8CF" />
    <Allow ID="ID_ALLOW_A_602" FriendlyName="C:\Program Files\git\usr\lib\gettext\cldr-plurals.exe Hash Page Sha256" Hash="EFB6ACE687DE4790594D6F31057CC366CC0F3F04C8D4D63C22B909A855BE7B5A" />
    <Allow ID="ID_ALLOW_A_603" FriendlyName="C:\Program Files\git\usr\lib\gettext\hostname.exe Hash Sha1" Hash="15617646E09C7E305AC0030F95E61FEC7481FE6E" />
    <Allow ID="ID_ALLOW_A_604" FriendlyName="C:\Program Files\git\usr\lib\gettext\hostname.exe Hash Sha256" Hash="519CD2FA8CDCEF45CB9F6D21CB22284353F0C6C5B3AEAFB4FF949072E4649958" />
    <Allow ID="ID_ALLOW_A_605" FriendlyName="C:\Program Files\git\usr\lib\gettext\hostname.exe Hash Page Sha1" Hash="4F7456FD43C8A388331815FED2A6FEE5158B0F0B" />
    <Allow ID="ID_ALLOW_A_606" FriendlyName="C:\Program Files\git\usr\lib\gettext\hostname.exe Hash Page Sha256" Hash="EA8D396E5FB9A39745C67E474300A4C59E95D3BFCE544B6F19CDB81667139C7A" />
    <Allow ID="ID_ALLOW_A_607" FriendlyName="C:\Program Files\git\usr\lib\gettext\urlget.exe Hash Sha1" Hash="1C624B2620E5278C0BCA526E578120ABCB86FF20" />
    <Allow ID="ID_ALLOW_A_608" FriendlyName="C:\Program Files\git\usr\lib\gettext\urlget.exe Hash Sha256" Hash="44E2696C829BF36E4B3C60D7278E75F60DEC4950C249464B503D91A567533BB3" />
    <Allow ID="ID_ALLOW_A_609" FriendlyName="C:\Program Files\git\usr\lib\gettext\urlget.exe Hash Page Sha1" Hash="4C24F706AF003F61CC91B187626E21315A839473" />
    <Allow ID="ID_ALLOW_A_60A" FriendlyName="C:\Program Files\git\usr\lib\gettext\urlget.exe Hash Page Sha256" Hash="72092985E47ABABC13293C19DD35ED1FD4C1B0C40D84DB4A550EE60588A2E18B" />
    <Allow ID="ID_ALLOW_A_60B" FriendlyName="C:\Program Files\git\usr\lib\gawk\filefuncs.dll Hash Sha1" Hash="B49508163D57F9642BDF13C151834E1D8A01A8FE" />
    <Allow ID="ID_ALLOW_A_60C" FriendlyName="C:\Program Files\git\usr\lib\gawk\filefuncs.dll Hash Sha256" Hash="29261F8C587223E9151250B6F08E0736845FD406D64F7929FC76761E4A6B5D78" />
    <Allow ID="ID_ALLOW_A_60D" FriendlyName="C:\Program Files\git\usr\lib\gawk\filefuncs.dll Hash Page Sha1" Hash="E042B07711CE0315BEEC6AF3C577CE05A4533327" />
    <Allow ID="ID_ALLOW_A_60E" FriendlyName="C:\Program Files\git\usr\lib\gawk\filefuncs.dll Hash Page Sha256" Hash="3D33A17F51E429A15CA4CF02BCB181D001021CE0C8E52FC1526782D95B8621FA" />
    <Allow ID="ID_ALLOW_A_60F" FriendlyName="C:\Program Files\git\usr\lib\gawk\fnmatch.dll Hash Sha1" Hash="6D7ED2B203AA116D5588A6DC89B28EDC45B00F7C" />
    <Allow ID="ID_ALLOW_A_610" FriendlyName="C:\Program Files\git\usr\lib\gawk\fnmatch.dll Hash Sha256" Hash="06CC10BF55F5E723D5D9E1098709EB29D9018B2479140887973943E7043E9158" />
    <Allow ID="ID_ALLOW_A_611" FriendlyName="C:\Program Files\git\usr\lib\gawk\fnmatch.dll Hash Page Sha1" Hash="E4D702503CA03C8C384018F7AE088A3220FEBD57" />
    <Allow ID="ID_ALLOW_A_612" FriendlyName="C:\Program Files\git\usr\lib\gawk\fnmatch.dll Hash Page Sha256" Hash="14DE2CFC2EEAB682A2C0E041D3A8DCEEEDB081E8ED893DFE46F1D89040F52C8C" />
    <Allow ID="ID_ALLOW_A_613" FriendlyName="C:\Program Files\git\usr\lib\gawk\fork.dll Hash Sha1" Hash="BB710322E5E6AB95486F9620644C2E55FC74E3C5" />
    <Allow ID="ID_ALLOW_A_614" FriendlyName="C:\Program Files\git\usr\lib\gawk\fork.dll Hash Sha256" Hash="E8F7D001025B7C83FE9A993018C18233F645DB49C83A8DEE84F21B19F78020B3" />
    <Allow ID="ID_ALLOW_A_615" FriendlyName="C:\Program Files\git\usr\lib\gawk\fork.dll Hash Page Sha1" Hash="385A8B0181966D4E4972C99245D9034BD9E8C85C" />
    <Allow ID="ID_ALLOW_A_616" FriendlyName="C:\Program Files\git\usr\lib\gawk\fork.dll Hash Page Sha256" Hash="37AC67511C935908E21E635A968CC0515353C6827CA906CBB338BFB8F1FB1372" />
    <Allow ID="ID_ALLOW_A_617" FriendlyName="C:\Program Files\git\usr\lib\gawk\inplace.dll Hash Sha1" Hash="3D2BDCDC02AF941EA8780C0F1B782FF736545B9A" />
    <Allow ID="ID_ALLOW_A_618" FriendlyName="C:\Program Files\git\usr\lib\gawk\inplace.dll Hash Sha256" Hash="439D8E4D7B628810618328FF87B3B22DD1B233008884A10E5E920E9F55550071" />
    <Allow ID="ID_ALLOW_A_619" FriendlyName="C:\Program Files\git\usr\lib\gawk\inplace.dll Hash Page Sha1" Hash="B4687F2241CCDBCDC71B1803E15DE3DE66158BF3" />
    <Allow ID="ID_ALLOW_A_61A" FriendlyName="C:\Program Files\git\usr\lib\gawk\inplace.dll Hash Page Sha256" Hash="ACC62E3EEA778BCD4D649923C2180B9C52966DA1DA895F82C9F35CD96DC53B58" />
    <Allow ID="ID_ALLOW_A_61B" FriendlyName="C:\Program Files\git\usr\lib\gawk\intdiv.dll Hash Sha1" Hash="D19DF504125FD7BFE21C89B8B1F5B7F6C73B798C" />
    <Allow ID="ID_ALLOW_A_61C" FriendlyName="C:\Program Files\git\usr\lib\gawk\intdiv.dll Hash Sha256" Hash="180945EFF97C08906BBF1E019BAE43C4C7268CC2058B94CF1E88525883E77BA9" />
    <Allow ID="ID_ALLOW_A_61D" FriendlyName="C:\Program Files\git\usr\lib\gawk\intdiv.dll Hash Page Sha1" Hash="F919ACEDA89378047217898277D9214C0DF97E16" />
    <Allow ID="ID_ALLOW_A_61E" FriendlyName="C:\Program Files\git\usr\lib\gawk\intdiv.dll Hash Page Sha256" Hash="C6E53E7375686A86860B1AF78FC9C191DB842E3FC35B70815E5BAD2CB953FC62" />
    <Allow ID="ID_ALLOW_A_61F" FriendlyName="C:\Program Files\git\usr\lib\gawk\ordchr.dll Hash Sha1" Hash="36072D39CF0DE27F8520B65F45BFDE4E475C0173" />
    <Allow ID="ID_ALLOW_A_620" FriendlyName="C:\Program Files\git\usr\lib\gawk\ordchr.dll Hash Sha256" Hash="BF80A91E46A8BAA19FE3D08E3438F8E90A5D18C3F647D0C471B62755D5D3CF32" />
    <Allow ID="ID_ALLOW_A_621" FriendlyName="C:\Program Files\git\usr\lib\gawk\ordchr.dll Hash Page Sha1" Hash="1ABDEB48C1A157AA2C9D5E6A97EE7986BAE78F31" />
    <Allow ID="ID_ALLOW_A_622" FriendlyName="C:\Program Files\git\usr\lib\gawk\ordchr.dll Hash Page Sha256" Hash="086CD744A0406C769EF95C9EEDC265C0C0E5516678539BBDD736545B0AA17DD7" />
    <Allow ID="ID_ALLOW_A_623" FriendlyName="C:\Program Files\git\usr\lib\gawk\readdir.dll Hash Sha1" Hash="EA99A30D036AF0B2F37D3A5E6FF387AED4E6A1BE" />
    <Allow ID="ID_ALLOW_A_624" FriendlyName="C:\Program Files\git\usr\lib\gawk\readdir.dll Hash Sha256" Hash="B54C323C64C4B6A80956CC2C67F710FDC5F5E31EE234B1413089B7CC70382851" />
    <Allow ID="ID_ALLOW_A_625" FriendlyName="C:\Program Files\git\usr\lib\gawk\readdir.dll Hash Page Sha1" Hash="1BB6922DE937324BE3EAD0FEEA193D59A2C66057" />
    <Allow ID="ID_ALLOW_A_626" FriendlyName="C:\Program Files\git\usr\lib\gawk\readdir.dll Hash Page Sha256" Hash="AB7583B3D4F7AE0A4C5786997649BD9A1A6598D48DC919227288762296D7C108" />
    <Allow ID="ID_ALLOW_A_627" FriendlyName="C:\Program Files\git\usr\lib\gawk\readfile.dll Hash Sha1" Hash="B057B6A4A1EB128CB5C8F943A82A3C8B08556059" />
    <Allow ID="ID_ALLOW_A_628" FriendlyName="C:\Program Files\git\usr\lib\gawk\readfile.dll Hash Sha256" Hash="5C4D9632A83563B7FEFA9A1C603DEFCB4794346FB67E480BC78D2D0C07434ACE" />
    <Allow ID="ID_ALLOW_A_629" FriendlyName="C:\Program Files\git\usr\lib\gawk\readfile.dll Hash Page Sha1" Hash="516E799CDFF926A60CC45132DDFB5264FCE60975" />
    <Allow ID="ID_ALLOW_A_62A" FriendlyName="C:\Program Files\git\usr\lib\gawk\readfile.dll Hash Page Sha256" Hash="38E979847917A8DD7DF2B8A5F852AA8EC05520BE3AC8D97BD85E897449AA647D" />
    <Allow ID="ID_ALLOW_A_62B" FriendlyName="C:\Program Files\git\usr\lib\gawk\revoutput.dll Hash Sha1" Hash="92B657A3636588E4433973847930C32F8A1DF093" />
    <Allow ID="ID_ALLOW_A_62C" FriendlyName="C:\Program Files\git\usr\lib\gawk\revoutput.dll Hash Sha256" Hash="A3EE32CD393614B25EECC5EE99E86E2871E306251D004D1817F93AEA5FA4521E" />
    <Allow ID="ID_ALLOW_A_62D" FriendlyName="C:\Program Files\git\usr\lib\gawk\revoutput.dll Hash Page Sha1" Hash="D4B2D2097281A02AE115B3B0E795E76AC78A3084" />
    <Allow ID="ID_ALLOW_A_62E" FriendlyName="C:\Program Files\git\usr\lib\gawk\revoutput.dll Hash Page Sha256" Hash="503C6F9FC49E7440AAB2B5019DA3C115D0E294BA13C2DD8652AABFF6942B4019" />
    <Allow ID="ID_ALLOW_A_62F" FriendlyName="C:\Program Files\git\usr\lib\gawk\revtwoway.dll Hash Sha1" Hash="5361A116B93813F0ED2C16CF238FB9EB6795A6C6" />
    <Allow ID="ID_ALLOW_A_630" FriendlyName="C:\Program Files\git\usr\lib\gawk\revtwoway.dll Hash Sha256" Hash="67F30D7F54581541A11948724C761A1227083A7D2D84258925C33F39F40E5027" />
    <Allow ID="ID_ALLOW_A_631" FriendlyName="C:\Program Files\git\usr\lib\gawk\revtwoway.dll Hash Page Sha1" Hash="4AD3C488618711F6BB5DADFAA7009DE579F517A1" />
    <Allow ID="ID_ALLOW_A_632" FriendlyName="C:\Program Files\git\usr\lib\gawk\revtwoway.dll Hash Page Sha256" Hash="5E2524526F16338D4E2CE2CBA837AAD3DAE3078055ED475089464E42CFF996CA" />
    <Allow ID="ID_ALLOW_A_633" FriendlyName="C:\Program Files\git\usr\lib\gawk\rwarray.dll Hash Sha1" Hash="A1257CFC702B790CDCFFEEE63EEE79148C567688" />
    <Allow ID="ID_ALLOW_A_634" FriendlyName="C:\Program Files\git\usr\lib\gawk\rwarray.dll Hash Sha256" Hash="29C5D5F7B4F3EC26B7B9628DBBE23A83BEC39AF34425828C2A41505C3FE6C7B2" />
    <Allow ID="ID_ALLOW_A_635" FriendlyName="C:\Program Files\git\usr\lib\gawk\rwarray.dll Hash Page Sha1" Hash="BF37265F5FE6373AC7EBD7AC64A5466BDA4CB7CF" />
    <Allow ID="ID_ALLOW_A_636" FriendlyName="C:\Program Files\git\usr\lib\gawk\rwarray.dll Hash Page Sha256" Hash="5C09A453184648354C259CCB70E155019BF528848B87C2F9BF7741D324397593" />
    <Allow ID="ID_ALLOW_A_637" FriendlyName="C:\Program Files\git\usr\lib\gawk\time.dll Hash Sha1" Hash="106A7EDED830172DAC861D351AA266F308BF65CB" />
    <Allow ID="ID_ALLOW_A_638" FriendlyName="C:\Program Files\git\usr\lib\gawk\time.dll Hash Sha256" Hash="B349CB0C19A69904DD8C1E53E0609E3687750D5A9F5103B05D8E09D07D8E49EC" />
    <Allow ID="ID_ALLOW_A_639" FriendlyName="C:\Program Files\git\usr\lib\gawk\time.dll Hash Page Sha1" Hash="9DB759DCC2B3122D876674D973ABC756A2BF6FE3" />
    <Allow ID="ID_ALLOW_A_63A" FriendlyName="C:\Program Files\git\usr\lib\gawk\time.dll Hash Page Sha256" Hash="A5C9B002FF36436B2A0924920BFA3B57A5D310BF7FB39783A3FA3CFE37F02FF8" />
    <Allow ID="ID_ALLOW_A_63B" FriendlyName="C:\Program Files\git\usr\lib\awk\grcat.exe Hash Sha1" Hash="3AD63D0277BE0AE3F56ABA5C58189ED793B4DDAB" />
    <Allow ID="ID_ALLOW_A_63C" FriendlyName="C:\Program Files\git\usr\lib\awk\grcat.exe Hash Sha256" Hash="89F0500BA3A216D11CAA13DDF6DF208207AA336B8D8179C397F10D34D51AAB8C" />
    <Allow ID="ID_ALLOW_A_63D" FriendlyName="C:\Program Files\git\usr\lib\awk\grcat.exe Hash Page Sha1" Hash="4690767F06C3113814435FAC3D807E9664E27B4B" />
    <Allow ID="ID_ALLOW_A_63E" FriendlyName="C:\Program Files\git\usr\lib\awk\grcat.exe Hash Page Sha256" Hash="F5FCB76806D52D91185950AA44837E276EC4118CB7FB594EA574CE3F86B04A21" />
    <Allow ID="ID_ALLOW_A_63F" FriendlyName="C:\Program Files\git\usr\lib\awk\pwcat.exe Hash Sha1" Hash="3742ED7ECD1663DE5E530F09DC0B4E1BC7134DAA" />
    <Allow ID="ID_ALLOW_A_640" FriendlyName="C:\Program Files\git\usr\lib\awk\pwcat.exe Hash Sha256" Hash="59DDFE2D69385FC428ED1D8266CDF4E5661537435C150C6F98E74CF0EE9D858E" />
    <Allow ID="ID_ALLOW_A_641" FriendlyName="C:\Program Files\git\usr\lib\awk\pwcat.exe Hash Page Sha1" Hash="52602C2F822A4CEDF608A36D57F8297BA75940CB" />
    <Allow ID="ID_ALLOW_A_642" FriendlyName="C:\Program Files\git\usr\lib\awk\pwcat.exe Hash Page Sha256" Hash="7ED031E86F68FD2A49F9DBA16B6F6D3184F388FF64DC3121366067BBD225C526" />
    <Allow ID="ID_ALLOW_A_643" FriendlyName="C:\Program Files\git\usr\bin\arch.exe Hash Sha1" Hash="AFCEF90BA112298FA07F8987CFE029E5F4AEF141" />
    <Allow ID="ID_ALLOW_A_644" FriendlyName="C:\Program Files\git\usr\bin\arch.exe Hash Sha256" Hash="9ACC6DDB6D36D7B60258BA11CB593D65546EB220376F604BD3389ED8061996AC" />
    <Allow ID="ID_ALLOW_A_645" FriendlyName="C:\Program Files\git\usr\bin\arch.exe Hash Page Sha1" Hash="E3E40A77D1EC94F616AC5AD066EACD8D3E9C6B1A" />
    <Allow ID="ID_ALLOW_A_646" FriendlyName="C:\Program Files\git\usr\bin\arch.exe Hash Page Sha256" Hash="75A55BD890C07B8E32A1A337DC808E2C7F9D4EC970BA4B3F9EF29BCD9B9FB8CE" />
    <Allow ID="ID_ALLOW_A_647" FriendlyName="C:\Program Files\git\usr\bin\awk.exe Hash Sha1" Hash="6240E784217244599A1DCA1179AFC6BB2F6B7875" />
    <Allow ID="ID_ALLOW_A_648" FriendlyName="C:\Program Files\git\usr\bin\awk.exe Hash Sha256" Hash="3EFC7C7424CD43081E4BCD7B29B0BA86C920AAA096378309DF6321ECB3280B0E" />
    <Allow ID="ID_ALLOW_A_649" FriendlyName="C:\Program Files\git\usr\bin\awk.exe Hash Page Sha1" Hash="36C03C370E4476D25F9590C0C903C0DDC1DB1D26" />
    <Allow ID="ID_ALLOW_A_64A" FriendlyName="C:\Program Files\git\usr\bin\awk.exe Hash Page Sha256" Hash="5B108E58919CBBDE5150D28B0B188C1644308BCA5A2A53EC87C6EE0DF4B38614" />
    <Allow ID="ID_ALLOW_A_64B" FriendlyName="C:\Program Files\git\usr\bin\b2sum.exe Hash Sha1" Hash="F69B2B8AE87B8C9F7EEEA63BB0A64CE2C5D4A390" />
    <Allow ID="ID_ALLOW_A_64C" FriendlyName="C:\Program Files\git\usr\bin\b2sum.exe Hash Sha256" Hash="375EB8B3C4FCC1FA3AEB8AD223DE863C2E60D8FDC4DBAD438EF23D3BAE5C73AA" />
    <Allow ID="ID_ALLOW_A_64D" FriendlyName="C:\Program Files\git\usr\bin\b2sum.exe Hash Page Sha1" Hash="8B79979F8954F05760CC869E86C2BF0102995E6B" />
    <Allow ID="ID_ALLOW_A_64E" FriendlyName="C:\Program Files\git\usr\bin\b2sum.exe Hash Page Sha256" Hash="C511D1B195949C4A4C4CC3C988E1AF9912D8EE24F67066B316E72491FF7601CF" />
    <Allow ID="ID_ALLOW_A_64F" FriendlyName="C:\Program Files\git\usr\bin\base32.exe Hash Sha1" Hash="26D8F5E31E3E5D875AE13708BC251854B188E0F8" />
    <Allow ID="ID_ALLOW_A_650" FriendlyName="C:\Program Files\git\usr\bin\base32.exe Hash Sha256" Hash="E2ACF86AA205E3CF383F9FFBF900449522E2F0F6CA822B380921BFBBCA64DA12" />
    <Allow ID="ID_ALLOW_A_651" FriendlyName="C:\Program Files\git\usr\bin\base32.exe Hash Page Sha1" Hash="41268124D54586FE8A26FBE55CBB04E8137A627A" />
    <Allow ID="ID_ALLOW_A_652" FriendlyName="C:\Program Files\git\usr\bin\base32.exe Hash Page Sha256" Hash="575E9D4113B80C41B12EF5A357CCC2EEEF0F1CD4AEA37097874555E45942202B" />
    <Allow ID="ID_ALLOW_A_653" FriendlyName="C:\Program Files\git\usr\bin\base64.exe Hash Sha1" Hash="08C149F4BBD2C186977F659510D0F022DD996F58" />
    <Allow ID="ID_ALLOW_A_654" FriendlyName="C:\Program Files\git\usr\bin\base64.exe Hash Sha256" Hash="E72053ADC37ECD09A10D75BD1C86CFFBECB5F69FD71A5E2AF788A6C110460F13" />
    <Allow ID="ID_ALLOW_A_655" FriendlyName="C:\Program Files\git\usr\bin\base64.exe Hash Page Sha1" Hash="191982337D12B825C5DD009D01EB284E7D6A9BD4" />
    <Allow ID="ID_ALLOW_A_656" FriendlyName="C:\Program Files\git\usr\bin\base64.exe Hash Page Sha256" Hash="2A7D8C23AADE001B8D207FBC12C73D41A2B37CC5D3857401A701165B1814726C" />
    <Allow ID="ID_ALLOW_A_657" FriendlyName="C:\Program Files\git\usr\bin\basename.exe Hash Sha1" Hash="728B35C4DE7B906EE2EA6BB856E0A0CEEC597D55" />
    <Allow ID="ID_ALLOW_A_658" FriendlyName="C:\Program Files\git\usr\bin\basename.exe Hash Sha256" Hash="C850DFDDEB44F235AEA69261B729D272851022A3D99D7EE88F91028A931FAA52" />
    <Allow ID="ID_ALLOW_A_659" FriendlyName="C:\Program Files\git\usr\bin\basename.exe Hash Page Sha1" Hash="D9942B7522876DF0951D2978BFE951495C07D408" />
    <Allow ID="ID_ALLOW_A_65A" FriendlyName="C:\Program Files\git\usr\bin\basename.exe Hash Page Sha256" Hash="26B4010985FF1DA46088D2498ABA032C046A0CD83BB784B3EA1298269067A091" />
    <Allow ID="ID_ALLOW_A_65B" FriendlyName="C:\Program Files\git\usr\bin\basenc.exe Hash Sha1" Hash="FABB67B15C49667ABD25EEAF853A50A21D059DF2" />
    <Allow ID="ID_ALLOW_A_65C" FriendlyName="C:\Program Files\git\usr\bin\basenc.exe Hash Sha256" Hash="C240AB8842E44329AA8E47CDE9C4F6DBF954405241A3AFA5C724242B3256F7E2" />
    <Allow ID="ID_ALLOW_A_65D" FriendlyName="C:\Program Files\git\usr\bin\basenc.exe Hash Page Sha1" Hash="73750EB8D45D78F7277F90E627E45C2D732EE688" />
    <Allow ID="ID_ALLOW_A_65E" FriendlyName="C:\Program Files\git\usr\bin\basenc.exe Hash Page Sha256" Hash="B37F23765C52EBC9E76E1860CD5ED8EF582E8BD39E7CD0BD1209D8DEF7547E7C" />
    <Allow ID="ID_ALLOW_A_65F" FriendlyName="C:\Program Files\git\usr\bin\bash.exe Hash Sha1" Hash="1FE28254DB4FA96C675F00D118D8A76CF8C06443" />
    <Allow ID="ID_ALLOW_A_660" FriendlyName="C:\Program Files\git\usr\bin\bash.exe Hash Sha256" Hash="09AEA0B7E0BE3C017DBB2B464AF26CE17584A85558CC6BFB796EC947DAA75E5A" />
    <Allow ID="ID_ALLOW_A_661" FriendlyName="C:\Program Files\git\usr\bin\bash.exe Hash Page Sha1" Hash="027C76966E301F964D925273216256767FA0E603" />
    <Allow ID="ID_ALLOW_A_662" FriendlyName="C:\Program Files\git\usr\bin\bash.exe Hash Page Sha256" Hash="6B15A083BFD75F6951EAD1431DB41B3710A6C061748D57443281BEEB107FE9DB" />
    <Allow ID="ID_ALLOW_A_663" FriendlyName="C:\Program Files\git\usr\bin\bunzip2.exe Hash Sha1" Hash="26CC7D1CA9BB1D57F1AFE523299C4DB087F37387" />
    <Allow ID="ID_ALLOW_A_664" FriendlyName="C:\Program Files\git\usr\bin\bunzip2.exe Hash Sha256" Hash="AC3A69E529814CD6918A38AC3BF74BA0384F9F66071E6A0D723A634EED8A2BED" />
    <Allow ID="ID_ALLOW_A_665" FriendlyName="C:\Program Files\git\usr\bin\bunzip2.exe Hash Page Sha1" Hash="D658FB4A8486F089B46CEA2AC45F65EDE4FF7CFD" />
    <Allow ID="ID_ALLOW_A_666" FriendlyName="C:\Program Files\git\usr\bin\bunzip2.exe Hash Page Sha256" Hash="8B769036418240C3DDF202BE4021ED87CFEA96522D2469AE83B4B30689B7ED98" />
    <Allow ID="ID_ALLOW_A_667" FriendlyName="C:\Program Files\git\usr\bin\bzip2recover.exe Hash Sha1" Hash="A31105735B7C6EC3335FD4EB07529B420F4D8B54" />
    <Allow ID="ID_ALLOW_A_668" FriendlyName="C:\Program Files\git\usr\bin\bzip2recover.exe Hash Sha256" Hash="EC54BF3CAA4CECC01C3EF9D907C88209A460F6F39656B467370CF48CE88D312A" />
    <Allow ID="ID_ALLOW_A_669" FriendlyName="C:\Program Files\git\usr\bin\bzip2recover.exe Hash Page Sha1" Hash="C804C99F11E42BD3E77309050665C95E17E4D3AA" />
    <Allow ID="ID_ALLOW_A_66A" FriendlyName="C:\Program Files\git\usr\bin\bzip2recover.exe Hash Page Sha256" Hash="5F18307B0EE228067D70D819287003604F70740CC554F6602A725206FC349812" />
    <Allow ID="ID_ALLOW_A_66B" FriendlyName="C:\Program Files\git\usr\bin\captoinfo.exe Hash Sha1" Hash="B5404BCD66EBF2FB5C46CA1639E5B1B1EE7A15EA" />
    <Allow ID="ID_ALLOW_A_66C" FriendlyName="C:\Program Files\git\usr\bin\captoinfo.exe Hash Sha256" Hash="F217F55762EF46733181E9EC0E3EDF66183FACBCB08935E33525FC69B66A3EC8" />
    <Allow ID="ID_ALLOW_A_66D" FriendlyName="C:\Program Files\git\usr\bin\captoinfo.exe Hash Page Sha1" Hash="DB5792E8BB3D4C9085A7F58E561F45D0B8BE0FA5" />
    <Allow ID="ID_ALLOW_A_66E" FriendlyName="C:\Program Files\git\usr\bin\captoinfo.exe Hash Page Sha256" Hash="D8019D7D5C2F5F403F392151E7237FFDCBA7A19C7EC956E96395E38BE9CDBF97" />
    <Allow ID="ID_ALLOW_A_66F" FriendlyName="C:\Program Files\git\usr\bin\cat.exe Hash Sha1" Hash="79ABC54D300F7EA6DAF4F619200E02A91D432141" />
    <Allow ID="ID_ALLOW_A_670" FriendlyName="C:\Program Files\git\usr\bin\cat.exe Hash Sha256" Hash="7918F6CA072BDA06C2FD59F48E70F5EB5847905882F6E0836337EFD78093D874" />
    <Allow ID="ID_ALLOW_A_671" FriendlyName="C:\Program Files\git\usr\bin\cat.exe Hash Page Sha1" Hash="5C7CD980121F6092227B0357164656C7382C9389" />
    <Allow ID="ID_ALLOW_A_672" FriendlyName="C:\Program Files\git\usr\bin\cat.exe Hash Page Sha256" Hash="AC781D43858CEB40189374946CCBAC251EBB7B6154CBD62F468248135ECBE900" />
    <Allow ID="ID_ALLOW_A_673" FriendlyName="C:\Program Files\git\usr\bin\chattr.exe Hash Sha1" Hash="45480C0B642715EF623F3698B8A6334578F40251" />
    <Allow ID="ID_ALLOW_A_674" FriendlyName="C:\Program Files\git\usr\bin\chattr.exe Hash Sha256" Hash="E033E843607BEA76EB18AF1624A164A5F960EDE58CD767A873D53597E7226D14" />
    <Allow ID="ID_ALLOW_A_675" FriendlyName="C:\Program Files\git\usr\bin\chattr.exe Hash Page Sha1" Hash="B0F9E7E42CCFF9692DC198DC507C110CB69B31CD" />
    <Allow ID="ID_ALLOW_A_676" FriendlyName="C:\Program Files\git\usr\bin\chattr.exe Hash Page Sha256" Hash="155E3EDD6D4F77145D7909C2DF81F64A1527DBFB5557A3652BBD1DEE5058AC1A" />
    <Allow ID="ID_ALLOW_A_677" FriendlyName="C:\Program Files\git\usr\bin\chcon.exe Hash Sha1" Hash="979DB8B26E44FEDF9F4006D7E789BE4A6A6FD7C2" />
    <Allow ID="ID_ALLOW_A_678" FriendlyName="C:\Program Files\git\usr\bin\chcon.exe Hash Sha256" Hash="98A6831F5A12E1C8243A3E0FB93692BE934E155B58F011D3F81E6324C740804F" />
    <Allow ID="ID_ALLOW_A_679" FriendlyName="C:\Program Files\git\usr\bin\chcon.exe Hash Page Sha1" Hash="B31AF1FE366EE737A07E1CF819753D5A56CB8F22" />
    <Allow ID="ID_ALLOW_A_67A" FriendlyName="C:\Program Files\git\usr\bin\chcon.exe Hash Page Sha256" Hash="3463F4BC268902F7C078E063977571478BAC71A1241A2034F3E8F8288253913F" />
    <Allow ID="ID_ALLOW_A_67B" FriendlyName="C:\Program Files\git\usr\bin\chgrp.exe Hash Sha1" Hash="9E8A4B3D4C50433BAA079082BCB43246014A61D5" />
    <Allow ID="ID_ALLOW_A_67C" FriendlyName="C:\Program Files\git\usr\bin\chgrp.exe Hash Sha256" Hash="D4FDC5152457B1E92B311714E56AD33836D387189F44FADFCBD02F252A60F9C9" />
    <Allow ID="ID_ALLOW_A_67D" FriendlyName="C:\Program Files\git\usr\bin\chgrp.exe Hash Page Sha1" Hash="653FD0C3146011EB1B371AAD81928718E9492545" />
    <Allow ID="ID_ALLOW_A_67E" FriendlyName="C:\Program Files\git\usr\bin\chgrp.exe Hash Page Sha256" Hash="C6A1862E599C1A0440A6176A7DBCAAC0CBF7E570A13F1492075A266C9C071084" />
    <Allow ID="ID_ALLOW_A_67F" FriendlyName="C:\Program Files\git\usr\bin\chmod.exe Hash Sha1" Hash="C7FA72E4E84A883466EE832E465722326D026B76" />
    <Allow ID="ID_ALLOW_A_680" FriendlyName="C:\Program Files\git\usr\bin\chmod.exe Hash Sha256" Hash="5D3A7EC7024587AB3C08B43B70F6567F9FBFC18544C501D6242DCE8A1AED6837" />
    <Allow ID="ID_ALLOW_A_681" FriendlyName="C:\Program Files\git\usr\bin\chmod.exe Hash Page Sha1" Hash="947B7E6659B14F1188D25FC76F87E112A615B43C" />
    <Allow ID="ID_ALLOW_A_682" FriendlyName="C:\Program Files\git\usr\bin\chmod.exe Hash Page Sha256" Hash="C8404CEE741ECC0A4E5A7BA35F3123B4C953C907D7124512CE6A572397F04DAC" />
    <Allow ID="ID_ALLOW_A_683" FriendlyName="C:\Program Files\git\usr\bin\chown.exe Hash Sha1" Hash="B5CB8B21AFB152FBC6E9B691741782C259E70909" />
    <Allow ID="ID_ALLOW_A_684" FriendlyName="C:\Program Files\git\usr\bin\chown.exe Hash Sha256" Hash="74DAFC6CC651B8CE714043C62717602DE1A7CD83ECB2D4D3381F0D0C9C9A12E8" />
    <Allow ID="ID_ALLOW_A_685" FriendlyName="C:\Program Files\git\usr\bin\chown.exe Hash Page Sha1" Hash="79A09A05A65E69100D157BE7D6D6E7E6B0631B6A" />
    <Allow ID="ID_ALLOW_A_686" FriendlyName="C:\Program Files\git\usr\bin\chown.exe Hash Page Sha256" Hash="91A396B5FDE90FE135ECB450632E901DA40E08A61BDC890CA7C42CD81F0FB003" />
    <Allow ID="ID_ALLOW_A_687" FriendlyName="C:\Program Files\git\usr\bin\chroot.exe Hash Sha1" Hash="5A485A53E4380AF9A842A85B8FB44E8E32DB4369" />
    <Allow ID="ID_ALLOW_A_688" FriendlyName="C:\Program Files\git\usr\bin\chroot.exe Hash Sha256" Hash="79857CAAA964653AAED265ADB9DA79432BE43227501E82F65318ECE9247B9176" />
    <Allow ID="ID_ALLOW_A_689" FriendlyName="C:\Program Files\git\usr\bin\chroot.exe Hash Page Sha1" Hash="F9D40793E8BB275231D5B01FF0DB8348057297AE" />
    <Allow ID="ID_ALLOW_A_68A" FriendlyName="C:\Program Files\git\usr\bin\chroot.exe Hash Page Sha256" Hash="816BA30F16493745F83541E5FF4E340DA55C599070BC19E8E92B90BCB85B6B01" />
    <Allow ID="ID_ALLOW_A_68B" FriendlyName="C:\Program Files\git\usr\bin\cksum.exe Hash Sha1" Hash="29E0E3862E9771FB833DE9F43F2337BD96B14A79" />
    <Allow ID="ID_ALLOW_A_68C" FriendlyName="C:\Program Files\git\usr\bin\cksum.exe Hash Sha256" Hash="719C7553E8096613EE5D9CE52DEF5FAB71313A3F36D54774F3E5158CA4061731" />
    <Allow ID="ID_ALLOW_A_68D" FriendlyName="C:\Program Files\git\usr\bin\cksum.exe Hash Page Sha1" Hash="B0EAE541E1FCCD4B53328ADA1A4E743599B7C01B" />
    <Allow ID="ID_ALLOW_A_68E" FriendlyName="C:\Program Files\git\usr\bin\cksum.exe Hash Page Sha256" Hash="BE8352640D7843A3C27A9DFFF6BF42F6C01E0AB3CAB08AD70572F9359A9617AF" />
    <Allow ID="ID_ALLOW_A_68F" FriendlyName="C:\Program Files\git\usr\bin\clear.exe Hash Sha1" Hash="728673353CAA9B5B34D6EE02A685970C4638501C" />
    <Allow ID="ID_ALLOW_A_690" FriendlyName="C:\Program Files\git\usr\bin\clear.exe Hash Sha256" Hash="3C3A8D95E7B0ABDC50C32119534AC1D7C2CEE79F71FB851D29FC9AD7BB01381C" />
    <Allow ID="ID_ALLOW_A_691" FriendlyName="C:\Program Files\git\usr\bin\clear.exe Hash Page Sha1" Hash="81A2895E771AD7F67C8ABBCF79DAEDD5BE0D3281" />
    <Allow ID="ID_ALLOW_A_692" FriendlyName="C:\Program Files\git\usr\bin\clear.exe Hash Page Sha256" Hash="08650D81219CA130D5232CF7A1451334B6431B88313DC73331557C396C7A8FA6" />
    <Allow ID="ID_ALLOW_A_693" FriendlyName="C:\Program Files\git\usr\bin\cmp.exe Hash Sha1" Hash="1C80B99332EF07EAB3B69169133874324981A6CF" />
    <Allow ID="ID_ALLOW_A_694" FriendlyName="C:\Program Files\git\usr\bin\cmp.exe Hash Sha256" Hash="F1C42ADADAC2E87C9EBF89AE616537677BD4EC23623311C1797BDE52C0F25585" />
    <Allow ID="ID_ALLOW_A_695" FriendlyName="C:\Program Files\git\usr\bin\cmp.exe Hash Page Sha1" Hash="5DC0EBC58933E2C5FBD85C411D343D6A4E3D4D91" />
    <Allow ID="ID_ALLOW_A_696" FriendlyName="C:\Program Files\git\usr\bin\cmp.exe Hash Page Sha256" Hash="E5ED896EAD273D803FF46C54F1B1CEF2B522305AB35270B8A31DE9F621504538" />
    <Allow ID="ID_ALLOW_A_697" FriendlyName="C:\Program Files\git\usr\bin\column.exe Hash Sha1" Hash="62F1FD6FA28A3BF4EAC490A7529EAEBF3FCEE519" />
    <Allow ID="ID_ALLOW_A_698" FriendlyName="C:\Program Files\git\usr\bin\column.exe Hash Sha256" Hash="8A03D38C49C1DF13EF4D2D7B68BCD3F874E5B64044C78BD0149A89B0BC721AC2" />
    <Allow ID="ID_ALLOW_A_699" FriendlyName="C:\Program Files\git\usr\bin\column.exe Hash Page Sha1" Hash="3FAA273A57D9C66F672C7C706CA450A3D2A821DE" />
    <Allow ID="ID_ALLOW_A_69A" FriendlyName="C:\Program Files\git\usr\bin\column.exe Hash Page Sha256" Hash="AEB4A8468F6183FC1D7E50FD9879B4077E72DA9AC6D12B8D52E9A92560CD0AD3" />
    <Allow ID="ID_ALLOW_A_69B" FriendlyName="C:\Program Files\git\usr\bin\comm.exe Hash Sha1" Hash="04E5D802D4D8F18E2FFE2EDC14A83831C1BAC71F" />
    <Allow ID="ID_ALLOW_A_69C" FriendlyName="C:\Program Files\git\usr\bin\comm.exe Hash Sha256" Hash="D83702BC97F9DA216EF7C4AA863ADD3FFAC6385056B0FE996E40BBC62933DBF1" />
    <Allow ID="ID_ALLOW_A_69D" FriendlyName="C:\Program Files\git\usr\bin\comm.exe Hash Page Sha1" Hash="9BE411E969B5800411EF1CDE502F83762B15905C" />
    <Allow ID="ID_ALLOW_A_69E" FriendlyName="C:\Program Files\git\usr\bin\comm.exe Hash Page Sha256" Hash="44B1F442D144A86F8E080E08211EBBA26B370244AC5CCD7AFFD9E1A32C5652C6" />
    <Allow ID="ID_ALLOW_A_69F" FriendlyName="C:\Program Files\git\usr\bin\cp.exe Hash Sha1" Hash="1B92E5378FFFA0E8BDE73CA4EEE15496E858CE0E" />
    <Allow ID="ID_ALLOW_A_6A0" FriendlyName="C:\Program Files\git\usr\bin\cp.exe Hash Sha256" Hash="EE68373B10C9AB32DD4E955B722D5216CA8EDE7B39D7D4140420500A9B9261C7" />
    <Allow ID="ID_ALLOW_A_6A1" FriendlyName="C:\Program Files\git\usr\bin\cp.exe Hash Page Sha1" Hash="96E6B50283A2BCE6C25DABEB68B80939400BD432" />
    <Allow ID="ID_ALLOW_A_6A2" FriendlyName="C:\Program Files\git\usr\bin\cp.exe Hash Page Sha256" Hash="2DE165AF4C249EE4D3E3EA7823A6E0209CCF6558E450EBFA160418A753BBDDAD" />
    <Allow ID="ID_ALLOW_A_6A3" FriendlyName="C:\Program Files\git\usr\bin\csplit.exe Hash Sha1" Hash="4079697BE37700F918684F0A88A59C9666A02B5C" />
    <Allow ID="ID_ALLOW_A_6A4" FriendlyName="C:\Program Files\git\usr\bin\csplit.exe Hash Sha256" Hash="59A1B5218AF25F74B8CE083879AA5710E0BC1B85F8798C52BFFEC2C6EDFC6E96" />
    <Allow ID="ID_ALLOW_A_6A5" FriendlyName="C:\Program Files\git\usr\bin\csplit.exe Hash Page Sha1" Hash="C5438F852486C3A4DD603B8FE9F51BED026A26AB" />
    <Allow ID="ID_ALLOW_A_6A6" FriendlyName="C:\Program Files\git\usr\bin\csplit.exe Hash Page Sha256" Hash="146284C0F26FC1F7E9F17AFD48747E04F8ED4BCFC6BF7F20A4FF3A259EEEF76B" />
    <Allow ID="ID_ALLOW_A_6A7" FriendlyName="C:\Program Files\git\usr\bin\cut.exe Hash Sha1" Hash="9E1C62BE9CB16AC0AABFB2331237D34257EBE554" />
    <Allow ID="ID_ALLOW_A_6A8" FriendlyName="C:\Program Files\git\usr\bin\cut.exe Hash Sha256" Hash="0B36A511D985E2DC289F06C55825FC6A6DB2CC06936040317D0DF9496593D589" />
    <Allow ID="ID_ALLOW_A_6A9" FriendlyName="C:\Program Files\git\usr\bin\cut.exe Hash Page Sha1" Hash="5CC3874647EC6CDBC9FF7F65A74C863EC50DA527" />
    <Allow ID="ID_ALLOW_A_6AA" FriendlyName="C:\Program Files\git\usr\bin\cut.exe Hash Page Sha256" Hash="991900FD4C0DBCC83ADE5FD5DC7725AAFB181671F7A793B0323AFC3D0C63C527" />
    <Allow ID="ID_ALLOW_A_6AB" FriendlyName="C:\Program Files\git\usr\bin\cygcheck.exe Hash Sha1" Hash="5B7B0B5ACDAA1F5B605AFD694A16E924F8B4363B" />
    <Allow ID="ID_ALLOW_A_6AC" FriendlyName="C:\Program Files\git\usr\bin\cygcheck.exe Hash Sha256" Hash="2566639DC914B49F81D109F7A48A80B2749FDE9936D8B345AB5336CBBBB25383" />
    <Allow ID="ID_ALLOW_A_6AD" FriendlyName="C:\Program Files\git\usr\bin\cygcheck.exe Hash Page Sha1" Hash="ED9227CA363F92E5BA1F4095595B2F196DB6F0B3" />
    <Allow ID="ID_ALLOW_A_6AE" FriendlyName="C:\Program Files\git\usr\bin\cygcheck.exe Hash Page Sha256" Hash="98572CDEACE0430115B74A2EC1EA65D74BA27DB0B63FCE986F666081DF4B218B" />
    <Allow ID="ID_ALLOW_A_6AF" FriendlyName="C:\Program Files\git\usr\bin\cygpath.exe Hash Sha1" Hash="65831E2E61969C83A1F1E37B6C591828231F6482" />
    <Allow ID="ID_ALLOW_A_6B0" FriendlyName="C:\Program Files\git\usr\bin\cygpath.exe Hash Sha256" Hash="A39D0BF4821B39506A9FBD5647F1F6185E19E7E2410A9E05035FDC0158A88616" />
    <Allow ID="ID_ALLOW_A_6B1" FriendlyName="C:\Program Files\git\usr\bin\cygpath.exe Hash Page Sha1" Hash="2D761D9FD2DA1D65317C3EB91D7E8A49B3373D79" />
    <Allow ID="ID_ALLOW_A_6B2" FriendlyName="C:\Program Files\git\usr\bin\cygpath.exe Hash Page Sha256" Hash="C1FF0FC9186CB15DD71CF8281A4C90AC8B6DE89D6FF105CECE577D407C10F2EE" />
    <Allow ID="ID_ALLOW_A_6B3" FriendlyName="C:\Program Files\git\usr\bin\cygwin-console-helper.exe Hash Sha1" Hash="E1A2A33BB83B2FE665F17F0B2C0FA4BB18B6175B" />
    <Allow ID="ID_ALLOW_A_6B4" FriendlyName="C:\Program Files\git\usr\bin\cygwin-console-helper.exe Hash Sha256" Hash="EE0CE9AB5914351C306D22143DD5F6149EC400AC1B2DDE32637BB366E0C1BA7E" />
    <Allow ID="ID_ALLOW_A_6B5" FriendlyName="C:\Program Files\git\usr\bin\cygwin-console-helper.exe Hash Page Sha1" Hash="6557FFFCD6AC5EBC4CC1AF333852CDF4D8294773" />
    <Allow ID="ID_ALLOW_A_6B6" FriendlyName="C:\Program Files\git\usr\bin\cygwin-console-helper.exe Hash Page Sha256" Hash="6A38E1BC79906CC6AFB8B07BABB8438D2EB1323BA7221EB404C2F03CAC50C582" />
    <Allow ID="ID_ALLOW_A_6B7" FriendlyName="C:\Program Files\git\usr\bin\d2u.exe Hash Sha1" Hash="DAAB4F73AE20C5310CCD15F0E7F5F137B0B89B1D" />
    <Allow ID="ID_ALLOW_A_6B8" FriendlyName="C:\Program Files\git\usr\bin\d2u.exe Hash Sha256" Hash="E5F77F0762A52EF2FF1A325E525A47B9AE969F196275D10259BD017987F803BB" />
    <Allow ID="ID_ALLOW_A_6B9" FriendlyName="C:\Program Files\git\usr\bin\d2u.exe Hash Page Sha1" Hash="DFA3B74C753573C49C7EE001AB4B0EBDB116F62C" />
    <Allow ID="ID_ALLOW_A_6BA" FriendlyName="C:\Program Files\git\usr\bin\d2u.exe Hash Page Sha256" Hash="64C14B3368CBCC3D7EC01C4B92FE87443874000CD5303AD7EBB17CCA9293E0A3" />
    <Allow ID="ID_ALLOW_A_6BB" FriendlyName="C:\Program Files\git\usr\bin\dash.exe Hash Sha1" Hash="DCD6A40140EE9AF330B2AA6EB1F94BD8C44E8FBE" />
    <Allow ID="ID_ALLOW_A_6BC" FriendlyName="C:\Program Files\git\usr\bin\dash.exe Hash Sha256" Hash="32EF2301A2E2B4BBD639D8098C7528CB29FD0C4A81A6B46927207E1C28C27D7D" />
    <Allow ID="ID_ALLOW_A_6BD" FriendlyName="C:\Program Files\git\usr\bin\dash.exe Hash Page Sha1" Hash="F1E874641E4F88970A467E203631DB27B085652C" />
    <Allow ID="ID_ALLOW_A_6BE" FriendlyName="C:\Program Files\git\usr\bin\dash.exe Hash Page Sha256" Hash="434E0BDC63B1E356EE8CA681034A582E4A0F305539D2C8214D980E055E9A64D4" />
    <Allow ID="ID_ALLOW_A_6BF" FriendlyName="C:\Program Files\git\usr\bin\date.exe Hash Sha1" Hash="9D72D6FC39505279282148B02E578CBC9F436A29" />
    <Allow ID="ID_ALLOW_A_6C0" FriendlyName="C:\Program Files\git\usr\bin\date.exe Hash Sha256" Hash="9298A02B707A956F770B31D9FF9D474E55DB81A70E23E6224B47362099CDB0A0" />
    <Allow ID="ID_ALLOW_A_6C1" FriendlyName="C:\Program Files\git\usr\bin\date.exe Hash Page Sha1" Hash="0D24FD17AC2EEFA4E3B2623623D2C0AE9A011E8E" />
    <Allow ID="ID_ALLOW_A_6C2" FriendlyName="C:\Program Files\git\usr\bin\date.exe Hash Page Sha256" Hash="8645B874A6A887A08E8436C579D0D95B2FF21B13822E46071FBFD73A57CE09B9" />
    <Allow ID="ID_ALLOW_A_6C3" FriendlyName="C:\Program Files\git\usr\bin\dd.exe Hash Sha1" Hash="607BA75FB20185DD0491085310E85BEF1D9E4B09" />
    <Allow ID="ID_ALLOW_A_6C4" FriendlyName="C:\Program Files\git\usr\bin\dd.exe Hash Sha256" Hash="6B83F2466ACDD5E30B2090BAD3D9F8F75B79A9048B40374BD003FF4AA6263959" />
    <Allow ID="ID_ALLOW_A_6C5" FriendlyName="C:\Program Files\git\usr\bin\dd.exe Hash Page Sha1" Hash="EAFEF628DC12DC7C202BC738558A1F23B8260131" />
    <Allow ID="ID_ALLOW_A_6C6" FriendlyName="C:\Program Files\git\usr\bin\dd.exe Hash Page Sha256" Hash="86D5D4870AC6327900AF00C86F329FF7F5178B229921D3ECA43FE051FA0252A4" />
    <Allow ID="ID_ALLOW_A_6C7" FriendlyName="C:\Program Files\git\usr\bin\df.exe Hash Sha1" Hash="2CAE81DC45AF6BB1C360F1F610DB9AFB192B7403" />
    <Allow ID="ID_ALLOW_A_6C8" FriendlyName="C:\Program Files\git\usr\bin\df.exe Hash Sha256" Hash="16D2B79FB49D9AA05926D9A056C4BFB15E36804D8F5C7391CA44581A659486C9" />
    <Allow ID="ID_ALLOW_A_6C9" FriendlyName="C:\Program Files\git\usr\bin\df.exe Hash Page Sha1" Hash="A8DC17074A3F4BC50391E9090CBF1555E5268193" />
    <Allow ID="ID_ALLOW_A_6CA" FriendlyName="C:\Program Files\git\usr\bin\df.exe Hash Page Sha256" Hash="1A1251FEE6BDED2B4DA2EB9E334DE16717F17ED3D91B2C1933A34B03E8DEEF4D" />
    <Allow ID="ID_ALLOW_A_6CB" FriendlyName="C:\Program Files\git\usr\bin\diff.exe Hash Sha1" Hash="B9C146E9BAF2D9EDC0C6F1F32759D20453F8F53F" />
    <Allow ID="ID_ALLOW_A_6CC" FriendlyName="C:\Program Files\git\usr\bin\diff.exe Hash Sha256" Hash="534B5213204118DE7E4A313BB399074D5464B545DA2358AC7B31A3A690D2BCBE" />
    <Allow ID="ID_ALLOW_A_6CD" FriendlyName="C:\Program Files\git\usr\bin\diff.exe Hash Page Sha1" Hash="D7F9CA686672A6A0901460942B7E5A2A941C39E1" />
    <Allow ID="ID_ALLOW_A_6CE" FriendlyName="C:\Program Files\git\usr\bin\diff.exe Hash Page Sha256" Hash="51D0C9B97B43E6FE6D945F532CE7623FC989D7A53926D28633F81E94B5DFF789" />
    <Allow ID="ID_ALLOW_A_6CF" FriendlyName="C:\Program Files\git\usr\bin\diff3.exe Hash Sha1" Hash="DA478C81F4FEE7F1178F70F7DF8EE3403A2519F8" />
    <Allow ID="ID_ALLOW_A_6D0" FriendlyName="C:\Program Files\git\usr\bin\diff3.exe Hash Sha256" Hash="8C76436A9321119D514D5DCC101AA5417F4DDD35E7CB407885EAF2EB737E948D" />
    <Allow ID="ID_ALLOW_A_6D1" FriendlyName="C:\Program Files\git\usr\bin\diff3.exe Hash Page Sha1" Hash="4FB3502ECAFFF0407AF3E86F6B97CC45A37B547A" />
    <Allow ID="ID_ALLOW_A_6D2" FriendlyName="C:\Program Files\git\usr\bin\diff3.exe Hash Page Sha256" Hash="C4E785AB12BECB8ECFC137A021FA2CA762985D7BD4512BA01116C9E745966381" />
    <Allow ID="ID_ALLOW_A_6D3" FriendlyName="C:\Program Files\git\usr\bin\dir.exe Hash Sha1" Hash="5C76B11200AD3C2D3038BF665E7086C9A5314791" />
    <Allow ID="ID_ALLOW_A_6D4" FriendlyName="C:\Program Files\git\usr\bin\dir.exe Hash Sha256" Hash="2EC8B9F82972B0DE0135CD4292EF6C8B7187953185D98C10FF9C67C69E72E8C3" />
    <Allow ID="ID_ALLOW_A_6D5" FriendlyName="C:\Program Files\git\usr\bin\dir.exe Hash Page Sha1" Hash="DDC56985352CC3925F9B3C79CFA5574ACCB21D12" />
    <Allow ID="ID_ALLOW_A_6D6" FriendlyName="C:\Program Files\git\usr\bin\dir.exe Hash Page Sha256" Hash="F9EE447CA22B14ADBD4C12C30C64A9CDFFB4D598F45BAE448E859B48EC40CB05" />
    <Allow ID="ID_ALLOW_A_6D7" FriendlyName="C:\Program Files\git\usr\bin\dircolors.exe Hash Sha1" Hash="FAC63CD81CD7E24DEC7D29CD341226F0A48E70A9" />
    <Allow ID="ID_ALLOW_A_6D8" FriendlyName="C:\Program Files\git\usr\bin\dircolors.exe Hash Sha256" Hash="CF98CE93D5179DEA31D012979FD6F256A22639230D4ED03AAD8F13D8E1169735" />
    <Allow ID="ID_ALLOW_A_6D9" FriendlyName="C:\Program Files\git\usr\bin\dircolors.exe Hash Page Sha1" Hash="D40DC9C0A2770E91643CF0870C3A34C93BCB2F85" />
    <Allow ID="ID_ALLOW_A_6DA" FriendlyName="C:\Program Files\git\usr\bin\dircolors.exe Hash Page Sha256" Hash="4FE250D3931881BEBC6655C411FF5F3DAF75557E687500B9E95384187FD885A9" />
    <Allow ID="ID_ALLOW_A_6DB" FriendlyName="C:\Program Files\git\usr\bin\dirmngr-client.exe Hash Sha1" Hash="11A480B472446FCDA648423A10A7ECA4FDF4D397" />
    <Allow ID="ID_ALLOW_A_6DC" FriendlyName="C:\Program Files\git\usr\bin\dirmngr-client.exe Hash Sha256" Hash="9EDD4560C6C7A31913E1E4E6DB6D52BC3D1BB41B9769B9A88BAAB1E34CCB2A61" />
    <Allow ID="ID_ALLOW_A_6DD" FriendlyName="C:\Program Files\git\usr\bin\dirmngr-client.exe Hash Page Sha1" Hash="7BA0E2F2CD5A10ED9B9BC6B9EC18B79CFC3DD0EA" />
    <Allow ID="ID_ALLOW_A_6DE" FriendlyName="C:\Program Files\git\usr\bin\dirmngr-client.exe Hash Page Sha256" Hash="BD9B8C56B679E15C6A3DF8B8F25C239E702861C4416F95EB14E3FBAD2F3D2E51" />
    <Allow ID="ID_ALLOW_A_6DF" FriendlyName="C:\Program Files\git\usr\bin\dirmngr.exe Hash Sha1" Hash="081812285CAEBAFD74FDCF80CE0FDF4DA5F22EDD" />
    <Allow ID="ID_ALLOW_A_6E0" FriendlyName="C:\Program Files\git\usr\bin\dirmngr.exe Hash Sha256" Hash="B162CED2E1CC60FE633193928DED2CDAFD26F5B01969139CA2969AD4A2912891" />
    <Allow ID="ID_ALLOW_A_6E1" FriendlyName="C:\Program Files\git\usr\bin\dirmngr.exe Hash Page Sha1" Hash="F9C72351E231A289109637B5097728F2C0B9C55B" />
    <Allow ID="ID_ALLOW_A_6E2" FriendlyName="C:\Program Files\git\usr\bin\dirmngr.exe Hash Page Sha256" Hash="1E0B59C541370E3998A393F6B94F0E62F06CD16027B8F8EBCBE33777597B3A80" />
    <Allow ID="ID_ALLOW_A_6E3" FriendlyName="C:\Program Files\git\usr\bin\dirname.exe Hash Sha1" Hash="4CEA333DF316ED0ED37A47EECA0A49A3D6C6066D" />
    <Allow ID="ID_ALLOW_A_6E4" FriendlyName="C:\Program Files\git\usr\bin\dirname.exe Hash Sha256" Hash="DC197444E5A8B89F5E82F2EBD97623DB00F60DCF9BC8A3D8E76A529A23FD9AB9" />
    <Allow ID="ID_ALLOW_A_6E5" FriendlyName="C:\Program Files\git\usr\bin\dirname.exe Hash Page Sha1" Hash="FA496618B5B749E3459ECFA4F55B172DC38B524B" />
    <Allow ID="ID_ALLOW_A_6E6" FriendlyName="C:\Program Files\git\usr\bin\dirname.exe Hash Page Sha256" Hash="6A0712CD71CCF90347E67869B49DF813496C2C0194CB705DD46101987259E820" />
    <Allow ID="ID_ALLOW_A_6E7" FriendlyName="C:\Program Files\git\usr\bin\du.exe Hash Sha1" Hash="B738387A1A2358D9E7ABD4CA9875D95B6E4F8AF8" />
    <Allow ID="ID_ALLOW_A_6E8" FriendlyName="C:\Program Files\git\usr\bin\du.exe Hash Sha256" Hash="2EE761A9FF870F233153081F1A852880DF892588346F03821A442737D2464A3F" />
    <Allow ID="ID_ALLOW_A_6E9" FriendlyName="C:\Program Files\git\usr\bin\du.exe Hash Page Sha1" Hash="ABD7B5C53CC7B10F65CEB2EA0A658A49367377EF" />
    <Allow ID="ID_ALLOW_A_6EA" FriendlyName="C:\Program Files\git\usr\bin\du.exe Hash Page Sha256" Hash="5BFF2451261ECF0023CC39691FBE68933FC74B5BD6BD4C08C41C6BA4C1865E7F" />
    <Allow ID="ID_ALLOW_A_6EB" FriendlyName="C:\Program Files\git\usr\bin\dumpsexp.exe Hash Sha1" Hash="0C1004B13AD00E28BCB84A3C4B0E9F8896418C08" />
    <Allow ID="ID_ALLOW_A_6EC" FriendlyName="C:\Program Files\git\usr\bin\dumpsexp.exe Hash Sha256" Hash="7B94AEB09C1B475A8E2A7DAACB98D1B093C199778457B464F06BBAAD7EF0C6FB" />
    <Allow ID="ID_ALLOW_A_6ED" FriendlyName="C:\Program Files\git\usr\bin\dumpsexp.exe Hash Page Sha1" Hash="45C30DABA99E097DF087059ABE24E88D154E8073" />
    <Allow ID="ID_ALLOW_A_6EE" FriendlyName="C:\Program Files\git\usr\bin\dumpsexp.exe Hash Page Sha256" Hash="4B4B18CCC4DA20BE8E8BF86B73310C49016314FE43BAEDE0A961C68BF867732C" />
    <Allow ID="ID_ALLOW_A_6EF" FriendlyName="C:\Program Files\git\usr\bin\echo.exe Hash Sha1" Hash="40361946CC5EDFE592CD33D194C649DB6C79C3C3" />
    <Allow ID="ID_ALLOW_A_6F0" FriendlyName="C:\Program Files\git\usr\bin\echo.exe Hash Sha256" Hash="361C1402ABCC3D95E244F13B90C64B5210F22B03A3DD6FF916C74F29F1B64C97" />
    <Allow ID="ID_ALLOW_A_6F1" FriendlyName="C:\Program Files\git\usr\bin\echo.exe Hash Page Sha1" Hash="138A01327675775CC4917DD863C35D4B8010B296" />
    <Allow ID="ID_ALLOW_A_6F2" FriendlyName="C:\Program Files\git\usr\bin\echo.exe Hash Page Sha256" Hash="B786A449113413128E738340FE11BD8D985174277B632F0D2AA65E6C7AF03120" />
    <Allow ID="ID_ALLOW_A_6F3" FriendlyName="C:\Program Files\git\usr\bin\env.exe Hash Sha1" Hash="34E44BB71D209386A7720D7DD47FBB57F3A3B68C" />
    <Allow ID="ID_ALLOW_A_6F4" FriendlyName="C:\Program Files\git\usr\bin\env.exe Hash Sha256" Hash="A4CEC00DF4D166D1548EB28C608161972CC13AADBB95C1644FFDB31A28F914A1" />
    <Allow ID="ID_ALLOW_A_6F5" FriendlyName="C:\Program Files\git\usr\bin\env.exe Hash Page Sha1" Hash="F566E7373D59F84052333256DDE334C6FD40519F" />
    <Allow ID="ID_ALLOW_A_6F6" FriendlyName="C:\Program Files\git\usr\bin\env.exe Hash Page Sha256" Hash="618FC2B6FED6A15D334A72C4ED9737AA38CFBB3AF83BA8ECEAA56318F21F477B" />
    <Allow ID="ID_ALLOW_A_6F7" FriendlyName="C:\Program Files\git\usr\bin\envsubst.exe Hash Sha1" Hash="27D133736D18489CFF38AED63F82CEFB4A10D37F" />
    <Allow ID="ID_ALLOW_A_6F8" FriendlyName="C:\Program Files\git\usr\bin\envsubst.exe Hash Sha256" Hash="F63B02A61ACEC0EECB426D73BBA1DDC7BEBC5A0D8613D4CBBD6FF2134A0227E2" />
    <Allow ID="ID_ALLOW_A_6F9" FriendlyName="C:\Program Files\git\usr\bin\envsubst.exe Hash Page Sha1" Hash="7FC83288540A0CA9183BF0417144BCF55C1247CA" />
    <Allow ID="ID_ALLOW_A_6FA" FriendlyName="C:\Program Files\git\usr\bin\envsubst.exe Hash Page Sha256" Hash="B64432063A364CFC4BAE6CBBD509FB74EBDB8E0C21144BA252DDAC6D390C0605" />
    <Allow ID="ID_ALLOW_A_6FB" FriendlyName="C:\Program Files\git\usr\bin\ex.exe Hash Sha1" Hash="93DE761FB1FC0FA1C54F98142179E8FDA9FD00D5" />
    <Allow ID="ID_ALLOW_A_6FC" FriendlyName="C:\Program Files\git\usr\bin\ex.exe Hash Sha256" Hash="0A1F0F1258CB28D887908565F72C1E75EEE1A8CBB5DAD52E1758DB3B121AA3D6" />
    <Allow ID="ID_ALLOW_A_6FD" FriendlyName="C:\Program Files\git\usr\bin\ex.exe Hash Page Sha1" Hash="7C5E775836AF4440B3CBAC2D0F321B4ABB726710" />
    <Allow ID="ID_ALLOW_A_6FE" FriendlyName="C:\Program Files\git\usr\bin\ex.exe Hash Page Sha256" Hash="7446F604903349A94BA7C314E7C801E8C2C2E297120568067015E7F28121793A" />
    <Allow ID="ID_ALLOW_A_6FF" FriendlyName="C:\Program Files\git\usr\bin\expand.exe Hash Sha1" Hash="BB77B09FC43019FD6C7D1366E313A995FC96D968" />
    <Allow ID="ID_ALLOW_A_700" FriendlyName="C:\Program Files\git\usr\bin\expand.exe Hash Sha256" Hash="CDE61F1005D7B140ECB5C7351CE561D8D5D6E8961B56BF6D9C225756E0F56557" />
    <Allow ID="ID_ALLOW_A_701" FriendlyName="C:\Program Files\git\usr\bin\expand.exe Hash Page Sha1" Hash="74599341C3E46A6D8A3DFC40806A52F84819A6A7" />
    <Allow ID="ID_ALLOW_A_702" FriendlyName="C:\Program Files\git\usr\bin\expand.exe Hash Page Sha256" Hash="22A8C6564EBB762759C698290AA5D23363D398217CB5FBAB41FC3539827F2A93" />
    <Allow ID="ID_ALLOW_A_703" FriendlyName="C:\Program Files\git\usr\bin\expr.exe Hash Sha1" Hash="C11D97433ABA10E54751C2B9E37DDAF26E41F817" />
    <Allow ID="ID_ALLOW_A_704" FriendlyName="C:\Program Files\git\usr\bin\expr.exe Hash Sha256" Hash="1BD83222CECDE785BD2030C42026B96DFA2F91F0304AD47D022CDA7E664D7989" />
    <Allow ID="ID_ALLOW_A_705" FriendlyName="C:\Program Files\git\usr\bin\expr.exe Hash Page Sha1" Hash="B4C574DB7629DFCFA4FF269452008543E77A5B37" />
    <Allow ID="ID_ALLOW_A_706" FriendlyName="C:\Program Files\git\usr\bin\expr.exe Hash Page Sha256" Hash="D14B3D0BAC295EA410CA967A17C550AE13273434EB42C76E435CD3202C98BCAE" />
    <Allow ID="ID_ALLOW_A_707" FriendlyName="C:\Program Files\git\usr\bin\factor.exe Hash Sha1" Hash="E3E3F00E55E696CA093A908497E27EC11A218731" />
    <Allow ID="ID_ALLOW_A_708" FriendlyName="C:\Program Files\git\usr\bin\factor.exe Hash Sha256" Hash="12F04259DF6C53A1E8DF5E4F5BBD9DDA1EEE47B31D6DA2DC6A0D0B1E0A996C42" />
    <Allow ID="ID_ALLOW_A_709" FriendlyName="C:\Program Files\git\usr\bin\factor.exe Hash Page Sha1" Hash="6ACD0822BB064D9A091CF25E9F437AE1002A59D0" />
    <Allow ID="ID_ALLOW_A_70A" FriendlyName="C:\Program Files\git\usr\bin\factor.exe Hash Page Sha256" Hash="BB9EC55E56B1FD0C467C442CC9E2DAA37A1D7C5329551BEF2B28E538476EF7D3" />
    <Allow ID="ID_ALLOW_A_70B" FriendlyName="C:\Program Files\git\usr\bin\false.exe Hash Sha1" Hash="C195C80384D64F3306E750ECD9F8AD8919A76C91" />
    <Allow ID="ID_ALLOW_A_70C" FriendlyName="C:\Program Files\git\usr\bin\false.exe Hash Sha256" Hash="B90F2C97EA119AD51A91663DA911AC9E95765112A2AD8B833B72466C40CE3169" />
    <Allow ID="ID_ALLOW_A_70D" FriendlyName="C:\Program Files\git\usr\bin\false.exe Hash Page Sha1" Hash="5D527E10461932BEE68D63E4D9E7B41138F863F8" />
    <Allow ID="ID_ALLOW_A_70E" FriendlyName="C:\Program Files\git\usr\bin\false.exe Hash Page Sha256" Hash="A46F92C211D8945DC866C8CC38434CF9A5518DA2B4269C4DE34FB8EFE94A7417" />
    <Allow ID="ID_ALLOW_A_70F" FriendlyName="C:\Program Files\git\usr\bin\fido2-assert.exe Hash Sha1" Hash="0B26E4A5706DF32D08BE129CAA908E538BE45CA5" />
    <Allow ID="ID_ALLOW_A_710" FriendlyName="C:\Program Files\git\usr\bin\fido2-assert.exe Hash Sha256" Hash="7C41EE70680CD20572F5ABFE7C37D754A911E5058FE898C040271C10F5CF5730" />
    <Allow ID="ID_ALLOW_A_711" FriendlyName="C:\Program Files\git\usr\bin\fido2-assert.exe Hash Page Sha1" Hash="E91F0FEAB84006AB140DA86221EC8C339FC32630" />
    <Allow ID="ID_ALLOW_A_712" FriendlyName="C:\Program Files\git\usr\bin\fido2-assert.exe Hash Page Sha256" Hash="36762E916A3911F156DA2768A877FDAA7FF9D9AC2555656D055849345C27EF3E" />
    <Allow ID="ID_ALLOW_A_713" FriendlyName="C:\Program Files\git\usr\bin\fido2-cred.exe Hash Sha1" Hash="03002AC1ED7821BFB9C2A989A3630709518DE4FD" />
    <Allow ID="ID_ALLOW_A_714" FriendlyName="C:\Program Files\git\usr\bin\fido2-cred.exe Hash Sha256" Hash="BF443A4CB95D7C79EF695F3B0504A8FA815283AAC526F2C3F609881F3BFE619B" />
    <Allow ID="ID_ALLOW_A_715" FriendlyName="C:\Program Files\git\usr\bin\fido2-cred.exe Hash Page Sha1" Hash="4451981AB2ABDFB36D4DB6D642736B57A981F94E" />
    <Allow ID="ID_ALLOW_A_716" FriendlyName="C:\Program Files\git\usr\bin\fido2-cred.exe Hash Page Sha256" Hash="F0612D87AE70F3380717EC04489E4DE16689BE257ACC67634A09B0B506188981" />
    <Allow ID="ID_ALLOW_A_717" FriendlyName="C:\Program Files\git\usr\bin\fido2-token.exe Hash Sha1" Hash="9C4665096E8898F623FDF88130AA651356803089" />
    <Allow ID="ID_ALLOW_A_718" FriendlyName="C:\Program Files\git\usr\bin\fido2-token.exe Hash Sha256" Hash="B3896BBE70DD4A02385A2A8011129ECBB1D8A95A41BBB86FDBD38C0BB917C59C" />
    <Allow ID="ID_ALLOW_A_719" FriendlyName="C:\Program Files\git\usr\bin\fido2-token.exe Hash Page Sha1" Hash="510133143EAE7E966C761F4214DCB722C3F074A7" />
    <Allow ID="ID_ALLOW_A_71A" FriendlyName="C:\Program Files\git\usr\bin\fido2-token.exe Hash Page Sha256" Hash="90F6E23A0E9DEEAB080A2C4438E3960D320DF9C588DD77BD9F9B0995F5BA1B2B" />
    <Allow ID="ID_ALLOW_A_71B" FriendlyName="C:\Program Files\git\usr\bin\file.exe Hash Sha1" Hash="A81A541C75E3275FCD233969D3740A2F40E4CB93" />
    <Allow ID="ID_ALLOW_A_71C" FriendlyName="C:\Program Files\git\usr\bin\file.exe Hash Sha256" Hash="F46D0B8311973DF1853EB276C22279C97859EA3C9184F44611CD928401427133" />
    <Allow ID="ID_ALLOW_A_71D" FriendlyName="C:\Program Files\git\usr\bin\file.exe Hash Page Sha1" Hash="A7DDEE48733CEFA159D71CAE5702650FE6EE15B2" />
    <Allow ID="ID_ALLOW_A_71E" FriendlyName="C:\Program Files\git\usr\bin\file.exe Hash Page Sha256" Hash="9EEAB39E5E3291AD8C23D91E0B8732E4837F2EDD6E9A91568EB9665815AE8449" />
    <Allow ID="ID_ALLOW_A_71F" FriendlyName="C:\Program Files\git\usr\bin\find.exe Hash Sha1" Hash="B0E663C4E1FC0DEEA98F61673EEFE478E07A985D" />
    <Allow ID="ID_ALLOW_A_720" FriendlyName="C:\Program Files\git\usr\bin\find.exe Hash Sha256" Hash="B13657D995AC83DAFC5075A2F453DDA6D858E08F626EBA23038BA74BA0830EA9" />
    <Allow ID="ID_ALLOW_A_721" FriendlyName="C:\Program Files\git\usr\bin\find.exe Hash Page Sha1" Hash="B1FB226A8A4E41D7CCF5CD393F58E624C7C872C5" />
    <Allow ID="ID_ALLOW_A_722" FriendlyName="C:\Program Files\git\usr\bin\find.exe Hash Page Sha256" Hash="3694CEE74230113FCA5AE66E2BCFC4DFDE155FDA2104F40F6617D40CD79B8940" />
    <Allow ID="ID_ALLOW_A_723" FriendlyName="C:\Program Files\git\usr\bin\fmt.exe Hash Sha1" Hash="5E9090A45B8932E5EA22F9968B76B3D64CF1CE3E" />
    <Allow ID="ID_ALLOW_A_724" FriendlyName="C:\Program Files\git\usr\bin\fmt.exe Hash Sha256" Hash="0D8AA26BC102380D7968AE2B6203BFF85BA8FC824E97A95C255D53887B50106B" />
    <Allow ID="ID_ALLOW_A_725" FriendlyName="C:\Program Files\git\usr\bin\fmt.exe Hash Page Sha1" Hash="8295FFD8D4A238A21A103FB41FABED70B1CCD256" />
    <Allow ID="ID_ALLOW_A_726" FriendlyName="C:\Program Files\git\usr\bin\fmt.exe Hash Page Sha256" Hash="31E0050332DDCCC61C06A25B759FA7E8F36CC8A1B6499BBFEEF5CF61A3120C1B" />
    <Allow ID="ID_ALLOW_A_727" FriendlyName="C:\Program Files\git\usr\bin\fold.exe Hash Sha1" Hash="E7D413B95D08EAB0E4CA81BA7CE4F539154F5D1A" />
    <Allow ID="ID_ALLOW_A_728" FriendlyName="C:\Program Files\git\usr\bin\fold.exe Hash Sha256" Hash="655B1FF01A6CE8B77FEBD844F78365D67908A59A076E01FE06A50098B326FED4" />
    <Allow ID="ID_ALLOW_A_729" FriendlyName="C:\Program Files\git\usr\bin\fold.exe Hash Page Sha1" Hash="DD241D6E5A8A92136607E52181E987CE07F392AD" />
    <Allow ID="ID_ALLOW_A_72A" FriendlyName="C:\Program Files\git\usr\bin\fold.exe Hash Page Sha256" Hash="87F75A19C6AA87CBC9B541D5C15852EACA267175660B922948656A3C0DA74EDF" />
    <Allow ID="ID_ALLOW_A_72B" FriendlyName="C:\Program Files\git\usr\bin\funzip.exe Hash Sha1" Hash="4D337A3EF3711937F14AF88D0F86C4A1F6B1D923" />
    <Allow ID="ID_ALLOW_A_72C" FriendlyName="C:\Program Files\git\usr\bin\funzip.exe Hash Sha256" Hash="4896B79FA794FC462E22271719CA8873B1E69A1990B07F5E77FDBA09534D9FFE" />
    <Allow ID="ID_ALLOW_A_72D" FriendlyName="C:\Program Files\git\usr\bin\funzip.exe Hash Page Sha1" Hash="85911972A81B59C39D3801EC4F06F4A40343491D" />
    <Allow ID="ID_ALLOW_A_72E" FriendlyName="C:\Program Files\git\usr\bin\funzip.exe Hash Page Sha256" Hash="1B0182C26FBE0C5EE03C766D29B61ADFE94199D319E8DC056037D997A16A2723" />
    <Allow ID="ID_ALLOW_A_72F" FriendlyName="C:\Program Files\git\usr\bin\gapplication.exe Hash Sha1" Hash="856E1051B80925259BD9F8C73AB7C37748AF5D4D" />
    <Allow ID="ID_ALLOW_A_730" FriendlyName="C:\Program Files\git\usr\bin\gapplication.exe Hash Sha256" Hash="2D9571586B3FB1AF3AC6D2EAD32444222D605DC1936CFD292222A2CC8D6B1107" />
    <Allow ID="ID_ALLOW_A_731" FriendlyName="C:\Program Files\git\usr\bin\gapplication.exe Hash Page Sha1" Hash="EB1FDE13ADE7ACEAA059FE7A4B2D0EE7DC33AEC3" />
    <Allow ID="ID_ALLOW_A_732" FriendlyName="C:\Program Files\git\usr\bin\gapplication.exe Hash Page Sha256" Hash="86E831382DE8C3064614E8CE50FEC68B25035ABE716CAD29327F40A1F758544F" />
    <Allow ID="ID_ALLOW_A_733" FriendlyName="C:\Program Files\git\usr\bin\gdbus.exe Hash Sha1" Hash="77EA2355EEBA74100C710B29D328C137B692326E" />
    <Allow ID="ID_ALLOW_A_734" FriendlyName="C:\Program Files\git\usr\bin\gdbus.exe Hash Sha256" Hash="06609F87AF3FF7375337440B09B82618FFDFB36D7DB62E8E8588718B6C9FE647" />
    <Allow ID="ID_ALLOW_A_735" FriendlyName="C:\Program Files\git\usr\bin\gdbus.exe Hash Page Sha1" Hash="28BB1F7D761671E54A3B5DEF5A80612A3C5C1A13" />
    <Allow ID="ID_ALLOW_A_736" FriendlyName="C:\Program Files\git\usr\bin\gdbus.exe Hash Page Sha256" Hash="51507D766B95949AC82C093D1DE9C02967B9E0D1B4D116653A1DED6ABD5954B2" />
    <Allow ID="ID_ALLOW_A_737" FriendlyName="C:\Program Files\git\usr\bin\gencat.exe Hash Sha1" Hash="BDC558D298AD4A2B05AC623FC614AA91719B21D6" />
    <Allow ID="ID_ALLOW_A_738" FriendlyName="C:\Program Files\git\usr\bin\gencat.exe Hash Sha256" Hash="C24F19DFF2BB1CA3ADD2DAF4EB84DE6582CF8DA966F991DFE851DBFDD4686A04" />
    <Allow ID="ID_ALLOW_A_739" FriendlyName="C:\Program Files\git\usr\bin\gencat.exe Hash Page Sha1" Hash="B0AE432B797EB05F1F25D281548A6FDCC1AB253F" />
    <Allow ID="ID_ALLOW_A_73A" FriendlyName="C:\Program Files\git\usr\bin\gencat.exe Hash Page Sha256" Hash="45B97D7919B1FA20E5EEE0F62BB7FCB7FE90C573C45DD44D2B46A0A8ECBE2FEB" />
    <Allow ID="ID_ALLOW_A_73B" FriendlyName="C:\Program Files\git\usr\bin\getconf.exe Hash Sha1" Hash="2019C550BAC7C19401ACF56C6042D8EA9E7DE366" />
    <Allow ID="ID_ALLOW_A_73C" FriendlyName="C:\Program Files\git\usr\bin\getconf.exe Hash Sha256" Hash="2E972EFA68AB634F7E0050E9B1D8012E35C1DC9E7BDE355D59CD163C7FD770B3" />
    <Allow ID="ID_ALLOW_A_73D" FriendlyName="C:\Program Files\git\usr\bin\getconf.exe Hash Page Sha1" Hash="8A2314B1652CC54FCAD406324BAB9ED54E105C77" />
    <Allow ID="ID_ALLOW_A_73E" FriendlyName="C:\Program Files\git\usr\bin\getconf.exe Hash Page Sha256" Hash="CF5F06813891D7AE2F8F66CEF88268552B5BB397ED63630015328604F5CAAED0" />
    <Allow ID="ID_ALLOW_A_73F" FriendlyName="C:\Program Files\git\usr\bin\getfacl.exe Hash Sha1" Hash="2C97D4DA88D37797EA86F741E91C55B98F813231" />
    <Allow ID="ID_ALLOW_A_740" FriendlyName="C:\Program Files\git\usr\bin\getfacl.exe Hash Sha256" Hash="39E5EB8C11CA368777A73A60BC0A36C4D9C0E5F39CA51F268998074A83322739" />
    <Allow ID="ID_ALLOW_A_741" FriendlyName="C:\Program Files\git\usr\bin\getfacl.exe Hash Page Sha1" Hash="15B99D8968E4370E16559550CA9940575C2DF4A4" />
    <Allow ID="ID_ALLOW_A_742" FriendlyName="C:\Program Files\git\usr\bin\getfacl.exe Hash Page Sha256" Hash="EF1BDD69F82E7F77C78C5800E32B4C4B44C57E40704669FD3F47C949ADA6A419" />
    <Allow ID="ID_ALLOW_A_743" FriendlyName="C:\Program Files\git\usr\bin\getopt.exe Hash Sha1" Hash="F0F35DAF6325EF448A5E28520CF201B08CD70309" />
    <Allow ID="ID_ALLOW_A_744" FriendlyName="C:\Program Files\git\usr\bin\getopt.exe Hash Sha256" Hash="55A0342C6AE44D83285B129E91E6878786C9732B6219446F0AFB260A02C30468" />
    <Allow ID="ID_ALLOW_A_745" FriendlyName="C:\Program Files\git\usr\bin\getopt.exe Hash Page Sha1" Hash="D5149DDD3AB4F6BB5B3D3953002BB92D83743EB6" />
    <Allow ID="ID_ALLOW_A_746" FriendlyName="C:\Program Files\git\usr\bin\getopt.exe Hash Page Sha256" Hash="61D5D3C0962050B9C6CC9D5BAF26591D23836D13540828D0316AE32EDF6B0933" />
    <Allow ID="ID_ALLOW_A_747" FriendlyName="C:\Program Files\git\usr\bin\gettext.exe Hash Sha1" Hash="22C8C8B68E80BFD3C78B93236C919CEDC24F6570" />
    <Allow ID="ID_ALLOW_A_748" FriendlyName="C:\Program Files\git\usr\bin\gettext.exe Hash Sha256" Hash="E9C5C6A2BF3E5452E9381F0CDDD6867E57CFFA6D3E501735FC6F3C1BF5D6D703" />
    <Allow ID="ID_ALLOW_A_749" FriendlyName="C:\Program Files\git\usr\bin\gettext.exe Hash Page Sha1" Hash="FD318D17D47164227112AAEFFB6AFDB6D1FB68D6" />
    <Allow ID="ID_ALLOW_A_74A" FriendlyName="C:\Program Files\git\usr\bin\gettext.exe Hash Page Sha256" Hash="4A3317AC0799BFE6124EC867C06377EB1697D86546AD7EE2E5526A368A41904A" />
    <Allow ID="ID_ALLOW_A_74B" FriendlyName="C:\Program Files\git\usr\bin\gio-querymodules.exe Hash Sha1" Hash="6FEE4FE48FF2305294900C2F833DF64ADFA4C18D" />
    <Allow ID="ID_ALLOW_A_74C" FriendlyName="C:\Program Files\git\usr\bin\gio-querymodules.exe Hash Sha256" Hash="CFEBC85C2A16EC6DE995459FCCAB83391908FD602F267DF29DC978431526B53D" />
    <Allow ID="ID_ALLOW_A_74D" FriendlyName="C:\Program Files\git\usr\bin\gio-querymodules.exe Hash Page Sha1" Hash="18D0543DE30D362313B7B415CB6EA04159C47857" />
    <Allow ID="ID_ALLOW_A_74E" FriendlyName="C:\Program Files\git\usr\bin\gio-querymodules.exe Hash Page Sha256" Hash="963E697FA1528FAA5C206014BBB905F2DC9ACE943308961D44D840595EF0E270" />
    <Allow ID="ID_ALLOW_A_74F" FriendlyName="C:\Program Files\git\usr\bin\gkill.exe Hash Sha1" Hash="A65A0E7C47E9E4ED63EE97A3ED12305750046F1B" />
    <Allow ID="ID_ALLOW_A_750" FriendlyName="C:\Program Files\git\usr\bin\gkill.exe Hash Sha256" Hash="17F91120A53B5EC3596ADB46F50E4DAB00102DCA6C75A98524890B37CF72E483" />
    <Allow ID="ID_ALLOW_A_751" FriendlyName="C:\Program Files\git\usr\bin\gkill.exe Hash Page Sha1" Hash="BE16078261B646F293C4EB141B1221F6745D9C1C" />
    <Allow ID="ID_ALLOW_A_752" FriendlyName="C:\Program Files\git\usr\bin\gkill.exe Hash Page Sha256" Hash="6E77E4993B310B340C0D0A4AB99B45A3B6D7706D2FEDE284BF5CC7FDC258124D" />
    <Allow ID="ID_ALLOW_A_753" FriendlyName="C:\Program Files\git\usr\bin\glib-compile-schemas.exe Hash Sha1" Hash="1403B13B4DED4F3DC7E51F04D7EB289155AEB9F2" />
    <Allow ID="ID_ALLOW_A_754" FriendlyName="C:\Program Files\git\usr\bin\glib-compile-schemas.exe Hash Sha256" Hash="10147B11D510BA167952475F5D7E293B2769C9C9EECC2A1067BFFC29C96D0AA0" />
    <Allow ID="ID_ALLOW_A_755" FriendlyName="C:\Program Files\git\usr\bin\glib-compile-schemas.exe Hash Page Sha1" Hash="B2D6497DD676B747FEC732E7F0EA9024E3B75F7D" />
    <Allow ID="ID_ALLOW_A_756" FriendlyName="C:\Program Files\git\usr\bin\glib-compile-schemas.exe Hash Page Sha256" Hash="22ADC9AAB75DFA85626005EF13A7A8D89A24C8D834188E32B2261B9DE19462FC" />
    <Allow ID="ID_ALLOW_A_757" FriendlyName="C:\Program Files\git\usr\bin\gobject-query.exe Hash Sha1" Hash="CCC2089C018087C58BD871349FB495D698E66639" />
    <Allow ID="ID_ALLOW_A_758" FriendlyName="C:\Program Files\git\usr\bin\gobject-query.exe Hash Sha256" Hash="63A115A071322B1562A5359386B98AFE59A0DE71AECC244596B61156EFDAD467" />
    <Allow ID="ID_ALLOW_A_759" FriendlyName="C:\Program Files\git\usr\bin\gobject-query.exe Hash Page Sha1" Hash="87D74272EC1F991A2B48F9129D8D62D4507829A4" />
    <Allow ID="ID_ALLOW_A_75A" FriendlyName="C:\Program Files\git\usr\bin\gobject-query.exe Hash Page Sha256" Hash="1C4FFD4A8E9034665F20D59810E36D1A3EE6FD17EDD108E0C6D8CB3B4BAE7D97" />
    <Allow ID="ID_ALLOW_A_75B" FriendlyName="C:\Program Files\git\usr\bin\gpg-agent.exe Hash Sha1" Hash="513D42FF11AE7B9061431449A98E14D4117B560B" />
    <Allow ID="ID_ALLOW_A_75C" FriendlyName="C:\Program Files\git\usr\bin\gpg-agent.exe Hash Sha256" Hash="BAF24074B84FB8D42B7B5885668424351F083C90A499111CCECBCB942CE5EBDA" />
    <Allow ID="ID_ALLOW_A_75D" FriendlyName="C:\Program Files\git\usr\bin\gpg-agent.exe Hash Page Sha1" Hash="B6201CCA9E9C0D860409BF75068528193F517893" />
    <Allow ID="ID_ALLOW_A_75E" FriendlyName="C:\Program Files\git\usr\bin\gpg-agent.exe Hash Page Sha256" Hash="B906358D3D4D3ABDD3C9674EA2BFEB3ABE0546E8DD9B42E80603ECBF11248AE5" />
    <Allow ID="ID_ALLOW_A_75F" FriendlyName="C:\Program Files\git\usr\bin\gpg-connect-agent.exe Hash Sha1" Hash="6C158E9DD4F8BF535C3AB511C575AEE10D0D2C38" />
    <Allow ID="ID_ALLOW_A_760" FriendlyName="C:\Program Files\git\usr\bin\gpg-connect-agent.exe Hash Sha256" Hash="C6FB9F2D69FA141A7099183764B3D8591DC3E1B51DA82AE118382FC9605091F3" />
    <Allow ID="ID_ALLOW_A_761" FriendlyName="C:\Program Files\git\usr\bin\gpg-connect-agent.exe Hash Page Sha1" Hash="1BBBF4B5BE82926B57AC5BF97CE985FDAEA85142" />
    <Allow ID="ID_ALLOW_A_762" FriendlyName="C:\Program Files\git\usr\bin\gpg-connect-agent.exe Hash Page Sha256" Hash="840139A95DF09351EBD983B7E32610067CA3066A50F838895D0AB16DEF47759A" />
    <Allow ID="ID_ALLOW_A_763" FriendlyName="C:\Program Files\git\usr\bin\gpg-error.exe Hash Sha1" Hash="D555700193B1DC49040AB53EF36CB84E140C9E18" />
    <Allow ID="ID_ALLOW_A_764" FriendlyName="C:\Program Files\git\usr\bin\gpg-error.exe Hash Sha256" Hash="72CC4098C7079124C7CD8A23043A1C2D95683DA3365773478D553A5538C972D1" />
    <Allow ID="ID_ALLOW_A_765" FriendlyName="C:\Program Files\git\usr\bin\gpg-error.exe Hash Page Sha1" Hash="3E9602DBE543EB0AD2CCC23185A227810D3E997C" />
    <Allow ID="ID_ALLOW_A_766" FriendlyName="C:\Program Files\git\usr\bin\gpg-error.exe Hash Page Sha256" Hash="056E794683727FFDF903486E974C59BB7AF89CCF218AFAE9837019B1F3945B19" />
    <Allow ID="ID_ALLOW_A_767" FriendlyName="C:\Program Files\git\usr\bin\gpg-wks-server.exe Hash Sha1" Hash="7367D3CF42A9652B7B2F048C1324D7954B304EDF" />
    <Allow ID="ID_ALLOW_A_768" FriendlyName="C:\Program Files\git\usr\bin\gpg-wks-server.exe Hash Sha256" Hash="8992BB21AFC1DA5EBE88C597F38DE83EC03A33F485F01360D98A3B80692B344C" />
    <Allow ID="ID_ALLOW_A_769" FriendlyName="C:\Program Files\git\usr\bin\gpg-wks-server.exe Hash Page Sha1" Hash="D2B0CE0AE97BDCB66A9893F4CB381E2B67E45A9C" />
    <Allow ID="ID_ALLOW_A_76A" FriendlyName="C:\Program Files\git\usr\bin\gpg-wks-server.exe Hash Page Sha256" Hash="3F2DF4A8220A1A3DD481DAA414C5FD26E16C3C31F35BF3D2F1195C181974991E" />
    <Allow ID="ID_ALLOW_A_76B" FriendlyName="C:\Program Files\git\usr\bin\gpg.exe Hash Sha1" Hash="3CF84766F5A8900F8D9EA358E0821A530DD9BB6E" />
    <Allow ID="ID_ALLOW_A_76C" FriendlyName="C:\Program Files\git\usr\bin\gpg.exe Hash Sha256" Hash="C408F48E947B82D4F9BCF30AECAFBBFDEBA80FE90633AC2BCE1D3E6A71B34680" />
    <Allow ID="ID_ALLOW_A_76D" FriendlyName="C:\Program Files\git\usr\bin\gpg.exe Hash Page Sha1" Hash="A2BCC604586A023E677583EEE8CAEE8CA35B9AEE" />
    <Allow ID="ID_ALLOW_A_76E" FriendlyName="C:\Program Files\git\usr\bin\gpg.exe Hash Page Sha256" Hash="E2DB01465C2E73CE1FFDDADAF14CFAD8569E46FE119003C94735467987815FC3" />
    <Allow ID="ID_ALLOW_A_76F" FriendlyName="C:\Program Files\git\usr\bin\gpgconf.exe Hash Sha1" Hash="7335B927E2F4B3335121D0B202E5120107AF8974" />
    <Allow ID="ID_ALLOW_A_770" FriendlyName="C:\Program Files\git\usr\bin\gpgconf.exe Hash Sha256" Hash="E9E47BC3186CE487F6CE653EAB73E65D9B064F6A074C28408E2783F34C8E6C47" />
    <Allow ID="ID_ALLOW_A_771" FriendlyName="C:\Program Files\git\usr\bin\gpgconf.exe Hash Page Sha1" Hash="31A833DD1E05B4F964D8A2EACD9C686DA9D8BC9A" />
    <Allow ID="ID_ALLOW_A_772" FriendlyName="C:\Program Files\git\usr\bin\gpgconf.exe Hash Page Sha256" Hash="52F7F265BA3138B3A0B55DF6233D7B46C9F66E9183D56FF2508F59425FA86A72" />
    <Allow ID="ID_ALLOW_A_773" FriendlyName="C:\Program Files\git\usr\bin\gpgparsemail.exe Hash Sha1" Hash="D7B4843F801DA4CF54F3EA38A9881A7774B86ADD" />
    <Allow ID="ID_ALLOW_A_774" FriendlyName="C:\Program Files\git\usr\bin\gpgparsemail.exe Hash Sha256" Hash="A4C66A07E018B28B202AB9B3E837EF231B4A3DBDA60C950BBEBA3E7F922EF5F8" />
    <Allow ID="ID_ALLOW_A_775" FriendlyName="C:\Program Files\git\usr\bin\gpgparsemail.exe Hash Page Sha1" Hash="17BC6AE3EDD836BC7DEF89C32855A075C831E7A3" />
    <Allow ID="ID_ALLOW_A_776" FriendlyName="C:\Program Files\git\usr\bin\gpgparsemail.exe Hash Page Sha256" Hash="3CF8900D3D092ABF1546C03CF8EDEE0AE9BC0427BA9674164402020A3DBD54CA" />
    <Allow ID="ID_ALLOW_A_777" FriendlyName="C:\Program Files\git\usr\bin\gpgscm.exe Hash Sha1" Hash="28FD3B40186C0DCF2B81409CFCB3FE85BE7A6066" />
    <Allow ID="ID_ALLOW_A_778" FriendlyName="C:\Program Files\git\usr\bin\gpgscm.exe Hash Sha256" Hash="96967FA0850FD5CFDFD039234F76998C3F2521FC8FE34D817680A2386763D90A" />
    <Allow ID="ID_ALLOW_A_779" FriendlyName="C:\Program Files\git\usr\bin\gpgscm.exe Hash Page Sha1" Hash="E7A1554FC975F045FD0AF946D3E8BF17431CB3D0" />
    <Allow ID="ID_ALLOW_A_77A" FriendlyName="C:\Program Files\git\usr\bin\gpgscm.exe Hash Page Sha256" Hash="1DFA4CB0CC3FA988B615889B98C6AF671AA117F0A3D919411491EF0AF9E400A1" />
    <Allow ID="ID_ALLOW_A_77B" FriendlyName="C:\Program Files\git\usr\bin\gpgsm.exe Hash Sha1" Hash="931CB2714E6A5361359661A775A9524E325820A4" />
    <Allow ID="ID_ALLOW_A_77C" FriendlyName="C:\Program Files\git\usr\bin\gpgsm.exe Hash Sha256" Hash="6E1E1A1A128896F3AD3BD0BDC6A655518DFC8202A98A11CC5D5AF626C4196100" />
    <Allow ID="ID_ALLOW_A_77D" FriendlyName="C:\Program Files\git\usr\bin\gpgsm.exe Hash Page Sha1" Hash="87F456C171D52C3ACB7FDD6ACF0A9A0F3A12CDC2" />
    <Allow ID="ID_ALLOW_A_77E" FriendlyName="C:\Program Files\git\usr\bin\gpgsm.exe Hash Page Sha256" Hash="11E080D2152AFF7987B39679B894AE4F69C9ED0FCD316DC6327E1C60516A43B9" />
    <Allow ID="ID_ALLOW_A_77F" FriendlyName="C:\Program Files\git\usr\bin\gpgsplit.exe Hash Sha1" Hash="F210C3C397D633B81F3DFADB37722CA86F997425" />
    <Allow ID="ID_ALLOW_A_780" FriendlyName="C:\Program Files\git\usr\bin\gpgsplit.exe Hash Sha256" Hash="A46B222B109383B38CC98072F6C633382553A817DF9384F238BF251122778B88" />
    <Allow ID="ID_ALLOW_A_781" FriendlyName="C:\Program Files\git\usr\bin\gpgsplit.exe Hash Page Sha1" Hash="24902344B8E6A1C7DBEE9526FE65682718241DE2" />
    <Allow ID="ID_ALLOW_A_782" FriendlyName="C:\Program Files\git\usr\bin\gpgsplit.exe Hash Page Sha256" Hash="2154B6ED54382E724791789D4E234A4B4E06CAC40A962560584E52BED5B7C33A" />
    <Allow ID="ID_ALLOW_A_783" FriendlyName="C:\Program Files\git\usr\bin\gpgtar.exe Hash Sha1" Hash="DB7BED21423FC968CAF186CC6236C357DBE462F7" />
    <Allow ID="ID_ALLOW_A_784" FriendlyName="C:\Program Files\git\usr\bin\gpgtar.exe Hash Sha256" Hash="C58A97249B7D09CBFDAD159D980EAA1D88EC91EDD204B11E609C755BF8E3E3CE" />
    <Allow ID="ID_ALLOW_A_785" FriendlyName="C:\Program Files\git\usr\bin\gpgtar.exe Hash Page Sha1" Hash="C0D0DA64999184B82146ECA0CA7440B8648507E5" />
    <Allow ID="ID_ALLOW_A_786" FriendlyName="C:\Program Files\git\usr\bin\gpgtar.exe Hash Page Sha256" Hash="79332B17BD9CEB1C7A444A4F1147123A576EC60DE4BF51D81FDE954F5E511B2B" />
    <Allow ID="ID_ALLOW_A_787" FriendlyName="C:\Program Files\git\usr\bin\gpgv.exe Hash Sha1" Hash="EA9FD1A90822E4FF1358688CDFA99DDE3CAB0A76" />
    <Allow ID="ID_ALLOW_A_788" FriendlyName="C:\Program Files\git\usr\bin\gpgv.exe Hash Sha256" Hash="4D11843585EA9DA56525F71A9E2D7A9DCB7916BE6552144ADAEB0E47317DC568" />
    <Allow ID="ID_ALLOW_A_789" FriendlyName="C:\Program Files\git\usr\bin\gpgv.exe Hash Page Sha1" Hash="29B54F940212C66FE332EF22288CF25172DC53DE" />
    <Allow ID="ID_ALLOW_A_78A" FriendlyName="C:\Program Files\git\usr\bin\gpgv.exe Hash Page Sha256" Hash="2EB9B964BBBD55D65261C45A330A6501F7BAC5645CA00A7737B4723A2886F5B8" />
    <Allow ID="ID_ALLOW_A_78B" FriendlyName="C:\Program Files\git\usr\bin\grep.exe Hash Sha1" Hash="E3125495C38511C3F207A6E2B3B88556C272E1C2" />
    <Allow ID="ID_ALLOW_A_78C" FriendlyName="C:\Program Files\git\usr\bin\grep.exe Hash Sha256" Hash="12BF4D44D39270D318EE0AECF7FCF690BEA8F1B48DCFB394BF8A03AB121F6B4D" />
    <Allow ID="ID_ALLOW_A_78D" FriendlyName="C:\Program Files\git\usr\bin\grep.exe Hash Page Sha1" Hash="A876BD78FDF6D15FF1A1838EE92FFA13F402F82A" />
    <Allow ID="ID_ALLOW_A_78E" FriendlyName="C:\Program Files\git\usr\bin\grep.exe Hash Page Sha256" Hash="C192A43651A1DF023FDFD5463EB4339C3B52ADF5D277E84D67F59D3A07C15E28" />
    <Allow ID="ID_ALLOW_A_78F" FriendlyName="C:\Program Files\git\usr\bin\groups.exe Hash Sha1" Hash="1FD59CA07B94E2709A953D03BB97E984513A2AC1" />
    <Allow ID="ID_ALLOW_A_790" FriendlyName="C:\Program Files\git\usr\bin\groups.exe Hash Sha256" Hash="4A18AD19A1F490578D7AD4540D80BB039A94D9473149056D0DDA4114C022CB5C" />
    <Allow ID="ID_ALLOW_A_791" FriendlyName="C:\Program Files\git\usr\bin\groups.exe Hash Page Sha1" Hash="0E968A6D6430FD907B3A376F65A439F7892C0F82" />
    <Allow ID="ID_ALLOW_A_792" FriendlyName="C:\Program Files\git\usr\bin\groups.exe Hash Page Sha256" Hash="7278165D016DBE3EA2E7293CB8CE8E0FCBD229AC175032C16EA85B4A1BAF28E2" />
    <Allow ID="ID_ALLOW_A_793" FriendlyName="C:\Program Files\git\usr\bin\gsettings.exe Hash Sha1" Hash="84004C3917589CC964C4B1A130F12A8E569F5F74" />
    <Allow ID="ID_ALLOW_A_794" FriendlyName="C:\Program Files\git\usr\bin\gsettings.exe Hash Sha256" Hash="862030524B022BF42E566B96316BA4162E1D581421C9885C81ECBACCE917E2EC" />
    <Allow ID="ID_ALLOW_A_795" FriendlyName="C:\Program Files\git\usr\bin\gsettings.exe Hash Page Sha1" Hash="450FA54A4CA121741D66C5C783D1BFD34B804241" />
    <Allow ID="ID_ALLOW_A_796" FriendlyName="C:\Program Files\git\usr\bin\gsettings.exe Hash Page Sha256" Hash="98B94123D6D563BC8692EF24F531403B9D8B83129147A3AE6BFF25FB03F7F8ED" />
    <Allow ID="ID_ALLOW_A_797" FriendlyName="C:\Program Files\git\usr\bin\gzip.exe Hash Sha1" Hash="8529486DD2D99B37C4AB380FF646ECC7C5D7D736" />
    <Allow ID="ID_ALLOW_A_798" FriendlyName="C:\Program Files\git\usr\bin\gzip.exe Hash Sha256" Hash="9F1DE45055C6290B3EB80ED464ECBBF244F9736EA6528F0AAB511DF9B3DBE8B5" />
    <Allow ID="ID_ALLOW_A_799" FriendlyName="C:\Program Files\git\usr\bin\gzip.exe Hash Page Sha1" Hash="D83D00AB84A724C7DE96B97ABA5284067E542D67" />
    <Allow ID="ID_ALLOW_A_79A" FriendlyName="C:\Program Files\git\usr\bin\gzip.exe Hash Page Sha256" Hash="58CE2FF22B1F868DD41629D221B3B1C1509A315F913FE8A4AAB784BE7986BB4E" />
    <Allow ID="ID_ALLOW_A_79B" FriendlyName="C:\Program Files\git\usr\bin\head.exe Hash Sha1" Hash="34ABD4E5A620D9E3BEB761F2FD3E186CF076D862" />
    <Allow ID="ID_ALLOW_A_79C" FriendlyName="C:\Program Files\git\usr\bin\head.exe Hash Sha256" Hash="EE707F161DE41A867C9068119FA0AEB5D77E18A638D6DF636232DF8F00D5EE02" />
    <Allow ID="ID_ALLOW_A_79D" FriendlyName="C:\Program Files\git\usr\bin\head.exe Hash Page Sha1" Hash="2A3DDC2CE64C9A76A4203BAC233621613D1331CE" />
    <Allow ID="ID_ALLOW_A_79E" FriendlyName="C:\Program Files\git\usr\bin\head.exe Hash Page Sha256" Hash="F009121E71287918BCAFD9108C264C204308C93E25E956D707221767BB2F8D36" />
    <Allow ID="ID_ALLOW_A_79F" FriendlyName="C:\Program Files\git\usr\bin\hmac256.exe Hash Sha1" Hash="52C5FFA197F151BB9D18062EDB8E447C1CBFB789" />
    <Allow ID="ID_ALLOW_A_7A0" FriendlyName="C:\Program Files\git\usr\bin\hmac256.exe Hash Sha256" Hash="2EA34B92542BCFBBA6D619BACDB3F79C661083D9E00D0C488D1C031DD2ED4234" />
    <Allow ID="ID_ALLOW_A_7A1" FriendlyName="C:\Program Files\git\usr\bin\hmac256.exe Hash Page Sha1" Hash="BB6FEB771DE3DC90E1EBB6D4A8413B997F98EFC7" />
    <Allow ID="ID_ALLOW_A_7A2" FriendlyName="C:\Program Files\git\usr\bin\hmac256.exe Hash Page Sha256" Hash="AA04D1A5163A632793A64834F43BB6586A0B332C5296129FD7B05D331421728F" />
    <Allow ID="ID_ALLOW_A_7A3" FriendlyName="C:\Program Files\git\usr\bin\hostid.exe Hash Sha1" Hash="B24B99C00EF4A19A457AD8B94D6592E60D59AF4C" />
    <Allow ID="ID_ALLOW_A_7A4" FriendlyName="C:\Program Files\git\usr\bin\hostid.exe Hash Sha256" Hash="3355B7BA4D721A26CD08CFF53EC1C386BE102829A8BC5D93CEC2027F8E4DD01C" />
    <Allow ID="ID_ALLOW_A_7A5" FriendlyName="C:\Program Files\git\usr\bin\hostid.exe Hash Page Sha1" Hash="026FBE3C4D79E1D8117EE264A2EFF3B856FB72FD" />
    <Allow ID="ID_ALLOW_A_7A6" FriendlyName="C:\Program Files\git\usr\bin\hostid.exe Hash Page Sha256" Hash="A32DFF8BF970E65DAFBD1E1925BA94E6849D1AD0BFCF737E3F04010D4E82258B" />
    <Allow ID="ID_ALLOW_A_7A7" FriendlyName="C:\Program Files\git\usr\bin\hostname.exe Hash Sha1" Hash="24B8EEC29EB4EAF7F6F3FD44EA91A0337D866E5F" />
    <Allow ID="ID_ALLOW_A_7A8" FriendlyName="C:\Program Files\git\usr\bin\hostname.exe Hash Sha256" Hash="0E29C2E9E5CF5621C541B1DC574CABB7346129F5E0CA569492ED07C79184107C" />
    <Allow ID="ID_ALLOW_A_7A9" FriendlyName="C:\Program Files\git\usr\bin\hostname.exe Hash Page Sha1" Hash="625A26A96998710BD09BD97E02B89C8A054B1205" />
    <Allow ID="ID_ALLOW_A_7AA" FriendlyName="C:\Program Files\git\usr\bin\hostname.exe Hash Page Sha256" Hash="616F91D80CACAB05D6DB4A1B4AB5633377DF3D3F03D2A217EDFF109732BA9995" />
    <Allow ID="ID_ALLOW_A_7AB" FriendlyName="C:\Program Files\git\usr\bin\iconv.exe Hash Sha1" Hash="0EF5ACB002E8830F37843E913E73E6879B157BCB" />
    <Allow ID="ID_ALLOW_A_7AC" FriendlyName="C:\Program Files\git\usr\bin\iconv.exe Hash Sha256" Hash="7BCFCF9037C6304E35506E888F8A16041EEE61D2F41218FDA512267977BBCCC9" />
    <Allow ID="ID_ALLOW_A_7AD" FriendlyName="C:\Program Files\git\usr\bin\iconv.exe Hash Page Sha1" Hash="E634793A7D9F5BF23E32CF40DD8D1170E09A2C6E" />
    <Allow ID="ID_ALLOW_A_7AE" FriendlyName="C:\Program Files\git\usr\bin\iconv.exe Hash Page Sha256" Hash="FF9BCDE59427817B932C6AD4EC382D96F13C7F97ECBEF2AA7D6E7BA67C1AFEB6" />
    <Allow ID="ID_ALLOW_A_7AF" FriendlyName="C:\Program Files\git\usr\bin\id.exe Hash Sha1" Hash="917DC0737CC32175F317FC69DBCF8B3EAE7B3FAD" />
    <Allow ID="ID_ALLOW_A_7B0" FriendlyName="C:\Program Files\git\usr\bin\id.exe Hash Sha256" Hash="9CC4AD7603FA414AACA73339C3345F7C06AF62EE076685C6490B3693F71D7BC0" />
    <Allow ID="ID_ALLOW_A_7B1" FriendlyName="C:\Program Files\git\usr\bin\id.exe Hash Page Sha1" Hash="4B629165A36CC4C5B89B1E02A9158D7E3C00B0C1" />
    <Allow ID="ID_ALLOW_A_7B2" FriendlyName="C:\Program Files\git\usr\bin\id.exe Hash Page Sha256" Hash="D1C55CA3128A779C8BF1072B7C8388389B6239940A69EE5EFE6571477D4D1EAA" />
    <Allow ID="ID_ALLOW_A_7B3" FriendlyName="C:\Program Files\git\usr\bin\infocmp.exe Hash Sha1" Hash="E502108E60AF0B867AC5F9279FB1D55CDDF4BCAF" />
    <Allow ID="ID_ALLOW_A_7B4" FriendlyName="C:\Program Files\git\usr\bin\infocmp.exe Hash Sha256" Hash="C3B8BA24B88AE6FED9C1F0E289AF4548DFA54EB8D2CE2584C33D4B015DFB90D9" />
    <Allow ID="ID_ALLOW_A_7B5" FriendlyName="C:\Program Files\git\usr\bin\infocmp.exe Hash Page Sha1" Hash="EC6C12C584D726125659CEF0252093F345851751" />
    <Allow ID="ID_ALLOW_A_7B6" FriendlyName="C:\Program Files\git\usr\bin\infocmp.exe Hash Page Sha256" Hash="08E030B328953AB4779AA9363192C79A7A7C519219D3B2A4C12FFE3088953208" />
    <Allow ID="ID_ALLOW_A_7B7" FriendlyName="C:\Program Files\git\usr\bin\install.exe Hash Sha1" Hash="05F230429B1D89577C26E51E8CC4A8C3E6472847" />
    <Allow ID="ID_ALLOW_A_7B8" FriendlyName="C:\Program Files\git\usr\bin\install.exe Hash Sha256" Hash="90914514CC6B1B5CF5F77B50982D72E2556B9ED9CB5291D153C326BF5038B11B" />
    <Allow ID="ID_ALLOW_A_7B9" FriendlyName="C:\Program Files\git\usr\bin\install.exe Hash Page Sha1" Hash="71A803628769F4206B466867CBCAA430A1C81701" />
    <Allow ID="ID_ALLOW_A_7BA" FriendlyName="C:\Program Files\git\usr\bin\install.exe Hash Page Sha256" Hash="18DBF8D500FFB04F7BF47A4E6C68EF1797D6F7846DACD9966167A8A8755B992D" />
    <Allow ID="ID_ALLOW_A_7BB" FriendlyName="C:\Program Files\git\usr\bin\join.exe Hash Sha1" Hash="85676EAFC0A98841D2C4CEDD52B4B71A668BFFD9" />
    <Allow ID="ID_ALLOW_A_7BC" FriendlyName="C:\Program Files\git\usr\bin\join.exe Hash Sha256" Hash="57F29140559DDD038BF2161F2168AE9EAF226AF5E9133CEC5D2C264790205D70" />
    <Allow ID="ID_ALLOW_A_7BD" FriendlyName="C:\Program Files\git\usr\bin\join.exe Hash Page Sha1" Hash="8BC79684C89A8A1027FB21105567A4571331B7AD" />
    <Allow ID="ID_ALLOW_A_7BE" FriendlyName="C:\Program Files\git\usr\bin\join.exe Hash Page Sha256" Hash="0163B85B14EDFA5B0EBBEAC1782125575F8A50D94211E2B575F084BD951C5EF1" />
    <Allow ID="ID_ALLOW_A_7BF" FriendlyName="C:\Program Files\git\usr\bin\kbxutil.exe Hash Sha1" Hash="22DCAAF01541D4050BB8659FAA1D786FD5CFB99B" />
    <Allow ID="ID_ALLOW_A_7C0" FriendlyName="C:\Program Files\git\usr\bin\kbxutil.exe Hash Sha256" Hash="F954A95FDCA93E66F3850F2A57B834B3858F65A91A7F11D672735C623BE27A12" />
    <Allow ID="ID_ALLOW_A_7C1" FriendlyName="C:\Program Files\git\usr\bin\kbxutil.exe Hash Page Sha1" Hash="489961378C8907EBCC3F7452D1E3401D62DC4D49" />
    <Allow ID="ID_ALLOW_A_7C2" FriendlyName="C:\Program Files\git\usr\bin\kbxutil.exe Hash Page Sha256" Hash="07656BCAC461B3991C59A0345F23C448BFA958E8171E5A41FE61BDB37951C74E" />
    <Allow ID="ID_ALLOW_A_7C3" FriendlyName="C:\Program Files\git\usr\bin\kill.exe Hash Sha1" Hash="F2F509669B633F8C9067E14EB79DB7B793EB868E" />
    <Allow ID="ID_ALLOW_A_7C4" FriendlyName="C:\Program Files\git\usr\bin\kill.exe Hash Sha256" Hash="617ADDA4D02A0052A59A9FF5E833D0E418D5CDAF3BD923219AF286BE75185B59" />
    <Allow ID="ID_ALLOW_A_7C5" FriendlyName="C:\Program Files\git\usr\bin\kill.exe Hash Page Sha1" Hash="4A207B14D32B6B9EAF425802A600FD82E0286CC7" />
    <Allow ID="ID_ALLOW_A_7C6" FriendlyName="C:\Program Files\git\usr\bin\kill.exe Hash Page Sha256" Hash="A968F2FE845B457702EF81FC956332BC3AEDDEAF3EE0E12749C1323D2AE39C12" />
    <Allow ID="ID_ALLOW_A_7C7" FriendlyName="C:\Program Files\git\usr\bin\ldd.exe Hash Sha1" Hash="90A94C66421B6C4282B162FA7845D8E0BA56D0D0" />
    <Allow ID="ID_ALLOW_A_7C8" FriendlyName="C:\Program Files\git\usr\bin\ldd.exe Hash Sha256" Hash="ED3627C171B50033F105DF18BA44C25C193B80EEB425C08581136BD2E2C3D6BB" />
    <Allow ID="ID_ALLOW_A_7C9" FriendlyName="C:\Program Files\git\usr\bin\ldd.exe Hash Page Sha1" Hash="8D80A53857D0DF836587EE4B43E547DA8F136D38" />
    <Allow ID="ID_ALLOW_A_7CA" FriendlyName="C:\Program Files\git\usr\bin\ldd.exe Hash Page Sha256" Hash="A90F8CF8A57FEAA9A8B41B038C8C029501427B53B70038788380ED2FD055C861" />
    <Allow ID="ID_ALLOW_A_7CB" FriendlyName="C:\Program Files\git\usr\bin\ldh.exe Hash Sha1" Hash="971286C0BC169C50C4B0F8075341A0928B0261A0" />
    <Allow ID="ID_ALLOW_A_7CC" FriendlyName="C:\Program Files\git\usr\bin\ldh.exe Hash Sha256" Hash="5A130113B2AB3E9437AF87DAA1F578766277ACF677C46876A0CCBF1C691B4215" />
    <Allow ID="ID_ALLOW_A_7CD" FriendlyName="C:\Program Files\git\usr\bin\ldh.exe Hash Page Sha1" Hash="F92C23F64C7B3A56FC74E0BF94F4506C4476A9ED" />
    <Allow ID="ID_ALLOW_A_7CE" FriendlyName="C:\Program Files\git\usr\bin\ldh.exe Hash Page Sha256" Hash="B8ED1798AA380CA6B3FCC2E8F2B0C2C69750BE8A2FFF6C119906C9A2442945F8" />
    <Allow ID="ID_ALLOW_A_7CF" FriendlyName="C:\Program Files\git\usr\bin\less.exe Hash Sha1" Hash="F76F659C874EBAC396DE1B2AFDC895F1F5C08C23" />
    <Allow ID="ID_ALLOW_A_7D0" FriendlyName="C:\Program Files\git\usr\bin\less.exe Hash Sha256" Hash="5A0D68FC5E6FC05ACA6D83777BFDEEBC639BCDE89173F7B609495AF013B27E44" />
    <Allow ID="ID_ALLOW_A_7D1" FriendlyName="C:\Program Files\git\usr\bin\less.exe Hash Page Sha1" Hash="0957B94041644D03D4196C18EE54B1FFACFFE54D" />
    <Allow ID="ID_ALLOW_A_7D2" FriendlyName="C:\Program Files\git\usr\bin\less.exe Hash Page Sha256" Hash="99D258085E0B6D93ACC0EDB74F557EB9C5F9D5A4449E117A0915E9D0A2C14D7C" />
    <Allow ID="ID_ALLOW_A_7D3" FriendlyName="C:\Program Files\git\usr\bin\lessecho.exe Hash Sha1" Hash="AC66F3D5C64AF739DDB0CE5B4D82EA2B541FD0D6" />
    <Allow ID="ID_ALLOW_A_7D4" FriendlyName="C:\Program Files\git\usr\bin\lessecho.exe Hash Sha256" Hash="9B9CAFB0797BA80D11071FB9EB17BBBE4585D16F2C3E4EF7B3CF51C445CC1B9D" />
    <Allow ID="ID_ALLOW_A_7D5" FriendlyName="C:\Program Files\git\usr\bin\lessecho.exe Hash Page Sha1" Hash="F6A93754FA7A7D9FBD48E7ABDC7DEC5193D17F0A" />
    <Allow ID="ID_ALLOW_A_7D6" FriendlyName="C:\Program Files\git\usr\bin\lessecho.exe Hash Page Sha256" Hash="9D3598BC4B4DDD4D0C6753C7D1634D100D8382853186897CBD39DC1C71B33EBF" />
    <Allow ID="ID_ALLOW_A_7D7" FriendlyName="C:\Program Files\git\usr\bin\lesskey.exe Hash Sha1" Hash="88A5727D37BF2AA1EC4CD572C7C8425358A8B956" />
    <Allow ID="ID_ALLOW_A_7D8" FriendlyName="C:\Program Files\git\usr\bin\lesskey.exe Hash Sha256" Hash="18E4EC12218F19A5ABB4E1E18A5A6C3DCFD0E586D07A2803903673FF7C20EEA1" />
    <Allow ID="ID_ALLOW_A_7D9" FriendlyName="C:\Program Files\git\usr\bin\lesskey.exe Hash Page Sha1" Hash="052A36FCE060863AD997444804C6C6C66FAB849B" />
    <Allow ID="ID_ALLOW_A_7DA" FriendlyName="C:\Program Files\git\usr\bin\lesskey.exe Hash Page Sha256" Hash="E1C12CD2B45427B1D2D59BABABC8E48A39657C84BD1D5B5459ECFF14A1365268" />
    <Allow ID="ID_ALLOW_A_7DB" FriendlyName="C:\Program Files\git\usr\bin\libtcl8.6.dll Hash Sha1" Hash="28E95672C77EE2DA8688FF560034520B5A5F80BD" />
    <Allow ID="ID_ALLOW_A_7DC" FriendlyName="C:\Program Files\git\usr\bin\libtcl8.6.dll Hash Sha256" Hash="7AB7F154C171B36B31543388A40E2A53FFD9A9E363891755AC077E45D84DB1C0" />
    <Allow ID="ID_ALLOW_A_7DD" FriendlyName="C:\Program Files\git\usr\bin\libtcl8.6.dll Hash Page Sha1" Hash="9AFA7D72F24446DF5F08BFB8C48EE4F781AAB5B3" />
    <Allow ID="ID_ALLOW_A_7DE" FriendlyName="C:\Program Files\git\usr\bin\libtcl8.6.dll Hash Page Sha256" Hash="24C61778B87206A0B6464E00FDF1C3F3ED4340955315151A63C04BE835D9EAD4" />
    <Allow ID="ID_ALLOW_A_7DF" FriendlyName="C:\Program Files\git\usr\bin\link.exe Hash Sha1" Hash="C7514B588525B671F5B02BA682688625297A2BAF" />
    <Allow ID="ID_ALLOW_A_7E0" FriendlyName="C:\Program Files\git\usr\bin\link.exe Hash Sha256" Hash="18AA36FB3C60BA463B3AD67CF56D497EEDFA3D3548CDB3EF661DD33F52686CA4" />
    <Allow ID="ID_ALLOW_A_7E1" FriendlyName="C:\Program Files\git\usr\bin\link.exe Hash Page Sha1" Hash="AB94F39D2D36C4384AF55B44551EFC0E5468CAD5" />
    <Allow ID="ID_ALLOW_A_7E2" FriendlyName="C:\Program Files\git\usr\bin\link.exe Hash Page Sha256" Hash="359760A9551734C6CC0CEA49E7EFAB9F64E17F486204AAFCA00AF08DB4C16101" />
    <Allow ID="ID_ALLOW_A_7E3" FriendlyName="C:\Program Files\git\usr\bin\ln.exe Hash Sha1" Hash="A7A1D50A5F7BE7062125F983110D61E0E13ACB66" />
    <Allow ID="ID_ALLOW_A_7E4" FriendlyName="C:\Program Files\git\usr\bin\ln.exe Hash Sha256" Hash="5D87F4011763999D22481BAB36D0DA9A4E7A3BF2B51061840DC96105AD55C72A" />
    <Allow ID="ID_ALLOW_A_7E5" FriendlyName="C:\Program Files\git\usr\bin\ln.exe Hash Page Sha1" Hash="0621ECBE5E6C58CD467D6F02464DD9AB5879B6AF" />
    <Allow ID="ID_ALLOW_A_7E6" FriendlyName="C:\Program Files\git\usr\bin\ln.exe Hash Page Sha256" Hash="506D50E32D1C1DC94036607494E0275EB5592FD699C0622445419DC3E3B24598" />
    <Allow ID="ID_ALLOW_A_7E7" FriendlyName="C:\Program Files\git\usr\bin\locale.exe Hash Sha1" Hash="8E0AA54520F930F6735DC77AABC7AC05250E7E6B" />
    <Allow ID="ID_ALLOW_A_7E8" FriendlyName="C:\Program Files\git\usr\bin\locale.exe Hash Sha256" Hash="82013BDEB8A59D2A2EFDD249ADC6767393657AD37E0D8A07C1571B311DFC3535" />
    <Allow ID="ID_ALLOW_A_7E9" FriendlyName="C:\Program Files\git\usr\bin\locale.exe Hash Page Sha1" Hash="826C90BA1016CDBBE6C34D89120819A3014E10FB" />
    <Allow ID="ID_ALLOW_A_7EA" FriendlyName="C:\Program Files\git\usr\bin\locale.exe Hash Page Sha256" Hash="0F7E87777A9626F17A24FEC3DEC46BA36778FF2898538F9C82A791C705599330" />
    <Allow ID="ID_ALLOW_A_7EB" FriendlyName="C:\Program Files\git\usr\bin\locate.exe Hash Sha1" Hash="F4FC0FD48915E863938CA2AF921A10ED5D01731C" />
    <Allow ID="ID_ALLOW_A_7EC" FriendlyName="C:\Program Files\git\usr\bin\locate.exe Hash Sha256" Hash="1DF39B2BE35BB771BCF5D6AD86D25C5F2A2A078041065028577F7B55DB3EA4D9" />
    <Allow ID="ID_ALLOW_A_7ED" FriendlyName="C:\Program Files\git\usr\bin\locate.exe Hash Page Sha1" Hash="713B19BA758E5FE7E913CF0DA321723AAD35C4EB" />
    <Allow ID="ID_ALLOW_A_7EE" FriendlyName="C:\Program Files\git\usr\bin\locate.exe Hash Page Sha256" Hash="036B848F4496CE6CAA1B3F4D1AD75B265060B9EBF1C40107C26F84AFF068CDF8" />
    <Allow ID="ID_ALLOW_A_7EF" FriendlyName="C:\Program Files\git\usr\bin\logname.exe Hash Sha1" Hash="3B9707D71B6F6960BB93746134826F40082FF536" />
    <Allow ID="ID_ALLOW_A_7F0" FriendlyName="C:\Program Files\git\usr\bin\logname.exe Hash Sha256" Hash="2700038427EFBCE4AE0759D6139420CFE8481CD23F1EEC222CF3EAE45D0E4ECF" />
    <Allow ID="ID_ALLOW_A_7F1" FriendlyName="C:\Program Files\git\usr\bin\logname.exe Hash Page Sha1" Hash="31CBDB8821DD850E74B490740192FEF234B95957" />
    <Allow ID="ID_ALLOW_A_7F2" FriendlyName="C:\Program Files\git\usr\bin\logname.exe Hash Page Sha256" Hash="E9414719AD8A95135EF7A0DC30B0FC9BA524C37FB53F9FCC484FA39A0A96A26C" />
    <Allow ID="ID_ALLOW_A_7F3" FriendlyName="C:\Program Files\git\usr\bin\ls.exe Hash Sha1" Hash="0C0A20357938E3A5D8759ED55959276FC8CBE573" />
    <Allow ID="ID_ALLOW_A_7F4" FriendlyName="C:\Program Files\git\usr\bin\ls.exe Hash Sha256" Hash="C8CF6B835B70BCA42F23513ECB1C3887EE06234E3AEFE768B55EE36F8CF4D81F" />
    <Allow ID="ID_ALLOW_A_7F5" FriendlyName="C:\Program Files\git\usr\bin\lsattr.exe Hash Sha1" Hash="D310F794DA71CBFD5F9F467A9569A48821BB919B" />
    <Allow ID="ID_ALLOW_A_7F6" FriendlyName="C:\Program Files\git\usr\bin\lsattr.exe Hash Sha256" Hash="69E45E63B813A43397AA639AA15755C587526BDBFA71BCD13DEA192F5CBDDDEB" />
    <Allow ID="ID_ALLOW_A_7F7" FriendlyName="C:\Program Files\git\usr\bin\lsattr.exe Hash Page Sha1" Hash="8120F32DEF977846320FF6C67899D4F01B872C93" />
    <Allow ID="ID_ALLOW_A_7F8" FriendlyName="C:\Program Files\git\usr\bin\lsattr.exe Hash Page Sha256" Hash="16DC1FE11E95F31E1AFC270340DFE5B6FB8A545F9D26B432CC84C914E035CBBA" />
    <Allow ID="ID_ALLOW_A_7F9" FriendlyName="C:\Program Files\git\usr\bin\md5sum.exe Hash Sha1" Hash="3F6AE516B90EB9928810FE9BC55660B767BFD00F" />
    <Allow ID="ID_ALLOW_A_7FA" FriendlyName="C:\Program Files\git\usr\bin\md5sum.exe Hash Sha256" Hash="A927E32C4A251568EF90422261DF064F5765D705C00A5779C09B319A9569D255" />
    <Allow ID="ID_ALLOW_A_7FB" FriendlyName="C:\Program Files\git\usr\bin\md5sum.exe Hash Page Sha1" Hash="053D201E91083A981055744C0797E8FAB174B8EF" />
    <Allow ID="ID_ALLOW_A_7FC" FriendlyName="C:\Program Files\git\usr\bin\md5sum.exe Hash Page Sha256" Hash="6560CCBB176A576A65768D147DF0660A421C62450093A575B325C199DB7EA3A7" />
    <Allow ID="ID_ALLOW_A_7FD" FriendlyName="C:\Program Files\git\usr\bin\minidumper.exe Hash Sha1" Hash="FFBA53C8F27740FA3DD5613251C69BEBA909FAE6" />
    <Allow ID="ID_ALLOW_A_7FE" FriendlyName="C:\Program Files\git\usr\bin\minidumper.exe Hash Sha256" Hash="EFBF1BE0F8F3781B5F80E909EC270D844B4D333D2E697E21075C6963DF801495" />
    <Allow ID="ID_ALLOW_A_7FF" FriendlyName="C:\Program Files\git\usr\bin\minidumper.exe Hash Page Sha1" Hash="A07CF1F5989779B70FCFDDF02DB205711E110A36" />
    <Allow ID="ID_ALLOW_A_800" FriendlyName="C:\Program Files\git\usr\bin\minidumper.exe Hash Page Sha256" Hash="5614A8E57CD7B4902306D933590407519C39A9C7F69ED259B314D114D42A47D3" />
    <Allow ID="ID_ALLOW_A_801" FriendlyName="C:\Program Files\git\usr\bin\mintty.exe Hash Sha1" Hash="B3FA63AA4312419F8BFB7F4D62FC827617124647" />
    <Allow ID="ID_ALLOW_A_802" FriendlyName="C:\Program Files\git\usr\bin\mintty.exe Hash Sha256" Hash="FBC468B1F34245212F79061731970D974AB6036873402C32D61EAD1339ADACCD" />
    <Allow ID="ID_ALLOW_A_803" FriendlyName="C:\Program Files\git\usr\bin\mintty.exe Hash Page Sha1" Hash="472683E809BFB93D00D7FEB211F493B68A4EAE03" />
    <Allow ID="ID_ALLOW_A_804" FriendlyName="C:\Program Files\git\usr\bin\mintty.exe Hash Page Sha256" Hash="392CC515B0FAAB74B7E8C436687BB27759B71ACC85582AEC835C5E0BE5EFE2E3" />
    <Allow ID="ID_ALLOW_A_805" FriendlyName="C:\Program Files\git\usr\bin\mkdir.exe Hash Sha1" Hash="14D81C75E71F501F1CA0BF426D78883B46470AB0" />
    <Allow ID="ID_ALLOW_A_806" FriendlyName="C:\Program Files\git\usr\bin\mkdir.exe Hash Sha256" Hash="FA6CBBAA9FCCBF81947919F009A53FE47BC2B3EBAF52BE0E4685853A397264E3" />
    <Allow ID="ID_ALLOW_A_807" FriendlyName="C:\Program Files\git\usr\bin\mkdir.exe Hash Page Sha1" Hash="8467CD322DF4D49A9ED2366D2AFC7D466B91AF27" />
    <Allow ID="ID_ALLOW_A_808" FriendlyName="C:\Program Files\git\usr\bin\mkdir.exe Hash Page Sha256" Hash="77303A3B460AD70A5B63D270FD7573FAA7CEE60BA6CFD382A4708885853B3AF9" />
    <Allow ID="ID_ALLOW_A_809" FriendlyName="C:\Program Files\git\usr\bin\mkfifo.exe Hash Sha1" Hash="556C9B6A94844C153F293715F85F9A3CB6C9B250" />
    <Allow ID="ID_ALLOW_A_80A" FriendlyName="C:\Program Files\git\usr\bin\mkfifo.exe Hash Sha256" Hash="6A3F28CED564418A8E768038F73B9C029622254555446E052F57800D7A933B9A" />
    <Allow ID="ID_ALLOW_A_80B" FriendlyName="C:\Program Files\git\usr\bin\mkfifo.exe Hash Page Sha1" Hash="6DDDDED7CE33BF9F2FD9246463B2BD6CCDB8E57A" />
    <Allow ID="ID_ALLOW_A_80C" FriendlyName="C:\Program Files\git\usr\bin\mkfifo.exe Hash Page Sha256" Hash="8F9E05F9129C3E238986B6C06EFA851738802842330F95B43FFE4C00807161FB" />
    <Allow ID="ID_ALLOW_A_80D" FriendlyName="C:\Program Files\git\usr\bin\mkgroup.exe Hash Sha1" Hash="92E38D1B799C7CA020A1F915F8A76CC3003CB333" />
    <Allow ID="ID_ALLOW_A_80E" FriendlyName="C:\Program Files\git\usr\bin\mkgroup.exe Hash Sha256" Hash="4593A89586479C12026C6685CB2E88369211837A3373E20F5F641DCAA103FCE0" />
    <Allow ID="ID_ALLOW_A_80F" FriendlyName="C:\Program Files\git\usr\bin\mkgroup.exe Hash Page Sha1" Hash="C1C37B6AB8B1C41776F4C6A2B613EA11C1A82B50" />
    <Allow ID="ID_ALLOW_A_810" FriendlyName="C:\Program Files\git\usr\bin\mkgroup.exe Hash Page Sha256" Hash="649C11348DAE8B06F81092A743DE466078A92B0D20837BEA9127963BFBAC8BF7" />
    <Allow ID="ID_ALLOW_A_811" FriendlyName="C:\Program Files\git\usr\bin\mknod.exe Hash Sha1" Hash="C8A988FF033CCA41D4DA74C061C9FBDD06959F97" />
    <Allow ID="ID_ALLOW_A_812" FriendlyName="C:\Program Files\git\usr\bin\mknod.exe Hash Sha256" Hash="BB403F0544457034224E00AAFBF1F08152D1252A90CA31BF63F174D19894A31C" />
    <Allow ID="ID_ALLOW_A_813" FriendlyName="C:\Program Files\git\usr\bin\mknod.exe Hash Page Sha1" Hash="DD87655546982DEA6AFE53799CA1177FBC159038" />
    <Allow ID="ID_ALLOW_A_814" FriendlyName="C:\Program Files\git\usr\bin\mknod.exe Hash Page Sha256" Hash="CCCA0E2EFBFF9603F7BF3DF08EE1D71FE64CAD0BB8C5B2E487BE1F0ECAE86034" />
    <Allow ID="ID_ALLOW_A_815" FriendlyName="C:\Program Files\git\usr\bin\mkpasswd.exe Hash Sha1" Hash="12BA03710CF000AEC9FF152598E4B6882DFD2208" />
    <Allow ID="ID_ALLOW_A_816" FriendlyName="C:\Program Files\git\usr\bin\mkpasswd.exe Hash Sha256" Hash="3269D579535FF975D407E5C65AA766A6B41B34EBAE9C5BA0904821CC569B34FB" />
    <Allow ID="ID_ALLOW_A_817" FriendlyName="C:\Program Files\git\usr\bin\mkpasswd.exe Hash Page Sha1" Hash="563E7EB4FA9627A4B246903F58E93D0B941270E1" />
    <Allow ID="ID_ALLOW_A_818" FriendlyName="C:\Program Files\git\usr\bin\mkpasswd.exe Hash Page Sha256" Hash="2E96CFD1E6899B173D3824CFC6F84B66C515A8E6835CEE6EC0AA4AD2E3786B63" />
    <Allow ID="ID_ALLOW_A_819" FriendlyName="C:\Program Files\git\usr\bin\mktemp.exe Hash Sha1" Hash="157527F21C02D23B9B6D4A951C683AEB9F2BAA02" />
    <Allow ID="ID_ALLOW_A_81A" FriendlyName="C:\Program Files\git\usr\bin\mktemp.exe Hash Sha256" Hash="5EFC22B86B4441FBA3019B1D9B2652413E0B3A7F7A6E45618F73E702219A01DF" />
    <Allow ID="ID_ALLOW_A_81B" FriendlyName="C:\Program Files\git\usr\bin\mktemp.exe Hash Page Sha1" Hash="4DE34C2075ED2F7459E83C76725C46A9057BF28E" />
    <Allow ID="ID_ALLOW_A_81C" FriendlyName="C:\Program Files\git\usr\bin\mktemp.exe Hash Page Sha256" Hash="B9C760978541D77B7B3E095DB1390CF8D20745BE831F2F045748A45017411D70" />
    <Allow ID="ID_ALLOW_A_81D" FriendlyName="C:\Program Files\git\usr\bin\mount.exe Hash Sha1" Hash="1FD76F9D49BBBABC5EED8189C5B7BEE93ACCEED3" />
    <Allow ID="ID_ALLOW_A_81E" FriendlyName="C:\Program Files\git\usr\bin\mount.exe Hash Sha256" Hash="FE51BA3CD18150D2BB35A80044F68F05D018F6405B9712DCCB61BE3CA7C15CD5" />
    <Allow ID="ID_ALLOW_A_81F" FriendlyName="C:\Program Files\git\usr\bin\mount.exe Hash Page Sha1" Hash="73EC1B658E0C8BB9F6F9EF8F4BA5E4B3B1AC64DA" />
    <Allow ID="ID_ALLOW_A_820" FriendlyName="C:\Program Files\git\usr\bin\mount.exe Hash Page Sha256" Hash="B1DF880EC14AD41D6FFED05A8008BD3F5F18B15EEF9B5E2A8B209AA7A07ED1AF" />
    <Allow ID="ID_ALLOW_A_821" FriendlyName="C:\Program Files\git\usr\bin\mpicalc.exe Hash Sha1" Hash="F08DE9CF4AD57A98D32F94E411955E8696BC6897" />
    <Allow ID="ID_ALLOW_A_822" FriendlyName="C:\Program Files\git\usr\bin\mpicalc.exe Hash Sha256" Hash="B9D1C83ED4ED8429CF949CCDF60E48B854E0938D833155A11195EA34CAF78266" />
    <Allow ID="ID_ALLOW_A_823" FriendlyName="C:\Program Files\git\usr\bin\mpicalc.exe Hash Page Sha1" Hash="BF37AEC098171AEE76DD43C10815D6E80A931E8F" />
    <Allow ID="ID_ALLOW_A_824" FriendlyName="C:\Program Files\git\usr\bin\mpicalc.exe Hash Page Sha256" Hash="99148130F640A5A61719B58BC2D06392F1B268FBF99CFD894A1CC56A86711B3C" />
    <Allow ID="ID_ALLOW_A_825" FriendlyName="C:\Program Files\git\usr\bin\msgattrib.exe Hash Sha1" Hash="FAEE6B67A55AC8FC543B8856CACF40C6B1DE1C7F" />
    <Allow ID="ID_ALLOW_A_826" FriendlyName="C:\Program Files\git\usr\bin\msgattrib.exe Hash Sha256" Hash="0BD6B071BE6161C7AAE8DD7C980E26A22E4BDD72D14F8E21FBD0BFD001D4503D" />
    <Allow ID="ID_ALLOW_A_827" FriendlyName="C:\Program Files\git\usr\bin\msgattrib.exe Hash Page Sha1" Hash="7A074A0BA414484F8F001C33B4496D1828CD3724" />
    <Allow ID="ID_ALLOW_A_828" FriendlyName="C:\Program Files\git\usr\bin\msgattrib.exe Hash Page Sha256" Hash="BB83CE48B841025E81EA52357A5F638AA48D2DFE7213CAD568DE84A89F9FFEEF" />
    <Allow ID="ID_ALLOW_A_829" FriendlyName="C:\Program Files\git\usr\bin\msgcat.exe Hash Sha1" Hash="7198D2766C164649413EE084DCF8608FD8076BAE" />
    <Allow ID="ID_ALLOW_A_82A" FriendlyName="C:\Program Files\git\usr\bin\msgcat.exe Hash Sha256" Hash="1B09A986E04667D4CB038F0CCC74977B2CAEBB12CCCF158776DEBF21FAF955DD" />
    <Allow ID="ID_ALLOW_A_82B" FriendlyName="C:\Program Files\git\usr\bin\msgcat.exe Hash Page Sha1" Hash="FC0C9D4D255F2BA6647600818F453E7E193745D0" />
    <Allow ID="ID_ALLOW_A_82C" FriendlyName="C:\Program Files\git\usr\bin\msgcat.exe Hash Page Sha256" Hash="707F7509D42FA138C60D1BD6C6036980C3A4165505A26484237A91A83593E46A" />
    <Allow ID="ID_ALLOW_A_82D" FriendlyName="C:\Program Files\git\usr\bin\msgcmp.exe Hash Sha1" Hash="B53C79045D8A9AC285DA867A3A9F8E53ABE6637C" />
    <Allow ID="ID_ALLOW_A_82E" FriendlyName="C:\Program Files\git\usr\bin\msgcmp.exe Hash Sha256" Hash="BDAF5B19986AC31263053ECF4F1B8EF6FDAC52CCD676CDD4AB1676AA9AF7817B" />
    <Allow ID="ID_ALLOW_A_82F" FriendlyName="C:\Program Files\git\usr\bin\msgcmp.exe Hash Page Sha1" Hash="A278A2CD99327D85A74B3EEAB58DE7295787025A" />
    <Allow ID="ID_ALLOW_A_830" FriendlyName="C:\Program Files\git\usr\bin\msgcmp.exe Hash Page Sha256" Hash="1DBFB545D68F73B382AB082765591CF629736D2030AAC3F0E335AFB2CC9CE4EB" />
    <Allow ID="ID_ALLOW_A_831" FriendlyName="C:\Program Files\git\usr\bin\msgcomm.exe Hash Sha1" Hash="4A1013AF09077E078CECF545A740A04D42A4F772" />
    <Allow ID="ID_ALLOW_A_832" FriendlyName="C:\Program Files\git\usr\bin\msgcomm.exe Hash Sha256" Hash="5F777B2C9C4B39DE91559AEC6B75E85014950A890AA6698C96CAACE6A328927C" />
    <Allow ID="ID_ALLOW_A_833" FriendlyName="C:\Program Files\git\usr\bin\msgcomm.exe Hash Page Sha1" Hash="7924227B1DDA8107EEAAD6C78EA4545C5792B192" />
    <Allow ID="ID_ALLOW_A_834" FriendlyName="C:\Program Files\git\usr\bin\msgcomm.exe Hash Page Sha256" Hash="FB89961A9F43EFD2E6566380C5DB9234F2194586CCAAC36D8AF00A2B350302E0" />
    <Allow ID="ID_ALLOW_A_835" FriendlyName="C:\Program Files\git\usr\bin\msgconv.exe Hash Sha1" Hash="40024A31C43F08EA5D1C34F338F33366C7C59A14" />
    <Allow ID="ID_ALLOW_A_836" FriendlyName="C:\Program Files\git\usr\bin\msgconv.exe Hash Sha256" Hash="4E28BF10A0B8D52C693237F029741A6007718E133F14F3BBCBD86F59206D2639" />
    <Allow ID="ID_ALLOW_A_837" FriendlyName="C:\Program Files\git\usr\bin\msgconv.exe Hash Page Sha1" Hash="0045A2477627602DFCDC52DEA252E54C99EADC6D" />
    <Allow ID="ID_ALLOW_A_838" FriendlyName="C:\Program Files\git\usr\bin\msgconv.exe Hash Page Sha256" Hash="EE82E80F7A2F12EDFCA200449594623F4878E92A232B58F62AA11E0CA0AA63E7" />
    <Allow ID="ID_ALLOW_A_839" FriendlyName="C:\Program Files\git\usr\bin\msgen.exe Hash Sha1" Hash="BE7A05453CBE4BE65A9D600CD40A613BAA528550" />
    <Allow ID="ID_ALLOW_A_83A" FriendlyName="C:\Program Files\git\usr\bin\msgen.exe Hash Sha256" Hash="C65495534300899DC63B4EEF0E185E304C5974DB1828A94787142FCBA0F2AE64" />
    <Allow ID="ID_ALLOW_A_83B" FriendlyName="C:\Program Files\git\usr\bin\msgen.exe Hash Page Sha1" Hash="860AEBB71C372C980DF3762632854FF0B2AF51EF" />
    <Allow ID="ID_ALLOW_A_83C" FriendlyName="C:\Program Files\git\usr\bin\msgen.exe Hash Page Sha256" Hash="97E927F09F840898DC77017B43C3E3CD9D9CFAF9EC33614CC9631C3A1749CCB1" />
    <Allow ID="ID_ALLOW_A_83D" FriendlyName="C:\Program Files\git\usr\bin\msgexec.exe Hash Sha1" Hash="B68F382403E3B71A9B844EC56D01E7CE5023D407" />
    <Allow ID="ID_ALLOW_A_83E" FriendlyName="C:\Program Files\git\usr\bin\msgexec.exe Hash Sha256" Hash="521413E1A9AFF61CA26CEEBC1757FF27331098DB5CD903B25AA462DA0BB26B40" />
    <Allow ID="ID_ALLOW_A_83F" FriendlyName="C:\Program Files\git\usr\bin\msgexec.exe Hash Page Sha1" Hash="F5A739D7F2C9026632FDE75074CB731EB3D65D57" />
    <Allow ID="ID_ALLOW_A_840" FriendlyName="C:\Program Files\git\usr\bin\msgexec.exe Hash Page Sha256" Hash="463E0955F69F366868F91A955CA5D67A0258091E48B365F66C084C663D5F3184" />
    <Allow ID="ID_ALLOW_A_841" FriendlyName="C:\Program Files\git\usr\bin\msgfilter.exe Hash Sha1" Hash="3DCF7B47C89E5577FF52486087E0BC98F821E602" />
    <Allow ID="ID_ALLOW_A_842" FriendlyName="C:\Program Files\git\usr\bin\msgfilter.exe Hash Sha256" Hash="1E40CBE69C01EA3375B22D82C45958D59BBF600CBB30108A7EE0896DB76B4422" />
    <Allow ID="ID_ALLOW_A_843" FriendlyName="C:\Program Files\git\usr\bin\msgfilter.exe Hash Page Sha1" Hash="1756A57D4FE6953127E3DFCB6F6D9DD74A81B75C" />
    <Allow ID="ID_ALLOW_A_844" FriendlyName="C:\Program Files\git\usr\bin\msgfilter.exe Hash Page Sha256" Hash="1B2CF6B1B87B0DDE7D9D2BA81240ED62C6699B1D9381601AB806112FA3543B1E" />
    <Allow ID="ID_ALLOW_A_845" FriendlyName="C:\Program Files\git\usr\bin\msgfmt.exe Hash Sha1" Hash="F201A68EDFBAEE69BFA181502B8B56E73EEF1215" />
    <Allow ID="ID_ALLOW_A_846" FriendlyName="C:\Program Files\git\usr\bin\msgfmt.exe Hash Sha256" Hash="B30C2BDB97B008EBB1CDA507BEFCA0A0BDD273B9F2AC1E31E67FF87D5921FD14" />
    <Allow ID="ID_ALLOW_A_847" FriendlyName="C:\Program Files\git\usr\bin\msgfmt.exe Hash Page Sha1" Hash="B4D012F49908B87778885C66A74104B099AF14EA" />
    <Allow ID="ID_ALLOW_A_848" FriendlyName="C:\Program Files\git\usr\bin\msgfmt.exe Hash Page Sha256" Hash="CF58A7BCB9DC8601807B7B85DBEA99FE3CAFA3AC12D3D6C696B2E32A79294AF5" />
    <Allow ID="ID_ALLOW_A_849" FriendlyName="C:\Program Files\git\usr\bin\msggrep.exe Hash Sha1" Hash="864288C76F1DE17A07E3BB54C2AFE131FC50A872" />
    <Allow ID="ID_ALLOW_A_84A" FriendlyName="C:\Program Files\git\usr\bin\msggrep.exe Hash Sha256" Hash="659C514E78D37DE7034094E7C0B58D1823552C7394B934B04B7168F2D74B467E" />
    <Allow ID="ID_ALLOW_A_84B" FriendlyName="C:\Program Files\git\usr\bin\msggrep.exe Hash Page Sha1" Hash="14A9DF43641BD2D4F26D92BD44820FC86ACEB313" />
    <Allow ID="ID_ALLOW_A_84C" FriendlyName="C:\Program Files\git\usr\bin\msggrep.exe Hash Page Sha256" Hash="394AB892525D377049AB538DD415E32F487F9C1CC55F249F331F9124DE63D207" />
    <Allow ID="ID_ALLOW_A_84D" FriendlyName="C:\Program Files\git\usr\bin\msginit.exe Hash Sha1" Hash="E4D9AF0D421D192E3C5D48EB8C6BC2619C741960" />
    <Allow ID="ID_ALLOW_A_84E" FriendlyName="C:\Program Files\git\usr\bin\msginit.exe Hash Sha256" Hash="42C70E73CF3BA490EBC28C8384380F1DE72DA93999A651B8621390311B109F39" />
    <Allow ID="ID_ALLOW_A_84F" FriendlyName="C:\Program Files\git\usr\bin\msginit.exe Hash Page Sha1" Hash="AD764A709EAF1B85AEBAFD25731A5976D7997723" />
    <Allow ID="ID_ALLOW_A_850" FriendlyName="C:\Program Files\git\usr\bin\msginit.exe Hash Page Sha256" Hash="7B8A91D577A366E494C66E8DD29F9C546691CBE227061E62E11711323B2A22F8" />
    <Allow ID="ID_ALLOW_A_851" FriendlyName="C:\Program Files\git\usr\bin\msgmerge.exe Hash Sha1" Hash="5F975981E7DF48C446D2075615DC2A71F4141C54" />
    <Allow ID="ID_ALLOW_A_852" FriendlyName="C:\Program Files\git\usr\bin\msgmerge.exe Hash Sha256" Hash="D7257D788CD27FAD314D01F845DB8B1851AD0276E3865D9334A9B9EE156BC4D5" />
    <Allow ID="ID_ALLOW_A_853" FriendlyName="C:\Program Files\git\usr\bin\msgmerge.exe Hash Page Sha1" Hash="F60BE4C209E3C1B56B59A54FEDDB0A393441E6DC" />
    <Allow ID="ID_ALLOW_A_854" FriendlyName="C:\Program Files\git\usr\bin\msgmerge.exe Hash Page Sha256" Hash="C3A3F76671A4117FE7AD9DEF41A5FF77133028452E6289E2D3E2A06B7DC8C7D6" />
    <Allow ID="ID_ALLOW_A_855" FriendlyName="C:\Program Files\git\usr\bin\msgunfmt.exe Hash Sha1" Hash="95A63ACCAA1594FD1D9973A5195BFCCB9FF5421E" />
    <Allow ID="ID_ALLOW_A_856" FriendlyName="C:\Program Files\git\usr\bin\msgunfmt.exe Hash Sha256" Hash="56DC31264B6EE2698A9C2C6D33540051251AD216E5D20EDA1D866C852EA56EDA" />
    <Allow ID="ID_ALLOW_A_857" FriendlyName="C:\Program Files\git\usr\bin\msgunfmt.exe Hash Page Sha1" Hash="7DFCC6286163B7B67A269E00B91B98C53B966531" />
    <Allow ID="ID_ALLOW_A_858" FriendlyName="C:\Program Files\git\usr\bin\msgunfmt.exe Hash Page Sha256" Hash="4DDB9D0FDB66C962A6366949A6DCA988AFB7F16392653DEFEB8A082CD43B1065" />
    <Allow ID="ID_ALLOW_A_859" FriendlyName="C:\Program Files\git\usr\bin\msguniq.exe Hash Sha1" Hash="990CEAA1C1349570607A0F2DD940C4A5F01F9228" />
    <Allow ID="ID_ALLOW_A_85A" FriendlyName="C:\Program Files\git\usr\bin\msguniq.exe Hash Sha256" Hash="4126183A31A3EA21D3455FE18320EFECE50D6E51D94518B5F456A22C6A76048E" />
    <Allow ID="ID_ALLOW_A_85B" FriendlyName="C:\Program Files\git\usr\bin\msguniq.exe Hash Page Sha1" Hash="C730157074F71CC70FD7211BD1CC82F0B8C88D1B" />
    <Allow ID="ID_ALLOW_A_85C" FriendlyName="C:\Program Files\git\usr\bin\msguniq.exe Hash Page Sha256" Hash="BF66B03F2CB45E1099F30D07175E620515FF2EB0199880E0C8B6DBB0A85B0745" />
    <Allow ID="ID_ALLOW_A_85D" FriendlyName="C:\Program Files\git\usr\bin\msys-2.0.dll Hash Sha1" Hash="CD81A6D9BA8DAEDFE61C3EB478430A2C7455739D" />
    <Allow ID="ID_ALLOW_A_85E" FriendlyName="C:\Program Files\git\usr\bin\msys-2.0.dll Hash Sha256" Hash="89E5268730C413E9DA5C3C6E89E02BDE42BFFA4A6744B62CF780113CD79860FA" />
    <Allow ID="ID_ALLOW_A_85F" FriendlyName="C:\Program Files\git\usr\bin\msys-2.0.dll Hash Page Sha1" Hash="891E787D85FD9635223830BE16BCD1B4550D07AD" />
    <Allow ID="ID_ALLOW_A_860" FriendlyName="C:\Program Files\git\usr\bin\msys-2.0.dll Hash Page Sha256" Hash="E581C7163722DC5033991844F5DF4A6978E2496D56211F68954BD81D2FA9B655" />
    <Allow ID="ID_ALLOW_A_861" FriendlyName="C:\Program Files\git\usr\bin\msys-apr-1-0.dll Hash Sha1" Hash="D465BF3BECD5E88800DE094CD46559C66F162537" />
    <Allow ID="ID_ALLOW_A_862" FriendlyName="C:\Program Files\git\usr\bin\msys-apr-1-0.dll Hash Sha256" Hash="BFFFE2831280B2AF3F6F3043937B2EE33B52E9147AE8E084E9B1E8776F4CB775" />
    <Allow ID="ID_ALLOW_A_863" FriendlyName="C:\Program Files\git\usr\bin\msys-apr-1-0.dll Hash Page Sha1" Hash="577A503DEDB9E14103F30127C6E6C6D6CEAB5383" />
    <Allow ID="ID_ALLOW_A_864" FriendlyName="C:\Program Files\git\usr\bin\msys-apr-1-0.dll Hash Page Sha256" Hash="57B43C5DBB1172D0301E7EBBF1E9CEB1C01258138EED7A570B9FB4BF19093885" />
    <Allow ID="ID_ALLOW_A_865" FriendlyName="C:\Program Files\git\usr\bin\msys-aprutil-1-0.dll Hash Sha1" Hash="44002AF7FE6C2FEE1BE79FF63020B0E2E49737D9" />
    <Allow ID="ID_ALLOW_A_866" FriendlyName="C:\Program Files\git\usr\bin\msys-aprutil-1-0.dll Hash Sha256" Hash="8CB14A5EB8EB1B9C0D05E926C9F21B7D252A456E82DB19134F65F0106147932D" />
    <Allow ID="ID_ALLOW_A_867" FriendlyName="C:\Program Files\git\usr\bin\msys-aprutil-1-0.dll Hash Page Sha1" Hash="E78A4E4A96FC2BB306B30ABC8E47D005CFCC3093" />
    <Allow ID="ID_ALLOW_A_868" FriendlyName="C:\Program Files\git\usr\bin\msys-aprutil-1-0.dll Hash Page Sha256" Hash="828F879030BBE0E0C8EA4A2446642111F597417F8A9575C83A1C0CF1F49E223D" />
    <Allow ID="ID_ALLOW_A_869" FriendlyName="C:\Program Files\git\usr\bin\msys-asn1-8.dll Hash Sha1" Hash="5F791B52BA64CE68EBA2E46CBA2611D91FE262C4" />
    <Allow ID="ID_ALLOW_A_86A" FriendlyName="C:\Program Files\git\usr\bin\msys-asn1-8.dll Hash Sha256" Hash="0BA06EF3E4C6A953CEDF43B9B1418A62D5683391A8E90FF3AE07F02EBE37621D" />
    <Allow ID="ID_ALLOW_A_86B" FriendlyName="C:\Program Files\git\usr\bin\msys-asn1-8.dll Hash Page Sha1" Hash="6B74393DCEB21315B7782BB8FA648C2532ACF588" />
    <Allow ID="ID_ALLOW_A_86C" FriendlyName="C:\Program Files\git\usr\bin\msys-asn1-8.dll Hash Page Sha256" Hash="2A7458B3685B4B9EC2B7379346E22AAF3519BD23F0020230AF25E4D2932A867D" />
    <Allow ID="ID_ALLOW_A_86D" FriendlyName="C:\Program Files\git\usr\bin\msys-assuan-0.dll Hash Sha1" Hash="97ADAC7F6AB236D26EEC37AA5B91C322A359CB28" />
    <Allow ID="ID_ALLOW_A_86E" FriendlyName="C:\Program Files\git\usr\bin\msys-assuan-0.dll Hash Sha256" Hash="5891245DF43D3CCD03FCBE438E950B2E7325014F29A0D6D20F41767768424276" />
    <Allow ID="ID_ALLOW_A_86F" FriendlyName="C:\Program Files\git\usr\bin\msys-assuan-0.dll Hash Page Sha1" Hash="AA7B880FCA128CD1EE6E519A7385A9D4FCF95AE2" />
    <Allow ID="ID_ALLOW_A_870" FriendlyName="C:\Program Files\git\usr\bin\msys-assuan-0.dll Hash Page Sha256" Hash="6D1705D2F6E19AD405878268391E2A91CF0A5BF39B585A4BF2B23B331E17B830" />
    <Allow ID="ID_ALLOW_A_871" FriendlyName="C:\Program Files\git\usr\bin\msys-bz2-1.dll Hash Sha1" Hash="346D599E985800AF16FA7DD3AFA2E3F0455B0385" />
    <Allow ID="ID_ALLOW_A_872" FriendlyName="C:\Program Files\git\usr\bin\msys-bz2-1.dll Hash Sha256" Hash="19978B371830BC95ACBD79F4A9F6A2575FD2752479A13E0282148A591ACF190D" />
    <Allow ID="ID_ALLOW_A_873" FriendlyName="C:\Program Files\git\usr\bin\msys-bz2-1.dll Hash Page Sha1" Hash="4521587FFB5D1305417C376659A3C0DEAC5E9A22" />
    <Allow ID="ID_ALLOW_A_874" FriendlyName="C:\Program Files\git\usr\bin\msys-bz2-1.dll Hash Page Sha256" Hash="F9CC178476329C59B8CD25CF8CD094C2E932CF7D6ECA74137E16355B1C6CCCFC" />
    <Allow ID="ID_ALLOW_A_875" FriendlyName="C:\Program Files\git\usr\bin\msys-cbor-0.8.dll Hash Sha1" Hash="D9B81C8710AF605918D812FA42EFEA9274B6247A" />
    <Allow ID="ID_ALLOW_A_876" FriendlyName="C:\Program Files\git\usr\bin\msys-cbor-0.8.dll Hash Sha256" Hash="F51EF177876F93E7C0C3F925A19A15556038FEA2809A9AC655CDA11B8181EFEB" />
    <Allow ID="ID_ALLOW_A_877" FriendlyName="C:\Program Files\git\usr\bin\msys-cbor-0.8.dll Hash Page Sha1" Hash="3FC0181B8E387CF50000909DEE59D8560AFB378D" />
    <Allow ID="ID_ALLOW_A_878" FriendlyName="C:\Program Files\git\usr\bin\msys-cbor-0.8.dll Hash Page Sha256" Hash="E61E3CDF2837F93909DE30DDA7CECA701B83284D1F67BBE3DD418C6955DB85EE" />
    <Allow ID="ID_ALLOW_A_879" FriendlyName="C:\Program Files\git\usr\bin\msys-com_err-1.dll Hash Sha1" Hash="18FE90D14295BB38DA20FEB91200F760C20B9C91" />
    <Allow ID="ID_ALLOW_A_87A" FriendlyName="C:\Program Files\git\usr\bin\msys-com_err-1.dll Hash Sha256" Hash="80C7C5B2C71AEB26F4DE3CA7FD7DBFB76897159EFAC6D17961DAA62D3BBEB111" />
    <Allow ID="ID_ALLOW_A_87B" FriendlyName="C:\Program Files\git\usr\bin\msys-com_err-1.dll Hash Page Sha1" Hash="A12DBF6C437F70FA373859A3F4190BB5782A0F13" />
    <Allow ID="ID_ALLOW_A_87C" FriendlyName="C:\Program Files\git\usr\bin\msys-com_err-1.dll Hash Page Sha256" Hash="61D6A1D7EBF940F7D3D95C0EF591402B9344A5CE2DC85216340B6DD04DB37BA1" />
    <Allow ID="ID_ALLOW_A_87D" FriendlyName="C:\Program Files\git\usr\bin\msys-crypt-0.dll Hash Sha1" Hash="33A5BFF28E622FFA719771CC8464D086074FCCA5" />
    <Allow ID="ID_ALLOW_A_87E" FriendlyName="C:\Program Files\git\usr\bin\msys-crypt-0.dll Hash Sha256" Hash="B5D77DB78BEA42270CD3C867379F0731231C742E039D0E373F83505F44438FD0" />
    <Allow ID="ID_ALLOW_A_87F" FriendlyName="C:\Program Files\git\usr\bin\msys-crypt-0.dll Hash Page Sha1" Hash="8B49CE501DC89D29C0D5F2914832DBE75B556581" />
    <Allow ID="ID_ALLOW_A_880" FriendlyName="C:\Program Files\git\usr\bin\msys-crypt-0.dll Hash Page Sha256" Hash="38A6D09E63DA5049B5E0C6BF7CCCA4C4CABF240A0300F4B1B9B7A4D81307215D" />
    <Allow ID="ID_ALLOW_A_881" FriendlyName="C:\Program Files\git\usr\bin\msys-crypto-1.1.dll Hash Sha1" Hash="B289E7C72B6D4AF9F1DCAC23F869E4DFC546C06D" />
    <Allow ID="ID_ALLOW_A_882" FriendlyName="C:\Program Files\git\usr\bin\msys-crypto-1.1.dll Hash Sha256" Hash="C05E33363F31B4BC7D9DC9C929B909761B6A6895F2ECFB8951EFFC92BD520ED1" />
    <Allow ID="ID_ALLOW_A_883" FriendlyName="C:\Program Files\git\usr\bin\msys-crypto-1.1.dll Hash Page Sha1" Hash="F379B5625738FE25F5CBFEA96071A9443E2E084E" />
    <Allow ID="ID_ALLOW_A_884" FriendlyName="C:\Program Files\git\usr\bin\msys-crypto-1.1.dll Hash Page Sha256" Hash="393FE2C29672B870F7D266AD05AD0474433EABCA7AD9838C05C74CE9E0A103D0" />
    <Allow ID="ID_ALLOW_A_885" FriendlyName="C:\Program Files\git\usr\bin\msys-edit-0.dll Hash Sha1" Hash="D32C8AB01C1E4AF39E2D9E9D29CDFE93FCA8D3F8" />
    <Allow ID="ID_ALLOW_A_886" FriendlyName="C:\Program Files\git\usr\bin\msys-edit-0.dll Hash Sha256" Hash="B169AC3979ADA59ACAAAA798CC6C90940C5D2CED0A4D6FCB1C80F47B4255BE5C" />
    <Allow ID="ID_ALLOW_A_887" FriendlyName="C:\Program Files\git\usr\bin\msys-edit-0.dll Hash Page Sha1" Hash="871B48644FF862F85283792460CA0086DEA6DF8E" />
    <Allow ID="ID_ALLOW_A_888" FriendlyName="C:\Program Files\git\usr\bin\msys-edit-0.dll Hash Page Sha256" Hash="0A4888670AC087505796D11A4D78CD3712024F013855E7E1B753D786083A295A" />
    <Allow ID="ID_ALLOW_A_889" FriendlyName="C:\Program Files\git\usr\bin\msys-expat-1.dll Hash Sha1" Hash="268CCE8C327BE3E81C65B5410594E430E381011A" />
    <Allow ID="ID_ALLOW_A_88A" FriendlyName="C:\Program Files\git\usr\bin\msys-expat-1.dll Hash Sha256" Hash="B8FD74B7A4023417E623AFF0B78780EEAB17850B2B12F892A3557439F9BC7D53" />
    <Allow ID="ID_ALLOW_A_88B" FriendlyName="C:\Program Files\git\usr\bin\msys-expat-1.dll Hash Page Sha1" Hash="78014AB392FEB70C151784CFF9502512BA1DD433" />
    <Allow ID="ID_ALLOW_A_88C" FriendlyName="C:\Program Files\git\usr\bin\msys-expat-1.dll Hash Page Sha256" Hash="9CDFAA299E2AED93AC245B9CD431F7A985F3E9813454013CF42DD6073134F67F" />
    <Allow ID="ID_ALLOW_A_88D" FriendlyName="C:\Program Files\git\usr\bin\msys-ffi-7.dll Hash Sha1" Hash="D48C5E9F892B18774ECC22EB8D7370E5AA1FA19B" />
    <Allow ID="ID_ALLOW_A_88E" FriendlyName="C:\Program Files\git\usr\bin\msys-ffi-7.dll Hash Sha256" Hash="12E72FAB9AC8B5D2B1A835B26307C567B41A8DCE91C25363EFA1299996F95909" />
    <Allow ID="ID_ALLOW_A_88F" FriendlyName="C:\Program Files\git\usr\bin\msys-ffi-7.dll Hash Page Sha1" Hash="23E4F930F5F5321C0331DF11124748124FC5EB5E" />
    <Allow ID="ID_ALLOW_A_890" FriendlyName="C:\Program Files\git\usr\bin\msys-ffi-7.dll Hash Page Sha256" Hash="6F2261EEF97E41E50ABF8E4359CBAA13AA9132A1D27EE3CF1A9733DBCCEC112F" />
    <Allow ID="ID_ALLOW_A_891" FriendlyName="C:\Program Files\git\usr\bin\msys-fido2-1.dll Hash Sha1" Hash="0DE4E3E71C4FD895A68A7F3CA89F5ACBBBC4E746" />
    <Allow ID="ID_ALLOW_A_892" FriendlyName="C:\Program Files\git\usr\bin\msys-fido2-1.dll Hash Sha256" Hash="7C9727043C979B1329B3BBF51F1D1A6C3269724F4F4E7C14DBD5C925DF303F5F" />
    <Allow ID="ID_ALLOW_A_893" FriendlyName="C:\Program Files\git\usr\bin\msys-fido2-1.dll Hash Page Sha1" Hash="7865A77B09CAB90BA51F7D364017BD506373D0C0" />
    <Allow ID="ID_ALLOW_A_894" FriendlyName="C:\Program Files\git\usr\bin\msys-fido2-1.dll Hash Page Sha256" Hash="CE853859EDDEF4D26E3E074D8F5CB7C5772EA38D268D290F8707F8D3542A6D53" />
    <Allow ID="ID_ALLOW_A_895" FriendlyName="C:\Program Files\git\usr\bin\msys-gcc_s-seh-1.dll Hash Sha1" Hash="E5582E23889ADA18B7E4BFE9412716929075D88A" />
    <Allow ID="ID_ALLOW_A_896" FriendlyName="C:\Program Files\git\usr\bin\msys-gcc_s-seh-1.dll Hash Sha256" Hash="23ABF6771DCF58D3FC61C35612DB7A0BBA6D600338E81B5C55C3A13B2352191A" />
    <Allow ID="ID_ALLOW_A_897" FriendlyName="C:\Program Files\git\usr\bin\msys-gcc_s-seh-1.dll Hash Page Sha1" Hash="1441A09BC83570B08713C5BFB7B1F33FA5B1058B" />
    <Allow ID="ID_ALLOW_A_898" FriendlyName="C:\Program Files\git\usr\bin\msys-gcc_s-seh-1.dll Hash Page Sha256" Hash="D1C3BC923449452E76FC94E47847C342919A9A1522654D8DD666E7B40F086140" />
    <Allow ID="ID_ALLOW_A_899" FriendlyName="C:\Program Files\git\usr\bin\msys-gcrypt-20.dll Hash Sha1" Hash="C75AE684ECBF3D09CF4BCE1BBE3BD2A5F9E701E1" />
    <Allow ID="ID_ALLOW_A_89A" FriendlyName="C:\Program Files\git\usr\bin\msys-gcrypt-20.dll Hash Sha256" Hash="787A7CBD32D51C3AF2187CD78FFE78493892A65BFC2A1451B504EC02416DA1C7" />
    <Allow ID="ID_ALLOW_A_89B" FriendlyName="C:\Program Files\git\usr\bin\msys-gcrypt-20.dll Hash Page Sha1" Hash="453B5A77E5ED185EE1878C94545DE3860B8D146F" />
    <Allow ID="ID_ALLOW_A_89C" FriendlyName="C:\Program Files\git\usr\bin\msys-gcrypt-20.dll Hash Page Sha256" Hash="7FEB30499FA9C5F62490AE2F27A34F09C81740BD3C8DB9338185018E3F7A0F42" />
    <Allow ID="ID_ALLOW_A_89D" FriendlyName="C:\Program Files\git\usr\bin\msys-gettextlib-0-19-8-1.dll Hash Sha1" Hash="D8CC70A1333A43D97AC281803905B348EFF47F08" />
    <Allow ID="ID_ALLOW_A_89E" FriendlyName="C:\Program Files\git\usr\bin\msys-gettextlib-0-19-8-1.dll Hash Sha256" Hash="858FCFECBEF2FD3FE87F3AE8F6EEECB8DFFB48220B13EA54BD30585BA87ABCC6" />
    <Allow ID="ID_ALLOW_A_89F" FriendlyName="C:\Program Files\git\usr\bin\msys-gettextlib-0-19-8-1.dll Hash Page Sha1" Hash="44E0C4A6646DCE491CAC9EB9E667B866DD069934" />
    <Allow ID="ID_ALLOW_A_8A0" FriendlyName="C:\Program Files\git\usr\bin\msys-gettextlib-0-19-8-1.dll Hash Page Sha256" Hash="4471775BBD1FD31A7FFCEE8C911AA0102B97A9F390C321FAE22103F7120FCC2B" />
    <Allow ID="ID_ALLOW_A_8A1" FriendlyName="C:\Program Files\git\usr\bin\msys-gettextsrc-0-19-8-1.dll Hash Sha1" Hash="150453A5628E153D726E1745EE2568689A4773A9" />
    <Allow ID="ID_ALLOW_A_8A2" FriendlyName="C:\Program Files\git\usr\bin\msys-gettextsrc-0-19-8-1.dll Hash Sha256" Hash="34E675C2F36420AB7447052813127F32FF49BBB851B32B4335533873EFD64036" />
    <Allow ID="ID_ALLOW_A_8A3" FriendlyName="C:\Program Files\git\usr\bin\msys-gettextsrc-0-19-8-1.dll Hash Page Sha1" Hash="3F5D77253192C2B44CD1FD0D79DB52D8CFBFB252" />
    <Allow ID="ID_ALLOW_A_8A4" FriendlyName="C:\Program Files\git\usr\bin\msys-gettextsrc-0-19-8-1.dll Hash Page Sha256" Hash="932D87DFE4DD3DC177A2D6B287032F1B086176B5E62CB43D241F7C9446B207D5" />
    <Allow ID="ID_ALLOW_A_8A5" FriendlyName="C:\Program Files\git\usr\bin\msys-gio-2.0-0.dll Hash Sha1" Hash="D36D595A846096502A0C685E84A2258DD5A38BAF" />
    <Allow ID="ID_ALLOW_A_8A6" FriendlyName="C:\Program Files\git\usr\bin\msys-gio-2.0-0.dll Hash Sha256" Hash="89861148923C8F2192E6271A13AA12598BFDCDB12D85C7B63B2CD125E175690F" />
    <Allow ID="ID_ALLOW_A_8A7" FriendlyName="C:\Program Files\git\usr\bin\msys-gio-2.0-0.dll Hash Page Sha1" Hash="A57E42B36520B8AEA5ED9A1B7AD8C6FECDE02709" />
    <Allow ID="ID_ALLOW_A_8A8" FriendlyName="C:\Program Files\git\usr\bin\msys-gio-2.0-0.dll Hash Page Sha256" Hash="94141C7182AEC7AE7C3F1CA0AD3B56DDFF074A987D904C3321C312A12B8B5389" />
    <Allow ID="ID_ALLOW_A_8A9" FriendlyName="C:\Program Files\git\usr\bin\msys-glib-2.0-0.dll Hash Sha1" Hash="66C753DD56743ED9113CE287ED38204B174D9458" />
    <Allow ID="ID_ALLOW_A_8AA" FriendlyName="C:\Program Files\git\usr\bin\msys-glib-2.0-0.dll Hash Sha256" Hash="F0B602BFAE027D734CD38BFD9C8CD950982D047E78FD261B2CDC15CE0F53B7DB" />
    <Allow ID="ID_ALLOW_A_8AB" FriendlyName="C:\Program Files\git\usr\bin\msys-glib-2.0-0.dll Hash Page Sha1" Hash="C0E767BFD02BFA03B5E9A1005701533ACDAF2275" />
    <Allow ID="ID_ALLOW_A_8AC" FriendlyName="C:\Program Files\git\usr\bin\msys-glib-2.0-0.dll Hash Page Sha256" Hash="D0BC9118404FC992B4DE4BE9CF8DC921AD0803181335612F06C70B653C77827B" />
    <Allow ID="ID_ALLOW_A_8AD" FriendlyName="C:\Program Files\git\usr\bin\msys-gmodule-2.0-0.dll Hash Sha1" Hash="174EAD59181EC65B8FA988218CD0E1CC6568C63F" />
    <Allow ID="ID_ALLOW_A_8AE" FriendlyName="C:\Program Files\git\usr\bin\msys-gmodule-2.0-0.dll Hash Sha256" Hash="74CE757B93E04DA9594DA4C9A5FD85D39A07B015718B72F5CDA354EB47655B5F" />
    <Allow ID="ID_ALLOW_A_8AF" FriendlyName="C:\Program Files\git\usr\bin\msys-gmodule-2.0-0.dll Hash Page Sha1" Hash="AD4EE27CC9C5551C601AF290EA73BA7495D0D687" />
    <Allow ID="ID_ALLOW_A_8B0" FriendlyName="C:\Program Files\git\usr\bin\msys-gmodule-2.0-0.dll Hash Page Sha256" Hash="72213BF1742FB6DCB18FE4B118F586B9A659A2507ECE5C7B1B564FA1F795E385" />
    <Allow ID="ID_ALLOW_A_8B1" FriendlyName="C:\Program Files\git\usr\bin\msys-gmp-10.dll Hash Sha1" Hash="A778C41357BA6C254E033E1F80FAE296986D22CD" />
    <Allow ID="ID_ALLOW_A_8B2" FriendlyName="C:\Program Files\git\usr\bin\msys-gmp-10.dll Hash Sha256" Hash="1198DBCF5105F3A39B9D5544B832FFA1E041A7FE9AFCB3E2F3F469040B72A59C" />
    <Allow ID="ID_ALLOW_A_8B3" FriendlyName="C:\Program Files\git\usr\bin\msys-gmp-10.dll Hash Page Sha1" Hash="F7A05D9E61BDE0054306D5E43F8571BE4476CB21" />
    <Allow ID="ID_ALLOW_A_8B4" FriendlyName="C:\Program Files\git\usr\bin\msys-gmp-10.dll Hash Page Sha256" Hash="43904B4E69BA2810769494A35D6AABBB344C4871372A0FED06979D011A1B62AC" />
    <Allow ID="ID_ALLOW_A_8B5" FriendlyName="C:\Program Files\git\usr\bin\msys-gnutls-30.dll Hash Sha1" Hash="73E95FA1DB0E590330826B888C4C8151D16509B5" />
    <Allow ID="ID_ALLOW_A_8B6" FriendlyName="C:\Program Files\git\usr\bin\msys-gnutls-30.dll Hash Sha256" Hash="6A3CBC2DCB1D85E0704BCA145BA52AE8446B74871899B9A188789E0795CE40AA" />
    <Allow ID="ID_ALLOW_A_8B7" FriendlyName="C:\Program Files\git\usr\bin\msys-gnutls-30.dll Hash Page Sha1" Hash="75DCE959BF7462607014B7E5772F9258290DCBE1" />
    <Allow ID="ID_ALLOW_A_8B8" FriendlyName="C:\Program Files\git\usr\bin\msys-gnutls-30.dll Hash Page Sha256" Hash="348D4AA2BC3F0662AC6336E8B16685AEC45EEC0CA61F280FCE92AA0348F3F384" />
    <Allow ID="ID_ALLOW_A_8B9" FriendlyName="C:\Program Files\git\usr\bin\msys-gobject-2.0-0.dll Hash Sha1" Hash="B6342ADC122759B1217844F4E2537C79E41E7E60" />
    <Allow ID="ID_ALLOW_A_8BA" FriendlyName="C:\Program Files\git\usr\bin\msys-gobject-2.0-0.dll Hash Sha256" Hash="94B469F838C56EC9B102125850F88AE04977DD94587EC9EBE82696536DA59CE2" />
    <Allow ID="ID_ALLOW_A_8BB" FriendlyName="C:\Program Files\git\usr\bin\msys-gobject-2.0-0.dll Hash Page Sha1" Hash="2BE88C48745BAEE48F69F5CF74DA25C6E9913642" />
    <Allow ID="ID_ALLOW_A_8BC" FriendlyName="C:\Program Files\git\usr\bin\msys-gobject-2.0-0.dll Hash Page Sha256" Hash="4AD2BAD00303E7C5ACB973A37E435C24808B3D9027D74F99623C89A972B99880" />
    <Allow ID="ID_ALLOW_A_8BD" FriendlyName="C:\Program Files\git\usr\bin\msys-gpg-error-0.dll Hash Sha1" Hash="A9AEFEB3E36CE2495D0C5D6D21C52312F57B925C" />
    <Allow ID="ID_ALLOW_A_8BE" FriendlyName="C:\Program Files\git\usr\bin\msys-gpg-error-0.dll Hash Sha256" Hash="293E26E3FBF15E81D61E7D635DFC8389685084CE1C8C6BB30B935BEF96258C80" />
    <Allow ID="ID_ALLOW_A_8BF" FriendlyName="C:\Program Files\git\usr\bin\msys-gpg-error-0.dll Hash Page Sha1" Hash="789DFCB56555FC8ABE48EEC3C07C3CEC611665C0" />
    <Allow ID="ID_ALLOW_A_8C0" FriendlyName="C:\Program Files\git\usr\bin\msys-gpg-error-0.dll Hash Page Sha256" Hash="17CEB9D04CABF8376672C9C172BF4C936BE190026EBBDC12E3094A4B74D5EF71" />
    <Allow ID="ID_ALLOW_A_8C1" FriendlyName="C:\Program Files\git\usr\bin\msys-gssapi-3.dll Hash Sha1" Hash="E831BAD9B02B7CE77972C0636FE630D298B63B19" />
    <Allow ID="ID_ALLOW_A_8C2" FriendlyName="C:\Program Files\git\usr\bin\msys-gssapi-3.dll Hash Sha256" Hash="27620A0AD5AE2A4C86534E2877FD726690034A74DE18FD628227CFF453D46161" />
    <Allow ID="ID_ALLOW_A_8C3" FriendlyName="C:\Program Files\git\usr\bin\msys-gssapi-3.dll Hash Page Sha1" Hash="99169B704082CA92B37D6829BDD42E0DCB0D745E" />
    <Allow ID="ID_ALLOW_A_8C4" FriendlyName="C:\Program Files\git\usr\bin\msys-gssapi-3.dll Hash Page Sha256" Hash="4F32D8527576F11F26DA1C3D3C78E56B7514AA6B9B7CA3A6975183A5FADA4388" />
    <Allow ID="ID_ALLOW_A_8C5" FriendlyName="C:\Program Files\git\usr\bin\msys-hcrypto-4.dll Hash Sha1" Hash="5B57356BDD6868773C2E73FABD5DEC1785A2E7EE" />
    <Allow ID="ID_ALLOW_A_8C6" FriendlyName="C:\Program Files\git\usr\bin\msys-hcrypto-4.dll Hash Sha256" Hash="520C1B1786B2330880DD123072EBD2BED5D5D873483CE8011EF7B01CB95DFBD1" />
    <Allow ID="ID_ALLOW_A_8C7" FriendlyName="C:\Program Files\git\usr\bin\msys-hcrypto-4.dll Hash Page Sha1" Hash="38219C9494B8A58CA67B1B305C2E4ED1E5D705B6" />
    <Allow ID="ID_ALLOW_A_8C8" FriendlyName="C:\Program Files\git\usr\bin\msys-hcrypto-4.dll Hash Page Sha256" Hash="EAE703CCE04F4C364649BFE2B7AB33244B694558A6F796CD3621293F3D84BD1D" />
    <Allow ID="ID_ALLOW_A_8C9" FriendlyName="C:\Program Files\git\usr\bin\msys-heimbase-1.dll Hash Sha1" Hash="7642E493AD38A871D7D5C0DD9881D66BCA693BEE" />
    <Allow ID="ID_ALLOW_A_8CA" FriendlyName="C:\Program Files\git\usr\bin\msys-heimbase-1.dll Hash Sha256" Hash="F5461EE7ADE1609714A1A4C1054B34CAE794F7FB1E5F790DD34728C6C760BB01" />
    <Allow ID="ID_ALLOW_A_8CB" FriendlyName="C:\Program Files\git\usr\bin\msys-heimbase-1.dll Hash Page Sha1" Hash="114FF7D65DCA8C3284577233D095F9FA41E88B34" />
    <Allow ID="ID_ALLOW_A_8CC" FriendlyName="C:\Program Files\git\usr\bin\msys-heimbase-1.dll Hash Page Sha256" Hash="D0B19B057C18CDA34D0EDD964AF8BEEF8C30861AA8FF010E079A5989825F8D42" />
    <Allow ID="ID_ALLOW_A_8CD" FriendlyName="C:\Program Files\git\usr\bin\msys-heimntlm-0.dll Hash Sha1" Hash="88194ECD45875CF4B27971919C20F990AB338747" />
    <Allow ID="ID_ALLOW_A_8CE" FriendlyName="C:\Program Files\git\usr\bin\msys-heimntlm-0.dll Hash Sha256" Hash="B894AC3AC71DA032E9A492D5913C474B2A950C984EB70611CF5A711E7B729277" />
    <Allow ID="ID_ALLOW_A_8CF" FriendlyName="C:\Program Files\git\usr\bin\msys-heimntlm-0.dll Hash Page Sha1" Hash="E64F94797DB3CF3BE8CA581D87C10277B17FF52D" />
    <Allow ID="ID_ALLOW_A_8D0" FriendlyName="C:\Program Files\git\usr\bin\msys-heimntlm-0.dll Hash Page Sha256" Hash="CEB8EC75C0E4DA3ADC9D5B8EAFC8C9C279D801B6EA92174710830B923CAFDC0D" />
    <Allow ID="ID_ALLOW_A_8D1" FriendlyName="C:\Program Files\git\usr\bin\msys-hogweed-6.dll Hash Sha1" Hash="BF5D4403F3FB534145E5EF7544DB857DB58E1CEC" />
    <Allow ID="ID_ALLOW_A_8D2" FriendlyName="C:\Program Files\git\usr\bin\msys-hogweed-6.dll Hash Sha256" Hash="B6DA68127AD8299C8ED74D731231459495CE94D572E6184AB207057B7E6BE3A3" />
    <Allow ID="ID_ALLOW_A_8D3" FriendlyName="C:\Program Files\git\usr\bin\msys-hogweed-6.dll Hash Page Sha1" Hash="E511464395E8C81ADEA4E173D6A137174C4C107B" />
    <Allow ID="ID_ALLOW_A_8D4" FriendlyName="C:\Program Files\git\usr\bin\msys-hogweed-6.dll Hash Page Sha256" Hash="822D1F8AFADFEEFE34903E2706984A4C2BF26F737C20C0D0C2C55D537BAE815A" />
    <Allow ID="ID_ALLOW_A_8D5" FriendlyName="C:\Program Files\git\usr\bin\msys-hx509-5.dll Hash Sha1" Hash="94912D58C3D19BA49569625C899D4C9D7DBEB212" />
    <Allow ID="ID_ALLOW_A_8D6" FriendlyName="C:\Program Files\git\usr\bin\msys-hx509-5.dll Hash Sha256" Hash="E83391CE812442AE374BEA35A3D2507669BFB1B73C013E5B38C59D8ABBB7F703" />
    <Allow ID="ID_ALLOW_A_8D7" FriendlyName="C:\Program Files\git\usr\bin\msys-hx509-5.dll Hash Page Sha1" Hash="DE323DA361331244AC090AA59F41E878BCF525B0" />
    <Allow ID="ID_ALLOW_A_8D8" FriendlyName="C:\Program Files\git\usr\bin\msys-hx509-5.dll Hash Page Sha256" Hash="11CD4060BA0BDA2FE1D1996AACD5165774BE99C05F70BD41E5AAE82ABA17F748" />
    <Allow ID="ID_ALLOW_A_8D9" FriendlyName="C:\Program Files\git\usr\bin\msys-iconv-2.dll Hash Sha1" Hash="7CA1B357E826EC30E0CB978BA94B43B9AD16103F" />
    <Allow ID="ID_ALLOW_A_8DA" FriendlyName="C:\Program Files\git\usr\bin\msys-iconv-2.dll Hash Sha256" Hash="BC5816BF4872156BB717CD213AF02FB9AC8B39CD912E05BF5BFB70C6DDB24B65" />
    <Allow ID="ID_ALLOW_A_8DB" FriendlyName="C:\Program Files\git\usr\bin\msys-iconv-2.dll Hash Page Sha1" Hash="16BCE2F628C52A7BAACEC48610EB6706EED4080D" />
    <Allow ID="ID_ALLOW_A_8DC" FriendlyName="C:\Program Files\git\usr\bin\msys-iconv-2.dll Hash Page Sha256" Hash="F83BFADC55A42069D594023AF3DACB9C70CCD800FE5179A5EB9497F0E10E0643" />
    <Allow ID="ID_ALLOW_A_8DD" FriendlyName="C:\Program Files\git\usr\bin\msys-idn2-0.dll Hash Sha1" Hash="8A635DBEF00D3CA96E61CBD1F460A2C78ABB958F" />
    <Allow ID="ID_ALLOW_A_8DE" FriendlyName="C:\Program Files\git\usr\bin\msys-idn2-0.dll Hash Sha256" Hash="A38EB140035BBE48F4AC41A294B62750394F57E83C2E1507FA151BE72ABEA590" />
    <Allow ID="ID_ALLOW_A_8DF" FriendlyName="C:\Program Files\git\usr\bin\msys-idn2-0.dll Hash Page Sha1" Hash="9EA2ABC438C991A55F3AC3FA67382D6961E1D666" />
    <Allow ID="ID_ALLOW_A_8E0" FriendlyName="C:\Program Files\git\usr\bin\msys-idn2-0.dll Hash Page Sha256" Hash="0A059CFD0D9DB89B40971871421E64B513C6818271FFC5C553FD86110CEC9FCC" />
    <Allow ID="ID_ALLOW_A_8E1" FriendlyName="C:\Program Files\git\usr\bin\msys-intl-8.dll Hash Sha1" Hash="D052A9888605394379ABDC9054C51B73EF80722A" />
    <Allow ID="ID_ALLOW_A_8E2" FriendlyName="C:\Program Files\git\usr\bin\msys-intl-8.dll Hash Sha256" Hash="2DE7BBA6AA250A13F908A189ACCC90F7A938495434CC8CEDCCBDAD3E950CCB4E" />
    <Allow ID="ID_ALLOW_A_8E3" FriendlyName="C:\Program Files\git\usr\bin\msys-intl-8.dll Hash Page Sha1" Hash="196AD840C93515B66B53822A94B1722723A4EB4A" />
    <Allow ID="ID_ALLOW_A_8E4" FriendlyName="C:\Program Files\git\usr\bin\msys-intl-8.dll Hash Page Sha256" Hash="09905F40973EA691AE98E1A0D6811BA1FDB8C9A3DA68465F5B57ECC4DFAABCEC" />
    <Allow ID="ID_ALLOW_A_8E5" FriendlyName="C:\Program Files\git\usr\bin\msys-kafs-0.dll Hash Sha1" Hash="73F2BA293116A3648A7E1C47F521B9724A6BA72B" />
    <Allow ID="ID_ALLOW_A_8E6" FriendlyName="C:\Program Files\git\usr\bin\msys-kafs-0.dll Hash Sha256" Hash="CABEBB91009B9007B24C85428828DCAED6660B74158B83AB64B9538F6389C8DF" />
    <Allow ID="ID_ALLOW_A_8E7" FriendlyName="C:\Program Files\git\usr\bin\msys-kafs-0.dll Hash Page Sha1" Hash="BD29D12C26CBE237C42C3765A00629A30C03CD2D" />
    <Allow ID="ID_ALLOW_A_8E8" FriendlyName="C:\Program Files\git\usr\bin\msys-kafs-0.dll Hash Page Sha256" Hash="D2AFE90B1AADAB5F5928F8DB976EE274F6A9C14A341B943AF798914F649CF851" />
    <Allow ID="ID_ALLOW_A_8E9" FriendlyName="C:\Program Files\git\usr\bin\msys-krb5-26.dll Hash Sha1" Hash="C9DF7F2B813C2B8E60B8AD85D50B1E78E4D1D4B7" />
    <Allow ID="ID_ALLOW_A_8EA" FriendlyName="C:\Program Files\git\usr\bin\msys-krb5-26.dll Hash Sha256" Hash="74B66C6D9BCF6AEEE46E92BE9FAB3E7F3B48B3463203C3F6C578F806FC709862" />
    <Allow ID="ID_ALLOW_A_8EB" FriendlyName="C:\Program Files\git\usr\bin\msys-krb5-26.dll Hash Page Sha1" Hash="4AD2BAC7B4BEC497EC2292F1D66C80CCD0B32DF0" />
    <Allow ID="ID_ALLOW_A_8EC" FriendlyName="C:\Program Files\git\usr\bin\msys-krb5-26.dll Hash Page Sha256" Hash="A2C670C4F8C01B093A894048F0A69CF323C994468DE85BB77AC3C2129E978BAF" />
    <Allow ID="ID_ALLOW_A_8ED" FriendlyName="C:\Program Files\git\usr\bin\msys-ksba-8.dll Hash Sha1" Hash="889C195E6C6C34805D27915B3B12CA1B35053A9C" />
    <Allow ID="ID_ALLOW_A_8EE" FriendlyName="C:\Program Files\git\usr\bin\msys-ksba-8.dll Hash Sha256" Hash="CA55B3EB4255EBB6615FA8BDDADE514CFCE074B73B95478E06DCAB03E5FC51D9" />
    <Allow ID="ID_ALLOW_A_8EF" FriendlyName="C:\Program Files\git\usr\bin\msys-ksba-8.dll Hash Page Sha1" Hash="79090BF5B4A8FB31DFE72F98FE8EB352E53F4B58" />
    <Allow ID="ID_ALLOW_A_8F0" FriendlyName="C:\Program Files\git\usr\bin\msys-ksba-8.dll Hash Page Sha256" Hash="1CF7C6A7EF2C01194AC86B1BF42829733EB917EDC22997257EC66B043D89C03A" />
    <Allow ID="ID_ALLOW_A_8F1" FriendlyName="C:\Program Files\git\usr\bin\msys-lz4-1.dll Hash Sha1" Hash="395FE828FB43AF554DC10ABC6BFD0E9233DC5011" />
    <Allow ID="ID_ALLOW_A_8F2" FriendlyName="C:\Program Files\git\usr\bin\msys-lz4-1.dll Hash Sha256" Hash="F1305083BD090AFADCB79C1998CA37D1231055FF2C9CA47A16C9D36CED457749" />
    <Allow ID="ID_ALLOW_A_8F3" FriendlyName="C:\Program Files\git\usr\bin\msys-lz4-1.dll Hash Page Sha1" Hash="F79212D90C3E3C7915769A9B53C578A74F9CC845" />
    <Allow ID="ID_ALLOW_A_8F4" FriendlyName="C:\Program Files\git\usr\bin\msys-lz4-1.dll Hash Page Sha256" Hash="BA195279972C950204B09D05D97E43A321D44CF3CEB08328A0D6715709243501" />
    <Allow ID="ID_ALLOW_A_8F5" FriendlyName="C:\Program Files\git\usr\bin\msys-magic-1.dll Hash Sha1" Hash="7B4FDB489943C9D35811E623D6EC803C7D9DEAED" />
    <Allow ID="ID_ALLOW_A_8F6" FriendlyName="C:\Program Files\git\usr\bin\msys-magic-1.dll Hash Sha256" Hash="225CD24B3F580C4BFFCD569C948BBA2104EA6159299C9039C6E5E554553A40AE" />
    <Allow ID="ID_ALLOW_A_8F7" FriendlyName="C:\Program Files\git\usr\bin\msys-magic-1.dll Hash Page Sha1" Hash="B21D85D33D36F78978EB96B14805B5DD4AEEE7A7" />
    <Allow ID="ID_ALLOW_A_8F8" FriendlyName="C:\Program Files\git\usr\bin\msys-magic-1.dll Hash Page Sha256" Hash="5BB84EAE9EB94CE79C60545D78AA15759A9339D7700B5081C03C46B1A6865AE3" />
    <Allow ID="ID_ALLOW_A_8F9" FriendlyName="C:\Program Files\git\usr\bin\msys-mpfr-6.dll Hash Sha1" Hash="919CA94D23C4C8EA6F1D51F5A98F4AB664CB4C19" />
    <Allow ID="ID_ALLOW_A_8FA" FriendlyName="C:\Program Files\git\usr\bin\msys-mpfr-6.dll Hash Sha256" Hash="CFEA6F5CE4542DF81BC8F2C841F46C44BB2D41219EF778959E8C89196ADAF32F" />
    <Allow ID="ID_ALLOW_A_8FB" FriendlyName="C:\Program Files\git\usr\bin\msys-mpfr-6.dll Hash Page Sha1" Hash="7F0D9896E1D57B41B4195BF8D6A36BA38F6FA264" />
    <Allow ID="ID_ALLOW_A_8FC" FriendlyName="C:\Program Files\git\usr\bin\msys-mpfr-6.dll Hash Page Sha256" Hash="9D442F44E4AFE0937CB96096962D1388B6655D014E22CF0F9397B8F0E4952C0B" />
    <Allow ID="ID_ALLOW_A_8FD" FriendlyName="C:\Program Files\git\usr\bin\msys-ncursesw6.dll Hash Sha1" Hash="99C1B47A4DFA00D05A01A0FB28D6CC941F30DEE9" />
    <Allow ID="ID_ALLOW_A_8FE" FriendlyName="C:\Program Files\git\usr\bin\msys-ncursesw6.dll Hash Sha256" Hash="65518880E3434A5466BC8843D088D6EEC079C20D8E81F12D14469DFB3EC2B1FA" />
    <Allow ID="ID_ALLOW_A_8FF" FriendlyName="C:\Program Files\git\usr\bin\msys-ncursesw6.dll Hash Page Sha1" Hash="5EF2543BFBF70D551BCA3FFD215CB8309A1D1CE8" />
    <Allow ID="ID_ALLOW_A_900" FriendlyName="C:\Program Files\git\usr\bin\msys-ncursesw6.dll Hash Page Sha256" Hash="CBC0A7FBB759DCF2F09340F06E5899EE1E260DD9F68444675AC93D860FD72B43" />
    <Allow ID="ID_ALLOW_A_901" FriendlyName="C:\Program Files\git\usr\bin\msys-nettle-8.dll Hash Sha1" Hash="7679D67BC17FC460010207CD3839852A36998DDF" />
    <Allow ID="ID_ALLOW_A_902" FriendlyName="C:\Program Files\git\usr\bin\msys-nettle-8.dll Hash Sha256" Hash="68312FEE5182B515F8DD466E406A2ADACDA5565C41961F06217BD673646BCDD2" />
    <Allow ID="ID_ALLOW_A_903" FriendlyName="C:\Program Files\git\usr\bin\msys-nettle-8.dll Hash Page Sha1" Hash="D793C3D9CA7DF8872B58F8862AEDB90460F732C0" />
    <Allow ID="ID_ALLOW_A_904" FriendlyName="C:\Program Files\git\usr\bin\msys-nettle-8.dll Hash Page Sha256" Hash="D91C969578C4F1DDC265C27378CA8AF7C3FDB80F573EA32629F72A6AFD5D587C" />
    <Allow ID="ID_ALLOW_A_905" FriendlyName="C:\Program Files\git\usr\bin\msys-npth-0.dll Hash Sha1" Hash="5C283003E4B03B631E17913DE47BD062C4643C7E" />
    <Allow ID="ID_ALLOW_A_906" FriendlyName="C:\Program Files\git\usr\bin\msys-npth-0.dll Hash Sha256" Hash="189EB4AB4F3B732CF4AB14EB4FF39326B56A585188CECB03E7D03BE2E2D07DBF" />
    <Allow ID="ID_ALLOW_A_907" FriendlyName="C:\Program Files\git\usr\bin\msys-npth-0.dll Hash Page Sha1" Hash="95B43D695CF12B0F242B697778D61A5083AF4E0D" />
    <Allow ID="ID_ALLOW_A_908" FriendlyName="C:\Program Files\git\usr\bin\msys-npth-0.dll Hash Page Sha256" Hash="40FBAA7E1D961189074072BAD30B5943FFECC2875036910072DD4E42153E282E" />
    <Allow ID="ID_ALLOW_A_909" FriendlyName="C:\Program Files\git\usr\bin\msys-p11-kit-0.dll Hash Sha1" Hash="3289093902DF0CC7641478985F278EA3BB70CE95" />
    <Allow ID="ID_ALLOW_A_90A" FriendlyName="C:\Program Files\git\usr\bin\msys-p11-kit-0.dll Hash Sha256" Hash="3D38083D27C626A336FC47304B4F29A669FBA658500A29A8A36DA23A37852B3E" />
    <Allow ID="ID_ALLOW_A_90B" FriendlyName="C:\Program Files\git\usr\bin\msys-p11-kit-0.dll Hash Page Sha1" Hash="1CDC1F37BA8C73B72D90D7DE743892FBA9FC76D2" />
    <Allow ID="ID_ALLOW_A_90C" FriendlyName="C:\Program Files\git\usr\bin\msys-p11-kit-0.dll Hash Page Sha256" Hash="E0C9BF7F55ABDA67B1A37D39B250B4ED4B38127F627D4C1541D006E5B32AF5C0" />
    <Allow ID="ID_ALLOW_A_90D" FriendlyName="C:\Program Files\git\usr\bin\msys-pcre-1.dll Hash Sha1" Hash="7C65A9F064BF8D467E8624D8219898D71B0CEF02" />
    <Allow ID="ID_ALLOW_A_90E" FriendlyName="C:\Program Files\git\usr\bin\msys-pcre-1.dll Hash Sha256" Hash="70EC10DE84EE24B504B7B4A04DBC98A997E238AF582AA62F621E5AF0C708060D" />
    <Allow ID="ID_ALLOW_A_90F" FriendlyName="C:\Program Files\git\usr\bin\msys-pcre-1.dll Hash Page Sha1" Hash="CEABC6C0CDBF6E3EAF0CE84109F203E9DE645BE3" />
    <Allow ID="ID_ALLOW_A_910" FriendlyName="C:\Program Files\git\usr\bin\msys-pcre-1.dll Hash Page Sha256" Hash="8F85ED6AB2E46FAA9EAA857858159ABEC9CD0450AB4AB372EF666686A250149F" />
    <Allow ID="ID_ALLOW_A_911" FriendlyName="C:\Program Files\git\usr\bin\msys-psl-5.dll Hash Sha1" Hash="4DAD38E0176612097F692341E3E7E75CCE691437" />
    <Allow ID="ID_ALLOW_A_912" FriendlyName="C:\Program Files\git\usr\bin\msys-psl-5.dll Hash Sha256" Hash="E87F64AB11036334FEBC8F2FE6C6895A3C4E12393F7BB28C751A721099F23DB1" />
    <Allow ID="ID_ALLOW_A_913" FriendlyName="C:\Program Files\git\usr\bin\msys-psl-5.dll Hash Page Sha1" Hash="F16352F6B4608D57A3EACD60C4D9610A6BE93F8B" />
    <Allow ID="ID_ALLOW_A_914" FriendlyName="C:\Program Files\git\usr\bin\msys-psl-5.dll Hash Page Sha256" Hash="B9451ED6E4AC4C25484EF628DC25BFDAA3DFD07635BB605EDB3B00FB3650DDD2" />
    <Allow ID="ID_ALLOW_A_915" FriendlyName="C:\Program Files\git\usr\bin\msys-readline8.dll Hash Sha1" Hash="72D83292C8487853F20845D886F0761F13D782BC" />
    <Allow ID="ID_ALLOW_A_916" FriendlyName="C:\Program Files\git\usr\bin\msys-readline8.dll Hash Sha256" Hash="7BC017CC0D2F050A1397088051449C41AE9E702455A44C1782A2D370553551C6" />
    <Allow ID="ID_ALLOW_A_917" FriendlyName="C:\Program Files\git\usr\bin\msys-readline8.dll Hash Page Sha1" Hash="14C1E78C356824521DB9CCB0CF0060BF06B075A4" />
    <Allow ID="ID_ALLOW_A_918" FriendlyName="C:\Program Files\git\usr\bin\msys-readline8.dll Hash Page Sha256" Hash="8E38E11AAFD2478EBF9A2EE165D3EF9CA9C325D41E07B7B5A52FD42AF0D0924F" />
    <Allow ID="ID_ALLOW_A_919" FriendlyName="C:\Program Files\git\usr\bin\msys-roken-18.dll Hash Sha1" Hash="A0A72DF1938BECF5E587EE1C0F40B08AD9A1EEF6" />
    <Allow ID="ID_ALLOW_A_91A" FriendlyName="C:\Program Files\git\usr\bin\msys-roken-18.dll Hash Sha256" Hash="EA53751246E500126F212261CFAEAFC850E7F2C964EC72CA64A8478AE46530FD" />
    <Allow ID="ID_ALLOW_A_91B" FriendlyName="C:\Program Files\git\usr\bin\msys-roken-18.dll Hash Page Sha1" Hash="66939E21A12F54F1A2C69179E1306E3508CF6863" />
    <Allow ID="ID_ALLOW_A_91C" FriendlyName="C:\Program Files\git\usr\bin\msys-roken-18.dll Hash Page Sha256" Hash="3D4654020D4EBC6CE65869E37691D3A5E24B322B7E2893FCAAACD6BD94D376CC" />
    <Allow ID="ID_ALLOW_A_91D" FriendlyName="C:\Program Files\git\usr\bin\msys-sasl2-3.dll Hash Sha1" Hash="10DA7E04992A60F850C27174745A20BB2457F4FD" />
    <Allow ID="ID_ALLOW_A_91E" FriendlyName="C:\Program Files\git\usr\bin\msys-sasl2-3.dll Hash Sha256" Hash="43CFC9A492BDC5134DDA138E919CAF14D04A99475BD08D848FD4330A3FBB6D31" />
    <Allow ID="ID_ALLOW_A_91F" FriendlyName="C:\Program Files\git\usr\bin\msys-sasl2-3.dll Hash Page Sha1" Hash="1056B0634BD0C378E788BACA898847DDB719C9F9" />
    <Allow ID="ID_ALLOW_A_920" FriendlyName="C:\Program Files\git\usr\bin\msys-sasl2-3.dll Hash Page Sha256" Hash="26C707FB0C68D327B55B0DBA36BC1494FC1BAFDCCC72FEB26FBDC35419A39515" />
    <Allow ID="ID_ALLOW_A_921" FriendlyName="C:\Program Files\git\usr\bin\msys-serf-1-0.dll Hash Sha1" Hash="BAAF62117B0A063B558F55CDA1D59068B64BE7A5" />
    <Allow ID="ID_ALLOW_A_922" FriendlyName="C:\Program Files\git\usr\bin\msys-serf-1-0.dll Hash Sha256" Hash="845BD9117FCABB033A58033131219A999A21632824AACA68B70679EC06C83053" />
    <Allow ID="ID_ALLOW_A_923" FriendlyName="C:\Program Files\git\usr\bin\msys-serf-1-0.dll Hash Page Sha1" Hash="D3939D7FC3CBBC957987C8F7D7E588EA4E35D3A9" />
    <Allow ID="ID_ALLOW_A_924" FriendlyName="C:\Program Files\git\usr\bin\msys-serf-1-0.dll Hash Page Sha256" Hash="D70F08C7D55F367C50BF36DAAC899E7F042D2A70972125DEE63B8BF485807E6E" />
    <Allow ID="ID_ALLOW_A_925" FriendlyName="C:\Program Files\git\usr\bin\msys-smartcols-1.dll Hash Sha1" Hash="F52E459C203FFCDA9F7CB6410E301B0D50CBDEA5" />
    <Allow ID="ID_ALLOW_A_926" FriendlyName="C:\Program Files\git\usr\bin\msys-smartcols-1.dll Hash Sha256" Hash="922E5CFB78D50555076A74C45053A964802A7A8D0B34F1D8FD7F0592A350D9C3" />
    <Allow ID="ID_ALLOW_A_927" FriendlyName="C:\Program Files\git\usr\bin\msys-smartcols-1.dll Hash Page Sha1" Hash="D13D383572DAA1BB218916F7C542A4923837579E" />
    <Allow ID="ID_ALLOW_A_928" FriendlyName="C:\Program Files\git\usr\bin\msys-smartcols-1.dll Hash Page Sha256" Hash="BF6F89E094C0557CD651F2FB01425418A604970622E43508F27AB825C3CD66B5" />
    <Allow ID="ID_ALLOW_A_929" FriendlyName="C:\Program Files\git\usr\bin\msys-sqlite3-0.dll Hash Sha1" Hash="B7CBD11FDDC6A329CAE834A72CD739C3121DC975" />
    <Allow ID="ID_ALLOW_A_92A" FriendlyName="C:\Program Files\git\usr\bin\msys-sqlite3-0.dll Hash Sha256" Hash="4CA4C4E86A825C545BF4DDF4E49A07F358574B031E4786EB2EBA502CDF8B16AA" />
    <Allow ID="ID_ALLOW_A_92B" FriendlyName="C:\Program Files\git\usr\bin\msys-sqlite3-0.dll Hash Page Sha1" Hash="0EE18E6F6CA2AD969EA039F8D405A134DF768FFC" />
    <Allow ID="ID_ALLOW_A_92C" FriendlyName="C:\Program Files\git\usr\bin\msys-sqlite3-0.dll Hash Page Sha256" Hash="3D35D1C5B7F817FC5976E424616B797E14470F555B5F888B443BED8506FC51F1" />
    <Allow ID="ID_ALLOW_A_92D" FriendlyName="C:\Program Files\git\usr\bin\msys-ssl-1.1.dll Hash Sha1" Hash="230B6333FC292194FE3448C5D112CC17CA183607" />
    <Allow ID="ID_ALLOW_A_92E" FriendlyName="C:\Program Files\git\usr\bin\msys-ssl-1.1.dll Hash Sha256" Hash="BC412266DB184D7FBDF5AFA9F0222A30E3DCEB6B99DFF4B2D3886EC5A057C7C4" />
    <Allow ID="ID_ALLOW_A_92F" FriendlyName="C:\Program Files\git\usr\bin\msys-ssl-1.1.dll Hash Page Sha1" Hash="B6DE604408D05FC7F8E6BB1E602A7544ECFB2534" />
    <Allow ID="ID_ALLOW_A_930" FriendlyName="C:\Program Files\git\usr\bin\msys-ssl-1.1.dll Hash Page Sha256" Hash="190785D73E88349CE3A6C332269F3F0764DC6196ECB0DFFAB1EDAEEBD8DF67BA" />
    <Allow ID="ID_ALLOW_A_931" FriendlyName="C:\Program Files\git\usr\bin\msys-svn_client-1-0.dll Hash Sha1" Hash="19D1084BC3D8783109D59A18D5FBEA99906D1655" />
    <Allow ID="ID_ALLOW_A_932" FriendlyName="C:\Program Files\git\usr\bin\msys-svn_client-1-0.dll Hash Sha256" Hash="F65CFEA8629A615689BB262D0DAFF43BB0F5087A010546C01FF9FE8A45F739DE" />
    <Allow ID="ID_ALLOW_A_933" FriendlyName="C:\Program Files\git\usr\bin\msys-svn_client-1-0.dll Hash Page Sha1" Hash="FA0F9080A3F9CA56DBEC1F3EE947AA2B716592A8" />
    <Allow ID="ID_ALLOW_A_934" FriendlyName="C:\Program Files\git\usr\bin\msys-svn_client-1-0.dll Hash Page Sha256" Hash="C0B1CE5AE847C642C0EE4AC7152587A9B61B75A633F640342EB2F5A74F4013F0" />
    <Allow ID="ID_ALLOW_A_935" FriendlyName="C:\Program Files\git\usr\bin\msys-svn_delta-1-0.dll Hash Sha1" Hash="EDAFD38416F2D2A6417C43B8D66DDE9C985E3BCE" />
    <Allow ID="ID_ALLOW_A_936" FriendlyName="C:\Program Files\git\usr\bin\msys-svn_delta-1-0.dll Hash Sha256" Hash="65C791E330B1013BF5461D125E7EAD100D0E7BC8B21C2BF3D66C0774EADF3D33" />
    <Allow ID="ID_ALLOW_A_937" FriendlyName="C:\Program Files\git\usr\bin\msys-svn_delta-1-0.dll Hash Page Sha1" Hash="D7AE7FC6F3DD339FAC5822266071871A800CFDC0" />
    <Allow ID="ID_ALLOW_A_938" FriendlyName="C:\Program Files\git\usr\bin\msys-svn_delta-1-0.dll Hash Page Sha256" Hash="8C2A9AC696DF219315A09A33903452E7DFBE7DA2CAFC056CB1DD8B06F34B6B18" />
    <Allow ID="ID_ALLOW_A_939" FriendlyName="C:\Program Files\git\usr\bin\msys-svn_diff-1-0.dll Hash Sha1" Hash="5A1AB4F0FC3E3FA46B39DAF170EE2950FC9142B5" />
    <Allow ID="ID_ALLOW_A_93A" FriendlyName="C:\Program Files\git\usr\bin\msys-svn_diff-1-0.dll Hash Sha256" Hash="904371FF780979F7F7B6303F6351DFAC201D371790825E7D075EC8AFD6ED8607" />
    <Allow ID="ID_ALLOW_A_93B" FriendlyName="C:\Program Files\git\usr\bin\msys-svn_diff-1-0.dll Hash Page Sha1" Hash="49E42479A6E5A48E96796C4422A9875FC927D027" />
    <Allow ID="ID_ALLOW_A_93C" FriendlyName="C:\Program Files\git\usr\bin\msys-svn_diff-1-0.dll Hash Page Sha256" Hash="34D7BB1C53B2114E0343DF7329EB6A4FDC1987B354789935479F20C5DB59AB56" />
    <Allow ID="ID_ALLOW_A_93D" FriendlyName="C:\Program Files\git\usr\bin\msys-svn_fs-1-0.dll Hash Sha1" Hash="4A8EA056A9378FDF8A8C04E35036400FC32A2BA8" />
    <Allow ID="ID_ALLOW_A_93E" FriendlyName="C:\Program Files\git\usr\bin\msys-svn_fs-1-0.dll Hash Sha256" Hash="EA3F7ED644CCB1C2ACDE44582988D38598600A1EC390BEBF895D85768552F817" />
    <Allow ID="ID_ALLOW_A_93F" FriendlyName="C:\Program Files\git\usr\bin\msys-svn_fs-1-0.dll Hash Page Sha1" Hash="93E3C4251F3587FA639E1CD6D86F4DADF41CE406" />
    <Allow ID="ID_ALLOW_A_940" FriendlyName="C:\Program Files\git\usr\bin\msys-svn_fs-1-0.dll Hash Page Sha256" Hash="5051619DAE6D01BFC0C52AAF265CC9C23EF25FE0C6105C270B02A44D1D7D1B22" />
    <Allow ID="ID_ALLOW_A_941" FriendlyName="C:\Program Files\git\usr\bin\msys-svn_fs_fs-1-0.dll Hash Sha1" Hash="FA16CD301002F7A713D8B9A684A42267537868D3" />
    <Allow ID="ID_ALLOW_A_942" FriendlyName="C:\Program Files\git\usr\bin\msys-svn_fs_fs-1-0.dll Hash Sha256" Hash="CCD89FEB8946A577DDCCA6EA23709638D2D61CD785E38B9B2E604CA7398869E5" />
    <Allow ID="ID_ALLOW_A_943" FriendlyName="C:\Program Files\git\usr\bin\msys-svn_fs_fs-1-0.dll Hash Page Sha1" Hash="2BDA97C2D06C15884B24372C108283CF5E701B16" />
    <Allow ID="ID_ALLOW_A_944" FriendlyName="C:\Program Files\git\usr\bin\msys-svn_fs_fs-1-0.dll Hash Page Sha256" Hash="7C91E9BC5C700D5F2D9370FE0EF2137DB27E3A16B8C2B5EB79767EC85F546AFD" />
    <Allow ID="ID_ALLOW_A_945" FriendlyName="C:\Program Files\git\usr\bin\msys-svn_fs_util-1-0.dll Hash Sha1" Hash="747403341C6465888DFDFFBF52C83F7A69EB6E0E" />
    <Allow ID="ID_ALLOW_A_946" FriendlyName="C:\Program Files\git\usr\bin\msys-svn_fs_util-1-0.dll Hash Sha256" Hash="C554B68F7C03C24835D66D808EC94300BD5080DA6D374846986E6A638264A187" />
    <Allow ID="ID_ALLOW_A_947" FriendlyName="C:\Program Files\git\usr\bin\msys-svn_fs_util-1-0.dll Hash Page Sha1" Hash="43F3A92D71625A29F7B3E2666AE6B24A508EB953" />
    <Allow ID="ID_ALLOW_A_948" FriendlyName="C:\Program Files\git\usr\bin\msys-svn_fs_util-1-0.dll Hash Page Sha256" Hash="79D911135DB2FA486C6752A8442032B0F7529E2397196099E67EBB0B63021573" />
    <Allow ID="ID_ALLOW_A_949" FriendlyName="C:\Program Files\git\usr\bin\msys-svn_fs_x-1-0.dll Hash Sha1" Hash="1369633BC2B1666AEF8C1650C3DDC57B09CF4D82" />
    <Allow ID="ID_ALLOW_A_94A" FriendlyName="C:\Program Files\git\usr\bin\msys-svn_fs_x-1-0.dll Hash Sha256" Hash="14FA2AC3E741D3E8A06592167F6F05344DDD8E5A13297002E15608838EF98528" />
    <Allow ID="ID_ALLOW_A_94B" FriendlyName="C:\Program Files\git\usr\bin\msys-svn_fs_x-1-0.dll Hash Page Sha1" Hash="2C2F6A486204D258C5739790A929DEF12134E72C" />
    <Allow ID="ID_ALLOW_A_94C" FriendlyName="C:\Program Files\git\usr\bin\msys-svn_fs_x-1-0.dll Hash Page Sha256" Hash="EEF3DFDEACF1D4BD52CBC7ADDFCCDC48984153F22C40012CE0FBE1334CD70129" />
    <Allow ID="ID_ALLOW_A_94D" FriendlyName="C:\Program Files\git\usr\bin\msys-svn_ra-1-0.dll Hash Sha1" Hash="D7AAF356A20AB98A72D28BFF0CF6AB4DBC2367F3" />
    <Allow ID="ID_ALLOW_A_94E" FriendlyName="C:\Program Files\git\usr\bin\msys-svn_ra-1-0.dll Hash Sha256" Hash="3E078EE14D98CD1B5F3595CE43198EF21C28AE159859503133C34EE0A5B08D40" />
    <Allow ID="ID_ALLOW_A_94F" FriendlyName="C:\Program Files\git\usr\bin\msys-svn_ra-1-0.dll Hash Page Sha1" Hash="876E0B29C61753C22539987D6245375EB9E18B58" />
    <Allow ID="ID_ALLOW_A_950" FriendlyName="C:\Program Files\git\usr\bin\msys-svn_ra-1-0.dll Hash Page Sha256" Hash="A1AC140039D14189CCAC4D0080AC39BAA6A599D59D86F9CB54F9642771DD7BEE" />
    <Allow ID="ID_ALLOW_A_951" FriendlyName="C:\Program Files\git\usr\bin\msys-svn_ra_local-1-0.dll Hash Sha1" Hash="CCC98F9CD3B55CE19682BCCEE82E9F6FA3B5622C" />
    <Allow ID="ID_ALLOW_A_952" FriendlyName="C:\Program Files\git\usr\bin\msys-svn_ra_local-1-0.dll Hash Sha256" Hash="9D50C59F2668F18ECD744547DE8EC57D5F4F053EB2197FF835268EAFC2AD2E2F" />
    <Allow ID="ID_ALLOW_A_953" FriendlyName="C:\Program Files\git\usr\bin\msys-svn_ra_local-1-0.dll Hash Page Sha1" Hash="0D49D06C8872B3DEDA8574F7A12C3BADFD3287EA" />
    <Allow ID="ID_ALLOW_A_954" FriendlyName="C:\Program Files\git\usr\bin\msys-svn_ra_local-1-0.dll Hash Page Sha256" Hash="2096233D80302356FBF72BE768C3B60AF8F39CF378FB149B79A0A5DE06AD0FBD" />
    <Allow ID="ID_ALLOW_A_955" FriendlyName="C:\Program Files\git\usr\bin\msys-svn_ra_serf-1-0.dll Hash Sha1" Hash="9F54C73312B8CB5FED9BAE38A056A11C70C9B8B2" />
    <Allow ID="ID_ALLOW_A_956" FriendlyName="C:\Program Files\git\usr\bin\msys-svn_ra_serf-1-0.dll Hash Sha256" Hash="685C402EC7539B8A2C99496DC362D61EBE2D442908FBB6E7F2F036C6A39A6107" />
    <Allow ID="ID_ALLOW_A_957" FriendlyName="C:\Program Files\git\usr\bin\msys-svn_ra_serf-1-0.dll Hash Page Sha1" Hash="711B01948F4264B6C6903D3A9A15556A848EDB87" />
    <Allow ID="ID_ALLOW_A_958" FriendlyName="C:\Program Files\git\usr\bin\msys-svn_ra_serf-1-0.dll Hash Page Sha256" Hash="A75A30D9AA08AE96E8929F6271FB01331B3CC3B370741083C2B57BF6696454D1" />
    <Allow ID="ID_ALLOW_A_959" FriendlyName="C:\Program Files\git\usr\bin\msys-svn_ra_svn-1-0.dll Hash Sha1" Hash="71AC6FB15DCF42E9042E62857D4846553DD68FA8" />
    <Allow ID="ID_ALLOW_A_95A" FriendlyName="C:\Program Files\git\usr\bin\msys-svn_ra_svn-1-0.dll Hash Sha256" Hash="B29CE2B314BF6C404F079AC84F0CDC8B1FBE6C869DF6A8F88568489483E6EA80" />
    <Allow ID="ID_ALLOW_A_95B" FriendlyName="C:\Program Files\git\usr\bin\msys-svn_ra_svn-1-0.dll Hash Page Sha1" Hash="90229D7CB8EB90BBDD350FB1BBDA5FF581DC6746" />
    <Allow ID="ID_ALLOW_A_95C" FriendlyName="C:\Program Files\git\usr\bin\msys-svn_ra_svn-1-0.dll Hash Page Sha256" Hash="BBB0553AE204E3AFC77FA62023C6B27CEB50BFB5811E1ADBB69B56F48DE76CF5" />
    <Allow ID="ID_ALLOW_A_95D" FriendlyName="C:\Program Files\git\usr\bin\msys-svn_repos-1-0.dll Hash Sha1" Hash="F2491D17BD80611FB348803F6B6EB1C1B84F04A7" />
    <Allow ID="ID_ALLOW_A_95E" FriendlyName="C:\Program Files\git\usr\bin\msys-svn_repos-1-0.dll Hash Sha256" Hash="51D97728B7A7C0BB201E6CFA0CFA44FEACB665615DB5D9689063D8164B8B1CDC" />
    <Allow ID="ID_ALLOW_A_95F" FriendlyName="C:\Program Files\git\usr\bin\msys-svn_repos-1-0.dll Hash Page Sha1" Hash="D6820D3A63864A9D96E35DD034420E3B445613FC" />
    <Allow ID="ID_ALLOW_A_960" FriendlyName="C:\Program Files\git\usr\bin\msys-svn_repos-1-0.dll Hash Page Sha256" Hash="53CD229F723C464563D4B249D2020A4733F982460424E0DFF016B8A0DE9E5ED9" />
    <Allow ID="ID_ALLOW_A_961" FriendlyName="C:\Program Files\git\usr\bin\msys-svn_subr-1-0.dll Hash Sha1" Hash="43624B265E859313174E670AC0D07A22A06A4D50" />
    <Allow ID="ID_ALLOW_A_962" FriendlyName="C:\Program Files\git\usr\bin\msys-svn_subr-1-0.dll Hash Sha256" Hash="6F793C7E93FFF8F1E8888B813028754F61B2821AD07E74AC2DE38CBB796CB254" />
    <Allow ID="ID_ALLOW_A_963" FriendlyName="C:\Program Files\git\usr\bin\msys-svn_subr-1-0.dll Hash Page Sha1" Hash="8D67DBECAD359EA52DF6F4B6EB59184D2068BD09" />
    <Allow ID="ID_ALLOW_A_964" FriendlyName="C:\Program Files\git\usr\bin\msys-svn_subr-1-0.dll Hash Page Sha256" Hash="96292F7543DBA8FD6C66EE696AEC16F08472DDA0CC26DCAF01BFD3811412618B" />
    <Allow ID="ID_ALLOW_A_965" FriendlyName="C:\Program Files\git\usr\bin\msys-svn_swig_perl-1-0.dll Hash Sha1" Hash="642E99F881967A374D3E22612123E6C301851932" />
    <Allow ID="ID_ALLOW_A_966" FriendlyName="C:\Program Files\git\usr\bin\msys-svn_swig_perl-1-0.dll Hash Sha256" Hash="2FA8068D8A100350E04C25CBAE5CADEE11FA261975340DF56C47D3ACC9880077" />
    <Allow ID="ID_ALLOW_A_967" FriendlyName="C:\Program Files\git\usr\bin\msys-svn_swig_perl-1-0.dll Hash Page Sha1" Hash="BBC6271C7A1FABCBDEF00A4C09F19969FCA36C58" />
    <Allow ID="ID_ALLOW_A_968" FriendlyName="C:\Program Files\git\usr\bin\msys-svn_swig_perl-1-0.dll Hash Page Sha256" Hash="DFF5A20C04B331E91FD50B21500D3A9DD2BA867D51480C6B8FB5D2E85326AD61" />
    <Allow ID="ID_ALLOW_A_969" FriendlyName="C:\Program Files\git\usr\bin\msys-svn_wc-1-0.dll Hash Sha1" Hash="8140A5B5FA0D0105BA0D1879503E763A664FA088" />
    <Allow ID="ID_ALLOW_A_96A" FriendlyName="C:\Program Files\git\usr\bin\msys-svn_wc-1-0.dll Hash Sha256" Hash="DCD702BD2F885652AB389806F90014BBBD08CCA5B3E19677B110D77C7754F9D6" />
    <Allow ID="ID_ALLOW_A_96B" FriendlyName="C:\Program Files\git\usr\bin\msys-svn_wc-1-0.dll Hash Page Sha1" Hash="6CCA6AD4EFBEE9BE247AD2C7DD3E78A87FD77209" />
    <Allow ID="ID_ALLOW_A_96C" FriendlyName="C:\Program Files\git\usr\bin\msys-svn_wc-1-0.dll Hash Page Sha256" Hash="2298ED8C5932C88CB8B7D50B81BEC764B92AE1F51E5DC5D362EA62F1DEBD7910" />
    <Allow ID="ID_ALLOW_A_96D" FriendlyName="C:\Program Files\git\usr\bin\msys-tasn1-6.dll Hash Sha1" Hash="5CE4ADEA000D3CA147D82EA5C7ABEC23122FD4EE" />
    <Allow ID="ID_ALLOW_A_96E" FriendlyName="C:\Program Files\git\usr\bin\msys-tasn1-6.dll Hash Sha256" Hash="36E8425838D54411BEF5630ABDE6107CA39A89058A23D9BF4F30730E6882AD31" />
    <Allow ID="ID_ALLOW_A_96F" FriendlyName="C:\Program Files\git\usr\bin\msys-tasn1-6.dll Hash Page Sha1" Hash="D15ACBE0C73DC826991D26152506D10010F0F008" />
    <Allow ID="ID_ALLOW_A_970" FriendlyName="C:\Program Files\git\usr\bin\msys-tasn1-6.dll Hash Page Sha256" Hash="32B687AC6885FF478BF2D8741C65C1397E01138A2B4B45C79BB695575DE28ECB" />
    <Allow ID="ID_ALLOW_A_971" FriendlyName="C:\Program Files\git\usr\bin\msys-ticw6.dll Hash Sha1" Hash="BFB1CBD9F04CF2E353E3FE3545743241A7FAF3FC" />
    <Allow ID="ID_ALLOW_A_972" FriendlyName="C:\Program Files\git\usr\bin\msys-ticw6.dll Hash Sha256" Hash="CA4D660E24ABB953811F7953C071CB46D0BD9E956C0F84A8A648BE4AB35A09B5" />
    <Allow ID="ID_ALLOW_A_973" FriendlyName="C:\Program Files\git\usr\bin\msys-ticw6.dll Hash Page Sha1" Hash="6C0A35462EFB147DA7FE911EDAC8C70EE37AC47A" />
    <Allow ID="ID_ALLOW_A_974" FriendlyName="C:\Program Files\git\usr\bin\msys-ticw6.dll Hash Page Sha256" Hash="C05D48843D678D8800B2341B095E6201874B163A27742B44FECFAA069F37B2F8" />
    <Allow ID="ID_ALLOW_A_975" FriendlyName="C:\Program Files\git\usr\bin\msys-unistring-2.dll Hash Sha1" Hash="42965C15998EF62B23E99E12A5EEBC987A1132C8" />
    <Allow ID="ID_ALLOW_A_976" FriendlyName="C:\Program Files\git\usr\bin\msys-unistring-2.dll Hash Sha256" Hash="4127B911769961674DA1B9C42EC02A34E6ACFD52342117770299F4A098476C4F" />
    <Allow ID="ID_ALLOW_A_977" FriendlyName="C:\Program Files\git\usr\bin\msys-unistring-2.dll Hash Page Sha1" Hash="E88CE6592366EE3B17DDF58D7A0D2B98B3349881" />
    <Allow ID="ID_ALLOW_A_978" FriendlyName="C:\Program Files\git\usr\bin\msys-unistring-2.dll Hash Page Sha256" Hash="FD1F8278FA52B1FACA91C8A6016BBB5E9CAFAE9CFB35DF1C6919CF03541B7569" />
    <Allow ID="ID_ALLOW_A_979" FriendlyName="C:\Program Files\git\usr\bin\msys-uuid-1.dll Hash Sha1" Hash="255E074DC6BD21CED9B54F0F2EE1B5C0B2597BE3" />
    <Allow ID="ID_ALLOW_A_97A" FriendlyName="C:\Program Files\git\usr\bin\msys-uuid-1.dll Hash Sha256" Hash="CF05E6127B5E120A546A94DE244DBD4111D65C85590C3F7EBB9FF25124012140" />
    <Allow ID="ID_ALLOW_A_97B" FriendlyName="C:\Program Files\git\usr\bin\msys-uuid-1.dll Hash Page Sha1" Hash="61D55F9679B3474B9FB2CE2616C96D1E98627675" />
    <Allow ID="ID_ALLOW_A_97C" FriendlyName="C:\Program Files\git\usr\bin\msys-uuid-1.dll Hash Page Sha256" Hash="264C5AD46B6710842F056EC92770BE939B52FD516281150B29465FE16B068F84" />
    <Allow ID="ID_ALLOW_A_97D" FriendlyName="C:\Program Files\git\usr\bin\msys-wind-0.dll Hash Sha1" Hash="909FFBC926376B90A29C5A6063DFEB16E9EFBECF" />
    <Allow ID="ID_ALLOW_A_97E" FriendlyName="C:\Program Files\git\usr\bin\msys-wind-0.dll Hash Sha256" Hash="C1E3B378C74B7BE574C4DB34306473F6A169E6E5F56E474CD11D8219A4D03358" />
    <Allow ID="ID_ALLOW_A_97F" FriendlyName="C:\Program Files\git\usr\bin\msys-wind-0.dll Hash Page Sha1" Hash="C80861C3C78B4A4CB1BA5E9B79C1E142E674AB97" />
    <Allow ID="ID_ALLOW_A_980" FriendlyName="C:\Program Files\git\usr\bin\msys-wind-0.dll Hash Page Sha256" Hash="F23B0220060D5047BE53087F65D0B07A8874C11C85848CA051D9E759B803C97C" />
    <Allow ID="ID_ALLOW_A_981" FriendlyName="C:\Program Files\git\usr\bin\msys-z.dll Hash Sha1" Hash="F95959E77885A44D0F14A0662FCEAB321ACA6982" />
    <Allow ID="ID_ALLOW_A_982" FriendlyName="C:\Program Files\git\usr\bin\msys-z.dll Hash Sha256" Hash="B949D35B532AA4E2F34B0BFFD109B5FA1DFE0FDB7DFC5E5F403FA829CAC9CE0B" />
    <Allow ID="ID_ALLOW_A_983" FriendlyName="C:\Program Files\git\usr\bin\msys-z.dll Hash Page Sha1" Hash="2B821045DE8F06247039E64CB0E04500AFC4BC49" />
    <Allow ID="ID_ALLOW_A_984" FriendlyName="C:\Program Files\git\usr\bin\msys-z.dll Hash Page Sha256" Hash="0D467469F80CD3F3BC60DF5B00496C6E7FF4B332929F9FBA722092AEA326ADB3" />
    <Allow ID="ID_ALLOW_A_985" FriendlyName="C:\Program Files\git\usr\bin\mv.exe Hash Sha1" Hash="DCD4771AEDDE094A8D8D8EC499630AEEEE7798BB" />
    <Allow ID="ID_ALLOW_A_986" FriendlyName="C:\Program Files\git\usr\bin\mv.exe Hash Sha256" Hash="DF848D4BBE6E23DACB873995883E68CB36386B1732B8E412B1589960FAFC5412" />
    <Allow ID="ID_ALLOW_A_987" FriendlyName="C:\Program Files\git\usr\bin\mv.exe Hash Page Sha1" Hash="0F57B4DAAC4B1A3FCBD4D3CD1364B35AF8BE78ED" />
    <Allow ID="ID_ALLOW_A_988" FriendlyName="C:\Program Files\git\usr\bin\mv.exe Hash Page Sha256" Hash="0E34C107EE18FF45574A242C0779468779636D73C4AF361D604F84771E90B69C" />
    <Allow ID="ID_ALLOW_A_989" FriendlyName="C:\Program Files\git\usr\bin\nano.exe Hash Sha1" Hash="EC3BDE1CA4B09087C82D2B9EE637637F3850394F" />
    <Allow ID="ID_ALLOW_A_98A" FriendlyName="C:\Program Files\git\usr\bin\nano.exe Hash Sha256" Hash="2AF0379BCB27171A6E6C68922E075BE06D833446A6F35CA227E5D3E3950F415E" />
    <Allow ID="ID_ALLOW_A_98B" FriendlyName="C:\Program Files\git\usr\bin\nano.exe Hash Page Sha1" Hash="FA28E1D298FA148BF84EE45E9C26691133C9A0DD" />
    <Allow ID="ID_ALLOW_A_98C" FriendlyName="C:\Program Files\git\usr\bin\nano.exe Hash Page Sha256" Hash="066180C6BB2C07AA9922399D553B469A173479148989EBCC6ED315F7C67690DE" />
    <Allow ID="ID_ALLOW_A_98D" FriendlyName="C:\Program Files\git\usr\bin\nettle-hash.exe Hash Sha1" Hash="11EED21FA7BD7C6B43A027A0D4DA7980BEFC2F01" />
    <Allow ID="ID_ALLOW_A_98E" FriendlyName="C:\Program Files\git\usr\bin\nettle-hash.exe Hash Sha256" Hash="A84DEA159265B5F55D377CF863B2D633D58F6B32F7E6598009F82A7636999E2F" />
    <Allow ID="ID_ALLOW_A_98F" FriendlyName="C:\Program Files\git\usr\bin\nettle-hash.exe Hash Page Sha1" Hash="9075712F8170EEA3468FE0C7AD3F24E6175A2386" />
    <Allow ID="ID_ALLOW_A_990" FriendlyName="C:\Program Files\git\usr\bin\nettle-hash.exe Hash Page Sha256" Hash="DFC473ABBFF7D1D48D058E82A7D70A69C75F46F32B0FA4A7DCDFCD6C231A28C5" />
    <Allow ID="ID_ALLOW_A_991" FriendlyName="C:\Program Files\git\usr\bin\nettle-lfib-stream.exe Hash Sha1" Hash="3DBEEEABFEF9AB2F01B9CCCA80AF6F618BFDC6B6" />
    <Allow ID="ID_ALLOW_A_992" FriendlyName="C:\Program Files\git\usr\bin\nettle-lfib-stream.exe Hash Sha256" Hash="CEE6E53EC13983E0B5F36AE1337A1A22058D49F69EA5CFCE2C1BA0921695C490" />
    <Allow ID="ID_ALLOW_A_993" FriendlyName="C:\Program Files\git\usr\bin\nettle-lfib-stream.exe Hash Page Sha1" Hash="ED5243103F7A58A7EA2E57800B640D44672EBBC4" />
    <Allow ID="ID_ALLOW_A_994" FriendlyName="C:\Program Files\git\usr\bin\nettle-lfib-stream.exe Hash Page Sha256" Hash="E64E6176A4220187704685A2CEF4689B5472B484A136053F2B88FD508CCD2B66" />
    <Allow ID="ID_ALLOW_A_995" FriendlyName="C:\Program Files\git\usr\bin\nettle-pbkdf2.exe Hash Sha1" Hash="328E627B36C0537EB3FC9CB3262D547D294F909B" />
    <Allow ID="ID_ALLOW_A_996" FriendlyName="C:\Program Files\git\usr\bin\nettle-pbkdf2.exe Hash Sha256" Hash="0D55D71A58830891F400C3208989B476EFD9CFC126A976989DCCAB19F43AE8E5" />
    <Allow ID="ID_ALLOW_A_997" FriendlyName="C:\Program Files\git\usr\bin\nettle-pbkdf2.exe Hash Page Sha1" Hash="A036CEE69AB8F3C10565BF08F828BEBF1A35F720" />
    <Allow ID="ID_ALLOW_A_998" FriendlyName="C:\Program Files\git\usr\bin\nettle-pbkdf2.exe Hash Page Sha256" Hash="D417D4EB88E86CFE38929FCA1C2A96F19327A12B1A0330D478E3B3B9686200A7" />
    <Allow ID="ID_ALLOW_A_999" FriendlyName="C:\Program Files\git\usr\bin\ngettext.exe Hash Sha1" Hash="51535DFFA32F2FCF803A3A729904C6FAD462D40F" />
    <Allow ID="ID_ALLOW_A_99A" FriendlyName="C:\Program Files\git\usr\bin\ngettext.exe Hash Sha256" Hash="2A88A45904470376E6F6AA63CC1E0103B6402728D7898DBD6B25CBD5957A7C95" />
    <Allow ID="ID_ALLOW_A_99B" FriendlyName="C:\Program Files\git\usr\bin\ngettext.exe Hash Page Sha1" Hash="59DEC2C304DFD2F24799BE444EADEC87E808E66B" />
    <Allow ID="ID_ALLOW_A_99C" FriendlyName="C:\Program Files\git\usr\bin\ngettext.exe Hash Page Sha256" Hash="96C439C6EF272BB6559A398184ACE17D7932836B7C3868E18B042ED6B5B7C461" />
    <Allow ID="ID_ALLOW_A_99D" FriendlyName="C:\Program Files\git\usr\bin\nice.exe Hash Sha1" Hash="4664EFCAA79D23DEF4E9A529244F8F962187A979" />
    <Allow ID="ID_ALLOW_A_99E" FriendlyName="C:\Program Files\git\usr\bin\nice.exe Hash Sha256" Hash="725F821F8B887B7C46896AB44EEF300ECA934CC4674B34198C92A2684EB3320A" />
    <Allow ID="ID_ALLOW_A_99F" FriendlyName="C:\Program Files\git\usr\bin\nice.exe Hash Page Sha1" Hash="A73E916E101FDF0068D839B50AC39DCA763BA2A0" />
    <Allow ID="ID_ALLOW_A_9A0" FriendlyName="C:\Program Files\git\usr\bin\nice.exe Hash Page Sha256" Hash="ECC57BEC9B7A0023D39B89B218015053CDB13667910D39EC1C39B83904369676" />
    <Allow ID="ID_ALLOW_A_9A1" FriendlyName="C:\Program Files\git\usr\bin\nl.exe Hash Sha1" Hash="F202772B4D4212FA4DF4CAECE5C33AA7345B91E6" />
    <Allow ID="ID_ALLOW_A_9A2" FriendlyName="C:\Program Files\git\usr\bin\nl.exe Hash Sha256" Hash="A2A51AF8BE98D58C387E1BC430BB336F07A4D433CFB6D3B56646EABF934A9DD1" />
    <Allow ID="ID_ALLOW_A_9A3" FriendlyName="C:\Program Files\git\usr\bin\nl.exe Hash Page Sha1" Hash="E9A5338B6C4F43045FB60120C4827DA5D378CE4B" />
    <Allow ID="ID_ALLOW_A_9A4" FriendlyName="C:\Program Files\git\usr\bin\nl.exe Hash Page Sha256" Hash="0DCF9F7DCFDD1902B7CE790483035CEA0431CFD9980EC1F821045B981185CBCB" />
    <Allow ID="ID_ALLOW_A_9A5" FriendlyName="C:\Program Files\git\usr\bin\nohup.exe Hash Sha1" Hash="A273D558D5F0B312BEB848F10DC107E3C6BCD6AF" />
    <Allow ID="ID_ALLOW_A_9A6" FriendlyName="C:\Program Files\git\usr\bin\nohup.exe Hash Sha256" Hash="ED52AD1E5AC7A3C0CD7E49FC81D97B0DC6B282D193F4737EB842BC3F233C0DFE" />
    <Allow ID="ID_ALLOW_A_9A7" FriendlyName="C:\Program Files\git\usr\bin\nohup.exe Hash Page Sha1" Hash="70913A0F985C5D3657C5B442F594FFC1BD5C5DF5" />
    <Allow ID="ID_ALLOW_A_9A8" FriendlyName="C:\Program Files\git\usr\bin\nohup.exe Hash Page Sha256" Hash="BA74D7F66E435C01E4444603AB7EBEF6B52404515EB4E82AFEC8D79179BB31B1" />
    <Allow ID="ID_ALLOW_A_9A9" FriendlyName="C:\Program Files\git\usr\bin\nproc.exe Hash Sha1" Hash="5043ADCEA26B3293D781F95CA9AFBF68C2FE3763" />
    <Allow ID="ID_ALLOW_A_9AA" FriendlyName="C:\Program Files\git\usr\bin\nproc.exe Hash Sha256" Hash="9BC2E7FF205CA1A09BB540DC223FDA78357C5640A10A80B0B21A1DDB377876DD" />
    <Allow ID="ID_ALLOW_A_9AB" FriendlyName="C:\Program Files\git\usr\bin\nproc.exe Hash Page Sha1" Hash="D94AF27193123454E3EE1CAF2F1D1B94101BE198" />
    <Allow ID="ID_ALLOW_A_9AC" FriendlyName="C:\Program Files\git\usr\bin\nproc.exe Hash Page Sha256" Hash="C640088A650B1FE42EC0461E59769FC65033AC0FC36870A0470DCD43A58C35A1" />
    <Allow ID="ID_ALLOW_A_9AD" FriendlyName="C:\Program Files\git\usr\bin\numfmt.exe Hash Sha1" Hash="77305E02A298B921A42359DFB3F50548C098AB6F" />
    <Allow ID="ID_ALLOW_A_9AE" FriendlyName="C:\Program Files\git\usr\bin\numfmt.exe Hash Sha256" Hash="50952AF93A9D0BE30784C44079735A8DCA73F3B20AA733780537675ADE0F7F84" />
    <Allow ID="ID_ALLOW_A_9AF" FriendlyName="C:\Program Files\git\usr\bin\numfmt.exe Hash Page Sha1" Hash="601EB13D4C1C4212988F9574920497CF9220AB7A" />
    <Allow ID="ID_ALLOW_A_9B0" FriendlyName="C:\Program Files\git\usr\bin\numfmt.exe Hash Page Sha256" Hash="5707FB5BF36ED075CE466F9366D8890204791BBB79930421325F175F78272A6F" />
    <Allow ID="ID_ALLOW_A_9B1" FriendlyName="C:\Program Files\git\usr\bin\od.exe Hash Sha1" Hash="87D5E2238030229517DF9088AAF835311D1DCDD6" />
    <Allow ID="ID_ALLOW_A_9B2" FriendlyName="C:\Program Files\git\usr\bin\od.exe Hash Sha256" Hash="B4F3FE00860784A16EF6D75CE7FEED9FD9EAFBB646EE6F5ED5293C67BE553283" />
    <Allow ID="ID_ALLOW_A_9B3" FriendlyName="C:\Program Files\git\usr\bin\od.exe Hash Page Sha1" Hash="E47EE4AE54EE5E71AC9733A00A35F37C4A752882" />
    <Allow ID="ID_ALLOW_A_9B4" FriendlyName="C:\Program Files\git\usr\bin\od.exe Hash Page Sha256" Hash="5A418D473A2D1139AA50C2D66111B0083E8BBEC0667DD8A1DAEA43E90DE35448" />
    <Allow ID="ID_ALLOW_A_9B5" FriendlyName="C:\Program Files\git\usr\bin\openssl.exe Hash Sha1" Hash="2C8F4D508E8F89BB993F08A016863D759A440525" />
    <Allow ID="ID_ALLOW_A_9B6" FriendlyName="C:\Program Files\git\usr\bin\openssl.exe Hash Sha256" Hash="A70CD1FF931C47C5AF00DD3B2B9BF4A9C716A620286FE062326E22AFBB25C3CB" />
    <Allow ID="ID_ALLOW_A_9B7" FriendlyName="C:\Program Files\git\usr\bin\openssl.exe Hash Page Sha1" Hash="277CAE9C19BB1938AE53CD33E8BF5EA810A4447E" />
    <Allow ID="ID_ALLOW_A_9B8" FriendlyName="C:\Program Files\git\usr\bin\openssl.exe Hash Page Sha256" Hash="AF9BD911E686F84382D39487BE5750C5A2CA0BF4726771541F2135EB3FE43D41" />
    <Allow ID="ID_ALLOW_A_9B9" FriendlyName="C:\Program Files\git\usr\bin\p11-kit.exe Hash Sha1" Hash="93F4A58870D68CAD4AD1A9ECA1EE9F5AC8143573" />
    <Allow ID="ID_ALLOW_A_9BA" FriendlyName="C:\Program Files\git\usr\bin\p11-kit.exe Hash Sha256" Hash="FF87D75D514899D1439AD2CE4490FE193912C683AA61BEF614C3114B823CF784" />
    <Allow ID="ID_ALLOW_A_9BB" FriendlyName="C:\Program Files\git\usr\bin\p11-kit.exe Hash Page Sha1" Hash="31051174B34D669613B37F8D14F3C3CF78584234" />
    <Allow ID="ID_ALLOW_A_9BC" FriendlyName="C:\Program Files\git\usr\bin\p11-kit.exe Hash Page Sha256" Hash="7ABCDA3A578E2BDEFEF7E9F326206D2112DDC4394F2367B18F48FDC4E5E92478" />
    <Allow ID="ID_ALLOW_A_9BD" FriendlyName="C:\Program Files\git\usr\bin\passwd.exe Hash Sha1" Hash="412233E06A39B424CFF77AD27D60E3DEA2C83346" />
    <Allow ID="ID_ALLOW_A_9BE" FriendlyName="C:\Program Files\git\usr\bin\passwd.exe Hash Sha256" Hash="BEBC56FC793C9A9330706344456F2763940ECFE529D15121E17B98936AE9A77A" />
    <Allow ID="ID_ALLOW_A_9BF" FriendlyName="C:\Program Files\git\usr\bin\passwd.exe Hash Page Sha1" Hash="552DEC229B46526DA07DD7E7B385ED86D71EA71B" />
    <Allow ID="ID_ALLOW_A_9C0" FriendlyName="C:\Program Files\git\usr\bin\passwd.exe Hash Page Sha256" Hash="BF8D0D6E29B39FB242109B7F6BDA769D87F71A88F294C73DFBC123C90B8CF085" />
    <Allow ID="ID_ALLOW_A_9C1" FriendlyName="C:\Program Files\git\usr\bin\paste.exe Hash Sha1" Hash="A060177856C234E7CDC0A4F3EFE1BC8E90AC7A09" />
    <Allow ID="ID_ALLOW_A_9C2" FriendlyName="C:\Program Files\git\usr\bin\paste.exe Hash Sha256" Hash="27D656FAC6CF70E182EC4E60FF562395C1540FEE6C30A5C3B68AF7876B80C116" />
    <Allow ID="ID_ALLOW_A_9C3" FriendlyName="C:\Program Files\git\usr\bin\paste.exe Hash Page Sha1" Hash="16EC4512384D609DDECD4F0D17D84F3E9F2BD2F3" />
    <Allow ID="ID_ALLOW_A_9C4" FriendlyName="C:\Program Files\git\usr\bin\paste.exe Hash Page Sha256" Hash="4F152264B2CBE2E5111A1DD73DC3AC7A34D7B49B6C8EC3A64232210581EC2F62" />
    <Allow ID="ID_ALLOW_A_9C5" FriendlyName="C:\Program Files\git\usr\bin\patch.exe Hash Sha1" Hash="B897822F656267EBE44A17D9F5DAEFC4DFA897FF" />
    <Allow ID="ID_ALLOW_A_9C6" FriendlyName="C:\Program Files\git\usr\bin\patch.exe Hash Sha256" Hash="114C9587D7D148AF6B1B30515BFA6117F386A6F9E6DE27749CC3FFDCDDF40F1F" />
    <Allow ID="ID_ALLOW_A_9C7" FriendlyName="C:\Program Files\git\usr\bin\patch.exe Hash Page Sha1" Hash="1D91A31A2CCAE270C36E85808D832DF4E86527BF" />
    <Allow ID="ID_ALLOW_A_9C8" FriendlyName="C:\Program Files\git\usr\bin\patch.exe Hash Page Sha256" Hash="E0B282D9E34F57FF4941B99C0E4895592C79759BF41A82B78F34060031981691" />
    <Allow ID="ID_ALLOW_A_9C9" FriendlyName="C:\Program Files\git\usr\bin\pathchk.exe Hash Sha1" Hash="342E497676167635D8A23977A7D6EAB57D63F08D" />
    <Allow ID="ID_ALLOW_A_9CA" FriendlyName="C:\Program Files\git\usr\bin\pathchk.exe Hash Sha256" Hash="4A7745FA7C8D723536CA98EF8C8271557C0BC70B5BC656BA1756A8F5C877F99E" />
    <Allow ID="ID_ALLOW_A_9CB" FriendlyName="C:\Program Files\git\usr\bin\pathchk.exe Hash Page Sha1" Hash="131861F48A0F279774B60A7C995080714202D1CB" />
    <Allow ID="ID_ALLOW_A_9CC" FriendlyName="C:\Program Files\git\usr\bin\pathchk.exe Hash Page Sha256" Hash="B8723C14AF3B4ECF4349EDB4B43677A1970A8C9D0289328EB584305824782EDC" />
    <Allow ID="ID_ALLOW_A_9CD" FriendlyName="C:\Program Files\git\usr\bin\perl.exe Hash Sha1" Hash="6F2975747EF70B1AD781C57A20BC3EE51F2341A2" />
    <Allow ID="ID_ALLOW_A_9CE" FriendlyName="C:\Program Files\git\usr\bin\perl.exe Hash Sha256" Hash="ABA80B41D06D2E85F0911D2F6BB9348EE28E53B074C7BB5B43BB5426623804A1" />
    <Allow ID="ID_ALLOW_A_9CF" FriendlyName="C:\Program Files\git\usr\bin\perl.exe Hash Page Sha1" Hash="A06BFC2995CD457FDCF8F8B81457618F2497DD79" />
    <Allow ID="ID_ALLOW_A_9D0" FriendlyName="C:\Program Files\git\usr\bin\perl.exe Hash Page Sha256" Hash="7B0D96F74D26C509A8A7841CE6369594B501415E6D2C110C6EB157A3430CE3D4" />
    <Allow ID="ID_ALLOW_A_9D1" FriendlyName="C:\Program Files\git\usr\bin\pinentry-w32.exe Hash Sha1" Hash="CF3A0DDADE0A579CC276324EA2C1CE3C13514C20" />
    <Allow ID="ID_ALLOW_A_9D2" FriendlyName="C:\Program Files\git\usr\bin\pinentry-w32.exe Hash Sha256" Hash="3909936432D18E93B0710D56958EC507FB7ECD3BDD5A7CA6D4C00E81B8319345" />
    <Allow ID="ID_ALLOW_A_9D3" FriendlyName="C:\Program Files\git\usr\bin\pinentry-w32.exe Hash Page Sha1" Hash="06B636EAF0B560C7D5CA80721064A9A7E7B644DB" />
    <Allow ID="ID_ALLOW_A_9D4" FriendlyName="C:\Program Files\git\usr\bin\pinentry-w32.exe Hash Page Sha256" Hash="48660E30F374B8432BC54B667D3593F3C4059DF247A6A26501E7523FCD145745" />
    <Allow ID="ID_ALLOW_A_9D5" FriendlyName="C:\Program Files\git\usr\bin\pinky.exe Hash Sha1" Hash="99898A710EC6EA05E0A9FDBF5DE0D65C02A4F8B3" />
    <Allow ID="ID_ALLOW_A_9D6" FriendlyName="C:\Program Files\git\usr\bin\pinky.exe Hash Sha256" Hash="242250CE7B3973DA0AE9045A521EEA39BC4CA942C69EFD03BFFABF790A2B245D" />
    <Allow ID="ID_ALLOW_A_9D7" FriendlyName="C:\Program Files\git\usr\bin\pinky.exe Hash Page Sha1" Hash="C02B90A94AC03EC711F61BFCE2726CAD21CDD7DB" />
    <Allow ID="ID_ALLOW_A_9D8" FriendlyName="C:\Program Files\git\usr\bin\pinky.exe Hash Page Sha256" Hash="7406F2527673155C8710FF89C65BB63D82C7B917A68C59CCC77BB409B06EAB55" />
    <Allow ID="ID_ALLOW_A_9D9" FriendlyName="C:\Program Files\git\usr\bin\pkcs1-conv.exe Hash Sha1" Hash="56633108701F9DD93DE0E0C4BF2E30ACEB5872B7" />
    <Allow ID="ID_ALLOW_A_9DA" FriendlyName="C:\Program Files\git\usr\bin\pkcs1-conv.exe Hash Sha256" Hash="6F925630AD30F2B2D27288256D26DFF7789D6989B9F978B0741678EB4235DE04" />
    <Allow ID="ID_ALLOW_A_9DB" FriendlyName="C:\Program Files\git\usr\bin\pkcs1-conv.exe Hash Page Sha1" Hash="86B1664F345F6786E1AC886AFC6ACD0893C64752" />
    <Allow ID="ID_ALLOW_A_9DC" FriendlyName="C:\Program Files\git\usr\bin\pkcs1-conv.exe Hash Page Sha256" Hash="9A55759B642AB5C3A4F3C38F2356C1EF136A88569C43AB0DEBB82ACAB7665187" />
    <Allow ID="ID_ALLOW_A_9DD" FriendlyName="C:\Program Files\git\usr\bin\pldd.exe Hash Sha1" Hash="E376A53FC7FDB582A7E047F154047443FF48C50B" />
    <Allow ID="ID_ALLOW_A_9DE" FriendlyName="C:\Program Files\git\usr\bin\pldd.exe Hash Sha256" Hash="845F3CB66059878AE2B15060791D73D8134DFC78BEFF80A1944FB5A53451391A" />
    <Allow ID="ID_ALLOW_A_9DF" FriendlyName="C:\Program Files\git\usr\bin\pldd.exe Hash Page Sha1" Hash="ABC31498218C3D9FCEC055493602B820E43BC27E" />
    <Allow ID="ID_ALLOW_A_9E0" FriendlyName="C:\Program Files\git\usr\bin\pldd.exe Hash Page Sha256" Hash="DDF3C690F8B0B4F00DBD7C10A128C5F5B6C0FCF85A16461939082D7A6D6F9627" />
    <Allow ID="ID_ALLOW_A_9E1" FriendlyName="C:\Program Files\git\usr\bin\pluginviewer.exe Hash Sha1" Hash="CB4F9CD11384FDAA4A4ACEC348972DD434C91ABD" />
    <Allow ID="ID_ALLOW_A_9E2" FriendlyName="C:\Program Files\git\usr\bin\pluginviewer.exe Hash Sha256" Hash="D693598D5AD9E94BE19F038D058B8FF8A1B12DC2405F8CC4ABB706A175F0291F" />
    <Allow ID="ID_ALLOW_A_9E3" FriendlyName="C:\Program Files\git\usr\bin\pluginviewer.exe Hash Page Sha1" Hash="834D2BB4EF3ECD0657D1AD1BA37EFCC1A74A169D" />
    <Allow ID="ID_ALLOW_A_9E4" FriendlyName="C:\Program Files\git\usr\bin\pluginviewer.exe Hash Page Sha256" Hash="B484BDFF1BE4280D0A08E46C7AA37BE6A2BE88E3C851DB0906D431339A29826B" />
    <Allow ID="ID_ALLOW_A_9E5" FriendlyName="C:\Program Files\git\usr\bin\pr.exe Hash Sha1" Hash="751ACB38B77056939A61E328C03E02D753EA8783" />
    <Allow ID="ID_ALLOW_A_9E6" FriendlyName="C:\Program Files\git\usr\bin\pr.exe Hash Sha256" Hash="B5114BDE7181CD82461EB5CC8A22F7A2B30D827DDDB03F9B47EAD04936C4FEF8" />
    <Allow ID="ID_ALLOW_A_9E7" FriendlyName="C:\Program Files\git\usr\bin\pr.exe Hash Page Sha1" Hash="92BBF60C69F46995B11BFDD76CF66A3C50F7A62F" />
    <Allow ID="ID_ALLOW_A_9E8" FriendlyName="C:\Program Files\git\usr\bin\pr.exe Hash Page Sha256" Hash="B9E0AC060A0C841EDE5460F6B486C738385EE713AB111933A729D32540A7824F" />
    <Allow ID="ID_ALLOW_A_9E9" FriendlyName="C:\Program Files\git\usr\bin\printenv.exe Hash Sha1" Hash="D801B0932A2662BBF6BA677DCA1812973EB5F384" />
    <Allow ID="ID_ALLOW_A_9EA" FriendlyName="C:\Program Files\git\usr\bin\printenv.exe Hash Sha256" Hash="D51E98B196ACEDA733138DAC12066F88133FA5365E4E373B1FF5B5BBABA4E8EE" />
    <Allow ID="ID_ALLOW_A_9EB" FriendlyName="C:\Program Files\git\usr\bin\printenv.exe Hash Page Sha1" Hash="407F87AC795F7B044BAE41AADAECC3BDF7810D8E" />
    <Allow ID="ID_ALLOW_A_9EC" FriendlyName="C:\Program Files\git\usr\bin\printenv.exe Hash Page Sha256" Hash="7D9C4F8132028F7F6234B9A71686C37ABCDB98E616FCF5C9B169D7B8029699B3" />
    <Allow ID="ID_ALLOW_A_9ED" FriendlyName="C:\Program Files\git\usr\bin\printf.exe Hash Sha1" Hash="BB7020AB8644540B455E38D93C1245572319C5D7" />
    <Allow ID="ID_ALLOW_A_9EE" FriendlyName="C:\Program Files\git\usr\bin\printf.exe Hash Sha256" Hash="E83BE0F56F004AF3399FED5A0CB6735237FB10C8A44D6B38EFE08512C490EE21" />
    <Allow ID="ID_ALLOW_A_9EF" FriendlyName="C:\Program Files\git\usr\bin\printf.exe Hash Page Sha1" Hash="ECB7A4732A9F4013276B93AB339E3EC80FE54D19" />
    <Allow ID="ID_ALLOW_A_9F0" FriendlyName="C:\Program Files\git\usr\bin\printf.exe Hash Page Sha256" Hash="A3BA10479FD6899D5E355B65B43787B5518DB4A81761307E13AEA2376C184783" />
    <Allow ID="ID_ALLOW_A_9F1" FriendlyName="C:\Program Files\git\usr\bin\ps.exe Hash Sha1" Hash="F2FB6E46F9CDE02DDB4EF4205F8ABEF68932BB2D" />
    <Allow ID="ID_ALLOW_A_9F2" FriendlyName="C:\Program Files\git\usr\bin\ps.exe Hash Sha256" Hash="8942FF49C26556ADFC4EE719F974217CBCDFBB429F52A11FB51AB9C17A7A3EAE" />
    <Allow ID="ID_ALLOW_A_9F3" FriendlyName="C:\Program Files\git\usr\bin\ps.exe Hash Page Sha1" Hash="4C87C98A7C3EA9C318CB66ADF58F4AFD76A2D920" />
    <Allow ID="ID_ALLOW_A_9F4" FriendlyName="C:\Program Files\git\usr\bin\ps.exe Hash Page Sha256" Hash="26DA35254BB28E54D9211F8743ABC38266F809129324467617A4D06CB991C976" />
    <Allow ID="ID_ALLOW_A_9F5" FriendlyName="C:\Program Files\git\usr\bin\psl.exe Hash Sha1" Hash="B40589D5413035D12023AACF8104C3AEEE7446BC" />
    <Allow ID="ID_ALLOW_A_9F6" FriendlyName="C:\Program Files\git\usr\bin\psl.exe Hash Sha256" Hash="45B560067169F90733DC49C78381F742E66B241BEE34AD54FC0CBF94935634F3" />
    <Allow ID="ID_ALLOW_A_9F7" FriendlyName="C:\Program Files\git\usr\bin\psl.exe Hash Page Sha1" Hash="A7543FD254B81C80286BCAF82C04D0FEADD1F0F9" />
    <Allow ID="ID_ALLOW_A_9F8" FriendlyName="C:\Program Files\git\usr\bin\psl.exe Hash Page Sha256" Hash="277711C393B51C57290028E588425F28FF96CD4FC51026E53BE5F0216A0CF2F4" />
    <Allow ID="ID_ALLOW_A_9F9" FriendlyName="C:\Program Files\git\usr\bin\ptx.exe Hash Sha1" Hash="27EA00739769246674A4EDDDFA4C22FCA6D0C368" />
    <Allow ID="ID_ALLOW_A_9FA" FriendlyName="C:\Program Files\git\usr\bin\ptx.exe Hash Sha256" Hash="564915DDA3412FC29DE31A5AB513C0A9903F3BF06517ACF767D45AE8F39420D1" />
    <Allow ID="ID_ALLOW_A_9FB" FriendlyName="C:\Program Files\git\usr\bin\ptx.exe Hash Page Sha1" Hash="F1B841ECD577CF0FC17754C9B7679F58A0D4E4F9" />
    <Allow ID="ID_ALLOW_A_9FC" FriendlyName="C:\Program Files\git\usr\bin\ptx.exe Hash Page Sha256" Hash="768A1BB39F74E93C0D1409F3139F21C4DC33C6C3A06735A2AE4D3119E8F4F274" />
    <Allow ID="ID_ALLOW_A_9FD" FriendlyName="C:\Program Files\git\usr\bin\pwd.exe Hash Sha1" Hash="C0A5D9614BB8BB9064E4F9CB43B38A894B61F13A" />
    <Allow ID="ID_ALLOW_A_9FE" FriendlyName="C:\Program Files\git\usr\bin\pwd.exe Hash Sha256" Hash="5D0110DEE4A00D4852FA55504D1C0B12059E70A840AE1E730238A26B9ADFCD94" />
    <Allow ID="ID_ALLOW_A_9FF" FriendlyName="C:\Program Files\git\usr\bin\pwd.exe Hash Page Sha1" Hash="790FAD9BE788E851F8B028B5088AF3379BDED837" />
    <Allow ID="ID_ALLOW_A_A00" FriendlyName="C:\Program Files\git\usr\bin\pwd.exe Hash Page Sha256" Hash="8B6EB5A3A02C382FC62694900B4CCA409E59CC11D19B2D0D53040D353DE8ACF6" />
    <Allow ID="ID_ALLOW_A_A01" FriendlyName="C:\Program Files\git\usr\bin\readlink.exe Hash Sha1" Hash="F68A7179177AFC4FBB34C369FAD9F2F19790398E" />
    <Allow ID="ID_ALLOW_A_A02" FriendlyName="C:\Program Files\git\usr\bin\readlink.exe Hash Sha256" Hash="8C3E502A79861D2A45700579269E1D4D69F497B9ABCD073587645BD407F720F3" />
    <Allow ID="ID_ALLOW_A_A03" FriendlyName="C:\Program Files\git\usr\bin\readlink.exe Hash Page Sha1" Hash="E2BDCCAD24F89501D1E431B0532BED49DB7966FA" />
    <Allow ID="ID_ALLOW_A_A04" FriendlyName="C:\Program Files\git\usr\bin\readlink.exe Hash Page Sha256" Hash="AE078B5C7190BF75BC971C2BC13D0DC88BC50F8F891D0C16120F52EB6097DAD3" />
    <Allow ID="ID_ALLOW_A_A05" FriendlyName="C:\Program Files\git\usr\bin\realpath.exe Hash Sha1" Hash="D1AB2F08ECA508CB5C4329C5CE2F5E11CF55D558" />
    <Allow ID="ID_ALLOW_A_A06" FriendlyName="C:\Program Files\git\usr\bin\realpath.exe Hash Sha256" Hash="B0032B5BD873E2A3FC1BEEA639F15CF1C21B332B8EDBA97FE88890C799DA2E7B" />
    <Allow ID="ID_ALLOW_A_A07" FriendlyName="C:\Program Files\git\usr\bin\realpath.exe Hash Page Sha1" Hash="D0ED7B79428A6BDD4B17A781CB6A80EB20D315A3" />
    <Allow ID="ID_ALLOW_A_A08" FriendlyName="C:\Program Files\git\usr\bin\realpath.exe Hash Page Sha256" Hash="AE970ADA33D4274E4E4F618F36BEF0A7973F638960A3B3B162523365198D8E61" />
    <Allow ID="ID_ALLOW_A_A09" FriendlyName="C:\Program Files\git\usr\bin\rebase.exe Hash Sha1" Hash="D443354E6E221FF08856E816DFBCC8987496BD21" />
    <Allow ID="ID_ALLOW_A_A0A" FriendlyName="C:\Program Files\git\usr\bin\rebase.exe Hash Sha256" Hash="A8A3957DF650396594700912BC73AAA73CDD65A9A32C671454FAEE5139B01809" />
    <Allow ID="ID_ALLOW_A_A0B" FriendlyName="C:\Program Files\git\usr\bin\rebase.exe Hash Page Sha1" Hash="75B655BA389A4F3367F69BA251BD7FA2255C9B35" />
    <Allow ID="ID_ALLOW_A_A0C" FriendlyName="C:\Program Files\git\usr\bin\rebase.exe Hash Page Sha256" Hash="48F7B13561749DF89DBF965F2FBEB29CA752D8292C42407F69F2E06DF3FF0477" />
    <Allow ID="ID_ALLOW_A_A0D" FriendlyName="C:\Program Files\git\usr\bin\recode-sr-latin.exe Hash Sha1" Hash="18FF70E637AB711B2791FFA97D7F579E5095C79F" />
    <Allow ID="ID_ALLOW_A_A0E" FriendlyName="C:\Program Files\git\usr\bin\recode-sr-latin.exe Hash Sha256" Hash="FA55AFE71E518B9A0EA056664246C674E9CB18A0DF8C9570033EA369334F1CAE" />
    <Allow ID="ID_ALLOW_A_A0F" FriendlyName="C:\Program Files\git\usr\bin\recode-sr-latin.exe Hash Page Sha1" Hash="5719DC8B6D71FF4C50F0F0DAF52A7B523BA245BA" />
    <Allow ID="ID_ALLOW_A_A10" FriendlyName="C:\Program Files\git\usr\bin\recode-sr-latin.exe Hash Page Sha256" Hash="20F937F8E6436985885645C389D9B8FDECD3347AB06B2FE8F4091E858D9FAAA9" />
    <Allow ID="ID_ALLOW_A_A11" FriendlyName="C:\Program Files\git\usr\bin\regtool.exe Hash Sha1" Hash="9346D4610079CBB6DDE59DE1E2963C90E9F46880" />
    <Allow ID="ID_ALLOW_A_A12" FriendlyName="C:\Program Files\git\usr\bin\regtool.exe Hash Sha256" Hash="06AEC92B2AC689A4065189B4F8C2C5FC8AE1B19BF3031D8195C4CE05EFC8900F" />
    <Allow ID="ID_ALLOW_A_A13" FriendlyName="C:\Program Files\git\usr\bin\regtool.exe Hash Page Sha1" Hash="21C19752E99D8D76EE0BCF08CE5835A7D85F1DB7" />
    <Allow ID="ID_ALLOW_A_A14" FriendlyName="C:\Program Files\git\usr\bin\regtool.exe Hash Page Sha256" Hash="6BB8FA1C449ED7E4835AF410287BAD8E10CC0D07F5CB931FD4C24C231F096642" />
    <Allow ID="ID_ALLOW_A_A15" FriendlyName="C:\Program Files\git\usr\bin\reset.exe Hash Sha1" Hash="D5D60D1EF0D6C71F01E9E5EB596F4935BD84CA6F" />
    <Allow ID="ID_ALLOW_A_A16" FriendlyName="C:\Program Files\git\usr\bin\reset.exe Hash Sha256" Hash="C7B3D5B34A0E3768C8A6CCBBE1B34CF3CC74433C7EB1CAF586848FC35B275198" />
    <Allow ID="ID_ALLOW_A_A17" FriendlyName="C:\Program Files\git\usr\bin\reset.exe Hash Page Sha1" Hash="EE8DF4820DBEED446D0C79E938A8BA35D3669346" />
    <Allow ID="ID_ALLOW_A_A18" FriendlyName="C:\Program Files\git\usr\bin\reset.exe Hash Page Sha256" Hash="116BCD1F19297539E4E123F228FAD1B84C35DC01C36C0687E40C2214CDFE0413" />
    <Allow ID="ID_ALLOW_A_A19" FriendlyName="C:\Program Files\git\usr\bin\rm.exe Hash Sha1" Hash="5F02A98337B05797C776D39B2BD42AA75FDA1C50" />
    <Allow ID="ID_ALLOW_A_A1A" FriendlyName="C:\Program Files\git\usr\bin\rm.exe Hash Sha256" Hash="5FC953400F45402E74D497036ED316E04C9219899E119980C5E496E4C588B521" />
    <Allow ID="ID_ALLOW_A_A1B" FriendlyName="C:\Program Files\git\usr\bin\rm.exe Hash Page Sha1" Hash="3538934215CF685BA405D16FBABD275BF0B6457F" />
    <Allow ID="ID_ALLOW_A_A1C" FriendlyName="C:\Program Files\git\usr\bin\rm.exe Hash Page Sha256" Hash="3D7CDDF2E2BAAB3738BD551125E3192C31F655119A846EA4076626C4D3F08EB6" />
    <Allow ID="ID_ALLOW_A_A1D" FriendlyName="C:\Program Files\git\usr\bin\rmdir.exe Hash Sha1" Hash="3E4467ECD10EEF8E11FC63D9AF064109EF170010" />
    <Allow ID="ID_ALLOW_A_A1E" FriendlyName="C:\Program Files\git\usr\bin\rmdir.exe Hash Sha256" Hash="E305DA45801310EEFEB3C1557EEAB96E92AA794A057D7612A2117AFEB45C3AFC" />
    <Allow ID="ID_ALLOW_A_A1F" FriendlyName="C:\Program Files\git\usr\bin\rmdir.exe Hash Page Sha1" Hash="3756E65731D57EF661BEDFBDC1C558683A30E66F" />
    <Allow ID="ID_ALLOW_A_A20" FriendlyName="C:\Program Files\git\usr\bin\rmdir.exe Hash Page Sha256" Hash="AE404B1DD499F5FA0FE18B1EDE8F39819800A7A0DC4EFA0BB77E660AD25BD604" />
    <Allow ID="ID_ALLOW_A_A21" FriendlyName="C:\Program Files\git\usr\bin\runcon.exe Hash Sha1" Hash="B59CDD26E30529CBEDC98E192A319CDBB92C97CE" />
    <Allow ID="ID_ALLOW_A_A22" FriendlyName="C:\Program Files\git\usr\bin\runcon.exe Hash Sha256" Hash="46779AB8A558A0AECFD7F3D36DDCE764D54F445AFF6978BD8EA8D0A09C0325DE" />
    <Allow ID="ID_ALLOW_A_A23" FriendlyName="C:\Program Files\git\usr\bin\runcon.exe Hash Page Sha1" Hash="8D307D384553922B7683B5232F36F5B7129D0081" />
    <Allow ID="ID_ALLOW_A_A24" FriendlyName="C:\Program Files\git\usr\bin\runcon.exe Hash Page Sha256" Hash="A003157F0F5BD470002D0A25081A948A253B83AE35D926079D2498B9F20D2463" />
    <Allow ID="ID_ALLOW_A_A25" FriendlyName="C:\Program Files\git\usr\bin\rview.exe Hash Sha1" Hash="02FF2C36BD3165570D3904FB95E6A0647F2E6A88" />
    <Allow ID="ID_ALLOW_A_A26" FriendlyName="C:\Program Files\git\usr\bin\rview.exe Hash Sha256" Hash="9566127BC106640BEBA92B7B374503CAE5F30F2B9468920D87BD02A9E32AB179" />
    <Allow ID="ID_ALLOW_A_A27" FriendlyName="C:\Program Files\git\usr\bin\rview.exe Hash Page Sha1" Hash="951B503682AC68D78C28667A829C511848900ACD" />
    <Allow ID="ID_ALLOW_A_A28" FriendlyName="C:\Program Files\git\usr\bin\rview.exe Hash Page Sha256" Hash="FB4FB141BC46C5EF0504D2EECCA177E14D9680BF481720DD5AB1A60E1A8C45B9" />
    <Allow ID="ID_ALLOW_A_A29" FriendlyName="C:\Program Files\git\usr\bin\scp.exe Hash Sha1" Hash="1E5C190DCE6D469A774AA567C578FAEBDF9DEE58" />
    <Allow ID="ID_ALLOW_A_A2A" FriendlyName="C:\Program Files\git\usr\bin\scp.exe Hash Sha256" Hash="504792DA6F8C6D400103D57F074E6D3112D6372D3D06E367EDB10748B86E098E" />
    <Allow ID="ID_ALLOW_A_A2B" FriendlyName="C:\Program Files\git\usr\bin\scp.exe Hash Page Sha1" Hash="1D5C84E6A53FDACD5DDF1C1809660AE4BE939AE8" />
    <Allow ID="ID_ALLOW_A_A2C" FriendlyName="C:\Program Files\git\usr\bin\scp.exe Hash Page Sha256" Hash="E0CDB58903415184685A08A8A38DBF93C8E5C8DA96FAC36E61196D6C9DB68F70" />
    <Allow ID="ID_ALLOW_A_A2D" FriendlyName="C:\Program Files\git\usr\bin\sdiff.exe Hash Sha1" Hash="51667D996559783659C8DDC8EDD251F5D154E3DD" />
    <Allow ID="ID_ALLOW_A_A2E" FriendlyName="C:\Program Files\git\usr\bin\sdiff.exe Hash Sha256" Hash="E78CBB65F7D072652BCB654877562CF6B5420B6E3E97C68466E9001556392CEC" />
    <Allow ID="ID_ALLOW_A_A2F" FriendlyName="C:\Program Files\git\usr\bin\sdiff.exe Hash Page Sha1" Hash="850AC31C8E5831E8363D0BF3608524BC61BE1FDB" />
    <Allow ID="ID_ALLOW_A_A30" FriendlyName="C:\Program Files\git\usr\bin\sdiff.exe Hash Page Sha256" Hash="42FDD0A4E64D3D67FB34ADCA8C572B92F67915ED1CFF74B89C94BC2C834C811D" />
    <Allow ID="ID_ALLOW_A_A31" FriendlyName="C:\Program Files\git\usr\bin\sed.exe Hash Sha1" Hash="6F9F06E9C403FAD8258EA7D994F0C4470122557D" />
    <Allow ID="ID_ALLOW_A_A32" FriendlyName="C:\Program Files\git\usr\bin\sed.exe Hash Sha256" Hash="9140AE4E692666D1EFE83DF01C640BEE47612FE700FE8C69BB8A11D9B1833D68" />
    <Allow ID="ID_ALLOW_A_A33" FriendlyName="C:\Program Files\git\usr\bin\sed.exe Hash Page Sha1" Hash="BACDD4B4610A973B52AB51BB502755F9213E3276" />
    <Allow ID="ID_ALLOW_A_A34" FriendlyName="C:\Program Files\git\usr\bin\sed.exe Hash Page Sha256" Hash="1C5807B12D19E3C3954EEC06EFAA0558EE5E8963356DB496E83D330457E0DFA9" />
    <Allow ID="ID_ALLOW_A_A35" FriendlyName="C:\Program Files\git\usr\bin\seq.exe Hash Sha1" Hash="5B6C68BF5BF839A3E4614E3A013043364A6ED3E8" />
    <Allow ID="ID_ALLOW_A_A36" FriendlyName="C:\Program Files\git\usr\bin\seq.exe Hash Sha256" Hash="B32D65C9C94D29D7FE1116636B1C3D545C2120E63C0F7A313AF1D476F717AAD8" />
    <Allow ID="ID_ALLOW_A_A37" FriendlyName="C:\Program Files\git\usr\bin\seq.exe Hash Page Sha1" Hash="5B703F7BC169F854875C5A54877CB087FF4F921B" />
    <Allow ID="ID_ALLOW_A_A38" FriendlyName="C:\Program Files\git\usr\bin\seq.exe Hash Page Sha256" Hash="C5EC413D973360B89C5E623AA75390D7CF2ED902C9CB073D74F40C6A9D3EFB63" />
    <Allow ID="ID_ALLOW_A_A39" FriendlyName="C:\Program Files\git\usr\bin\setfacl.exe Hash Sha1" Hash="ED331206A5883DEEA2F21B976AAE73EB6FAF89D1" />
    <Allow ID="ID_ALLOW_A_A3A" FriendlyName="C:\Program Files\git\usr\bin\setfacl.exe Hash Sha256" Hash="7BB385153A26948665DDE1D929D60B6BA03BDC01F07C89E6767CE764996D84E7" />
    <Allow ID="ID_ALLOW_A_A3B" FriendlyName="C:\Program Files\git\usr\bin\setfacl.exe Hash Page Sha1" Hash="35824DA327733D1EF4593799B89ACA11D3B93578" />
    <Allow ID="ID_ALLOW_A_A3C" FriendlyName="C:\Program Files\git\usr\bin\setfacl.exe Hash Page Sha256" Hash="D054C75A388A65E60FC1B33E16B6BA98D39E550F728142337115556824AD07B0" />
    <Allow ID="ID_ALLOW_A_A3D" FriendlyName="C:\Program Files\git\usr\bin\setmetamode.exe Hash Sha1" Hash="821E8F0C7A0C276AC39B42767715F319B30D405E" />
    <Allow ID="ID_ALLOW_A_A3E" FriendlyName="C:\Program Files\git\usr\bin\setmetamode.exe Hash Sha256" Hash="CF6C2B44D7E3E680053AB55F8F1F257FC9E98DAA2F8B8052E66A136111C2AE4C" />
    <Allow ID="ID_ALLOW_A_A3F" FriendlyName="C:\Program Files\git\usr\bin\setmetamode.exe Hash Page Sha1" Hash="409F30959B1F96623864A97F508BCA643968D610" />
    <Allow ID="ID_ALLOW_A_A40" FriendlyName="C:\Program Files\git\usr\bin\setmetamode.exe Hash Page Sha256" Hash="E837AC5FF4E51C63969DB2801AB4F0D5C964B079607FD39134719428CD097177" />
    <Allow ID="ID_ALLOW_A_A41" FriendlyName="C:\Program Files\git\usr\bin\sexp-conv.exe Hash Sha1" Hash="043AA78686A2BBD7FBC07F69A7AD123C2AE9DDD7" />
    <Allow ID="ID_ALLOW_A_A42" FriendlyName="C:\Program Files\git\usr\bin\sexp-conv.exe Hash Sha256" Hash="AD2C849060E7A5D7B085C54E80B7FE7BF6638907F26E8B0EBE5410F367E5D802" />
    <Allow ID="ID_ALLOW_A_A43" FriendlyName="C:\Program Files\git\usr\bin\sexp-conv.exe Hash Page Sha1" Hash="67B78925C7DD723FA72D7FD042DB2676826091F6" />
    <Allow ID="ID_ALLOW_A_A44" FriendlyName="C:\Program Files\git\usr\bin\sexp-conv.exe Hash Page Sha256" Hash="5402C3D0DA07AE4E56995C995F1F6FE5BAFC947ADFD0C258B150CEB4EE29155E" />
    <Allow ID="ID_ALLOW_A_A45" FriendlyName="C:\Program Files\git\usr\bin\sftp.exe Hash Sha1" Hash="D63C5F2A5C2B9CAD74E74E58E84C36E6FC93B76A" />
    <Allow ID="ID_ALLOW_A_A46" FriendlyName="C:\Program Files\git\usr\bin\sftp.exe Hash Sha256" Hash="05D6B94C41760B54DB0ADF6A4BB988C3FCAE1E4E9E0ED3E4921A28D336DD22AA" />
    <Allow ID="ID_ALLOW_A_A47" FriendlyName="C:\Program Files\git\usr\bin\sftp.exe Hash Page Sha1" Hash="F438DDB97FF112BC875143594BE34E5E9F130774" />
    <Allow ID="ID_ALLOW_A_A48" FriendlyName="C:\Program Files\git\usr\bin\sftp.exe Hash Page Sha256" Hash="5D9B697AE8BEFAE5BB07E844FC3687FD4853A643EAB4DF7F800942BFB2BF0284" />
    <Allow ID="ID_ALLOW_A_A49" FriendlyName="C:\Program Files\git\usr\bin\sha1sum.exe Hash Sha1" Hash="4D3F27797F38324617004C50BF8F8C130002D6A1" />
    <Allow ID="ID_ALLOW_A_A4A" FriendlyName="C:\Program Files\git\usr\bin\sha1sum.exe Hash Sha256" Hash="807A594BE52FC2AED6F71143029A3FB1AEA253B30652EFA612491A016723002B" />
    <Allow ID="ID_ALLOW_A_A4B" FriendlyName="C:\Program Files\git\usr\bin\sha1sum.exe Hash Page Sha1" Hash="F09708867667C7B50E493CEBA54461555B335973" />
    <Allow ID="ID_ALLOW_A_A4C" FriendlyName="C:\Program Files\git\usr\bin\sha1sum.exe Hash Page Sha256" Hash="B6DAD32F6588164CD97192CE32D5BC2F1FB8A48B9FCF006E1E5C897FEF725F0E" />
    <Allow ID="ID_ALLOW_A_A4D" FriendlyName="C:\Program Files\git\usr\bin\sha224sum.exe Hash Sha1" Hash="8962CB9601565E117D0F3679A1E32E84EB4DCCAC" />
    <Allow ID="ID_ALLOW_A_A4E" FriendlyName="C:\Program Files\git\usr\bin\sha224sum.exe Hash Sha256" Hash="C48049607E479887717BE6D14289A5B9912783060628E879A45CF89A9B5CB2AE" />
    <Allow ID="ID_ALLOW_A_A4F" FriendlyName="C:\Program Files\git\usr\bin\sha224sum.exe Hash Page Sha1" Hash="E63F6E437EA79C6DA8D690B47811FC820B3F4FEF" />
    <Allow ID="ID_ALLOW_A_A50" FriendlyName="C:\Program Files\git\usr\bin\sha224sum.exe Hash Page Sha256" Hash="73BE74AAEA1D47EA981058C9EF0D0C3BB6C1521B20E24C0344F465A63D5FA32C" />
    <Allow ID="ID_ALLOW_A_A51" FriendlyName="C:\Program Files\git\usr\bin\sha256sum.exe Hash Sha1" Hash="C308FD0BB5E036A90C2792EBE93A90F534553EC3" />
    <Allow ID="ID_ALLOW_A_A52" FriendlyName="C:\Program Files\git\usr\bin\sha256sum.exe Hash Sha256" Hash="D7588DAD71996AF3BF85010009EBFC3872B6DA9ACE321691533AAADF217D21E8" />
    <Allow ID="ID_ALLOW_A_A53" FriendlyName="C:\Program Files\git\usr\bin\sha384sum.exe Hash Sha1" Hash="97053E292DBA1AC338226D490C90626C19194316" />
    <Allow ID="ID_ALLOW_A_A54" FriendlyName="C:\Program Files\git\usr\bin\sha384sum.exe Hash Sha256" Hash="88C060EB7E40DCF0A8FFDF0F181C6B75C8221C32ACAFC5237B34AB2F2D7DC53D" />
    <Allow ID="ID_ALLOW_A_A55" FriendlyName="C:\Program Files\git\usr\bin\sha384sum.exe Hash Page Sha1" Hash="7C4E21C3EF4A9265869C9AC3FE680D97C6A6C608" />
    <Allow ID="ID_ALLOW_A_A56" FriendlyName="C:\Program Files\git\usr\bin\sha384sum.exe Hash Page Sha256" Hash="2D7B5EC62F40C21295FE1E28160189CE67179EDA4310A2AA8F6E5268210CDBB6" />
    <Allow ID="ID_ALLOW_A_A57" FriendlyName="C:\Program Files\git\usr\bin\sha512sum.exe Hash Sha1" Hash="311BF19DCDD7EFFD1F0ED50E3CF6B16B68A39851" />
    <Allow ID="ID_ALLOW_A_A58" FriendlyName="C:\Program Files\git\usr\bin\sha512sum.exe Hash Sha256" Hash="925C35EC4E2175FD893957FF420A1D9C11AC747BB9DE700F87FB9665F449ECD3" />
    <Allow ID="ID_ALLOW_A_A59" FriendlyName="C:\Program Files\git\usr\bin\shred.exe Hash Sha1" Hash="0189E03638603D42A5F892000615F3B8B3C46A97" />
    <Allow ID="ID_ALLOW_A_A5A" FriendlyName="C:\Program Files\git\usr\bin\shred.exe Hash Sha256" Hash="6580D63755BFCCDE96D9FBF1911BCE80DFD11381F22A98FCA76DBA6CB259238E" />
    <Allow ID="ID_ALLOW_A_A5B" FriendlyName="C:\Program Files\git\usr\bin\shred.exe Hash Page Sha1" Hash="2C059149108D92CF4763C4302D9C2F1D0C773131" />
    <Allow ID="ID_ALLOW_A_A5C" FriendlyName="C:\Program Files\git\usr\bin\shred.exe Hash Page Sha256" Hash="4ADE9161281D806186CD31D1A0C8139CBBEB96214116589FD5AF9A312025C5E0" />
    <Allow ID="ID_ALLOW_A_A5D" FriendlyName="C:\Program Files\git\usr\bin\shuf.exe Hash Sha1" Hash="5B894197EF87B5C34969609A2E33E67CEA54A685" />
    <Allow ID="ID_ALLOW_A_A5E" FriendlyName="C:\Program Files\git\usr\bin\shuf.exe Hash Sha256" Hash="8626D5119B6768CB7115FCB07E93004DC680B446B9FF5E34E25DD3B75B9F2A18" />
    <Allow ID="ID_ALLOW_A_A5F" FriendlyName="C:\Program Files\git\usr\bin\shuf.exe Hash Page Sha1" Hash="8AC581E9845D67925B4AE2A2EE9082691469F846" />
    <Allow ID="ID_ALLOW_A_A60" FriendlyName="C:\Program Files\git\usr\bin\shuf.exe Hash Page Sha256" Hash="F6D53877B81441A92AA8FF064D2FAB68F3C01810FF126355921D3C9F02A68E97" />
    <Allow ID="ID_ALLOW_A_A61" FriendlyName="C:\Program Files\git\usr\bin\sleep.exe Hash Sha1" Hash="693C759C586AF6FB2C57062EB413049234867F54" />
    <Allow ID="ID_ALLOW_A_A62" FriendlyName="C:\Program Files\git\usr\bin\sleep.exe Hash Sha256" Hash="83A456FC75BA645726B7D82A2A0FFEBE8129FC2A2C96735BD99AE48B4FCBD8F7" />
    <Allow ID="ID_ALLOW_A_A63" FriendlyName="C:\Program Files\git\usr\bin\sleep.exe Hash Page Sha1" Hash="CE6F57A1DBAEC3BE2C17C84756B685DBDE79AF24" />
    <Allow ID="ID_ALLOW_A_A64" FriendlyName="C:\Program Files\git\usr\bin\sleep.exe Hash Page Sha256" Hash="607AD35B971C96DB5AFDB3D4622A5C27A6E26137F746F9F1312DE1342D785C8A" />
    <Allow ID="ID_ALLOW_A_A65" FriendlyName="C:\Program Files\git\usr\bin\sort.exe Hash Sha1" Hash="92828A8F1EBAA0A6D8D46C60E1EA29E008507F84" />
    <Allow ID="ID_ALLOW_A_A66" FriendlyName="C:\Program Files\git\usr\bin\sort.exe Hash Sha256" Hash="E8AB00260CC8A935D94AAA2B2D9753A1A64A3A297A602B01A028038DF0FD63FD" />
    <Allow ID="ID_ALLOW_A_A67" FriendlyName="C:\Program Files\git\usr\bin\sort.exe Hash Page Sha1" Hash="CD61B94FB3D618DC55B2F4E5B2B121575C85F6C7" />
    <Allow ID="ID_ALLOW_A_A68" FriendlyName="C:\Program Files\git\usr\bin\sort.exe Hash Page Sha256" Hash="590C08F3CCC9C9B637AAE45497847F5D71CA4651E60B45CAEF176E1ADE2A690D" />
    <Allow ID="ID_ALLOW_A_A69" FriendlyName="C:\Program Files\git\usr\bin\split.exe Hash Sha1" Hash="FB0EAD1016DCB126D91CFDC1581C514A82EA8A02" />
    <Allow ID="ID_ALLOW_A_A6A" FriendlyName="C:\Program Files\git\usr\bin\split.exe Hash Sha256" Hash="582774C6BFE636E520B1626B7A35460A809402DD1AFB5712231814F6A91DBCEA" />
    <Allow ID="ID_ALLOW_A_A6B" FriendlyName="C:\Program Files\git\usr\bin\split.exe Hash Page Sha1" Hash="35D6CC6DF26E022B2F7E880151B25DF71AA61E6C" />
    <Allow ID="ID_ALLOW_A_A6C" FriendlyName="C:\Program Files\git\usr\bin\split.exe Hash Page Sha256" Hash="4D15E28756C267D2000ED7337700FDF0E11BABF76EF5C309AFF8A14B379C3DDF" />
    <Allow ID="ID_ALLOW_A_A6D" FriendlyName="C:\Program Files\git\usr\bin\ssh-add.exe Hash Sha1" Hash="8FADD2666B8F347B7FE5196783D4BDF75200DF1B" />
    <Allow ID="ID_ALLOW_A_A6E" FriendlyName="C:\Program Files\git\usr\bin\ssh-add.exe Hash Sha256" Hash="AE60282E7F65E141798C5E17DD0DB749ECBCEC2A3348C9219D4398F57C399113" />
    <Allow ID="ID_ALLOW_A_A6F" FriendlyName="C:\Program Files\git\usr\bin\ssh-add.exe Hash Page Sha1" Hash="A390AF1B1DC2ACB323E087DFD9B6EB6BD0A5C1DF" />
    <Allow ID="ID_ALLOW_A_A70" FriendlyName="C:\Program Files\git\usr\bin\ssh-add.exe Hash Page Sha256" Hash="9CDBC8D62868A9B8C80679673B08603E80BFE7F50538C4703FF16DC8F7C9FF5F" />
    <Allow ID="ID_ALLOW_A_A71" FriendlyName="C:\Program Files\git\usr\bin\ssh-agent.exe Hash Sha1" Hash="1195168F9A6B946756A2D1069BE5208893E14ED3" />
    <Allow ID="ID_ALLOW_A_A72" FriendlyName="C:\Program Files\git\usr\bin\ssh-agent.exe Hash Sha256" Hash="FB43A2D1099D15E18E0B1FE58C558556F99122F5784D0BD683427A35D4346905" />
    <Allow ID="ID_ALLOW_A_A73" FriendlyName="C:\Program Files\git\usr\bin\ssh-agent.exe Hash Page Sha1" Hash="315C5E10BDCA7121411B08EAC55A61ACF0D71E2F" />
    <Allow ID="ID_ALLOW_A_A74" FriendlyName="C:\Program Files\git\usr\bin\ssh-agent.exe Hash Page Sha256" Hash="808EE298D8BC0BA3EFAB61451898534E1F48ABBA33961E91BDB2E6F287318A98" />
    <Allow ID="ID_ALLOW_A_A75" FriendlyName="C:\Program Files\git\usr\bin\ssh-keygen.exe Hash Sha1" Hash="9D0B8E722443783801949A17AEC0B3DB40EA0AD8" />
    <Allow ID="ID_ALLOW_A_A76" FriendlyName="C:\Program Files\git\usr\bin\ssh-keygen.exe Hash Sha256" Hash="B5ACD70F79349BF07A617C1C9263D8EC73BA99BCA99DE88F1AB1163E6201B213" />
    <Allow ID="ID_ALLOW_A_A77" FriendlyName="C:\Program Files\git\usr\bin\ssh-keygen.exe Hash Page Sha1" Hash="B5205C98561632DCBE4961638A60E2D39FCB0EAD" />
    <Allow ID="ID_ALLOW_A_A78" FriendlyName="C:\Program Files\git\usr\bin\ssh-keygen.exe Hash Page Sha256" Hash="D9BDDDF29D322851C73F3127394A4D23F59304D65EB62433507DBC3A3C0F72FA" />
    <Allow ID="ID_ALLOW_A_A79" FriendlyName="C:\Program Files\git\usr\bin\ssh-keyscan.exe Hash Sha1" Hash="2B44FFFFBFDA1ACF5C295F12E2FFBE3E5A95A8FB" />
    <Allow ID="ID_ALLOW_A_A7A" FriendlyName="C:\Program Files\git\usr\bin\ssh-keyscan.exe Hash Sha256" Hash="C02D3FDECEB2554F0923424052EF2ECB973DC37E2D2F23647D253DA2873C9BE9" />
    <Allow ID="ID_ALLOW_A_A7B" FriendlyName="C:\Program Files\git\usr\bin\ssh-keyscan.exe Hash Page Sha1" Hash="71F31B390E667154CD71CDC34A2D02C1D8E2F38C" />
    <Allow ID="ID_ALLOW_A_A7C" FriendlyName="C:\Program Files\git\usr\bin\ssh-keyscan.exe Hash Page Sha256" Hash="2D85D4FC5840EE96028D7B6B0C77CFD09001CF8DADA24ECF80186B2F86CA228A" />
    <Allow ID="ID_ALLOW_A_A7D" FriendlyName="C:\Program Files\git\usr\bin\ssh-pageant.exe Hash Sha1" Hash="B87AE6FBA8379BF2771DB902124B0F373D7C41C4" />
    <Allow ID="ID_ALLOW_A_A7E" FriendlyName="C:\Program Files\git\usr\bin\ssh-pageant.exe Hash Sha256" Hash="38F7CF1CB5919A945B0187BB6F09C46AEC033488055BC618CC81950747408B90" />
    <Allow ID="ID_ALLOW_A_A7F" FriendlyName="C:\Program Files\git\usr\bin\ssh-pageant.exe Hash Page Sha1" Hash="B95F18BE6571BD7A9BBCCE68103C644837968C90" />
    <Allow ID="ID_ALLOW_A_A80" FriendlyName="C:\Program Files\git\usr\bin\ssh-pageant.exe Hash Page Sha256" Hash="6A24FC4E92386E9E84288326AFF2B03714B783FE03EA7429B4C2D85575764D34" />
    <Allow ID="ID_ALLOW_A_A81" FriendlyName="C:\Program Files\git\usr\bin\ssh.exe Hash Sha1" Hash="8F5C873CC4BA2224516D69DD3B4A2613AB263664" />
    <Allow ID="ID_ALLOW_A_A82" FriendlyName="C:\Program Files\git\usr\bin\ssh.exe Hash Sha256" Hash="35C78B5CCA02216FE8108D17F2F38A8A9FB8E25CBC233D00EAB76CB664EF1F8A" />
    <Allow ID="ID_ALLOW_A_A83" FriendlyName="C:\Program Files\git\usr\bin\ssh.exe Hash Page Sha1" Hash="3EC6DA50002B59046F38AB03DD72825E089D50ED" />
    <Allow ID="ID_ALLOW_A_A84" FriendlyName="C:\Program Files\git\usr\bin\ssh.exe Hash Page Sha256" Hash="9EF5BCF915A911BA14597E7D8AB96F5F329207014FB9A837B6B27390EB9B7026" />
    <Allow ID="ID_ALLOW_A_A85" FriendlyName="C:\Program Files\git\usr\bin\sshd.exe Hash Sha1" Hash="8F0A776D6C892A7F4E73DA60F9CB7F253AC106C2" />
    <Allow ID="ID_ALLOW_A_A86" FriendlyName="C:\Program Files\git\usr\bin\sshd.exe Hash Sha256" Hash="2D9A62E09D8AA3EE57E2F3112F95D52934CF12870DF88C124E9B1981AC5B37E3" />
    <Allow ID="ID_ALLOW_A_A87" FriendlyName="C:\Program Files\git\usr\bin\sshd.exe Hash Page Sha1" Hash="806007EF2C16238D833F06CEA526C4174CFBAF3A" />
    <Allow ID="ID_ALLOW_A_A88" FriendlyName="C:\Program Files\git\usr\bin\sshd.exe Hash Page Sha256" Hash="158B748AC08DEE75B7FB699E71777153322EC29F4BC49C4DDFB9EEEA804F5DE0" />
    <Allow ID="ID_ALLOW_A_A89" FriendlyName="C:\Program Files\git\usr\bin\ssp.exe Hash Sha1" Hash="A38CB42AFAD7626B812FACB97AC2F63227668A62" />
    <Allow ID="ID_ALLOW_A_A8A" FriendlyName="C:\Program Files\git\usr\bin\ssp.exe Hash Sha256" Hash="B1972AA21CE06292F832B5AC96D987A9327822A1937346044EB69AB6A070ED0E" />
    <Allow ID="ID_ALLOW_A_A8B" FriendlyName="C:\Program Files\git\usr\bin\ssp.exe Hash Page Sha1" Hash="4A8D8F003260D6BAA9A9B39E789A81E76021A3ED" />
    <Allow ID="ID_ALLOW_A_A8C" FriendlyName="C:\Program Files\git\usr\bin\ssp.exe Hash Page Sha256" Hash="474A5DB2DDFED580539C70745DB7E652F928AD99DACDEDFA1E45AE640556FBD1" />
    <Allow ID="ID_ALLOW_A_A8D" FriendlyName="C:\Program Files\git\usr\bin\stat.exe Hash Sha1" Hash="766AA58C080CBE89309C46199C2862C302529F3B" />
    <Allow ID="ID_ALLOW_A_A8E" FriendlyName="C:\Program Files\git\usr\bin\stat.exe Hash Sha256" Hash="08ED94880945814374F1CA6A5F1783A5B39C685BF4FCFB1E310A387A140C59FC" />
    <Allow ID="ID_ALLOW_A_A8F" FriendlyName="C:\Program Files\git\usr\bin\stat.exe Hash Page Sha1" Hash="4C199CA4DF11BF84991962944515B52736AD53EC" />
    <Allow ID="ID_ALLOW_A_A90" FriendlyName="C:\Program Files\git\usr\bin\stat.exe Hash Page Sha256" Hash="A5012C803DD2B9F4DBBFE8426C56E3267522ADA2E44E96FADFE175F853C1E2DA" />
    <Allow ID="ID_ALLOW_A_A91" FriendlyName="C:\Program Files\git\usr\bin\strace.exe Hash Sha1" Hash="23DCC4A9CFCB8C6B468384E726FD7B1BD0D829F1" />
    <Allow ID="ID_ALLOW_A_A92" FriendlyName="C:\Program Files\git\usr\bin\strace.exe Hash Sha256" Hash="D4492C78CBE0A4664E13A8FE5C89B1CAEF1DE00A3B4A2645195A872293BE8A7B" />
    <Allow ID="ID_ALLOW_A_A93" FriendlyName="C:\Program Files\git\usr\bin\strace.exe Hash Page Sha1" Hash="46494870CB0833138BA337DDDF31FF5D1A3E6408" />
    <Allow ID="ID_ALLOW_A_A94" FriendlyName="C:\Program Files\git\usr\bin\strace.exe Hash Page Sha256" Hash="846EBA80F2C575F0066C96E1667CC8C6CDC6DD4B68CD8C12049C8F6A7E2C6619" />
    <Allow ID="ID_ALLOW_A_A95" FriendlyName="C:\Program Files\git\usr\bin\stty.exe Hash Sha1" Hash="78FFCD0251348ED637B9D7045C14125F18D548DA" />
    <Allow ID="ID_ALLOW_A_A96" FriendlyName="C:\Program Files\git\usr\bin\stty.exe Hash Sha256" Hash="3366021E30AA0A2D43679946D1421D80C88BA086FFA25358B43ACDF873CA6ABF" />
    <Allow ID="ID_ALLOW_A_A97" FriendlyName="C:\Program Files\git\usr\bin\stty.exe Hash Page Sha1" Hash="9C876B0E5B32066FCAC9810D96FE72F9077240E3" />
    <Allow ID="ID_ALLOW_A_A98" FriendlyName="C:\Program Files\git\usr\bin\stty.exe Hash Page Sha256" Hash="D4BC7136EEAB834980F9EAE2FA1D88E1C3DEFAD6512AFF5AFDCD65B156A30508" />
    <Allow ID="ID_ALLOW_A_A99" FriendlyName="C:\Program Files\git\usr\bin\sum.exe Hash Sha1" Hash="C826D8917D226A7D67850A9C222398C694EE54CB" />
    <Allow ID="ID_ALLOW_A_A9A" FriendlyName="C:\Program Files\git\usr\bin\sum.exe Hash Sha256" Hash="6F748F3039CB021885BD15CA55E53F17B2ED8BE3EF6302CFB07D575998A6E95D" />
    <Allow ID="ID_ALLOW_A_A9B" FriendlyName="C:\Program Files\git\usr\bin\sum.exe Hash Page Sha1" Hash="93ACF089209F0B5CFE3164BBDF58BC96E7FB30B2" />
    <Allow ID="ID_ALLOW_A_A9C" FriendlyName="C:\Program Files\git\usr\bin\sum.exe Hash Page Sha256" Hash="12E241ECB708905A34CBE581A176411E03A0998A4677A521176109932D75E8A6" />
    <Allow ID="ID_ALLOW_A_A9D" FriendlyName="C:\Program Files\git\usr\bin\sync.exe Hash Sha1" Hash="7DA46D19B7744DD5461D7F09810EDB6A2BF0A86B" />
    <Allow ID="ID_ALLOW_A_A9E" FriendlyName="C:\Program Files\git\usr\bin\sync.exe Hash Sha256" Hash="EBCE7264416A013F528E67D6E2F67E8042DECED80B882A09B8A3A33C84C9365A" />
    <Allow ID="ID_ALLOW_A_A9F" FriendlyName="C:\Program Files\git\usr\bin\sync.exe Hash Page Sha1" Hash="B00F9EE7773C1F7C7DD0B67E1C383EB848E3F38E" />
    <Allow ID="ID_ALLOW_A_AA0" FriendlyName="C:\Program Files\git\usr\bin\sync.exe Hash Page Sha256" Hash="49785AEE2BA4D14FD55224387CB39FCFFC5A9BF24B66388B77F3BA0125B9DEDF" />
    <Allow ID="ID_ALLOW_A_AA1" FriendlyName="C:\Program Files\git\usr\bin\tabs.exe Hash Sha1" Hash="83B78A1238E441E786FC9958E818D28ADA35CE0F" />
    <Allow ID="ID_ALLOW_A_AA2" FriendlyName="C:\Program Files\git\usr\bin\tabs.exe Hash Sha256" Hash="AFEF8748FA3BDCF472B745DFBCE978B4AD08B4269A48C010382674FEA3F21A9D" />
    <Allow ID="ID_ALLOW_A_AA3" FriendlyName="C:\Program Files\git\usr\bin\tabs.exe Hash Page Sha1" Hash="9B16CEC147F5713F0E00F43B0A22A5B5C62FEE4B" />
    <Allow ID="ID_ALLOW_A_AA4" FriendlyName="C:\Program Files\git\usr\bin\tabs.exe Hash Page Sha256" Hash="6E13A0252E31F535C8AA507995B8B3936B45853777A60CA684CAC66003D15CBC" />
    <Allow ID="ID_ALLOW_A_AA5" FriendlyName="C:\Program Files\git\usr\bin\tac.exe Hash Sha1" Hash="6E4E2EBE6534137C43A4BD4E64CA6EEF9C294D7B" />
    <Allow ID="ID_ALLOW_A_AA6" FriendlyName="C:\Program Files\git\usr\bin\tac.exe Hash Sha256" Hash="4D68ACFC81F9818D4A2293A929309DD72ED462A512699ED465EEEE78D2FC0335" />
    <Allow ID="ID_ALLOW_A_AA7" FriendlyName="C:\Program Files\git\usr\bin\tac.exe Hash Page Sha1" Hash="E3FF64919131BF80F4BD20AECDA4AF263F7E0353" />
    <Allow ID="ID_ALLOW_A_AA8" FriendlyName="C:\Program Files\git\usr\bin\tac.exe Hash Page Sha256" Hash="384CF5BB9686C070D15B212B32D0D6DF902B5EA3A553B791B227E4E6E58A3DB3" />
    <Allow ID="ID_ALLOW_A_AA9" FriendlyName="C:\Program Files\git\usr\bin\tail.exe Hash Sha1" Hash="492B15130AE9C21287F86A4D44CE6ED1E81CF830" />
    <Allow ID="ID_ALLOW_A_AAA" FriendlyName="C:\Program Files\git\usr\bin\tail.exe Hash Sha256" Hash="02DD2DDEB4DE17A844791D4AB45B523B789B31371310884832865B140558B4C8" />
    <Allow ID="ID_ALLOW_A_AAB" FriendlyName="C:\Program Files\git\usr\bin\tail.exe Hash Page Sha1" Hash="BDCA3F509042843EA5B128CBBC430E3B4A2203E7" />
    <Allow ID="ID_ALLOW_A_AAC" FriendlyName="C:\Program Files\git\usr\bin\tail.exe Hash Page Sha256" Hash="9097D3EFCF17B2F8ED3E211C9D96C17A024FC4B2AE2CAA8551F3EFC557F743F5" />
    <Allow ID="ID_ALLOW_A_AAD" FriendlyName="C:\Program Files\git\usr\bin\tar.exe Hash Sha1" Hash="861273A4D2B5AF6521A0747340B1AC173BD633DD" />
    <Allow ID="ID_ALLOW_A_AAE" FriendlyName="C:\Program Files\git\usr\bin\tar.exe Hash Sha256" Hash="2828BA3B6C376B2708DB7B9EC77777B1837228613AE1B6CADF985176C8D3740A" />
    <Allow ID="ID_ALLOW_A_AAF" FriendlyName="C:\Program Files\git\usr\bin\tar.exe Hash Page Sha1" Hash="21DA6F0419427B4BB5D6008B7D93EC675B98DBD5" />
    <Allow ID="ID_ALLOW_A_AB0" FriendlyName="C:\Program Files\git\usr\bin\tar.exe Hash Page Sha256" Hash="347F7866A7A382C585CAB205FD2EB782C49AFAB570AB1411B83D191629E52911" />
    <Allow ID="ID_ALLOW_A_AB1" FriendlyName="C:\Program Files\git\usr\bin\tclsh.exe Hash Sha1" Hash="A34B09F481B0604C7A9380202D44C8F5C2B30541" />
    <Allow ID="ID_ALLOW_A_AB2" FriendlyName="C:\Program Files\git\usr\bin\tclsh.exe Hash Sha256" Hash="92C7280FB0E88479C7E0CF9EA25CE9288B7870CDA938664E2D0CA457C016216A" />
    <Allow ID="ID_ALLOW_A_AB3" FriendlyName="C:\Program Files\git\usr\bin\tclsh.exe Hash Page Sha1" Hash="C734C6810BFEA595CE18240271F5B9E235BB8000" />
    <Allow ID="ID_ALLOW_A_AB4" FriendlyName="C:\Program Files\git\usr\bin\tclsh.exe Hash Page Sha256" Hash="18B07DDBE7D131CF214CE0635C303FD14220BF499A8E0080A70197057E3D42D6" />
    <Allow ID="ID_ALLOW_A_AB5" FriendlyName="C:\Program Files\git\usr\bin\tee.exe Hash Sha1" Hash="6968A2E6A05A861D0AD50E5F13BE74907DBC80A1" />
    <Allow ID="ID_ALLOW_A_AB6" FriendlyName="C:\Program Files\git\usr\bin\tee.exe Hash Sha256" Hash="E3B08ECDF6B78A12136C618F9054E763C781F28CA1B69E1663BDF8AFCCE50245" />
    <Allow ID="ID_ALLOW_A_AB7" FriendlyName="C:\Program Files\git\usr\bin\tee.exe Hash Page Sha1" Hash="A00690D72CCE12E6C5A252CD625EBF2C885E4605" />
    <Allow ID="ID_ALLOW_A_AB8" FriendlyName="C:\Program Files\git\usr\bin\tee.exe Hash Page Sha256" Hash="B2EA7EDC58BE3E4C8211260FE80B07C1E49386C8BAF5697E2135604C97F0CD1D" />
    <Allow ID="ID_ALLOW_A_AB9" FriendlyName="C:\Program Files\git\usr\bin\test.exe Hash Sha1" Hash="ADC1D8F88244F5A25D7DAD1FE5059819FE6A18DE" />
    <Allow ID="ID_ALLOW_A_ABA" FriendlyName="C:\Program Files\git\usr\bin\test.exe Hash Sha256" Hash="2B6C0CEEC9A7AE6270802DA8B042CF0D53C0BCDF81BB7D0DFE0E825BFBD5ABDC" />
    <Allow ID="ID_ALLOW_A_ABB" FriendlyName="C:\Program Files\git\usr\bin\test.exe Hash Page Sha1" Hash="11F8CF4BFB340EE104C0A8D229DD45D6AC771BDA" />
    <Allow ID="ID_ALLOW_A_ABC" FriendlyName="C:\Program Files\git\usr\bin\test.exe Hash Page Sha256" Hash="7C9EA0E85EAC23A7DD3F497BE0A305AEC45404099479BE8A165054C826A64554" />
    <Allow ID="ID_ALLOW_A_ABD" FriendlyName="C:\Program Files\git\usr\bin\tig.exe Hash Sha1" Hash="5D3502CA3410477C70CE2D2F20EDA96C6BEA98A3" />
    <Allow ID="ID_ALLOW_A_ABE" FriendlyName="C:\Program Files\git\usr\bin\tig.exe Hash Sha256" Hash="EB0B16E7482C5E0196B43BC82FDAE366684BDFC5B42A57A7C36FCBBAE439FF12" />
    <Allow ID="ID_ALLOW_A_ABF" FriendlyName="C:\Program Files\git\usr\bin\tig.exe Hash Page Sha1" Hash="2A1CA0AF7DA18EFF0B5411B3CB9095DF52DF9EAF" />
    <Allow ID="ID_ALLOW_A_AC0" FriendlyName="C:\Program Files\git\usr\bin\tig.exe Hash Page Sha256" Hash="846090585A3AC511C8C0E13A2D69B00DF7DB56F2CA67509FA3E434EFE4B4A1AF" />
    <Allow ID="ID_ALLOW_A_AC1" FriendlyName="C:\Program Files\git\usr\bin\timeout.exe Hash Sha1" Hash="6769D52518456A9E0FEE1CAAC384C1B94E767303" />
    <Allow ID="ID_ALLOW_A_AC2" FriendlyName="C:\Program Files\git\usr\bin\timeout.exe Hash Sha256" Hash="5F41F974B254A866FBF879035719A7E0C63E21E7C67A8DBAD628CD1094B1FBE2" />
    <Allow ID="ID_ALLOW_A_AC3" FriendlyName="C:\Program Files\git\usr\bin\timeout.exe Hash Page Sha1" Hash="101FC48E84A57A067465A89B978D45E4D9CC3179" />
    <Allow ID="ID_ALLOW_A_AC4" FriendlyName="C:\Program Files\git\usr\bin\timeout.exe Hash Page Sha256" Hash="00A30C55E68946BE8C5C8A2023996E6ACE47E81D4F76074289E26C0194F86BDE" />
    <Allow ID="ID_ALLOW_A_AC5" FriendlyName="C:\Program Files\git\usr\bin\toe.exe Hash Sha1" Hash="5F46D87A14F3E986A74ADC8E5E7D06A8D7292165" />
    <Allow ID="ID_ALLOW_A_AC6" FriendlyName="C:\Program Files\git\usr\bin\toe.exe Hash Sha256" Hash="077236AF6B31406E8F3393FA789852014935F31754FAA2F9AE129EE9A09D0FE2" />
    <Allow ID="ID_ALLOW_A_AC7" FriendlyName="C:\Program Files\git\usr\bin\toe.exe Hash Page Sha1" Hash="CAD577FE95A91C0C87E9E854743C8D177604B7A8" />
    <Allow ID="ID_ALLOW_A_AC8" FriendlyName="C:\Program Files\git\usr\bin\toe.exe Hash Page Sha256" Hash="B37E09A5FE9B4C972C88A04ACDAEB36734CF3431E1A84D3E44D008F3A174443C" />
    <Allow ID="ID_ALLOW_A_AC9" FriendlyName="C:\Program Files\git\usr\bin\touch.exe Hash Sha1" Hash="EE5474E0D75C1207F7635E2D07D9C76E84E5B5E7" />
    <Allow ID="ID_ALLOW_A_ACA" FriendlyName="C:\Program Files\git\usr\bin\touch.exe Hash Sha256" Hash="063D9C85E7E8331848C3FD6578401143DF1F7849992048DB91ECF8B8AD3290F9" />
    <Allow ID="ID_ALLOW_A_ACB" FriendlyName="C:\Program Files\git\usr\bin\touch.exe Hash Page Sha1" Hash="CC961F08D1BD050985AA636E11BE54255D11706B" />
    <Allow ID="ID_ALLOW_A_ACC" FriendlyName="C:\Program Files\git\usr\bin\touch.exe Hash Page Sha256" Hash="CAF59240C750C3DE62F6A4971D81FA41DFFAB2A6AC02C65DDBF584ADFD66BB27" />
    <Allow ID="ID_ALLOW_A_ACD" FriendlyName="C:\Program Files\git\usr\bin\tput.exe Hash Sha1" Hash="CD5456BE28AE415517A73F6E594C36ECE2912036" />
    <Allow ID="ID_ALLOW_A_ACE" FriendlyName="C:\Program Files\git\usr\bin\tput.exe Hash Sha256" Hash="DEB86424AB28B8FA17D9CC60FB61C2872A0334B9C1E72BF5AB34C377CBDCF8AF" />
    <Allow ID="ID_ALLOW_A_ACF" FriendlyName="C:\Program Files\git\usr\bin\tput.exe Hash Page Sha1" Hash="E33E6FDE70D3ECA942B23295068EF50FF1BBF49A" />
    <Allow ID="ID_ALLOW_A_AD0" FriendlyName="C:\Program Files\git\usr\bin\tput.exe Hash Page Sha256" Hash="00898FACD25D9AA816CB6A6F16745E68DEE605F008ABB2C4B35E35000FBD4694" />
    <Allow ID="ID_ALLOW_A_AD1" FriendlyName="C:\Program Files\git\usr\bin\tr.exe Hash Sha1" Hash="20B329B035FEF74B61988990E8B66F1D06F32E1A" />
    <Allow ID="ID_ALLOW_A_AD2" FriendlyName="C:\Program Files\git\usr\bin\tr.exe Hash Sha256" Hash="FCE0162CA83334983C541660A4C9C8F2BEB654FC2C31757FF8C8109013CEF5F0" />
    <Allow ID="ID_ALLOW_A_AD3" FriendlyName="C:\Program Files\git\usr\bin\tr.exe Hash Page Sha1" Hash="3D4BAFE6C87AA235088C2881F3809474A7E524FC" />
    <Allow ID="ID_ALLOW_A_AD4" FriendlyName="C:\Program Files\git\usr\bin\tr.exe Hash Page Sha256" Hash="774BB612A17D47C605C5685D45E7419970FA3F7BC25A66013736D4F8523195D1" />
    <Allow ID="ID_ALLOW_A_AD5" FriendlyName="C:\Program Files\git\usr\bin\true.exe Hash Sha1" Hash="9E84850B548D142493F820DB62256B6C36EB29EE" />
    <Allow ID="ID_ALLOW_A_AD6" FriendlyName="C:\Program Files\git\usr\bin\true.exe Hash Sha256" Hash="CA27F245DE025B66CE75D998445DFE779CBD707CEBE2FE3B8A3EEB5D7AC4D502" />
    <Allow ID="ID_ALLOW_A_AD7" FriendlyName="C:\Program Files\git\usr\bin\truncate.exe Hash Sha1" Hash="9DF053BD19D3469D29C46813FD9F7FB0FE216CFA" />
    <Allow ID="ID_ALLOW_A_AD8" FriendlyName="C:\Program Files\git\usr\bin\truncate.exe Hash Sha256" Hash="A944AAC9FFBB7BB75187D1B959F537BDDAA6694896AA759BAAC037F00C8CAE0F" />
    <Allow ID="ID_ALLOW_A_AD9" FriendlyName="C:\Program Files\git\usr\bin\truncate.exe Hash Page Sha1" Hash="D3A5912BD4A22CF08FFDFF06649BABB2FDE2C1ED" />
    <Allow ID="ID_ALLOW_A_ADA" FriendlyName="C:\Program Files\git\usr\bin\truncate.exe Hash Page Sha256" Hash="CBA078A14F9B49211A38F7B12C9AE0133CBC905FB8A12BF8D54880FD8F7687C2" />
    <Allow ID="ID_ALLOW_A_ADB" FriendlyName="C:\Program Files\git\usr\bin\trust.exe Hash Sha1" Hash="57EE51BFB71B84959B466F94B0D8CB1D08242409" />
    <Allow ID="ID_ALLOW_A_ADC" FriendlyName="C:\Program Files\git\usr\bin\trust.exe Hash Sha256" Hash="A90B1C6EFAC6F8B42EA5C7F92978C2A835C411E0E3318EA7677A332E941EFCE3" />
    <Allow ID="ID_ALLOW_A_ADD" FriendlyName="C:\Program Files\git\usr\bin\trust.exe Hash Page Sha1" Hash="B9CED408F13B31EF7EB300176FDB7348864E2AE9" />
    <Allow ID="ID_ALLOW_A_ADE" FriendlyName="C:\Program Files\git\usr\bin\trust.exe Hash Page Sha256" Hash="A752CD31BD25C35E1ABFB79C0543AF60F430B2740470AF8562D0F7386DD208D3" />
    <Allow ID="ID_ALLOW_A_ADF" FriendlyName="C:\Program Files\git\usr\bin\tsort.exe Hash Sha1" Hash="5FBC246F4C62998461683A8B1626C9526354A0D4" />
    <Allow ID="ID_ALLOW_A_AE0" FriendlyName="C:\Program Files\git\usr\bin\tsort.exe Hash Sha256" Hash="22D1C1CD1FB737DB8FEF73673ACDBEE9A07C48BCB7B7B135288EACE82EDEC441" />
    <Allow ID="ID_ALLOW_A_AE1" FriendlyName="C:\Program Files\git\usr\bin\tsort.exe Hash Page Sha1" Hash="E73BF3BB6A3C7236C60200132291ACF344B66C4E" />
    <Allow ID="ID_ALLOW_A_AE2" FriendlyName="C:\Program Files\git\usr\bin\tsort.exe Hash Page Sha256" Hash="E40FED2071D1CD3FD3F66784378F39C4506F00A7D1CCC1D5FC37FFB7B12F7D3A" />
    <Allow ID="ID_ALLOW_A_AE3" FriendlyName="C:\Program Files\git\usr\bin\tty.exe Hash Sha1" Hash="7164160FDE6FFD8FCF6B12936F1F0EB17E607D59" />
    <Allow ID="ID_ALLOW_A_AE4" FriendlyName="C:\Program Files\git\usr\bin\tty.exe Hash Sha256" Hash="41818C9B016EC78747957899D5BD0C7754AC86966177E6E3B02CA30C8324F7C0" />
    <Allow ID="ID_ALLOW_A_AE5" FriendlyName="C:\Program Files\git\usr\bin\tty.exe Hash Page Sha1" Hash="BBD4DC7F0EF55AAA7633FA3F105ED0006F6C143C" />
    <Allow ID="ID_ALLOW_A_AE6" FriendlyName="C:\Program Files\git\usr\bin\tty.exe Hash Page Sha256" Hash="9AE986AC259FED5D5E557FF87A7D045565155BFDC43B5D2A79212ACD361C40D0" />
    <Allow ID="ID_ALLOW_A_AE7" FriendlyName="C:\Program Files\git\usr\bin\tzset.exe Hash Sha1" Hash="4FDDD8AB1B73DA3075BA64B9A365C7754C260CA0" />
    <Allow ID="ID_ALLOW_A_AE8" FriendlyName="C:\Program Files\git\usr\bin\tzset.exe Hash Sha256" Hash="EA78DAC2EF4F9744AA8E1A683918B4F5102BEAAF38DDAAFE76511B2C7BD3311C" />
    <Allow ID="ID_ALLOW_A_AE9" FriendlyName="C:\Program Files\git\usr\bin\tzset.exe Hash Page Sha1" Hash="16F70B96898ABC62D04D84C9CF076F8091498EC4" />
    <Allow ID="ID_ALLOW_A_AEA" FriendlyName="C:\Program Files\git\usr\bin\tzset.exe Hash Page Sha256" Hash="41ED000314F45B1427ADE8059AE5FDE8C28FE73379D616B41589508F940A17EC" />
    <Allow ID="ID_ALLOW_A_AEB" FriendlyName="C:\Program Files\git\usr\bin\u2d.exe Hash Sha1" Hash="3AF5779A7B1D37A409D149DB1211BE3DCD7215B1" />
    <Allow ID="ID_ALLOW_A_AEC" FriendlyName="C:\Program Files\git\usr\bin\u2d.exe Hash Sha256" Hash="7448E1FB99D75E00AE06D260E81C9A342E12ED34D63526987B37CB65E1853B84" />
    <Allow ID="ID_ALLOW_A_AED" FriendlyName="C:\Program Files\git\usr\bin\u2d.exe Hash Page Sha1" Hash="E6DFD13F7EBAD0CE6CC7D3610481DF2C9784991E" />
    <Allow ID="ID_ALLOW_A_AEE" FriendlyName="C:\Program Files\git\usr\bin\u2d.exe Hash Page Sha256" Hash="CF343DA09C592D3E97B0346AF708352782C7BB965959617C93032CBC7CEC8745" />
    <Allow ID="ID_ALLOW_A_AEF" FriendlyName="C:\Program Files\git\usr\bin\umount.exe Hash Sha1" Hash="6A946CCBB41E9228FD9A3BBFD274C9E574A2AB1B" />
    <Allow ID="ID_ALLOW_A_AF0" FriendlyName="C:\Program Files\git\usr\bin\umount.exe Hash Sha256" Hash="522E04B91C63C77EEB90B1183AFA66BD0BD3F7C8A48A76EAAF5B24BFAC1D68B4" />
    <Allow ID="ID_ALLOW_A_AF1" FriendlyName="C:\Program Files\git\usr\bin\umount.exe Hash Page Sha1" Hash="1652BC2FD2CCC8C738A684D816D580CEB5338DEA" />
    <Allow ID="ID_ALLOW_A_AF2" FriendlyName="C:\Program Files\git\usr\bin\umount.exe Hash Page Sha256" Hash="5F95664A8DDC1F7A85009E070AD88A7E35355D9FE4A8EDBE62738F97ACA25AFA" />
    <Allow ID="ID_ALLOW_A_AF3" FriendlyName="C:\Program Files\git\usr\bin\uname.exe Hash Sha1" Hash="CE7C0A74F367C44D0A64DA04F3A41D0BDFB0CC10" />
    <Allow ID="ID_ALLOW_A_AF4" FriendlyName="C:\Program Files\git\usr\bin\uname.exe Hash Sha256" Hash="ED6B1C1740D408CBD4107D64A1ABDAD30911B1FE78A4C82818C67CA775C260B6" />
    <Allow ID="ID_ALLOW_A_AF5" FriendlyName="C:\Program Files\git\usr\bin\unexpand.exe Hash Sha1" Hash="75C968A49EA5B1EFDF185752F8A08C90BC663F5A" />
    <Allow ID="ID_ALLOW_A_AF6" FriendlyName="C:\Program Files\git\usr\bin\unexpand.exe Hash Sha256" Hash="6870255826F5A81EBF542497275094C23CF70589634BCFBF187FB667096B614F" />
    <Allow ID="ID_ALLOW_A_AF7" FriendlyName="C:\Program Files\git\usr\bin\unexpand.exe Hash Page Sha1" Hash="7F6ACD451E95073925DFC93DE88A7CD52676264F" />
    <Allow ID="ID_ALLOW_A_AF8" FriendlyName="C:\Program Files\git\usr\bin\unexpand.exe Hash Page Sha256" Hash="2F3D8A0B3D5B29FC16AADC31F8597457A724D6C75F63D642F8C506E174EF41FB" />
    <Allow ID="ID_ALLOW_A_AF9" FriendlyName="C:\Program Files\git\usr\bin\uniq.exe Hash Sha1" Hash="5457D638D704E0AC69CE8BF4D4475CE8473C2A75" />
    <Allow ID="ID_ALLOW_A_AFA" FriendlyName="C:\Program Files\git\usr\bin\uniq.exe Hash Sha256" Hash="FC67A15734C5BFC856A587D23E73D6460734C23978AA8052BBAB6034083E7063" />
    <Allow ID="ID_ALLOW_A_AFB" FriendlyName="C:\Program Files\git\usr\bin\uniq.exe Hash Page Sha1" Hash="7096910917EFEC868A79279F8172F186D2B1A8FF" />
    <Allow ID="ID_ALLOW_A_AFC" FriendlyName="C:\Program Files\git\usr\bin\uniq.exe Hash Page Sha256" Hash="FE45CFBB6745AE197D7FD61D237F3E69B95E4BC330F23E5AA51785DB410CF801" />
    <Allow ID="ID_ALLOW_A_AFD" FriendlyName="C:\Program Files\git\usr\bin\unlink.exe Hash Sha1" Hash="49F4D5B659BB7C4CEE31E3D49DEE08C84F58A88E" />
    <Allow ID="ID_ALLOW_A_AFE" FriendlyName="C:\Program Files\git\usr\bin\unlink.exe Hash Sha256" Hash="5935C1A8937AA421D9D4155D454FC94580F54D2C03F1DE467E49CD2204577151" />
    <Allow ID="ID_ALLOW_A_AFF" FriendlyName="C:\Program Files\git\usr\bin\unlink.exe Hash Page Sha1" Hash="E6EFC2D1E62FBFF3CB33B7E0C2D284BD3B093B92" />
    <Allow ID="ID_ALLOW_A_B00" FriendlyName="C:\Program Files\git\usr\bin\unlink.exe Hash Page Sha256" Hash="BC704B7F455A80A167456FA7C08E46C685E2E39F4420998F6955B1B459F57CA3" />
    <Allow ID="ID_ALLOW_A_B01" FriendlyName="C:\Program Files\git\usr\bin\unzip.exe Hash Sha1" Hash="43143ABEAFD82BF797EC237F59BCED8E2B6D3153" />
    <Allow ID="ID_ALLOW_A_B02" FriendlyName="C:\Program Files\git\usr\bin\unzip.exe Hash Sha256" Hash="3C6CAE112B9F3C0CDFC94BB1B6BC45A7E19C86709A10D8E5A2D6DF1F045D9D9B" />
    <Allow ID="ID_ALLOW_A_B03" FriendlyName="C:\Program Files\git\usr\bin\unzip.exe Hash Page Sha1" Hash="E5E055878A721B05FF849DBE20CF22508DC67CA5" />
    <Allow ID="ID_ALLOW_A_B04" FriendlyName="C:\Program Files\git\usr\bin\unzip.exe Hash Page Sha256" Hash="D6B147BEE5E956C295D6BDDA076201534833176512D0FBB1121261510B854ACE" />
    <Allow ID="ID_ALLOW_A_B05" FriendlyName="C:\Program Files\git\usr\bin\unzipsfx.exe Hash Sha1" Hash="6CE2B9BA46F04A4A1EEB08A327DE346FD7003B26" />
    <Allow ID="ID_ALLOW_A_B06" FriendlyName="C:\Program Files\git\usr\bin\unzipsfx.exe Hash Sha256" Hash="53916AA76BE3D2709CEA3FE7D5FB17BE83F6A4D5CF418E25B44C9AF4CFCCB5D9" />
    <Allow ID="ID_ALLOW_A_B07" FriendlyName="C:\Program Files\git\usr\bin\unzipsfx.exe Hash Page Sha1" Hash="C52E862B8719A69E3CAACC92ED5527F355F16EE1" />
    <Allow ID="ID_ALLOW_A_B08" FriendlyName="C:\Program Files\git\usr\bin\unzipsfx.exe Hash Page Sha256" Hash="2FB0B4F33EED81B67B909CACDFCB86E19A0C6843DF61E52AAE03960896DB5C13" />
    <Allow ID="ID_ALLOW_A_B09" FriendlyName="C:\Program Files\git\usr\bin\users.exe Hash Sha1" Hash="BDB54341CC1BBF5BF638241806143967008A5CE6" />
    <Allow ID="ID_ALLOW_A_B0A" FriendlyName="C:\Program Files\git\usr\bin\users.exe Hash Sha256" Hash="24F365F14ECBDAB57A125ADD19EA1395C02FC1D44777E4FA94F6A9C88A99D65A" />
    <Allow ID="ID_ALLOW_A_B0B" FriendlyName="C:\Program Files\git\usr\bin\users.exe Hash Page Sha1" Hash="5B1EDE15FFB479293CB0144C5151CDEC2BDF9461" />
    <Allow ID="ID_ALLOW_A_B0C" FriendlyName="C:\Program Files\git\usr\bin\users.exe Hash Page Sha256" Hash="CE77439B9447C38BBB7861DEDAF6CC6B0713DCB47B5150424CFB46152DFAB262" />
    <Allow ID="ID_ALLOW_A_B0D" FriendlyName="C:\Program Files\git\usr\bin\vdir.exe Hash Sha1" Hash="D239F9AFA08759E53AADC0B4064EFF556E30EE94" />
    <Allow ID="ID_ALLOW_A_B0E" FriendlyName="C:\Program Files\git\usr\bin\vdir.exe Hash Sha256" Hash="084BEE2CA984B3B250B9B5D724EC2F03D31CFF8A9512E6FC44601D5820C5214F" />
    <Allow ID="ID_ALLOW_A_B0F" FriendlyName="C:\Program Files\git\usr\bin\watchgnupg.exe Hash Sha1" Hash="57A51B5D62BD771BE910D1810CE7BC8FAD33D044" />
    <Allow ID="ID_ALLOW_A_B10" FriendlyName="C:\Program Files\git\usr\bin\watchgnupg.exe Hash Sha256" Hash="167844DE211C8F3D9404CB4DFECC43254031DB1E9924BF76B2E0B37E43D1A329" />
    <Allow ID="ID_ALLOW_A_B11" FriendlyName="C:\Program Files\git\usr\bin\watchgnupg.exe Hash Page Sha1" Hash="8B00449498E0C165C5FECC792E658BCF96C8F80F" />
    <Allow ID="ID_ALLOW_A_B12" FriendlyName="C:\Program Files\git\usr\bin\watchgnupg.exe Hash Page Sha256" Hash="0417B9C36D06E85D458D0A05963B6573492C0B8C97D52DDAB2920CAA54AEE6A7" />
    <Allow ID="ID_ALLOW_A_B13" FriendlyName="C:\Program Files\git\usr\bin\wc.exe Hash Sha1" Hash="C7CA6254E266B648D31251CD4447BF2F536125B3" />
    <Allow ID="ID_ALLOW_A_B14" FriendlyName="C:\Program Files\git\usr\bin\wc.exe Hash Sha256" Hash="D7C7C9301CDC4588396127E259C0123E615F6E36BCFB1BAE73D14673C1DFEBB9" />
    <Allow ID="ID_ALLOW_A_B15" FriendlyName="C:\Program Files\git\usr\bin\wc.exe Hash Page Sha1" Hash="8BD8664FA2E3F98DC4B3AA0D0CDCA7728F3D3DFF" />
    <Allow ID="ID_ALLOW_A_B16" FriendlyName="C:\Program Files\git\usr\bin\wc.exe Hash Page Sha256" Hash="5315482937DD593592E88CBEAC1AA63C8EFB4AF4FD92420F824423BE4505280C" />
    <Allow ID="ID_ALLOW_A_B17" FriendlyName="C:\Program Files\git\usr\bin\which.exe Hash Sha1" Hash="4B6B3745F0B41A867DB2DFBA9C73787DC1D94CC7" />
    <Allow ID="ID_ALLOW_A_B18" FriendlyName="C:\Program Files\git\usr\bin\which.exe Hash Sha256" Hash="BD7C9A4C811E26A2E5329008082145EFCC3E32E89B9A384916D468D59973E1C2" />
    <Allow ID="ID_ALLOW_A_B19" FriendlyName="C:\Program Files\git\usr\bin\which.exe Hash Page Sha1" Hash="2C431D714B868D66FAECE11A286D3D627784D24E" />
    <Allow ID="ID_ALLOW_A_B1A" FriendlyName="C:\Program Files\git\usr\bin\which.exe Hash Page Sha256" Hash="AE3A19B4D72398099500E431A0A3B170A147C7FE608B11759F347ACB91455C82" />
    <Allow ID="ID_ALLOW_A_B1B" FriendlyName="C:\Program Files\git\usr\bin\who.exe Hash Sha1" Hash="69CCE311099103CF0759F515A6EFA1B5A6F69194" />
    <Allow ID="ID_ALLOW_A_B1C" FriendlyName="C:\Program Files\git\usr\bin\who.exe Hash Sha256" Hash="9105CA375380FFEC4F3D9ADF6B61AF776C5ADA3FF497B7B6E244929F31633B41" />
    <Allow ID="ID_ALLOW_A_B1D" FriendlyName="C:\Program Files\git\usr\bin\who.exe Hash Page Sha1" Hash="B47566C60191FA96D19D4BF18B639C48D91EC2B8" />
    <Allow ID="ID_ALLOW_A_B1E" FriendlyName="C:\Program Files\git\usr\bin\who.exe Hash Page Sha256" Hash="80536BBA9F2FAFFDAB92C6510A8652A4C0E0ADE7EA7B88D6694FE21A981F1344" />
    <Allow ID="ID_ALLOW_A_B1F" FriendlyName="C:\Program Files\git\usr\bin\whoami.exe Hash Sha1" Hash="2063993A60B191965F0C3C2BE1194B55A3682A98" />
    <Allow ID="ID_ALLOW_A_B20" FriendlyName="C:\Program Files\git\usr\bin\whoami.exe Hash Sha256" Hash="E7068F5F944981524549E54998473262E1EBEEAFBEC1AD57588C36B97BE511AD" />
    <Allow ID="ID_ALLOW_A_B21" FriendlyName="C:\Program Files\git\usr\bin\whoami.exe Hash Page Sha1" Hash="91DF61B94D5A46252F624D80551649291DDB6C5C" />
    <Allow ID="ID_ALLOW_A_B22" FriendlyName="C:\Program Files\git\usr\bin\whoami.exe Hash Page Sha256" Hash="D2CFE8A15B62B154798930A1F06F95210C8D69FAE4191FC86D68F536BC72F1AE" />
    <Allow ID="ID_ALLOW_A_B23" FriendlyName="C:\Program Files\git\usr\bin\winpty-agent.exe Hash Sha1" Hash="124AB857E81FD227B9B2A63B1890C4F96F107F9D" />
    <Allow ID="ID_ALLOW_A_B24" FriendlyName="C:\Program Files\git\usr\bin\winpty-agent.exe Hash Sha256" Hash="2666589600DFE7165A68C170676204392ED6A8368890AED97FB15AE974D4F4B0" />
    <Allow ID="ID_ALLOW_A_B25" FriendlyName="C:\Program Files\git\usr\bin\winpty-agent.exe Hash Page Sha1" Hash="66384C9EAC22A3B853FCC804CE486B9893285C3F" />
    <Allow ID="ID_ALLOW_A_B26" FriendlyName="C:\Program Files\git\usr\bin\winpty-agent.exe Hash Page Sha256" Hash="7312B5600D285FCA7C164B512BE028B72EE31F2DD2C31DB7262D5F23149F561D" />
    <Allow ID="ID_ALLOW_A_B27" FriendlyName="C:\Program Files\git\usr\bin\winpty-debugserver.exe Hash Sha1" Hash="712BFD20417A5B6C0F396F1CD958A2FADE1508DA" />
    <Allow ID="ID_ALLOW_A_B28" FriendlyName="C:\Program Files\git\usr\bin\winpty-debugserver.exe Hash Sha256" Hash="579E33FA944C515522C024B087621B23D36286BC54C5A190F5A9999203B6D865" />
    <Allow ID="ID_ALLOW_A_B29" FriendlyName="C:\Program Files\git\usr\bin\winpty-debugserver.exe Hash Page Sha1" Hash="45D90E97E9D65CF68F6E60FF7DCC90071598D74E" />
    <Allow ID="ID_ALLOW_A_B2A" FriendlyName="C:\Program Files\git\usr\bin\winpty-debugserver.exe Hash Page Sha256" Hash="A991EF3AFD907E42BF0F911E0D10937EB87B9C5327C8DEC9AC969A0625F35017" />
    <Allow ID="ID_ALLOW_A_B2B" FriendlyName="C:\Program Files\git\usr\bin\winpty.dll Hash Sha1" Hash="1076D1D111BB5029C844D12C2F8352B5F62A5BDB" />
    <Allow ID="ID_ALLOW_A_B2C" FriendlyName="C:\Program Files\git\usr\bin\winpty.dll Hash Sha256" Hash="DD06F432269911E49C71B08C80EC7ADFD96520D10E6CA85078C1CC12D877D903" />
    <Allow ID="ID_ALLOW_A_B2D" FriendlyName="C:\Program Files\git\usr\bin\winpty.dll Hash Page Sha1" Hash="7F29FF0AAC064C3896BB7E7824CF053A10045954" />
    <Allow ID="ID_ALLOW_A_B2E" FriendlyName="C:\Program Files\git\usr\bin\winpty.dll Hash Page Sha256" Hash="54B7A2BDF1B4A296898CB70C62298F4CD069566E1D3F3A9603574CCED5CC1241" />
    <Allow ID="ID_ALLOW_A_B2F" FriendlyName="C:\Program Files\git\usr\bin\winpty.exe Hash Sha1" Hash="27FE31A57C0F3CB7BE2B16976454E373D1877F40" />
    <Allow ID="ID_ALLOW_A_B30" FriendlyName="C:\Program Files\git\usr\bin\winpty.exe Hash Sha256" Hash="A1CEA9F12F2BAA5C888507CFE4687B4A4D05F52955047E9AA496C7FCD73678CC" />
    <Allow ID="ID_ALLOW_A_B31" FriendlyName="C:\Program Files\git\usr\bin\winpty.exe Hash Page Sha1" Hash="C7B488FEA2C9F8CC42AADD37E0AC382558A0FF5F" />
    <Allow ID="ID_ALLOW_A_B32" FriendlyName="C:\Program Files\git\usr\bin\winpty.exe Hash Page Sha256" Hash="85C4E7549EFDD2A1455F204AF19E371AAD2B13BD7C372126A949CFF2B4BB388F" />
    <Allow ID="ID_ALLOW_A_B33" FriendlyName="C:\Program Files\git\usr\bin\xargs.exe Hash Sha1" Hash="8F03D8B8584B4143670F8B5DA112B7680BD46375" />
    <Allow ID="ID_ALLOW_A_B34" FriendlyName="C:\Program Files\git\usr\bin\xargs.exe Hash Sha256" Hash="50FCF87CF599B666CD2168ED678581E8C80E1608C7C489BB58FAB8DE05FE6AAE" />
    <Allow ID="ID_ALLOW_A_B35" FriendlyName="C:\Program Files\git\usr\bin\xargs.exe Hash Page Sha1" Hash="E308DA8DFA66EE88C3C0FC1CE793D3BA874BE553" />
    <Allow ID="ID_ALLOW_A_B36" FriendlyName="C:\Program Files\git\usr\bin\xargs.exe Hash Page Sha256" Hash="3EF82359C34D1243C41D1849F7F90F3C3A496A5D0B7E5405022F10D1983046BC" />
    <Allow ID="ID_ALLOW_A_B37" FriendlyName="C:\Program Files\git\usr\bin\xgettext.exe Hash Sha1" Hash="BBF050D3D4F40DBE975639A8D91E4D650690F400" />
    <Allow ID="ID_ALLOW_A_B38" FriendlyName="C:\Program Files\git\usr\bin\xgettext.exe Hash Sha256" Hash="1E04D0D904D93247D8E028718723A9E099FF759400BC433A2DD893052817B675" />
    <Allow ID="ID_ALLOW_A_B39" FriendlyName="C:\Program Files\git\usr\bin\xgettext.exe Hash Page Sha1" Hash="28E604840964629A4C689689F5A9D9B2364EC11F" />
    <Allow ID="ID_ALLOW_A_B3A" FriendlyName="C:\Program Files\git\usr\bin\xgettext.exe Hash Page Sha256" Hash="5037D016736A1079782ECFB780A4E89DE89E346C55E42CA23052E040347A4DF4" />
    <Allow ID="ID_ALLOW_A_B3B" FriendlyName="C:\Program Files\git\usr\bin\xxd.exe Hash Sha1" Hash="8FFA7C40EAE6C500FEA4F60ABF23AD9D302CA07B" />
    <Allow ID="ID_ALLOW_A_B3C" FriendlyName="C:\Program Files\git\usr\bin\xxd.exe Hash Sha256" Hash="5E38D7B083BCCD941F7A64B281F05C083ECD2A84D333A9892308F6BA72CDEF59" />
    <Allow ID="ID_ALLOW_A_B3D" FriendlyName="C:\Program Files\git\usr\bin\xxd.exe Hash Page Sha1" Hash="60011FBC3500DEC6260F8EB7985E7BE87073BE9C" />
    <Allow ID="ID_ALLOW_A_B3E" FriendlyName="C:\Program Files\git\usr\bin\xxd.exe Hash Page Sha256" Hash="93935F3C86A3F8A5212914565F51F32C3A07D21C2E25CD8ADAA389354C4794F2" />
    <Allow ID="ID_ALLOW_A_B3F" FriendlyName="C:\Program Files\git\usr\bin\yat2m.exe Hash Sha1" Hash="C8AC4340020D75FF4E527F3B79C10062C11FAB58" />
    <Allow ID="ID_ALLOW_A_B40" FriendlyName="C:\Program Files\git\usr\bin\yat2m.exe Hash Sha256" Hash="507F73F60B4DB065EECBC8B7884151440C0686541F9EE46E6183A4CC05E835FA" />
    <Allow ID="ID_ALLOW_A_B41" FriendlyName="C:\Program Files\git\usr\bin\yat2m.exe Hash Page Sha1" Hash="BB65B628B53C678F489AC17098810E962EE5256C" />
    <Allow ID="ID_ALLOW_A_B42" FriendlyName="C:\Program Files\git\usr\bin\yat2m.exe Hash Page Sha256" Hash="6BE59AA9AC125F6875067E8B58FB9B57A22AAEE0545427CD27D94A0F9E0BA084" />
    <Allow ID="ID_ALLOW_A_B43" FriendlyName="C:\Program Files\git\usr\bin\yes.exe Hash Sha1" Hash="8EEFB00EA4096B8DE3C0476CBDD10BADB069D8D7" />
    <Allow ID="ID_ALLOW_A_B44" FriendlyName="C:\Program Files\git\usr\bin\yes.exe Hash Sha256" Hash="8A37C322FCCD3ED64193F183004390D727D0B8B90669FF0AFBED30F7C25D7464" />
    <Allow ID="ID_ALLOW_A_B45" FriendlyName="C:\Program Files\git\usr\bin\yes.exe Hash Page Sha1" Hash="050ED843AA6EE15A2CD1AB21CAA59249B68F9A2D" />
    <Allow ID="ID_ALLOW_A_B46" FriendlyName="C:\Program Files\git\usr\bin\yes.exe Hash Page Sha256" Hash="7F4560ED3030867A47BC397FBA979460CF848AE8A3CB939762E3A676B8759441" />
    <Allow ID="ID_ALLOW_A_B47" FriendlyName="C:\Program Files\git\usr\bin\[.exe Hash Sha1" Hash="BC223117625402A7F9DAF315484179891AB2DA6C" />
    <Allow ID="ID_ALLOW_A_B48" FriendlyName="C:\Program Files\git\usr\bin\[.exe Hash Sha256" Hash="C7AF76DED4E57D24908DB930396C3276A1F79F853012DDC50E692EFBE7B1DCBF" />
    <Allow ID="ID_ALLOW_A_B49" FriendlyName="C:\Program Files\git\usr\bin\[.exe Hash Page Sha1" Hash="F48EF04C3894547CBEAEA03F20A30C2756250F8C" />
    <Allow ID="ID_ALLOW_A_B4A" FriendlyName="C:\Program Files\git\usr\bin\[.exe Hash Page Sha256" Hash="669EFB4F9A042DD819B2EBA52B2B157C3CB4D971ED218EA2CDE68572231AE617" />
    <Allow ID="ID_ALLOW_A_B4B" FriendlyName="C:\Program Files\git\mingw64\share\git-gui\lib\win32_shortcut.js Hash Sha1" Hash="479F6BB0B3176419F84ED3325772FA13817914A5" />
    <Allow ID="ID_ALLOW_A_B4C" FriendlyName="C:\Program Files\git\mingw64\share\git-gui\lib\win32_shortcut.js Hash Sha256" Hash="05D41A2B7CA9D6DB5913CC6BB8DAEDEF3F36831F216C0D5FC36126898439C92C" />
    <Allow ID="ID_ALLOW_A_B4D" FriendlyName="C:\Program Files\git\mingw64\share\git-gui\lib\win32_shortcut.js Hash Authenticode SIP Sha256" Hash="AD2016D9BDACF3683BD2533F042E08C09FDB5E938E24F010782E70D451C9A7B4" />
    <Allow ID="ID_ALLOW_A_B4E" FriendlyName="C:\Program Files\git\mingw64\share\git\edit-git-bash.exe Hash Sha1" Hash="48B209354C22BE9F67873ADD24BBBEE98BF1BD9C" />
    <Allow ID="ID_ALLOW_A_B4F" FriendlyName="C:\Program Files\git\mingw64\share\git\edit-git-bash.exe Hash Sha256" Hash="56676101FDE65EBC00149971423461704EF5E770DCD26ECE60D9E6CE91F0F4FB" />
    <Allow ID="ID_ALLOW_A_B50" FriendlyName="C:\Program Files\git\mingw64\share\git\edit-git-bash.exe Hash Page Sha1" Hash="205C3F401222BFA2065E98DAAA5F653011ED6B95" />
    <Allow ID="ID_ALLOW_A_B51" FriendlyName="C:\Program Files\git\mingw64\share\git\edit-git-bash.exe Hash Page Sha256" Hash="A52235F218ACE4B6B0BA0E5317C217BCC46643B7050B3452495EAF527CD381CB" />
    <Allow ID="ID_ALLOW_A_B56" FriendlyName="C:\Program Files\git\mingw64\libexec\git-core\Bitbucket.Authentication.dll Hash Sha1" Hash="A24F6CEF94A74D07DB6A93359305CD40AF114DE6" />
    <Allow ID="ID_ALLOW_A_B57" FriendlyName="C:\Program Files\git\mingw64\libexec\git-core\Bitbucket.Authentication.dll Hash Sha256" Hash="C992924DBE2909BEEA9D327685A317D0BDDD423D09BC6FC113FB5123384BA717" />
    <Allow ID="ID_ALLOW_A_B58" FriendlyName="C:\Program Files\git\mingw64\libexec\git-core\Bitbucket.Authentication.dll Hash Page Sha1" Hash="A787208D88B2E3BDCFA9D4D6B234D4BDA3F08470" />
    <Allow ID="ID_ALLOW_A_B59" FriendlyName="C:\Program Files\git\mingw64\libexec\git-core\Bitbucket.Authentication.dll Hash Page Sha256" Hash="5AD460711379DB0891F84BD49AC9995429D5E9185074B6CB01DC7B2560760349" />
    <Allow ID="ID_ALLOW_A_B5A" FriendlyName="C:\Program Files\git\mingw64\libexec\git-core\edit.dll Hash Sha1" Hash="BF73A4149FD59F1CFBB1083365AB69551E82F431" />
    <Allow ID="ID_ALLOW_A_B5B" FriendlyName="C:\Program Files\git\mingw64\libexec\git-core\edit.dll Hash Sha256" Hash="6A966CF53C9D4A599C7F27939109820BCEB87FD8466CBB5CDE19361D98A31896" />
    <Allow ID="ID_ALLOW_A_B5C" FriendlyName="C:\Program Files\git\mingw64\libexec\git-core\edit.dll Hash Page Sha1" Hash="610F0C233BF01DEA054B6A07035099CA7F1B1EB5" />
    <Allow ID="ID_ALLOW_A_B5D" FriendlyName="C:\Program Files\git\mingw64\libexec\git-core\edit.dll Hash Page Sha256" Hash="FACB0F182458CC166850DB5DB560A9492BD315603948B3B0E7DF62DE7EA8E249" />
    <Allow ID="ID_ALLOW_A_B62" FriendlyName="C:\Program Files\git\mingw64\libexec\git-core\GitHub.Authentication.exe Hash Sha1" Hash="6581D1AACD55D02FEFCF97A0E0F831CE02F65516" />
    <Allow ID="ID_ALLOW_A_B63" FriendlyName="C:\Program Files\git\mingw64\libexec\git-core\GitHub.Authentication.exe Hash Sha256" Hash="86F083134A42C98776EA249D615EC8F576C3B1BB8714296CD233BD4B1D41313D" />
    <Allow ID="ID_ALLOW_A_B64" FriendlyName="C:\Program Files\git\mingw64\libexec\git-core\GitHub.Authentication.exe Hash Page Sha1" Hash="C85EE6DEFA296437B3C07107CC047E9AFF15577F" />
    <Allow ID="ID_ALLOW_A_B65" FriendlyName="C:\Program Files\git\mingw64\libexec\git-core\GitHub.Authentication.exe Hash Page Sha256" Hash="36BE51CCD8DB137322F1932ADAF439CE55538D767EACCD0202BBC22025921FF5" />
    <Allow ID="ID_ALLOW_A_B66" FriendlyName="C:\Program Files\git\mingw64\libexec\git-core\libbrotlicommon.dll Hash Sha1" Hash="D07839BD1B41E99993715CE528541BCF047A9029" />
    <Allow ID="ID_ALLOW_A_B67" FriendlyName="C:\Program Files\git\mingw64\libexec\git-core\libbrotlicommon.dll Hash Sha256" Hash="DF0398ADB02C5717CD75ADE1CEBD839B3EFBBDACF8257B1F8A08A1F5BBA55E27" />
    <Allow ID="ID_ALLOW_A_B68" FriendlyName="C:\Program Files\git\mingw64\libexec\git-core\libbrotlicommon.dll Hash Page Sha1" Hash="70BEA4FCA4E63EF3F69D4E2A9820E41EFAC5D3EB" />
    <Allow ID="ID_ALLOW_A_B69" FriendlyName="C:\Program Files\git\mingw64\libexec\git-core\libbrotlicommon.dll Hash Page Sha256" Hash="38CDD24AD083EDD1BB7C4E76AC4846694FAC82FE4322E4BEE908C4D18FDC825C" />
    <Allow ID="ID_ALLOW_A_B6A" FriendlyName="C:\Program Files\git\mingw64\libexec\git-core\libbrotlidec.dll Hash Sha1" Hash="4D34500CD44B99552513F8C7A725B4EC4BFA9CF1" />
    <Allow ID="ID_ALLOW_A_B6B" FriendlyName="C:\Program Files\git\mingw64\libexec\git-core\libbrotlidec.dll Hash Sha256" Hash="E108C5E0ADB2FC250A894A82A5BA62886EB70BEB680DC4C80188A94A0EE5A856" />
    <Allow ID="ID_ALLOW_A_B6C" FriendlyName="C:\Program Files\git\mingw64\libexec\git-core\libbrotlidec.dll Hash Page Sha1" Hash="74D2916B1EFFF1819C6B087DB503F823D2CD6C9E" />
    <Allow ID="ID_ALLOW_A_B6D" FriendlyName="C:\Program Files\git\mingw64\libexec\git-core\libbrotlidec.dll Hash Page Sha256" Hash="7D5A0642725EF9BCCA8D2B5DF5C5DEB8741E710E8620AC19274484F4F48544CD" />
    <Allow ID="ID_ALLOW_A_B6E" FriendlyName="C:\Program Files\git\mingw64\libexec\git-core\libbz2-1.dll Hash Sha1" Hash="35DD1770659D51F23C0358D61639D2D1D18E1236" />
    <Allow ID="ID_ALLOW_A_B6F" FriendlyName="C:\Program Files\git\mingw64\libexec\git-core\libbz2-1.dll Hash Sha256" Hash="49328177F4B0BD990C46086B02CE45259BC7A0B1E2805CA7755DC72B04592818" />
    <Allow ID="ID_ALLOW_A_B70" FriendlyName="C:\Program Files\git\mingw64\libexec\git-core\libbz2-1.dll Hash Page Sha1" Hash="BAB54FAEAAD458BCB15A6F5F3EF6714DE0362B2F" />
    <Allow ID="ID_ALLOW_A_B71" FriendlyName="C:\Program Files\git\mingw64\libexec\git-core\libbz2-1.dll Hash Page Sha256" Hash="5B829B91160546A5317B8BD697BBD0BD32F53EDADC008FFAD0A726345C4F9766" />
    <Allow ID="ID_ALLOW_A_B72" FriendlyName="C:\Program Files\git\mingw64\libexec\git-core\libcares-4.dll Hash Sha1" Hash="93CB70308A04C4F845554B8CF24AA87F0C2B7E4A" />
    <Allow ID="ID_ALLOW_A_B73" FriendlyName="C:\Program Files\git\mingw64\libexec\git-core\libcares-4.dll Hash Sha256" Hash="D39A66CABBB8E2407495178AD3805919ECFD659EA805E3513A473D1751203355" />
    <Allow ID="ID_ALLOW_A_B74" FriendlyName="C:\Program Files\git\mingw64\libexec\git-core\libcares-4.dll Hash Page Sha1" Hash="EC71FC80628112CEF63124100F9D3ECDBEB75173" />
    <Allow ID="ID_ALLOW_A_B75" FriendlyName="C:\Program Files\git\mingw64\libexec\git-core\libcares-4.dll Hash Page Sha256" Hash="30FE39FD094EBD84E32CC2849213AC3D20903BC02299634C7E859AF6BBC87621" />
    <Allow ID="ID_ALLOW_A_B76" FriendlyName="C:\Program Files\git\mingw64\libexec\git-core\libcrypto-1_1-x64.dll Hash Sha1" Hash="A41CF7C0749E8268F166AF626801780234503A8F" />
    <Allow ID="ID_ALLOW_A_B77" FriendlyName="C:\Program Files\git\mingw64\libexec\git-core\libcrypto-1_1-x64.dll Hash Sha256" Hash="E87D90E41E7CCCA299DDF7D037975CCE09C824DE9622E875C2889A5EA880C4F3" />
    <Allow ID="ID_ALLOW_A_B78" FriendlyName="C:\Program Files\git\mingw64\libexec\git-core\libcrypto-1_1-x64.dll Hash Page Sha1" Hash="72DA8573CD6836B8BB5F20816B2EE79C2E00DFE2" />
    <Allow ID="ID_ALLOW_A_B79" FriendlyName="C:\Program Files\git\mingw64\libexec\git-core\libcrypto-1_1-x64.dll Hash Page Sha256" Hash="E86DBDA0A14B1B2D4FCC21253C499BEF5F0E28925A28854C2FCC417106067199" />
    <Allow ID="ID_ALLOW_A_B7A" FriendlyName="C:\Program Files\git\mingw64\libexec\git-core\libcurl-4.dll Hash Sha1" Hash="AFC29328EA996D40DAC9E24E6DB10D248C9189DE" />
    <Allow ID="ID_ALLOW_A_B7B" FriendlyName="C:\Program Files\git\mingw64\libexec\git-core\libcurl-4.dll Hash Sha256" Hash="CCE34AF80C54CA40174296FDD9E96BE8A09818369241BBB4571AA188EEBB7F03" />
    <Allow ID="ID_ALLOW_A_B7C" FriendlyName="C:\Program Files\git\mingw64\libexec\git-core\libcurl-4.dll Hash Page Sha1" Hash="D5FA5392846CC938CA86665D4D9F70E318235A45" />
    <Allow ID="ID_ALLOW_A_B7D" FriendlyName="C:\Program Files\git\mingw64\libexec\git-core\libcurl-4.dll Hash Page Sha256" Hash="E443BCF1557E8DD574E87ECE1F65FA8C049868B03EF4191A44E4135FAC2410C2" />
    <Allow ID="ID_ALLOW_A_B7E" FriendlyName="C:\Program Files\git\mingw64\libexec\git-core\libexpat-1.dll Hash Sha1" Hash="C5FF61EFD880AF4C601C7B8876C41A21AC7C69E4" />
    <Allow ID="ID_ALLOW_A_B7F" FriendlyName="C:\Program Files\git\mingw64\libexec\git-core\libexpat-1.dll Hash Sha256" Hash="95F6F3B9702838C4425784C408C909C9369879C19807E1A8DE3465E4BC651B54" />
    <Allow ID="ID_ALLOW_A_B80" FriendlyName="C:\Program Files\git\mingw64\libexec\git-core\libexpat-1.dll Hash Page Sha1" Hash="5DC7A0194ACD9706325DB900FFFC18BF297A978A" />
    <Allow ID="ID_ALLOW_A_B81" FriendlyName="C:\Program Files\git\mingw64\libexec\git-core\libexpat-1.dll Hash Page Sha256" Hash="CEA3F27CD06571407440ED4807D44B3A00EF3A628BF706E72F26EF2F9791DF75" />
    <Allow ID="ID_ALLOW_A_B82" FriendlyName="C:\Program Files\git\mingw64\libexec\git-core\libgcc_s_seh-1.dll Hash Sha1" Hash="DF6DF73627365F749D3E7B090045266F1FAB2A9A" />
    <Allow ID="ID_ALLOW_A_B83" FriendlyName="C:\Program Files\git\mingw64\libexec\git-core\libgcc_s_seh-1.dll Hash Sha256" Hash="C781A8CA70AF4FFCA186018B143D788DAFCC955E8370DF7AAEC1FF32E3F71881" />
    <Allow ID="ID_ALLOW_A_B84" FriendlyName="C:\Program Files\git\mingw64\libexec\git-core\libgcc_s_seh-1.dll Hash Page Sha1" Hash="92C1138C582E36DE8F5AF35C73BA6E88B6C23EB7" />
    <Allow ID="ID_ALLOW_A_B85" FriendlyName="C:\Program Files\git\mingw64\libexec\git-core\libgcc_s_seh-1.dll Hash Page Sha256" Hash="04736D2FEA7C309C4CB821D32F437FAF39B604CF5EE95D568912D82FD366EB80" />
    <Allow ID="ID_ALLOW_A_B86" FriendlyName="C:\Program Files\git\mingw64\libexec\git-core\libgmp-10.dll Hash Sha1" Hash="8861BC13757060976D2617615E1920E07055D8DB" />
    <Allow ID="ID_ALLOW_A_B87" FriendlyName="C:\Program Files\git\mingw64\libexec\git-core\libgmp-10.dll Hash Sha256" Hash="36F16817A5562C9029BDFD3C1D2ED839FD921F3A9E5AA473B52EA46E402B5340" />
    <Allow ID="ID_ALLOW_A_B88" FriendlyName="C:\Program Files\git\mingw64\libexec\git-core\libgmp-10.dll Hash Page Sha1" Hash="D6F618130307C3F5F3147EC07AF558272D949D60" />
    <Allow ID="ID_ALLOW_A_B89" FriendlyName="C:\Program Files\git\mingw64\libexec\git-core\libgmp-10.dll Hash Page Sha256" Hash="1C7B16DE4AB05F2FB374DA7B0D264A5E26355947A8D629D8449721235CA97926" />
    <Allow ID="ID_ALLOW_A_B8A" FriendlyName="C:\Program Files\git\mingw64\libexec\git-core\libhogweed-6.dll Hash Sha1" Hash="616140DBEA01BC432F4F7A69920BDF69FAC08005" />
    <Allow ID="ID_ALLOW_A_B8B" FriendlyName="C:\Program Files\git\mingw64\libexec\git-core\libhogweed-6.dll Hash Sha256" Hash="B902055907AE036F094EBB758183E3790CB136EB3DBBD51372681AD651F00F33" />
    <Allow ID="ID_ALLOW_A_B8C" FriendlyName="C:\Program Files\git\mingw64\libexec\git-core\libhogweed-6.dll Hash Page Sha1" Hash="C35CB7531ECEA1670B2C91D5F4C7124084F4E93B" />
    <Allow ID="ID_ALLOW_A_B8D" FriendlyName="C:\Program Files\git\mingw64\libexec\git-core\libhogweed-6.dll Hash Page Sha256" Hash="AED1127445876DBCEFF6D0CEB63BD89E4F4CE6F9088FA82878C4D5AB558CDABF" />
    <Allow ID="ID_ALLOW_A_B8E" FriendlyName="C:\Program Files\git\mingw64\libexec\git-core\libiconv-2.dll Hash Sha1" Hash="80F1B6462779DC87353DF39F3DBBACC773CB260C" />
    <Allow ID="ID_ALLOW_A_B8F" FriendlyName="C:\Program Files\git\mingw64\libexec\git-core\libiconv-2.dll Hash Sha256" Hash="564EBFBBB4CD74D0F2F61381B4B83EDED17E474CDD62ECC1FF6309C56115519B" />
    <Allow ID="ID_ALLOW_A_B90" FriendlyName="C:\Program Files\git\mingw64\libexec\git-core\libiconv-2.dll Hash Page Sha1" Hash="D4D7C69585B3CCEE414892B02E9E817903341E09" />
    <Allow ID="ID_ALLOW_A_B91" FriendlyName="C:\Program Files\git\mingw64\libexec\git-core\libiconv-2.dll Hash Page Sha256" Hash="F5F464BC9EFDF2C745538C5E2412731BA9AC3B1BD062E302CF77480A5BA26BB0" />
    <Allow ID="ID_ALLOW_A_B92" FriendlyName="C:\Program Files\git\mingw64\libexec\git-core\libidn2-0.dll Hash Sha1" Hash="EDF7EBD65E6560F32505D43280311EBE60994079" />
    <Allow ID="ID_ALLOW_A_B93" FriendlyName="C:\Program Files\git\mingw64\libexec\git-core\libidn2-0.dll Hash Sha256" Hash="D97739AB7E333F8D771D6137682E344E0C729CF9BAC4B1CB511D60134E8D83AB" />
    <Allow ID="ID_ALLOW_A_B94" FriendlyName="C:\Program Files\git\mingw64\libexec\git-core\libidn2-0.dll Hash Page Sha1" Hash="16D7FCA7DBA7C369448C892FB6BCB68F72890B4D" />
    <Allow ID="ID_ALLOW_A_B95" FriendlyName="C:\Program Files\git\mingw64\libexec\git-core\libidn2-0.dll Hash Page Sha256" Hash="583887762FA4FEEE8F7B970D477D239E302AD67E1D9ED19A2E9680D4458937AF" />
    <Allow ID="ID_ALLOW_A_B96" FriendlyName="C:\Program Files\git\mingw64\libexec\git-core\libintl-8.dll Hash Sha1" Hash="64C4082CA9CF913B29AB8F2C3B6158E011965A16" />
    <Allow ID="ID_ALLOW_A_B97" FriendlyName="C:\Program Files\git\mingw64\libexec\git-core\libintl-8.dll Hash Sha256" Hash="5374E73012D579B93ECF38E544616B7C4E9447F1FB13DDC5F8FAEEBB53066B6C" />
    <Allow ID="ID_ALLOW_A_B98" FriendlyName="C:\Program Files\git\mingw64\libexec\git-core\libintl-8.dll Hash Page Sha1" Hash="4C9BB63FA8FBCBDF2FA80B6EDE782AB9BAD8C9A3" />
    <Allow ID="ID_ALLOW_A_B99" FriendlyName="C:\Program Files\git\mingw64\libexec\git-core\libintl-8.dll Hash Page Sha256" Hash="EF790B7B885798CABEC53B674F53BF626F34706109C355B2D6F7E223580B38BD" />
    <Allow ID="ID_ALLOW_A_B9A" FriendlyName="C:\Program Files\git\mingw64\libexec\git-core\libjansson-4.dll Hash Sha1" Hash="62AE85DB3AB5A373E7BA635146B9461F2F6B90CC" />
    <Allow ID="ID_ALLOW_A_B9B" FriendlyName="C:\Program Files\git\mingw64\libexec\git-core\libjansson-4.dll Hash Sha256" Hash="84B1AA8C00B6E1E10AB8167D1542D461A5448ADC703BE359F26C6010E811ADD3" />
    <Allow ID="ID_ALLOW_A_B9C" FriendlyName="C:\Program Files\git\mingw64\libexec\git-core\libjansson-4.dll Hash Page Sha1" Hash="F6431F6B3C2C45038DA851346F639D561C52960C" />
    <Allow ID="ID_ALLOW_A_B9D" FriendlyName="C:\Program Files\git\mingw64\libexec\git-core\libjansson-4.dll Hash Page Sha256" Hash="28B1F73765A761840C9E5F3EC974EED260AF6708E8B44BF9AEC5E71B22A7ACF7" />
    <Allow ID="ID_ALLOW_A_B9E" FriendlyName="C:\Program Files\git\mingw64\libexec\git-core\libjemalloc.dll Hash Sha1" Hash="C9BBA702C1DEA3B4C37FD2F85050A4922E442C01" />
    <Allow ID="ID_ALLOW_A_B9F" FriendlyName="C:\Program Files\git\mingw64\libexec\git-core\libjemalloc.dll Hash Sha256" Hash="A5CDB4A555CF93A5FEF840054A3275987F06857593381EA24A7B9322A2BC73EB" />
    <Allow ID="ID_ALLOW_A_BA0" FriendlyName="C:\Program Files\git\mingw64\libexec\git-core\libjemalloc.dll Hash Page Sha1" Hash="83F7FB4275B2B6F1999EAF1D731BCC5FAF84A039" />
    <Allow ID="ID_ALLOW_A_BA1" FriendlyName="C:\Program Files\git\mingw64\libexec\git-core\libjemalloc.dll Hash Page Sha256" Hash="AD7FB74ECCCD7D6EAFCBA0F4648D7CF78E54D0C3CB345DC3B10A14B56AAA05E5" />
    <Allow ID="ID_ALLOW_A_BA2" FriendlyName="C:\Program Files\git\mingw64\libexec\git-core\liblzma-5.dll Hash Sha1" Hash="660113118FED68C497F721E1E1E3FEAFB6758FDC" />
    <Allow ID="ID_ALLOW_A_BA3" FriendlyName="C:\Program Files\git\mingw64\libexec\git-core\liblzma-5.dll Hash Sha256" Hash="60EDBCA63E02FCBC28F675CEFB57D807611AAE2092E86B6456A5127010127BAF" />
    <Allow ID="ID_ALLOW_A_BA4" FriendlyName="C:\Program Files\git\mingw64\libexec\git-core\liblzma-5.dll Hash Page Sha1" Hash="561A0A0F5B24C8ED254683FD778BC82FAAEFB6D2" />
    <Allow ID="ID_ALLOW_A_BA5" FriendlyName="C:\Program Files\git\mingw64\libexec\git-core\liblzma-5.dll Hash Page Sha256" Hash="8AF29C7CCAC879DCAB946A0BA60B8EB64E4179540CDC3AB4EDB4DC5C1F6DF15E" />
    <Allow ID="ID_ALLOW_A_BA6" FriendlyName="C:\Program Files\git\mingw64\libexec\git-core\libmetalink-3.dll Hash Sha1" Hash="40F396B2C352FE0DB61474379355DF380BEB21A4" />
    <Allow ID="ID_ALLOW_A_BA7" FriendlyName="C:\Program Files\git\mingw64\libexec\git-core\libmetalink-3.dll Hash Sha256" Hash="C57A2E91DB7ADE5D4B0EC27869A6CDE3A75BD12CF3AA8D8C0CC381B66F78F147" />
    <Allow ID="ID_ALLOW_A_BA8" FriendlyName="C:\Program Files\git\mingw64\libexec\git-core\libmetalink-3.dll Hash Page Sha1" Hash="B2389339638C08AC938B3453B327F4A19EC3BCB9" />
    <Allow ID="ID_ALLOW_A_BA9" FriendlyName="C:\Program Files\git\mingw64\libexec\git-core\libmetalink-3.dll Hash Page Sha256" Hash="C4D18248165F91E3CD71F980DF15BF608FE68CC7EFEDCA56F3F9406948494C62" />
    <Allow ID="ID_ALLOW_A_BAA" FriendlyName="C:\Program Files\git\mingw64\libexec\git-core\libnettle-8.dll Hash Sha1" Hash="0224289D7EB4AA8A760C8B2E3CC5CA9132AEB108" />
    <Allow ID="ID_ALLOW_A_BAB" FriendlyName="C:\Program Files\git\mingw64\libexec\git-core\libnettle-8.dll Hash Sha256" Hash="40B34E4297B1B6B5BE36DF7424BD455FFE653BB1618645FFBDC4EB13DCF9007A" />
    <Allow ID="ID_ALLOW_A_BAC" FriendlyName="C:\Program Files\git\mingw64\libexec\git-core\libnettle-8.dll Hash Page Sha1" Hash="6ABB16519DFC60012675BC9B4D3667F4B50E7255" />
    <Allow ID="ID_ALLOW_A_BAD" FriendlyName="C:\Program Files\git\mingw64\libexec\git-core\libnettle-8.dll Hash Page Sha256" Hash="69E12194B3FC0EF1A92972D183A1BC703A3A602539FA4C3D61222863B333A4D9" />
    <Allow ID="ID_ALLOW_A_BAE" FriendlyName="C:\Program Files\git\mingw64\libexec\git-core\libnghttp2-14.dll Hash Sha1" Hash="13D2E2441963792F6326E73C486ABA9236504FED" />
    <Allow ID="ID_ALLOW_A_BAF" FriendlyName="C:\Program Files\git\mingw64\libexec\git-core\libnghttp2-14.dll Hash Sha256" Hash="E4B8A62EF44B7B00C2D4D9A5EDCC1F17B6E9CBA927577A6F9D5ADBDFD2C7DF39" />
    <Allow ID="ID_ALLOW_A_BB0" FriendlyName="C:\Program Files\git\mingw64\libexec\git-core\libnghttp2-14.dll Hash Page Sha1" Hash="FE5A8B42001022CDFD22291F3F06DAC7C1E2E693" />
    <Allow ID="ID_ALLOW_A_BB1" FriendlyName="C:\Program Files\git\mingw64\libexec\git-core\libnghttp2-14.dll Hash Page Sha256" Hash="7115D3B07510C2E7B31AC687F6CAA667C8390FCFC68A6B9EF6CEC58E34F1D71C" />
    <Allow ID="ID_ALLOW_A_BB2" FriendlyName="C:\Program Files\git\mingw64\libexec\git-core\libpcre-1.dll Hash Sha1" Hash="3829BB160171B0CE86B3C4D5105315DBA35F6E86" />
    <Allow ID="ID_ALLOW_A_BB3" FriendlyName="C:\Program Files\git\mingw64\libexec\git-core\libpcre-1.dll Hash Sha256" Hash="D293A8A165037EB4A6D4C59698A2494E53190077CE1933F659D19067B0C9DA9E" />
    <Allow ID="ID_ALLOW_A_BB4" FriendlyName="C:\Program Files\git\mingw64\libexec\git-core\libpcre-1.dll Hash Page Sha1" Hash="92217EE7864923855A7248A17FC5CFD1BA0DA416" />
    <Allow ID="ID_ALLOW_A_BB5" FriendlyName="C:\Program Files\git\mingw64\libexec\git-core\libpcre-1.dll Hash Page Sha256" Hash="C78B96139DFA871DC2A004692874386D040F5FAEE16F37169893A307BD7E4A87" />
    <Allow ID="ID_ALLOW_A_BB6" FriendlyName="C:\Program Files\git\mingw64\libexec\git-core\libpcre2-8-0.dll Hash Sha1" Hash="C31B1C8A6DE948F880A7BF5DD8EF170203B989D4" />
    <Allow ID="ID_ALLOW_A_BB7" FriendlyName="C:\Program Files\git\mingw64\libexec\git-core\libpcre2-8-0.dll Hash Sha256" Hash="7B6E4F7D4D465863D4659961F439DD6880778EEDD93A1CF808D81C58FBACB1EE" />
    <Allow ID="ID_ALLOW_A_BB8" FriendlyName="C:\Program Files\git\mingw64\libexec\git-core\libpcre2-8-0.dll Hash Page Sha1" Hash="73BA1483D52D9E05C7CCB4DF37E169DEF2385B1D" />
    <Allow ID="ID_ALLOW_A_BB9" FriendlyName="C:\Program Files\git\mingw64\libexec\git-core\libpcre2-8-0.dll Hash Page Sha256" Hash="1FFC4D86B276E51678D2D8504900463B6140735FE8727EEA80C9FB3BA6D2FEF3" />
    <Allow ID="ID_ALLOW_A_BBA" FriendlyName="C:\Program Files\git\mingw64\libexec\git-core\libpcreposix-0.dll Hash Sha1" Hash="D4DADD84916A0E1193F2E58D04B1BAA8F02B9413" />
    <Allow ID="ID_ALLOW_A_BBB" FriendlyName="C:\Program Files\git\mingw64\libexec\git-core\libpcreposix-0.dll Hash Sha256" Hash="765E789D4BADB7AB17ADD84F7281233BD82AB392FEFC9011DF40793E2D30D028" />
    <Allow ID="ID_ALLOW_A_BBC" FriendlyName="C:\Program Files\git\mingw64\libexec\git-core\libpcreposix-0.dll Hash Page Sha1" Hash="E76C56ACCF97A07343FBE266F9BD718D4F715612" />
    <Allow ID="ID_ALLOW_A_BBD" FriendlyName="C:\Program Files\git\mingw64\libexec\git-core\libpcreposix-0.dll Hash Page Sha256" Hash="5C17CF9233352942F7343A5ECB12B137D704EF63FEA94C23D41176C14FBC3946" />
    <Allow ID="ID_ALLOW_A_BBE" FriendlyName="C:\Program Files\git\mingw64\libexec\git-core\libssh2-1.dll Hash Sha1" Hash="1E8BC606F9A889CF91FAEBF10362635263A8389D" />
    <Allow ID="ID_ALLOW_A_BBF" FriendlyName="C:\Program Files\git\mingw64\libexec\git-core\libssh2-1.dll Hash Sha256" Hash="08816BEBD01BFF57BFBCCB8CE6D9CB4A2D0D4F3A4285CC3B62EC7F800117ECCA" />
    <Allow ID="ID_ALLOW_A_BC0" FriendlyName="C:\Program Files\git\mingw64\libexec\git-core\libssh2-1.dll Hash Page Sha1" Hash="791810AE1271030A5F0F36B2452DAA453A5A3172" />
    <Allow ID="ID_ALLOW_A_BC1" FriendlyName="C:\Program Files\git\mingw64\libexec\git-core\libssh2-1.dll Hash Page Sha256" Hash="D4F4D38D471803F235DAA632E59F825089FA9C7C450E56B62D08FF08BCBB1C60" />
    <Allow ID="ID_ALLOW_A_BC2" FriendlyName="C:\Program Files\git\mingw64\libexec\git-core\libssl-1_1-x64.dll Hash Sha1" Hash="1908987E0E42982D655D241A5133FECBC112A9A4" />
    <Allow ID="ID_ALLOW_A_BC3" FriendlyName="C:\Program Files\git\mingw64\libexec\git-core\libssl-1_1-x64.dll Hash Sha256" Hash="4FAB4395B77A086DAE0B46B483E2E21BD5AAB296BD073EFFCB0A2845D54CA868" />
    <Allow ID="ID_ALLOW_A_BC4" FriendlyName="C:\Program Files\git\mingw64\libexec\git-core\libssl-1_1-x64.dll Hash Page Sha1" Hash="20B49B718B100F90FF9A295624775D41FD67B960" />
    <Allow ID="ID_ALLOW_A_BC5" FriendlyName="C:\Program Files\git\mingw64\libexec\git-core\libssl-1_1-x64.dll Hash Page Sha256" Hash="37244391A598C49B8D4D1F700CC902C1129CC8D3FA83A3BEF403B5325F568C6A" />
    <Allow ID="ID_ALLOW_A_BC6" FriendlyName="C:\Program Files\git\mingw64\libexec\git-core\libssp-0.dll Hash Sha1" Hash="F82320C77BFD10F933625475D4B571C4622F1D93" />
    <Allow ID="ID_ALLOW_A_BC7" FriendlyName="C:\Program Files\git\mingw64\libexec\git-core\libssp-0.dll Hash Sha256" Hash="5242481912B0F8A47720315D4AE089736B4F02A24AA3501143293F919AC2DD29" />
    <Allow ID="ID_ALLOW_A_BC8" FriendlyName="C:\Program Files\git\mingw64\libexec\git-core\libssp-0.dll Hash Page Sha1" Hash="02CBA37A3879DB0927D67B4684EB82FE6217D703" />
    <Allow ID="ID_ALLOW_A_BC9" FriendlyName="C:\Program Files\git\mingw64\libexec\git-core\libssp-0.dll Hash Page Sha256" Hash="031CC869D2F183E7D15CBC45FE9650A2B4FBEF12C8229B639FA746FFF07539F5" />
    <Allow ID="ID_ALLOW_A_BCA" FriendlyName="C:\Program Files\git\mingw64\libexec\git-core\libstdc++-6.dll Hash Sha1" Hash="64D6D71B2E69023ACFC1BB586C58A57CB012586F" />
    <Allow ID="ID_ALLOW_A_BCB" FriendlyName="C:\Program Files\git\mingw64\libexec\git-core\libstdc++-6.dll Hash Sha256" Hash="77CE920BE86BF8290FA34A6613DFAE0CAE52E0E882405C8F63CAF2DD0D4F7062" />
    <Allow ID="ID_ALLOW_A_BCC" FriendlyName="C:\Program Files\git\mingw64\libexec\git-core\libstdc++-6.dll Hash Page Sha1" Hash="A78FA9EAD430E0E3DF2116FC2E56AAD2024A57BD" />
    <Allow ID="ID_ALLOW_A_BCD" FriendlyName="C:\Program Files\git\mingw64\libexec\git-core\libstdc++-6.dll Hash Page Sha256" Hash="3B3AA52F2479A17E6D599A371AA94EE9AE100EADFD5C3A106C1A4BE53241FBC6" />
    <Allow ID="ID_ALLOW_A_BCE" FriendlyName="C:\Program Files\git\mingw64\libexec\git-core\libtre-5.dll Hash Sha1" Hash="27424D42849EC151E3C73DABFA9D210830C94A4F" />
    <Allow ID="ID_ALLOW_A_BCF" FriendlyName="C:\Program Files\git\mingw64\libexec\git-core\libtre-5.dll Hash Sha256" Hash="5077A8AB892186963BEA4DB07D8CFC3176A6DA6067C2619FFDAF05FD9CF23E6C" />
    <Allow ID="ID_ALLOW_A_BD0" FriendlyName="C:\Program Files\git\mingw64\libexec\git-core\libtre-5.dll Hash Page Sha1" Hash="693E6A2EDA93CE7558B8244B7CE0A1AFC2CEE55E" />
    <Allow ID="ID_ALLOW_A_BD1" FriendlyName="C:\Program Files\git\mingw64\libexec\git-core\libtre-5.dll Hash Page Sha256" Hash="9A743E86A94B47F58328E2E5CFA4C0AB87A489250F68F15AC13DE713D4497FCD" />
    <Allow ID="ID_ALLOW_A_BD2" FriendlyName="C:\Program Files\git\mingw64\libexec\git-core\libunistring-2.dll Hash Sha1" Hash="41EF847E20AC87A4A6C6E485F751044F46E60408" />
    <Allow ID="ID_ALLOW_A_BD3" FriendlyName="C:\Program Files\git\mingw64\libexec\git-core\libunistring-2.dll Hash Sha256" Hash="4814A014A5A78AABE2DE8E71FE5E10E5A27391C00466F469475C3C166730B522" />
    <Allow ID="ID_ALLOW_A_BD4" FriendlyName="C:\Program Files\git\mingw64\libexec\git-core\libunistring-2.dll Hash Page Sha1" Hash="053F2EAF3C7D1EBCF24DAFF93B6237FAAE515DD3" />
    <Allow ID="ID_ALLOW_A_BD5" FriendlyName="C:\Program Files\git\mingw64\libexec\git-core\libunistring-2.dll Hash Page Sha256" Hash="552089FDE8675624F49C9C210B0636A2E2A490FE91571BF79A561F23259759DC" />
    <Allow ID="ID_ALLOW_A_BD6" FriendlyName="C:\Program Files\git\mingw64\libexec\git-core\libwinpthread-1.dll Hash Sha1" Hash="3539FA1396F2EE979735D6DBD3FEB3011CBDCD55" />
    <Allow ID="ID_ALLOW_A_BD7" FriendlyName="C:\Program Files\git\mingw64\libexec\git-core\libwinpthread-1.dll Hash Sha256" Hash="76670A8F40F0CAD9142D6F2368E39AAF14A9D5371D7F3CE83CC10D41FEC7CEC8" />
    <Allow ID="ID_ALLOW_A_BD8" FriendlyName="C:\Program Files\git\mingw64\libexec\git-core\libwinpthread-1.dll Hash Page Sha1" Hash="A2E5FB60D45A394AA7597A8E54EDB529C00B607C" />
    <Allow ID="ID_ALLOW_A_BD9" FriendlyName="C:\Program Files\git\mingw64\libexec\git-core\libwinpthread-1.dll Hash Page Sha256" Hash="E44F08BFA2B1E04C35F8FA086319723CFA7FED1DD15F575023BC28FBD2461769" />
    <Allow ID="ID_ALLOW_A_BDA" FriendlyName="C:\Program Files\git\mingw64\libexec\git-core\libzstd.dll Hash Sha1" Hash="EA273DC4EF4AED9E73A9BF711328666609D61E93" />
    <Allow ID="ID_ALLOW_A_BDB" FriendlyName="C:\Program Files\git\mingw64\libexec\git-core\libzstd.dll Hash Sha256" Hash="3E518A1D0C7ED26DF61CD95F052F609B19CF46BE2001363ADE5A16D611727A3F" />
    <Allow ID="ID_ALLOW_A_BDC" FriendlyName="C:\Program Files\git\mingw64\libexec\git-core\libzstd.dll Hash Page Sha1" Hash="06942976F90C108A2140ADE5595CEAD82B932894" />
    <Allow ID="ID_ALLOW_A_BDD" FriendlyName="C:\Program Files\git\mingw64\libexec\git-core\libzstd.dll Hash Page Sha256" Hash="0AA1F6F3E32EEFEEA8C91949EDA6074CD00C7010C49E4FB5731A0A0364BA1A55" />
    <Allow ID="ID_ALLOW_A_BDE" FriendlyName="C:\Program Files\git\mingw64\libexec\git-core\tcl86.dll Hash Sha1" Hash="BC0757E60B50D80C4DFB586F7E461A543C1C1C84" />
    <Allow ID="ID_ALLOW_A_BDF" FriendlyName="C:\Program Files\git\mingw64\libexec\git-core\tcl86.dll Hash Sha256" Hash="1E5F103232CC732946239CE6C8D9268858C33EB1DBB33334AA30AB9BEC4E7427" />
    <Allow ID="ID_ALLOW_A_BE0" FriendlyName="C:\Program Files\git\mingw64\libexec\git-core\tcl86.dll Hash Page Sha1" Hash="EFF0258D2AC4874C1AE0B1D960661D388BB4D61A" />
    <Allow ID="ID_ALLOW_A_BE1" FriendlyName="C:\Program Files\git\mingw64\libexec\git-core\tcl86.dll Hash Page Sha256" Hash="85C9EB3233B3445E6302B8AAAA4B4B92053066B105132B8D89EF12E952DAED2E" />
    <Allow ID="ID_ALLOW_A_BE2" FriendlyName="C:\Program Files\git\mingw64\libexec\git-core\tk86.dll Hash Sha1" Hash="2C7059300E4C88D44F0664EE43A9A352D262B6AF" />
    <Allow ID="ID_ALLOW_A_BE3" FriendlyName="C:\Program Files\git\mingw64\libexec\git-core\tk86.dll Hash Sha256" Hash="D02D0959793C0FE5F0AB893D3EB676D943A2029F813802021EAEB18FD57D9D3E" />
    <Allow ID="ID_ALLOW_A_BE4" FriendlyName="C:\Program Files\git\mingw64\libexec\git-core\tk86.dll Hash Page Sha1" Hash="9E39724FD4C46C570B5FABB040BE475620C3AF99" />
    <Allow ID="ID_ALLOW_A_BE5" FriendlyName="C:\Program Files\git\mingw64\libexec\git-core\tk86.dll Hash Page Sha256" Hash="41E08A07C0F5768DBD653CABC920D5A53497DD971F13E8C43AFAF24617899E19" />
    <Allow ID="ID_ALLOW_A_BE6" FriendlyName="C:\Program Files\git\mingw64\libexec\git-core\zlib1.dll Hash Sha1" Hash="9D2587363C7F3EB23829BA535B57B81E210CF066" />
    <Allow ID="ID_ALLOW_A_BE7" FriendlyName="C:\Program Files\git\mingw64\libexec\git-core\zlib1.dll Hash Sha256" Hash="6677E82D29270672FEFD9D634038A6D2CB91A5AA78CD5730334A589A41F88A5A" />
    <Allow ID="ID_ALLOW_A_BE8" FriendlyName="C:\Program Files\git\mingw64\libexec\git-core\zlib1.dll Hash Page Sha1" Hash="08884C6D8FA184BEFF9C795BC763A340DD2145EE" />
    <Allow ID="ID_ALLOW_A_BE9" FriendlyName="C:\Program Files\git\mingw64\libexec\git-core\zlib1.dll Hash Page Sha256" Hash="7FCC90C4C4EA183A2F3B931AE618931AF5183BAB5DFFBFCA4EF70A1F1CCAEDB0" />
    <Allow ID="ID_ALLOW_A_BEA" FriendlyName="C:\Program Files\git\mingw64\lib\thread2.8.5\thread285.dll Hash Sha1" Hash="5E70C47AA286F655E0530FA3F8A84CB43D80F8A0" />
    <Allow ID="ID_ALLOW_A_BEB" FriendlyName="C:\Program Files\git\mingw64\lib\thread2.8.5\thread285.dll Hash Sha256" Hash="37FBCCBFC843413EE2132F520DF7B96E9FE2EB2766E919938D3D67DFFB38F939" />
    <Allow ID="ID_ALLOW_A_BEC" FriendlyName="C:\Program Files\git\mingw64\lib\thread2.8.5\thread285.dll Hash Page Sha1" Hash="51CC91169A0DFBFE7E996A264C45EC05A5B8B5C0" />
    <Allow ID="ID_ALLOW_A_BED" FriendlyName="C:\Program Files\git\mingw64\lib\thread2.8.5\thread285.dll Hash Page Sha256" Hash="13803319247B19F3C91412D596C21D310D7FBC98E178CB6AFEB78589A8A41F53" />
    <Allow ID="ID_ALLOW_A_BEE" FriendlyName="C:\Program Files\git\mingw64\lib\reg1.3\tclreg13.dll Hash Sha1" Hash="A4FF2B5612F13A42D5342DA066DA548053A8C172" />
    <Allow ID="ID_ALLOW_A_BEF" FriendlyName="C:\Program Files\git\mingw64\lib\reg1.3\tclreg13.dll Hash Sha256" Hash="4D74C33F5AF15CD6EF167A5B94FA7A86A129358649C31C931762D86C0DFAFE33" />
    <Allow ID="ID_ALLOW_A_BF0" FriendlyName="C:\Program Files\git\mingw64\lib\reg1.3\tclreg13.dll Hash Page Sha1" Hash="FBA9534D14EDB73ACD63357383C6AC9482B116D9" />
    <Allow ID="ID_ALLOW_A_BF1" FriendlyName="C:\Program Files\git\mingw64\lib\reg1.3\tclreg13.dll Hash Page Sha256" Hash="CF897255B3E1E4410768A5D991ED3DF7C1350184F545901181FAFEB8EBF6B7DE" />
    <Allow ID="ID_ALLOW_A_BF2" FriendlyName="C:\Program Files\git\mingw64\lib\engines-1_1\capi.dll Hash Sha1" Hash="97A821DEEEFF18F35A1B4C680BDBF6DD22EB058D" />
    <Allow ID="ID_ALLOW_A_BF3" FriendlyName="C:\Program Files\git\mingw64\lib\engines-1_1\capi.dll Hash Sha256" Hash="507672ADEA4E57607D2D89FBD35835B649ACB4DD0188DD82F0056957754A3954" />
    <Allow ID="ID_ALLOW_A_BF4" FriendlyName="C:\Program Files\git\mingw64\lib\engines-1_1\capi.dll Hash Page Sha1" Hash="D18F9A619A7AE3408C979C980DF2BE3F37542E8F" />
    <Allow ID="ID_ALLOW_A_BF5" FriendlyName="C:\Program Files\git\mingw64\lib\engines-1_1\capi.dll Hash Page Sha256" Hash="61CD64C54F07052B7ADB0C189BC90D50C527F06975CFD486BC796335D29DDF87" />
    <Allow ID="ID_ALLOW_A_BF6" FriendlyName="C:\Program Files\git\mingw64\lib\engines-1_1\padlock.dll Hash Sha1" Hash="A966DDD76020315E203C2E810D5DA3E2EEB970EB" />
    <Allow ID="ID_ALLOW_A_BF7" FriendlyName="C:\Program Files\git\mingw64\lib\engines-1_1\padlock.dll Hash Sha256" Hash="3DEDBC78274CE5EE158B3B45FDE15EB4D1C17EE4CEA5E1392E544EFD333F4803" />
    <Allow ID="ID_ALLOW_A_BF8" FriendlyName="C:\Program Files\git\mingw64\lib\engines-1_1\padlock.dll Hash Page Sha1" Hash="DB90C0E44202FA569BBD08E7EC70F7B417137D70" />
    <Allow ID="ID_ALLOW_A_BF9" FriendlyName="C:\Program Files\git\mingw64\lib\engines-1_1\padlock.dll Hash Page Sha256" Hash="091703FE5099CE42CA7EB409D69E98647AC855A1C83402BEB4D12DE2622F5AD5" />
    <Allow ID="ID_ALLOW_A_BFA" FriendlyName="C:\Program Files\git\mingw64\bin\acountry.exe Hash Sha1" Hash="396A68F68A6FA03C1FE1041483DBF1121F31592B" />
    <Allow ID="ID_ALLOW_A_BFB" FriendlyName="C:\Program Files\git\mingw64\bin\acountry.exe Hash Sha256" Hash="4FEBA6CDA871860DCBF417EB5A2397AA3DEEF9F79C653CD01EF521338FF63A8E" />
    <Allow ID="ID_ALLOW_A_BFC" FriendlyName="C:\Program Files\git\mingw64\bin\acountry.exe Hash Page Sha1" Hash="D388F47D7302CE13F8E05B4725F65E90FBFEF2E8" />
    <Allow ID="ID_ALLOW_A_BFD" FriendlyName="C:\Program Files\git\mingw64\bin\acountry.exe Hash Page Sha256" Hash="ED5413AAC8178CB62136AF5275449F06D76A0D21942EAECCAE9112A3631DEA24" />
    <Allow ID="ID_ALLOW_A_BFE" FriendlyName="C:\Program Files\git\mingw64\bin\adig.exe Hash Sha1" Hash="13219CE3FD92CC0F26B18FDD183F0D771AD3B5F2" />
    <Allow ID="ID_ALLOW_A_BFF" FriendlyName="C:\Program Files\git\mingw64\bin\adig.exe Hash Sha256" Hash="F5C58D85D102F34E3FD971113DCE2B1EED9A06B1E2092116AE08076A1781C7DB" />
    <Allow ID="ID_ALLOW_A_C00" FriendlyName="C:\Program Files\git\mingw64\bin\adig.exe Hash Page Sha1" Hash="40DEBBF381026BD4C8F1E3BF5E7DB421D26327B6" />
    <Allow ID="ID_ALLOW_A_C01" FriendlyName="C:\Program Files\git\mingw64\bin\adig.exe Hash Page Sha256" Hash="3414076F410F3D4F90CBBAE2F0A5986A822426813C0C90CC8A0E8EFABDA3225C" />
    <Allow ID="ID_ALLOW_A_C02" FriendlyName="C:\Program Files\git\mingw64\bin\ahost.exe Hash Sha1" Hash="97A312B9CDB644FB185C1FD1C1D2578E71E74D6D" />
    <Allow ID="ID_ALLOW_A_C03" FriendlyName="C:\Program Files\git\mingw64\bin\ahost.exe Hash Sha256" Hash="11990B5A3CEFD93863E7FF97799EAD8C65BD133CB5404ADD8EACAA8365E45ECA" />
    <Allow ID="ID_ALLOW_A_C04" FriendlyName="C:\Program Files\git\mingw64\bin\ahost.exe Hash Page Sha1" Hash="1B579E218ACFE3965FF164B21A70E3CEC4E373F0" />
    <Allow ID="ID_ALLOW_A_C05" FriendlyName="C:\Program Files\git\mingw64\bin\ahost.exe Hash Page Sha256" Hash="9ACC8D8D743AF84080E8E8863DC762C303391E972D572681AAD1B859D230F78D" />
    <Allow ID="ID_ALLOW_A_C06" FriendlyName="C:\Program Files\git\mingw64\bin\antiword.exe Hash Sha1" Hash="A6554D3AE96DD16FCE519F7D2C7293E0FC65C77D" />
    <Allow ID="ID_ALLOW_A_C07" FriendlyName="C:\Program Files\git\mingw64\bin\antiword.exe Hash Sha256" Hash="442349966375EAA46BF386B7632180720CDD9E16AAFFAD108F65ED6AAA2283C9" />
    <Allow ID="ID_ALLOW_A_C08" FriendlyName="C:\Program Files\git\mingw64\bin\antiword.exe Hash Page Sha1" Hash="30686DF9AE03BA017E71C209664337F6023715CC" />
    <Allow ID="ID_ALLOW_A_C09" FriendlyName="C:\Program Files\git\mingw64\bin\antiword.exe Hash Page Sha256" Hash="72424DF2F2BAA3FC0612854E70B6D49A3AD187F49AE2415A6694D4F34E673F30" />
    <Allow ID="ID_ALLOW_A_C0A" FriendlyName="C:\Program Files\git\mingw64\bin\blocked-file-util.exe Hash Sha1" Hash="32392D117AF02730917F182684450FC4FF6A56AC" />
    <Allow ID="ID_ALLOW_A_C0B" FriendlyName="C:\Program Files\git\mingw64\bin\blocked-file-util.exe Hash Sha256" Hash="872D433A2FE787A3AA0876623821071FA4FF22C89FAFE570EC016F9458B08F72" />
    <Allow ID="ID_ALLOW_A_C0C" FriendlyName="C:\Program Files\git\mingw64\bin\blocked-file-util.exe Hash Page Sha1" Hash="B928B2543875D56E8F6B5959AA60A790A5E4F14B" />
    <Allow ID="ID_ALLOW_A_C0D" FriendlyName="C:\Program Files\git\mingw64\bin\blocked-file-util.exe Hash Page Sha256" Hash="587A38341BA81D3575E2119604E8459BB1DCA321EB91DE88BDF80F8DE7DA0BED" />
    <Allow ID="ID_ALLOW_A_C0E" FriendlyName="C:\Program Files\git\mingw64\bin\brotli.exe Hash Sha1" Hash="E1C7F08A491EF92B44140EE8283A19AD78C546F1" />
    <Allow ID="ID_ALLOW_A_C0F" FriendlyName="C:\Program Files\git\mingw64\bin\brotli.exe Hash Sha256" Hash="2D36F836A2F7E7185D5C67E4334E08F9B7F4F4E8064A8FD0EBF84520065E2924" />
    <Allow ID="ID_ALLOW_A_C10" FriendlyName="C:\Program Files\git\mingw64\bin\brotli.exe Hash Page Sha1" Hash="08323FDC28DC44C458FD765564D548C778EEE196" />
    <Allow ID="ID_ALLOW_A_C11" FriendlyName="C:\Program Files\git\mingw64\bin\brotli.exe Hash Page Sha256" Hash="30C04F63B08EBF9FBDFC941F6145BB59036E16FFFD48A1240F6A32F64BD4106E" />
    <Allow ID="ID_ALLOW_A_C12" FriendlyName="C:\Program Files\git\mingw64\bin\bunzip2.exe Hash Sha1" Hash="52D5DAA9D4C4E7E4ABFD2658FC3BFBE265A42E37" />
    <Allow ID="ID_ALLOW_A_C13" FriendlyName="C:\Program Files\git\mingw64\bin\bunzip2.exe Hash Sha256" Hash="E71CEFE477A0287E0A397C85E33937F62B5F1DD67BCCBE743B4E9944F32DF93A" />
    <Allow ID="ID_ALLOW_A_C14" FriendlyName="C:\Program Files\git\mingw64\bin\bunzip2.exe Hash Page Sha1" Hash="4A65BDBD8955D123207DD91D9BCCBCCA0A83E25A" />
    <Allow ID="ID_ALLOW_A_C15" FriendlyName="C:\Program Files\git\mingw64\bin\bunzip2.exe Hash Page Sha256" Hash="743C9B221740E398DD421169DA523DC2A4EAD265E93F46EDC0546602074F82A6" />
    <Allow ID="ID_ALLOW_A_C16" FriendlyName="C:\Program Files\git\mingw64\bin\bzip2recover.exe Hash Sha1" Hash="158757498FB3237AA25534E7991ACA01FB73BA5A" />
    <Allow ID="ID_ALLOW_A_C17" FriendlyName="C:\Program Files\git\mingw64\bin\bzip2recover.exe Hash Sha256" Hash="54F1037CD9AD2E79D4F3ABF6485B0B8E3A1D1769CD0F371790052F54C4651993" />
    <Allow ID="ID_ALLOW_A_C18" FriendlyName="C:\Program Files\git\mingw64\bin\bzip2recover.exe Hash Page Sha1" Hash="A6D949D13B3623E297CD86EEBD5E0418C506CEF5" />
    <Allow ID="ID_ALLOW_A_C19" FriendlyName="C:\Program Files\git\mingw64\bin\bzip2recover.exe Hash Page Sha256" Hash="D6B739D1FE4B00FF91EBE1FB6CA1FCCAF6AF9ACD6091EDEA123A9FFE892A3A93" />
    <Allow ID="ID_ALLOW_A_C1A" FriendlyName="C:\Program Files\git\mingw64\bin\connect.exe Hash Sha1" Hash="D3FD454563F72C35BB4A616EAB533D8C0349F459" />
    <Allow ID="ID_ALLOW_A_C1B" FriendlyName="C:\Program Files\git\mingw64\bin\connect.exe Hash Sha256" Hash="A69351F6FCFFA5BF34AFE52A52E470E4A8B6995F97CD285DBBF101FD291BD6CB" />
    <Allow ID="ID_ALLOW_A_C1C" FriendlyName="C:\Program Files\git\mingw64\bin\connect.exe Hash Page Sha1" Hash="792E4B30C5236E99C046EE4BBB01C32086550D6A" />
    <Allow ID="ID_ALLOW_A_C1D" FriendlyName="C:\Program Files\git\mingw64\bin\connect.exe Hash Page Sha256" Hash="D334478C784C4E31C3AC487A2E9544D3EDCC256059E25CE3A1DBFD7A3249FE26" />
    <Allow ID="ID_ALLOW_A_C1E" FriendlyName="C:\Program Files\git\mingw64\bin\create-shortcut.exe Hash Sha1" Hash="5B4B9DB0B1F077DAD1CEA0057EC6768D9F12CAAB" />
    <Allow ID="ID_ALLOW_A_C1F" FriendlyName="C:\Program Files\git\mingw64\bin\create-shortcut.exe Hash Sha256" Hash="092F22F95E393B9B8BFC15AEE9C600DB561A25840AB0207926AE6B1DA4EC585E" />
    <Allow ID="ID_ALLOW_A_C20" FriendlyName="C:\Program Files\git\mingw64\bin\create-shortcut.exe Hash Page Sha1" Hash="3DB1CBEB74D51443DE561D59AD446E353F9B4ADB" />
    <Allow ID="ID_ALLOW_A_C21" FriendlyName="C:\Program Files\git\mingw64\bin\create-shortcut.exe Hash Page Sha256" Hash="7E52D7355A3DEE4C07EADC6EBF650A5D2C55A3D4E8D70B36755718E341479C1E" />
    <Allow ID="ID_ALLOW_A_C22" FriendlyName="C:\Program Files\git\mingw64\bin\curl.exe Hash Sha1" Hash="C73F693FC78FA58BAAE507EE6FF15B2B003DC7D3" />
    <Allow ID="ID_ALLOW_A_C23" FriendlyName="C:\Program Files\git\mingw64\bin\curl.exe Hash Sha256" Hash="331AAF785A3B233509E42AB1B392C8B3BC46D6A54C9B1ABEB2A9C238611463B9" />
    <Allow ID="ID_ALLOW_A_C24" FriendlyName="C:\Program Files\git\mingw64\bin\curl.exe Hash Page Sha1" Hash="3DFA0B405EFB5C64618F9E9F0D0606A727E24D56" />
    <Allow ID="ID_ALLOW_A_C25" FriendlyName="C:\Program Files\git\mingw64\bin\curl.exe Hash Page Sha256" Hash="644BFCD760A310D650673093FEF9B4D0C18B1EEF90ADFA04EC0675B9788B9AB5" />
    <Allow ID="ID_ALLOW_A_C26" FriendlyName="C:\Program Files\git\mingw64\bin\edit_test.exe Hash Sha1" Hash="A74802059D30A0626CF0BBFB1694F7DC066A2CC0" />
    <Allow ID="ID_ALLOW_A_C27" FriendlyName="C:\Program Files\git\mingw64\bin\edit_test.exe Hash Sha256" Hash="946FA68AC3722E0087B48D35FFA3513505CC11F938708493DD131ABE4EF18955" />
    <Allow ID="ID_ALLOW_A_C28" FriendlyName="C:\Program Files\git\mingw64\bin\edit_test.exe Hash Page Sha1" Hash="CFFE92B703670A702884AF796E35361D6DCC3795" />
    <Allow ID="ID_ALLOW_A_C29" FriendlyName="C:\Program Files\git\mingw64\bin\edit_test.exe Hash Page Sha256" Hash="CA438EA4F7C3376151732D74B52A89DF87BF62E86E20B1989C312EB65E0B3675" />
    <Allow ID="ID_ALLOW_A_C2A" FriendlyName="C:\Program Files\git\mingw64\bin\edit_test_dll.exe Hash Sha1" Hash="06914CE9C375DC3966343CB0C4CE39BB38E05117" />
    <Allow ID="ID_ALLOW_A_C2B" FriendlyName="C:\Program Files\git\mingw64\bin\edit_test_dll.exe Hash Sha256" Hash="CD898AD64B2392349DE7F536B29C5929D4F685B1E33D4CAC869540F0E2A18ADA" />
    <Allow ID="ID_ALLOW_A_C2C" FriendlyName="C:\Program Files\git\mingw64\bin\edit_test_dll.exe Hash Page Sha1" Hash="5D7A4CE5305A26BEA007C51EC993FAB18BC3845E" />
    <Allow ID="ID_ALLOW_A_C2D" FriendlyName="C:\Program Files\git\mingw64\bin\edit_test_dll.exe Hash Page Sha256" Hash="F8376B65D7FB316E8880EB305014A28BFF18E5D83E516DBDFE7C2E9985F6EE38" />
    <Allow ID="ID_ALLOW_A_C2E" FriendlyName="C:\Program Files\git\mingw64\bin\envsubst.exe Hash Sha1" Hash="DA1E3A76F7301059E65FF85C93F3B0C5A8D9B130" />
    <Allow ID="ID_ALLOW_A_C2F" FriendlyName="C:\Program Files\git\mingw64\bin\envsubst.exe Hash Sha256" Hash="74D6F403DEF90AD8DC8F4F795C2EFD5A4A6C6BAC806372119B04F769C8AA6F18" />
    <Allow ID="ID_ALLOW_A_C30" FriendlyName="C:\Program Files\git\mingw64\bin\envsubst.exe Hash Page Sha1" Hash="3066E3156CB3FC14660D9D2A01F786DDC9E470A4" />
    <Allow ID="ID_ALLOW_A_C31" FriendlyName="C:\Program Files\git\mingw64\bin\envsubst.exe Hash Page Sha256" Hash="DB4BFE21D48E5326F8BAE7069AB01A8FAEFE54870335B0B9478BA42D47CF84E9" />
    <Allow ID="ID_ALLOW_A_C32" FriendlyName="C:\Program Files\git\mingw64\bin\gettext.exe Hash Sha1" Hash="F018FD6D5998B33B7F7C06ACFC61DE26970AD618" />
    <Allow ID="ID_ALLOW_A_C33" FriendlyName="C:\Program Files\git\mingw64\bin\gettext.exe Hash Sha256" Hash="D861CACD6804C67FA90BEDCC5EFB49989549E7CE949BDC6B195B31850B3724FA" />
    <Allow ID="ID_ALLOW_A_C34" FriendlyName="C:\Program Files\git\mingw64\bin\gettext.exe Hash Page Sha1" Hash="571632F6026026144B7BEB9060F81FDCC10B38B7" />
    <Allow ID="ID_ALLOW_A_C35" FriendlyName="C:\Program Files\git\mingw64\bin\gettext.exe Hash Page Sha256" Hash="0CAEA4C22300FA043977418AC0F73EA34069ABD72420C4C605FA461733D3E04B" />
    <Allow ID="ID_ALLOW_A_C36" FriendlyName="C:\Program Files\git\mingw64\bin\git-askyesno.exe Hash Sha1" Hash="CE8008659F5E2535EF7FE5DE939B819335EA6830" />
    <Allow ID="ID_ALLOW_A_C37" FriendlyName="C:\Program Files\git\mingw64\bin\git-askyesno.exe Hash Sha256" Hash="CB4CC1137292B90A08A19D91E024D1BCE0BE4FC1207420EC0B6D622CF74B9D6C" />
    <Allow ID="ID_ALLOW_A_C38" FriendlyName="C:\Program Files\git\mingw64\bin\git-askyesno.exe Hash Page Sha1" Hash="2C6B27A0C7EDEC5315FE5FC9E055AEF0F13A3D0C" />
    <Allow ID="ID_ALLOW_A_C39" FriendlyName="C:\Program Files\git\mingw64\bin\git-askyesno.exe Hash Page Sha256" Hash="FB7D96D78D9D81C50BB2E1CC885AEF1F5FAA00D15DAC164FC4DE753020816779" />
    <Allow ID="ID_ALLOW_A_C3A" FriendlyName="C:\Program Files\git\mingw64\bin\git-credential-helper-selector.exe Hash Sha1" Hash="64467C923759849B4892A15206092EB4AF0178B5" />
    <Allow ID="ID_ALLOW_A_C3B" FriendlyName="C:\Program Files\git\mingw64\bin\git-credential-helper-selector.exe Hash Sha256" Hash="E687BB6DFF8A2991713BCBF1CEC0B447A536DA1E43C3982F14D655E2520342EB" />
    <Allow ID="ID_ALLOW_A_C3C" FriendlyName="C:\Program Files\git\mingw64\bin\git-credential-helper-selector.exe Hash Page Sha1" Hash="E23CC4F20CC2065C5EA7FCB6F4925D08D331A384" />
    <Allow ID="ID_ALLOW_A_C3D" FriendlyName="C:\Program Files\git\mingw64\bin\git-credential-helper-selector.exe Hash Page Sha256" Hash="0E207718E53D147BF0EB5E102E55E96C886AFE5DF20CFCE6EE2F993E12CBC602" />
    <Allow ID="ID_ALLOW_A_C3E" FriendlyName="C:\Program Files\git\mingw64\bin\lzmadec.exe Hash Sha1" Hash="6B1EF7F78B6AF7CF380BAC321250F6ED9E29B5E4" />
    <Allow ID="ID_ALLOW_A_C3F" FriendlyName="C:\Program Files\git\mingw64\bin\lzmadec.exe Hash Sha256" Hash="3DC5D6ECA5A7AFAA3C281BB3B80FD0B2F7B64AB85DC91927C2D7C1502B46BB83" />
    <Allow ID="ID_ALLOW_A_C40" FriendlyName="C:\Program Files\git\mingw64\bin\lzmadec.exe Hash Page Sha1" Hash="1D093C2C763743B30961A7746DEF3D6E79D3DF78" />
    <Allow ID="ID_ALLOW_A_C41" FriendlyName="C:\Program Files\git\mingw64\bin\lzmadec.exe Hash Page Sha256" Hash="35492BD370840D0307EA71B63518537A4FBB67426E64B7CD426F98D88FBD1C50" />
    <Allow ID="ID_ALLOW_A_C42" FriendlyName="C:\Program Files\git\mingw64\bin\lzmainfo.exe Hash Sha1" Hash="B537A58286CF3D42FD962CA1E75295E4A1194417" />
    <Allow ID="ID_ALLOW_A_C43" FriendlyName="C:\Program Files\git\mingw64\bin\lzmainfo.exe Hash Sha256" Hash="CC720DE671F3994DF9BF9F438C567FAAA23A30FC7625BCE14487CA95D7F762B5" />
    <Allow ID="ID_ALLOW_A_C44" FriendlyName="C:\Program Files\git\mingw64\bin\lzmainfo.exe Hash Page Sha1" Hash="941DD6EEADF69C732E32AFF1D2E92C40BEEE0264" />
    <Allow ID="ID_ALLOW_A_C45" FriendlyName="C:\Program Files\git\mingw64\bin\lzmainfo.exe Hash Page Sha256" Hash="0DD32BE797368FB39F681716AB0D2C485BD74282C3356F0B3A4158CF213F62A9" />
    <Allow ID="ID_ALLOW_A_C46" FriendlyName="C:\Program Files\git\mingw64\bin\odt2txt.exe Hash Sha1" Hash="15B9EBCF08B79066AC5663FFB1B6A75AE20EA22B" />
    <Allow ID="ID_ALLOW_A_C47" FriendlyName="C:\Program Files\git\mingw64\bin\odt2txt.exe Hash Sha256" Hash="D6BEB7315FA993FEB2F587E8CD63667BC2DBE4A805414ACC36CC44A312391FAC" />
    <Allow ID="ID_ALLOW_A_C48" FriendlyName="C:\Program Files\git\mingw64\bin\odt2txt.exe Hash Page Sha1" Hash="E05567ADC08580B74FB9C8E0E25EA59CA21F32A7" />
    <Allow ID="ID_ALLOW_A_C49" FriendlyName="C:\Program Files\git\mingw64\bin\odt2txt.exe Hash Page Sha256" Hash="CD4F45CC08D7B900A5B83FC95706D7974460A96D6985EE9CAEDA1273D3B57FC1" />
    <Allow ID="ID_ALLOW_A_C4A" FriendlyName="C:\Program Files\git\mingw64\bin\openssl.exe Hash Sha1" Hash="15389F2FB2E20531588D82D728FF2BAEBD62263B" />
    <Allow ID="ID_ALLOW_A_C4B" FriendlyName="C:\Program Files\git\mingw64\bin\openssl.exe Hash Sha256" Hash="27D3132EEA49ADFD2D4E6320F050707EF75516A27CEA1E986175F61164282833" />
    <Allow ID="ID_ALLOW_A_C4C" FriendlyName="C:\Program Files\git\mingw64\bin\openssl.exe Hash Page Sha1" Hash="7FE32E6376496C54792CF501F89FA2B75898725A" />
    <Allow ID="ID_ALLOW_A_C4D" FriendlyName="C:\Program Files\git\mingw64\bin\openssl.exe Hash Page Sha256" Hash="99B5653BD85B5346DC1FFF6F28DB2A0AA274D34B8C55B14F20282C98BF4C2736" />
    <Allow ID="ID_ALLOW_A_C4E" FriendlyName="C:\Program Files\git\mingw64\bin\pdftotext.exe Hash Sha1" Hash="79934E0E989DE906B1F014399F4CA362C0B93FCE" />
    <Allow ID="ID_ALLOW_A_C4F" FriendlyName="C:\Program Files\git\mingw64\bin\pdftotext.exe Hash Sha256" Hash="E66B3D742022759940C2111D2BF141C6DEDCA8A9D9DCDE22F32897EBE4D06ED2" />
    <Allow ID="ID_ALLOW_A_C50" FriendlyName="C:\Program Files\git\mingw64\bin\pdftotext.exe Hash Page Sha1" Hash="7D793A58AE1BA82B02AD91AF677E9EDAECC42DB2" />
    <Allow ID="ID_ALLOW_A_C51" FriendlyName="C:\Program Files\git\mingw64\bin\pdftotext.exe Hash Page Sha256" Hash="BFCECEAFB95FB175D1091974F19BFE7005BEB5EF5EA0E867FCB63D185AAB08CA" />
    <Allow ID="ID_ALLOW_A_C52" FriendlyName="C:\Program Files\git\mingw64\bin\pkcs1-conv.exe Hash Sha1" Hash="ACE83BFEEBC09A8D968FD42B2F9D42F6E3900718" />
    <Allow ID="ID_ALLOW_A_C53" FriendlyName="C:\Program Files\git\mingw64\bin\pkcs1-conv.exe Hash Sha256" Hash="0ECDF0F07899CD43D639C66A374B08E2F92AA6D7631D352DC229CB880C7CCFF6" />
    <Allow ID="ID_ALLOW_A_C54" FriendlyName="C:\Program Files\git\mingw64\bin\pkcs1-conv.exe Hash Page Sha1" Hash="B994C3AC05146AD8F9562C0C8ACB750DEC4E7722" />
    <Allow ID="ID_ALLOW_A_C55" FriendlyName="C:\Program Files\git\mingw64\bin\pkcs1-conv.exe Hash Page Sha256" Hash="23AE8D0877989E25CBA94AB76467E00CE9669BD1F4F33F831E9010D8A64FA2C5" />
    <Allow ID="ID_ALLOW_A_C56" FriendlyName="C:\Program Files\git\mingw64\bin\proxy-lookup.exe Hash Sha1" Hash="DF35828AA7C365EC8153C529BEF924B6D0F2C7B0" />
    <Allow ID="ID_ALLOW_A_C57" FriendlyName="C:\Program Files\git\mingw64\bin\proxy-lookup.exe Hash Sha256" Hash="418ECAA7D95E0B97F724F40AE2D939F3830DDF5522DB8E2E37CAA89DE641BF48" />
    <Allow ID="ID_ALLOW_A_C58" FriendlyName="C:\Program Files\git\mingw64\bin\proxy-lookup.exe Hash Page Sha1" Hash="58B35A4F6A7351F41C366E22E31683A9C136A2F2" />
    <Allow ID="ID_ALLOW_A_C59" FriendlyName="C:\Program Files\git\mingw64\bin\proxy-lookup.exe Hash Page Sha256" Hash="9B08DB9A9EAC02F72CE40B6E614A5A680DB07A5E813F15D55A71B57AAF8776E3" />
    <Allow ID="ID_ALLOW_A_C5A" FriendlyName="C:\Program Files\git\mingw64\bin\sexp-conv.exe Hash Sha1" Hash="39CDDAE80DABE95706548E338677CA462E22D278" />
    <Allow ID="ID_ALLOW_A_C5B" FriendlyName="C:\Program Files\git\mingw64\bin\sexp-conv.exe Hash Sha256" Hash="3BF80ECAD40B7D972CF75107A67299F8ABE11C82E0AC810C577F41B603E446BB" />
    <Allow ID="ID_ALLOW_A_C5C" FriendlyName="C:\Program Files\git\mingw64\bin\sexp-conv.exe Hash Page Sha1" Hash="7E0E92C3DD71F893DE2A994DD87420FE4F9A9995" />
    <Allow ID="ID_ALLOW_A_C5D" FriendlyName="C:\Program Files\git\mingw64\bin\sexp-conv.exe Hash Page Sha256" Hash="3E9B3BE4120488B6F262E5C975A60F63F10B4BC471502C9117BA50DE38486155" />
    <Allow ID="ID_ALLOW_A_C5E" FriendlyName="C:\Program Files\git\mingw64\bin\tclsh.exe Hash Sha1" Hash="2525D699FF9B83752098F1133DA569C43C9D888F" />
    <Allow ID="ID_ALLOW_A_C5F" FriendlyName="C:\Program Files\git\mingw64\bin\tclsh.exe Hash Sha256" Hash="9FF9345D60B0CA4F5655E3BF186207F2FD2BBA6257B665436C6C635C95758F6B" />
    <Allow ID="ID_ALLOW_A_C60" FriendlyName="C:\Program Files\git\mingw64\bin\tclsh.exe Hash Page Sha1" Hash="E0ADD5FD056AA8D482B64511D3B57AA971B0BE32" />
    <Allow ID="ID_ALLOW_A_C61" FriendlyName="C:\Program Files\git\mingw64\bin\tclsh.exe Hash Page Sha256" Hash="D213565B6E499939304B4A9C3C3AC275FD8CEB368C833183AE2C0D926616FB55" />
    <Allow ID="ID_ALLOW_A_C62" FriendlyName="C:\Program Files\git\mingw64\bin\unxz.exe Hash Sha1" Hash="4ACF2A385A7764472E40AF3625C4EB67DD2A2993" />
    <Allow ID="ID_ALLOW_A_C63" FriendlyName="C:\Program Files\git\mingw64\bin\unxz.exe Hash Sha256" Hash="8817347C5BDB1275C8B7906F16F59E63ED63F87B182D8548E02CD7D895400397" />
    <Allow ID="ID_ALLOW_A_C64" FriendlyName="C:\Program Files\git\mingw64\bin\unxz.exe Hash Page Sha1" Hash="F7B382F29D771E72E4BB6D8C40E6E39FD499C02D" />
    <Allow ID="ID_ALLOW_A_C65" FriendlyName="C:\Program Files\git\mingw64\bin\unxz.exe Hash Page Sha256" Hash="BE3500EE756A0AE58D2F4E411A452A747F27CFC2908052701BF7FECF0156AFCE" />
    <Allow ID="ID_ALLOW_A_C66" FriendlyName="C:\Program Files\git\mingw64\bin\WhoUses.exe Hash Sha1" Hash="CE8833301B9EDBA71201A740E20355B7E5A80626" />
    <Allow ID="ID_ALLOW_A_C67" FriendlyName="C:\Program Files\git\mingw64\bin\WhoUses.exe Hash Sha256" Hash="A5B430B4860ED832AC61A0E2AE3FC611397A49C3CC5605317655A80E4401E968" />
    <Allow ID="ID_ALLOW_A_C68" FriendlyName="C:\Program Files\git\mingw64\bin\WhoUses.exe Hash Page Sha1" Hash="E774A5A313D0C3C9730E11E262346E6900D367FF" />
    <Allow ID="ID_ALLOW_A_C69" FriendlyName="C:\Program Files\git\mingw64\bin\WhoUses.exe Hash Page Sha256" Hash="6E3FD31080FB4CF08B81704E8C3ADFBFB196490C9E04037BE0B304962E948366" />
    <Allow ID="ID_ALLOW_A_C6A" FriendlyName="C:\Program Files\git\mingw64\bin\wish.exe Hash Sha1" Hash="A7EF3AF0967E1384CB019E3C5BA1D4FEBFC70C43" />
    <Allow ID="ID_ALLOW_A_C6B" FriendlyName="C:\Program Files\git\mingw64\bin\wish.exe Hash Sha256" Hash="AA21909D01881131BBC251C7236BF4E8EA74BD82168763B8DFB59B3FFFE10BD4" />
    <Allow ID="ID_ALLOW_A_C6C" FriendlyName="C:\Program Files\git\mingw64\bin\wish.exe Hash Page Sha1" Hash="5014D5DFE1D70F633C7DF41C8A55EF5F6543EC1B" />
    <Allow ID="ID_ALLOW_A_C6D" FriendlyName="C:\Program Files\git\mingw64\bin\wish.exe Hash Page Sha256" Hash="9D4B5853BB4A03E01480948B5C439C94383D70ADE1EF88CAFDEA66625D10E051" />
    <Allow ID="ID_ALLOW_A_C6E" FriendlyName="C:\Program Files\git\mingw64\bin\x86_64-w64-mingw32-agrep.exe Hash Sha1" Hash="5FD92B103B5202630F38F86719250EB93851BD27" />
    <Allow ID="ID_ALLOW_A_C6F" FriendlyName="C:\Program Files\git\mingw64\bin\x86_64-w64-mingw32-agrep.exe Hash Sha256" Hash="93A6D9BD939728C1A7293FAF322FED5B0411AC1F9CAAFC848B6999FF784242D0" />
    <Allow ID="ID_ALLOW_A_C70" FriendlyName="C:\Program Files\git\mingw64\bin\x86_64-w64-mingw32-agrep.exe Hash Page Sha1" Hash="521987FB8DDE4CDB3893C2B19D93578EFD06433A" />
    <Allow ID="ID_ALLOW_A_C71" FriendlyName="C:\Program Files\git\mingw64\bin\x86_64-w64-mingw32-agrep.exe Hash Page Sha256" Hash="374D18E0D48C49D0ED8DC0B34AC38AE7858A955483F38253E37A179265D7E2A2" />
    <Allow ID="ID_ALLOW_A_C72" FriendlyName="C:\Program Files\git\mingw64\bin\x86_64-w64-mingw32-deflatehd.exe Hash Sha1" Hash="025BB004D2893C8D30028CF4D7BE2C7A5C9C67FB" />
    <Allow ID="ID_ALLOW_A_C73" FriendlyName="C:\Program Files\git\mingw64\bin\x86_64-w64-mingw32-deflatehd.exe Hash Sha256" Hash="6DBEFED1780D966D749253487DEE039EACFFFEFDC42DD2EDFA8FF7848411FD35" />
    <Allow ID="ID_ALLOW_A_C74" FriendlyName="C:\Program Files\git\mingw64\bin\x86_64-w64-mingw32-deflatehd.exe Hash Page Sha1" Hash="98577F70A1113F6F5532668982AB97B16B29B062" />
    <Allow ID="ID_ALLOW_A_C75" FriendlyName="C:\Program Files\git\mingw64\bin\x86_64-w64-mingw32-deflatehd.exe Hash Page Sha256" Hash="B7E3E306C63D9ECD2EADA4822713717CFDC0ACE987DDC95FFA2CD43A291C5B5B" />
    <Allow ID="ID_ALLOW_A_C76" FriendlyName="C:\Program Files\git\mingw64\bin\x86_64-w64-mingw32-inflatehd.exe Hash Sha1" Hash="78A642A50EB61A0EF3680EAA6EAE8980A9864A85" />
    <Allow ID="ID_ALLOW_A_C77" FriendlyName="C:\Program Files\git\mingw64\bin\x86_64-w64-mingw32-inflatehd.exe Hash Sha256" Hash="9FD3928A5E6F4278E8C7C922E2AF230D6CDFDD0FBEB23833B341122155DADB5D" />
    <Allow ID="ID_ALLOW_A_C78" FriendlyName="C:\Program Files\git\mingw64\bin\x86_64-w64-mingw32-inflatehd.exe Hash Page Sha1" Hash="4E12898F2FB5C9FBEA0589CB362B128F288125B4" />
    <Allow ID="ID_ALLOW_A_C79" FriendlyName="C:\Program Files\git\mingw64\bin\x86_64-w64-mingw32-inflatehd.exe Hash Page Sha256" Hash="4D9AE69DE8B83387BD22507678CCCA7D35072A0AB1C0160ADB393B4F33AD38DA" />
    <Allow ID="ID_ALLOW_A_C7A" FriendlyName="C:\Program Files\git\mingw64\bin\xmlwf.exe Hash Sha1" Hash="0323C54943D191C793474766F9BC703DCCF5A883" />
    <Allow ID="ID_ALLOW_A_C7B" FriendlyName="C:\Program Files\git\mingw64\bin\xmlwf.exe Hash Sha256" Hash="23822804F11BCA53ED7D10791E0AF17D66AB91F8D5732E02A8F45AB8DBBDEE8C" />
    <Allow ID="ID_ALLOW_A_C7C" FriendlyName="C:\Program Files\git\mingw64\bin\xmlwf.exe Hash Page Sha1" Hash="B79D994B2B48A2BBB5AFCC551FBED236DDCCFD92" />
    <Allow ID="ID_ALLOW_A_C7D" FriendlyName="C:\Program Files\git\mingw64\bin\xmlwf.exe Hash Page Sha256" Hash="794A0FD7CF8E500F51F0B53AC183CF3CE1E49104BA71F730A273B8FAC5587B9E" />
    <Allow ID="ID_ALLOW_A_C7E" FriendlyName="C:\Program Files\git\mingw64\bin\xzdec.exe Hash Sha1" Hash="810F7A798B6F509173E99D65C0EEF56A1F671DD2" />
    <Allow ID="ID_ALLOW_A_C7F" FriendlyName="C:\Program Files\git\mingw64\bin\xzdec.exe Hash Sha256" Hash="FFCDE89BB0BD63070AD317EF3A9AEE0C665E83AEFED7F256A01D9991B80C39D7" />
    <Allow ID="ID_ALLOW_A_C80" FriendlyName="C:\Program Files\git\mingw64\bin\xzdec.exe Hash Page Sha1" Hash="7F4519C22894088B68958B66A4DA2B32DB3092D0" />
    <Allow ID="ID_ALLOW_A_C81" FriendlyName="C:\Program Files\git\mingw64\bin\xzdec.exe Hash Page Sha256" Hash="B15DA1619089C9E2A3816528935B11AF430DCBF7000D000380783FE98713A473" />
    <Allow ID="ID_ALLOW_A_C82" FriendlyName="C:\Program Files\git\mingw64\bin\zipcmp.exe Hash Sha1" Hash="00A0B85899EE2C07E040ACA369CE5991D9D1E649" />
    <Allow ID="ID_ALLOW_A_C83" FriendlyName="C:\Program Files\git\mingw64\bin\zipcmp.exe Hash Sha256" Hash="3F36EAE59A41566D0C71710062D915CD498EE2BA0629778248B9A5C74D1D9EF5" />
    <Allow ID="ID_ALLOW_A_C84" FriendlyName="C:\Program Files\git\mingw64\bin\zipcmp.exe Hash Page Sha1" Hash="927DADED11FA2D1740CC8883F262D40FBF88A5CD" />
    <Allow ID="ID_ALLOW_A_C85" FriendlyName="C:\Program Files\git\mingw64\bin\zipcmp.exe Hash Page Sha256" Hash="FE098EB9CD73CC5C28C7746AC05E253751B305ED6D6AB0DEBDF3F8591AA47B57" />
    <Allow ID="ID_ALLOW_A_C86" FriendlyName="C:\Program Files\git\mingw64\bin\zipmerge.exe Hash Sha1" Hash="DF6149C75A5E0F832413ED55DC345B6340BEA34A" />
    <Allow ID="ID_ALLOW_A_C87" FriendlyName="C:\Program Files\git\mingw64\bin\zipmerge.exe Hash Sha256" Hash="368EDA95CC316C40D03ED76288F046A4B4D107C7D89F86659645F923C8FEE960" />
    <Allow ID="ID_ALLOW_A_C88" FriendlyName="C:\Program Files\git\mingw64\bin\zipmerge.exe Hash Page Sha1" Hash="5EF68C181E59434299C103A062E8E6D785760DB0" />
    <Allow ID="ID_ALLOW_A_C89" FriendlyName="C:\Program Files\git\mingw64\bin\zipmerge.exe Hash Page Sha256" Hash="2A0B3816FCA1700E2876E8C3F42F984B07611553D729D720321BAFC39EB9ED70" />
    <Allow ID="ID_ALLOW_A_C8A" FriendlyName="C:\Program Files\git\mingw64\bin\ziptool.exe Hash Sha1" Hash="13A5FE964DFBA02F5C82B7385BAA3F689E66AD9B" />
    <Allow ID="ID_ALLOW_A_C8B" FriendlyName="C:\Program Files\git\mingw64\bin\ziptool.exe Hash Sha256" Hash="04489496959CBD94D02B4F24C18BA0A06EA2A19675D5C696E5ACD9F68456A373" />
    <Allow ID="ID_ALLOW_A_C8C" FriendlyName="C:\Program Files\git\mingw64\bin\ziptool.exe Hash Page Sha1" Hash="F38CE3B1C7C315A4E245CF0AC821C342C20A1579" />
    <Allow ID="ID_ALLOW_A_C8D" FriendlyName="C:\Program Files\git\mingw64\bin\ziptool.exe Hash Page Sha256" Hash="6982B2555C0FAFD99C14CB49B6A52E5A41B8BD239DAF7F7B79AC10C6105C4819" />
  </FileRules>
  <!--Signers-->
  <Signers>
    <Signer ID="ID_SIGNER_S_1D3" Name="USERTrust RSA Certification Authority">
      <CertRoot Type="TBS" Value="13BAA039635F1C5292A8C2F36AAE7E1D25C025202E9092F5B0F53F5F752DFA9C71B3D1B8D9A6358FCEE6EC75622FABF9" />
      <CertPublisher Value="Johannes Schindelin" />
    </Signer>
    <Signer ID="ID_SIGNER_S_1D7" Name="Microsoft Code Signing PCA 2011">
      <CertRoot Type="TBS" Value="F6F717A43AD9ABDDC8CEFDDE1C505462535E7D1307E630F9544A2D14FE8BF26E" />
      <CertPublisher Value="Microsoft Corporation" />
    </Signer>
    <Signer ID="ID_SIGNER_S_1D8" Name="Microsoft Code Signing PCA 2011">
      <CertRoot Type="TBS" Value="F6F717A43AD9ABDDC8CEFDDE1C505462535E7D1307E630F9544A2D14FE8BF26E" />
      <CertPublisher Value="Microsoft Corporation" />
    </Signer>
    <Signer ID="ID_SIGNER_S_281" Name="Microsoft Code Signing PCA">
      <CertRoot Type="TBS" Value="27543A3F7612DE2261C7228321722402F63A07DE" />
      <CertPublisher Value="Microsoft Corporation" />
    </Signer>
    <Signer ID="ID_SIGNER_S_283" Name="Microsoft Code Signing PCA">
      <CertRoot Type="TBS" Value="27543A3F7612DE2261C7228321722402F63A07DE" />
      <CertPublisher Value="Microsoft Corporation" />
    </Signer>
    <Signer ID="ID_SIGNER_S_285" Name="Microsoft Code Signing PCA">
      <CertRoot Type="TBS" Value="122F29FFE889713C1568EF7A0F663373C98570E8" />
      <CertPublisher Value="Microsoft Corporation" />
    </Signer>
    <Signer ID="ID_SIGNER_S_287" Name="Microsoft Code Signing PCA">
      <CertRoot Type="TBS" Value="122F29FFE889713C1568EF7A0F663373C98570E8" />
      <CertPublisher Value="Microsoft Corporation" />
    </Signer>
    <Signer ID="ID_SIGNER_S_291" Name=".NET Foundation Projects Code Signing CA">
      <CertRoot Type="TBS" Value="B6D27A37A95C0A174C66496B3225B11A6250846BBC31FCA10D69D61F92DF5084" />
      <CertPublisher Value="Json.NET (.NET Foundation)" />
    </Signer>
    <Signer ID="ID_SIGNER_S_292" Name="Microsoft Windows Third Party Component CA 2012">
      <CertRoot Type="TBS" Value="CEC1AFD0E310C55C1DCC601AB8E172917706AA32FB5EAF826813547FDF02DD46" />
      <CertPublisher Value="Microsoft Windows Hardware Compatibility Publisher" />
    </Signer>
    <Signer ID="ID_SIGNER_S_2A0" Name=".NET Foundation Projects Code Signing CA">
      <CertRoot Type="TBS" Value="B6D27A37A95C0A174C66496B3225B11A6250846BBC31FCA10D69D61F92DF5084" />
      <CertPublisher Value="Json.NET (.NET Foundation)" />
    </Signer>
    <Signer ID="ID_SIGNER_S_2A1" Name="Microsoft Windows Third Party Component CA 2012">
      <CertRoot Type="TBS" Value="CEC1AFD0E310C55C1DCC601AB8E172917706AA32FB5EAF826813547FDF02DD46" />
      <CertPublisher Value="Microsoft Windows Hardware Compatibility Publisher" />
    </Signer>
    <Signer ID="ID_SIGNER_S_2AF" Name="DigiCert SHA2 Assured ID Code Signing CA">
      <CertRoot Type="TBS" Value="E767799478F64A34B3F53FF3BB9057FE1768F4AB178041B0DCC0FF1E210CBA65" />
      <CertPublisher Value="GitHub, Inc." />
    </Signer>
    <Signer ID="ID_SIGNER_S_2B4" Name="COMODO RSA Code Signing CA">
      <CertRoot Type="TBS" Value="4805A7E23D6C8FF5E149F197B744BCB2346E73F19A48835A2F64129183981109256B75EA371A331746D01FD4E135AB6E" />
      <CertPublisher Value="Johannes Schindelin" />
    </Signer>
  </Signers>
  <!--Driver Signing Scenarios-->
  <SigningScenarios>
    <SigningScenario Value="131" ID="ID_SIGNINGSCENARIO_DRIVERS_1" FriendlyName="Auto generated policy on 04-12-2021">
      <ProductSigners>
        <FileRulesRef>
          <FileRuleRef RuleID="ID_ALLOW_A_B52" />
          <FileRuleRef RuleID="ID_ALLOW_A_B53" />
          <FileRuleRef RuleID="ID_ALLOW_A_B54" />
          <FileRuleRef RuleID="ID_ALLOW_A_B55" />
          <FileRuleRef RuleID="ID_ALLOW_A_B5E" />
          <FileRuleRef RuleID="ID_ALLOW_A_B5F" />
          <FileRuleRef RuleID="ID_ALLOW_A_B60" />
          <FileRuleRef RuleID="ID_ALLOW_A_B61" />
        </FileRulesRef>
        <AllowedSigners>
          <AllowedSigner SignerId="ID_SIGNER_S_1D7" />
          <AllowedSigner SignerId="ID_SIGNER_S_281" />
          <AllowedSigner SignerId="ID_SIGNER_S_285" />
          <AllowedSigner SignerId="ID_SIGNER_S_291" />
          <AllowedSigner SignerId="ID_SIGNER_S_292" />
        </AllowedSigners>
      </ProductSigners>
    </SigningScenario>
    <SigningScenario Value="12" ID="ID_SIGNINGSCENARIO_WINDOWS" FriendlyName="Auto generated policy on 04-12-2021">
      <ProductSigners>
        <FileRulesRef>
          <FileRuleRef RuleID="ID_ALLOW_A_48A" />
          <FileRuleRef RuleID="ID_ALLOW_A_48B" />
          <FileRuleRef RuleID="ID_ALLOW_A_48C" />
          <FileRuleRef RuleID="ID_ALLOW_A_48D" />
          <FileRuleRef RuleID="ID_ALLOW_A_48E" />
          <FileRuleRef RuleID="ID_ALLOW_A_48F" />
          <FileRuleRef RuleID="ID_ALLOW_A_490" />
          <FileRuleRef RuleID="ID_ALLOW_A_491" />
          <FileRuleRef RuleID="ID_ALLOW_A_492" />
          <FileRuleRef RuleID="ID_ALLOW_A_493" />
          <FileRuleRef RuleID="ID_ALLOW_A_494" />
          <FileRuleRef RuleID="ID_ALLOW_A_495" />
          <FileRuleRef RuleID="ID_ALLOW_A_496" />
          <FileRuleRef RuleID="ID_ALLOW_A_497" />
          <FileRuleRef RuleID="ID_ALLOW_A_498" />
          <FileRuleRef RuleID="ID_ALLOW_A_499" />
          <FileRuleRef RuleID="ID_ALLOW_A_49A" />
          <FileRuleRef RuleID="ID_ALLOW_A_49B" />
          <FileRuleRef RuleID="ID_ALLOW_A_49C" />
          <FileRuleRef RuleID="ID_ALLOW_A_49D" />
          <FileRuleRef RuleID="ID_ALLOW_A_49E" />
          <FileRuleRef RuleID="ID_ALLOW_A_49F" />
          <FileRuleRef RuleID="ID_ALLOW_A_4A0" />
          <FileRuleRef RuleID="ID_ALLOW_A_4A1" />
          <FileRuleRef RuleID="ID_ALLOW_A_4A2" />
          <FileRuleRef RuleID="ID_ALLOW_A_4A3" />
          <FileRuleRef RuleID="ID_ALLOW_A_4A4" />
          <FileRuleRef RuleID="ID_ALLOW_A_4A5" />
          <FileRuleRef RuleID="ID_ALLOW_A_4A6" />
          <FileRuleRef RuleID="ID_ALLOW_A_4A7" />
          <FileRuleRef RuleID="ID_ALLOW_A_4A8" />
          <FileRuleRef RuleID="ID_ALLOW_A_4A9" />
          <FileRuleRef RuleID="ID_ALLOW_A_4AA" />
          <FileRuleRef RuleID="ID_ALLOW_A_4AB" />
          <FileRuleRef RuleID="ID_ALLOW_A_4AC" />
          <FileRuleRef RuleID="ID_ALLOW_A_4AD" />
          <FileRuleRef RuleID="ID_ALLOW_A_4AE" />
          <FileRuleRef RuleID="ID_ALLOW_A_4AF" />
          <FileRuleRef RuleID="ID_ALLOW_A_4B0" />
          <FileRuleRef RuleID="ID_ALLOW_A_4B1" />
          <FileRuleRef RuleID="ID_ALLOW_A_4B2" />
          <FileRuleRef RuleID="ID_ALLOW_A_4B3" />
          <FileRuleRef RuleID="ID_ALLOW_A_4B4" />
          <FileRuleRef RuleID="ID_ALLOW_A_4B5" />
          <FileRuleRef RuleID="ID_ALLOW_A_4B6" />
          <FileRuleRef RuleID="ID_ALLOW_A_4B7" />
          <FileRuleRef RuleID="ID_ALLOW_A_4B8" />
          <FileRuleRef RuleID="ID_ALLOW_A_4B9" />
          <FileRuleRef RuleID="ID_ALLOW_A_4BA" />
          <FileRuleRef RuleID="ID_ALLOW_A_4BB" />
          <FileRuleRef RuleID="ID_ALLOW_A_4BC" />
          <FileRuleRef RuleID="ID_ALLOW_A_4BD" />
          <FileRuleRef RuleID="ID_ALLOW_A_4BE" />
          <FileRuleRef RuleID="ID_ALLOW_A_4BF" />
          <FileRuleRef RuleID="ID_ALLOW_A_4C0" />
          <FileRuleRef RuleID="ID_ALLOW_A_4C1" />
          <FileRuleRef RuleID="ID_ALLOW_A_4C2" />
          <FileRuleRef RuleID="ID_ALLOW_A_4C3" />
          <FileRuleRef RuleID="ID_ALLOW_A_4C4" />
          <FileRuleRef RuleID="ID_ALLOW_A_4C5" />
          <FileRuleRef RuleID="ID_ALLOW_A_4C6" />
          <FileRuleRef RuleID="ID_ALLOW_A_4C7" />
          <FileRuleRef RuleID="ID_ALLOW_A_4C8" />
          <FileRuleRef RuleID="ID_ALLOW_A_4C9" />
          <FileRuleRef RuleID="ID_ALLOW_A_4CA" />
          <FileRuleRef RuleID="ID_ALLOW_A_4CB" />
          <FileRuleRef RuleID="ID_ALLOW_A_4CC" />
          <FileRuleRef RuleID="ID_ALLOW_A_4CD" />
          <FileRuleRef RuleID="ID_ALLOW_A_4CE" />
          <FileRuleRef RuleID="ID_ALLOW_A_4CF" />
          <FileRuleRef RuleID="ID_ALLOW_A_4D0" />
          <FileRuleRef RuleID="ID_ALLOW_A_4D1" />
          <FileRuleRef RuleID="ID_ALLOW_A_4D2" />
          <FileRuleRef RuleID="ID_ALLOW_A_4D3" />
          <FileRuleRef RuleID="ID_ALLOW_A_4D4" />
          <FileRuleRef RuleID="ID_ALLOW_A_4D5" />
          <FileRuleRef RuleID="ID_ALLOW_A_4D6" />
          <FileRuleRef RuleID="ID_ALLOW_A_4D7" />
          <FileRuleRef RuleID="ID_ALLOW_A_4D8" />
          <FileRuleRef RuleID="ID_ALLOW_A_4D9" />
          <FileRuleRef RuleID="ID_ALLOW_A_4DA" />
          <FileRuleRef RuleID="ID_ALLOW_A_4DB" />
          <FileRuleRef RuleID="ID_ALLOW_A_4DC" />
          <FileRuleRef RuleID="ID_ALLOW_A_4DD" />
          <FileRuleRef RuleID="ID_ALLOW_A_4DE" />
          <FileRuleRef RuleID="ID_ALLOW_A_4DF" />
          <FileRuleRef RuleID="ID_ALLOW_A_4E0" />
          <FileRuleRef RuleID="ID_ALLOW_A_4E1" />
          <FileRuleRef RuleID="ID_ALLOW_A_4E2" />
          <FileRuleRef RuleID="ID_ALLOW_A_4E3" />
          <FileRuleRef RuleID="ID_ALLOW_A_4E4" />
          <FileRuleRef RuleID="ID_ALLOW_A_4E5" />
          <FileRuleRef RuleID="ID_ALLOW_A_4E6" />
          <FileRuleRef RuleID="ID_ALLOW_A_4E7" />
          <FileRuleRef RuleID="ID_ALLOW_A_4E8" />
          <FileRuleRef RuleID="ID_ALLOW_A_4E9" />
          <FileRuleRef RuleID="ID_ALLOW_A_4EA" />
          <FileRuleRef RuleID="ID_ALLOW_A_4EB" />
          <FileRuleRef RuleID="ID_ALLOW_A_4EC" />
          <FileRuleRef RuleID="ID_ALLOW_A_4ED" />
          <FileRuleRef RuleID="ID_ALLOW_A_4EE" />
          <FileRuleRef RuleID="ID_ALLOW_A_4EF" />
          <FileRuleRef RuleID="ID_ALLOW_A_4F0" />
          <FileRuleRef RuleID="ID_ALLOW_A_4F1" />
          <FileRuleRef RuleID="ID_ALLOW_A_4F2" />
          <FileRuleRef RuleID="ID_ALLOW_A_4F3" />
          <FileRuleRef RuleID="ID_ALLOW_A_4F4" />
          <FileRuleRef RuleID="ID_ALLOW_A_4F5" />
          <FileRuleRef RuleID="ID_ALLOW_A_4F6" />
          <FileRuleRef RuleID="ID_ALLOW_A_4F7" />
          <FileRuleRef RuleID="ID_ALLOW_A_4F8" />
          <FileRuleRef RuleID="ID_ALLOW_A_4F9" />
          <FileRuleRef RuleID="ID_ALLOW_A_4FA" />
          <FileRuleRef RuleID="ID_ALLOW_A_4FB" />
          <FileRuleRef RuleID="ID_ALLOW_A_4FC" />
          <FileRuleRef RuleID="ID_ALLOW_A_4FD" />
          <FileRuleRef RuleID="ID_ALLOW_A_4FE" />
          <FileRuleRef RuleID="ID_ALLOW_A_4FF" />
          <FileRuleRef RuleID="ID_ALLOW_A_500" />
          <FileRuleRef RuleID="ID_ALLOW_A_501" />
          <FileRuleRef RuleID="ID_ALLOW_A_502" />
          <FileRuleRef RuleID="ID_ALLOW_A_503" />
          <FileRuleRef RuleID="ID_ALLOW_A_504" />
          <FileRuleRef RuleID="ID_ALLOW_A_505" />
          <FileRuleRef RuleID="ID_ALLOW_A_506" />
          <FileRuleRef RuleID="ID_ALLOW_A_507" />
          <FileRuleRef RuleID="ID_ALLOW_A_508" />
          <FileRuleRef RuleID="ID_ALLOW_A_509" />
          <FileRuleRef RuleID="ID_ALLOW_A_50A" />
          <FileRuleRef RuleID="ID_ALLOW_A_50B" />
          <FileRuleRef RuleID="ID_ALLOW_A_50C" />
          <FileRuleRef RuleID="ID_ALLOW_A_50D" />
          <FileRuleRef RuleID="ID_ALLOW_A_50E" />
          <FileRuleRef RuleID="ID_ALLOW_A_50F" />
          <FileRuleRef RuleID="ID_ALLOW_A_510" />
          <FileRuleRef RuleID="ID_ALLOW_A_511" />
          <FileRuleRef RuleID="ID_ALLOW_A_512" />
          <FileRuleRef RuleID="ID_ALLOW_A_513" />
          <FileRuleRef RuleID="ID_ALLOW_A_514" />
          <FileRuleRef RuleID="ID_ALLOW_A_515" />
          <FileRuleRef RuleID="ID_ALLOW_A_516" />
          <FileRuleRef RuleID="ID_ALLOW_A_517" />
          <FileRuleRef RuleID="ID_ALLOW_A_518" />
          <FileRuleRef RuleID="ID_ALLOW_A_519" />
          <FileRuleRef RuleID="ID_ALLOW_A_51A" />
          <FileRuleRef RuleID="ID_ALLOW_A_51B" />
          <FileRuleRef RuleID="ID_ALLOW_A_51C" />
          <FileRuleRef RuleID="ID_ALLOW_A_51D" />
          <FileRuleRef RuleID="ID_ALLOW_A_51E" />
          <FileRuleRef RuleID="ID_ALLOW_A_51F" />
          <FileRuleRef RuleID="ID_ALLOW_A_520" />
          <FileRuleRef RuleID="ID_ALLOW_A_521" />
          <FileRuleRef RuleID="ID_ALLOW_A_522" />
          <FileRuleRef RuleID="ID_ALLOW_A_523" />
          <FileRuleRef RuleID="ID_ALLOW_A_524" />
          <FileRuleRef RuleID="ID_ALLOW_A_525" />
          <FileRuleRef RuleID="ID_ALLOW_A_526" />
          <FileRuleRef RuleID="ID_ALLOW_A_527" />
          <FileRuleRef RuleID="ID_ALLOW_A_528" />
          <FileRuleRef RuleID="ID_ALLOW_A_529" />
          <FileRuleRef RuleID="ID_ALLOW_A_52A" />
          <FileRuleRef RuleID="ID_ALLOW_A_52B" />
          <FileRuleRef RuleID="ID_ALLOW_A_52C" />
          <FileRuleRef RuleID="ID_ALLOW_A_52D" />
          <FileRuleRef RuleID="ID_ALLOW_A_52E" />
          <FileRuleRef RuleID="ID_ALLOW_A_52F" />
          <FileRuleRef RuleID="ID_ALLOW_A_530" />
          <FileRuleRef RuleID="ID_ALLOW_A_531" />
          <FileRuleRef RuleID="ID_ALLOW_A_532" />
          <FileRuleRef RuleID="ID_ALLOW_A_533" />
          <FileRuleRef RuleID="ID_ALLOW_A_534" />
          <FileRuleRef RuleID="ID_ALLOW_A_535" />
          <FileRuleRef RuleID="ID_ALLOW_A_536" />
          <FileRuleRef RuleID="ID_ALLOW_A_537" />
          <FileRuleRef RuleID="ID_ALLOW_A_538" />
          <FileRuleRef RuleID="ID_ALLOW_A_539" />
          <FileRuleRef RuleID="ID_ALLOW_A_53A" />
          <FileRuleRef RuleID="ID_ALLOW_A_53B" />
          <FileRuleRef RuleID="ID_ALLOW_A_53C" />
          <FileRuleRef RuleID="ID_ALLOW_A_53D" />
          <FileRuleRef RuleID="ID_ALLOW_A_53E" />
          <FileRuleRef RuleID="ID_ALLOW_A_53F" />
          <FileRuleRef RuleID="ID_ALLOW_A_540" />
          <FileRuleRef RuleID="ID_ALLOW_A_541" />
          <FileRuleRef RuleID="ID_ALLOW_A_542" />
          <FileRuleRef RuleID="ID_ALLOW_A_543" />
          <FileRuleRef RuleID="ID_ALLOW_A_544" />
          <FileRuleRef RuleID="ID_ALLOW_A_545" />
          <FileRuleRef RuleID="ID_ALLOW_A_546" />
          <FileRuleRef RuleID="ID_ALLOW_A_547" />
          <FileRuleRef RuleID="ID_ALLOW_A_548" />
          <FileRuleRef RuleID="ID_ALLOW_A_549" />
          <FileRuleRef RuleID="ID_ALLOW_A_54A" />
          <FileRuleRef RuleID="ID_ALLOW_A_54B" />
          <FileRuleRef RuleID="ID_ALLOW_A_54C" />
          <FileRuleRef RuleID="ID_ALLOW_A_54D" />
          <FileRuleRef RuleID="ID_ALLOW_A_54E" />
          <FileRuleRef RuleID="ID_ALLOW_A_54F" />
          <FileRuleRef RuleID="ID_ALLOW_A_550" />
          <FileRuleRef RuleID="ID_ALLOW_A_551" />
          <FileRuleRef RuleID="ID_ALLOW_A_552" />
          <FileRuleRef RuleID="ID_ALLOW_A_553" />
          <FileRuleRef RuleID="ID_ALLOW_A_554" />
          <FileRuleRef RuleID="ID_ALLOW_A_555" />
          <FileRuleRef RuleID="ID_ALLOW_A_556" />
          <FileRuleRef RuleID="ID_ALLOW_A_557" />
          <FileRuleRef RuleID="ID_ALLOW_A_558" />
          <FileRuleRef RuleID="ID_ALLOW_A_559" />
          <FileRuleRef RuleID="ID_ALLOW_A_55A" />
          <FileRuleRef RuleID="ID_ALLOW_A_55B" />
          <FileRuleRef RuleID="ID_ALLOW_A_55C" />
          <FileRuleRef RuleID="ID_ALLOW_A_55D" />
          <FileRuleRef RuleID="ID_ALLOW_A_55E" />
          <FileRuleRef RuleID="ID_ALLOW_A_55F" />
          <FileRuleRef RuleID="ID_ALLOW_A_560" />
          <FileRuleRef RuleID="ID_ALLOW_A_561" />
          <FileRuleRef RuleID="ID_ALLOW_A_562" />
          <FileRuleRef RuleID="ID_ALLOW_A_563" />
          <FileRuleRef RuleID="ID_ALLOW_A_564" />
          <FileRuleRef RuleID="ID_ALLOW_A_565" />
          <FileRuleRef RuleID="ID_ALLOW_A_566" />
          <FileRuleRef RuleID="ID_ALLOW_A_567" />
          <FileRuleRef RuleID="ID_ALLOW_A_568" />
          <FileRuleRef RuleID="ID_ALLOW_A_569" />
          <FileRuleRef RuleID="ID_ALLOW_A_56A" />
          <FileRuleRef RuleID="ID_ALLOW_A_56B" />
          <FileRuleRef RuleID="ID_ALLOW_A_56C" />
          <FileRuleRef RuleID="ID_ALLOW_A_56D" />
          <FileRuleRef RuleID="ID_ALLOW_A_56E" />
          <FileRuleRef RuleID="ID_ALLOW_A_56F" />
          <FileRuleRef RuleID="ID_ALLOW_A_570" />
          <FileRuleRef RuleID="ID_ALLOW_A_571" />
          <FileRuleRef RuleID="ID_ALLOW_A_572" />
          <FileRuleRef RuleID="ID_ALLOW_A_573" />
          <FileRuleRef RuleID="ID_ALLOW_A_574" />
          <FileRuleRef RuleID="ID_ALLOW_A_575" />
          <FileRuleRef RuleID="ID_ALLOW_A_576" />
          <FileRuleRef RuleID="ID_ALLOW_A_577" />
          <FileRuleRef RuleID="ID_ALLOW_A_578" />
          <FileRuleRef RuleID="ID_ALLOW_A_579" />
          <FileRuleRef RuleID="ID_ALLOW_A_57A" />
          <FileRuleRef RuleID="ID_ALLOW_A_57B" />
          <FileRuleRef RuleID="ID_ALLOW_A_57C" />
          <FileRuleRef RuleID="ID_ALLOW_A_57D" />
          <FileRuleRef RuleID="ID_ALLOW_A_57E" />
          <FileRuleRef RuleID="ID_ALLOW_A_57F" />
          <FileRuleRef RuleID="ID_ALLOW_A_580" />
          <FileRuleRef RuleID="ID_ALLOW_A_581" />
          <FileRuleRef RuleID="ID_ALLOW_A_582" />
          <FileRuleRef RuleID="ID_ALLOW_A_583" />
          <FileRuleRef RuleID="ID_ALLOW_A_584" />
          <FileRuleRef RuleID="ID_ALLOW_A_585" />
          <FileRuleRef RuleID="ID_ALLOW_A_586" />
          <FileRuleRef RuleID="ID_ALLOW_A_587" />
          <FileRuleRef RuleID="ID_ALLOW_A_588" />
          <FileRuleRef RuleID="ID_ALLOW_A_589" />
          <FileRuleRef RuleID="ID_ALLOW_A_58A" />
          <FileRuleRef RuleID="ID_ALLOW_A_58B" />
          <FileRuleRef RuleID="ID_ALLOW_A_58C" />
          <FileRuleRef RuleID="ID_ALLOW_A_58D" />
          <FileRuleRef RuleID="ID_ALLOW_A_58E" />
          <FileRuleRef RuleID="ID_ALLOW_A_58F" />
          <FileRuleRef RuleID="ID_ALLOW_A_590" />
          <FileRuleRef RuleID="ID_ALLOW_A_591" />
          <FileRuleRef RuleID="ID_ALLOW_A_592" />
          <FileRuleRef RuleID="ID_ALLOW_A_593" />
          <FileRuleRef RuleID="ID_ALLOW_A_594" />
          <FileRuleRef RuleID="ID_ALLOW_A_595" />
          <FileRuleRef RuleID="ID_ALLOW_A_596" />
          <FileRuleRef RuleID="ID_ALLOW_A_597" />
          <FileRuleRef RuleID="ID_ALLOW_A_598" />
          <FileRuleRef RuleID="ID_ALLOW_A_599" />
          <FileRuleRef RuleID="ID_ALLOW_A_59A" />
          <FileRuleRef RuleID="ID_ALLOW_A_59B" />
          <FileRuleRef RuleID="ID_ALLOW_A_59C" />
          <FileRuleRef RuleID="ID_ALLOW_A_59D" />
          <FileRuleRef RuleID="ID_ALLOW_A_59E" />
          <FileRuleRef RuleID="ID_ALLOW_A_59F" />
          <FileRuleRef RuleID="ID_ALLOW_A_5A0" />
          <FileRuleRef RuleID="ID_ALLOW_A_5A1" />
          <FileRuleRef RuleID="ID_ALLOW_A_5A2" />
          <FileRuleRef RuleID="ID_ALLOW_A_5A3" />
          <FileRuleRef RuleID="ID_ALLOW_A_5A4" />
          <FileRuleRef RuleID="ID_ALLOW_A_5A5" />
          <FileRuleRef RuleID="ID_ALLOW_A_5A6" />
          <FileRuleRef RuleID="ID_ALLOW_A_5A7" />
          <FileRuleRef RuleID="ID_ALLOW_A_5A8" />
          <FileRuleRef RuleID="ID_ALLOW_A_5A9" />
          <FileRuleRef RuleID="ID_ALLOW_A_5AA" />
          <FileRuleRef RuleID="ID_ALLOW_A_5AB" />
          <FileRuleRef RuleID="ID_ALLOW_A_5AC" />
          <FileRuleRef RuleID="ID_ALLOW_A_5AD" />
          <FileRuleRef RuleID="ID_ALLOW_A_5AE" />
          <FileRuleRef RuleID="ID_ALLOW_A_5AF" />
          <FileRuleRef RuleID="ID_ALLOW_A_5B0" />
          <FileRuleRef RuleID="ID_ALLOW_A_5B1" />
          <FileRuleRef RuleID="ID_ALLOW_A_5B2" />
          <FileRuleRef RuleID="ID_ALLOW_A_5B3" />
          <FileRuleRef RuleID="ID_ALLOW_A_5B4" />
          <FileRuleRef RuleID="ID_ALLOW_A_5B5" />
          <FileRuleRef RuleID="ID_ALLOW_A_5B6" />
          <FileRuleRef RuleID="ID_ALLOW_A_5B7" />
          <FileRuleRef RuleID="ID_ALLOW_A_5B8" />
          <FileRuleRef RuleID="ID_ALLOW_A_5B9" />
          <FileRuleRef RuleID="ID_ALLOW_A_5BA" />
          <FileRuleRef RuleID="ID_ALLOW_A_5BB" />
          <FileRuleRef RuleID="ID_ALLOW_A_5BC" />
          <FileRuleRef RuleID="ID_ALLOW_A_5BD" />
          <FileRuleRef RuleID="ID_ALLOW_A_5BE" />
          <FileRuleRef RuleID="ID_ALLOW_A_5BF" />
          <FileRuleRef RuleID="ID_ALLOW_A_5C0" />
          <FileRuleRef RuleID="ID_ALLOW_A_5C1" />
          <FileRuleRef RuleID="ID_ALLOW_A_5C2" />
          <FileRuleRef RuleID="ID_ALLOW_A_5C3" />
          <FileRuleRef RuleID="ID_ALLOW_A_5C4" />
          <FileRuleRef RuleID="ID_ALLOW_A_5C5" />
          <FileRuleRef RuleID="ID_ALLOW_A_5C6" />
          <FileRuleRef RuleID="ID_ALLOW_A_5C7" />
          <FileRuleRef RuleID="ID_ALLOW_A_5C8" />
          <FileRuleRef RuleID="ID_ALLOW_A_5C9" />
          <FileRuleRef RuleID="ID_ALLOW_A_5CA" />
          <FileRuleRef RuleID="ID_ALLOW_A_5CB" />
          <FileRuleRef RuleID="ID_ALLOW_A_5CC" />
          <FileRuleRef RuleID="ID_ALLOW_A_5CD" />
          <FileRuleRef RuleID="ID_ALLOW_A_5CE" />
          <FileRuleRef RuleID="ID_ALLOW_A_5CF" />
          <FileRuleRef RuleID="ID_ALLOW_A_5D0" />
          <FileRuleRef RuleID="ID_ALLOW_A_5D1" />
          <FileRuleRef RuleID="ID_ALLOW_A_5D2" />
          <FileRuleRef RuleID="ID_ALLOW_A_5D3" />
          <FileRuleRef RuleID="ID_ALLOW_A_5D4" />
          <FileRuleRef RuleID="ID_ALLOW_A_5D5" />
          <FileRuleRef RuleID="ID_ALLOW_A_5D6" />
          <FileRuleRef RuleID="ID_ALLOW_A_5D7" />
          <FileRuleRef RuleID="ID_ALLOW_A_5D8" />
          <FileRuleRef RuleID="ID_ALLOW_A_5D9" />
          <FileRuleRef RuleID="ID_ALLOW_A_5DA" />
          <FileRuleRef RuleID="ID_ALLOW_A_5DB" />
          <FileRuleRef RuleID="ID_ALLOW_A_5DC" />
          <FileRuleRef RuleID="ID_ALLOW_A_5DD" />
          <FileRuleRef RuleID="ID_ALLOW_A_5DE" />
          <FileRuleRef RuleID="ID_ALLOW_A_5DF" />
          <FileRuleRef RuleID="ID_ALLOW_A_5E0" />
          <FileRuleRef RuleID="ID_ALLOW_A_5E1" />
          <FileRuleRef RuleID="ID_ALLOW_A_5E2" />
          <FileRuleRef RuleID="ID_ALLOW_A_5E3" />
          <FileRuleRef RuleID="ID_ALLOW_A_5E4" />
          <FileRuleRef RuleID="ID_ALLOW_A_5E5" />
          <FileRuleRef RuleID="ID_ALLOW_A_5E6" />
          <FileRuleRef RuleID="ID_ALLOW_A_5E7" />
          <FileRuleRef RuleID="ID_ALLOW_A_5E8" />
          <FileRuleRef RuleID="ID_ALLOW_A_5E9" />
          <FileRuleRef RuleID="ID_ALLOW_A_5EA" />
          <FileRuleRef RuleID="ID_ALLOW_A_5EB" />
          <FileRuleRef RuleID="ID_ALLOW_A_5EC" />
          <FileRuleRef RuleID="ID_ALLOW_A_5ED" />
          <FileRuleRef RuleID="ID_ALLOW_A_5EE" />
          <FileRuleRef RuleID="ID_ALLOW_A_5EF" />
          <FileRuleRef RuleID="ID_ALLOW_A_5F0" />
          <FileRuleRef RuleID="ID_ALLOW_A_5F1" />
          <FileRuleRef RuleID="ID_ALLOW_A_5F2" />
          <FileRuleRef RuleID="ID_ALLOW_A_5F3" />
          <FileRuleRef RuleID="ID_ALLOW_A_5F4" />
          <FileRuleRef RuleID="ID_ALLOW_A_5F5" />
          <FileRuleRef RuleID="ID_ALLOW_A_5F6" />
          <FileRuleRef RuleID="ID_ALLOW_A_5F7" />
          <FileRuleRef RuleID="ID_ALLOW_A_5F8" />
          <FileRuleRef RuleID="ID_ALLOW_A_5F9" />
          <FileRuleRef RuleID="ID_ALLOW_A_5FA" />
          <FileRuleRef RuleID="ID_ALLOW_A_5FB" />
          <FileRuleRef RuleID="ID_ALLOW_A_5FC" />
          <FileRuleRef RuleID="ID_ALLOW_A_5FD" />
          <FileRuleRef RuleID="ID_ALLOW_A_5FE" />
          <FileRuleRef RuleID="ID_ALLOW_A_5FF" />
          <FileRuleRef RuleID="ID_ALLOW_A_600" />
          <FileRuleRef RuleID="ID_ALLOW_A_601" />
          <FileRuleRef RuleID="ID_ALLOW_A_602" />
          <FileRuleRef RuleID="ID_ALLOW_A_603" />
          <FileRuleRef RuleID="ID_ALLOW_A_604" />
          <FileRuleRef RuleID="ID_ALLOW_A_605" />
          <FileRuleRef RuleID="ID_ALLOW_A_606" />
          <FileRuleRef RuleID="ID_ALLOW_A_607" />
          <FileRuleRef RuleID="ID_ALLOW_A_608" />
          <FileRuleRef RuleID="ID_ALLOW_A_609" />
          <FileRuleRef RuleID="ID_ALLOW_A_60A" />
          <FileRuleRef RuleID="ID_ALLOW_A_60B" />
          <FileRuleRef RuleID="ID_ALLOW_A_60C" />
          <FileRuleRef RuleID="ID_ALLOW_A_60D" />
          <FileRuleRef RuleID="ID_ALLOW_A_60E" />
          <FileRuleRef RuleID="ID_ALLOW_A_60F" />
          <FileRuleRef RuleID="ID_ALLOW_A_610" />
          <FileRuleRef RuleID="ID_ALLOW_A_611" />
          <FileRuleRef RuleID="ID_ALLOW_A_612" />
          <FileRuleRef RuleID="ID_ALLOW_A_613" />
          <FileRuleRef RuleID="ID_ALLOW_A_614" />
          <FileRuleRef RuleID="ID_ALLOW_A_615" />
          <FileRuleRef RuleID="ID_ALLOW_A_616" />
          <FileRuleRef RuleID="ID_ALLOW_A_617" />
          <FileRuleRef RuleID="ID_ALLOW_A_618" />
          <FileRuleRef RuleID="ID_ALLOW_A_619" />
          <FileRuleRef RuleID="ID_ALLOW_A_61A" />
          <FileRuleRef RuleID="ID_ALLOW_A_61B" />
          <FileRuleRef RuleID="ID_ALLOW_A_61C" />
          <FileRuleRef RuleID="ID_ALLOW_A_61D" />
          <FileRuleRef RuleID="ID_ALLOW_A_61E" />
          <FileRuleRef RuleID="ID_ALLOW_A_61F" />
          <FileRuleRef RuleID="ID_ALLOW_A_620" />
          <FileRuleRef RuleID="ID_ALLOW_A_621" />
          <FileRuleRef RuleID="ID_ALLOW_A_622" />
          <FileRuleRef RuleID="ID_ALLOW_A_623" />
          <FileRuleRef RuleID="ID_ALLOW_A_624" />
          <FileRuleRef RuleID="ID_ALLOW_A_625" />
          <FileRuleRef RuleID="ID_ALLOW_A_626" />
          <FileRuleRef RuleID="ID_ALLOW_A_627" />
          <FileRuleRef RuleID="ID_ALLOW_A_628" />
          <FileRuleRef RuleID="ID_ALLOW_A_629" />
          <FileRuleRef RuleID="ID_ALLOW_A_62A" />
          <FileRuleRef RuleID="ID_ALLOW_A_62B" />
          <FileRuleRef RuleID="ID_ALLOW_A_62C" />
          <FileRuleRef RuleID="ID_ALLOW_A_62D" />
          <FileRuleRef RuleID="ID_ALLOW_A_62E" />
          <FileRuleRef RuleID="ID_ALLOW_A_62F" />
          <FileRuleRef RuleID="ID_ALLOW_A_630" />
          <FileRuleRef RuleID="ID_ALLOW_A_631" />
          <FileRuleRef RuleID="ID_ALLOW_A_632" />
          <FileRuleRef RuleID="ID_ALLOW_A_633" />
          <FileRuleRef RuleID="ID_ALLOW_A_634" />
          <FileRuleRef RuleID="ID_ALLOW_A_635" />
          <FileRuleRef RuleID="ID_ALLOW_A_636" />
          <FileRuleRef RuleID="ID_ALLOW_A_637" />
          <FileRuleRef RuleID="ID_ALLOW_A_638" />
          <FileRuleRef RuleID="ID_ALLOW_A_639" />
          <FileRuleRef RuleID="ID_ALLOW_A_63A" />
          <FileRuleRef RuleID="ID_ALLOW_A_63B" />
          <FileRuleRef RuleID="ID_ALLOW_A_63C" />
          <FileRuleRef RuleID="ID_ALLOW_A_63D" />
          <FileRuleRef RuleID="ID_ALLOW_A_63E" />
          <FileRuleRef RuleID="ID_ALLOW_A_63F" />
          <FileRuleRef RuleID="ID_ALLOW_A_640" />
          <FileRuleRef RuleID="ID_ALLOW_A_641" />
          <FileRuleRef RuleID="ID_ALLOW_A_642" />
          <FileRuleRef RuleID="ID_ALLOW_A_643" />
          <FileRuleRef RuleID="ID_ALLOW_A_644" />
          <FileRuleRef RuleID="ID_ALLOW_A_645" />
          <FileRuleRef RuleID="ID_ALLOW_A_646" />
          <FileRuleRef RuleID="ID_ALLOW_A_647" />
          <FileRuleRef RuleID="ID_ALLOW_A_648" />
          <FileRuleRef RuleID="ID_ALLOW_A_649" />
          <FileRuleRef RuleID="ID_ALLOW_A_64A" />
          <FileRuleRef RuleID="ID_ALLOW_A_64B" />
          <FileRuleRef RuleID="ID_ALLOW_A_64C" />
          <FileRuleRef RuleID="ID_ALLOW_A_64D" />
          <FileRuleRef RuleID="ID_ALLOW_A_64E" />
          <FileRuleRef RuleID="ID_ALLOW_A_64F" />
          <FileRuleRef RuleID="ID_ALLOW_A_650" />
          <FileRuleRef RuleID="ID_ALLOW_A_651" />
          <FileRuleRef RuleID="ID_ALLOW_A_652" />
          <FileRuleRef RuleID="ID_ALLOW_A_653" />
          <FileRuleRef RuleID="ID_ALLOW_A_654" />
          <FileRuleRef RuleID="ID_ALLOW_A_655" />
          <FileRuleRef RuleID="ID_ALLOW_A_656" />
          <FileRuleRef RuleID="ID_ALLOW_A_657" />
          <FileRuleRef RuleID="ID_ALLOW_A_658" />
          <FileRuleRef RuleID="ID_ALLOW_A_659" />
          <FileRuleRef RuleID="ID_ALLOW_A_65A" />
          <FileRuleRef RuleID="ID_ALLOW_A_65B" />
          <FileRuleRef RuleID="ID_ALLOW_A_65C" />
          <FileRuleRef RuleID="ID_ALLOW_A_65D" />
          <FileRuleRef RuleID="ID_ALLOW_A_65E" />
          <FileRuleRef RuleID="ID_ALLOW_A_65F" />
          <FileRuleRef RuleID="ID_ALLOW_A_660" />
          <FileRuleRef RuleID="ID_ALLOW_A_661" />
          <FileRuleRef RuleID="ID_ALLOW_A_662" />
          <FileRuleRef RuleID="ID_ALLOW_A_663" />
          <FileRuleRef RuleID="ID_ALLOW_A_664" />
          <FileRuleRef RuleID="ID_ALLOW_A_665" />
          <FileRuleRef RuleID="ID_ALLOW_A_666" />
          <FileRuleRef RuleID="ID_ALLOW_A_667" />
          <FileRuleRef RuleID="ID_ALLOW_A_668" />
          <FileRuleRef RuleID="ID_ALLOW_A_669" />
          <FileRuleRef RuleID="ID_ALLOW_A_66A" />
          <FileRuleRef RuleID="ID_ALLOW_A_66B" />
          <FileRuleRef RuleID="ID_ALLOW_A_66C" />
          <FileRuleRef RuleID="ID_ALLOW_A_66D" />
          <FileRuleRef RuleID="ID_ALLOW_A_66E" />
          <FileRuleRef RuleID="ID_ALLOW_A_66F" />
          <FileRuleRef RuleID="ID_ALLOW_A_670" />
          <FileRuleRef RuleID="ID_ALLOW_A_671" />
          <FileRuleRef RuleID="ID_ALLOW_A_672" />
          <FileRuleRef RuleID="ID_ALLOW_A_673" />
          <FileRuleRef RuleID="ID_ALLOW_A_674" />
          <FileRuleRef RuleID="ID_ALLOW_A_675" />
          <FileRuleRef RuleID="ID_ALLOW_A_676" />
          <FileRuleRef RuleID="ID_ALLOW_A_677" />
          <FileRuleRef RuleID="ID_ALLOW_A_678" />
          <FileRuleRef RuleID="ID_ALLOW_A_679" />
          <FileRuleRef RuleID="ID_ALLOW_A_67A" />
          <FileRuleRef RuleID="ID_ALLOW_A_67B" />
          <FileRuleRef RuleID="ID_ALLOW_A_67C" />
          <FileRuleRef RuleID="ID_ALLOW_A_67D" />
          <FileRuleRef RuleID="ID_ALLOW_A_67E" />
          <FileRuleRef RuleID="ID_ALLOW_A_67F" />
          <FileRuleRef RuleID="ID_ALLOW_A_680" />
          <FileRuleRef RuleID="ID_ALLOW_A_681" />
          <FileRuleRef RuleID="ID_ALLOW_A_682" />
          <FileRuleRef RuleID="ID_ALLOW_A_683" />
          <FileRuleRef RuleID="ID_ALLOW_A_684" />
          <FileRuleRef RuleID="ID_ALLOW_A_685" />
          <FileRuleRef RuleID="ID_ALLOW_A_686" />
          <FileRuleRef RuleID="ID_ALLOW_A_687" />
          <FileRuleRef RuleID="ID_ALLOW_A_688" />
          <FileRuleRef RuleID="ID_ALLOW_A_689" />
          <FileRuleRef RuleID="ID_ALLOW_A_68A" />
          <FileRuleRef RuleID="ID_ALLOW_A_68B" />
          <FileRuleRef RuleID="ID_ALLOW_A_68C" />
          <FileRuleRef RuleID="ID_ALLOW_A_68D" />
          <FileRuleRef RuleID="ID_ALLOW_A_68E" />
          <FileRuleRef RuleID="ID_ALLOW_A_68F" />
          <FileRuleRef RuleID="ID_ALLOW_A_690" />
          <FileRuleRef RuleID="ID_ALLOW_A_691" />
          <FileRuleRef RuleID="ID_ALLOW_A_692" />
          <FileRuleRef RuleID="ID_ALLOW_A_693" />
          <FileRuleRef RuleID="ID_ALLOW_A_694" />
          <FileRuleRef RuleID="ID_ALLOW_A_695" />
          <FileRuleRef RuleID="ID_ALLOW_A_696" />
          <FileRuleRef RuleID="ID_ALLOW_A_697" />
          <FileRuleRef RuleID="ID_ALLOW_A_698" />
          <FileRuleRef RuleID="ID_ALLOW_A_699" />
          <FileRuleRef RuleID="ID_ALLOW_A_69A" />
          <FileRuleRef RuleID="ID_ALLOW_A_69B" />
          <FileRuleRef RuleID="ID_ALLOW_A_69C" />
          <FileRuleRef RuleID="ID_ALLOW_A_69D" />
          <FileRuleRef RuleID="ID_ALLOW_A_69E" />
          <FileRuleRef RuleID="ID_ALLOW_A_69F" />
          <FileRuleRef RuleID="ID_ALLOW_A_6A0" />
          <FileRuleRef RuleID="ID_ALLOW_A_6A1" />
          <FileRuleRef RuleID="ID_ALLOW_A_6A2" />
          <FileRuleRef RuleID="ID_ALLOW_A_6A3" />
          <FileRuleRef RuleID="ID_ALLOW_A_6A4" />
          <FileRuleRef RuleID="ID_ALLOW_A_6A5" />
          <FileRuleRef RuleID="ID_ALLOW_A_6A6" />
          <FileRuleRef RuleID="ID_ALLOW_A_6A7" />
          <FileRuleRef RuleID="ID_ALLOW_A_6A8" />
          <FileRuleRef RuleID="ID_ALLOW_A_6A9" />
          <FileRuleRef RuleID="ID_ALLOW_A_6AA" />
          <FileRuleRef RuleID="ID_ALLOW_A_6AB" />
          <FileRuleRef RuleID="ID_ALLOW_A_6AC" />
          <FileRuleRef RuleID="ID_ALLOW_A_6AD" />
          <FileRuleRef RuleID="ID_ALLOW_A_6AE" />
          <FileRuleRef RuleID="ID_ALLOW_A_6AF" />
          <FileRuleRef RuleID="ID_ALLOW_A_6B0" />
          <FileRuleRef RuleID="ID_ALLOW_A_6B1" />
          <FileRuleRef RuleID="ID_ALLOW_A_6B2" />
          <FileRuleRef RuleID="ID_ALLOW_A_6B3" />
          <FileRuleRef RuleID="ID_ALLOW_A_6B4" />
          <FileRuleRef RuleID="ID_ALLOW_A_6B5" />
          <FileRuleRef RuleID="ID_ALLOW_A_6B6" />
          <FileRuleRef RuleID="ID_ALLOW_A_6B7" />
          <FileRuleRef RuleID="ID_ALLOW_A_6B8" />
          <FileRuleRef RuleID="ID_ALLOW_A_6B9" />
          <FileRuleRef RuleID="ID_ALLOW_A_6BA" />
          <FileRuleRef RuleID="ID_ALLOW_A_6BB" />
          <FileRuleRef RuleID="ID_ALLOW_A_6BC" />
          <FileRuleRef RuleID="ID_ALLOW_A_6BD" />
          <FileRuleRef RuleID="ID_ALLOW_A_6BE" />
          <FileRuleRef RuleID="ID_ALLOW_A_6BF" />
          <FileRuleRef RuleID="ID_ALLOW_A_6C0" />
          <FileRuleRef RuleID="ID_ALLOW_A_6C1" />
          <FileRuleRef RuleID="ID_ALLOW_A_6C2" />
          <FileRuleRef RuleID="ID_ALLOW_A_6C3" />
          <FileRuleRef RuleID="ID_ALLOW_A_6C4" />
          <FileRuleRef RuleID="ID_ALLOW_A_6C5" />
          <FileRuleRef RuleID="ID_ALLOW_A_6C6" />
          <FileRuleRef RuleID="ID_ALLOW_A_6C7" />
          <FileRuleRef RuleID="ID_ALLOW_A_6C8" />
          <FileRuleRef RuleID="ID_ALLOW_A_6C9" />
          <FileRuleRef RuleID="ID_ALLOW_A_6CA" />
          <FileRuleRef RuleID="ID_ALLOW_A_6CB" />
          <FileRuleRef RuleID="ID_ALLOW_A_6CC" />
          <FileRuleRef RuleID="ID_ALLOW_A_6CD" />
          <FileRuleRef RuleID="ID_ALLOW_A_6CE" />
          <FileRuleRef RuleID="ID_ALLOW_A_6CF" />
          <FileRuleRef RuleID="ID_ALLOW_A_6D0" />
          <FileRuleRef RuleID="ID_ALLOW_A_6D1" />
          <FileRuleRef RuleID="ID_ALLOW_A_6D2" />
          <FileRuleRef RuleID="ID_ALLOW_A_6D3" />
          <FileRuleRef RuleID="ID_ALLOW_A_6D4" />
          <FileRuleRef RuleID="ID_ALLOW_A_6D5" />
          <FileRuleRef RuleID="ID_ALLOW_A_6D6" />
          <FileRuleRef RuleID="ID_ALLOW_A_6D7" />
          <FileRuleRef RuleID="ID_ALLOW_A_6D8" />
          <FileRuleRef RuleID="ID_ALLOW_A_6D9" />
          <FileRuleRef RuleID="ID_ALLOW_A_6DA" />
          <FileRuleRef RuleID="ID_ALLOW_A_6DB" />
          <FileRuleRef RuleID="ID_ALLOW_A_6DC" />
          <FileRuleRef RuleID="ID_ALLOW_A_6DD" />
          <FileRuleRef RuleID="ID_ALLOW_A_6DE" />
          <FileRuleRef RuleID="ID_ALLOW_A_6DF" />
          <FileRuleRef RuleID="ID_ALLOW_A_6E0" />
          <FileRuleRef RuleID="ID_ALLOW_A_6E1" />
          <FileRuleRef RuleID="ID_ALLOW_A_6E2" />
          <FileRuleRef RuleID="ID_ALLOW_A_6E3" />
          <FileRuleRef RuleID="ID_ALLOW_A_6E4" />
          <FileRuleRef RuleID="ID_ALLOW_A_6E5" />
          <FileRuleRef RuleID="ID_ALLOW_A_6E6" />
          <FileRuleRef RuleID="ID_ALLOW_A_6E7" />
          <FileRuleRef RuleID="ID_ALLOW_A_6E8" />
          <FileRuleRef RuleID="ID_ALLOW_A_6E9" />
          <FileRuleRef RuleID="ID_ALLOW_A_6EA" />
          <FileRuleRef RuleID="ID_ALLOW_A_6EB" />
          <FileRuleRef RuleID="ID_ALLOW_A_6EC" />
          <FileRuleRef RuleID="ID_ALLOW_A_6ED" />
          <FileRuleRef RuleID="ID_ALLOW_A_6EE" />
          <FileRuleRef RuleID="ID_ALLOW_A_6EF" />
          <FileRuleRef RuleID="ID_ALLOW_A_6F0" />
          <FileRuleRef RuleID="ID_ALLOW_A_6F1" />
          <FileRuleRef RuleID="ID_ALLOW_A_6F2" />
          <FileRuleRef RuleID="ID_ALLOW_A_6F3" />
          <FileRuleRef RuleID="ID_ALLOW_A_6F4" />
          <FileRuleRef RuleID="ID_ALLOW_A_6F5" />
          <FileRuleRef RuleID="ID_ALLOW_A_6F6" />
          <FileRuleRef RuleID="ID_ALLOW_A_6F7" />
          <FileRuleRef RuleID="ID_ALLOW_A_6F8" />
          <FileRuleRef RuleID="ID_ALLOW_A_6F9" />
          <FileRuleRef RuleID="ID_ALLOW_A_6FA" />
          <FileRuleRef RuleID="ID_ALLOW_A_6FB" />
          <FileRuleRef RuleID="ID_ALLOW_A_6FC" />
          <FileRuleRef RuleID="ID_ALLOW_A_6FD" />
          <FileRuleRef RuleID="ID_ALLOW_A_6FE" />
          <FileRuleRef RuleID="ID_ALLOW_A_6FF" />
          <FileRuleRef RuleID="ID_ALLOW_A_700" />
          <FileRuleRef RuleID="ID_ALLOW_A_701" />
          <FileRuleRef RuleID="ID_ALLOW_A_702" />
          <FileRuleRef RuleID="ID_ALLOW_A_703" />
          <FileRuleRef RuleID="ID_ALLOW_A_704" />
          <FileRuleRef RuleID="ID_ALLOW_A_705" />
          <FileRuleRef RuleID="ID_ALLOW_A_706" />
          <FileRuleRef RuleID="ID_ALLOW_A_707" />
          <FileRuleRef RuleID="ID_ALLOW_A_708" />
          <FileRuleRef RuleID="ID_ALLOW_A_709" />
          <FileRuleRef RuleID="ID_ALLOW_A_70A" />
          <FileRuleRef RuleID="ID_ALLOW_A_70B" />
          <FileRuleRef RuleID="ID_ALLOW_A_70C" />
          <FileRuleRef RuleID="ID_ALLOW_A_70D" />
          <FileRuleRef RuleID="ID_ALLOW_A_70E" />
          <FileRuleRef RuleID="ID_ALLOW_A_70F" />
          <FileRuleRef RuleID="ID_ALLOW_A_710" />
          <FileRuleRef RuleID="ID_ALLOW_A_711" />
          <FileRuleRef RuleID="ID_ALLOW_A_712" />
          <FileRuleRef RuleID="ID_ALLOW_A_713" />
          <FileRuleRef RuleID="ID_ALLOW_A_714" />
          <FileRuleRef RuleID="ID_ALLOW_A_715" />
          <FileRuleRef RuleID="ID_ALLOW_A_716" />
          <FileRuleRef RuleID="ID_ALLOW_A_717" />
          <FileRuleRef RuleID="ID_ALLOW_A_718" />
          <FileRuleRef RuleID="ID_ALLOW_A_719" />
          <FileRuleRef RuleID="ID_ALLOW_A_71A" />
          <FileRuleRef RuleID="ID_ALLOW_A_71B" />
          <FileRuleRef RuleID="ID_ALLOW_A_71C" />
          <FileRuleRef RuleID="ID_ALLOW_A_71D" />
          <FileRuleRef RuleID="ID_ALLOW_A_71E" />
          <FileRuleRef RuleID="ID_ALLOW_A_71F" />
          <FileRuleRef RuleID="ID_ALLOW_A_720" />
          <FileRuleRef RuleID="ID_ALLOW_A_721" />
          <FileRuleRef RuleID="ID_ALLOW_A_722" />
          <FileRuleRef RuleID="ID_ALLOW_A_723" />
          <FileRuleRef RuleID="ID_ALLOW_A_724" />
          <FileRuleRef RuleID="ID_ALLOW_A_725" />
          <FileRuleRef RuleID="ID_ALLOW_A_726" />
          <FileRuleRef RuleID="ID_ALLOW_A_727" />
          <FileRuleRef RuleID="ID_ALLOW_A_728" />
          <FileRuleRef RuleID="ID_ALLOW_A_729" />
          <FileRuleRef RuleID="ID_ALLOW_A_72A" />
          <FileRuleRef RuleID="ID_ALLOW_A_72B" />
          <FileRuleRef RuleID="ID_ALLOW_A_72C" />
          <FileRuleRef RuleID="ID_ALLOW_A_72D" />
          <FileRuleRef RuleID="ID_ALLOW_A_72E" />
          <FileRuleRef RuleID="ID_ALLOW_A_72F" />
          <FileRuleRef RuleID="ID_ALLOW_A_730" />
          <FileRuleRef RuleID="ID_ALLOW_A_731" />
          <FileRuleRef RuleID="ID_ALLOW_A_732" />
          <FileRuleRef RuleID="ID_ALLOW_A_733" />
          <FileRuleRef RuleID="ID_ALLOW_A_734" />
          <FileRuleRef RuleID="ID_ALLOW_A_735" />
          <FileRuleRef RuleID="ID_ALLOW_A_736" />
          <FileRuleRef RuleID="ID_ALLOW_A_737" />
          <FileRuleRef RuleID="ID_ALLOW_A_738" />
          <FileRuleRef RuleID="ID_ALLOW_A_739" />
          <FileRuleRef RuleID="ID_ALLOW_A_73A" />
          <FileRuleRef RuleID="ID_ALLOW_A_73B" />
          <FileRuleRef RuleID="ID_ALLOW_A_73C" />
          <FileRuleRef RuleID="ID_ALLOW_A_73D" />
          <FileRuleRef RuleID="ID_ALLOW_A_73E" />
          <FileRuleRef RuleID="ID_ALLOW_A_73F" />
          <FileRuleRef RuleID="ID_ALLOW_A_740" />
          <FileRuleRef RuleID="ID_ALLOW_A_741" />
          <FileRuleRef RuleID="ID_ALLOW_A_742" />
          <FileRuleRef RuleID="ID_ALLOW_A_743" />
          <FileRuleRef RuleID="ID_ALLOW_A_744" />
          <FileRuleRef RuleID="ID_ALLOW_A_745" />
          <FileRuleRef RuleID="ID_ALLOW_A_746" />
          <FileRuleRef RuleID="ID_ALLOW_A_747" />
          <FileRuleRef RuleID="ID_ALLOW_A_748" />
          <FileRuleRef RuleID="ID_ALLOW_A_749" />
          <FileRuleRef RuleID="ID_ALLOW_A_74A" />
          <FileRuleRef RuleID="ID_ALLOW_A_74B" />
          <FileRuleRef RuleID="ID_ALLOW_A_74C" />
          <FileRuleRef RuleID="ID_ALLOW_A_74D" />
          <FileRuleRef RuleID="ID_ALLOW_A_74E" />
          <FileRuleRef RuleID="ID_ALLOW_A_74F" />
          <FileRuleRef RuleID="ID_ALLOW_A_750" />
          <FileRuleRef RuleID="ID_ALLOW_A_751" />
          <FileRuleRef RuleID="ID_ALLOW_A_752" />
          <FileRuleRef RuleID="ID_ALLOW_A_753" />
          <FileRuleRef RuleID="ID_ALLOW_A_754" />
          <FileRuleRef RuleID="ID_ALLOW_A_755" />
          <FileRuleRef RuleID="ID_ALLOW_A_756" />
          <FileRuleRef RuleID="ID_ALLOW_A_757" />
          <FileRuleRef RuleID="ID_ALLOW_A_758" />
          <FileRuleRef RuleID="ID_ALLOW_A_759" />
          <FileRuleRef RuleID="ID_ALLOW_A_75A" />
          <FileRuleRef RuleID="ID_ALLOW_A_75B" />
          <FileRuleRef RuleID="ID_ALLOW_A_75C" />
          <FileRuleRef RuleID="ID_ALLOW_A_75D" />
          <FileRuleRef RuleID="ID_ALLOW_A_75E" />
          <FileRuleRef RuleID="ID_ALLOW_A_75F" />
          <FileRuleRef RuleID="ID_ALLOW_A_760" />
          <FileRuleRef RuleID="ID_ALLOW_A_761" />
          <FileRuleRef RuleID="ID_ALLOW_A_762" />
          <FileRuleRef RuleID="ID_ALLOW_A_763" />
          <FileRuleRef RuleID="ID_ALLOW_A_764" />
          <FileRuleRef RuleID="ID_ALLOW_A_765" />
          <FileRuleRef RuleID="ID_ALLOW_A_766" />
          <FileRuleRef RuleID="ID_ALLOW_A_767" />
          <FileRuleRef RuleID="ID_ALLOW_A_768" />
          <FileRuleRef RuleID="ID_ALLOW_A_769" />
          <FileRuleRef RuleID="ID_ALLOW_A_76A" />
          <FileRuleRef RuleID="ID_ALLOW_A_76B" />
          <FileRuleRef RuleID="ID_ALLOW_A_76C" />
          <FileRuleRef RuleID="ID_ALLOW_A_76D" />
          <FileRuleRef RuleID="ID_ALLOW_A_76E" />
          <FileRuleRef RuleID="ID_ALLOW_A_76F" />
          <FileRuleRef RuleID="ID_ALLOW_A_770" />
          <FileRuleRef RuleID="ID_ALLOW_A_771" />
          <FileRuleRef RuleID="ID_ALLOW_A_772" />
          <FileRuleRef RuleID="ID_ALLOW_A_773" />
          <FileRuleRef RuleID="ID_ALLOW_A_774" />
          <FileRuleRef RuleID="ID_ALLOW_A_775" />
          <FileRuleRef RuleID="ID_ALLOW_A_776" />
          <FileRuleRef RuleID="ID_ALLOW_A_777" />
          <FileRuleRef RuleID="ID_ALLOW_A_778" />
          <FileRuleRef RuleID="ID_ALLOW_A_779" />
          <FileRuleRef RuleID="ID_ALLOW_A_77A" />
          <FileRuleRef RuleID="ID_ALLOW_A_77B" />
          <FileRuleRef RuleID="ID_ALLOW_A_77C" />
          <FileRuleRef RuleID="ID_ALLOW_A_77D" />
          <FileRuleRef RuleID="ID_ALLOW_A_77E" />
          <FileRuleRef RuleID="ID_ALLOW_A_77F" />
          <FileRuleRef RuleID="ID_ALLOW_A_780" />
          <FileRuleRef RuleID="ID_ALLOW_A_781" />
          <FileRuleRef RuleID="ID_ALLOW_A_782" />
          <FileRuleRef RuleID="ID_ALLOW_A_783" />
          <FileRuleRef RuleID="ID_ALLOW_A_784" />
          <FileRuleRef RuleID="ID_ALLOW_A_785" />
          <FileRuleRef RuleID="ID_ALLOW_A_786" />
          <FileRuleRef RuleID="ID_ALLOW_A_787" />
          <FileRuleRef RuleID="ID_ALLOW_A_788" />
          <FileRuleRef RuleID="ID_ALLOW_A_789" />
          <FileRuleRef RuleID="ID_ALLOW_A_78A" />
          <FileRuleRef RuleID="ID_ALLOW_A_78B" />
          <FileRuleRef RuleID="ID_ALLOW_A_78C" />
          <FileRuleRef RuleID="ID_ALLOW_A_78D" />
          <FileRuleRef RuleID="ID_ALLOW_A_78E" />
          <FileRuleRef RuleID="ID_ALLOW_A_78F" />
          <FileRuleRef RuleID="ID_ALLOW_A_790" />
          <FileRuleRef RuleID="ID_ALLOW_A_791" />
          <FileRuleRef RuleID="ID_ALLOW_A_792" />
          <FileRuleRef RuleID="ID_ALLOW_A_793" />
          <FileRuleRef RuleID="ID_ALLOW_A_794" />
          <FileRuleRef RuleID="ID_ALLOW_A_795" />
          <FileRuleRef RuleID="ID_ALLOW_A_796" />
          <FileRuleRef RuleID="ID_ALLOW_A_797" />
          <FileRuleRef RuleID="ID_ALLOW_A_798" />
          <FileRuleRef RuleID="ID_ALLOW_A_799" />
          <FileRuleRef RuleID="ID_ALLOW_A_79A" />
          <FileRuleRef RuleID="ID_ALLOW_A_79B" />
          <FileRuleRef RuleID="ID_ALLOW_A_79C" />
          <FileRuleRef RuleID="ID_ALLOW_A_79D" />
          <FileRuleRef RuleID="ID_ALLOW_A_79E" />
          <FileRuleRef RuleID="ID_ALLOW_A_79F" />
          <FileRuleRef RuleID="ID_ALLOW_A_7A0" />
          <FileRuleRef RuleID="ID_ALLOW_A_7A1" />
          <FileRuleRef RuleID="ID_ALLOW_A_7A2" />
          <FileRuleRef RuleID="ID_ALLOW_A_7A3" />
          <FileRuleRef RuleID="ID_ALLOW_A_7A4" />
          <FileRuleRef RuleID="ID_ALLOW_A_7A5" />
          <FileRuleRef RuleID="ID_ALLOW_A_7A6" />
          <FileRuleRef RuleID="ID_ALLOW_A_7A7" />
          <FileRuleRef RuleID="ID_ALLOW_A_7A8" />
          <FileRuleRef RuleID="ID_ALLOW_A_7A9" />
          <FileRuleRef RuleID="ID_ALLOW_A_7AA" />
          <FileRuleRef RuleID="ID_ALLOW_A_7AB" />
          <FileRuleRef RuleID="ID_ALLOW_A_7AC" />
          <FileRuleRef RuleID="ID_ALLOW_A_7AD" />
          <FileRuleRef RuleID="ID_ALLOW_A_7AE" />
          <FileRuleRef RuleID="ID_ALLOW_A_7AF" />
          <FileRuleRef RuleID="ID_ALLOW_A_7B0" />
          <FileRuleRef RuleID="ID_ALLOW_A_7B1" />
          <FileRuleRef RuleID="ID_ALLOW_A_7B2" />
          <FileRuleRef RuleID="ID_ALLOW_A_7B3" />
          <FileRuleRef RuleID="ID_ALLOW_A_7B4" />
          <FileRuleRef RuleID="ID_ALLOW_A_7B5" />
          <FileRuleRef RuleID="ID_ALLOW_A_7B6" />
          <FileRuleRef RuleID="ID_ALLOW_A_7B7" />
          <FileRuleRef RuleID="ID_ALLOW_A_7B8" />
          <FileRuleRef RuleID="ID_ALLOW_A_7B9" />
          <FileRuleRef RuleID="ID_ALLOW_A_7BA" />
          <FileRuleRef RuleID="ID_ALLOW_A_7BB" />
          <FileRuleRef RuleID="ID_ALLOW_A_7BC" />
          <FileRuleRef RuleID="ID_ALLOW_A_7BD" />
          <FileRuleRef RuleID="ID_ALLOW_A_7BE" />
          <FileRuleRef RuleID="ID_ALLOW_A_7BF" />
          <FileRuleRef RuleID="ID_ALLOW_A_7C0" />
          <FileRuleRef RuleID="ID_ALLOW_A_7C1" />
          <FileRuleRef RuleID="ID_ALLOW_A_7C2" />
          <FileRuleRef RuleID="ID_ALLOW_A_7C3" />
          <FileRuleRef RuleID="ID_ALLOW_A_7C4" />
          <FileRuleRef RuleID="ID_ALLOW_A_7C5" />
          <FileRuleRef RuleID="ID_ALLOW_A_7C6" />
          <FileRuleRef RuleID="ID_ALLOW_A_7C7" />
          <FileRuleRef RuleID="ID_ALLOW_A_7C8" />
          <FileRuleRef RuleID="ID_ALLOW_A_7C9" />
          <FileRuleRef RuleID="ID_ALLOW_A_7CA" />
          <FileRuleRef RuleID="ID_ALLOW_A_7CB" />
          <FileRuleRef RuleID="ID_ALLOW_A_7CC" />
          <FileRuleRef RuleID="ID_ALLOW_A_7CD" />
          <FileRuleRef RuleID="ID_ALLOW_A_7CE" />
          <FileRuleRef RuleID="ID_ALLOW_A_7CF" />
          <FileRuleRef RuleID="ID_ALLOW_A_7D0" />
          <FileRuleRef RuleID="ID_ALLOW_A_7D1" />
          <FileRuleRef RuleID="ID_ALLOW_A_7D2" />
          <FileRuleRef RuleID="ID_ALLOW_A_7D3" />
          <FileRuleRef RuleID="ID_ALLOW_A_7D4" />
          <FileRuleRef RuleID="ID_ALLOW_A_7D5" />
          <FileRuleRef RuleID="ID_ALLOW_A_7D6" />
          <FileRuleRef RuleID="ID_ALLOW_A_7D7" />
          <FileRuleRef RuleID="ID_ALLOW_A_7D8" />
          <FileRuleRef RuleID="ID_ALLOW_A_7D9" />
          <FileRuleRef RuleID="ID_ALLOW_A_7DA" />
          <FileRuleRef RuleID="ID_ALLOW_A_7DB" />
          <FileRuleRef RuleID="ID_ALLOW_A_7DC" />
          <FileRuleRef RuleID="ID_ALLOW_A_7DD" />
          <FileRuleRef RuleID="ID_ALLOW_A_7DE" />
          <FileRuleRef RuleID="ID_ALLOW_A_7DF" />
          <FileRuleRef RuleID="ID_ALLOW_A_7E0" />
          <FileRuleRef RuleID="ID_ALLOW_A_7E1" />
          <FileRuleRef RuleID="ID_ALLOW_A_7E2" />
          <FileRuleRef RuleID="ID_ALLOW_A_7E3" />
          <FileRuleRef RuleID="ID_ALLOW_A_7E4" />
          <FileRuleRef RuleID="ID_ALLOW_A_7E5" />
          <FileRuleRef RuleID="ID_ALLOW_A_7E6" />
          <FileRuleRef RuleID="ID_ALLOW_A_7E7" />
          <FileRuleRef RuleID="ID_ALLOW_A_7E8" />
          <FileRuleRef RuleID="ID_ALLOW_A_7E9" />
          <FileRuleRef RuleID="ID_ALLOW_A_7EA" />
          <FileRuleRef RuleID="ID_ALLOW_A_7EB" />
          <FileRuleRef RuleID="ID_ALLOW_A_7EC" />
          <FileRuleRef RuleID="ID_ALLOW_A_7ED" />
          <FileRuleRef RuleID="ID_ALLOW_A_7EE" />
          <FileRuleRef RuleID="ID_ALLOW_A_7EF" />
          <FileRuleRef RuleID="ID_ALLOW_A_7F0" />
          <FileRuleRef RuleID="ID_ALLOW_A_7F1" />
          <FileRuleRef RuleID="ID_ALLOW_A_7F2" />
          <FileRuleRef RuleID="ID_ALLOW_A_7F3" />
          <FileRuleRef RuleID="ID_ALLOW_A_7F4" />
          <FileRuleRef RuleID="ID_ALLOW_A_7F5" />
          <FileRuleRef RuleID="ID_ALLOW_A_7F6" />
          <FileRuleRef RuleID="ID_ALLOW_A_7F7" />
          <FileRuleRef RuleID="ID_ALLOW_A_7F8" />
          <FileRuleRef RuleID="ID_ALLOW_A_7F9" />
          <FileRuleRef RuleID="ID_ALLOW_A_7FA" />
          <FileRuleRef RuleID="ID_ALLOW_A_7FB" />
          <FileRuleRef RuleID="ID_ALLOW_A_7FC" />
          <FileRuleRef RuleID="ID_ALLOW_A_7FD" />
          <FileRuleRef RuleID="ID_ALLOW_A_7FE" />
          <FileRuleRef RuleID="ID_ALLOW_A_7FF" />
          <FileRuleRef RuleID="ID_ALLOW_A_800" />
          <FileRuleRef RuleID="ID_ALLOW_A_801" />
          <FileRuleRef RuleID="ID_ALLOW_A_802" />
          <FileRuleRef RuleID="ID_ALLOW_A_803" />
          <FileRuleRef RuleID="ID_ALLOW_A_804" />
          <FileRuleRef RuleID="ID_ALLOW_A_805" />
          <FileRuleRef RuleID="ID_ALLOW_A_806" />
          <FileRuleRef RuleID="ID_ALLOW_A_807" />
          <FileRuleRef RuleID="ID_ALLOW_A_808" />
          <FileRuleRef RuleID="ID_ALLOW_A_809" />
          <FileRuleRef RuleID="ID_ALLOW_A_80A" />
          <FileRuleRef RuleID="ID_ALLOW_A_80B" />
          <FileRuleRef RuleID="ID_ALLOW_A_80C" />
          <FileRuleRef RuleID="ID_ALLOW_A_80D" />
          <FileRuleRef RuleID="ID_ALLOW_A_80E" />
          <FileRuleRef RuleID="ID_ALLOW_A_80F" />
          <FileRuleRef RuleID="ID_ALLOW_A_810" />
          <FileRuleRef RuleID="ID_ALLOW_A_811" />
          <FileRuleRef RuleID="ID_ALLOW_A_812" />
          <FileRuleRef RuleID="ID_ALLOW_A_813" />
          <FileRuleRef RuleID="ID_ALLOW_A_814" />
          <FileRuleRef RuleID="ID_ALLOW_A_815" />
          <FileRuleRef RuleID="ID_ALLOW_A_816" />
          <FileRuleRef RuleID="ID_ALLOW_A_817" />
          <FileRuleRef RuleID="ID_ALLOW_A_818" />
          <FileRuleRef RuleID="ID_ALLOW_A_819" />
          <FileRuleRef RuleID="ID_ALLOW_A_81A" />
          <FileRuleRef RuleID="ID_ALLOW_A_81B" />
          <FileRuleRef RuleID="ID_ALLOW_A_81C" />
          <FileRuleRef RuleID="ID_ALLOW_A_81D" />
          <FileRuleRef RuleID="ID_ALLOW_A_81E" />
          <FileRuleRef RuleID="ID_ALLOW_A_81F" />
          <FileRuleRef RuleID="ID_ALLOW_A_820" />
          <FileRuleRef RuleID="ID_ALLOW_A_821" />
          <FileRuleRef RuleID="ID_ALLOW_A_822" />
          <FileRuleRef RuleID="ID_ALLOW_A_823" />
          <FileRuleRef RuleID="ID_ALLOW_A_824" />
          <FileRuleRef RuleID="ID_ALLOW_A_825" />
          <FileRuleRef RuleID="ID_ALLOW_A_826" />
          <FileRuleRef RuleID="ID_ALLOW_A_827" />
          <FileRuleRef RuleID="ID_ALLOW_A_828" />
          <FileRuleRef RuleID="ID_ALLOW_A_829" />
          <FileRuleRef RuleID="ID_ALLOW_A_82A" />
          <FileRuleRef RuleID="ID_ALLOW_A_82B" />
          <FileRuleRef RuleID="ID_ALLOW_A_82C" />
          <FileRuleRef RuleID="ID_ALLOW_A_82D" />
          <FileRuleRef RuleID="ID_ALLOW_A_82E" />
          <FileRuleRef RuleID="ID_ALLOW_A_82F" />
          <FileRuleRef RuleID="ID_ALLOW_A_830" />
          <FileRuleRef RuleID="ID_ALLOW_A_831" />
          <FileRuleRef RuleID="ID_ALLOW_A_832" />
          <FileRuleRef RuleID="ID_ALLOW_A_833" />
          <FileRuleRef RuleID="ID_ALLOW_A_834" />
          <FileRuleRef RuleID="ID_ALLOW_A_835" />
          <FileRuleRef RuleID="ID_ALLOW_A_836" />
          <FileRuleRef RuleID="ID_ALLOW_A_837" />
          <FileRuleRef RuleID="ID_ALLOW_A_838" />
          <FileRuleRef RuleID="ID_ALLOW_A_839" />
          <FileRuleRef RuleID="ID_ALLOW_A_83A" />
          <FileRuleRef RuleID="ID_ALLOW_A_83B" />
          <FileRuleRef RuleID="ID_ALLOW_A_83C" />
          <FileRuleRef RuleID="ID_ALLOW_A_83D" />
          <FileRuleRef RuleID="ID_ALLOW_A_83E" />
          <FileRuleRef RuleID="ID_ALLOW_A_83F" />
          <FileRuleRef RuleID="ID_ALLOW_A_840" />
          <FileRuleRef RuleID="ID_ALLOW_A_841" />
          <FileRuleRef RuleID="ID_ALLOW_A_842" />
          <FileRuleRef RuleID="ID_ALLOW_A_843" />
          <FileRuleRef RuleID="ID_ALLOW_A_844" />
          <FileRuleRef RuleID="ID_ALLOW_A_845" />
          <FileRuleRef RuleID="ID_ALLOW_A_846" />
          <FileRuleRef RuleID="ID_ALLOW_A_847" />
          <FileRuleRef RuleID="ID_ALLOW_A_848" />
          <FileRuleRef RuleID="ID_ALLOW_A_849" />
          <FileRuleRef RuleID="ID_ALLOW_A_84A" />
          <FileRuleRef RuleID="ID_ALLOW_A_84B" />
          <FileRuleRef RuleID="ID_ALLOW_A_84C" />
          <FileRuleRef RuleID="ID_ALLOW_A_84D" />
          <FileRuleRef RuleID="ID_ALLOW_A_84E" />
          <FileRuleRef RuleID="ID_ALLOW_A_84F" />
          <FileRuleRef RuleID="ID_ALLOW_A_850" />
          <FileRuleRef RuleID="ID_ALLOW_A_851" />
          <FileRuleRef RuleID="ID_ALLOW_A_852" />
          <FileRuleRef RuleID="ID_ALLOW_A_853" />
          <FileRuleRef RuleID="ID_ALLOW_A_854" />
          <FileRuleRef RuleID="ID_ALLOW_A_855" />
          <FileRuleRef RuleID="ID_ALLOW_A_856" />
          <FileRuleRef RuleID="ID_ALLOW_A_857" />
          <FileRuleRef RuleID="ID_ALLOW_A_858" />
          <FileRuleRef RuleID="ID_ALLOW_A_859" />
          <FileRuleRef RuleID="ID_ALLOW_A_85A" />
          <FileRuleRef RuleID="ID_ALLOW_A_85B" />
          <FileRuleRef RuleID="ID_ALLOW_A_85C" />
          <FileRuleRef RuleID="ID_ALLOW_A_85D" />
          <FileRuleRef RuleID="ID_ALLOW_A_85E" />
          <FileRuleRef RuleID="ID_ALLOW_A_85F" />
          <FileRuleRef RuleID="ID_ALLOW_A_860" />
          <FileRuleRef RuleID="ID_ALLOW_A_861" />
          <FileRuleRef RuleID="ID_ALLOW_A_862" />
          <FileRuleRef RuleID="ID_ALLOW_A_863" />
          <FileRuleRef RuleID="ID_ALLOW_A_864" />
          <FileRuleRef RuleID="ID_ALLOW_A_865" />
          <FileRuleRef RuleID="ID_ALLOW_A_866" />
          <FileRuleRef RuleID="ID_ALLOW_A_867" />
          <FileRuleRef RuleID="ID_ALLOW_A_868" />
          <FileRuleRef RuleID="ID_ALLOW_A_869" />
          <FileRuleRef RuleID="ID_ALLOW_A_86A" />
          <FileRuleRef RuleID="ID_ALLOW_A_86B" />
          <FileRuleRef RuleID="ID_ALLOW_A_86C" />
          <FileRuleRef RuleID="ID_ALLOW_A_86D" />
          <FileRuleRef RuleID="ID_ALLOW_A_86E" />
          <FileRuleRef RuleID="ID_ALLOW_A_86F" />
          <FileRuleRef RuleID="ID_ALLOW_A_870" />
          <FileRuleRef RuleID="ID_ALLOW_A_871" />
          <FileRuleRef RuleID="ID_ALLOW_A_872" />
          <FileRuleRef RuleID="ID_ALLOW_A_873" />
          <FileRuleRef RuleID="ID_ALLOW_A_874" />
          <FileRuleRef RuleID="ID_ALLOW_A_875" />
          <FileRuleRef RuleID="ID_ALLOW_A_876" />
          <FileRuleRef RuleID="ID_ALLOW_A_877" />
          <FileRuleRef RuleID="ID_ALLOW_A_878" />
          <FileRuleRef RuleID="ID_ALLOW_A_879" />
          <FileRuleRef RuleID="ID_ALLOW_A_87A" />
          <FileRuleRef RuleID="ID_ALLOW_A_87B" />
          <FileRuleRef RuleID="ID_ALLOW_A_87C" />
          <FileRuleRef RuleID="ID_ALLOW_A_87D" />
          <FileRuleRef RuleID="ID_ALLOW_A_87E" />
          <FileRuleRef RuleID="ID_ALLOW_A_87F" />
          <FileRuleRef RuleID="ID_ALLOW_A_880" />
          <FileRuleRef RuleID="ID_ALLOW_A_881" />
          <FileRuleRef RuleID="ID_ALLOW_A_882" />
          <FileRuleRef RuleID="ID_ALLOW_A_883" />
          <FileRuleRef RuleID="ID_ALLOW_A_884" />
          <FileRuleRef RuleID="ID_ALLOW_A_885" />
          <FileRuleRef RuleID="ID_ALLOW_A_886" />
          <FileRuleRef RuleID="ID_ALLOW_A_887" />
          <FileRuleRef RuleID="ID_ALLOW_A_888" />
          <FileRuleRef RuleID="ID_ALLOW_A_889" />
          <FileRuleRef RuleID="ID_ALLOW_A_88A" />
          <FileRuleRef RuleID="ID_ALLOW_A_88B" />
          <FileRuleRef RuleID="ID_ALLOW_A_88C" />
          <FileRuleRef RuleID="ID_ALLOW_A_88D" />
          <FileRuleRef RuleID="ID_ALLOW_A_88E" />
          <FileRuleRef RuleID="ID_ALLOW_A_88F" />
          <FileRuleRef RuleID="ID_ALLOW_A_890" />
          <FileRuleRef RuleID="ID_ALLOW_A_891" />
          <FileRuleRef RuleID="ID_ALLOW_A_892" />
          <FileRuleRef RuleID="ID_ALLOW_A_893" />
          <FileRuleRef RuleID="ID_ALLOW_A_894" />
          <FileRuleRef RuleID="ID_ALLOW_A_895" />
          <FileRuleRef RuleID="ID_ALLOW_A_896" />
          <FileRuleRef RuleID="ID_ALLOW_A_897" />
          <FileRuleRef RuleID="ID_ALLOW_A_898" />
          <FileRuleRef RuleID="ID_ALLOW_A_899" />
          <FileRuleRef RuleID="ID_ALLOW_A_89A" />
          <FileRuleRef RuleID="ID_ALLOW_A_89B" />
          <FileRuleRef RuleID="ID_ALLOW_A_89C" />
          <FileRuleRef RuleID="ID_ALLOW_A_89D" />
          <FileRuleRef RuleID="ID_ALLOW_A_89E" />
          <FileRuleRef RuleID="ID_ALLOW_A_89F" />
          <FileRuleRef RuleID="ID_ALLOW_A_8A0" />
          <FileRuleRef RuleID="ID_ALLOW_A_8A1" />
          <FileRuleRef RuleID="ID_ALLOW_A_8A2" />
          <FileRuleRef RuleID="ID_ALLOW_A_8A3" />
          <FileRuleRef RuleID="ID_ALLOW_A_8A4" />
          <FileRuleRef RuleID="ID_ALLOW_A_8A5" />
          <FileRuleRef RuleID="ID_ALLOW_A_8A6" />
          <FileRuleRef RuleID="ID_ALLOW_A_8A7" />
          <FileRuleRef RuleID="ID_ALLOW_A_8A8" />
          <FileRuleRef RuleID="ID_ALLOW_A_8A9" />
          <FileRuleRef RuleID="ID_ALLOW_A_8AA" />
          <FileRuleRef RuleID="ID_ALLOW_A_8AB" />
          <FileRuleRef RuleID="ID_ALLOW_A_8AC" />
          <FileRuleRef RuleID="ID_ALLOW_A_8AD" />
          <FileRuleRef RuleID="ID_ALLOW_A_8AE" />
          <FileRuleRef RuleID="ID_ALLOW_A_8AF" />
          <FileRuleRef RuleID="ID_ALLOW_A_8B0" />
          <FileRuleRef RuleID="ID_ALLOW_A_8B1" />
          <FileRuleRef RuleID="ID_ALLOW_A_8B2" />
          <FileRuleRef RuleID="ID_ALLOW_A_8B3" />
          <FileRuleRef RuleID="ID_ALLOW_A_8B4" />
          <FileRuleRef RuleID="ID_ALLOW_A_8B5" />
          <FileRuleRef RuleID="ID_ALLOW_A_8B6" />
          <FileRuleRef RuleID="ID_ALLOW_A_8B7" />
          <FileRuleRef RuleID="ID_ALLOW_A_8B8" />
          <FileRuleRef RuleID="ID_ALLOW_A_8B9" />
          <FileRuleRef RuleID="ID_ALLOW_A_8BA" />
          <FileRuleRef RuleID="ID_ALLOW_A_8BB" />
          <FileRuleRef RuleID="ID_ALLOW_A_8BC" />
          <FileRuleRef RuleID="ID_ALLOW_A_8BD" />
          <FileRuleRef RuleID="ID_ALLOW_A_8BE" />
          <FileRuleRef RuleID="ID_ALLOW_A_8BF" />
          <FileRuleRef RuleID="ID_ALLOW_A_8C0" />
          <FileRuleRef RuleID="ID_ALLOW_A_8C1" />
          <FileRuleRef RuleID="ID_ALLOW_A_8C2" />
          <FileRuleRef RuleID="ID_ALLOW_A_8C3" />
          <FileRuleRef RuleID="ID_ALLOW_A_8C4" />
          <FileRuleRef RuleID="ID_ALLOW_A_8C5" />
          <FileRuleRef RuleID="ID_ALLOW_A_8C6" />
          <FileRuleRef RuleID="ID_ALLOW_A_8C7" />
          <FileRuleRef RuleID="ID_ALLOW_A_8C8" />
          <FileRuleRef RuleID="ID_ALLOW_A_8C9" />
          <FileRuleRef RuleID="ID_ALLOW_A_8CA" />
          <FileRuleRef RuleID="ID_ALLOW_A_8CB" />
          <FileRuleRef RuleID="ID_ALLOW_A_8CC" />
          <FileRuleRef RuleID="ID_ALLOW_A_8CD" />
          <FileRuleRef RuleID="ID_ALLOW_A_8CE" />
          <FileRuleRef RuleID="ID_ALLOW_A_8CF" />
          <FileRuleRef RuleID="ID_ALLOW_A_8D0" />
          <FileRuleRef RuleID="ID_ALLOW_A_8D1" />
          <FileRuleRef RuleID="ID_ALLOW_A_8D2" />
          <FileRuleRef RuleID="ID_ALLOW_A_8D3" />
          <FileRuleRef RuleID="ID_ALLOW_A_8D4" />
          <FileRuleRef RuleID="ID_ALLOW_A_8D5" />
          <FileRuleRef RuleID="ID_ALLOW_A_8D6" />
          <FileRuleRef RuleID="ID_ALLOW_A_8D7" />
          <FileRuleRef RuleID="ID_ALLOW_A_8D8" />
          <FileRuleRef RuleID="ID_ALLOW_A_8D9" />
          <FileRuleRef RuleID="ID_ALLOW_A_8DA" />
          <FileRuleRef RuleID="ID_ALLOW_A_8DB" />
          <FileRuleRef RuleID="ID_ALLOW_A_8DC" />
          <FileRuleRef RuleID="ID_ALLOW_A_8DD" />
          <FileRuleRef RuleID="ID_ALLOW_A_8DE" />
          <FileRuleRef RuleID="ID_ALLOW_A_8DF" />
          <FileRuleRef RuleID="ID_ALLOW_A_8E0" />
          <FileRuleRef RuleID="ID_ALLOW_A_8E1" />
          <FileRuleRef RuleID="ID_ALLOW_A_8E2" />
          <FileRuleRef RuleID="ID_ALLOW_A_8E3" />
          <FileRuleRef RuleID="ID_ALLOW_A_8E4" />
          <FileRuleRef RuleID="ID_ALLOW_A_8E5" />
          <FileRuleRef RuleID="ID_ALLOW_A_8E6" />
          <FileRuleRef RuleID="ID_ALLOW_A_8E7" />
          <FileRuleRef RuleID="ID_ALLOW_A_8E8" />
          <FileRuleRef RuleID="ID_ALLOW_A_8E9" />
          <FileRuleRef RuleID="ID_ALLOW_A_8EA" />
          <FileRuleRef RuleID="ID_ALLOW_A_8EB" />
          <FileRuleRef RuleID="ID_ALLOW_A_8EC" />
          <FileRuleRef RuleID="ID_ALLOW_A_8ED" />
          <FileRuleRef RuleID="ID_ALLOW_A_8EE" />
          <FileRuleRef RuleID="ID_ALLOW_A_8EF" />
          <FileRuleRef RuleID="ID_ALLOW_A_8F0" />
          <FileRuleRef RuleID="ID_ALLOW_A_8F1" />
          <FileRuleRef RuleID="ID_ALLOW_A_8F2" />
          <FileRuleRef RuleID="ID_ALLOW_A_8F3" />
          <FileRuleRef RuleID="ID_ALLOW_A_8F4" />
          <FileRuleRef RuleID="ID_ALLOW_A_8F5" />
          <FileRuleRef RuleID="ID_ALLOW_A_8F6" />
          <FileRuleRef RuleID="ID_ALLOW_A_8F7" />
          <FileRuleRef RuleID="ID_ALLOW_A_8F8" />
          <FileRuleRef RuleID="ID_ALLOW_A_8F9" />
          <FileRuleRef RuleID="ID_ALLOW_A_8FA" />
          <FileRuleRef RuleID="ID_ALLOW_A_8FB" />
          <FileRuleRef RuleID="ID_ALLOW_A_8FC" />
          <FileRuleRef RuleID="ID_ALLOW_A_8FD" />
          <FileRuleRef RuleID="ID_ALLOW_A_8FE" />
          <FileRuleRef RuleID="ID_ALLOW_A_8FF" />
          <FileRuleRef RuleID="ID_ALLOW_A_900" />
          <FileRuleRef RuleID="ID_ALLOW_A_901" />
          <FileRuleRef RuleID="ID_ALLOW_A_902" />
          <FileRuleRef RuleID="ID_ALLOW_A_903" />
          <FileRuleRef RuleID="ID_ALLOW_A_904" />
          <FileRuleRef RuleID="ID_ALLOW_A_905" />
          <FileRuleRef RuleID="ID_ALLOW_A_906" />
          <FileRuleRef RuleID="ID_ALLOW_A_907" />
          <FileRuleRef RuleID="ID_ALLOW_A_908" />
          <FileRuleRef RuleID="ID_ALLOW_A_909" />
          <FileRuleRef RuleID="ID_ALLOW_A_90A" />
          <FileRuleRef RuleID="ID_ALLOW_A_90B" />
          <FileRuleRef RuleID="ID_ALLOW_A_90C" />
          <FileRuleRef RuleID="ID_ALLOW_A_90D" />
          <FileRuleRef RuleID="ID_ALLOW_A_90E" />
          <FileRuleRef RuleID="ID_ALLOW_A_90F" />
          <FileRuleRef RuleID="ID_ALLOW_A_910" />
          <FileRuleRef RuleID="ID_ALLOW_A_911" />
          <FileRuleRef RuleID="ID_ALLOW_A_912" />
          <FileRuleRef RuleID="ID_ALLOW_A_913" />
          <FileRuleRef RuleID="ID_ALLOW_A_914" />
          <FileRuleRef RuleID="ID_ALLOW_A_915" />
          <FileRuleRef RuleID="ID_ALLOW_A_916" />
          <FileRuleRef RuleID="ID_ALLOW_A_917" />
          <FileRuleRef RuleID="ID_ALLOW_A_918" />
          <FileRuleRef RuleID="ID_ALLOW_A_919" />
          <FileRuleRef RuleID="ID_ALLOW_A_91A" />
          <FileRuleRef RuleID="ID_ALLOW_A_91B" />
          <FileRuleRef RuleID="ID_ALLOW_A_91C" />
          <FileRuleRef RuleID="ID_ALLOW_A_91D" />
          <FileRuleRef RuleID="ID_ALLOW_A_91E" />
          <FileRuleRef RuleID="ID_ALLOW_A_91F" />
          <FileRuleRef RuleID="ID_ALLOW_A_920" />
          <FileRuleRef RuleID="ID_ALLOW_A_921" />
          <FileRuleRef RuleID="ID_ALLOW_A_922" />
          <FileRuleRef RuleID="ID_ALLOW_A_923" />
          <FileRuleRef RuleID="ID_ALLOW_A_924" />
          <FileRuleRef RuleID="ID_ALLOW_A_925" />
          <FileRuleRef RuleID="ID_ALLOW_A_926" />
          <FileRuleRef RuleID="ID_ALLOW_A_927" />
          <FileRuleRef RuleID="ID_ALLOW_A_928" />
          <FileRuleRef RuleID="ID_ALLOW_A_929" />
          <FileRuleRef RuleID="ID_ALLOW_A_92A" />
          <FileRuleRef RuleID="ID_ALLOW_A_92B" />
          <FileRuleRef RuleID="ID_ALLOW_A_92C" />
          <FileRuleRef RuleID="ID_ALLOW_A_92D" />
          <FileRuleRef RuleID="ID_ALLOW_A_92E" />
          <FileRuleRef RuleID="ID_ALLOW_A_92F" />
          <FileRuleRef RuleID="ID_ALLOW_A_930" />
          <FileRuleRef RuleID="ID_ALLOW_A_931" />
          <FileRuleRef RuleID="ID_ALLOW_A_932" />
          <FileRuleRef RuleID="ID_ALLOW_A_933" />
          <FileRuleRef RuleID="ID_ALLOW_A_934" />
          <FileRuleRef RuleID="ID_ALLOW_A_935" />
          <FileRuleRef RuleID="ID_ALLOW_A_936" />
          <FileRuleRef RuleID="ID_ALLOW_A_937" />
          <FileRuleRef RuleID="ID_ALLOW_A_938" />
          <FileRuleRef RuleID="ID_ALLOW_A_939" />
          <FileRuleRef RuleID="ID_ALLOW_A_93A" />
          <FileRuleRef RuleID="ID_ALLOW_A_93B" />
          <FileRuleRef RuleID="ID_ALLOW_A_93C" />
          <FileRuleRef RuleID="ID_ALLOW_A_93D" />
          <FileRuleRef RuleID="ID_ALLOW_A_93E" />
          <FileRuleRef RuleID="ID_ALLOW_A_93F" />
          <FileRuleRef RuleID="ID_ALLOW_A_940" />
          <FileRuleRef RuleID="ID_ALLOW_A_941" />
          <FileRuleRef RuleID="ID_ALLOW_A_942" />
          <FileRuleRef RuleID="ID_ALLOW_A_943" />
          <FileRuleRef RuleID="ID_ALLOW_A_944" />
          <FileRuleRef RuleID="ID_ALLOW_A_945" />
          <FileRuleRef RuleID="ID_ALLOW_A_946" />
          <FileRuleRef RuleID="ID_ALLOW_A_947" />
          <FileRuleRef RuleID="ID_ALLOW_A_948" />
          <FileRuleRef RuleID="ID_ALLOW_A_949" />
          <FileRuleRef RuleID="ID_ALLOW_A_94A" />
          <FileRuleRef RuleID="ID_ALLOW_A_94B" />
          <FileRuleRef RuleID="ID_ALLOW_A_94C" />
          <FileRuleRef RuleID="ID_ALLOW_A_94D" />
          <FileRuleRef RuleID="ID_ALLOW_A_94E" />
          <FileRuleRef RuleID="ID_ALLOW_A_94F" />
          <FileRuleRef RuleID="ID_ALLOW_A_950" />
          <FileRuleRef RuleID="ID_ALLOW_A_951" />
          <FileRuleRef RuleID="ID_ALLOW_A_952" />
          <FileRuleRef RuleID="ID_ALLOW_A_953" />
          <FileRuleRef RuleID="ID_ALLOW_A_954" />
          <FileRuleRef RuleID="ID_ALLOW_A_955" />
          <FileRuleRef RuleID="ID_ALLOW_A_956" />
          <FileRuleRef RuleID="ID_ALLOW_A_957" />
          <FileRuleRef RuleID="ID_ALLOW_A_958" />
          <FileRuleRef RuleID="ID_ALLOW_A_959" />
          <FileRuleRef RuleID="ID_ALLOW_A_95A" />
          <FileRuleRef RuleID="ID_ALLOW_A_95B" />
          <FileRuleRef RuleID="ID_ALLOW_A_95C" />
          <FileRuleRef RuleID="ID_ALLOW_A_95D" />
          <FileRuleRef RuleID="ID_ALLOW_A_95E" />
          <FileRuleRef RuleID="ID_ALLOW_A_95F" />
          <FileRuleRef RuleID="ID_ALLOW_A_960" />
          <FileRuleRef RuleID="ID_ALLOW_A_961" />
          <FileRuleRef RuleID="ID_ALLOW_A_962" />
          <FileRuleRef RuleID="ID_ALLOW_A_963" />
          <FileRuleRef RuleID="ID_ALLOW_A_964" />
          <FileRuleRef RuleID="ID_ALLOW_A_965" />
          <FileRuleRef RuleID="ID_ALLOW_A_966" />
          <FileRuleRef RuleID="ID_ALLOW_A_967" />
          <FileRuleRef RuleID="ID_ALLOW_A_968" />
          <FileRuleRef RuleID="ID_ALLOW_A_969" />
          <FileRuleRef RuleID="ID_ALLOW_A_96A" />
          <FileRuleRef RuleID="ID_ALLOW_A_96B" />
          <FileRuleRef RuleID="ID_ALLOW_A_96C" />
          <FileRuleRef RuleID="ID_ALLOW_A_96D" />
          <FileRuleRef RuleID="ID_ALLOW_A_96E" />
          <FileRuleRef RuleID="ID_ALLOW_A_96F" />
          <FileRuleRef RuleID="ID_ALLOW_A_970" />
          <FileRuleRef RuleID="ID_ALLOW_A_971" />
          <FileRuleRef RuleID="ID_ALLOW_A_972" />
          <FileRuleRef RuleID="ID_ALLOW_A_973" />
          <FileRuleRef RuleID="ID_ALLOW_A_974" />
          <FileRuleRef RuleID="ID_ALLOW_A_975" />
          <FileRuleRef RuleID="ID_ALLOW_A_976" />
          <FileRuleRef RuleID="ID_ALLOW_A_977" />
          <FileRuleRef RuleID="ID_ALLOW_A_978" />
          <FileRuleRef RuleID="ID_ALLOW_A_979" />
          <FileRuleRef RuleID="ID_ALLOW_A_97A" />
          <FileRuleRef RuleID="ID_ALLOW_A_97B" />
          <FileRuleRef RuleID="ID_ALLOW_A_97C" />
          <FileRuleRef RuleID="ID_ALLOW_A_97D" />
          <FileRuleRef RuleID="ID_ALLOW_A_97E" />
          <FileRuleRef RuleID="ID_ALLOW_A_97F" />
          <FileRuleRef RuleID="ID_ALLOW_A_980" />
          <FileRuleRef RuleID="ID_ALLOW_A_981" />
          <FileRuleRef RuleID="ID_ALLOW_A_982" />
          <FileRuleRef RuleID="ID_ALLOW_A_983" />
          <FileRuleRef RuleID="ID_ALLOW_A_984" />
          <FileRuleRef RuleID="ID_ALLOW_A_985" />
          <FileRuleRef RuleID="ID_ALLOW_A_986" />
          <FileRuleRef RuleID="ID_ALLOW_A_987" />
          <FileRuleRef RuleID="ID_ALLOW_A_988" />
          <FileRuleRef RuleID="ID_ALLOW_A_989" />
          <FileRuleRef RuleID="ID_ALLOW_A_98A" />
          <FileRuleRef RuleID="ID_ALLOW_A_98B" />
          <FileRuleRef RuleID="ID_ALLOW_A_98C" />
          <FileRuleRef RuleID="ID_ALLOW_A_98D" />
          <FileRuleRef RuleID="ID_ALLOW_A_98E" />
          <FileRuleRef RuleID="ID_ALLOW_A_98F" />
          <FileRuleRef RuleID="ID_ALLOW_A_990" />
          <FileRuleRef RuleID="ID_ALLOW_A_991" />
          <FileRuleRef RuleID="ID_ALLOW_A_992" />
          <FileRuleRef RuleID="ID_ALLOW_A_993" />
          <FileRuleRef RuleID="ID_ALLOW_A_994" />
          <FileRuleRef RuleID="ID_ALLOW_A_995" />
          <FileRuleRef RuleID="ID_ALLOW_A_996" />
          <FileRuleRef RuleID="ID_ALLOW_A_997" />
          <FileRuleRef RuleID="ID_ALLOW_A_998" />
          <FileRuleRef RuleID="ID_ALLOW_A_999" />
          <FileRuleRef RuleID="ID_ALLOW_A_99A" />
          <FileRuleRef RuleID="ID_ALLOW_A_99B" />
          <FileRuleRef RuleID="ID_ALLOW_A_99C" />
          <FileRuleRef RuleID="ID_ALLOW_A_99D" />
          <FileRuleRef RuleID="ID_ALLOW_A_99E" />
          <FileRuleRef RuleID="ID_ALLOW_A_99F" />
          <FileRuleRef RuleID="ID_ALLOW_A_9A0" />
          <FileRuleRef RuleID="ID_ALLOW_A_9A1" />
          <FileRuleRef RuleID="ID_ALLOW_A_9A2" />
          <FileRuleRef RuleID="ID_ALLOW_A_9A3" />
          <FileRuleRef RuleID="ID_ALLOW_A_9A4" />
          <FileRuleRef RuleID="ID_ALLOW_A_9A5" />
          <FileRuleRef RuleID="ID_ALLOW_A_9A6" />
          <FileRuleRef RuleID="ID_ALLOW_A_9A7" />
          <FileRuleRef RuleID="ID_ALLOW_A_9A8" />
          <FileRuleRef RuleID="ID_ALLOW_A_9A9" />
          <FileRuleRef RuleID="ID_ALLOW_A_9AA" />
          <FileRuleRef RuleID="ID_ALLOW_A_9AB" />
          <FileRuleRef RuleID="ID_ALLOW_A_9AC" />
          <FileRuleRef RuleID="ID_ALLOW_A_9AD" />
          <FileRuleRef RuleID="ID_ALLOW_A_9AE" />
          <FileRuleRef RuleID="ID_ALLOW_A_9AF" />
          <FileRuleRef RuleID="ID_ALLOW_A_9B0" />
          <FileRuleRef RuleID="ID_ALLOW_A_9B1" />
          <FileRuleRef RuleID="ID_ALLOW_A_9B2" />
          <FileRuleRef RuleID="ID_ALLOW_A_9B3" />
          <FileRuleRef RuleID="ID_ALLOW_A_9B4" />
          <FileRuleRef RuleID="ID_ALLOW_A_9B5" />
          <FileRuleRef RuleID="ID_ALLOW_A_9B6" />
          <FileRuleRef RuleID="ID_ALLOW_A_9B7" />
          <FileRuleRef RuleID="ID_ALLOW_A_9B8" />
          <FileRuleRef RuleID="ID_ALLOW_A_9B9" />
          <FileRuleRef RuleID="ID_ALLOW_A_9BA" />
          <FileRuleRef RuleID="ID_ALLOW_A_9BB" />
          <FileRuleRef RuleID="ID_ALLOW_A_9BC" />
          <FileRuleRef RuleID="ID_ALLOW_A_9BD" />
          <FileRuleRef RuleID="ID_ALLOW_A_9BE" />
          <FileRuleRef RuleID="ID_ALLOW_A_9BF" />
          <FileRuleRef RuleID="ID_ALLOW_A_9C0" />
          <FileRuleRef RuleID="ID_ALLOW_A_9C1" />
          <FileRuleRef RuleID="ID_ALLOW_A_9C2" />
          <FileRuleRef RuleID="ID_ALLOW_A_9C3" />
          <FileRuleRef RuleID="ID_ALLOW_A_9C4" />
          <FileRuleRef RuleID="ID_ALLOW_A_9C5" />
          <FileRuleRef RuleID="ID_ALLOW_A_9C6" />
          <FileRuleRef RuleID="ID_ALLOW_A_9C7" />
          <FileRuleRef RuleID="ID_ALLOW_A_9C8" />
          <FileRuleRef RuleID="ID_ALLOW_A_9C9" />
          <FileRuleRef RuleID="ID_ALLOW_A_9CA" />
          <FileRuleRef RuleID="ID_ALLOW_A_9CB" />
          <FileRuleRef RuleID="ID_ALLOW_A_9CC" />
          <FileRuleRef RuleID="ID_ALLOW_A_9CD" />
          <FileRuleRef RuleID="ID_ALLOW_A_9CE" />
          <FileRuleRef RuleID="ID_ALLOW_A_9CF" />
          <FileRuleRef RuleID="ID_ALLOW_A_9D0" />
          <FileRuleRef RuleID="ID_ALLOW_A_9D1" />
          <FileRuleRef RuleID="ID_ALLOW_A_9D2" />
          <FileRuleRef RuleID="ID_ALLOW_A_9D3" />
          <FileRuleRef RuleID="ID_ALLOW_A_9D4" />
          <FileRuleRef RuleID="ID_ALLOW_A_9D5" />
          <FileRuleRef RuleID="ID_ALLOW_A_9D6" />
          <FileRuleRef RuleID="ID_ALLOW_A_9D7" />
          <FileRuleRef RuleID="ID_ALLOW_A_9D8" />
          <FileRuleRef RuleID="ID_ALLOW_A_9D9" />
          <FileRuleRef RuleID="ID_ALLOW_A_9DA" />
          <FileRuleRef RuleID="ID_ALLOW_A_9DB" />
          <FileRuleRef RuleID="ID_ALLOW_A_9DC" />
          <FileRuleRef RuleID="ID_ALLOW_A_9DD" />
          <FileRuleRef RuleID="ID_ALLOW_A_9DE" />
          <FileRuleRef RuleID="ID_ALLOW_A_9DF" />
          <FileRuleRef RuleID="ID_ALLOW_A_9E0" />
          <FileRuleRef RuleID="ID_ALLOW_A_9E1" />
          <FileRuleRef RuleID="ID_ALLOW_A_9E2" />
          <FileRuleRef RuleID="ID_ALLOW_A_9E3" />
          <FileRuleRef RuleID="ID_ALLOW_A_9E4" />
          <FileRuleRef RuleID="ID_ALLOW_A_9E5" />
          <FileRuleRef RuleID="ID_ALLOW_A_9E6" />
          <FileRuleRef RuleID="ID_ALLOW_A_9E7" />
          <FileRuleRef RuleID="ID_ALLOW_A_9E8" />
          <FileRuleRef RuleID="ID_ALLOW_A_9E9" />
          <FileRuleRef RuleID="ID_ALLOW_A_9EA" />
          <FileRuleRef RuleID="ID_ALLOW_A_9EB" />
          <FileRuleRef RuleID="ID_ALLOW_A_9EC" />
          <FileRuleRef RuleID="ID_ALLOW_A_9ED" />
          <FileRuleRef RuleID="ID_ALLOW_A_9EE" />
          <FileRuleRef RuleID="ID_ALLOW_A_9EF" />
          <FileRuleRef RuleID="ID_ALLOW_A_9F0" />
          <FileRuleRef RuleID="ID_ALLOW_A_9F1" />
          <FileRuleRef RuleID="ID_ALLOW_A_9F2" />
          <FileRuleRef RuleID="ID_ALLOW_A_9F3" />
          <FileRuleRef RuleID="ID_ALLOW_A_9F4" />
          <FileRuleRef RuleID="ID_ALLOW_A_9F5" />
          <FileRuleRef RuleID="ID_ALLOW_A_9F6" />
          <FileRuleRef RuleID="ID_ALLOW_A_9F7" />
          <FileRuleRef RuleID="ID_ALLOW_A_9F8" />
          <FileRuleRef RuleID="ID_ALLOW_A_9F9" />
          <FileRuleRef RuleID="ID_ALLOW_A_9FA" />
          <FileRuleRef RuleID="ID_ALLOW_A_9FB" />
          <FileRuleRef RuleID="ID_ALLOW_A_9FC" />
          <FileRuleRef RuleID="ID_ALLOW_A_9FD" />
          <FileRuleRef RuleID="ID_ALLOW_A_9FE" />
          <FileRuleRef RuleID="ID_ALLOW_A_9FF" />
          <FileRuleRef RuleID="ID_ALLOW_A_A00" />
          <FileRuleRef RuleID="ID_ALLOW_A_A01" />
          <FileRuleRef RuleID="ID_ALLOW_A_A02" />
          <FileRuleRef RuleID="ID_ALLOW_A_A03" />
          <FileRuleRef RuleID="ID_ALLOW_A_A04" />
          <FileRuleRef RuleID="ID_ALLOW_A_A05" />
          <FileRuleRef RuleID="ID_ALLOW_A_A06" />
          <FileRuleRef RuleID="ID_ALLOW_A_A07" />
          <FileRuleRef RuleID="ID_ALLOW_A_A08" />
          <FileRuleRef RuleID="ID_ALLOW_A_A09" />
          <FileRuleRef RuleID="ID_ALLOW_A_A0A" />
          <FileRuleRef RuleID="ID_ALLOW_A_A0B" />
          <FileRuleRef RuleID="ID_ALLOW_A_A0C" />
          <FileRuleRef RuleID="ID_ALLOW_A_A0D" />
          <FileRuleRef RuleID="ID_ALLOW_A_A0E" />
          <FileRuleRef RuleID="ID_ALLOW_A_A0F" />
          <FileRuleRef RuleID="ID_ALLOW_A_A10" />
          <FileRuleRef RuleID="ID_ALLOW_A_A11" />
          <FileRuleRef RuleID="ID_ALLOW_A_A12" />
          <FileRuleRef RuleID="ID_ALLOW_A_A13" />
          <FileRuleRef RuleID="ID_ALLOW_A_A14" />
          <FileRuleRef RuleID="ID_ALLOW_A_A15" />
          <FileRuleRef RuleID="ID_ALLOW_A_A16" />
          <FileRuleRef RuleID="ID_ALLOW_A_A17" />
          <FileRuleRef RuleID="ID_ALLOW_A_A18" />
          <FileRuleRef RuleID="ID_ALLOW_A_A19" />
          <FileRuleRef RuleID="ID_ALLOW_A_A1A" />
          <FileRuleRef RuleID="ID_ALLOW_A_A1B" />
          <FileRuleRef RuleID="ID_ALLOW_A_A1C" />
          <FileRuleRef RuleID="ID_ALLOW_A_A1D" />
          <FileRuleRef RuleID="ID_ALLOW_A_A1E" />
          <FileRuleRef RuleID="ID_ALLOW_A_A1F" />
          <FileRuleRef RuleID="ID_ALLOW_A_A20" />
          <FileRuleRef RuleID="ID_ALLOW_A_A21" />
          <FileRuleRef RuleID="ID_ALLOW_A_A22" />
          <FileRuleRef RuleID="ID_ALLOW_A_A23" />
          <FileRuleRef RuleID="ID_ALLOW_A_A24" />
          <FileRuleRef RuleID="ID_ALLOW_A_A25" />
          <FileRuleRef RuleID="ID_ALLOW_A_A26" />
          <FileRuleRef RuleID="ID_ALLOW_A_A27" />
          <FileRuleRef RuleID="ID_ALLOW_A_A28" />
          <FileRuleRef RuleID="ID_ALLOW_A_A29" />
          <FileRuleRef RuleID="ID_ALLOW_A_A2A" />
          <FileRuleRef RuleID="ID_ALLOW_A_A2B" />
          <FileRuleRef RuleID="ID_ALLOW_A_A2C" />
          <FileRuleRef RuleID="ID_ALLOW_A_A2D" />
          <FileRuleRef RuleID="ID_ALLOW_A_A2E" />
          <FileRuleRef RuleID="ID_ALLOW_A_A2F" />
          <FileRuleRef RuleID="ID_ALLOW_A_A30" />
          <FileRuleRef RuleID="ID_ALLOW_A_A31" />
          <FileRuleRef RuleID="ID_ALLOW_A_A32" />
          <FileRuleRef RuleID="ID_ALLOW_A_A33" />
          <FileRuleRef RuleID="ID_ALLOW_A_A34" />
          <FileRuleRef RuleID="ID_ALLOW_A_A35" />
          <FileRuleRef RuleID="ID_ALLOW_A_A36" />
          <FileRuleRef RuleID="ID_ALLOW_A_A37" />
          <FileRuleRef RuleID="ID_ALLOW_A_A38" />
          <FileRuleRef RuleID="ID_ALLOW_A_A39" />
          <FileRuleRef RuleID="ID_ALLOW_A_A3A" />
          <FileRuleRef RuleID="ID_ALLOW_A_A3B" />
          <FileRuleRef RuleID="ID_ALLOW_A_A3C" />
          <FileRuleRef RuleID="ID_ALLOW_A_A3D" />
          <FileRuleRef RuleID="ID_ALLOW_A_A3E" />
          <FileRuleRef RuleID="ID_ALLOW_A_A3F" />
          <FileRuleRef RuleID="ID_ALLOW_A_A40" />
          <FileRuleRef RuleID="ID_ALLOW_A_A41" />
          <FileRuleRef RuleID="ID_ALLOW_A_A42" />
          <FileRuleRef RuleID="ID_ALLOW_A_A43" />
          <FileRuleRef RuleID="ID_ALLOW_A_A44" />
          <FileRuleRef RuleID="ID_ALLOW_A_A45" />
          <FileRuleRef RuleID="ID_ALLOW_A_A46" />
          <FileRuleRef RuleID="ID_ALLOW_A_A47" />
          <FileRuleRef RuleID="ID_ALLOW_A_A48" />
          <FileRuleRef RuleID="ID_ALLOW_A_A49" />
          <FileRuleRef RuleID="ID_ALLOW_A_A4A" />
          <FileRuleRef RuleID="ID_ALLOW_A_A4B" />
          <FileRuleRef RuleID="ID_ALLOW_A_A4C" />
          <FileRuleRef RuleID="ID_ALLOW_A_A4D" />
          <FileRuleRef RuleID="ID_ALLOW_A_A4E" />
          <FileRuleRef RuleID="ID_ALLOW_A_A4F" />
          <FileRuleRef RuleID="ID_ALLOW_A_A50" />
          <FileRuleRef RuleID="ID_ALLOW_A_A51" />
          <FileRuleRef RuleID="ID_ALLOW_A_A52" />
          <FileRuleRef RuleID="ID_ALLOW_A_A53" />
          <FileRuleRef RuleID="ID_ALLOW_A_A54" />
          <FileRuleRef RuleID="ID_ALLOW_A_A55" />
          <FileRuleRef RuleID="ID_ALLOW_A_A56" />
          <FileRuleRef RuleID="ID_ALLOW_A_A57" />
          <FileRuleRef RuleID="ID_ALLOW_A_A58" />
          <FileRuleRef RuleID="ID_ALLOW_A_A59" />
          <FileRuleRef RuleID="ID_ALLOW_A_A5A" />
          <FileRuleRef RuleID="ID_ALLOW_A_A5B" />
          <FileRuleRef RuleID="ID_ALLOW_A_A5C" />
          <FileRuleRef RuleID="ID_ALLOW_A_A5D" />
          <FileRuleRef RuleID="ID_ALLOW_A_A5E" />
          <FileRuleRef RuleID="ID_ALLOW_A_A5F" />
          <FileRuleRef RuleID="ID_ALLOW_A_A60" />
          <FileRuleRef RuleID="ID_ALLOW_A_A61" />
          <FileRuleRef RuleID="ID_ALLOW_A_A62" />
          <FileRuleRef RuleID="ID_ALLOW_A_A63" />
          <FileRuleRef RuleID="ID_ALLOW_A_A64" />
          <FileRuleRef RuleID="ID_ALLOW_A_A65" />
          <FileRuleRef RuleID="ID_ALLOW_A_A66" />
          <FileRuleRef RuleID="ID_ALLOW_A_A67" />
          <FileRuleRef RuleID="ID_ALLOW_A_A68" />
          <FileRuleRef RuleID="ID_ALLOW_A_A69" />
          <FileRuleRef RuleID="ID_ALLOW_A_A6A" />
          <FileRuleRef RuleID="ID_ALLOW_A_A6B" />
          <FileRuleRef RuleID="ID_ALLOW_A_A6C" />
          <FileRuleRef RuleID="ID_ALLOW_A_A6D" />
          <FileRuleRef RuleID="ID_ALLOW_A_A6E" />
          <FileRuleRef RuleID="ID_ALLOW_A_A6F" />
          <FileRuleRef RuleID="ID_ALLOW_A_A70" />
          <FileRuleRef RuleID="ID_ALLOW_A_A71" />
          <FileRuleRef RuleID="ID_ALLOW_A_A72" />
          <FileRuleRef RuleID="ID_ALLOW_A_A73" />
          <FileRuleRef RuleID="ID_ALLOW_A_A74" />
          <FileRuleRef RuleID="ID_ALLOW_A_A75" />
          <FileRuleRef RuleID="ID_ALLOW_A_A76" />
          <FileRuleRef RuleID="ID_ALLOW_A_A77" />
          <FileRuleRef RuleID="ID_ALLOW_A_A78" />
          <FileRuleRef RuleID="ID_ALLOW_A_A79" />
          <FileRuleRef RuleID="ID_ALLOW_A_A7A" />
          <FileRuleRef RuleID="ID_ALLOW_A_A7B" />
          <FileRuleRef RuleID="ID_ALLOW_A_A7C" />
          <FileRuleRef RuleID="ID_ALLOW_A_A7D" />
          <FileRuleRef RuleID="ID_ALLOW_A_A7E" />
          <FileRuleRef RuleID="ID_ALLOW_A_A7F" />
          <FileRuleRef RuleID="ID_ALLOW_A_A80" />
          <FileRuleRef RuleID="ID_ALLOW_A_A81" />
          <FileRuleRef RuleID="ID_ALLOW_A_A82" />
          <FileRuleRef RuleID="ID_ALLOW_A_A83" />
          <FileRuleRef RuleID="ID_ALLOW_A_A84" />
          <FileRuleRef RuleID="ID_ALLOW_A_A85" />
          <FileRuleRef RuleID="ID_ALLOW_A_A86" />
          <FileRuleRef RuleID="ID_ALLOW_A_A87" />
          <FileRuleRef RuleID="ID_ALLOW_A_A88" />
          <FileRuleRef RuleID="ID_ALLOW_A_A89" />
          <FileRuleRef RuleID="ID_ALLOW_A_A8A" />
          <FileRuleRef RuleID="ID_ALLOW_A_A8B" />
          <FileRuleRef RuleID="ID_ALLOW_A_A8C" />
          <FileRuleRef RuleID="ID_ALLOW_A_A8D" />
          <FileRuleRef RuleID="ID_ALLOW_A_A8E" />
          <FileRuleRef RuleID="ID_ALLOW_A_A8F" />
          <FileRuleRef RuleID="ID_ALLOW_A_A90" />
          <FileRuleRef RuleID="ID_ALLOW_A_A91" />
          <FileRuleRef RuleID="ID_ALLOW_A_A92" />
          <FileRuleRef RuleID="ID_ALLOW_A_A93" />
          <FileRuleRef RuleID="ID_ALLOW_A_A94" />
          <FileRuleRef RuleID="ID_ALLOW_A_A95" />
          <FileRuleRef RuleID="ID_ALLOW_A_A96" />
          <FileRuleRef RuleID="ID_ALLOW_A_A97" />
          <FileRuleRef RuleID="ID_ALLOW_A_A98" />
          <FileRuleRef RuleID="ID_ALLOW_A_A99" />
          <FileRuleRef RuleID="ID_ALLOW_A_A9A" />
          <FileRuleRef RuleID="ID_ALLOW_A_A9B" />
          <FileRuleRef RuleID="ID_ALLOW_A_A9C" />
          <FileRuleRef RuleID="ID_ALLOW_A_A9D" />
          <FileRuleRef RuleID="ID_ALLOW_A_A9E" />
          <FileRuleRef RuleID="ID_ALLOW_A_A9F" />
          <FileRuleRef RuleID="ID_ALLOW_A_AA0" />
          <FileRuleRef RuleID="ID_ALLOW_A_AA1" />
          <FileRuleRef RuleID="ID_ALLOW_A_AA2" />
          <FileRuleRef RuleID="ID_ALLOW_A_AA3" />
          <FileRuleRef RuleID="ID_ALLOW_A_AA4" />
          <FileRuleRef RuleID="ID_ALLOW_A_AA5" />
          <FileRuleRef RuleID="ID_ALLOW_A_AA6" />
          <FileRuleRef RuleID="ID_ALLOW_A_AA7" />
          <FileRuleRef RuleID="ID_ALLOW_A_AA8" />
          <FileRuleRef RuleID="ID_ALLOW_A_AA9" />
          <FileRuleRef RuleID="ID_ALLOW_A_AAA" />
          <FileRuleRef RuleID="ID_ALLOW_A_AAB" />
          <FileRuleRef RuleID="ID_ALLOW_A_AAC" />
          <FileRuleRef RuleID="ID_ALLOW_A_AAD" />
          <FileRuleRef RuleID="ID_ALLOW_A_AAE" />
          <FileRuleRef RuleID="ID_ALLOW_A_AAF" />
          <FileRuleRef RuleID="ID_ALLOW_A_AB0" />
          <FileRuleRef RuleID="ID_ALLOW_A_AB1" />
          <FileRuleRef RuleID="ID_ALLOW_A_AB2" />
          <FileRuleRef RuleID="ID_ALLOW_A_AB3" />
          <FileRuleRef RuleID="ID_ALLOW_A_AB4" />
          <FileRuleRef RuleID="ID_ALLOW_A_AB5" />
          <FileRuleRef RuleID="ID_ALLOW_A_AB6" />
          <FileRuleRef RuleID="ID_ALLOW_A_AB7" />
          <FileRuleRef RuleID="ID_ALLOW_A_AB8" />
          <FileRuleRef RuleID="ID_ALLOW_A_AB9" />
          <FileRuleRef RuleID="ID_ALLOW_A_ABA" />
          <FileRuleRef RuleID="ID_ALLOW_A_ABB" />
          <FileRuleRef RuleID="ID_ALLOW_A_ABC" />
          <FileRuleRef RuleID="ID_ALLOW_A_ABD" />
          <FileRuleRef RuleID="ID_ALLOW_A_ABE" />
          <FileRuleRef RuleID="ID_ALLOW_A_ABF" />
          <FileRuleRef RuleID="ID_ALLOW_A_AC0" />
          <FileRuleRef RuleID="ID_ALLOW_A_AC1" />
          <FileRuleRef RuleID="ID_ALLOW_A_AC2" />
          <FileRuleRef RuleID="ID_ALLOW_A_AC3" />
          <FileRuleRef RuleID="ID_ALLOW_A_AC4" />
          <FileRuleRef RuleID="ID_ALLOW_A_AC5" />
          <FileRuleRef RuleID="ID_ALLOW_A_AC6" />
          <FileRuleRef RuleID="ID_ALLOW_A_AC7" />
          <FileRuleRef RuleID="ID_ALLOW_A_AC8" />
          <FileRuleRef RuleID="ID_ALLOW_A_AC9" />
          <FileRuleRef RuleID="ID_ALLOW_A_ACA" />
          <FileRuleRef RuleID="ID_ALLOW_A_ACB" />
          <FileRuleRef RuleID="ID_ALLOW_A_ACC" />
          <FileRuleRef RuleID="ID_ALLOW_A_ACD" />
          <FileRuleRef RuleID="ID_ALLOW_A_ACE" />
          <FileRuleRef RuleID="ID_ALLOW_A_ACF" />
          <FileRuleRef RuleID="ID_ALLOW_A_AD0" />
          <FileRuleRef RuleID="ID_ALLOW_A_AD1" />
          <FileRuleRef RuleID="ID_ALLOW_A_AD2" />
          <FileRuleRef RuleID="ID_ALLOW_A_AD3" />
          <FileRuleRef RuleID="ID_ALLOW_A_AD4" />
          <FileRuleRef RuleID="ID_ALLOW_A_AD5" />
          <FileRuleRef RuleID="ID_ALLOW_A_AD6" />
          <FileRuleRef RuleID="ID_ALLOW_A_AD7" />
          <FileRuleRef RuleID="ID_ALLOW_A_AD8" />
          <FileRuleRef RuleID="ID_ALLOW_A_AD9" />
          <FileRuleRef RuleID="ID_ALLOW_A_ADA" />
          <FileRuleRef RuleID="ID_ALLOW_A_ADB" />
          <FileRuleRef RuleID="ID_ALLOW_A_ADC" />
          <FileRuleRef RuleID="ID_ALLOW_A_ADD" />
          <FileRuleRef RuleID="ID_ALLOW_A_ADE" />
          <FileRuleRef RuleID="ID_ALLOW_A_ADF" />
          <FileRuleRef RuleID="ID_ALLOW_A_AE0" />
          <FileRuleRef RuleID="ID_ALLOW_A_AE1" />
          <FileRuleRef RuleID="ID_ALLOW_A_AE2" />
          <FileRuleRef RuleID="ID_ALLOW_A_AE3" />
          <FileRuleRef RuleID="ID_ALLOW_A_AE4" />
          <FileRuleRef RuleID="ID_ALLOW_A_AE5" />
          <FileRuleRef RuleID="ID_ALLOW_A_AE6" />
          <FileRuleRef RuleID="ID_ALLOW_A_AE7" />
          <FileRuleRef RuleID="ID_ALLOW_A_AE8" />
          <FileRuleRef RuleID="ID_ALLOW_A_AE9" />
          <FileRuleRef RuleID="ID_ALLOW_A_AEA" />
          <FileRuleRef RuleID="ID_ALLOW_A_AEB" />
          <FileRuleRef RuleID="ID_ALLOW_A_AEC" />
          <FileRuleRef RuleID="ID_ALLOW_A_AED" />
          <FileRuleRef RuleID="ID_ALLOW_A_AEE" />
          <FileRuleRef RuleID="ID_ALLOW_A_AEF" />
          <FileRuleRef RuleID="ID_ALLOW_A_AF0" />
          <FileRuleRef RuleID="ID_ALLOW_A_AF1" />
          <FileRuleRef RuleID="ID_ALLOW_A_AF2" />
          <FileRuleRef RuleID="ID_ALLOW_A_AF3" />
          <FileRuleRef RuleID="ID_ALLOW_A_AF4" />
          <FileRuleRef RuleID="ID_ALLOW_A_AF5" />
          <FileRuleRef RuleID="ID_ALLOW_A_AF6" />
          <FileRuleRef RuleID="ID_ALLOW_A_AF7" />
          <FileRuleRef RuleID="ID_ALLOW_A_AF8" />
          <FileRuleRef RuleID="ID_ALLOW_A_AF9" />
          <FileRuleRef RuleID="ID_ALLOW_A_AFA" />
          <FileRuleRef RuleID="ID_ALLOW_A_AFB" />
          <FileRuleRef RuleID="ID_ALLOW_A_AFC" />
          <FileRuleRef RuleID="ID_ALLOW_A_AFD" />
          <FileRuleRef RuleID="ID_ALLOW_A_AFE" />
          <FileRuleRef RuleID="ID_ALLOW_A_AFF" />
          <FileRuleRef RuleID="ID_ALLOW_A_B00" />
          <FileRuleRef RuleID="ID_ALLOW_A_B01" />
          <FileRuleRef RuleID="ID_ALLOW_A_B02" />
          <FileRuleRef RuleID="ID_ALLOW_A_B03" />
          <FileRuleRef RuleID="ID_ALLOW_A_B04" />
          <FileRuleRef RuleID="ID_ALLOW_A_B05" />
          <FileRuleRef RuleID="ID_ALLOW_A_B06" />
          <FileRuleRef RuleID="ID_ALLOW_A_B07" />
          <FileRuleRef RuleID="ID_ALLOW_A_B08" />
          <FileRuleRef RuleID="ID_ALLOW_A_B09" />
          <FileRuleRef RuleID="ID_ALLOW_A_B0A" />
          <FileRuleRef RuleID="ID_ALLOW_A_B0B" />
          <FileRuleRef RuleID="ID_ALLOW_A_B0C" />
          <FileRuleRef RuleID="ID_ALLOW_A_B0D" />
          <FileRuleRef RuleID="ID_ALLOW_A_B0E" />
          <FileRuleRef RuleID="ID_ALLOW_A_B0F" />
          <FileRuleRef RuleID="ID_ALLOW_A_B10" />
          <FileRuleRef RuleID="ID_ALLOW_A_B11" />
          <FileRuleRef RuleID="ID_ALLOW_A_B12" />
          <FileRuleRef RuleID="ID_ALLOW_A_B13" />
          <FileRuleRef RuleID="ID_ALLOW_A_B14" />
          <FileRuleRef RuleID="ID_ALLOW_A_B15" />
          <FileRuleRef RuleID="ID_ALLOW_A_B16" />
          <FileRuleRef RuleID="ID_ALLOW_A_B17" />
          <FileRuleRef RuleID="ID_ALLOW_A_B18" />
          <FileRuleRef RuleID="ID_ALLOW_A_B19" />
          <FileRuleRef RuleID="ID_ALLOW_A_B1A" />
          <FileRuleRef RuleID="ID_ALLOW_A_B1B" />
          <FileRuleRef RuleID="ID_ALLOW_A_B1C" />
          <FileRuleRef RuleID="ID_ALLOW_A_B1D" />
          <FileRuleRef RuleID="ID_ALLOW_A_B1E" />
          <FileRuleRef RuleID="ID_ALLOW_A_B1F" />
          <FileRuleRef RuleID="ID_ALLOW_A_B20" />
          <FileRuleRef RuleID="ID_ALLOW_A_B21" />
          <FileRuleRef RuleID="ID_ALLOW_A_B22" />
          <FileRuleRef RuleID="ID_ALLOW_A_B23" />
          <FileRuleRef RuleID="ID_ALLOW_A_B24" />
          <FileRuleRef RuleID="ID_ALLOW_A_B25" />
          <FileRuleRef RuleID="ID_ALLOW_A_B26" />
          <FileRuleRef RuleID="ID_ALLOW_A_B27" />
          <FileRuleRef RuleID="ID_ALLOW_A_B28" />
          <FileRuleRef RuleID="ID_ALLOW_A_B29" />
          <FileRuleRef RuleID="ID_ALLOW_A_B2A" />
          <FileRuleRef RuleID="ID_ALLOW_A_B2B" />
          <FileRuleRef RuleID="ID_ALLOW_A_B2C" />
          <FileRuleRef RuleID="ID_ALLOW_A_B2D" />
          <FileRuleRef RuleID="ID_ALLOW_A_B2E" />
          <FileRuleRef RuleID="ID_ALLOW_A_B2F" />
          <FileRuleRef RuleID="ID_ALLOW_A_B30" />
          <FileRuleRef RuleID="ID_ALLOW_A_B31" />
          <FileRuleRef RuleID="ID_ALLOW_A_B32" />
          <FileRuleRef RuleID="ID_ALLOW_A_B33" />
          <FileRuleRef RuleID="ID_ALLOW_A_B34" />
          <FileRuleRef RuleID="ID_ALLOW_A_B35" />
          <FileRuleRef RuleID="ID_ALLOW_A_B36" />
          <FileRuleRef RuleID="ID_ALLOW_A_B37" />
          <FileRuleRef RuleID="ID_ALLOW_A_B38" />
          <FileRuleRef RuleID="ID_ALLOW_A_B39" />
          <FileRuleRef RuleID="ID_ALLOW_A_B3A" />
          <FileRuleRef RuleID="ID_ALLOW_A_B3B" />
          <FileRuleRef RuleID="ID_ALLOW_A_B3C" />
          <FileRuleRef RuleID="ID_ALLOW_A_B3D" />
          <FileRuleRef RuleID="ID_ALLOW_A_B3E" />
          <FileRuleRef RuleID="ID_ALLOW_A_B3F" />
          <FileRuleRef RuleID="ID_ALLOW_A_B40" />
          <FileRuleRef RuleID="ID_ALLOW_A_B41" />
          <FileRuleRef RuleID="ID_ALLOW_A_B42" />
          <FileRuleRef RuleID="ID_ALLOW_A_B43" />
          <FileRuleRef RuleID="ID_ALLOW_A_B44" />
          <FileRuleRef RuleID="ID_ALLOW_A_B45" />
          <FileRuleRef RuleID="ID_ALLOW_A_B46" />
          <FileRuleRef RuleID="ID_ALLOW_A_B47" />
          <FileRuleRef RuleID="ID_ALLOW_A_B48" />
          <FileRuleRef RuleID="ID_ALLOW_A_B49" />
          <FileRuleRef RuleID="ID_ALLOW_A_B4A" />
          <FileRuleRef RuleID="ID_ALLOW_A_B4B" />
          <FileRuleRef RuleID="ID_ALLOW_A_B4C" />
          <FileRuleRef RuleID="ID_ALLOW_A_B4D" />
          <FileRuleRef RuleID="ID_ALLOW_A_B4E" />
          <FileRuleRef RuleID="ID_ALLOW_A_B4F" />
          <FileRuleRef RuleID="ID_ALLOW_A_B50" />
          <FileRuleRef RuleID="ID_ALLOW_A_B51" />
          <FileRuleRef RuleID="ID_ALLOW_A_B56" />
          <FileRuleRef RuleID="ID_ALLOW_A_B57" />
          <FileRuleRef RuleID="ID_ALLOW_A_B58" />
          <FileRuleRef RuleID="ID_ALLOW_A_B59" />
          <FileRuleRef RuleID="ID_ALLOW_A_B5A" />
          <FileRuleRef RuleID="ID_ALLOW_A_B5B" />
          <FileRuleRef RuleID="ID_ALLOW_A_B5C" />
          <FileRuleRef RuleID="ID_ALLOW_A_B5D" />
          <FileRuleRef RuleID="ID_ALLOW_A_B62" />
          <FileRuleRef RuleID="ID_ALLOW_A_B63" />
          <FileRuleRef RuleID="ID_ALLOW_A_B64" />
          <FileRuleRef RuleID="ID_ALLOW_A_B65" />
          <FileRuleRef RuleID="ID_ALLOW_A_B66" />
          <FileRuleRef RuleID="ID_ALLOW_A_B67" />
          <FileRuleRef RuleID="ID_ALLOW_A_B68" />
          <FileRuleRef RuleID="ID_ALLOW_A_B69" />
          <FileRuleRef RuleID="ID_ALLOW_A_B6A" />
          <FileRuleRef RuleID="ID_ALLOW_A_B6B" />
          <FileRuleRef RuleID="ID_ALLOW_A_B6C" />
          <FileRuleRef RuleID="ID_ALLOW_A_B6D" />
          <FileRuleRef RuleID="ID_ALLOW_A_B6E" />
          <FileRuleRef RuleID="ID_ALLOW_A_B6F" />
          <FileRuleRef RuleID="ID_ALLOW_A_B70" />
          <FileRuleRef RuleID="ID_ALLOW_A_B71" />
          <FileRuleRef RuleID="ID_ALLOW_A_B72" />
          <FileRuleRef RuleID="ID_ALLOW_A_B73" />
          <FileRuleRef RuleID="ID_ALLOW_A_B74" />
          <FileRuleRef RuleID="ID_ALLOW_A_B75" />
          <FileRuleRef RuleID="ID_ALLOW_A_B76" />
          <FileRuleRef RuleID="ID_ALLOW_A_B77" />
          <FileRuleRef RuleID="ID_ALLOW_A_B78" />
          <FileRuleRef RuleID="ID_ALLOW_A_B79" />
          <FileRuleRef RuleID="ID_ALLOW_A_B7A" />
          <FileRuleRef RuleID="ID_ALLOW_A_B7B" />
          <FileRuleRef RuleID="ID_ALLOW_A_B7C" />
          <FileRuleRef RuleID="ID_ALLOW_A_B7D" />
          <FileRuleRef RuleID="ID_ALLOW_A_B7E" />
          <FileRuleRef RuleID="ID_ALLOW_A_B7F" />
          <FileRuleRef RuleID="ID_ALLOW_A_B80" />
          <FileRuleRef RuleID="ID_ALLOW_A_B81" />
          <FileRuleRef RuleID="ID_ALLOW_A_B82" />
          <FileRuleRef RuleID="ID_ALLOW_A_B83" />
          <FileRuleRef RuleID="ID_ALLOW_A_B84" />
          <FileRuleRef RuleID="ID_ALLOW_A_B85" />
          <FileRuleRef RuleID="ID_ALLOW_A_B86" />
          <FileRuleRef RuleID="ID_ALLOW_A_B87" />
          <FileRuleRef RuleID="ID_ALLOW_A_B88" />
          <FileRuleRef RuleID="ID_ALLOW_A_B89" />
          <FileRuleRef RuleID="ID_ALLOW_A_B8A" />
          <FileRuleRef RuleID="ID_ALLOW_A_B8B" />
          <FileRuleRef RuleID="ID_ALLOW_A_B8C" />
          <FileRuleRef RuleID="ID_ALLOW_A_B8D" />
          <FileRuleRef RuleID="ID_ALLOW_A_B8E" />
          <FileRuleRef RuleID="ID_ALLOW_A_B8F" />
          <FileRuleRef RuleID="ID_ALLOW_A_B90" />
          <FileRuleRef RuleID="ID_ALLOW_A_B91" />
          <FileRuleRef RuleID="ID_ALLOW_A_B92" />
          <FileRuleRef RuleID="ID_ALLOW_A_B93" />
          <FileRuleRef RuleID="ID_ALLOW_A_B94" />
          <FileRuleRef RuleID="ID_ALLOW_A_B95" />
          <FileRuleRef RuleID="ID_ALLOW_A_B96" />
          <FileRuleRef RuleID="ID_ALLOW_A_B97" />
          <FileRuleRef RuleID="ID_ALLOW_A_B98" />
          <FileRuleRef RuleID="ID_ALLOW_A_B99" />
          <FileRuleRef RuleID="ID_ALLOW_A_B9A" />
          <FileRuleRef RuleID="ID_ALLOW_A_B9B" />
          <FileRuleRef RuleID="ID_ALLOW_A_B9C" />
          <FileRuleRef RuleID="ID_ALLOW_A_B9D" />
          <FileRuleRef RuleID="ID_ALLOW_A_B9E" />
          <FileRuleRef RuleID="ID_ALLOW_A_B9F" />
          <FileRuleRef RuleID="ID_ALLOW_A_BA0" />
          <FileRuleRef RuleID="ID_ALLOW_A_BA1" />
          <FileRuleRef RuleID="ID_ALLOW_A_BA2" />
          <FileRuleRef RuleID="ID_ALLOW_A_BA3" />
          <FileRuleRef RuleID="ID_ALLOW_A_BA4" />
          <FileRuleRef RuleID="ID_ALLOW_A_BA5" />
          <FileRuleRef RuleID="ID_ALLOW_A_BA6" />
          <FileRuleRef RuleID="ID_ALLOW_A_BA7" />
          <FileRuleRef RuleID="ID_ALLOW_A_BA8" />
          <FileRuleRef RuleID="ID_ALLOW_A_BA9" />
          <FileRuleRef RuleID="ID_ALLOW_A_BAA" />
          <FileRuleRef RuleID="ID_ALLOW_A_BAB" />
          <FileRuleRef RuleID="ID_ALLOW_A_BAC" />
          <FileRuleRef RuleID="ID_ALLOW_A_BAD" />
          <FileRuleRef RuleID="ID_ALLOW_A_BAE" />
          <FileRuleRef RuleID="ID_ALLOW_A_BAF" />
          <FileRuleRef RuleID="ID_ALLOW_A_BB0" />
          <FileRuleRef RuleID="ID_ALLOW_A_BB1" />
          <FileRuleRef RuleID="ID_ALLOW_A_BB2" />
          <FileRuleRef RuleID="ID_ALLOW_A_BB3" />
          <FileRuleRef RuleID="ID_ALLOW_A_BB4" />
          <FileRuleRef RuleID="ID_ALLOW_A_BB5" />
          <FileRuleRef RuleID="ID_ALLOW_A_BB6" />
          <FileRuleRef RuleID="ID_ALLOW_A_BB7" />
          <FileRuleRef RuleID="ID_ALLOW_A_BB8" />
          <FileRuleRef RuleID="ID_ALLOW_A_BB9" />
          <FileRuleRef RuleID="ID_ALLOW_A_BBA" />
          <FileRuleRef RuleID="ID_ALLOW_A_BBB" />
          <FileRuleRef RuleID="ID_ALLOW_A_BBC" />
          <FileRuleRef RuleID="ID_ALLOW_A_BBD" />
          <FileRuleRef RuleID="ID_ALLOW_A_BBE" />
          <FileRuleRef RuleID="ID_ALLOW_A_BBF" />
          <FileRuleRef RuleID="ID_ALLOW_A_BC0" />
          <FileRuleRef RuleID="ID_ALLOW_A_BC1" />
          <FileRuleRef RuleID="ID_ALLOW_A_BC2" />
          <FileRuleRef RuleID="ID_ALLOW_A_BC3" />
          <FileRuleRef RuleID="ID_ALLOW_A_BC4" />
          <FileRuleRef RuleID="ID_ALLOW_A_BC5" />
          <FileRuleRef RuleID="ID_ALLOW_A_BC6" />
          <FileRuleRef RuleID="ID_ALLOW_A_BC7" />
          <FileRuleRef RuleID="ID_ALLOW_A_BC8" />
          <FileRuleRef RuleID="ID_ALLOW_A_BC9" />
          <FileRuleRef RuleID="ID_ALLOW_A_BCA" />
          <FileRuleRef RuleID="ID_ALLOW_A_BCB" />
          <FileRuleRef RuleID="ID_ALLOW_A_BCC" />
          <FileRuleRef RuleID="ID_ALLOW_A_BCD" />
          <FileRuleRef RuleID="ID_ALLOW_A_BCE" />
          <FileRuleRef RuleID="ID_ALLOW_A_BCF" />
          <FileRuleRef RuleID="ID_ALLOW_A_BD0" />
          <FileRuleRef RuleID="ID_ALLOW_A_BD1" />
          <FileRuleRef RuleID="ID_ALLOW_A_BD2" />
          <FileRuleRef RuleID="ID_ALLOW_A_BD3" />
          <FileRuleRef RuleID="ID_ALLOW_A_BD4" />
          <FileRuleRef RuleID="ID_ALLOW_A_BD5" />
          <FileRuleRef RuleID="ID_ALLOW_A_BD6" />
          <FileRuleRef RuleID="ID_ALLOW_A_BD7" />
          <FileRuleRef RuleID="ID_ALLOW_A_BD8" />
          <FileRuleRef RuleID="ID_ALLOW_A_BD9" />
          <FileRuleRef RuleID="ID_ALLOW_A_BDA" />
          <FileRuleRef RuleID="ID_ALLOW_A_BDB" />
          <FileRuleRef RuleID="ID_ALLOW_A_BDC" />
          <FileRuleRef RuleID="ID_ALLOW_A_BDD" />
          <FileRuleRef RuleID="ID_ALLOW_A_BDE" />
          <FileRuleRef RuleID="ID_ALLOW_A_BDF" />
          <FileRuleRef RuleID="ID_ALLOW_A_BE0" />
          <FileRuleRef RuleID="ID_ALLOW_A_BE1" />
          <FileRuleRef RuleID="ID_ALLOW_A_BE2" />
          <FileRuleRef RuleID="ID_ALLOW_A_BE3" />
          <FileRuleRef RuleID="ID_ALLOW_A_BE4" />
          <FileRuleRef RuleID="ID_ALLOW_A_BE5" />
          <FileRuleRef RuleID="ID_ALLOW_A_BE6" />
          <FileRuleRef RuleID="ID_ALLOW_A_BE7" />
          <FileRuleRef RuleID="ID_ALLOW_A_BE8" />
          <FileRuleRef RuleID="ID_ALLOW_A_BE9" />
          <FileRuleRef RuleID="ID_ALLOW_A_BEA" />
          <FileRuleRef RuleID="ID_ALLOW_A_BEB" />
          <FileRuleRef RuleID="ID_ALLOW_A_BEC" />
          <FileRuleRef RuleID="ID_ALLOW_A_BED" />
          <FileRuleRef RuleID="ID_ALLOW_A_BEE" />
          <FileRuleRef RuleID="ID_ALLOW_A_BEF" />
          <FileRuleRef RuleID="ID_ALLOW_A_BF0" />
          <FileRuleRef RuleID="ID_ALLOW_A_BF1" />
          <FileRuleRef RuleID="ID_ALLOW_A_BF2" />
          <FileRuleRef RuleID="ID_ALLOW_A_BF3" />
          <FileRuleRef RuleID="ID_ALLOW_A_BF4" />
          <FileRuleRef RuleID="ID_ALLOW_A_BF5" />
          <FileRuleRef RuleID="ID_ALLOW_A_BF6" />
          <FileRuleRef RuleID="ID_ALLOW_A_BF7" />
          <FileRuleRef RuleID="ID_ALLOW_A_BF8" />
          <FileRuleRef RuleID="ID_ALLOW_A_BF9" />
          <FileRuleRef RuleID="ID_ALLOW_A_BFA" />
          <FileRuleRef RuleID="ID_ALLOW_A_BFB" />
          <FileRuleRef RuleID="ID_ALLOW_A_BFC" />
          <FileRuleRef RuleID="ID_ALLOW_A_BFD" />
          <FileRuleRef RuleID="ID_ALLOW_A_BFE" />
          <FileRuleRef RuleID="ID_ALLOW_A_BFF" />
          <FileRuleRef RuleID="ID_ALLOW_A_C00" />
          <FileRuleRef RuleID="ID_ALLOW_A_C01" />
          <FileRuleRef RuleID="ID_ALLOW_A_C02" />
          <FileRuleRef RuleID="ID_ALLOW_A_C03" />
          <FileRuleRef RuleID="ID_ALLOW_A_C04" />
          <FileRuleRef RuleID="ID_ALLOW_A_C05" />
          <FileRuleRef RuleID="ID_ALLOW_A_C06" />
          <FileRuleRef RuleID="ID_ALLOW_A_C07" />
          <FileRuleRef RuleID="ID_ALLOW_A_C08" />
          <FileRuleRef RuleID="ID_ALLOW_A_C09" />
          <FileRuleRef RuleID="ID_ALLOW_A_C0A" />
          <FileRuleRef RuleID="ID_ALLOW_A_C0B" />
          <FileRuleRef RuleID="ID_ALLOW_A_C0C" />
          <FileRuleRef RuleID="ID_ALLOW_A_C0D" />
          <FileRuleRef RuleID="ID_ALLOW_A_C0E" />
          <FileRuleRef RuleID="ID_ALLOW_A_C0F" />
          <FileRuleRef RuleID="ID_ALLOW_A_C10" />
          <FileRuleRef RuleID="ID_ALLOW_A_C11" />
          <FileRuleRef RuleID="ID_ALLOW_A_C12" />
          <FileRuleRef RuleID="ID_ALLOW_A_C13" />
          <FileRuleRef RuleID="ID_ALLOW_A_C14" />
          <FileRuleRef RuleID="ID_ALLOW_A_C15" />
          <FileRuleRef RuleID="ID_ALLOW_A_C16" />
          <FileRuleRef RuleID="ID_ALLOW_A_C17" />
          <FileRuleRef RuleID="ID_ALLOW_A_C18" />
          <FileRuleRef RuleID="ID_ALLOW_A_C19" />
          <FileRuleRef RuleID="ID_ALLOW_A_C1A" />
          <FileRuleRef RuleID="ID_ALLOW_A_C1B" />
          <FileRuleRef RuleID="ID_ALLOW_A_C1C" />
          <FileRuleRef RuleID="ID_ALLOW_A_C1D" />
          <FileRuleRef RuleID="ID_ALLOW_A_C1E" />
          <FileRuleRef RuleID="ID_ALLOW_A_C1F" />
          <FileRuleRef RuleID="ID_ALLOW_A_C20" />
          <FileRuleRef RuleID="ID_ALLOW_A_C21" />
          <FileRuleRef RuleID="ID_ALLOW_A_C22" />
          <FileRuleRef RuleID="ID_ALLOW_A_C23" />
          <FileRuleRef RuleID="ID_ALLOW_A_C24" />
          <FileRuleRef RuleID="ID_ALLOW_A_C25" />
          <FileRuleRef RuleID="ID_ALLOW_A_C26" />
          <FileRuleRef RuleID="ID_ALLOW_A_C27" />
          <FileRuleRef RuleID="ID_ALLOW_A_C28" />
          <FileRuleRef RuleID="ID_ALLOW_A_C29" />
          <FileRuleRef RuleID="ID_ALLOW_A_C2A" />
          <FileRuleRef RuleID="ID_ALLOW_A_C2B" />
          <FileRuleRef RuleID="ID_ALLOW_A_C2C" />
          <FileRuleRef RuleID="ID_ALLOW_A_C2D" />
          <FileRuleRef RuleID="ID_ALLOW_A_C2E" />
          <FileRuleRef RuleID="ID_ALLOW_A_C2F" />
          <FileRuleRef RuleID="ID_ALLOW_A_C30" />
          <FileRuleRef RuleID="ID_ALLOW_A_C31" />
          <FileRuleRef RuleID="ID_ALLOW_A_C32" />
          <FileRuleRef RuleID="ID_ALLOW_A_C33" />
          <FileRuleRef RuleID="ID_ALLOW_A_C34" />
          <FileRuleRef RuleID="ID_ALLOW_A_C35" />
          <FileRuleRef RuleID="ID_ALLOW_A_C36" />
          <FileRuleRef RuleID="ID_ALLOW_A_C37" />
          <FileRuleRef RuleID="ID_ALLOW_A_C38" />
          <FileRuleRef RuleID="ID_ALLOW_A_C39" />
          <FileRuleRef RuleID="ID_ALLOW_A_C3A" />
          <FileRuleRef RuleID="ID_ALLOW_A_C3B" />
          <FileRuleRef RuleID="ID_ALLOW_A_C3C" />
          <FileRuleRef RuleID="ID_ALLOW_A_C3D" />
          <FileRuleRef RuleID="ID_ALLOW_A_C3E" />
          <FileRuleRef RuleID="ID_ALLOW_A_C3F" />
          <FileRuleRef RuleID="ID_ALLOW_A_C40" />
          <FileRuleRef RuleID="ID_ALLOW_A_C41" />
          <FileRuleRef RuleID="ID_ALLOW_A_C42" />
          <FileRuleRef RuleID="ID_ALLOW_A_C43" />
          <FileRuleRef RuleID="ID_ALLOW_A_C44" />
          <FileRuleRef RuleID="ID_ALLOW_A_C45" />
          <FileRuleRef RuleID="ID_ALLOW_A_C46" />
          <FileRuleRef RuleID="ID_ALLOW_A_C47" />
          <FileRuleRef RuleID="ID_ALLOW_A_C48" />
          <FileRuleRef RuleID="ID_ALLOW_A_C49" />
          <FileRuleRef RuleID="ID_ALLOW_A_C4A" />
          <FileRuleRef RuleID="ID_ALLOW_A_C4B" />
          <FileRuleRef RuleID="ID_ALLOW_A_C4C" />
          <FileRuleRef RuleID="ID_ALLOW_A_C4D" />
          <FileRuleRef RuleID="ID_ALLOW_A_C4E" />
          <FileRuleRef RuleID="ID_ALLOW_A_C4F" />
          <FileRuleRef RuleID="ID_ALLOW_A_C50" />
          <FileRuleRef RuleID="ID_ALLOW_A_C51" />
          <FileRuleRef RuleID="ID_ALLOW_A_C52" />
          <FileRuleRef RuleID="ID_ALLOW_A_C53" />
          <FileRuleRef RuleID="ID_ALLOW_A_C54" />
          <FileRuleRef RuleID="ID_ALLOW_A_C55" />
          <FileRuleRef RuleID="ID_ALLOW_A_C56" />
          <FileRuleRef RuleID="ID_ALLOW_A_C57" />
          <FileRuleRef RuleID="ID_ALLOW_A_C58" />
          <FileRuleRef RuleID="ID_ALLOW_A_C59" />
          <FileRuleRef RuleID="ID_ALLOW_A_C5A" />
          <FileRuleRef RuleID="ID_ALLOW_A_C5B" />
          <FileRuleRef RuleID="ID_ALLOW_A_C5C" />
          <FileRuleRef RuleID="ID_ALLOW_A_C5D" />
          <FileRuleRef RuleID="ID_ALLOW_A_C5E" />
          <FileRuleRef RuleID="ID_ALLOW_A_C5F" />
          <FileRuleRef RuleID="ID_ALLOW_A_C60" />
          <FileRuleRef RuleID="ID_ALLOW_A_C61" />
          <FileRuleRef RuleID="ID_ALLOW_A_C62" />
          <FileRuleRef RuleID="ID_ALLOW_A_C63" />
          <FileRuleRef RuleID="ID_ALLOW_A_C64" />
          <FileRuleRef RuleID="ID_ALLOW_A_C65" />
          <FileRuleRef RuleID="ID_ALLOW_A_C66" />
          <FileRuleRef RuleID="ID_ALLOW_A_C67" />
          <FileRuleRef RuleID="ID_ALLOW_A_C68" />
          <FileRuleRef RuleID="ID_ALLOW_A_C69" />
          <FileRuleRef RuleID="ID_ALLOW_A_C6A" />
          <FileRuleRef RuleID="ID_ALLOW_A_C6B" />
          <FileRuleRef RuleID="ID_ALLOW_A_C6C" />
          <FileRuleRef RuleID="ID_ALLOW_A_C6D" />
          <FileRuleRef RuleID="ID_ALLOW_A_C6E" />
          <FileRuleRef RuleID="ID_ALLOW_A_C6F" />
          <FileRuleRef RuleID="ID_ALLOW_A_C70" />
          <FileRuleRef RuleID="ID_ALLOW_A_C71" />
          <FileRuleRef RuleID="ID_ALLOW_A_C72" />
          <FileRuleRef RuleID="ID_ALLOW_A_C73" />
          <FileRuleRef RuleID="ID_ALLOW_A_C74" />
          <FileRuleRef RuleID="ID_ALLOW_A_C75" />
          <FileRuleRef RuleID="ID_ALLOW_A_C76" />
          <FileRuleRef RuleID="ID_ALLOW_A_C77" />
          <FileRuleRef RuleID="ID_ALLOW_A_C78" />
          <FileRuleRef RuleID="ID_ALLOW_A_C79" />
          <FileRuleRef RuleID="ID_ALLOW_A_C7A" />
          <FileRuleRef RuleID="ID_ALLOW_A_C7B" />
          <FileRuleRef RuleID="ID_ALLOW_A_C7C" />
          <FileRuleRef RuleID="ID_ALLOW_A_C7D" />
          <FileRuleRef RuleID="ID_ALLOW_A_C7E" />
          <FileRuleRef RuleID="ID_ALLOW_A_C7F" />
          <FileRuleRef RuleID="ID_ALLOW_A_C80" />
          <FileRuleRef RuleID="ID_ALLOW_A_C81" />
          <FileRuleRef RuleID="ID_ALLOW_A_C82" />
          <FileRuleRef RuleID="ID_ALLOW_A_C83" />
          <FileRuleRef RuleID="ID_ALLOW_A_C84" />
          <FileRuleRef RuleID="ID_ALLOW_A_C85" />
          <FileRuleRef RuleID="ID_ALLOW_A_C86" />
          <FileRuleRef RuleID="ID_ALLOW_A_C87" />
          <FileRuleRef RuleID="ID_ALLOW_A_C88" />
          <FileRuleRef RuleID="ID_ALLOW_A_C89" />
          <FileRuleRef RuleID="ID_ALLOW_A_C8A" />
          <FileRuleRef RuleID="ID_ALLOW_A_C8B" />
          <FileRuleRef RuleID="ID_ALLOW_A_C8C" />
          <FileRuleRef RuleID="ID_ALLOW_A_C8D" />
        </FileRulesRef>
        <AllowedSigners>
          <AllowedSigner SignerId="ID_SIGNER_S_1D3" />
          <AllowedSigner SignerId="ID_SIGNER_S_1D8" />
          <AllowedSigner SignerId="ID_SIGNER_S_283" />
          <AllowedSigner SignerId="ID_SIGNER_S_287" />
          <AllowedSigner SignerId="ID_SIGNER_S_2A0" />
          <AllowedSigner SignerId="ID_SIGNER_S_2A1" />
          <AllowedSigner SignerId="ID_SIGNER_S_2AF" />
          <AllowedSigner SignerId="ID_SIGNER_S_2B4" />
        </AllowedSigners>
      </ProductSigners>
    </SigningScenario>
  </SigningScenarios>
  <UpdatePolicySigners />
  <CiSigners>
    <CiSigner SignerId="ID_SIGNER_S_1D3" />
    <CiSigner SignerId="ID_SIGNER_S_1D8" />
    <CiSigner SignerId="ID_SIGNER_S_283" />
    <CiSigner SignerId="ID_SIGNER_S_287" />
    <CiSigner SignerId="ID_SIGNER_S_2A0" />
    <CiSigner SignerId="ID_SIGNER_S_2A1" />
    <CiSigner SignerId="ID_SIGNER_S_2AF" />
    <CiSigner SignerId="ID_SIGNER_S_2B4" />
  </CiSigners>
  <HvciOptions>0</HvciOptions>
  <Settings />
  <BasePolicyID>{A244370E-44C9-4C06-B551-F6016E563076}</BasePolicyID>
  <PolicyID>{F1FA71F2-AFB0-4106-9A26-E89788335B21}</PolicyID>
</SiPolicy>
```