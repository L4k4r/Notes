
## **1. ActiveDirectory PowerShell Module**

- Built-in PowerShell module for **managing and enumerating AD**.
- **147 cmdlets** available, useful for gathering domain and user information.

 **Key Commands:**
```powershell
# Check if the ActiveDirectory module is available 
Get-Module 

# Load the module if not already loaded 
Import-Module ActiveDirectory
```

**Enumeration:**
```powershell
# Get domain information
Get-ADDomain

# Get users with ServicePrincipalName (potential Kerberoasting targets)
Get-ADUser -Filter {ServicePrincipalName -ne "$null"} -Properties ServicePrincipalName

# Get domain trust relationships
Get-ADTrust -Filter *

# Enumerate all groups
Get-ADGroup -Filter * | Select Name

# Get detailed group information
Get-ADGroup -Identity "Backup Operators"

# Get group members
Get-ADGroupMember -Identity "Backup Operators"
```

**Find domain trusts, privileged users, and potential lateral movement paths.**

---

## **2. PowerView**

- PowerShell-based tool for **Active Directory reconnaissance**.
- Can enumerate **users, groups, ACLs, GPOs, domain trusts, and share permissions**.

**Key Commands:**
```powershell
# Get user details
Get-DomainUser -Identity mmorgan -Domain inlanefreight.local | Select-Object name, samaccountname, whencreated, admincount

# Enumerate domain groups and memberships (including nested groups)
Get-DomainGroupMember -Identity "Domain Admins" -Recurse

# Enumerate domain trust relationships
Get-DomainTrustMapping

# Test for local admin access on a machine
Test-AdminAccess -ComputerName ACADEMY-EA-MS01

# Find users with SPN set (Kerberoasting targets)
Get-DomainUser -SPN -Properties samaccountname,ServicePrincipalName
```

Useful for discovering attack paths, local admin rights, and Kerberoasting opportunities.

---

## **3. SharpView**

- **.NET port of PowerView** (useful when PowerShell is restricted).
- **Same functionality** as PowerView but does not rely on PowerShell.

**Example:**
```powershell
# Get user information
.\SharpView.exe Get-DomainUser -Identity forend
```

**Use when PowerShell execution is blocked.**

---

## **4. Snaffler**

- **Automated share enumeration** to find **sensitive data** (passwords, configs, keys).
- Scans **all readable shares** in the domain.

**Execution:**
```powershell
# Run Snaffler
Snaffler.exe -d inlanefreight.local -s -o snaffler.log -v data
```

Finds password files, SSH keys, database dumps, and other sensitive data.

---

## **5. SharpHound & BloodHound**

- **BloodHound:** Graph-based tool for **AD attack path analysis**.
- **SharpHound:** Data collector for BloodHound.

```powershell
# Collect AD data with SharpHound
.\SharpHound.exe -c All --zipfilename ILFREIGHT
```

Finds privilege escalation paths, misconfigurations, and excessive permissions.