# Active Directory Enterprise Lab (Azure)

This project documents my hands-on Active Directory lab built in Microsoft Azure using Windows Server 2025 Datacenter. I simulated a small company with 30 users and configured role-based access, departments, group policies, and common IT support scenarios.


## Technologies Used

- Microsoft Azure
- Windows Server 2025 Datacenter
- Active Directory Domain Services
- Group Policy Management
- PowerShell
- Windows Remote Desktop

## Project Summary

In this lab, I created a Windows Server virtual machine in Azure and promoted it to a Domain Controller. 
Then I built a company structure with:<br>

- 23 Employees  
- 5 Managers  
- 2 IT Admins  
- 3 Departments (Sales, HR, Finance)  

I practiced:
- Creating users with PowerShell
- Creating Organizational Units (OUs)
- Creating security groups
- Applying Group Policies
- Locking and unlocking accounts
- Resetting passwords
- Disabling and deleting users

This lab helped me understand how Active Directory works in a real company environment.

---

## 1. Environment Setup

### Azure Virtual Machine

- Created a Windows Server 2025 Datacenter VM in Azure
- Size: Standard B2als v2 (2 vCPUs, 4GB RAM)
- Region: West US 3
- Enabled RDP access (port 3389)
- Connected using Microsoft Remote Desktop from my Mac

Screenshot:

<img width="2878" height="1340" alt="Screenshot 2026-02-19 at 6 55 03 PM" src="https://github.com/user-attachments/assets/e11e71c2-b568-4e49-8893-d7998ff7dd89" />


---

## 2. Promoting to Domain Controller

After logging into the server:

1. Installed Active Directory Domain Services (AD DS)
2. Promoted the server to a Domain Controller
3. Created a new domain: `lab.local`
4. DNS was configured automatically
5. Rebooted and logged in as domain admin

Screenshots:
<img width="1820" height="1138" alt="Screenshot 2026-02-19 at 7 04 19 PM" src="https://github.com/user-attachments/assets/b5b92da8-36b0-49f8-8c9e-e57d33e7b197" />
<img width="1824" height="1140" alt="Screenshot 2026-02-19 at 7 05 51 PM" src="https://github.com/user-attachments/assets/8dc0b9e3-3fe1-4639-ae09-523894d539aa" />
<img width="1696" height="944" alt="Screenshot 2026-02-19 at 7 08 57 PM" src="https://github.com/user-attachments/assets/5e6b9a3e-a506-4e13-b1d2-b075fd9caf35" />
<img width="1712" height="1008" alt="Screenshot 2026-02-19 at 7 09 49 PM" src="https://github.com/user-attachments/assets/8694c4c4-2b68-49ed-a3eb-0de7679ac918" />



---

## 3. Designing the Company Structure

### Organizational Units (OUs)

I created top-level OUs:

- IT-Admins
- Managers
- Employees

Inside the Employees OU, I created department OUs:

- Sales
- HR
- Finance

This keeps everything organized by both role and department.

Add Screenshot Here:


---

## 4. Creating Users with PowerShell

Instead of creating users manually one by one, I used PowerShell.

### Employee Creation Script (23 Employees)

Saved as: `powershell-user-creation.ps1`

```powershell
Import-Module ActiveDirectory

for ($i = 1; $i -le 23; $i++) {
    $username = "emp$i"
    $password = ConvertTo-SecureString "Temp@1234!" -AsPlainText -Force

    New-ADUser `
        -Name $username `
        -SamAccountName $username `
        -UserPrincipalName "$username@lab.local" `
        -AccountPassword $password `
        -Enabled $true `
        -ChangePasswordAtLogon $true `
        -Path "OU=Employees,DC=lab,DC=local"

    Add-ADGroupMember -Identity "Employees-Group" -Members $username
} 
```

All employees were required to change their password at first login.

Screenshot:  

---

## Manager Creation Script (5 Managers)

Saved as: `powershell-user-creation.ps1`

```powershell
Import-Module ActiveDirectory

for ($i = 1; $i -le 5; $i++) {
    $username = "manager$i"
    $password = ConvertTo-SecureString "Temp@1234!" -AsPlainText -Force

    New-ADUser `
        -Name $username `
        -SamAccountName $username `
        -UserPrincipalName "$username@lab.local" `
        -AccountPassword $password `
        -Enabled $true `
        -ChangePasswordAtLogon $true `
        -Path "OU=Managers,DC=lab,DC=local"

    Add-ADGroupMember -Identity "Managers-Group" -Members $username
}
```

---


## IT Admin Creation Script (2 IT Admins)

Saved as: `powershell-user-creation.ps1`

```powershell
Import-Module ActiveDirectory

$admins = @("itadmin1", "itadmin2")
$password = ConvertTo-SecureString "Temp@1234!" -AsPlainText -Force

foreach ($user in $admins) {
    New-ADUser `
        -Name $user `
        -SamAccountName $user `
        -UserPrincipalName "$user@lab.local" `
        -AccountPassword $password `
        -Enabled $true `
        -ChangePasswordAtLogon $true `
        -Path "OU=IT-Admins,DC=lab,DC=local"
}
```

IT Admins were added to the **IT-Admins-Group** with elevated permissions.

---

## 5. Security Groups

Created the following security groups:

- Employees-Group  
- Managers-Group  
- IT-Admins-Group  
- Sales-Group  
- HR-Group  
- Finance-Group  

Users were added to:

- Domain Users  
- Their role group  
- Their department group  

### Examples

- emp1 → HR-Group  
- emp6 → Finance-Group  
- emp9 → Sales-Group  

This follows **Role-Based Access Control (RBAC)**, where access is assigned based on role and department.

Screenshot:
_Add screenshot of Security Groups and user memberships here._


---

## 6. Group Policy Configuration

### Employees Restrictions GPO

Applied to the **Employees OU**.

Employees were restricted from:

- Opening Control Panel  
- Using Command Prompt  
- Using Registry Editor  
- Opening Task Manager  
- Installing software from removable media  

Screenshot:
_Add screenshot of Employees GPO settings here._

---

### Managers Policy

Managers had fewer restrictions:

- Allowed Command Prompt  
- Allowed Task Manager  
- Limited Control Panel access  
- Still blocked from Registry Editor  

Screenshot:
_Add screenshot of Managers GPO settings here._

---

## 7. Account Lockout Policy

Configured domain policy to:

- Lock account after 5 failed login attempts  
- Set lockout duration  
- Reset counter after a defined time  

This simulates basic enterprise security controls.

Screenshot:
_Add screenshot of Account Lockout Policy settings here._

---

## 8. IT Support Scenarios Practiced

### Password Reset Scenario

Scenario: **emp10 forgot their password**

Steps:

1. Open Active Directory Users and Computers  
2. Right-click the user  
3. Select **Reset Password**  
4. Force password change at next logon  

Screenshot:
_Add screenshot of password reset window here._

---

### Account Lockout Scenario

Entered the wrong password 5 times to lock the account.

An IT Admin unlocked the account through user properties in Active Directory.

Screenshot:
_Add screenshot of locked account properties here._

---

### Disable Account (Terminated Employee)

- Right-click user → Disable Account  
- User can no longer log in  

Screenshot:


---

### Delete User

- Right-click user → Delete  
- Account permanently removed from the domain  

Screenshot:

---

## 9. Remote Access

Connected to the server using:

- Microsoft Remote Desktop (Mac)  
- Azure VM public IP address  
- Domain credentials  

Tested logins using:

- IT Admin accounts  
- Manager accounts  
- Employee accounts  

Each role had different restrictions based on applied Group Policies.

Screenshot:
_Add screenshot of RDP session here._

---

## What I Learned

- How to deploy a Windows Server VM in Azure  
- How to promote a Domain Controller  
- The difference between OUs and Security Groups  
- How to automate user creation using PowerShell  
- How Group Policy controls user behavior  
- How account lockout and password reset work  
- How to simulate real IT admin tasks  

This lab helped me understand how enterprise Active Directory environments are structured and managed in real-world IT settings.


---

## Possible Improvements

- Join a Windows 11 client machine to the domain  
- Add department file shares with NTFS permissions  
- Add login scripts  
- Enable auditing policies  
- Implement backup testing  


