# Kerberos Delegation

### Resource Based Constrained Delegation
- Unconstrained and Constrained Delegation Requires `SeEnableDelegation` privilege, which is only available to Domain Admins and Enterprise Admins. 
- Constrained Delegation is detemined by `msds-allowedtodelegateto` field in user proerty. If the user account who is allowed to delegate is compromised then, attacker can impersonate as *any user* and *any service* on the target server where Constrained delegation is allowed.
- Resource Based Constrained Delegation (RBCD) is configured the other way around. The resource who's service is being accessed, configures the delegation.
- RBCD abuse requires two steps, unlike other delegation types where it is already configured, RBCD abuse requires 
    - First configure the RBCD for an account where `GenericAll` or `GenericWrite` is available. 
    - The target object must have an SPN configured. A domain user can create and configure a machine account (Upto 10, [**_yes fake computer accounts are allowed to be created_**](https://www.youtube.com/watch?v=RUbADHcBLKg))