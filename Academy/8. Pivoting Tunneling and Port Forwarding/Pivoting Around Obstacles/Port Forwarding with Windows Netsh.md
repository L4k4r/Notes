
[Netsh](https://docs.microsoft.com/en-us/windows-server/networking/technologies/netsh/netsh-contexts) is a Windows command-line tool that can help with the network configuration of a particular Windows system.

Let's take an example of the below scenario where our compromised host is a Windows 10-based IT admin's workstation (`10.129.15.150`,`172.16.5.25`).

We can use `netsh.exe` to forward all data received on a specific port (say 8080) to a remote host on a remote port. This can be performed using the below command.
#### Using Netsh.exe to Port Forward

```cmd-session
netsh.exe interface portproxy add v4tov4 listenport=8080 listenaddress=0.0.0.0 (or localhost) connectport=3389 connectaddress=172.16.5.25
```

#### Verifying Port Forward

```cmd-session
netsh.exe interface portproxy show v4tov4
```

#### Setting up RDP from attack host

```bash
xfreerdp /v:10.129.172.193:8080 /u:victor /p:pass@123
```
