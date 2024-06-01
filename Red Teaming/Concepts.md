# # `AMSI(Anti-Malware Scan Interface)` -

Amsi is an interface that allows integration of Application & Services with any Anti-Malware Products.

## `Scans` -

- Memory and Stream Scanning.
- Content Source URL
- File Scanning

## `Integration with the folllowing` -

- UAC
- Powershell Scripts
- Windows Script Host
- Java Script & VBS
- Office Macros

## `WorkFlow` -

- AMSI Functions
- Amsiinitialize()				[Initialize AMSI API]
- AmsiOpenSession()			[Opens a session within which multiple scan requests can be done]
- AmsiScanBuffer()			[Scans a buffer full of content for Malware & returns the result]
- AmsiScanString()			[Scan strings for Malware]
- AmsiResultsMalware()		[Determines the result of AmsiScanBuffer() & AmsiScanString() if yes then should be blocked]
- AmsiUninitialize()			[Removes the instances of the AMSI API opened by Amsiinitialize() function]
- AmsiCloseSession()		        [Closes the session]



Disabling Amsi via Powershell -

```powershell
Set-MpPreference -DisableRealtimeMonitoring $True -Verbose
```

# # `Script Block Logging/Transcript` -

- Script Block Logging provides the ability to log de-obfuscated PowerShell code to event logs.

- Script Block Logging records the content of the script blocks it processes as well as the generated script code at execution time

## `Enabling SBE` -

```powershell
PS C:\Users\napst> New-Item -Path "HKLM:\SOFTWARE\Wow6432Node\Policies\Microsoft\Windows\PowerShell\ScriptBlockLogging" -Force
PS C:\Users\napst> Set-ItemProperty -Path "HKLM:\SOFTWARE\Wow6432Node\Policies\Microsoft\Windows\PowerShell\ScriptBlockLogging" -Name "EnableScriptBlockLogging" -Value 1 -Force
```

- Event Viewer Location :

	Applications and Services Logs-->Microsoft-->Windows-->PowerShell-->Operational Log


## `PowerShell Transcript` -

• Transcript is a text file that contains a history of commands and their output.

• Specifically designed for PowerShell, comes with `Microsoft.PowerShell.Host` PowerShell module.

• The log file can then be utilized for analysing the set of commands executed in a PowerShell session.

## `Enabling PsT` -

```powershell
PS C:\Users\napst> Get-Command -Module Microsoft.PowerShell.Host

CommandType     Name                                               Version    Source
-----------     ----                                               -------    ------
Cmdlet          Start-Transcript                                   3.0.0.0    Microsoft.PowerShell.Host
Cmdlet          Stop-Transcript                                    3.0.0.0    Microsoft.PowerShell.Host
PS C:\Users\napst> Start-Transcript
PS C:\Users\napst> Stop-Transcript
```

- Check Logged File Location :

```powershell
C:\Users\<User_Name>\Documents\PowerShell_transcript.COMP_NAME.BZAS4sU3.<Time_Stamp>.txt
```

## `PowerShell History File` -

The default location for this file is -

```bash
$env:APPDATA\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt
```

# # `CLM(Constrained Language Mode)` -

Prevents from running arbitrary powershell commands.Powershell has `LanguageMode` option that allows user to switch between allowed/disallowed syntaxes.Available Language modes -

- FullLanguage
- RestrictedLanguage
- NoLanguage
- ConstrainedLanguage

Powershell limits/blocks the usuage of following items when `CLM` is  enforced -

- Powershell Classes, APIs
- COM Objects
- Unapproved .NET Types
- XAML based workflows
- Powershell Add Types is blocked 
- Powershell Invoking

Implementation of CLM can be done by the following ways -

- Via Command Line	[not effective]
- Via Environment Variable	[least effective]
- Via Group Policy	[depend on Cmd or Env. Variable]
- In integration with Application Control Solution	[Most Effective]

Way to check Language Mode -

```powershell
PS C:\Users\napst> $ExecutionContext.SessionState.LanguageMode
FullLanguage
```

```powershell
PS C:\Users\napst> [Environment]::GetEnvironmentVariable('__PSLockdownPolicy')
```

Enabling `CLM` -

```powershell
PS C:\Users\napst> $ExecutionContext.SessionState.LanguageMode = "ConstrainedLanguage"
PS C:\Users\napst> $ExecutionContext.SessionState.LanguageMode
ConstrainedLanguage
```

CLM is originally designed to work with system-wide Application Controls (Ex : AppLocker, Microsoft Device Guard), hence there is no sense of enabling it in a standalone mode as there are multiple bypasses available.

# # `AppLocker` -

- AppLocker is a Software based whitelisting solution that restricts user to a specific set of apps on a device.
- Whitelisting in AppLocker happens through rules. They specify which applications are allowed to run on the device.
- AppLocker can : − 
	- Audit / Block execution of executable files (.exe and .com), scripts (ps1, cmd, vbs, bat, js), Installer files and DLL files 
	- Assign rule to a specific security group or an individual user 
	- Rules Types can be specified on the basis of : Path, Publisher, File Hash
- Based on the environment it can be configured in two ways :


Enabling Applocker -

```powershell
PS C:\Users\napst> Start-Service AppIDSvc –verbose -Force
PS C:\Users\napst> gpupdate /force
```

The recorded events in audit/block mode can be seen from :

Applications and Services Logs-->Microsoft-->Windows-->AppLocker

# # `Microsoft Defender Exploit Guard(ASR)` -

- WDEG and it’s 4 components are designed to lock down the device against a wide variety of common attack vectors by blocking them at various stages of Kill Chain.

- WDEG have 4 components :

1.  `Attack Surface Reduction (ASR)` – A set of controls that enterprises can enable to prevent malware from getting on the machine by blocking Office-, script-, and email-based threats
2. `Network Protection` – Blocks threats establishing outbound connection to untrusted domains / Ips.
3. `Controlled Folder Access` – Blocks untrusted processes from accessing controlled / protected folders.
4. `Exploit Protection` – Applies system wide mitigation settings like DEP, ASLR, CFG etc

- WDEG – ASR Component, gather real-time insights from WDAV and blocks most common techniques employed by Threat Actors.


# # `Credential Guard` -

- Credential guard is a security feature that used virtualization-based security to isolate domain credentials.
- Prior to Credential Guard, Windows stored secrets in the `Local Security Authority (LSA)`.
- Credentials are not stored in local memory rather it is stored in a virtual container preventing it from unauthorized access.
- With Credential guard enabled, is new process called `LSAlso` runs that stores & protects the domain credentials.
- LSA uses RPC to communicate with the isolated LSA process.
- When Credential guard is enabled, Kerberos does not allow unconstrained delegation.

# # `JEA(Just Enough Administration)` -

- It provides `Just Enough Privileges` to manage and perform tasks as a high-privileged user without providing full control.
- Not application specific `Role Based Access Control(RBAC)` functionality to anything that can be managed through Powershell.
- Basically, it provides a way for administrators to `delegate` certain admin tasks to non-administrator using Powershell.
- It can be used to reduce Administrators on the system, restrict/limit the capability of the users and read & understand the user behaviour by analysing the transcripts,logs etc.
- JEA follows, `Least Privilege principle` as it enables administrators to remove widely privileged local / domain administrator.
- Implementation of JEA :

	Creating JEA role capability files->Creating Session Configuration File->JEA


## `Usuage` -

1. First of all, create a configuration file of a PowerShell session (***.pssc**). To do it, run this command on your domain controller:

```powershell
New-PSSessionConfigurationFile -Path c:\JEA-Test\PSSessionConfigurationFile.pssc
```

The PSSC file sets who may connect to this JEA endpoint and under what account the commands in the JEA session will run.

Modify the following values:

-   **SessionType** from Default to **RestrictedRemoteServer**. This mode allows to use the following PowerShell cmdlets: Clear-Host, Exit-PSSession, Get-Command, Get-FormatData, Get-Help, Measure-Object, Out-Default or Select-Objectl
-   Specify a folder (create it) in the **TranscriptDirectory** parameter. Here you will log all JEA user actions: `TranscriptDirectory = C:\PS\JEA_logs`
-   The **RunAsVirtualAccount** option allows to run commands under a virtual administrator account (member of the local Administrator or Domain Administrator group): `RunAsVirtualAccount = $true`

2. In the RoleDefinitions directive, specify the AD security group allowed to connect to the PowerShell session and the name of the JEA role (it must match the PSRC file name we are going to create later).

```powershell
RoleDefinitions = @{‘woshub.com\HelpDesk' = @{ RoleCapabilities = 'HelpDesk_admins' }}
```

Save the config. file.

3. Create a new directory to keep the JEA configuration file, for example:

```powershell
New-Item -Path 'C:\JEA-Test\Modules\JEA\RoleCapabilities' -ItemType Directory
```

```powershell
New-PSRoleCapabilityFile -Path 'C:\JEA-Test\Modules\JEA\RoleCapabilities\PSRoleCapabilityFile.psrc'
```

The PSRC file specifies what is allowed to do in the current JEA session. In the **VisibleCmdlets** directive, you may specify the cmdlets (and their valid parameters) allowed to be used for a given user group.

4. Then register a new PSSession configuration for your PSSC file:

```powershell
Register-PSSessionConfiguration –Name testHelpDesk -Path 'C:\JEA-Test\PSSessionConfigurationFile.pssc'
```

5. Restart winrm Service -

```powershell
Restart-Service WinRM
```

6. You can list the available JEA endpoints:

```powershell
Get-PSSessionConfiguration|ft name
```

7. You can connect to the created JEA endpoint under a user account from the security group specified in the configuration file -

```powershell
Enter-PSSession -ComputerName dc01 -ConfigurationName testHelpDesk
```

# # `JIT(Just In Time Administration)` -

Instead of granting permanant access to users(like JEA), JIT gives access to specific resources for a specific timeframe.
There are three types of JIT -

1. Broker & Remove Access
2. Ephemeral Accounts
3. Temporary Accounts

# # `PAW(Privileged Access Workstation)` -

- PAW is a dedicated computing environment for performing admininstrative/sensitive tasks by privileged users.
- Highest Security Configuration designed for extremely sesnsitive roles (Domain Administrators/Enterprise Administrators etc) that would significantly impact the organisation if the workstation is compromised.
- Features of PAW :
				- Highly Restricted attack surface.
				- Hardened env. with Application Control & Application Guard deployed.
				- Credential Guard,Exploit Guard to protect the host from malicious behaviour.
				- Local disks encrypted with BitLocker.
				- Web browsing,Email,external USB access prohibited.
				- Very limited connection access can only be accessed via internally. 

# # `PAM(Privileged Access Management)` -

- It isolates the use of privileged accounts to reduce the risk of stolen credentials.
- PAM adds protection to privileged groups(Enterprise Admins,Domain Admins) that control access across range of domain-joined servers/computers.
- PAM uses/is based on the concepts of  `Just Enough Administration(JEA) & Just-In-Time Administration(JIT)`.

## `PAM Trust` -

- A trust between a `Bastion Forest` and a production forest.
-  PAM trust provide the ability to access the production forest with high privileges without using the credentials of bastion forest.
-  `Shadow Principals` with the SIDs value of privileged groups in Production Forest are created in Bastion Forest.

Example -

*`Bastion Host` - Server whose purpose is to provide access to a  private network from an untrusted external network.*
*`Shadow Principals`-Part of PAM feature,an AD object that represents a user,group or machine account from another forest.*

# # `LAPS(Local Administrator Passowrd Solution)` -

- LAPS provides centralized storage of local adminstrator passwords  in Active Directory with periodic randomizing.
- It significantly lowers the risk of Pass-the-Hash attack.
- The transmission of the password is encrypted(Kerberos) however the storage of the credentials is in `Clear-Text`.
- Only `Privileged groups` like Domain Admin or Enterprise Admin can read clear text passwords.
- However, through ACLs one can explicitly allow users to read the credentials

Location/Shows LAPS is in use -

```bash
C:\Program Files\LAPS\CSE\admpwd.dll
```

# # `EDR(Endpoint Detection & Response)` -

EDR is a solution that continuously monitors,stores endpoint-devices behaviour to detect and block suspicious/malicious activities and also provides remediation facilities all at one place(single dashboard).

## `Features` -

- Continuously Updating Database.
- Focuses on Indicator of Attacks(ie intention)
- Cloud Base Protection
- Realtime Monitoring

## `Examples` -

- FireEye Endpoint Security
- CrowdStrike Falcon Insight
- Microsoft Defender Advanced Threat Protection(ATP)

# # `Powershell Remoting` -

- PowerShell Remoting lets you run PowerShell commands or access full PowerShell sessions on remote Windows systems

- It’s similar to SSH for accessing remote terminals on other operating systems.

##  `Enabling PS Remoting` -

```poweshell
PS C:\Users\napst> Enable-PSRemoting -Force
```

```powershell
PS C:\Users\napst> Set-PSSessionConfiguration -ShowSecurityDescriptorUI -Name microsoft.PowerShell -Force
```

Describes session configurations, which determine the users who can connect to the computer remotely and the commands they can run.

```powershelll
PS C:\Users\napst> Set-Item WSMan:localhost\client\trustedhosts -value *
```

The asterisk is a wildcard symbol for all PCs. If instead you want to restrict computers that can connect, you can replace the asterisk with a comma-separated list of IP addresses or computer names for approved PCs.

```powerhell
PS C:\Users\napst> Get-Item WSMan:\localhost\Client\TrustedHosts
```

## `Usuage` -

Remote Command -

```powershell
Invoke-Command –ComputerName COMPUTER -ScriptBlock { COMMAND }
```

Remote Session -

```powershell
Enter-PSSession –ComputerName COMPUTER –Credentials USER
```

# # `Powershell Web Access` -

- Windows PowerShell commands and scripts can be run from a Windows PowerShell console in a web browser, with no Windows PowerShell, remote management software, or browser plug-in installation necessary on the client device.
- Users can access a Windows PowerShell console by using a web browser. When users open the secured Windows PowerShell Web Access website, they can run a web-based Windows PowerShell console after successful authentication.

## `Enabling PSWA`

```powershell
Set-PSSessionConfiguration -ShowSecurityDescriptorUI -Name microsoft.PowerShell -Force
```

Install Windows PowerShell Web Access -

```powershell
Install-WindowsFeature –Name WindowsPowerShellWebAccess
```

Configure Windows PowerShell Web Access, we will configure Windows PowerShell Web Access by installing the web application and configuring a predefined gateway rule-

```powershell
Install-pswaWebApplication -useTestCertificate
```

Set the authorization rule on who all can have rights for powershell web access,  * means all have access -

```powershell
Add-PswaAuthorizationRule –UserName * -ComputerName * -ConfigurationName *
```

