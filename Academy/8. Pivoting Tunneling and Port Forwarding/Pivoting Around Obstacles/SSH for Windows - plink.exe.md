The Windows attack host starts a plink.exe process with the below command-line arguments to start a dynamic port forward over the Ubuntu server. This starts an SSH session between the Windows attack host and the Ubuntu server, and then plink starts listening on port 9050.

```cmd-session
plink -ssh -D 9050 ubuntu@10.129.15.50
```

Another Windows-based tool called [Proxifier](https://www.proxifier.com) can be used to start a SOCKS tunnel via the SSH session we created.
It is possible to create a profile where we can provide the configuration for our SOCKS server started by Plink on port 9050.

After configuring the SOCKS server for `127.0.0.1` and port 9050, we can directly start `mstsc.exe` to start an RDP session with a Windows target that allows RDP connections.

Once the traffic is routed through Proxifier, here's how the RDP connection is established:

1. **Local Traffic to SOCKS Proxy**:
    
    - The RDP client (`mstsc.exe`) initiates a connection to the Windows target (`172.16.0.1:3389`).
    - Proxifier reroutes this traffic to the SOCKS proxy on `127.0.0.1:9050`.
2. **Traffic Forwarded via SSH**:
    
    - The SOCKS proxy on port `9050` encrypts the RDP traffic and sends it over the SSH tunnel to the Ubuntu server (`10.129.15.50`).
3. **Ubuntu Server Routes Traffic**:
    
    - The Ubuntu server decrypts the traffic and forwards it to the final destination (`172.16.0.1:3389`) in the internal network.
    - This assumes the Ubuntu server has network access to the target machine (`172.16.0.1`) and that the target allows RDP connections.
4. **Target Responds**:
    
    - The RDP server on `172.16.0.1` responds to the RDP request.
    - The response is sent back through the same path: `172.16.0.1 -> Ubuntu server -> SSH tunnel -> Windows attack host (via Proxifier) -> RDP client`.