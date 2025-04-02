## Enumerating Users with Kerbrute

This tool uses *Kerberos Pre-Authentication*, which is a much faster and potentially stealthier way to perform password spraying. This method does not generate Windows event ID *4625: An account failed to log on, or a logon failure* which is often monitored for.

```bash
kerbrute userenum -d inlanefreight.local --dc 172.16.5.5 /opt/jsmith.txt 
```

Using Kerbrute for username enumeration will generate event ID 4768: A Kerberos authentication ticket (TGT) was requested. This will only be triggered if Kerberos event logging is enabled via Group Policy. Defenders can tune their SIEM tools to look for an influx of this event ID, which may indicate an attack. If we are successful with this method during a penetration test, this can be an excellent recommendation to add to our report.

#### Using NXC with Valid Credentials

```bash
sudo nxc smb 172.16.5.5 -u htb-student -p Academy_student_AD! --users
```

