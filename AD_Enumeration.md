## Active Directory Enumeration (Tools of the trade)
- PowerView
- ActiveDirectory Module 
> **Note:** To utilize ActiveDirectory module, import the module by entering `ipmo -Name ActiveDirectory` in a powershell console. ActiveDirectory module may not be available all computers in the domain,the only exeception is the Domain Controller where AD management tools are installed. The module can be Downloaded and imported externally from the [here](https://github.com/samratashok/ADModule).
1. Microsoft.ActiveDirectory.Management.dll
2. ActiveDirectory.psd1
- Bloodhound

## Active Directory Enumeration (What to enumerate)
This section focuses on what to enumerate from an active directory environment. The enumeration techniques present here cover both domain level enumeration and few enumeration points to do at a local system level as well.  

### Get Current Domain
- `$env:USERDOMAIN` (This command will return NETBIOS name  of the domain)
-  `Get-Domain` (PowerView) , the `-Domain` switch allows to retrieve details of a trusting domain
- `Get-ADDomain` (AD Module), the `-Identity` switch allows to retrieve details of a trusting domain

### Current Target Domain SID (Needed to create golden/silver tickets)
- Get-DomainSID 
- (Get-ADDomain).DomainSID

### Enumerate domain trust relationships
- `nltest /domain_trusts` (Native Windows command available on Windows server editions)
- `Get-DomainTrust` (Powerview), optional switch `-Domain` to specify a domain
- `Get-ADTrust` (AD Module), optional switch `-Identity` to specify a domain

### Domain Policy Enumeration
- `Get-DomainPolicyData` (PowerView) , This command will return domain policies such as Domain Password Policies and Kerberos Policies, which is needed to forge kerberos policy compliant golden\silver tickets, which is unlikely to stand out as anomalous iin event logs.
- `Get-ADDefaultDomainPasswordPolicy` (AD Module), To return the password policies enforced, Unfortunately I could not find any powershell cmdlet to get Kerberos Policies from PowerShell without importing `GroupPolicy` module. Just like AD module, by default it is not installed in non-DC servers. We can use `rsop.msc` from Run dialog(Win+R) to get resultant group policies.

### Identify Domain Controller 
- `Get-DomainController` (PowerView) , the `-Domain` switch allows to retrieve details of a trusting domain
- `Get-ADDomainController -Discover` (AD Module), the `-DomainName` switch allows to retrieve details of a trusting domain

### User Enumeration
- `Get-DomainUser -Properties *` (PowerView) , the `-Identity` switch allows to retrieve details of a particular user
- `Get-DomainUser` (AD Module), the `-Identity` switch allows to retrieve details of a particular user
> **Note:** We must look out for honey\decoy accounts by tracking login counts associated with each account. `Get-DomainUser -Properties samaccountname, logonCount | ft` command lists each account name and the logon count in a table format. The accounts which have less logon counts, are likely to be spotted by security teams or could very well be decoy accounts planted in the environment.

###  Group Enumeration
- `Get-DomainGroup` (PowerView) , the `-Domain` switch allows to specify a different domain
- `Get-ADGroup -Properties *` (AD Module)

### Group Membership Enumeration
- Get all members of a group
    - `Get-DomainGroupMember -Recurse -Identity <Group Name>`
    - `Get-ADGroupMember -Recursive -Identity "Domain Admins"`
- Get group membership of a user
    - `Get-DomainGroup -Username <username>`
    - `Get-ADPrincipalGroupMembership -Identity <username>`

### Computer Object Enumeration
- `Get-DomainComputer` (PowerView) , the `-Ping` switch only lists active computers 
- `Get-ADComputer -Filter * -Properties DNSHostName | %{Test-Connection -Count 1 -ComputerName $_.DNSHostName}` (AD Module), The piped command functions same as ping, which only lists active computers
> **Note:** (A computer object may not necessarily be an actual computer. A domain user has the default right to create/join upto 10 computer object, and yes you are correct, fake computer objects can be created and this is what we will do in AD CS exploitation for privilege escalation\Persistance)

### Enumerate Local Groups
- `Get-NetLocalGroup -ComputerName <Computer> ListGroups` (PowerView, needs Administrator privileges on non-dc machines, `-Recurse` is optional parameter)
- `Get-NetLocalGroupMember -ComputerName <Computer> -GroupName <Group Name>` (PowerView, needs Administrator privileges on non-dc machines)

### Enumerate Logged-on Users
- `qwinsta /server:<Computer Name>` (Natively available on Windows)
- `Get-NetLoggedon  -ComputerName <Computer Name>` (Powerview, to list actively logged on a computer, needs local administrator privilege)
- `Get-LoggedonLocal -ComputerName <Computer Name>` (Powerview, get locally logged on users on a system, needs remote registry)

### Enumerate Network File Shares
- `Invoke-ShareFinder -Verbose` (Powerview), Enumerate shares on hosts in current domain
- `Invoke-FileFinder -Verbose` (Powerview), Enumerate sensitive files on computers on domain
- `Get-NetFileServer` (Powerview), List all File-Servers in the domain

### Enumerate Group Policies
>**Note:** Only GPO names can be retrieved using PowerView, not the associated settings. Only locally applicable GPO can be found using rsop.msc or gpedit.msc
- `Get-DomainGPO` (Powerview), optional parameter `-ComputerIdentity` allows selecting a computer
- `Get-DomainGPOLocalGroup` (Powerview), Enumerate GPOs that are applicable on Restricted Groups)
- `Get-DomainGPOComputerLocalGroupMapping` (Powerview), The command has two optional parameters -Identity <User> or -ComputerIdentity <Computer>, with either of the switches selected, a user's or a computer's local group membership using a GPO can be enumerated

### Enumerate Organizational Units
>**Note:** GPOs are applied on Organizational Units
- `Get-DomainOU` (PowerView)
- `Get-DomainGPO -Identity <OU GUID>` (Powerview), Enumerates GPO and OU mapping with OU identity specified by OU's GUID retrieved from OU's gplink attribute
- `Get-ADOrganizationalUnit -Filter * -Properties *` (AD Module)

### Enumerate ACLs
>**Note:** All "Authenticated Users" can read ACLs unless it is explicitly blocked, Findings of Over-Permissive ACLs for a user by Enumerating group membership of a user, ACLs may not directly include a username for permission most of the time, however, ACLs may be present for an object which has GenericAll or Write permission for a group user is in.
- `Find-InterestingDomainAcl -ResolveGUIDs` (Powerview), to enumerate interesting ACLs in the domain
- `Get-DomainObjectAcl -SamAccountName <account name> -ResolveGUIDs -Verbose` (Powerview), optional switch `-SearchBase` allows specifying filtering criteria using LDAP syntax
- `Get-Acl
'AD:\CN=Administrator,CN Users,DC dollarcorp,DC moneycorp,DC=local'` (AD Module)

### Enumerate Forest Mapping
- Get details about current forest
    - `Get-Forest` (Powerview), optional parameter `-Forest` allows specifying forest.
    - `Get-ADForest` (AD Module), optional parameter `-Identity` allows specifying forest.
- Get all domains in the current forest
    - `Get-ForestDomain` (Powerview), optional parameter  `-Forest` allows specifying forest.
    - `(Get-ADForest).Domains` (AD Module)
- Get forest trust mapping
    - `Get-ForestTrust` (Powerview), optional parameter  `-Forest` allows specifying forest.
    - `Get-ADTrust -Filter 'msDS-TrustForestTrustInfo -ne
"$null"'` (AD Module)

## Hunting for user privileges (Pre-PrivEsc)
- `Find-DomainUserLocation` (check lab manual for syntax)

## SQL Server Enumeration
- [SQL Server Enumeration](AD_PrivEsc.md#ms-sql-service-enumeration)