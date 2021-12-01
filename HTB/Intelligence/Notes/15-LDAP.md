# # `Enumeration` -

```bash
root@napster:~/Documents/HTB/Intelligence# ldapsearch -x -h intelligence.htb -s base namingcontexts
# extended LDIF
#
# LDAPv3
# base <> (default) with scope baseObject
# filter: (objectclass=*)
# requesting: namingcontexts 
#

#
dn:
namingcontexts: DC=intelligence,DC=htb
namingcontexts: CN=Configuration,DC=intelligence,DC=htb
namingcontexts: CN=Schema,CN=Configuration,DC=intelligence,DC=htb
namingcontexts: DC=DomainDnsZones,DC=intelligence,DC=htb
namingcontexts: DC=ForestDnsZones,DC=intelligence,DC=htb

# search result
search: 2
result: 0 Success

# numResponses: 2
# numEntries: 1
```

Nothing unusual