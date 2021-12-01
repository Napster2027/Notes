# # `Enumeration` -

```bash
root@napster:~/Documents/HTB/Intelligence# enum4linux 10.10.10.248
Starting enum4linux v0.8.9 ( http://labs.portcullis.co.uk/application/enum4linux/ ) on Wed Jul  7 12:32:25 2021

 ==========================
|    Target Information    |
 ==========================
Target ........... 10.10.10.248
RID Range ........ 500-550,1000-1050
Username ......... ''
Password ......... ''
Known Usernames .. administrator, guest, krbtgt, domain admins, root, bin, none


 ====================================================
|    Enumerating Workgroup/Domain on 10.10.10.248    |
 ====================================================
[E] Can't find workgroup/domain


 ============================================
|    Nbtstat Information for 10.10.10.248    |
 ============================================
Looking up status of 10.10.10.248
No reply from 10.10.10.248

 =====================================
|    Session Check on 10.10.10.248    |
 =====================================
Use of uninitialized value $global_workgroup in concatenation (.) or string at ./enum4linux.pl line 437.
[+] Server 10.10.10.248 allows sessions using username '', password ''
Use of uninitialized value $global_workgroup in concatenation (.) or string at ./enum4linux.pl line 451.
[+] Got domain/workgroup name:

 ===========================================
|    Getting domain SID for 10.10.10.248    |
 ===========================================
Use of uninitialized value $global_workgroup in concatenation (.) or string at ./enum4linux.pl line 359.
Domain Name: intelligence
Domain Sid: S-1-5-21-4210132550-3389855604-3437519686
[+] Host is part of a domain (not a workgroup)

 ======================================
|    OS information on 10.10.10.248    |
 ======================================
Use of uninitialized value $global_workgroup in concatenation (.) or string at ./enum4linux.pl line 458.
Use of uninitialized value $os_info in concatenation (.) or string at ./enum4linux.pl line 464.
[+] Got OS info for 10.10.10.248 from smbclient:
Use of uninitialized value $global_workgroup in concatenation (.) or string at ./enum4linux.pl line 467.
[+] Got OS info for 10.10.10.248 from srvinfo:
Could not initialise srvsvc. Error was NT_STATUS_ACCESS_DENIED

 =============================
|    Users on 10.10.10.248    |
 =============================
Use of uninitialized value $global_workgroup in concatenation (.) or string at ./enum4linux.pl line 866.
[E] Couldn't find users using querydispinfo: NT_STATUS_ACCESS_DENIED

Use of uninitialized value $global_workgroup in concatenation (.) or string at ./enum4linux.pl line 881.
[E] Couldn't find users using enumdomusers: NT_STATUS_ACCESS_DENIED

 =========================================
|    Share Enumeration on 10.10.10.248    |
 =========================================
Use of uninitialized value $global_workgroup in concatenation (.) or string at ./enum4linux.pl line 640.

        Sharename       Type      Comment
        ---------       ----      -------
SMB1 disabled -- no workgroup available

[+] Attempting to map shares on 10.10.10.248

 ====================================================
|    Password Policy Information for 10.10.10.248    |
 ====================================================
[E] Unexpected error from polenum:


[+] Attaching to 10.10.10.248 using a NULL share

[+] Trying protocol 139/SMB...

        [!] Protocol failed: Cannot request session (Called Name:10.10.10.248)

[+] Trying protocol 445/SMB...

        [!] Protocol failed: SAMR SessionError: code: 0xc0000022 - STATUS_ACCESS_DENIED - {Access Denied} A process has requested access to an object but has not been granted those access rights.

Use of uninitialized value $global_workgroup in concatenation (.) or string at ./enum4linux.pl line 501.

[E] Failed to get password policy with rpcclient


 ==============================
|    Groups on 10.10.10.248    |
 ==============================
Use of uninitialized value $global_workgroup in concatenation (.) or string at ./enum4linux.pl line 542.

[+] Getting builtin groups:

[+] Getting builtin group memberships:
Use of uninitialized value $global_workgroup in concatenation (.) or string at ./enum4linux.pl line 542.

[+] Getting local groups:

[+] Getting local group memberships:
Use of uninitialized value $global_workgroup in concatenation (.) or string at ./enum4linux.pl line 593.

[+] Getting domain groups:

[+] Getting domain group memberships:

 =======================================================================
|    Users on 10.10.10.248 via RID cycling (RIDS: 500-550,1000-1050)    |
 =======================================================================
Use of uninitialized value $global_workgroup in concatenation (.) or string at ./enum4linux.pl line 710.
[E] Couldn't get SID: NT_STATUS_ACCESS_DENIED.  RID cycling not possible.
Use of uninitialized value $global_workgroup in concatenation (.) or string at ./enum4linux.pl line 742.

 =============================================
|    Getting printer info for 10.10.10.248    |
 =============================================
Use of uninitialized value $global_workgroup in concatenation (.) or string at ./enum4linux.pl line 991.
Could not initialise spoolss. Error was NT_STATUS_ACCESS_DENIED


enum4linux complete on Wed Jul  7 12:33:22 2021
```

Nothing Useful :(