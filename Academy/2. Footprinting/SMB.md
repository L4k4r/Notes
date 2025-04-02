___
### Server Message Block (SMB)

**SMB** is a protocol used for sharing files, printers and other network resources in Windows environment. It operates over **TCP port 445**.

1. When working with `smbclient` these are the key flags:
	- `-L`: List available shares on the target server.
	- `-N`: Suppresses password prompt for anonymous login.
	- `-U`: Specifies the username for authentication.
	- `--workgroup`: Sets the workgroup or domain.
	- `--no-pass`: Similar to `-N`.
	- `-c`: Executes a specific command on the SMB server.
	
```bash
	 smbclient -L //<target> -N
```

2. SMB Commands:
	- `ls`: List files and directories in the current share.
	- `get <file>`: Downloads a file.
	- `put <file>`: Uploads a file (if write permission exists).
	
3. Nmap scripts
```bash
	nmap --script smb-enum-shares -p 445 <target>
```
```bash
	nmap --script smb-enum-users -p 445 <target>
```

#### rpcclient 
---
**Rpcclient** is a command-line utility that comes with the Samba suite. It is used for interacting with the **Remote Procedure Call (RPC)** services on a Windows server over SMB (Server Message Block). These RPC services allow remote management and administration of Windows systems.

**Common Uses of rpcclient**
- Enumerating users, groups, shares, and policies on a Windows server.
- Querying details about the system (e.g., domain, password policies).
- Performing administrative tasks like adding/deleting users (if sufficient permissions are granted).

1. When working with rpcclient these are the important flags:
	`-U`: Specifies the username.
	`-N`: Skips password prompt.
	`-c`: Executes a command on the RPC server.
	`-I`: Specify the IP address of the server.
	`--workgroup`: Specifies the workgroup or domain for authentication.
	`-A <auth-file>`: Uses an authentication file containing credentials to log on.
	
	**Example anonymous login:**
	
```bash
	rpcclient -U "" -N <target>
```

2. Common rpcclient commands:
	`enumdomusers`: Enumerate all domain users.
	`queryuser <user>`: display user information.
	`enumdomgroups`: List all domain groups.
	`querygroup <group>`: Display group information.
	`netshareenumall`: Enumerate all available shares.
	`netsharegetinfo <share>`: Provides information about the share.
	`srvinfo`: Provide information about the server (OS version).
	`getdompwinfo`: Retrieve information about the domain password policy.

##### Tools used for SMB footprinting
 
Impackets [samrdump.py](https://github.com/fortra/impacket/blob/master/examples/samrdump.py) is used for listing system user accounts, available resource shares and other information.

The information gained with rpcclient can also be obtained using [SMBMap](https://github.com/ShawnDEvans/smbmap) and [CrackMapExec](https://github.com/byt3bl33d3r/CrackMapExec).

**Examples**;

```bash
smbmap -H <target>
```

```bash
crackmapexec smb <target> --shares -u '' -p ''
```

Another tool is [enum4linux-ng](https://github.com/cddmp/enum4linux-ng) which is based on the older enum4linux. This tool automates many of the queries, but not all, and can return a large amount of information.

**Installation and example:**

```bash
git clone https://github.com/cddmp/enum4linux-ng.git && cd enum4linux-ng && pip3 install -r requirements.txt
```

```bash
./enum4linux-ng.py <target> -A
```