
#### Configuring MSF's SOCKS Proxy

Setup sock proxy in metasploit
```bash
msf6 > use auxiliary/server/socks_proxy

msf6 auxiliary(server/socks_proxy) > set SRVPORT 9050
SRVPORT => 9050
msf6 auxiliary(server/socks_proxy) > set SRVHOST 0.0.0.0
SRVHOST => 0.0.0.0
msf6 auxiliary(server/socks_proxy) > set version 4a
version => 4a
msf6 auxiliary(server/socks_proxy) > run
```

Confirm it's running
```bash
msf6 auxiliary(server/socks_proxy) > jobs

Jobs
====

  Id  Name                           Payload  Payload opts
  --  ----                           -------  ------------
  0   Auxiliary: server/socks_proxy
```

Now we need to add an autoroute so Metasploit knows where to send the traffic to.

```bash
msf6 > use post/multi/manage/autoroute

msf6 post(multi/manage/autoroute) > set SESSION 1
SESSION => 1
msf6 post(multi/manage/autoroute) > set SUBNET 172.16.5.0
SUBNET => 172.16.5.0
msf6 post(multi/manage/autoroute) > run
```

Or from within the meterpreter session
```bash
meterpreter > run autoroute -s 172.16.5.0/23
```


When you run `proxychains nmap` with the `socks_proxy` module, the traffic is routed through Metasploit and, thanks to `autoroute`, it knows to forward the traffic through the Meterpreter session to the target network. This setup makes it easy to pivot and interact with otherwise inaccessible networks.