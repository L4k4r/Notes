----

Two of the most common utilities in Linux distributions to interact with web applications are `wget` and `curl`. These tools are installed on many Linux distributions.

To download a file using `wget`, we need to specify the URL and the option `-O' to set the output filename
```bash
wget https://<url>/file -O <output-file>
```

`cURL` is very similar to `wget`, but the output filename option is lowercase `-o'

```bash
curl -o <output-file> https://<url>/file
```