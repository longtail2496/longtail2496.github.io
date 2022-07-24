
# Active Directory Enumeration
### Get Current Domain Name
```$env:USERDOMAIN``` (This command will return NETBIOS name  of the domain)

```ipmo -Name ActiveDirectory; Get-ADDomain```  (This command imports the ActiveDirectory Module in powershell)

**Note:** ActiveDirectory may not be available all computers in the domain. The module can be installed or imported externally by importing below files (Download available [here](https://github.com/samratashok/ADModule]))
1. Microsoft.ActiveDirectory.Management.dll
2. ActiveDirectory.psd1

### Enumerate all domains in current forest
```nltest /domain_trusts```

