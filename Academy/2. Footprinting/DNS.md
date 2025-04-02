---

---
---
### Domain Name System (DNS)

**DNS** translates domain names to IP addresses and provides other domain-related information.

1. Types of server
	- **Root Server**: Handles top-level domains (TLDs).
	- **Authoritative Nameserver**: Responsible for specific zones.
	- **Caching Server**: Temporarily stores DNS query results.
	- **Forwarding Server**: Forwards queries to another DNS server.
	- **Resolver**: Resolves domain names locally or via recursive queries.


2. Important DNS records
	- **A**: Maps a domain to an IPv4 address.
	- **AAAA**: Maps a domain to an IPv6 address.
	- **MX**: Identifies mail servers for a domain.
	- **NS**: Specifies nameserver for a domain.
	- **TXT**: Stores arbitrary text, often used for SPF, DMARC, or verification.
	- **CNAME**: Aliases one domain to another.
	- **PTR**: Performs reverse lookup (IP to domain).
	- **SOA**: Contains zone information and admin email.

3. Key commands for footprinting
	 - List DNS records
```bash
	dig ns <domain> @<dns-server>
	dig mx <domain> @<dns-server>
	dig txt <domain> @<dns-server>
	dig axfr <domain> @<dns-server>
	dig soa <domain> @<dns-server>
```

4. DNS version query
```bash
	 dig CH TXT version.bind @<dns-server>
```

5. Subdomain enumeration
	- Brute-force subdomains with a wordlist
```bash
for sub in $(cat <wordlist>); do dig $sub.<domain> @<dns-server>; done
```

- Using dnsenum
```bash
	dnsenum --dnsserver <dns-server> --enum -f <wordlist> <domain>
```

6. Resolve an IP to a domain name
```bash
	dig -x <ip-address>
```