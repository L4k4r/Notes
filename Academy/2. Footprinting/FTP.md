___
### File Transfer Protocol (FTP)

FTP is a standard network protocol used to transfer files between a client and a server over a TCP/IP network. It operates on **port 21 (control)** for commands and **port 20 (data)** for transferring files (though passive mode uses random ports for data).

1. Anonymous Login:
	   - Many FTP servers allow anonymous access for public files.
	   - Use `ftp <target>` and log in with username `anonymous` and a blank password.

2. **FTP commands:**
	- `USER`: Specify a username.
	- `PASS`: Provide the password.
	- `LIST`: List files and directories.
	- `PWD`: Print the current directory.
	- `CWD`: Change directory.
	- `QUIT`: Exit session.

3. **Nmap script**
	 -  Anonymous Login Test
```bash
	nmap --script ftp-anon -p 21 <target>
```

