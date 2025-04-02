---

---
---
### Internet Message Access Protocol (IMAP) / Post Office Protocol (POP3)

**IMAP** allows managing emails directly on the server, supporting folder structures. It operates on **port 143** (unencrypted) and **port 993** (SSL/TLS). Commands allow listing, fetching, deleting and managing emails. 
**POP3** is a simplistic approach for downloading emails to a local client and removing them from the server. It operates on **port 110** (unencrypted) and **port 995** (SSL/TLS). It has a limited functionality compared to IMAP.

1. Key IMAP commands
	- `1 LOGIN <user> <pass>`: Log in to the server using username and password.
	-  `1 LIST "" *` *: List all mailboxes and folders.
	- `1 SELECT INBOX`: Opens the mailbox for accessing messages.
	- `1 FETCH <id> ALL`: Retrieves the specified message(s).
	- `1 CREATE "INBOX"`: Creates a new folder.
	- `1 DELETE "INBOX"`: Deletes a specified folder.
	- `1 LOGOUT`: Closes the conection to the server.

2. Key POP3 commands
	- `USER <username>`: Identifies the user to the server.
	- `PASS <password>`: Authenticates the user with the server.
	- `STAT`: DIsplays the number of and size of messages.
	- `LIST`: List all messages with their IDs and sizes.
	- `RETR <id>`: Retrieves the message with the specified ID.
	- `DELE <id>`: Deletes the message with the specified ID.
	- `QUIT`: Terminates the connection.


#### Footprinting IMAP / POP3 services
---
1. Check server features and settings:
	- IMAP
```bash
	curl -k 'imaps://<target_ip>' --user <username>:<password>
```

- POP3
```bash
	curl -k 'pop3s://<target_ip>' --user <username>:<password>
```

2. Logging In
	- Test credentials to access mailboxes
```bash
	openssl s_client -connect <target-ip>:993
```

3. SSL/TLS Analysis
	- Analyze the server's SSL/TLS certificates and encryption details
		- Look for certificate details (validity, issuer), encryption protocol (TLS 1.2, 1.3)
```bash
	openssl s_client -connect <target_ip>:imaps
```

