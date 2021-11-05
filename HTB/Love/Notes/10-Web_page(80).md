# Enumeration-

![[Pasted image 20210702021044.png]]


# Gobuster-

```bash
root@napster:~/Documents/HTB/Love# gobuster dir -u http://10.10.10.239/ -w /usr/share/wordlists/seclists/Discovery/Web-Content/raft-small-words.txt -t 20 -o g
obuster1.txt                                                                                                                                                  
===============================================================                                                                                               
Gobuster v3.0.1                                                                                                                                               
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)                                                                                               
===============================================================                                                                                               
[+] Url:            http://10.10.10.239/                                                                                                                      
[+] Threads:        20                                                                                                                                        
[+] Wordlist:       /usr/share/wordlists/seclists/Discovery/Web-Content/raft-small-words.txt                                                                  
[+] Status codes:   200,204,301,302,307,401,403                                                                                                               
[+] User Agent:     gobuster/3.0.1                                                                                                                            
[+] Timeout:        10s                                                                                                                                       
===============================================================
2021/07/02 02:05:26 Starting gobuster
===============================================================
/plugins (Status: 301)
/includes (Status: 301)
/.html (Status: 403)
/images (Status: 301)
/admin (Status: 301)
/.htm (Status: 403)
/Admin (Status: 301)
/webalizer (Status: 403)
/Images (Status: 301)
/. (Status: 200)
/phpmyadmin (Status: 403)
/.htaccess (Status: 403)
/Includes (Status: 301)
/ADMIN (Status: 301)
/.htc (Status: 403)
/dist (Status: 301)
/IMAGES (Status: 301)
/.html_var_DE (Status: 403)
/licenses (Status: 403)
```

\>>> Interesting directory-

/admin	(301)


### Visiting the page-

We get the similar web page-

![[Pasted image 20210702021432.png]]
