This section covers enumeration using **only built-in Windows tools**, avoiding detection by security solutions like EDR, SIEM, and firewalls.
## **1. Basic Host & Network Recon**

Windows provides several built-in commands to gather **system and network information**.

Key Commands
```powershell
# Get system and domain details
hostname                          # Get system hostname
systeminfo                        # Detailed OS, patch, and domain information
[System.Environment]::OSVersion    # Get OS version

# List applied patches
wmic qfe get Caption,Description,HotFixID,InstalledOn

# Network configuration
ipconfig /all                      # Network settings, DHCP, DNS servers
set                                 # Lists environment variables
echo %USERDOMAIN%                   # Get domain name
echo %logonserver%                   # Identify domain controller
```

- Helps **understand the system state** before taking further actions.
- **Minimal footprint**, **low chance of triggering alerts**.

---
## **2. Harnessing PowerShell**

PowerShell is a **powerful administration tool** that allows **deep system interaction**.

**Key Commands**
```powershell
# Check available modules
Get-Module  

# Check execution policy (to determine if scripts can be run)
Get-ExecutionPolicy -List  

# Bypass script execution policy (temporary)
Set-ExecutionPolicy Bypass -Scope Process  

# Get environment variables
Get-ChildItem Env: | ft Key,Value  

# Retrieve PowerShell command history (may contain credentials!)
Get-Content $env:APPDATA\Microsoft\Windows\Powershell\PSReadline\ConsoleHost_history.txt  
```
- PowerShell can **list AD objects**, **search for credentials**, and **run scripts** stealthily.

**PowerShell Downgrade for Evasion***
```powershell
# Check current PowerShell version
Get-Host  

# Downgrade to PowerShell 2.0 (bypasses modern logging)
powershell.exe -version 2  
```
- PowerShell 2.0 **disables script block logging**, **reducing detection risk**.

---
## **3. Checking Defenses (Firewall & Antivirus)**

Understanding **firewall & AV settings** helps determine **attack feasibility**.

**Key Commands:**
```powershell
# Check Windows Firewall status
netsh advfirewall show allprofiles  

# Check Windows Defender status (from CMD)
sc query windefend  

# Get Windows Defender configuration (from PowerShell)
Get-MpComputerStatus  
```
- Determines **if defenses are active**.
- Identifies **potential gaps in security**.

---
## **4. Identifying Logged-in Users**

Avoid working on systems where **admins or users are active** to **reduce detection risk**.

 **Key Commands**
 ```powershell
 qwinsta  # Lists active and disconnected sessions
```
- Prevents actions that could **alert an active user**.

---
## **5. Enumerating Network Information**

Find **connected hosts** and **network routes** for **lateral movement**.

 **Key Commands**
```powershell
# List network devices the system knows about
arp -a  

# Print routing table (shows reachable networks)
route print  
```
- Helps **identify other machines** to pivot to.

---
## **6. Windows Management Instrumentation (WMI)**

WMI is a **built-in Windows service** used for **system management**.

 **Key Commands**
 ```powershell
 # List installed patches
wmic qfe get Caption,Description,HotFixID,InstalledOn  

# Get basic host information
wmic computersystem get Name,Domain,Manufacturer,Model,Username,Roles /format:List  

# List running processes
wmic process list /format:list  

# Get domain controllers
wmic ntdomain list /format:list  

# List all user accounts (local & domain)
wmic useraccount list /format:list  

# List system accounts (may contain service accounts)
wmic sysaccount list /format:list  
```
- **Fast enumeration** of system and domain-related objects.
- **Useful when PowerShell is restricted**.

---
## **7. Net Commands (User & Group Enumeration)**

Net commands help retrieve **user & group information**.

 **Key Commands**
 ```powershell
 # Get domain password policy
net accounts /domain  

# Get all domain groups
net group /domain  

# Get members of a specific group (e.g., Domain Admins)
net group "Domain Admins" /domain  

# List all domain users
net user /domain  

# Get detailed information on a user
net user <username> /domain  

# List available network shares
net view /domain  
```

**Evasion Trick:**  
Use `net1` instead of `net` to bypass **some security detections**.
```powershell
net1 user /domain  
```
- Finds **privileged users and groups** for **potential privilege escalation**.
- Identifies **network shares** containing **sensitive data**.

---
## **8. Dsquery (Active Directory Enumeration)**

`dsquery` searches **AD objects** when **PowerView/BloodHound are unavailable**.

**Key Commands**
```powershell
# Get all users
dsquery user  

# Get all computers
dsquery computer  

# Find Domain Controllers
dsquery * -filter "(userAccountControl:1.2.840.113556.1.4.803:=8192)" -limit 5 -attr sAMAccountName  

# Find users with no password requirement
dsquery * -filter "(&(objectCategory=person)(objectClass=user)(userAccountControl:1.2.840.113556.1.4.803:=32))" -attr distinguishedName userAccountControl  
```
- Quickly **finds critical AD objects**.
- Searches **user accounts with weak security settings**.

---
## **9. LDAP Filters for AD Search**

AD queries use **LDAP filters** to refine searches.

 **Example: Find Users Who Can’t Change Passwords**
```powershell
dsquery * -filter "(&(objectClass=user)(userAccountControl:1.2.840.113556.1.4.803:=64))"
```
- Allows **complex queries** on **users, groups, permissions, and policies**.
