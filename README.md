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

#### Screenshot:
<img src="https://github.com/user-attachments/assets/e11e71c2-b568-4e49-8893-d7998ff7dd89" width="600"/>


---

## 2. Promoting to Domain Controller

After logging into the server:

1. Installed Active Directory Domain Services (AD DS)
2. Promoted the server to a Domain Controller
3. Created a new domain: `lab.local`
4. DNS was configured automatically
5. Rebooted and logged in as domain admin

#### Screenshot:
<p align="center">
<img width="500" alt="Screenshot 2026-02-19 at 7 04 19 PM" src="https://github.com/user-attachments/assets/b5b92da8-36b0-49f8-8c9e-e57d33e7b197" />
<img width="500" alt="Screenshot 2026-02-19 at 7 09 49 PM" src="https://github.com/user-attachments/assets/8694c4c4-2b68-49ed-a3eb-0de7679ac918" />
</p>
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

#### Screenshot:
<img width="600" alt="Screenshot 2026-02-19 at 7 26 00 PM" src="https://github.com/user-attachments/assets/d8849f17-329e-4fa0-b887-393d3ad43478" />

---

## 4. Creating Users with PowerShell

Instead of creating users manually one by one, I used PowerShell.

### Employee Creation Script (23 Employees)

```powershell
Import-Module ActiveDirectory

# Create employees accounts with temporary passwords
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

} 
```

All employees were required to change their password at first login.


---

## Manager Creation Script (5 Managers)

```powershell
Import-Module ActiveDirectory

# Create manager accounts with temporary passwords
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

}
```

---


## IT Admin Creation Script (2 IT Admins)

```powershell
Import-Module ActiveDirectory

# Create IT Admin accounts with unique passwords
$itAdmins = @(
    @{ Name = "itadmin1"; Password = "TempAdmin@123!" },
    @{ Name = "itadmin2"; Password = "TempAdmin@123!" }
)

foreach ($admin in $itAdmins) {

    $securePassword = ConvertTo-SecureString $admin.Password -AsPlainText -Force

    New-ADUser `
        -Name $admin.Name `
        -SamAccountName $admin.Name `
        -UserPrincipalName "$($admin.Name)@lab.local" `
        -AccountPassword $securePassword `
        -Enabled $true `
        -Path "OU=IT-Admins,DC=lab,DC=local"

}

```

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

### Breakdown

- emp1 - emp2 → HR-Group  
- emp3 - emp6 → Finance-Group  
- emp7 - emp23 → Sales-Group
- emp1 - emp23 → Employees-Group
- manager1 - manager5 → Manager-Group
- itadmin1 - itadmin2 → IT-Admin_Group
- IT-Admin_Group → Administrators

--> IT Admins were added to IT-Admins-Group and IT-Admins-Group was added to built-in Administrators group.


This follows **Role-Based Access Control (RBAC)**, where access is assigned based on role and department.

#### Screenshot:
<p align="center">
<img width="500" alt="Screenshot 2026-02-19 at 7 26 17 PM" src="https://github.com/user-attachments/assets/3e3c7034-1e7a-46de-9ea3-40935541f521" />
<img width="500" alt="Screenshot 2026-02-19 at 7 26 26 PM" src="https://github.com/user-attachments/assets/7d9dc164-18bb-4d97-9766-8435d4400c4a" />
</p>

<p align="center">
<img width="500" alt="Screenshot 2026-02-19 at 7 26 32 PM" src="https://github.com/user-attachments/assets/99e20639-7fd4-4555-ba89-4aaf7e40769c" />
<img width="500" alt="Screenshot 2026-02-19 at 7 26 46 PM" src="https://github.com/user-attachments/assets/773d43d5-431a-4e32-983a-cba8c87c1e25" />
</p>

<img width="500" alt="Screenshot 2026-02-19 at 7 26 39 PM" src="https://github.com/user-attachments/assets/4bceeff2-9bdd-476f-bf5a-fea40e4f3c85" />



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

#### Screenshot:
<p align="center">
<img width="500" alt="Screenshot 2026-02-19 at 3 43 10 PM" src="https://github.com/user-attachments/assets/a1fdbe9b-10fb-467e-9b03-e763c20662f7" />
<img width="500" alt="Screenshot 2026-02-19 at 3 43 19 PM" src="https://github.com/user-attachments/assets/cf80f42d-f786-4a16-ad75-3a7530082710" />
</p>

---

### Managers Policy

Managers had fewer restrictions:

- Allowed Command Prompt  
- Allowed Task Manager  
- Limited Control Panel access  
- Still blocked from Registry Editor  

#### Screenshot:
<img width="600" alt="Screenshot 2026-02-19 at 3 43 37 PM" src="https://github.com/user-attachments/assets/6e5f781e-e574-4f01-87d6-9b39df7a91e8" />

---

## 7. Account Lockout Policy

Implemented account lockout policy to mitigate brute force login attempts and reduce risk of credential attacks.

Configured domain policy to:

- Lock account after 5 failed login attempts  
- Set lockout duration to 15 min 
- Reset counter after 15 min


#### Screenshot:
<p align="center">
<img width="500" alt="Screenshot 2026-02-19 at 4 51 48 PM" src="https://github.com/user-attachments/assets/c3c5c738-65c6-4f21-9488-05466be819a0" />
<img width="500" alt="Screenshot 2026-02-19 at 4 52 05 PM" src="https://github.com/user-attachments/assets/c5c401e5-6a76-439c-805f-d909949f77ff" />


---

## 8. IT Support Scenarios Practiced

### Password Reset Scenario

Scenario: **emp10 forgot their password**

Steps:

1. Open Active Directory Users and Computers  
2. Right-click the user  
3. Select **Reset Password**  
4. Force password change at next logon  

#### Screenshot:
<p align="center">
<img width="500" alt="Screenshot 2026-02-19 at 4 00 13 PM" src="https://github.com/user-attachments/assets/eb27eb57-c82d-4029-a32e-b3e31227e8ae" />
<img width="500" alt="Screenshot 2026-02-19 at 4 00 48 PM" src="https://github.com/user-attachments/assets/dd8bab3e-4f7a-45cd-88b1-1032a69d3d72" />
</p>




---


### Disable Account (Terminated Employee)

- Right-click user → Disable Account  
- User can no longer log in  


#### Screenshot:
<p align="center">
<img width="500" alt="Screenshot 2026-02-19 at 4 43 32 PM" src="https://github.com/user-attachments/assets/ce211c98-1786-4fac-9201-14e606428313" />
<img width="500" alt="Screenshot 2026-02-19 at 4 43 54 PM" src="https://github.com/user-attachments/assets/3b4b9552-598c-4462-8de9-3bf82eed97bf" />
</p>

---

### Delete User

- Right-click user → Delete  
- Account permanently removed from the domain  

#### Screenshot:
<p align="center">
<img width="500" alt="Screenshot 2026-02-19 at 4 56 22 PM" src="https://github.com/user-attachments/assets/0a1333ea-1863-4281-ab37-80ebdc344d74" />
<img width="500" alt="Screenshot 2026-02-19 at 4 56 33 PM" src="https://github.com/user-attachments/assets/243ab6a8-9247-4764-93e3-ecf8e27cf69f" />
</p>

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

#### Screenshot:
<img width="600" alt="Screenshot 2026-02-19 at 7 00 58 PM" src="https://github.com/user-attachments/assets/28910bf3-28a0-4450-903f-647ba906491b" />


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


