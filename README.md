
## Active Directory Security Testing

<details>
    <summary>Active Directory Enumeration</summary><p>

### Get Current Domain
- `$env:USERDOMAIN` (This command will return NETBIOS name  of the domain)

- `ipmo -Name ActiveDirectory; Get-ADDomain`  (This command imports the ActiveDirectory Module in powershell)

> **Note:** ActiveDirectory may not be available all computers in the domain,the only exeception is the Domain Controller where AD management tools are installed. The module can be Downloaded and imported externally from the [here](https://github.com/samratashok/ADModule).
1. Microsoft.ActiveDirectory.Management.dll
2. ActiveDirectory.psd1

### Enumerate all domains in current forest
- `nltest /domain_trusts` (This command returns domains in the current forest and the associated trusts)

</p></details>


