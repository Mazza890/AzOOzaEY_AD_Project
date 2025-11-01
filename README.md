# AzOOzaEY_AD_Project 
# by Mazza Bushara
Foundational Active Directory setup and security hardening in a virtual lab using Windows Server 2022

##  Project Objective

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

###  Domain Promotion

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

###  User and Group Management

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

## DHCP and Network Layout – mazza.local Lab 

### Overview
This document outlines how DHCP, DNS, and static IPs are configured in the virtual lab for the mazza.local domain.
IP Plan

The router uses IP address 10.0.2.1 and serves as the default gateway.
The DNS server is located at 10.0.2.15 and also functions as the domain controller.
The DHCP scope assigns dynamic IPs from 10.0.2.100 to 10.0.2.200.
Static IPs are reserved outside this range to avoid conflicts.
The HP LaserJet 6MP printer is reserved at 10.0.2.80 using its MAC address f20000ada4d5.

### DHCP Scope Options
Option 003 (Router): 10.0.2.1
Option 006 (DNS Server): 10.0.2.15
Option 015 (DNS Domain Name): mazza.local
These options ensure that clients receive the correct gateway, DNS server, and domain suffix when they obtain an IP address.

### DHCP Reservation
A reservation was created for the HP LaserJet 6MP printer.
Reservation name: HPLaserJet_6MP
IP address: 10.0.2.80
MAC address: f20000ada4d5
This ensures the printer always receives the same IP address from DHCP. 

#### Notes
Static infrastructure devices like the router and DNS server are assigned IPs outside the DHCP scope.
The DHCP scope is only used for domain-joined clients and other dynamic devices.
Reservations help maintain consistent IPs for devices that need reliable access, such as printers or servers.

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


 

## DHCP configuration 
I installed and configured the DHCP Server role on Windows Server to support dynamic IP address assignment in the AzOOzaEY lab. The server was authorized in Active Directory, and a new IPv4 scope was created with the following configuration:
• 	IP range: 10.0.2.100  to  10.0.2.200
• 	Default gateway: 10.0.2.1
• 	DNS server: 10.0.2.15
• 	Excluded range:  10.0.2.150 to 10.0.2.160
To ensure consistent IP assignment for key devices, I added a reservation named HPLaserJet_6MP with the IP address 10.0.2.80  and f20000ada4d5 MAC address. This ensures that the printer always receives the same IP address from the DHCP server.
This DHCP configuration integrates with DNS and Group Policy services already deployed in the lab, supporting reliable connectivity for domain-joined clients and enhancing overall network management.

## Roaming User Profiles Deployment 

### Roaming User Profiles (Africa > Users OU)

- Created a dedicated file share: \\MB-Project260\Roaming_profiles$
- Applied GPO: Roaming User Profile Settings to Africa/Users OU
- Profile path: \\MB-Project260\Roaming_Profiles$\%username%
- Verified roaming status on Windows 11 Pro client (Client800-869) via Control Panel > System > User Profiles


This roaming profile deployment builds on the foundational Active Directory structure already implemented in the AzOOzaEY lab. The Africa  OU contains a  Users sub-OU, where roaming profiles were configured for selected domain users. A dedicated file share named Roaming_Profiles$ was created on MB-Project260 with explicit permissions to support secure profile storage. This share is isolated from redirected folders to prevent caching conflicts and ensure profile integrity.

To enforce roaming behavior, I created a Group Policy Object titled "Roaming User Profile Settings" and linked it to the "Africa/Users" OU. The GPO specifies the roaming path using the  %username%  variable, allowing each user’s profile to be stored in a unique subfolder within the share. Inheritance was disabled on the share, and permissions were customized to meet best practices for roaming profile access.

Testing was performed on a Windows 11 Pro client (Client800-869) joined to the domain. After signing in with a configured user account and running gpupdate/force, I verified that the profile type was listed as " Roaming "under Control Panel > System > User Profiles. This confirms that the GPO and file share integration are functioning as expected.
This feature enhances centralized management and supports user mobility across domain-joined devices. It complements other Group Policy configurations already documented in the project, such as folder redirection and security hardening. Future updates may include quota enforcement, FSRM integration, or primary computer targeting for roaming profiles.

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

