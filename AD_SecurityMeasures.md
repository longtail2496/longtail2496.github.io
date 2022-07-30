## Active Directory Security Measures 

### Enumerating Security Measures
- Powershell Constrained Lanugage Mode (CLM):
    - To check if powershell *Constrained Mode* is enabled, execute the command `$ExecutionContext.SessionState.LanguageMode`
- Applocker Policies: 
    - `Get-AppLockerPolicy -Effective`
    - `reg.exe query HKLM\Software\Policies\Microsoft\Windows\SRPV2`
- Windows Device Guard (WDAG)

### Security Measure Bypass:
- Windows Defender Bypass:
    - Disabling Defender Real-Time monitoring:
    - Disabling Defender AntiVirus Protection (IOfficeAntivirus):
    - Exclusion List:
    
- AMSI Bypass: