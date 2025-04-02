---

---
---
### Simple Mail Transfer Protocol (SMTP)

Sending and forwarding emails over an IP network. Default port for SMTP communication is **port 25**. The secure port for authenticated email submission (STARTTLS) is **port 587**. The legacy port for encrypted SMTP (SSL/TLS) is **port 465**

1. Components of SMTP
	- **Mail Transfer Agent (MTA)**: Sends and receives emails.
	- **Mail Submission Agent (MSA)**: Pre-checks emails before MTA.
	- **Mail Delivery Agent (MDA)**: Delivers emails to the recipient's mailbox.

2. Key SMTP commands
	- **AUTH**: Authenticates the client to the server
	- **HELO/EHLO**: Starts the SMTP session with the server's hostname.
	- **MAIL FROM**: Specifies the sender's email address.
	- **RCPT TO**: Specifies the recipient's email address.
	- **DATA**: Sends the content of the email (ends with `<CR><LF>.<CR><LF>`)
	- **RSET**: Reset the current session without disconnecting.
	- **VRFY**: Verifies if a mailbox exists on the server.
	- **EXPN**: Expands mailing lists (if configured).
	- **QUIT**: Terminates the SMTP session.

#### Footprinting SMTP services
---
1. Enumerating SMTP features
```bash
	telnet <target_ip> 25
```

```bash
	EHLO <domain>
```

2. User enumeration
```bash
	VRFY <username>
```
- If the response is `252` the user exists. The results depend on server configuration and may give false positives.

3. Sending emails
```bash
MAIL FROM: <sender@example.com> 
RCPT TO: <recipient@example.com> 
DATA 
From: <sender@example.com> 
To: <recipient@example.com> 
Subject: Test Email 

This is a test email. 
.
```

4. Nmap scripts
	- Test if the SMTP server is misconfigured as an open relay
```bash
	nmap -p 25 --script smtp-open-relay <target_ip>
```
