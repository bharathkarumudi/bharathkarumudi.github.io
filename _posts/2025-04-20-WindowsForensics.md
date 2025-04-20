---
title: 'What Windows Remembers: A Forensics Perspective'
date: 2025-04-20 00:00:00
featured_image: '/images/posts/dfir-windows-featured.png'
excerpt: Every click, file, and device leaves a footprint—and Windows remembers. In the world of Digital Forensics and Incident Response (DFIR), analysts rely on the Windows Registry and system artifacts to piece together what happened, when it occurred, and who was involved.
---

![](/images/posts/dfir-windows.png){:height="40%" width="40%"}  
  
[Image](): THM 
  

# Windows Forensics 

## Introduction
In the world of DFIR, Windows Operating System is one of the richest sources of forensic data. The operating system quietly records a wealth of information - from user activity to device history - across its registry and file systems. This post walks through the key forensic artifacts and tools that help investigators piece together what happened, when, and how.  

### Why Windows Registry Matters in Forensics
The Windows Registry is a goldmine of information for forensic investigators. It stores records of user activities, system configurations, and application executions, making it a valuable source of digital evidence. Understanding how to extract and analyze this data is critical for uncovering security incidents and malicious activities.

## Forensic Artifacts
Forensic artifacts are essential pieces of digital evidence that provide insight into user and system activities. These artifacts leave small traces on a system that can be analyzed to reconstruct past actions, such as file access, program execution, and network connections. Investigators use these artifacts to understand user behavior and system state at a given time.

---
## Windows Registry and Forensics
The Windows Registry is a hierarchical database that stores configuration settings and options for the operating system and installed applications. It acts as a central repository for user preferences, system settings, and hardware configurations. Understanding the registry is essential for forensic investigations, as it contains valuable information about user activity and system changes.

### What is a Registry Hive?
A registry hive is a logical group of keys, subkeys, and values that are stored in a single file on disk. These hives help organize system and user settings in a structured way. Forensic analysts often extract and analyze these hives to uncover important digital evidence.

### Structure of the Registry
The following table summarizes the key registry hives and their functions:

| Ref | Registry Hive         | Abbreviation | Description |
|-----|----------------------|--------------|-------------|
| RH-1 | HKEY_CURRENT_USER   | HKCU         | Stores configuration settings for the currently logged-in user. |
| RH-2 | HKEY_USERS          | HKU          | Contains actively loaded user profiles on the computer. |
| RH-3 | HKEY_LOCAL_MACHINE  | HKLM         | Holds configuration settings specific to the computer. |
| RH-4 | HKEY_CLASSES_ROOT   | HKCR         | Ensures the correct program opens when launching a file. |
| RH-5 | HKEY_CURRENT_CONFIG | HKCC         | Stores the hardware profile used at system startup. |

### Commonly Used Registry Keys
Registry keys provide valuable forensic data that can help analysts track user activity, program execution, and device connections. Below are some critical registry locations used in forensic investigations:

| Ref   | Registry Key               | Forensic Relevance                                                      | Location                                                                                |
|-------|----------------------------|-------------------------------------------------------------------------|-----------------------------------------------------------------------------------------|
| RK-1  | OS Version                 | Identifies the Windows version and build number.                        | `SOFTWARE\Microsoft\Windows NT\CurrentVersion`                                          |
| RK-2  | Control Set Configurations | Stores machine configuration data for system startup.                   | `SYSTEM\CurrentControlSet\`                                                             |
| RK-3  | Past Network Connections   | Tracks previously connected networks.                                   | `SOFTWARE\Microsoft\Windows NT\CurrentVersion\NetworkList\Signatures\`                  |
| RK-4  | User Account Info          | Contains login and group membership details.                            | `SAM\Domains\Account\Users`                                                             |
| RK-5  | Recent Files               | Tracks recently accessed files.                                         | `NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\Explorer\RecentDocs`              |
| RK-6  | Office Files               | Stores usage data for Microsoft Office applications.                    | `NTUSER.DAT\Software\Microsoft\Office\`                                                 |
| RK-7  | O365 File MRU              | Lists recently opened Office 365 files.                                 | `NTUSER.DAT\Software\Microsoft\Office\VERSION\UserMRU\LiveID_####\FileMRU`              |
| RK-8  | UserAssist                 | Records applications launched by the user (GUI-based).                  | `NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\Explorer\UserAssist\{GUID}\Count` |
| RK-9  | ShimCache                  | Logs execution history of applications.                                 | `SYSTEM\CurrentControlSet\Control\Session Manager\AppCompatCache`                       |
| RK-10 | AmCache                    | Similar to ShimCache but also stores SHA-1 hashes of executed programs. | `C:\Windows\appcompat\Programs\Amcache.hve`                                             |
| RK-11 | BAM/DAM Monitoring         | Tracks application background activity.                                 | `SYSTEM\CurrentControlSet\Services\bam\UserSettings\{SID}`                              |
| RK-12 | USB Devices                | Stores information about connected USB storage devices.                 | `SYSTEM\CurrentControlSet\Enum\USBSTOR`                                                 |
| RK-13 | USB Connection Timestamps  | Records first connection, last connection, and removal timestamps.      | `SYSTEM\CurrentControlSet\Enum\USBSTOR\Ven_Prod_Version\USBSerial#\Properties\`         |
| RK-14 | USB Device Volume Name     | Stores volume names of USB devices.                                     | `SOFTWARE\Microsoft\Windows Portable Devices\Devices`                                   |


---
## Exploring the Windows Registry
Tools to view and analyze registry content:
- **AccessData Registry Viewer** – Views one hive at a time.
- **Zimmerman’s Registry Explorer** – Supports transaction logs and multiple hives.
- **RegRipper** – Extracts registry-based forensic data with plugins.

---
## System Information and Accounts
- Refer to **RK-1, RK-2, RK-3, RK-4** for OS, network history, and user account info.

---
## Evidence of File and Application Usage
- Refer to **RK-5 through RK-11** to trace file access and execution of applications.

---
## Real-World Use Case: Tracing Unauthorized Access
Investigate unauthorized access using:
- **RK-4 (User Account Info)** – Who logged in?
- **RK-5 (Recent Files)** – What was opened?
- **RK-8 (UserAssist)** – What was executed?

---
## Best Practices for Registry Analysis
- Work on copies of registry hives.
- Use multiple tools for cross-verification.
- Combine registry data with event logs.

---
## File System & Artifact-Based Forensics

### File Systems Overview
#### FAT & exFAT:
- FAT12, FAT16, FAT32 – Legacy file systems.
- exFAT – Supports files up to 128 PB. Common in SD cards.

#### NTFS:
- Default Windows file system since XP.
- Supports:
  - Journaling
  - Access Control Lists (ACLs)
  - Shadow Copy
  - Alternate Data Streams (ADS)
  - Master File Table (MFT)
    - Important metadata files:
      - `$MFT`, `$LOGFILE`, `$UsnJrnl`

- **Tool:** `MFTECmd.exe -f <MFT> --csv <output>`

### Recovering Deleted Files
- **Disk Images**: Bit-by-bit copies of drives.
- **Autopsy Tool**: Load images, recover deleted data.

### Additional Execution Artifacts
#### Windows Prefetch
- Stored in `C:\Windows\Prefetch`, `.pf` extension.
- Shows last run time, count, associated files.
- **Tool:** `PECmd.exe -f <prefetch> --csv <output>`

#### Windows Timeline
- SQLite DB:  
  `C:\Users\<username>\AppData\Local\ConnectedDevicesPlatform\{randomfolder}\ActivitiesCache.db`
- Stores recent app use, focus time.
- **Tool:** `WxTCmd.exe -f <file> --csv <output>`

#### Jump Lists
- Stores recent file usage by apps.
- Path: `C:\Users\<username>\AppData\Roaming\Microsoft\Windows\Recent\AutomaticDestinations`
- **Tool:** `JLECmd.exe -f <file> --csv <output>`

### File/Folder Metadata
#### Shortcut (.lnk) Files
- Logs file paths and open times.
- Paths:  
  - `C:\<>\AppData\Roaming\Microsoft\Windows\Recent\`
  - `C:\<>\AppData\Roaming\Microsoft\Office\Recent\`
- **Tool:** `LECmd.exe -f <file> --csv <output>`

#### IE/Edge History
- Location:  
 `C:\<>\AppData\Local\Microsoft\Windows\WebCache\WebCacheV*.dat`
- Tool: **Autopsy**

### USB Setup Logs
- `C:\Windows\inf\setupapi.dev.log` – Setup details of connected USB devices.

---
## Conclusion
From registry keys to shortcut files, Windows forensics offers a wealth of insight for digital investigators. With the right tools and an understanding of where to look, you can uncover valuable evidence and build stronger timelines. Got thoughts or questions? I’d love to hear how others are approaching Windows artifact analysis. 


