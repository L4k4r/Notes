----
In **active reconnaissance**, the attacker **directly interacts with the target system** to gather information.

| **Technique**              | **Tools**                    | **Risk of Detection**                                                                                                 |
| ---------------------- | ------------------------ | ----------------------------------------------------------------------------------------------------------------- |
| Port Scanning          | Nmap, Masscan            | High: Direct interaction with the target can trigger IDS and firewalls                                            |
| Vulnerability Scanning | Nessus, Nikto, OpenVAS   | High: Vulnerability scanners send exploit payloads that security solutions can detect                             |
| Network Mapping        | Traceroute, Nmap         | Medium to High: Excessive or unusual network traffic can raise suspicion                                          |
| Banner Grabbing        | Netcat, Curl             | Low: Banner grabbing typically involves minimal interaction but can still be logged.                              |
| OS Fingerprinting      | nmap, Xprobe2            | Low: OS fingerprinting is usually passive, but some advanced techniques can be detected                           |
| Service Enumeration    | Nmap                     | Low: Similar to banner grabbing, service enumeration can be logged but it less likely to trigger alerts           |
| Web Spidering          | BurpSuite Spider, Scrapy | Low to MediumL Can be detected if the crawler's behaviour is not carefully configured to mimic legitimate traffic |

**Passive reconnaissance** involves gathering information about the target **without directly interacting** with it.

| **Technique**             | **Tools**                                                | **Risk of Detection**                                                                                     |
| --------------------- | ---------------------------------------------------- | ----------------------------------------------------------------------------------------------------- |
| Search Engine Queries | Google, DuckDuckGo, Bing, Shodan                     | Very low: Search engine queires are normal internet activity and unlikely to trigger alerts           |
| WHOIS Lookups         | whois command-line tool, online WHOIS lookup service | Very Low: WHOIS queries are legitimate and do not raise suspicion                                     |
| DNS                   | dig, nslookup, host, dnsenum, fierce, dnsrecon       | Very Low: DNS queries are essential for internet browsing and are not typically flagged as suspicious |
| Web Archive Analysis  | Wayback Machine                                      | Very Low: Accessing archived versions of websites is a normal activity                                |
| Social Media Analysis | LinkedIn, Twitter, Facebook, specialised OSINT tools | Very Low: Accessing public social media profiles is not considered intrusive                          |
| Code Repositories     | GitHub, GitLab                                       | Very Low: Code repositories are meant for public access, and searching them is not suspicious         |
