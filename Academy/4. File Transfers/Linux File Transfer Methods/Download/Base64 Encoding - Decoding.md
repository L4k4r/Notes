-----

Depending on the file size we want to transfer, we can use a method that does not require network communication. If we have access to a terminal, we can encode a file to a base64 string, copy its content into the terminal and perform the reverse operation. 

Encode file to Base64
```bash
cat id_rsa |base64 -w 0;echo
```

Decode the file
```bash
echo -n '<string>' | base64 -d > id_rsa
```

Finally, we can confirm if the file was transferred successfully using the `md5sum` command.