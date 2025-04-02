---

---
---
### Simple Network Management Protocol (SNMP)

**SNMP** is used for monitoring and managing network devices like routers, switces and servers. It uses **port 161** for queries and control commands and **port 162** for traps (unsolicited notifications from devices to clients).

There are different versions of SNMP. 
	- SNMPv1: Basic functionality, no encryption or authentication.
	- SNMPv2c: Similar to v1 but with more features, still no encryption.
	- SNMPv3: Includes authentication and encryption, providing better security.

**Key concepts**
	A critical component of SNMP is the Management Information Base (MIB). This is a database used for managing the entities. It contains **Object Identifiers (OIDs)** representing network devices. Community strings act as passwords to access SNMP data. Default strings are `public` (read-only) and `private` (read/write). They are transmitted in plaintext in v1 and v2. 

#### Footprinting SNMP

1. SNMPwalk 
	-  Queries all available OIDs for information
```bash
	snmpwalk -v<version> -c <community string> <target_ip>
```

2. Onesixtyone
	- Brute-forces community strings from a wordlist
```bash
	onesixtyone -c <wordlist> <target_ip>
```

3. Braa
	- Enumerates OIDs and retrieves detailed information
```bash
	braa <community string>@<IP>:<OID>
```

4. Snmpget
	- Retrieves specific OIDs
```bash
snmpget -v<version> -c <community string> <target_ip> <OID>
```