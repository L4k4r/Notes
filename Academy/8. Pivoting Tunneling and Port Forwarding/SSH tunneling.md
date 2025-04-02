
### Local Port Fowarding
**Purpose:** Forward a local port (on your machine) to a specific port on a remote server or host.

```bash
ssh -L 8080:example.com:80 user@remote_host
```

### Dynamic Port Forwarding
**Purpose:** Acts as a SOCKS proxy, dynamically forwarding traffic to any remote host and port based on the application’s request.

```bash
ssh -D 9050 user@remote_host
```

See the end of `/etc/proxychains.conf` what port to use.

Example:
	- I want to establish an RDP connection to a host (`172.16.5.19`) located in the `172.16.5.0/24` network, but my machine is on a different network (`10.129.76.5/23`). A pivot host, which has interfaces in both networks (`10.129.76.0/23` and `172.16.5.0/24`), acts as a bridge.
	- To make this possible, I establish a **dynamic port forwarding** tunnel using SSH to the pivot host. Dynamic forwarding sets up a SOCKS proxy on my local machine. With the SOCKS proxy active, I use the command `proxychains xfreerdp /v:172.16.5.19` to redirect my RDP connection through the SOCKS proxy and the SSH tunnel. The SSH server on the pivot host then forwards the traffic to the destination (`172.16.5.19`) in the `172.16.5.0/24` network.
### Remote Port Fowarding
**Purpose:** Forward a port on the **remote server** to a port on your local machine.

```bash
ssh -R <InternalIPofPivotHost>:8080:0.0.0.0:8000 ubuntu@<ipAddressofTarget> -vN
```

- **`-v`**: Enables verbose logging to see how the forwarding is set up.
- **`-N`**: Ensures no remote command or shell is started

Example:
	I want to establish a reverse shell connection from a host in the `172.16.5.0/24` network (`172.16.5.19`) to my local machine, which is on a different network (`10.129.76.5/23`).
	A pivot host exists with interfaces in both networks (`10.129.76.0/23` and `172.16.5.0/24`), and I will use it to bridge the gap.
	Here’s how I set it up:
	1. On my local machine, I start a **Metasploit `multi/handler`** and set it to listen on `0.0.0.0:8000` to receive incoming connections.
	2. I create a payload that directs the reverse shell traffic to `172.16.5.129:8080`. This IP and port combination is in the `172.16.5.0/24` network, which the pivot host can access.
	3. I set up **remote port forwarding** using SSH:
	```ssh -R 172.16.5.129:8080:0.0.0.0:8000 ubuntu@10.129.76.122 -vN```
		This command tells the pivot host (`10.129.76.122`) to:
			- Listen on `172.16.5.129:8080`
			- Forward any traffic it receives on this port to my local machine's `0.0.0.0:8000` via the SSH tunnel.
	4. The target host (`172.16.5.19`) in the `172.16.5.0/24` network sends its reverse shell traffic to `172.16.5.129:8080`.
	5. The pivot host forwards this traffic through the SSH tunnel to `0.0.0.0:8000` on my local machine.

**Notes for Clarity**
- The command `ssh -R 172.16.5.129:8080:0.0.0.0:8000` sets up the reverse port forwarding and must remain active for the duration of the reverse shell session.
- The IP `172.16.5.129` in the `-R` flag could be replaced with `0.0.0.0` if you want the SSH server to listen on all interfaces in the `172.16.5.0/24` network.
### Choosing the Right Option

- Use **`-L`** when you need to access **remote resources locally**.
- Use **`-R`** when you need to **expose local resources remotely**.
- Use **`-D`** for **proxying or routing multiple connections dynamically**.