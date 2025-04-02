https://orange-cyberdefense.github.io/ocd-mindmaps/img/pentest_ad_dark_2022_11.svg
## **Tools for Enumeration**

### **1. CrackMapExec (CME)**

 - **Swiss army knife for AD enumeration.**  
- Supports **SMB, WinRM, SSH, MSSQL** protocols.

**Basic Commands:**
```bash
# Enumerate domain users
sudo crackmapexec smb <DC_IP> -u forend -p Klmcargo2 --users

# Enumerate domain groups
sudo crackmapexec smb <DC_IP> -u forend -p Klmcargo2 --groups

# Enumerate logged-on users
sudo crackmapexec smb <Target_IP> -u forend -p Klmcargo2 --loggedon-users

# Enumerate accessible SMB shares
sudo crackmapexec smb <DC_IP> -u forend -p Klmcargo2 --shares

# Search for files in shared directories
sudo crackmapexec smb <DC_IP> -u forend -p Klmcargo2 -M spider_plus --share '<SHARE_NAME>'
```

**Key Information Retrieved:**  
	 **Domain users** with attributes like **badPwdCount** (useful for password spraying).  
	 **Domain groups** and members (watch for **Admins, IT staff, Executives**).  
	 **Currently logged-in users** on systems (potential **privilege escalation** targets).  
	 **SMB shares** with **READ/WRITE** access (search for sensitive files).

---
### **2. SMBMap**

🔹 **Lists SMB shares, permissions, and contents.**  
🔹 Can search for files and recursively list directories.

Basic Commands:
```bash
# List available SMB shares and permissions
smbmap -u forend -p Klmcargo2 -d INLANEFREIGHT.LOCAL -H <DC_IP>

# Recursively list directories in a share
smbmap -u forend -p Klmcargo2 -d INLANEFREIGHT.LOCAL -H <DC_IP> -R '<SHARE_NAME>' --dir-only
```

**Identifies accessible shares** (e.g., **SYSVOL, NETLOGON, User Shares**).  
**Useful for searching scripts, configs, or credentials** inside shared files.

---
### **3. rpcclient**

- **SMB enumeration tool for Active Directory (AD).**  
- Can work **without authentication** if SMB NULL sessions are allowed.

**Basic Commands:**
```bash
# Connect anonymously if allowed
rpcclient -U "" -N <DC_IP>

# Enumerate domain users (includes RIDs)
rpcclient $> enumdomusers

# Query user details via RID
rpcclient $> queryuser <RID>
```

 **Lists domain users** & **their RIDs**.  
 **Can extract login details**, bad password count, and last login times.

---

### **4. Impacket Toolkit**

- **Python-based tools for AD attacks & enumeration.**

**psexec.py**
```bash
psexec.py inlanefreight.local/wley:'transporter@4'@<Target_IP>
```
**Remote shell as SYSTEM** (high privilege execution).

**wmiexec.py** (Stealthier than `psexec.py`)
```bash
wmiexec.py inlanefreight.local/wley:'transporter@4'@<Target_IP>
```
**Runs commands without dropping files** (avoids antivirus detection).

---

### **5. Windapsearch (LDAP Enumeration)**

-  **Enumerates users, groups, and computers via LDAP.**

**Basic Commands:**
```bash
# List Domain Admins
python3 windapsearch.py --dc-ip <DC_IP> -u forend@inlanefreight.local -p Klmcargo2 --da

# Find all privileged users
python3 windapsearch.py --dc-ip <DC_IP> -u forend@inlanefreight.local -p Klmcargo2 -PU
```

 **Finds all Domain Admins & Privileged users**.  
 **Detects nested group memberships** (hidden admin accounts).

---
### **6. BloodHound.py (Graph-Based AD Analysis)**

🔹 **Maps AD attack paths & privilege escalation routes.**

**Data Collection:**
```bash
# Collect all domain data
sudo bloodhound-python -u 'forend' -p 'Klmcargo2' -ns <DC_IP> -d inlanefreight.local -c all
```

**Extracts AD relationships: Users, Groups, Sessions, ACLs, Trusts.**  
**Finds paths to escalate to Domain Admin.**

---
https://orange-cyberdefense.github.io/ocd-mindmaps/img/pentest_ad_dark_2022_11.svg