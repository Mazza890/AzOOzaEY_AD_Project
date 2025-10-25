# AzOOzaEY_AD_Project 
# by Mazza Bushara
Foundational Active Directory setup and security hardening in a virtual lab using Windows Server 2022

## ** Project Objective**

This project demonstrates the setup of a secure and scalable Active Directory (AD) infrastructure for a fictional enterprise, AzOOzaEY, using Windows Server 2022 and Windows 11 Pro in a virtual lab. It is designed to help IT students and junior administrators understand the foundational steps of domain creation, OU design, user/group management, GPO deployment, file sharing, and security auditing.

##   Lab Environment
### Tools Used
• 	Oracle VirtualBox (virtualization)

• 	Windows Server 2022 ISO (Domain Controller)

• 	Windows 11 Pro ISO (Client Machine)

• 	Group Policy Management Console (GPMC)

• 	File Server Resource Manager (FSRM)

• 	PingCastle (AD security audit)

![VirtualBox Setup](Screenshots/VirtualBox_Setup.png)
![VM List](Screenshots/VM_List.png)



 ### Configuration
• 	Internal Network setup in VirtualBox

• 	Server static IP: 10.0.2.15

• 	DNS: loopback 127.0.0.1 , alternate 8.8.8.8

• 	Client IP: manually set  to 10.0.2.20 

• 	DNS on client: pointed to Domain Controller IP, which is 10.0.2.15

• 	Verified connectivity using  Ping  and  nslookup

### 🛠️ Domain Promotion

- Installed AD DS role
- Promoted server to Domain Controller
- Domain name: `mazza.local`

![Domain Promotion](Screenshots/Domain_Promotion.png)

##  Why Promote to Domain Controller

Promoting the server to a Domain Controller transforms it from a basic server into the central authority for identity management within your network. This is the foundation of Active Directory.

 **Key Benefits**
- **Authentication & Authorization**: Manages logins and access to resources across the domain.
- **Group Policy Management**: Enforces security settings and user configurations.
- **Directory Services**: Organizes users, computers, and groups in a scalable structure.
- **DNS Integration**: Enables name resolution critical for domain operations.

🔧 **What Changes After Promotion**

| Before Promotion     | After Promotion                     |
|----------------------|-------------------------------------|
| Standalone Server    | Domain Controller (AD DS role)      |
| No user database     | Centralized user/group management   |
| No GPO control       | Full Group Policy capabilities      |
| No domain join       | Clients can join the domain         |


  ###  OU Structure for AzOOzaEY

#### Top-Level OUs
- Africa
- USA
- Asia
- Australia

>  Each top-level OU follows the same internal structure for consistency and scalability.

#### Sub-OUs for Each Region

| Sub-OU Name | Purpose               | Example Path                  |
|-------------|------------------------|-------------------------------|
| Users       | Domain users           | Africa > Users                |
| Computers   | Client machines        | Africa > Computers            |
| Servers     | Infrastructure roles   | Africa > Servers              |

![OU Structure](Screenshots/OU_Structure.png)

#### Example OU Paths
- Africa > Users  
- Africa > Computers  
- Africa > Servers  
- USA > Users  
- USA > Computers  
- USA > Servers  
- Asia > Users  
- Asia > Computers  
- Asia > Servers  
- Australia > Users  
- Australia > Computers  
- Australia > Servers

### 👥 User and Group Management

#### User Creation
- Created users for each department and region (e.g., `Mohammed Hamedo`, `jdonor`, `fteam1`)
- Assigned users to appropriate OUs based on geography and role
- Ensured naming consistency and placement in:
  - Africa > Users
  - USA > Users
  - Asia > Users
  - Australia > Users
![User Creation](Screenshots/User_Creation.png)
![Group Creation](Screenshots/Group_Creation.png)


#### Group Creation

| Group Name     | Scope   | Type         | OU Location         | Purpose                                      |
|----------------|---------|--------------|----------------------|----------------------------------------------|
| IT             | Global  | Security     | Africa > Users       | Assign permissions to IT staff               |
| DL-ITAdmins    | Global  | Distribution | Africa > Users       | Email distribution list for IT administrators |
| HR             | Global  | Security     | USA > Users          | Access control for HR department             |
| Finance        | Global  | Security     | Asia > Users         | Permissions for finance-related resources    |
| DL-FinanceTeam | Global  | Distribution | Asia > Users         | Email list for finance team                  |

>  Security groups were used for access control.  
>  Distribution groups were used for email communication.

Group Scope & Type Summary
- Global: Used across the domain
- Security: Grants access rights
- Distribution: Used for email lists

 Computer Management
- Joined Windows 11 Pro client to mazza.local
- Moved client object to appropriate OU (e.g., Africa > Computers)
- Verified domain login using domain user credentials

###  Group Policy Objects (GPOs)

Group Policy Objects (GPOs) were created and linked to each top-level OU (Africa, USA, Asia, Australia) to enforce consistent security and user experience across the domain.

#### GPOs Applied Across All Regional OUs

| GPO Name                  | Type                | Target OU       | Purpose                                                   | Notes                            |
|---------------------------|---------------------|------------------|------------------------------------------------------------|----------------------------------|
| Password Policy           | Computer Policy     | Computers OU     | Enforce strong password requirements                       | Minimum length, complexity rules |
| Drive Mapping             | User Configuration  | Users OU         | Map M:\ to `SharedEY` folder                               | Persistent via GPO               |
| Desktop Wallpaper         | User Policy         | Users OU         | Set default wallpaper and prevent changes                  | Black wallpaper issue pending    |
| Restricted Control Panel  | User Policy         | Users OU         | Block access to Control Panel                              | Prevents unauthorized changes    |
| Disable USB Storage       | Computer Policy     | Computers OU     | Deny access to removable storage devices                   | Security hardening               |
| Account Lockout Policy    | Computer Policy     | Computers OU     | Lock account after 5 failed login attempts (10-min lockout)| Brute-force protection           |
| Logon Message             | Computer Policy     | Computers OU     | Display legal notice at login                              | Optional compliance feature      |

 All GPOs were tested using `gpupdate /force` and verified on Windows 11 Pro domain clients.  
  Same GPO structure applied to each top-level OU for consistency.

 ![GPO Editor](Screenshots/GPO_Editor.png)
![Drive Mapping GPO](Screenshots/Drive_Mapping_GPO.png)


 ###  GPO Testing

After creating and linking Group Policy Objects (GPOs), policies were tested to ensure they applied correctly across all regional OUs.

#### Steps Taken
- Ran `gpupdate /force` on Windows 11 Pro client to apply policies immediately
- Verified effects:
  - Control Panel access blocked
  - M:\ drive automatically mapped to `SharedEY`
  - USB storage disabled
  - Desktop wallpaper applied (black screen issue pending)

![GPO Update](Screenshots/GPO_Update.png)
![Mapped Drive](Screenshots/Mapped_Drive.png)
![Control Panel Blocked](Screenshots/ControlPanel_Blocked.png)


### File Sharing and Permissions

#### Shared Folder Setup
- Created `C:\SharedEY` on the Domain Controller
- **Share Permissions**:
  - `Domain Users` → Read
- **NTFS Permissions**:
  - Assigned Modify or Read access to specific users and groups
  - 
![Shared Folder](Screenshots/SharedEY_Properties.png)
![NTFS Permissions](Screenshots/NTFS_Permissions.png)


#### Sharing Methods

| Method           | Drive Letter | Description                                      |
|------------------|--------------|--------------------------------------------------|
| Manual Mapping   | M:           | Temporary mapping via File Explorer              |
| GPO Drive Mapping| M:           | Persistent mapping applied through Group Policy  |

 NTFS permissions control actual file access  
 Share permissions control visibility over the network  
 GPO-based mapping ensures consistency across all domain users


###  File Server Resource Manager (FSRM)

FSRM was configured to manage storage usage and enforce file screening policies on the shared folder `C:\SharedEY`.

#### Configuration Steps
- Installed FSRM via Server Manager
- **Quota Management**:
  - Applied quota to `C:\SharedEY`
  - Set maximum storage limit to control usage
- **File Screening**:
  - Blocked file types: `.mp3`, `.mp4`, `.jpg`, `.zip`

#### Purpose
- Prevent unnecessary storage usage
- Notify users when nearing quota thresholds
- Reduce hardware costs by managing space efficiently


###  PingCastle Security Audit

PingCastle was used to perform a security health check of the AzOOzaEY Active Directory domain.

#### Audit Steps
- Ran PingCastle on the Domain Controller
- Generated full Health Check Report
- Transferred report to the host machine for review
- Validated AD setup and identified hardening opportunities

#### Purpose
- Assess AD security posture
- Identify misconfigurations or vulnerabilities
- Support future hardening and compliance efforts
- 
![PingCastle Scan](Screenshots/PingCastle_Scan.png)


#### Report Access
The full PingCastle report is available in the repository as:

 [`ad_hc_mazza.local`](ad_hc_mazza.local)

> This file contains the complete health check results, including risk scores, domain configuration analysis, and recommended security improvements.


 ###  Troubleshooting & Fixes

Throughout the project, several issues were encountered and resolved to ensure a stable and functional Active Directory environment.

#### 1. Client Could Not Join Domain (DNS SRV Record Error)
- **Issue**: “An Active Directory Domain Controller for the domain ‘mazza.local’ could not be contacted.”
- **Fix**: Set the client’s DNS to `10.0.2.15` (Domain Controller IP)
- **Verification**: Confirmed with `ping` and `nslookup mazza.local`

#### 2. GPOs Not Applying Immediately
- **Issue**: Group Policy settings were not reflected on the client
- **Fix**: Ran `gpupdate /force` on the client machine
- **Result**: Policies applied instantly (e.g., drive mapping, Control Panel restriction)

#### 3. Network Settings Would Not Open
- **Issue**: Clicking “Network & Internet Settings” caused the window to flash and close
- **Fix**: Disabled conflicting GPOs temporarily (Control Panel restriction, USB block, lockout policy)
- **Result**: Settings became accessible again

#### 4. Black Desktop Wallpaper
- **Issue**: GPO applied wallpaper setting, but the desktop appeared black
- **Fix**: Pending — next step is to verify wallpaper file path and permissions

#### 5. Client IP Conflict
- **Issue**: DHCP-assigned IP caused inconsistent connectivity
- **Fix**: Manually assigned static IP 10.0.2.20
- **Result**: Stable connectivity and successful domain join
 
 ![DNS Fix](Screenshots/DNS_Fix.png)
![Static IP](Screenshots/Static_IP.png)
![GPO Conflict](Screenshots/GPO_Conflict.png)




