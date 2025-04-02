---

---
---
### Network File System (NFS)

**NFS** is a network file system developed by Sun Microsystems and has the same purpose as SMB. Its purpose is to access file systems over a network as if they were local. NFS is used between Linux and Unix systems on **port 2049**. This means that an NFS client cannot communicate directly with SMB servers. **NFS** is based on the ONC-RPC/SUN-RPC protocol exposed on TCP and UDP **ports 111**.

1. Show available NFS shares

	```bash
	showmount -e <target>
	```

2. Mounting NFS shares

```bash
	mkdir target-NFS
```

```bash
	sudo mount -t nfs <target>:/ ./target-nfs/ -o nolock
```

3. Nmap scripts
	Nmaps has some NSE scripts that can be used to the scans. These can then show the content for the share and its stats.
```bash
	sudo nmap --script nfs* <target> -sV -p 111,2049
```