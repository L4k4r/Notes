Default minimum password length for newly created domains is 7.
Usually it is 8.

We can use rpcclient to check a Domain Controller for SMB NULL session access.
```bash
rpcclient -U "" -N <ip>
```
To get info in RPC we can use `querydominfo`.
To obtain the password policy we can use `getdompwinfo`

With nxc we can get the password policy with an SMB NULL session

```bash
nxc smb <ip> -u '' -p '' --pass-pol
```

---
## Enumerating Null Session - from Windows

It is less common to do this type of null session attack from Windows, but we could use the command `net use \\host\ipc$ "" /u:""` to establish a null session from a windows machine and confirm if we can perform more of this type of attack.

#### Using net.exe
If we can authenticate to the domain from a Windows host, we can use built-in Windows binaries such as `net.exe` to retrieve the password policy.
```cmd
net accounts
```
