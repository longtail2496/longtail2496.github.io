## Active Directory Lateral Movement (Tools of the trade)
- Powershell
- PsExec
- winrs
- xcopy

## Active Directory Lateral Movement (Techniques)

### Powershell Remoting
- **Interactive Remoting** (This allows creation of a remote powershell session on a target host, session can be created just in time and destroyed as soon as exited or session info can stored into a variable to use a previous session either for interactive access or copy to and from a remote system), Powershell Remoting does not execute powershell.exe on remote system, it is rather a different process named *wsmprovhost.exe* that hosts PsSesssions
    - **One time session** `Enter-PsSession -ComputerName <Computer Name>` 
    - **Saved Session (Stateful)** `$sess=New-PsSession; Enter-PsSession -Session $sess` (with `Invoke-Commnd -Session`, commands can be run remotely as well over an existing powershell remote session)
- **Fan-out Remoting** (Allows to execute a scirpt-block or a file remotely in the memory of remote servers, This is a easy way to manage servers in an organization), the parameter `-ComputerName` allows to specify a single computer name or it can receive multiple computers from a list as shown below
    - **Remote Script-Block Execution** `Invoke-Command -ScriptBlock {<cmdlet>} -ComputerName (gc <List of servers>)`
    - **Remote Script Execution** `Invoke-Command -FilePath <Local File Path> -ComputerName (gc <List of servers>)`
- **Running Local Functions on Remote Machines** (The Powershell *Function Store* (can be accessed by $function) contains list local functions, which can be run on remote systems), Optional parameter `-ArgumentList` can be used to pass parameters to the function being invoked 
    - `Invoke-Command -ScriptBlock ${function:<Function Name>} -ComputerName (gc <List of servers>)`

## Copying Files to a Remote Computer

### Copying Files using Powershell Session
- **File Copy to a Remote System** `Copy-Item -ToSession $sess "<source path> -Destination <destination path>`
- **File Copy from a Remote System** `Copy-Item -FromSession $sess "<source path> -Destination <destination path>`