## Kerberos Delegation

### Unconstrained Delegation

### Constrained Delegation (with Protocol Transition)
- Protocol transition allows a user to autheticate to a server using a non-Kerberos compatible authetication system, and the server then request a TGS for itself without any pre-authentication. 
    - The KDC checks the service account's `userAccountValue` attribute for the value `TRUSTED_TO_AUTHETICATE_FOR_DELEGATION` to check if delegation is allowed. Next, the KDC returns a forwardable ticket for the *user's account (not the service account)*, known as **S4U2Self**.
- In next phase, the service on the aforementioned server can request a TGS for a service for specific servers that the particular server is allowed to delegate to (defined in `msds-allowedtodelegateto` attribute of the service account), by by passing the S4U2Self ticket to the KDC.
- The KDC returns a service ticket known as **S4U2Proxy** for the target service which is being requested by the first server.
- The S4U2Proxy service ticket is forwaded to the target server to access the service. 
>**Note:** Constrained delegation allows abuse of servers allowed to be delegated in two ways. Service can be accessed by 1) impersonating **any domain user**, 2) KDC does not validate if first server is requesting service ticket for only the specified service it is allowed to access, thus we can request **any service using the same service account** for the allowed target servers.

### Resource Based Constrained Delegation
- Unconstrained and Constrained Delegation Requires `SeEnableDelegation` privilege, which is only available to Domain Admins and Enterprise Admins. 
- Constrained Delegation is detemined by `msds-allowedtodelegateto` field in user proerty. If the user account who is allowed to delegate is compromised then, attacker can impersonate as *any user* and *any service* on the target server where Constrained delegation is allowed.
- Resource Based Constrained Delegation (RBCD) is configured the other way around. The resource who's service is being accessed, configures the delegation.
- RBCD abuse requires two steps, unlike other delegation types where it is already configured, RBCD abuse requires 
    - First configure the RBCD for an account where `GenericAll` or `GenericWrite` is available. 
    - The target object must have an SPN configured. A domain user can create and configure a machine account (Upto 10, [**_yes fake computer accounts are allowed to be created_**](https://www.youtube.com/watch?v=RUbADHcBLKg))