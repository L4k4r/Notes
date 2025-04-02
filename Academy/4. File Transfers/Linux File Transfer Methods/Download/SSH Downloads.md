----
SSH (or Secure Shell) is a protocol that allows secure access to remote computers. SSH implementation comes with an `SCP` utility for remote file transfer that, by default, uses the SSH protocol.

#### Enabling the SSH Server
```bash
sudo systemctl enable ssh
```

#### Starting the SSH Server
```bash
sudo systemctl start ssh
```

#### Checking for SSH Listening Port
```bash
netstat -lnpt
```

#### Downloading Files Using SCP
```bash
scp plaintext@<ip>:/root/myroot.txt .
```

**Note:** You can create a temporary user account for file transfers and avoid using your primary credentials or keys on a remote computer.