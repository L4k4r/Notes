----
While manual analysis of HTTP headers and reverse DNS lookups can be effective, specialised virtual host discovery tools automate and streamline the process, making it more efficient and comprehensive. These tools emloy various techniques to probe the target server and uncover potential virtual hosts.

| Tool        | Description                                                                                                     | Features                                                       |
| ----------- | --------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------- |
| gobuster    | A multi-purpose tool often used for directory/file brute-forcing, but also effective for virtual host discovery | Fast, supports multiple HTTP methods, can use custom wordlists |
| Feroxbuster | Similar to Gobuster, but with a Rust-based implementation, known for its speed and flexibility                  | Supports recursion, wildcard discovery and various filters     |
| ffuf        | Another Fast web fuzzer that can be used for virtual host discovery by fuzzing the host header                  | Customizable wordlist input and filtering options              |
**Gobuster example**

```bash
gobuster vhost -u http://<target_ip> -w <wordlist> --append-domain
```

Make sure to add the subdomain to the /etc/host file with the right IP address.