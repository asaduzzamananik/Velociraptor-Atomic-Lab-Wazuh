# Velociraptor-Atomic-Lab-Wazuh

Complete End-to-End Lab: Simulating Linux MITRE ATT&amp;CK with Atomic Red Team, Velociraptor &amp; Wazuh

---

## Credits & Original POC Reference

This project was inspired by a proof-of-concept video by **Taylor Walton** :

🎥 **Video Title:**  
[Simulate Linux Attacks and Tune Detection Rules with Atomic Red Team](https://www.youtube.com/watch?v=yll8-yqVv0w)

Referenced Tools:
- Atomic Red Team
- Velociraptor DFIR
- Wazuh Security Platform
- PowerShell Core for Linux

---

## Project Objective
This open-source security lab demonstrates how to simulate MITRE ATT&CK techniques on Linux and detect them using real-world blue team tooling. The goal is to help learners understand attacker behavior and validate detection using modern EDR and DFIR frameworks.

Key Goals:
  - Understand and simulate MITRE ATT&CK behavior using Atomic Red Team (e.g., T1059.004)
  - Detect adversarial activity with Wazuh SIEM/EDR
  - Hunt and collect artifacts using Velociraptor
  - Provide a fully documented and beginner-friendly guide with tooling explanations

This project helps bridge the gap between offensive simulation and defensive monitoring through hands-on practice

---

 ## Network Architecture
 
 ```text
[ Ubuntu Server ]
  ├─ Wazuh Manager (SIEM/EDR)
  └─ Velociraptor Server (Forensics)

[ Kali Linux ]
  ├─ Wazuh Agent
  ├─ Velociraptor Client
  └─ Atomic Red Team Testbed
```
---
  

## Virtual Machines (VirtualBox)
| **VM NAME**  | **Network Adapter** | **Purpose** | **Tools Used** |
|---------------|-------------|---------------|---------------|
| **Ubuntu**    | **Adapter 1: NAT Network**  | Wazuh Manager + Velociraptor  | Detection, logging, Wazuh , Velociraptor) |             
| **Kali Linux** | **Adapter 1: NAT Network**   | Test Agent (Attacker) | Simulate techniques using Atomic Red Team |

---


**Network Settings for All VMs**

Select Adapter 1:

Check Enable Network Adapter
Attached to: NAT Network
Adapter Type: Intel PRO/1000 MT Desktop (default is fine)
Cable Connected: Checked


Repeat the same for Kali Linux And Ubuntu

## Ubuntu Network Config
<img width="717" height="569" alt="image" src="https://github.com/user-attachments/assets/aaec7cc4-c0ec-461a-8659-5b108470ca00" />

## kali Network Config
<img width="712" height="594" alt="image" src="https://github.com/user-attachments/assets/585d6356-2ad3-402b-a0de-1e1ed46aca11" />

---

## Tool Overview: Why Each Tool Was Used

| **Tool**  | **What It Is** | **Why It Was Used** | 
|---------------|-------------|---------------|
| **Wazuh** | Open-source XDR/SIEM platform  | To detect attacker behavior via log analysis and provide alerting/visuals |
| **Velociraptor** | DFIR (Digital Forensics and Incident Response) Framework | To collect and query forensic data on endpoints |
| **Atomic Red Team** | Library of scripted adversary techniques from MITRE ATT&CK | To simulate real-world attack behaviors in a controlled lab environment |
| **PowerShell** | Cross-platform version of Windows PowerShell | Required to execute Atomic Red Team techniques written in PowerShell |

---

## Lab Setup 

## Ubuntu Server Phase (Wazuh + Velociraptor)

### Phase 1: Install Wazuh Manager

This step sets up the main SIEM/XDR system for alerting.

```bash
curl -sO https://packages.wazuh.com/4.7/wazuh-install.sh
sudo bash ./wazuh-install.sh -a
```

**⚠️ Note: In this lab, Wazuh was installed using this community repo for ease: https://github.com/samiul008ghub/soc_setup**

Dashboard: https://<your_server_ip>:5601

<img width="1809" height="884" alt="image" src="https://github.com/user-attachments/assets/28c80240-f939-4088-9ab6-eaf4f6c353b3" />

---

### Phase 2: Install Velociraptor Server

```bash
wget https://docs.velociraptor.app/velociraptor-v0.74.5-linux-amd64 -O velociraptor
chmod +x velociraptor
sudo mv velociraptor /usr/local/bin/velociraptor
```
Then Run:

```bash
cd /etc
velociraptor config generate > velociraptor.config.yaml
velociraptor --config velociraptor.config.yaml frontend
```

Dashboard: https://<IP>:8000

<img width="808" height="69" alt="image" src="https://github.com/user-attachments/assets/aaa297bd-79f8-4d75-af91-64043e35240b" />

<img width="1854" height="928" alt="image" src="https://github.com/user-attachments/assets/535aaec6-b93e-465d-8610-4f6f489e3942" />

**For Login credentials**
```bash
velociraptor --config server.config.yaml user add admin --role administrator
```
It will prompt you to enter and confirm a password.

Then run Velociraptor again:

```bash
sudo pkill velociraptor
velociraptor --config velociraptor.config.yaml frontend
```

Open your browser and go to:

```bash
https://localhost:8000
or 
https://<your-server-ip>:8000
```

Log in with:
  - Username: admin
  - Password: the one you just created

<img width="1850" height="890" alt="image" src="https://github.com/user-attachments/assets/c00ce3bf-00d9-4842-89d4-1967645a5f5c" />

---

## Phase 3:
## Prepare Kali Linux Attack Host

**1. Install PowerShell on Linux**
```bash
###################################
# Prerequisites

# Update the list of packages
sudo apt-get update

# Install pre-requisite packages.
sudo apt-get install -y wget

# Get the version of Debian
source /etc/os-release

# Download the Microsoft repository GPG keys
wget -q https://packages.microsoft.com/config/debian/$VERSION_ID/packages-microsoft-prod.deb

# Register the Microsoft repository GPG keys
sudo dpkg -i packages-microsoft-prod.deb

# Delete the Microsoft repository GPG keys file
rm packages-microsoft-prod.deb

# Update the list of packages after we added packages.microsoft.com
sudo apt-get update

###################################
# Install PowerShell
sudo apt-get install -y powershell

# Start PowerShell
pwsh
```

---

**2. Installing Invoke AtomicRedTeam**
**Install Execution Framework Only**

```bash
Install-Module -Name invoke-atomicredteam,powershell-yaml -Scope CurrentUser
```

Then run :

```bash
IEX (IWR 'https://raw.githubusercontent.com/redcanaryco/invoke-atomicredteam/master/install-atomicredteam.ps1' -UseBasicParsing);
Install-AtomicRedTeam -getAtomics -Force
```

Exit Powershell and paste : 
```bash apt-get install git -y```

Run a very basic test for confirming Atomid Red is Running:

```bash
pwsh -exec bypass -Command "Import-Module "/usr/local/share/powershell/Modules/Invoke-AtomicRedTeam/Invoke-AtomicRedTeam.psd1" -Force; Invoke-AtomicTest T1005 -TestNumbers 2 -GetPreReqs; Invoke-AtomicTest T1005 -TestNumbers 2 -ExecutionLogPath /tmp/ARTExec.csv;"
```

---

## Setup Velociraptor Client In Kali

**1. Install client on Kali**

Download Velociraptor , rename and chmod:
```bash
wget https://docs.velociraptor.app/velociraptor-v0.74.5-linux-amd64 -O velociraptor_client
chmod +x velociraptor_client
sudo mv velociraptor /usr/local/bin/velociraptor_client
```

**2. Configure client**

Here, we Generate client config In **Ubuntu** which is velociraptor server and then copy it in **Kali machine** :
```bash
cd /etc
velociraptor --config velociraptor.config.yaml config client > client.config.yaml
```
Move **client config file** to Kali Linux via **SCP**
```bash
scp client.config.yaml root@<your ip>:/root/
```

Go to kali linux and paste : 
```bash
mv client.config.yaml /etc/velociraptor.config.yaml
```

**Config Velociraptor client in kali**
Edit systemd to run as root :
```bash
sudo nano /etc/systemd/system/velociraptor.service
```
And paste this :

```bash
[Unit]
Description=Velociraptor Client

[Service]
ExecStart=/usr/local/bin/velociraptor --config /etc/velociraptor.config.yaml client
User=root
Restart=always

[Install]
WantedBy=multi-user.target
```

**Enable & start Velociraptor Client:**

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now velociraptor
```
Now Velociraptor client is set .
<img width="1807" height="397" alt="image" src="https://github.com/user-attachments/assets/0ed37a4a-555e-4f21-a76e-5a0ee1a0e510" />

---

## Phase 4: Create Custom Artifact in Velociraptor GUI

1. Import the Linux simulation artifact (https://raw.githubusercontent.com/socfortress/VELOCIRAPTOR-ATOMIC-RED-ARTIFACTS/refs/heads/main/Linux.AttackSimulation.AtmoticRedTeam.yaml):
Upload: GUI → Artifacts → Upload → Paste YAML → Save.

Velociraptor Artifact: 

```bash
name: Linux.AttackSimulation.AtomicRedTeam
author: SOCFortress LLC
description: |
    This artifact allows you to run Atomic Red Team tests on Linux
    endpoints using Invoke-AtomicRedTeam.

    NOTE: You first need to install powershell on your linux machine:
    https://learn.microsoft.com/en-us/powershell/scripting/install/install-debian?view=powershell-7.5
    
    git clone https://github.com/redcanaryco/invoke-atomicredteam.git /usr/local/share/powershell/Modules/Invoke-AtomicRedTeam
    
    Import-Module Invoke-AtomicRedTeam
    
    

    **Reference:**

    https://github.com/redcanaryco/invoke-atomicredteam

    **Description:**

    Invoke-AtomicRedTeam is a PowerShell module to execute tests as
    defined in the atomics folder of Red Canary's Atomic Red Team
    project. The "atomics folder" contains a folder for each Technique
    defined by the MITRE ATT&CK™ Framework. Inside of each of these
    "T#" folders you'll find a yaml file that defines the attack
    procedures for each atomic test as well as an easier to read
    markdown (md) version of the same data.

    - Executing atomic tests may leave your system in an undesirable
      state. You are responsible for understanding what a test does
      before executing.

    - Ensure you have permission to test before you begin.

    - It is recommended to set up a test machine for atomic test
      execution that is similar to the build in your environment. Be
      sure you have your collection/EDR solution in place, and that
      the endpoint is checking in and active.

type: CLIENT

column_types:
  - name: Technique
    type: safe_url

parameters:
    - name: ExecutionLogFile
      description: Path to log file (CSV) for executions by ART tests
      default: /tmp/ARTExec.csv
      
    - name: EnsureExecLog
      description: Create ExecutionLogFile if it does not exist first
      default: Y
      type: bool

    - name: RemoveExecLog
      description: Remove execution log before running artifact (in the event we don't want to intertwine results from previous tests)
      default: Y
      type: bool

    - name: Cleanup
      description: Clean up execution artifacts
      default: Y
      type: bool

    - name: RunAll
      description: NOT RECOMMENDED...USE WITH CAUTION - Run all ART tests
      default: N
      type: bool

    - name: T1580 - 1
      description: AWS - EC2 Enumeration from Cloud Instance
      type: bool
    - name: T1580 - 2
      description: AWS - EC2 Security Group Enumeration
      type: bool
    - name: T1574.006 - 1
      description: Shared Library Injection via /etc/ld.so.preload
      type: bool
    - name: T1574.006 - 2
      description: Shared Library Injection via LD_PRELOAD
      type: bool
    - name: T1562.012 - 1
      description: Delete all auditd rules using auditctl
      type: bool
    - name: T1562.012 - 2
      description: Disable auditd using auditctl
      type: bool
    - name: T1562.008 - 4
      description: AWS - Disable CloudTrail Logging Through Event Selectors using Stratus
      type: bool
    - name: T1562.008 - 5
      description: AWS - CloudTrail Logs Impairment Through S3 Lifecycle Rule using Stratus
      type: bool 
    - name: T1562.008 - 6
      description: AWS - Remove VPC Flow Logs using Stratus
      type: bool  
    - name: T1556.003 - 1
      description: Malicious PAM rule
      type: bool
    - name: T1552.003 - 2
      description: Malicious PAM rule (freebsd)
      type: bool
    - name: T1552.003 - 3
      description: Malicious PAM module
      type: bool
    - name: T1552.007 - 3
      description: Cat the contents of a Kubernetes service account token file
      type: bool
    - name: T1552.003 - 1
      description: Search Through Bash History
      type: bool
    - name: T1552.003 - 2
      description: Search Through sh History
      type: bool
    - name: T1548.003 - 1
      description: Sudo usage
      type: bool
    - name: T1548.003 - 2
      description: Sudo usage (freebsd)
      type: bool
    - name: T1548.003 - 3
      description: Unlimited sudo cache timeout
      type: bool
    - name: T1548.003 - 4
      description: Unlimited sudo cache timeout (freebsd)
      type: bool
    - name: T1548.003 - 5
      description: Disable tty_tickets for sudo caching
      type: bool
    - name: T1548.003 - 6
      description: Disable tty_tickets for sudo caching (freebsd)
      type: bool
    - name: T1548.001 - 1
      description: Make and modify binary from C source
      type: bool
    - name: T1548.001 - 2
      description: Make and modify binary from C source (freebsd)
      type: bool
    - name: T1548.001 - 3
      description: Set a SetUID flag on file
      type: bool
    - name: T1548.001 - 4
      description: Set a SetUID flag on file (freebsd)
      type: bool
    - name: T1548.001 - 5
      description: Set a SetGID flag on file
      type: bool
    - name: T1548.001 - 6
      description: Set a SetGID flag on file (freebsd)
      type: bool
    - name: T1548.001 - 7
      description: Make and modify capabilities of a binary
      type: bool
    - name: T1548.001 - 8
      description: Provide the SetUID capability to a file
      type: bool
    - name: T1548.001 - 9
      description: Do reconnaissance for files that have the setuid bit set
      type: bool
    - name: T1548.001 - 10
      description: Do reconnaissance for files that have the setgid bit set
      type: bool
    - name: T1547.006 - 1
      description: Linux - Load Kernel Module via insmod
      type: bool
    - name: T1546.005 - 1
      description: Trap EXIT
      type: bool
    - name: T1546.005 - 2
      description: Trap EXIT (freebsd)
      type: bool
    - name: T1546.005 - 3
      description: Trap SIGINT
      type: bool
    - name: T1546.005 - 4
      description: Trap SIGINT (freebsd)
      type: bool
    - name: T1546.004 - 1
      description: Add command to .bash_profile
      type: bool
    - name: T1546.004 - 2
      description: Add command to .bashrc
      type: bool   
    - name: T1546.004 - 3
      description: Add command to .shrc
      type: bool
    - name: T1546.004 - 4
      description: Append to the system shell profile
      type: bool 
    - name: T1546.004 - 5
      description: Append commands user shell profile
      type: bool 
    - name: T1546.004 - 6
      description: System shell profile scripts
      type: bool 
    - name: T1546.004 - 7
      description: Create/Append to .bash_logout
      type: bool 
    - name: T1543.002 - 1
      description: Create Systemd Service
      type: bool
    - name: T1543.002 - 2
      description: Create SysV Service
      type: bool
    - name: T1543.002 - 3
      description: Create Systemd Service file, Enable the service , Modify and Reload the service.
      type: bool
    - name: T1222.002 - 1
      description: chmod - Change file or folder mode (numeric mode)
      type: bool
    - name: T1222.002 - 2
      description: chmod - Change file or folder mode (symbolic mode)
      type: bool   
    - name: T1222.002 - 3
      description: chmod - Change file or folder mode (numeric mode) recursively
      type: bool      
    - name: T1222.002 - 4
      description: chmod - Change file or folder mode (symbolic mode) recursively
      type: bool    
    - name: T1222.002 - 5
      description: chown - Change file or folder ownership and group
      type: bool    
    - name: T1222.002 - 6
      description: chown - Change file or folder ownership and group recursively
      type: bool      
    - name: T1222.002 - 7
      description: chown - Change file or folder mode ownership only
      type: bool       
    - name: T1222.002 - 8
      description: chown - Change file or folder ownership recursively
      type: bool     
    - name: T1222.002 - 9
      description: chattr - Remove immutable file attribute
      type: bool       
    - name: T1222.002 - 10
      description: chflags - Remove immutable file attribute
      type: bool   
    - name: T1222.002 - 11
      description: Chmod through c script
      type: bool     
    - name: T1222.002 - 12
      description: Chmod through c script (freebsd)
      type: bool    
    - name: T1222.002 - 13
      description: Chown through c script
      type: bool   
    - name: T1222.002 - 14
      description: Chown through c script (freebsd)
      type: bool    
    - name: T1098.004 - 1
      description: Modify SSH Authorized Keys
      type: bool
    - name: T1070.002 - 3
      description: Delete log files using built-in log utility
      type: bool 
    - name: T1070.002 - 4
      description: Truncate system log files via truncate utility
      type: bool 
    - name: T1070.002 - 5
      description: Truncate system log files via truncate utility (freebsd)
      type: bool 
    - name: T1070.002 - 6
      description: Delete log files via cat utility by appending /dev/null or /dev/zero
      type: bool 
    - name: T1070.002 - 7
      description: Delete log files via cat utility by appending /dev/null or /dev/zero (freebsd)
      type: bool 
    - name: T1070.002 - 10
      description: Overwrite FreeBSD system log via echo utility
      type: bool 
    - name: T1070.002 - 13
      description: Delete system log files via unlink utility (freebsd)
      type: bool 
    - name: T1070.002 - 18
      description: Delete system journal logs via rm and journalctl utilities
      type: bool 
    - name: T1070.002 - 19
      description: Overwrite Linux Mail Spool
      type: bool 
    - name: T1070.002 - 20
      description: Overwrite Linux Log
      type: bool 
    - name: T1059.004 - 1
      description: Create and Execute Bash Shell Script
      type: bool 
    - name: T1059.004 - 2
      description: Command-Line Interface
      type: bool 
    - name: T1059.004 - 3
      description: Harvest SUID executable files
      type: bool      
    - name: T1059.004 - 4
      description: LinEnum tool execution
      type: bool      
    - name: T1059.004 - 5
      description: New script file in the tmp directory
      type: bool  
    - name: T1059.004 - 6
      description: What shell is running
      type: bool 
    - name: T1059.004 - 7
      description: What shells are available
      type: bool 
    - name: T1059.004 - 8
      description: Command line scripts
      type: bool 
    - name: T1059.004 - 9
      description: Obfuscated command line scripts
      type: bool 
    - name: T1059.004 - 10
      description: Change login shell
      type: bool 
    - name: T1059.004 - 11
      description: Environment variable scripts
      type: bool
    - name: T1059.004 - 12
      description: Detecting pipe-to-shell
      type: bool
    - name: T1059.004 - 13  
      description: Current kernel information enumeration
      type: bool
    - name: T1059.004 - 14
      description: Shell Creation using awk command
      type: bool
    - name: T1059.004 - 15
      description: Creating shell using cpan command
      type: bool
    - name: T1059.004 - 16
      description: Shell Creation using busybox commands
      type: bool
    - name: T1059.004 - 17
      description: emacs spawning an interactive system shell
      type: bool
    - name: T1053.006 - 1
      description: Create Systemd Service and Timer
      type: bool 
    - name: T1053.006 - 2
      description: Create a user level transient systemd service and timer
      type: bool 
    - name: T1053.006 - 3
      description: Create a system level transient systemd service and timer
      type: bool 
    - name: T1037.004 - 2
      description: rc.common
      type: bool 
    - name: T1037.004 - 3
      description: rc.local
      type: bool 
    - name: T1036.006 - 1
      description: Space After Filename
      type: bool 
    - name: T1003.008 - 1
      description: Access /etc/shadow (Local)
      type: bool    
    - name: T1003.008 - 2
      description: Access /etc/master.passwd (Local)
      type: bool  
    - name: T1003.008 - 3
      description: Access /etc/passwd (Local)
      type: bool 
    - name: T1003.008 - 4
      description: Access /etc/{shadow,passwd,master.passwd} with a standard bin that's not cat
      type: bool 
    - name: T1003.008 - 5
      description: Access /etc/{shadow,passwd,master.passwd} with shell builtins
      type: bool 
    - name: T1003.007 - 1
      description: Dump individual process memory with sh (Local)
      type: bool    
    - name: T1003.007 - 2
      description: Dump individual process memory with sh on FreeBSD (Local)
      type: bool  
    - name: T1003.007 - 3
      description: Dump individual process memory with Python (Local)
      type: bool  
    - name: T1003.007 - 4
      description: Capture Passwords with MimiPenguin
      type: bool  
    - name: T1014 - 1
      description: Loadable Kernel Module based Rootkit
      type: bool
    - name: T1014 - 2
      description: Loadable Kernel Module based Rootkit
      type: bool
    - name: T1014 - 3
      description: dynamic-linker based rootkit (libprocesshider)
      type: bool
    - name: T1014 - 4
      description: Loadable Kernel Module based Rootkit (Diamorphine)
      type: bool
    - name: T1036.005 - 1
      description: Execute a process from a directory masquerading as the current parent directory.
      type: bool
    - name: T1497.001 - 1
      description: Detect Virtualization Environment (Linux)
      type: bool
    - name: T1497.001 - 2
      description: Detect Virtualization Environment (FreeBSD)
      type: bool
    - name: T1070.003 - 1
      description: Clear Bash history (rm)
      type: bool
    - name: T1070.003 - 2
      description: Clear Bash history (echo)
      type: bool
    - name: T1070.003 - 3
      description: Clear Bash history (cat dev/null)
      type: bool
    - name: T1070.003 - 4
      description: Clear Bash history (ln dev/null)
      type: bool
    - name: T1070.003 - 5
      description: Clear Bash history (truncate)
      type: bool
    - name: T1070.003 - 6
      description: Clear history of a bunch of shells
      type: bool
    - name: T1070.003 - 7
      description: Clear and Disable Bash History Logging
      type: bool
    - name: T1070.003 - 8
      description: Use Space Before Command to Avoid Logging to History
      type: bool
    - name: T1070.003 - 9
      description: Disable Bash History Logging with SSH -T
      type: bool
    - name: T1070.003 - 10
      description: Clear Docker Container Logs
      type: bool
    - name: T1140 - 3
      description: Base64 decoding with Python
      type: bool
    - name: T1140 - 4
      description: Base64 decoding with Perl
      type: bool
    - name: T1140 - 5
      description: Base64 decoding with shell utilities
      type: bool
    - name: T1140 - 6
      description: Base64 decoding with shell utilities (freebsd)
      type: bool
    - name: T1140 - 7
      description: FreeBSD b64encode Shebang in CLI
      type: bool
    - name: T1140 - 8
      description: Hex decoding with shell utilities
      type: bool
    - name: T1140 - 9
      description: Linux Base64 Encoded Shebang in CLI
      type: bool
    - name: T1140 - 10
      description: XOR decoding and command execution using Python
      type: bool
    - name: T1562 - 2
      description: Disable journal logging via systemctl utility
      type: bool
    - name: T1562 - 3
      description: Disable journal logging via sed utility
      type: bool
    - name: T1070.008 - 2
      description: Copy and Delete Mailbox Data on Linux
      type: bool
    - name: T1070.008 - 5
      description: Copy and Modify Mailbox Data on Linux
      type: bool
    - name: T1070.006 - 1
      description: Set a file's access timestamp
      type: bool
    - name: T1070.006 - 2
      description: Set a file's modification timestamp
      type: bool
    - name: T1070.006 - 3
      description: Set a file's creation timestamp
      type: bool
    - name: T1070.006 - 4
      description: Modify file timestamps using reference file
      type: bool
    - name: T1497.003 - 1
      description: Delay execution with ping
      type: bool
    - name: T1562.004 - 7
      description: Stop/Start UFW firewall
      type: bool
    - name: T1562.004 - 8
      description: Stop/Start Packet Filter
      type: bool
    - name: T1562.004 - 9
      description: Stop/Start UFW firewall systemctl
      type: bool
    - name: T1562.004 - 10
      description: Turn off UFW logging
      type: bool
    - name: T1562.004 - 11
      description: Add and delete UFW firewall rules
      type: bool
    - name: T1562.004 - 12
      description: Add and delete Packet Filter rules
      type: bool
    - name: T1562.004 - 13
      description: Edit UFW firewall user.rules file
      type: bool
    - name: T1562.004 - 14
      description: Edit UFW firewall ufw.conf file
      type: bool
    - name: T1562.004 - 15
      description: Edit UFW firewall sysctl.conf file
      type: bool
    - name: T1562.004 - 16
      description: Edit UFW firewall main configuration file
      type: bool
    - name: T1562.004 - 17
      description: Tail the UFW firewall log file
      type: bool
    - name: T1562.004 - 18
      description: Disable iptables
      type: bool
    - name: T1562.004 - 19
      description: Modify/delete iptables firewall rules
      type: bool
    - name: T1027.001 - 1
      description: Pad Binary to Change Hash - Linux/macOS dd
      type: bool
    - name: T1027.001 - 2
      description: Pad Binary to Change Hash using truncate command - Linux/macOS
      type: bool
    - name: T1562.006 - 1
      description: Auditing Configuration Changes on Linux Host
      type: bool
    - name: T1562.006 - 2
      description: Auditing Configuration Changes on FreeBSD Host
      type: bool
    - name: T1562.006 - 3
      description: Logging Configuration Changes on Linux Host
      type: bool
    - name: T1562.006 - 4
      description: Logging Configuration Changes on FreeBSD Host
      type: bool
    - name: T1036.004 - 3
      description: linux rename /proc/pid/comm using prctl
      type: bool
    - name: T1036.004 - 4
      description: Hiding a malicious process with bind mounts
      type: bool
    - name: T1562.010 - 1
      description: ESXi - Change VIB acceptance level to CommunitySupported via PowerCLI
      type: bool
    - name: T1562.003 - 1
      description: Disable history collection
      type: bool
    - name: T1562.003 - 2
      description: Disable history collection (freebsd)
      type: bool
    - name: T1562.003 - 3
      description: Mac HISTCONTROL
      type: bool
    - name: T1562.003 - 4
      description: Clear bash history
      type: bool
    - name: T1562.003 - 5
      description: Setting the HISTCONTROL environment variable
      type: bool
    - name: T1562.003 - 6
      description: Setting the HISTFILESIZE environment variable
      type: bool
    - name: T1562.003 - 7
      description: Setting the HISTSIZE environment variable
      type: bool
    - name: T1562.003 - 8
      description: Setting the HISTFILE environment variable
      type: bool
    - name: T1562.003 - 9
      description: Setting the HISTFILE environment variable (freebsd)
      type: bool
    - name: T1562.003 - 10
      description: Setting the HISTIGNORE environment variable
      type: bool
    - name: T1562.001 - 1
      description: Disable syslog
      type: bool
    - name: T1562.001 - 2
      description: Disable syslog (freebsd)
      type: bool
    - name: T1562.001 - 3
      description: Disable Cb Response
      type: bool
    - name: T1562.001 - 4
      description: Disable SELinux
      type: bool
    - name: T1562.001 - 5
      description: Stop Crowdstrike Falcon on Linux
      type: bool
    - name: T1562.001 - 39
      description: Clear History
      type: bool
    - name: T1562.001 - 40
      description: Suspend History
      type: bool
    - name: T1562.001 - 41
      description: Reboot Linux Host via Kernel System Request
      type: bool
    - name: T1562.001 - 42
      description: Clear Pagging Cache
      type: bool
    - name: T1562.001 - 43
      description: Disable Memory Swap
      type: bool
    - name: T1562.001 - 47
      description: Tamper with Defender ATP on Linux/MacOS
      type: bool
    - name: T1562.001 - 50
      description: ESXi - Disable Account Lockout Policy via PowerCLI
      type: bool
    - name: T1027 - 1
      description: Decode base64 Data into Script
      type: bool
    - name: T1036.003 - 2
      description: Masquerading as FreeBSD or Linux crond process.
      type: bool
    - name: T1553.004 - 1
      description: Install root CA on CentOS/RHEL
      type: bool
    - name: T1553.004 - 2
      description: Install root CA on FreeBSD
      type: bool
    - name: T1553.004 - 3
      description: Install root CA on Debian/Ubuntu
      type: bool
    - name: T1027.004 - 3
      description: C compile
      type: bool
    - name: T1027.004 - 4
      description: CC compile
      type: bool
    - name: T1027.004 - 5
      description: Go compile
      type: bool
    - name: T1070.004 - 1
      description: Delete a single file - FreeBSD/Linux/macOS
      type: bool
    - name: T1070.004 - 2
      description: Delete an entire folder - FreeBSD/Linux/macOS
      type: bool
    - name: T1070.004 - 3
      description: Overwrite and delete a file with shred
      type: bool
    - name: T1070.004 - 8
      description: Delete Filesystem - Linux
      type: bool
    - name: T1027.002 - 1
      description: Binary simply packed by UPX (linux)
      type: bool
    - name: T1027.002 - 2
      description: Binary packed by UPX, with modified headers (linux)
      type: bool
    - name: T1564.001 - 1
      description: Create a hidden file in a hidden directory
      type: bool
    - name: T1078.003 - 8
      description: Create local account (Linux)
      type: bool
    - name: T1078.003 - 9
      description: Reactivate a locked/expired account (Linux)
      type: bool
    - name: T1078.003 - 10
      description: Reactivate a locked/expired account (FreeBSD)
      type: bool
    - name: T1078.003 - 11
      description: Login as nobody (Linux)
      type: bool
    - name: T1078.003 - 12
      description: Login as nobody (freebsd)
      type: bool
    - name: T1053.002 - 2
      description: At - Schedule a job
      type: bool
    - name: T1078.003 - 8
      description: Create local account (Linux)
      type: bool
    - name: T1078.003 - 9
      description: Reactivate a locked/expired account (Linux)
      type: bool
    - name: T1078.003 - 10
      description: Reactivate a locked/expired account (FreeBSD)
      type: bool
    - name: T1078.003 - 11
      description: Login as nobody (Linux)
      type: bool
    - name: T1078.003 - 12
      description: Login as nobody (freebsd)
      type: bool
    - name: T1059.006 - 1
      description: Execute shell script via python's command mode arguement
      type: bool
    - name: T1059.006 - 2
      description: Execute Python via scripts
      type: bool
    - name: T1059.006 - 3
      description: Execute Python via Python executables
      type: bool
    - name: T1059.006 - 4
      description: Python pty module and spawn function used to spawn sh or bash
      type: bool
    - name: T1569.002 - 3
      description: psexec.py (Impacket)
      type: bool
    - name: T1053.002 - 2
      description: At - Schedule a job
      type: bool
    - name: T1176 - 1
      description: Chrome/Chromium (Developer Mode)
      type: bool
    - name: T1176 - 2
      description: Chrome/Chromium (Chrome Web Store)
      type: bool
    - name: T1176 - 3
      description: Firefox
      type: bool
    - name: T1136.001 - 1
      description: Create a user account on a Linux system
      type: bool
    - name: T1136.001 - 2
      description: Create a user account on a FreeBSD system
      type: bool
    - name: T1136.001 - 6
      description: Create a new user in Linux with `root` UID and GID.
      type: bool
    - name: T1136.001 - 7
      description: Create a new user in FreeBSD with `root` GID.
      type: bool
    - name: T1136.002 - 4
      description: Active Directory Create Admin Account
      type: bool
    - name: T1136.002 - 5
      description: Active Directory Create User Account (Non-elevated)
      type: bool
    - name: T1053.002 - 2
      description: At - Schedule a job
      type: bool
    - name: T1078.003 - 8
      description: Create local account (Linux)
      type: bool
    - name: T1078.003 - 9
      description: Reactivate a locked/expired account (Linux)
      type: bool
    - name: T1078.003 - 10
      description: Reactivate a locked/expired account (FreeBSD)
      type: bool
    - name: T1078.003 - 11
      description: Login as nobody (Linux)
      type: bool
    - name: T1078.003 - 12
      description: Login as nobody (freebsd)
      type: bool
    - name: T1132.001 - 1
      description: Base64 Encoded data.
      type: bool
    - name: T1132.001 - 2
      description: Base64 Encoded data (freebsd)
      type: bool
    - name: T1572 - 5
      description: Microsoft Dev tunnels (Linux/macOS)
      type: bool
    - name: T1572 - 6
      description: VSCode tunnels (Linux/macOS)
      type: bool
    - name: T1572 - 7
      description: Cloudflare tunnels (Linux/macOS)
      type: bool
    - name: T1090.003 - 3
      description: Tor Proxy Usage - Debian/Ubuntu/FreeBSD
      type: bool
    - name: T1571 - 2
      description: Testing usage of uncommonly used port
      type: bool
    - name: T1095 - 4
      description: Linux ICMP Reverse Shell using icmp-cnc
      type: bool
    - name: T1071.001 - 3
      description: Malicious User Agents - Nix
      type: bool
    - name: T1105 - 1
      description: rsync remote file copy (push)
      type: bool
    - name: T1105 - 2
      description: rsync remote file copy (pull)
      type: bool
    - name: T1105 - 3
      description: scp remote file copy (push)
      type: bool
    - name: T1105 - 4
      description: scp remote file copy (pull)
      type: bool
    - name: T1105 - 5
      description: sftp remote file copy (push)
      type: bool
    - name: T1105 - 6
      description: sftp remote file copy (pull)
      type: bool
    - name: T1105 - 14
      description: whois file download
      type: bool
    - name: T1105 - 27
      description: Linux Download File and Run
      type: bool
    - name: T1001.002 - 3
      description: Execute Embedded Script in Image via Steganography
      type: bool
    - name: T1090.001 - 1
      description: Connection Proxy
      type: bool
    - name: T1560.001 - 5
      description: Data Compressed - nix - zip
      type: bool
    - name: T1560.001 - 6
      description: Data Compressed - nix - gzip Single File
      type: bool
    - name: T1560.001 - 7
      description: Data Compressed - nix - tar Folder or File
      type: bool
    - name: T1560.001 - 8
      description: Data Encrypted with zip and gpg symmetric
      type: bool
    - name: T1560.001 - 9
      description: Encrypts collected data with AES-256 and Base64
      type: bool
    - name: T1113 - 3
      description: X Windows Capture
      type: bool
    - name: T1113 - 4
      description: X Windows Capture (freebsd)
      type: bool
    - name: T1113 - 5
      description: Capture Linux Desktop using Import Tool
      type: bool
    - name: T1113 - 6
      description: Capture Linux Desktop using Import Tool (freebsd)
      type: bool
    - name: T1056.001 - 2
      description: Living off the land Terminal Input Capture on Linux with pam.d
      type: bool
    - name: T1056.001 - 3
      description: Logging bash history to syslog
      type: bool
    - name: T1056.001 - 4
      description: Logging sh history to syslog/messages
      type: bool
    - name: T1056.001 - 5
      description: Bash session based keylogger
      type: bool
    - name: T1056.001 - 6
      description: SSHD PAM keylogger
      type: bool
    - name: T1056.001 - 7
      description: Auditd keylogger
      type: bool
    - name: T1074.001 - 2
      description: Stage data from Discovery.sh
      type: bool
    - name: T1115 - 5
      description: Add or copy content to clipboard with xClip
      type: bool
    - name: T1005 - 2
      description: Find and dump sqlite databases (Linux)
      type: bool
    - name: T1560.002 - 1
      description: Compressing data using GZip in Python (FreeBSD/Linux)
      type: bool
    - name: T1560.002 - 2
      description: Compressing data using bz2 in Python (FreeBSD/Linux)
      type: bool
    - name: T1560.002 - 3
      description: Compressing data using zipfile in Python (FreeBSD/Linux)
      type: bool
    - name: T1560.002 - 4
      description: Compressing data using tarfile in Python (FreeBSD/Linux)
      type: bool
    - name: T1056.001 - 2
      description: Living off the land Terminal Input Capture on Linux with pam.d
      type: bool
    - name: T1056.001 - 3
      description: Logging bash history to syslog
      type: bool
    - name: T1056.001 - 4
      description: Logging sh history to syslog/messages
      type: bool
    - name: T1056.001 - 5
      description: Bash session based keylogger
      type: bool
    - name: T1056.001 - 6
      description: SSHD PAM keylogger
      type: bool
    - name: T1056.001 - 7
      description: Auditd keylogger
      type: bool
    - name: T1110.001 - 5
      description: SUDO Brute Force - Debian
      type: bool
    - name: T1110.001 - 6
      description: SUDO Brute Force - Redhat
      type: bool
    - name: T1110.001 - 7
      description: SUDO Brute Force - FreeBSD
      type: bool
    - name: T1040 - 1
      description: Packet Capture Linux using tshark or tcpdump
      type: bool
    - name: T1040 - 2
      description: Packet Capture FreeBSD using tshark or tcpdump
      type: bool
    - name: T1040 - 10
      description: Packet Capture FreeBSD using /dev/bpfN with sudo
      type: bool
    - name: T1040 - 11
      description: Filtered Packet Capture FreeBSD using /dev/bpfN with sudo
      type: bool
    - name: T1040 - 12
      description: Packet Capture Linux socket AF_PACKET,SOCK_RAW with sudo
      type: bool
    - name: T1040 - 13
      description: Packet Capture Linux socket AF_INET,SOCK_RAW,TCP with sudo
      type: bool
    - name: T1040 - 14
      description: Packet Capture Linux socket AF_INET,SOCK_PACKET,UDP with sudo
      type: bool
    - name: T1040 - 15
      description: Packet Capture Linux socket AF_PACKET,SOCK_RAW with BPF filter for UDP with sudo
      type: bool
    - name: T1552 - 1
      description: AWS - Retrieve EC2 Password Data using stratus
      type: bool
    - name: T1555.003 - 9
      description: LaZagne.py - Dump Credentials from Firefox Browser
      type: bool
    - name: T1552.004 - 2
      description: Discover Private SSH Keys
      type: bool
    - name: T1552.004 - 3
      description: Copy Private SSH Keys with CP
      type: bool
    - name: T1552.004 - 4
      description: Copy Private SSH Keys with CP (freebsd)
      type: bool
    - name: T1552.004 - 5
      description: Copy Private SSH Keys with rsync
      type: bool
    - name: T1552.004 - 6
      description: Copy Private SSH Keys with rsync (freebsd)
      type: bool
    - name: T1552.004 - 7
      description: Copy the users GnuPG directory with rsync
      type: bool
    - name: T1552.004 - 8
      description: Copy the users GnuPG directory with rsync (freebsd)
      type: bool
    - name: T1552.001 - 1
      description: Find AWS credentials
      type: bool
    - name: T1552.001 - 3
      description: Extract passwords with grep
      type: bool
    - name: T1552.001 - 6
      description: Find and Access Github Credentials
      type: bool
    - name: T1552.001 - 15
      description: Find Azure credentials
      type: bool
    - name: T1552.001 - 16
      description: Find GCP credentials
      type: bool
    - name: T1552.001 - 17
      description: Find OCI credentials
      type: bool
    - name: T1110.004 - 1
      description: SSH Credential Stuffing From Linux
      type: bool
    - name: T1110.004 - 3
      description: SSH Credential Stuffing From FreeBSD
      type: bool
    - name: T1033 - 2
      description: System Owner/User Discovery
      type: bool
    - name: T1016.001 - 2
      description: Check internet connection using ping freebsd, linux or macos
      type: bool
    - name: T1087.002 - 23
      description: Active Directory Domain Search
      type: bool
    - name: T1087.002 - 24
      description: Account Enumeration with LDAPDomainDump
      type: bool
    - name: T1087.001 - 1
      description: Enumerate all accounts (Local)
      type: bool
    - name: T1087.001 - 2
      description: View sudoers access
      type: bool
    - name: T1087.001 - 3
      description: View accounts with UID 0
      type: bool
    - name: T1087.001 - 4
      description: List opened files by user
      type: bool
    - name: T1087.001 - 5
      description: Show if a user account has ever logged in remotely
      type: bool
    - name: T1087.001 - 6
      description: Enumerate users and groups
      type: bool
    - name: T1497.001 - 1
      description: Detect Virtualization Environment (Linux)
      type: bool
    - name: T1497.001 - 2
      description: Detect Virtualization Environment (FreeBSD)
      type: bool
    - name: T1069.002 - 15
      description: Active Directory Domain Search Using LDAP - Linux (Ubuntu)/macOS
      type: bool
    - name: T1007 - 3
      description: System Service Discovery - systemctl/service
      type: bool
    - name: T1040 - 1
      description: Packet Capture Linux using tshark or tcpdump
      type: bool
    - name: T1040 - 2
      description: Packet Capture FreeBSD using tshark or tcpdump
      type: bool
    - name: T1040 - 10
      description: Packet Capture FreeBSD using /dev/bpfN with sudo
      type: bool
    - name: T1040 - 11
      description: Filtered Packet Capture FreeBSD using /dev/bpfN with sudo
      type: bool
    - name: T1040 - 12
      description: Packet Capture Linux socket AF_PACKET,SOCK_RAW with sudo
      type: bool
    - name: T1040 - 13
      description: Packet Capture Linux socket AF_INET,SOCK_RAW,TCP with sudo
      type: bool
    - name: T1040 - 14
      description: Packet Capture Linux socket AF_INET,SOCK_PACKET,UDP with sudo
      type: bool
    - name: T1040 - 15
      description: Packet Capture Linux socket AF_PACKET,SOCK_RAW with BPF filter for UDP with sudo
      type: bool
    - name: T1135 - 2
      description: Network Share Discovery - linux
      type: bool
    - name: T1135 - 3
      description: Network Share Discovery - FreeBSD
      type: bool
    - name: T1082 - 3
      description: List OS Information
      type: bool
    - name: T1082 - 4
      description: Linux VM Check via Hardware
      type: bool
    - name: T1082 - 5
      description: Linux VM Check via Kernel Modules
      type: bool
    - name: T1082 - 6
      description: FreeBSD VM Check via Kernel Modules
      type: bool
    - name: T1082 - 8
      description: Hostname Discovery
      type: bool
    - name: T1082 - 12
      description: Environment variables discovery on freebsd, macos and linux
      type: bool
    - name: T1082 - 25
      description: Linux List Kernel Modules
      type: bool
    - name: T1082 - 26
      description: FreeBSD List Kernel Modules
      type: bool
    - name: T1497.003 - 1
      description: Delay execution with ping
      type: bool
    - name: T1217 - 1
      description: List Mozilla Firefox Bookmark Database Files on FreeBSD/Linux
      type: bool
    - name: T1217 - 4
      description: List Google Chromium Bookmark JSON Files on FreeBSD
      type: bool
    - name: T1016 - 3
      description: System Network Configuration Discovery
      type: bool
    - name: T1083 - 3
      description: Nix File and Directory Discovery
      type: bool
    - name: T1083 - 4
      description: Nix File and Directory Discovery 2
      type: bool
    - name: T1049 - 3
      description: System Network Connections Discovery FreeBSD, Linux & MacOS
      type: bool
    - name: T1057 - 1
      description: Process Discovery - ps
      type: bool
    - name: T1069.001 - 1
      description: Permission Groups Discovery (Local)
      type: bool
    - name: T1201 - 1
      description: Examine password complexity policy - Ubuntu
      type: bool
    - name: T1201 - 2
      description: Examine password complexity policy - FreeBSD
      type: bool
    - name: T1201 - 3
      description: Examine password complexity policy - CentOS/RHEL 7.x
      type: bool
    - name: T1201 - 4
      description: Examine password complexity policy - CentOS/RHEL 6.x
      type: bool
    - name: T1201 - 5
      description: Examine password expiration policy - All Linux
      type: bool
    - name: T1614.001 - 3
      description: Discover System Language with locale
      type: bool
    - name: T1614.001 - 4
      description: Discover System Language with localectl
      type: bool
    - name: T1614.001 - 5
      description: Discover System Language by locale file
      type: bool
    - name: T1614.001 - 6
      description: Discover System Language by Environment Variable Query
      type: bool
    - name: T1614 - 2
      description: Get geolocation info through IP-Lookup services using curl freebsd, linux or macos
      type: bool
    - name: T1518.001 - 4
      description: Security Software Discovery - ps (Linux)
      type: bool
    - name: T1518.001 - 5
      description: Security Software Discovery - pgrep (FreeBSD)
      type: bool
    - name: T1018 - 6
      description: Remote System Discovery - arp nix
      type: bool
    - name: T1018 - 7
      description: Remote System Discovery - sweep
      type: bool
    - name: T1018 - 12
      description: Remote System Discovery - ip neighbour
      type: bool
    - name: T1018 - 13
      description: Remote System Discovery - ip route
      type: bool
    - name: T1018 - 14
      description: Remote System Discovery - netstat
      type: bool
    - name: T1018 - 15
      description: Remote System Discovery - ip tcp_metrics
      type: bool
    - name: T1046 - 1
      description: Port Scan
      type: bool
    - name: T1046 - 2
      description: Port Scan Nmap
      type: bool
    - name: T1046 - 12
      description: Port Scan using nmap (Port range)
      type: bool
    - name: T1124 - 3
      description: System Time Discovery in FreeBSD/macOS
      type: bool
    - name: T1489 - 4
      description: Linux - Stop service using systemctl
      type: bool
    - name: T1489 - 5
      description: Linux - Stop service by killing process using killall
      type: bool
    - name: T1489 - 6
      description: Linux - Stop service by killing process using kill
      type: bool
    - name: T1489 - 7
      description: Linux - Stop service by killing process using pkill
      type: bool
    - name: T1531 - 4
      description: Change User Password via passwd
      type: bool
    - name: T1486 - 1
      description: Encrypt files using gpg (FreeBSD/Linux)
      type: bool
    - name: T1486 - 2
      description: Encrypt files using 7z (FreeBSD/Linux)
      type: bool
    - name: T1486 - 3
      description: Encrypt files using ccrypt (FreeBSD/Linux)
      type: bool
    - name: T1486 - 4
      description: Encrypt files using openssl (FreeBSD/Linux)
      type: bool
    - name: T1496 - 1
      description: FreeBSD/macOS/Linux - Simulate CPU Load with Yes
      type: bool
    - name: T1485 - 2
      description: FreeBSD/macOS/Linux - Overwrite file with DD
      type: bool
    - name: T1529 - 3
      description: Restart System via `shutdown` - FreeBSD/macOS/Linux
      type: bool
    - name: T1529 - 4
      description: Shutdown System via `shutdown` - FreeBSD/macOS/Linux
      type: bool
    - name: T1529 - 5
      description: Restart System via `reboot` - FreeBSD/macOS/Linux
      type: bool
    - name: T1529 - 6
      description: Shutdown System via `halt` - FreeBSD/Linux
      type: bool
    - name: T1529 - 7
      description: Reboot System via `halt` - FreeBSD
      type: bool
    - name: T1529 - 8
      description: Reboot System via `halt` - Linux
      type: bool
    - name: T1529 - 9
      description: Shutdown System via `poweroff` - FreeBSD/Linux
      type: bool
    - name: T1529 - 10
      description: Reboot System via `poweroff` - FreeBSD
      type: bool
    - name: T1529 - 11
      description: Reboot System via `poweroff` - Linux
      type: bool
    - name: T1078.003 - 8
      description: Create local account (Linux)
      type: bool
    - name: T1078.003 - 9
      description: Reactivate a locked/expired account (Linux)
      type: bool
    - name: T1078.003 - 10
      description: Reactivate a locked/expired account (FreeBSD)
      type: bool
    - name: T1078.003 - 11
      description: Login as nobody (Linux)
      type: bool
    - name: T1078.003 - 12
      description: Login as nobody (freebsd)
      type: bool
    - name: T1048.002 - 2
      description: Exfiltrate data HTTPS using curl freebsd,linux or macos
      type: bool
    - name: T1048.002 - 3
      description: Exfiltrate data in a file over HTTPS using wget
      type: bool
    - name: T1048.002 - 4
      description: Exfiltrate data as text over HTTPS using wget
      type: bool
    - name: T1048 - 1
      description: Exfiltration Over Alternative Protocol - SSH
      type: bool
    - name: T1048 - 2
      description: Exfiltration Over Alternative Protocol - SSH
      type: bool
    - name: T1048 - 4
      description: Exfiltrate Data using DNS Queries via dig
      type: bool
    - name: T1567.002 - 2
      description: Exfiltrate data with rclone to cloud Storage - AWS S3
      type: bool
    - name: T1030 - 1
      description: Data Transfer Size Limits
      type: bool
    - name: T1048.003 - 1
      description: Exfiltration Over Alternative Protocol - HTTP
      type: bool
    - name: T1048.003 - 3
      description: Exfiltration Over Alternative Protocol - DNS
      type: bool
    - name: T1048.003 - 8
      description: Python3 http.server
      type: bool

    - name: T1053.003 - 1
      description: Cron - Replace crontab with referenced file
      type: bool
      
    - name: T1053.003 - 2
      description: Cron - Add script to all cron subfolders
      type: bool
      
    - name: T1053.003 - 3
      description: Cron - Add script to /etc/cron.d folder 
      type: bool
      
    - name: T1053.003 - 4
      description: Cron - Add script to /var/spool/cron/crontabs/ folder
      type: bool
    


precondition: SELECT OS From info() where OS = 'linux'
sources:
  - query: |
     LET CommandTable = SELECT * FROM parse_csv(accessor="data", filename='''
     Flag,Command
     T1580 - 1,"Invoke-AtomicTest T1580 -TestNumbers 1"
     T1580 - 2,"Invoke-AtomicTest T1580 -TestNumbers 2"
     T1574.006 - 1,"Invoke-AtomicTest T1574.006 -TestNumbers 1"
     T1574.006 - 2,"Invoke-AtomicTest T1574.006 -TestNumbers 2"
     T1562.012 - 1,"Invoke-AtomicTest T1562.012 -TestNumbers 1"
     T1562.012 - 2,"Invoke-AtomicTest T1562.012 -TestNumbers 2"
     T1562.008 - 4,"Invoke-AtomicTest T1562.008 -TestNumbers 4"
     T1562.008 - 5,"Invoke-AtomicTest T1562.008 -TestNumbers 5"
     T1562.008 - 6,"Invoke-AtomicTest T1562.008 -TestNumbers 6"
     T1556.003 - 1,"Invoke-AtomicTest T1556.003 -TestNumbers 1"
     T1556.003 - 2,"Invoke-AtomicTest T1556.003 -TestNumbers 2"
     T1556.003 - 3,"Invoke-AtomicTest T1556.003 -TestNumbers 3"
     T1552.007 - 3,"Invoke-AtomicTest T1552.007 -TestNumbers 3"
     T1552.003 - 1,"Invoke-AtomicTest T1552.003 -TestNumbers 1"
     T1552.003 - 2,"Invoke-AtomicTest T1552.003 -TestNumbers 2"
     T1548.003 - 1,"Invoke-AtomicTest T1548.003 -TestNumbers 1"
     T1548.003 - 2,"Invoke-AtomicTest T1548.003 -TestNumbers 2"
     T1548.003 - 3,"Invoke-AtomicTest T1548.003 -TestNumbers 3"
     T1548.003 - 4,"Invoke-AtomicTest T1548.003 -TestNumbers 4"
     T1548.003- 5,"Invoke-AtomicTest T1548.003 -TestNumbers 5"
     T1548.003 - 6,"Invoke-AtomicTest T1548.003 -TestNumbers 6"
     T1548.001 - 1,"Invoke-AtomicTest T1548.001 -TestNumbers 1"
     T1548.001 - 2,"Invoke-AtomicTest T1548.001 -TestNumbers 2"
     T1548.001 - 3,"Invoke-AtomicTest T1548.001 -TestNumbers 3"
     T1548.001 - 4,"Invoke-AtomicTest T1548.001 -TestNumbers 4"
     T1548.001- 5,"Invoke-AtomicTest T1548.001 -TestNumbers 5"
     T1548.001 - 6,"Invoke-AtomicTest T1548.001 -TestNumbers 6"
     T1548.001 - 7,"Invoke-AtomicTest T1548.001 -TestNumbers 7"
     T1548.001 - 8,"Invoke-AtomicTest T1548.001 -TestNumbers 8"
     T1548.001 - 9,"Invoke-AtomicTest T1548.001 -TestNumbers 9"
     T1548.001- 10,"Invoke-AtomicTest T1548.001 -TestNumbers 10"
     T1547.006 - 1,"Invoke-AtomicTest T1547.006 -TestNumbers 1"
     T1546.005 - 1,"Invoke-AtomicTest T1546.005 -TestNumbers 1"
     T1546.005 - 2,"Invoke-AtomicTest T1546.005 -TestNumbers 2"
     T1546.005 - 3,"Invoke-AtomicTest T1546.005 -TestNumbers 3"
     T1546.005 - 4,"Invoke-AtomicTest T1546.004 -TestNumbers 4"
     T1546.005 - 5,"Invoke-AtomicTest T1546.005 -TestNumbers 5"
     T1546.004 - 1,"Invoke-AtomicTest T1546.004 -TestNumbers 1"
     T1546.004 - 2,"Invoke-AtomicTest T1546.004 -TestNumbers 2"
     T1546.004 - 3,"Invoke-AtomicTest T1546.004 -TestNumbers 3"
     T1546.004 - 4,"Invoke-AtomicTest T1546.004 -TestNumbers 4"
     T1546.004 - 5,"Invoke-AtomicTest T1546.004 -TestNumbers 5"
     T1546.004 - 6,"Invoke-AtomicTest T1546.004 -TestNumbers 6"
     T1546.004 - 7,"Invoke-AtomicTest T1546.004 -TestNumbers 7"
     T1543.002 - 1,"Invoke-AtomicTest T1543.002 -TestNumbers 1"
     T1543.002 - 2,"Invoke-AtomicTest T1543.002 -TestNumbers 2"
     T1543.002 - 3,"Invoke-AtomicTest T1543.002 -TestNumbers 3"
     T1222.002 - 1,"Invoke-AtomicTest T1222.002 -TestNumbers 1"
     T1222.002 - 2,"Invoke-AtomicTest T1222.002 -TestNumbers 2"
     T1222.002 - 3,"Invoke-AtomicTest T1222.002 -TestNumbers 3"
     T1222.002 - 4,"Invoke-AtomicTest T1222.002 -TestNumbers 4"
     T1222.002 - 5,"Invoke-AtomicTest T1222.002 -TestNumbers 5"
     T1222.002 - 6,"Invoke-AtomicTest T1222.002 -TestNumbers 6"
     T1222.002 - 7,"Invoke-AtomicTest T1222.002 -TestNumbers 7"
     T1222.002 - 8,"Invoke-AtomicTest T1222.002 -TestNumbers 8"
     T1222.002 - 9,"Invoke-AtomicTest T1222.002 -TestNumbers 9"
     T1222.002 - 10,"Invoke-AtomicTest T1222.002 -TestNumbers 10"
     T1222.002 - 11,"Invoke-AtomicTest T1222.002 -TestNumbers 11"
     T1222.002 - 12,"Invoke-AtomicTest T1222.002 -TestNumbers 12"
     T1222.002 - 13,"Invoke-AtomicTest T1222.002 -TestNumbers 13"
     T1222.002 - 14,"Invoke-AtomicTest T1222.002 -TestNumbers 14"
     T1098.004 - 3,"Invoke-AtomicTest T1098.004 -TestNumbers 1"
     T1070.002 - 3,"Invoke-AtomicTest T1070.002 -TestNumbers 3"
     T1070.002 - 4,"Invoke-AtomicTest T1070.002 -TestNumbers 4"
     T1070.002 - 5,"Invoke-AtomicTest T1070.002 -TestNumbers 5"
     T1070.002 - 6,"Invoke-AtomicTest T1070.002 -TestNumbers 6"
     T1070.002 - 7,"Invoke-AtomicTest T1070.002 -TestNumbers 7"
     T1070.002 - 10,"Invoke-AtomicTest T1070.002 -TestNumbers 10"
     T1070.002 - 13,"Invoke-AtomicTest T1070.002 -TestNumbers 13"
     T1070.002 - 18,"Invoke-AtomicTest T1070.002 -TestNumbers 18"
     T1070.002 - 19,"Invoke-AtomicTest T1070.002 -TestNumbers 19"
     T1070.002 - 20,"Invoke-AtomicTest T1070.002 -TestNumbers 20"
     T1053.006 - 1,"Invoke-AtomicTest T1053.006 -TestNumbers 1"
     T1053.006 - 2,"Invoke-AtomicTest T1053.006 -TestNumbers 2"
     T1053.006 - 3,"Invoke-AtomicTest T1053.006 -TestNumbers 3"
     T1037.004 - 2,"Invoke-AtomicTest T1037.004 -TestNumbers 2"
     T1037.004 - 3,"Invoke-AtomicTest T1037.004 -TestNumbers 3"
     T1036.006 - 2,"Invoke-AtomicTest T1036.006 -TestNumbers 2"
     T1003.008 - 1,"Invoke-AtomicTest T1003.008 -TestNumbers 1"
     T1003.008 - 2,"Invoke-AtomicTest T1003.008 -TestNumbers 2"
     T1003.008 - 3,"Invoke-AtomicTest T1003.008 -TestNumbers 3"
     T1003.008 - 4,"Invoke-AtomicTest T1003.008 -TestNumbers 4"
     T1003.008 - 5,"Invoke-AtomicTest T1003.008 -TestNumbers 5"
     T1003.007 - 1,"Invoke-AtomicTest T1003.007 -TestNumbers 1"
     T1003.007 - 2,"Invoke-AtomicTest T1003.007 -TestNumbers 2"
     T1003.007 - 3,"Invoke-AtomicTest T1003.007 -TestNumbers 3"
     T1003.007 - 4,"Invoke-AtomicTest T1003.007 -TestNumbers 4"
     T1014 - 1,"Invoke-AtomicTest T1014 -TestNumbers 1"
     T1014 - 2,"Invoke-AtomicTest T1014 -TestNumbers 2"
     T1014 - 3,"Invoke-AtomicTest T1014 -TestNumbers 3"
     T1014 - 4,"Invoke-AtomicTest T1014 -TestNumbers 4"
     T1036.005 - 1,"Invoke-AtomicTest T1036.005 -TestNumbers 1"
     T1497.001 - 1,"Invoke-AtomicTest T1497.001 -TestNumbers 1"
     T1497.001 - 2,"Invoke-AtomicTest T1497.001 -TestNumbers 2"
     T1070.003 - 1,"Invoke-AtomicTest T1070.003 -TestNumbers 1"
     T1070.003 - 2,"Invoke-AtomicTest T1070.003 -TestNumbers 2"
     T1070.003 - 3,"Invoke-AtomicTest T1070.003 -TestNumbers 3"
     T1070.003 - 4,"Invoke-AtomicTest T1070.003 -TestNumbers 4"
     T1070.003 - 5,"Invoke-AtomicTest T1070.003 -TestNumbers 5"
     T1070.003 - 6,"Invoke-AtomicTest T1070.003 -TestNumbers 6"
     T1070.003 - 7,"Invoke-AtomicTest T1070.003 -TestNumbers 7"
     T1070.003 - 8,"Invoke-AtomicTest T1070.003 -TestNumbers 8"
     T1070.003 - 9,"Invoke-AtomicTest T1070.003 -TestNumbers 9"
     T1070.003 - 10,"Invoke-AtomicTest T1070.003 -TestNumbers 10"
     T1140 - 3,"Invoke-AtomicTest T1140 -TestNumbers 3"
     T1140 - 4,"Invoke-AtomicTest T1140 -TestNumbers 4"
     T1140 - 5,"Invoke-AtomicTest T1140 -TestNumbers 5"
     T1140 - 6,"Invoke-AtomicTest T1140 -TestNumbers 6"
     T1140 - 7,"Invoke-AtomicTest T1140 -TestNumbers 7"
     T1140 - 8,"Invoke-AtomicTest T1140 -TestNumbers 8"
     T1140 - 9,"Invoke-AtomicTest T1140 -TestNumbers 9"
     T1140 - 10,"Invoke-AtomicTest T1140 -TestNumbers 10"
     T1562 - 2,"Invoke-AtomicTest T1562 -TestNumbers 2"
     T1562 - 3,"Invoke-AtomicTest T1562 -TestNumbers 3"
     T1070.008 - 2,"Invoke-AtomicTest T1070.008 -TestNumbers 2"
     T1070.008 - 5,"Invoke-AtomicTest T1070.008 -TestNumbers 5"
     T1070.006 - 1,"Invoke-AtomicTest T1070.006 -TestNumbers 1"
     T1070.006 - 2,"Invoke-AtomicTest T1070.006 -TestNumbers 2"
     T1070.006 - 3,"Invoke-AtomicTest T1070.006 -TestNumbers 3"
     T1070.006 - 4,"Invoke-AtomicTest T1070.006 -TestNumbers 4"
     T1497.003 - 1,"Invoke-AtomicTest T1497.003 -TestNumbers 1"
     T1562.004 - 7,"Invoke-AtomicTest T1562.004 -TestNumbers 7"
     T1562.004 - 8,"Invoke-AtomicTest T1562.004 -TestNumbers 8"
     T1562.004 - 9,"Invoke-AtomicTest T1562.004 -TestNumbers 9"
     T1562.004 - 10,"Invoke-AtomicTest T1562.004 -TestNumbers 10"
     T1562.004 - 11,"Invoke-AtomicTest T1562.004 -TestNumbers 11"
     T1562.004 - 12,"Invoke-AtomicTest T1562.004 -TestNumbers 12"
     T1562.004 - 13,"Invoke-AtomicTest T1562.004 -TestNumbers 13"
     T1562.004 - 14,"Invoke-AtomicTest T1562.004 -TestNumbers 14"
     T1562.004 - 15,"Invoke-AtomicTest T1562.004 -TestNumbers 15"
     T1562.004 - 16,"Invoke-AtomicTest T1562.004 -TestNumbers 16"
     T1562.004 - 17,"Invoke-AtomicTest T1562.004 -TestNumbers 17"
     T1562.004 - 18,"Invoke-AtomicTest T1562.004 -TestNumbers 18"
     T1562.004 - 19,"Invoke-AtomicTest T1562.004 -TestNumbers 19"
     T1027.001 - 1,"Invoke-AtomicTest T1027.001 -TestNumbers 1"
     T1027.001 - 2,"Invoke-AtomicTest T1027.001 -TestNumbers 2"
     T1562.006 - 1,"Invoke-AtomicTest T1562.006 -TestNumbers 1"
     T1562.006 - 2,"Invoke-AtomicTest T1562.006 -TestNumbers 2"
     T1562.006 - 3,"Invoke-AtomicTest T1562.006 -TestNumbers 3"
     T1562.006 - 4,"Invoke-AtomicTest T1562.006 -TestNumbers 4"
     T1036.004 - 3,"Invoke-AtomicTest T1036.004 -TestNumbers 3"
     T1036.004 - 4,"Invoke-AtomicTest T1036.004 -TestNumbers 4"
     T1562.010 - 1,"Invoke-AtomicTest T1562.010 -TestNumbers 1"
     T1562.003 - 1,"Invoke-AtomicTest T1562.003 -TestNumbers 1"
     T1562.003 - 2,"Invoke-AtomicTest T1562.003 -TestNumbers 2"
     T1562.003 - 3,"Invoke-AtomicTest T1562.003 -TestNumbers 3"
     T1562.003 - 4,"Invoke-AtomicTest T1562.003 -TestNumbers 4"
     T1562.003 - 5,"Invoke-AtomicTest T1562.003 -TestNumbers 5"
     T1562.003 - 6,"Invoke-AtomicTest T1562.003 -TestNumbers 6"
     T1562.003 - 7,"Invoke-AtomicTest T1562.003 -TestNumbers 7"
     T1562.003 - 8,"Invoke-AtomicTest T1562.003 -TestNumbers 8"
     T1562.003 - 9,"Invoke-AtomicTest T1562.003 -TestNumbers 9"
     T1562.003 - 10,"Invoke-AtomicTest T1562.003 -TestNumbers 10"
     T1562.001 - 1,"Invoke-AtomicTest T1562.001 -TestNumbers 1"
     T1562.001 - 2,"Invoke-AtomicTest T1562.001 -TestNumbers 2"
     T1562.001 - 3,"Invoke-AtomicTest T1562.001 -TestNumbers 3"
     T1562.001 - 4,"Invoke-AtomicTest T1562.001 -TestNumbers 4"
     T1562.001 - 5,"Invoke-AtomicTest T1562.001 -TestNumbers 5"
     T1562.001 - 39,"Invoke-AtomicTest T1562.001 -TestNumbers 39"
     T1562.001 - 40,"Invoke-AtomicTest T1562.001 -TestNumbers 40"
     T1562.001 - 41,"Invoke-AtomicTest T1562.001 -TestNumbers 41"
     T1562.001 - 42,"Invoke-AtomicTest T1562.001 -TestNumbers 42"
     T1562.001 - 43,"Invoke-AtomicTest T1562.001 -TestNumbers 43"
     T1562.001 - 47,"Invoke-AtomicTest T1562.001 -TestNumbers 47"
     T1562.001 - 50,"Invoke-AtomicTest T1562.001 -TestNumbers 50"
     T1027 - 1,"Invoke-AtomicTest T1027 -TestNumbers 1"
     T1036.003 - 2,"Invoke-AtomicTest T1036.003 -TestNumbers 2"
     T1553.004 - 1,"Invoke-AtomicTest T1553.004 -TestNumbers 1"
     T1553.004 - 2,"Invoke-AtomicTest T1553.004 -TestNumbers 2"
     T1553.004 - 3,"Invoke-AtomicTest T1553.004 -TestNumbers 3"
     T1027.004 - 3,"Invoke-AtomicTest T1027.004 -TestNumbers 3"
     T1027.004 - 4,"Invoke-AtomicTest T1027.004 -TestNumbers 4"
     T1027.004 - 5,"Invoke-AtomicTest T1027.004 -TestNumbers 5"
     T1070.004 - 1,"Invoke-AtomicTest T1070.004 -TestNumbers 1"
     T1070.004 - 2,"Invoke-AtomicTest T1070.004 -TestNumbers 2"
     T1070.004 - 3,"Invoke-AtomicTest T1070.004 -TestNumbers 3"
     T1070.004 - 8,"Invoke-AtomicTest T1070.004 -TestNumbers 8"
     T1027.002 - 1,"Invoke-AtomicTest T1027.002 -TestNumbers 1"
     T1027.002 - 2,"Invoke-AtomicTest T1027.002 -TestNumbers 2"
     T1564.001 - 1,"Invoke-AtomicTest T1564.001 -TestNumbers 1"
     T1078.003 - 8,"Invoke-AtomicTest T1078.003 -TestNumbers 8"
     T1078.003 - 9,"Invoke-AtomicTest T1078.003 -TestNumbers 9"
     T1078.003 - 10,"Invoke-AtomicTest T1078.003 -TestNumbers 10"
     T1078.003 - 11,"Invoke-AtomicTest T1078.003 -TestNumbers 11"
     T1078.003 - 12,"Invoke-AtomicTest T1078.003 -TestNumbers 12"
     T1053.002 - 2,"Invoke-AtomicTest T1053.002 -TestNumbers 2"
     T1078.003 - 8,"Invoke-AtomicTest T1078.003 -TestNumbers 8"
     T1078.003 - 9,"Invoke-AtomicTest T1078.003 -TestNumbers 9"
     T1078.003 - 10,"Invoke-AtomicTest T1078.003 -TestNumbers 10"
     T1078.003 - 11,"Invoke-AtomicTest T1078.003 -TestNumbers 11"
     T1078.003 - 12,"Invoke-AtomicTest T1078.003 -TestNumbers 12"
     T1059.006 - 1,"Invoke-AtomicTest T1059.006 -TestNumbers 1"
     T1059.006 - 2,"Invoke-AtomicTest T1059.006 -TestNumbers 2"
     T1059.006 - 3,"Invoke-AtomicTest T1059.006 -TestNumbers 3"
     T1059.006 - 4,"Invoke-AtomicTest T1059.006 -TestNumbers 4"
     T1569.002 - 3,"Invoke-AtomicTest T1569.002 -TestNumbers 3"
     T1053.002 - 2,"Invoke-AtomicTest T1053.002 -TestNumbers 2"
     T1176 - 1,"Invoke-AtomicTest T1176 -TestNumbers 1"
     T1176 - 2,"Invoke-AtomicTest T1176 -TestNumbers 2"
     T1176 - 3,"Invoke-AtomicTest T1176 -TestNumbers 3"
     T1136.001 - 1,"Invoke-AtomicTest T1136.001 -TestNumbers 1"
     T1136.001 - 2,"Invoke-AtomicTest T1136.001 -TestNumbers 2"
     T1136.001 - 6,"Invoke-AtomicTest T1136.001 -TestNumbers 6"
     T1136.001 - 7,"Invoke-AtomicTest T1136.001 -TestNumbers 7"
     T1136.002 - 4,"Invoke-AtomicTest T1136.002 -TestNumbers 4"
     T1136.002 - 5,"Invoke-AtomicTest T1136.002 -TestNumbers 5"
     T1053.002 - 2,"Invoke-AtomicTest T1053.002 -TestNumbers 2"
     T1078.003 - 8,"Invoke-AtomicTest T1078.003 -TestNumbers 8"
     T1078.003 - 9,"Invoke-AtomicTest T1078.003 -TestNumbers 9"
     T1078.003 - 10,"Invoke-AtomicTest T1078.003 -TestNumbers 10"
     T1078.003 - 11,"Invoke-AtomicTest T1078.003 -TestNumbers 11"
     T1078.003 - 12,"Invoke-AtomicTest T1078.003 -TestNumbers 12"
     T1132.001 - 1,"Invoke-AtomicTest T1132.001 -TestNumbers 1"
     T1132.001 - 2,"Invoke-AtomicTest T1132.001 -TestNumbers 2"
     T1572 - 5,"Invoke-AtomicTest T1572 -TestNumbers 5"
     T1572 - 6,"Invoke-AtomicTest T1572 -TestNumbers 6"
     T1572 - 7,"Invoke-AtomicTest T1572 -TestNumbers 7"
     T1090.003 - 3,"Invoke-AtomicTest T1090.003 -TestNumbers 3"
     T1571 - 2,"Invoke-AtomicTest T1571 -TestNumbers 2"
     T1095 - 4,"Invoke-AtomicTest T1095 -TestNumbers 4"
     T1071.001 - 3,"Invoke-AtomicTest T1071.001 -TestNumbers 3"
     T1105 - 1,"Invoke-AtomicTest T1105 -TestNumbers 1"
     T1105 - 2,"Invoke-AtomicTest T1105 -TestNumbers 2"
     T1105 - 3,"Invoke-AtomicTest T1105 -TestNumbers 3"
     T1105 - 4,"Invoke-AtomicTest T1105 -TestNumbers 4"
     T1105 - 5,"Invoke-AtomicTest T1105 -TestNumbers 5"
     T1105 - 6,"Invoke-AtomicTest T1105 -TestNumbers 6"
     T1105 - 14,"Invoke-AtomicTest T1105 -TestNumbers 14"
     T1105 - 27,"Invoke-AtomicTest T1105 -TestNumbers 27"
     T1001.002 - 3,"Invoke-AtomicTest T1001.002 -TestNumbers 3"
     T1090.001 - 1,"Invoke-AtomicTest T1090.001 -TestNumbers 1"
     T1560.001 - 5,"Invoke-AtomicTest T1560.001 -TestNumbers 5"
     T1560.001 - 6,"Invoke-AtomicTest T1560.001 -TestNumbers 6"
     T1560.001 - 7,"Invoke-AtomicTest T1560.001 -TestNumbers 7"
     T1560.001 - 8,"Invoke-AtomicTest T1560.001 -TestNumbers 8"
     T1560.001 - 9,"Invoke-AtomicTest T1560.001 -TestNumbers 9"
     T1113 - 3,"Invoke-AtomicTest T1113 -TestNumbers 3"
     T1113 - 4,"Invoke-AtomicTest T1113 -TestNumbers 4"
     T1113 - 5,"Invoke-AtomicTest T1113 -TestNumbers 5"
     T1113 - 6,"Invoke-AtomicTest T1113 -TestNumbers 6"
     T1056.001 - 2,"Invoke-AtomicTest T1056.001 -TestNumbers 2"
     T1056.001 - 3,"Invoke-AtomicTest T1056.001 -TestNumbers 3"
     T1056.001 - 4,"Invoke-AtomicTest T1056.001 -TestNumbers 4"
     T1056.001 - 5,"Invoke-AtomicTest T1056.001 -TestNumbers 5"
     T1056.001 - 6,"Invoke-AtomicTest T1056.001 -TestNumbers 6"
     T1056.001 - 7,"Invoke-AtomicTest T1056.001 -TestNumbers 7"
     T1074.001 - 2,"Invoke-AtomicTest T1074.001 -TestNumbers 2"
     T1115 - 5,"Invoke-AtomicTest T1115 -TestNumbers 5"
     T1005 - 2,"Invoke-AtomicTest T1005 -TestNumbers 2"
     T1560.002 - 1,"Invoke-AtomicTest T1560.002 -TestNumbers 1"
     T1560.002 - 2,"Invoke-AtomicTest T1560.002 -TestNumbers 2"
     T1560.002 - 3,"Invoke-AtomicTest T1560.002 -TestNumbers 3"
     T1560.002 - 4,"Invoke-AtomicTest T1560.002 -TestNumbers 4"
     T1056.001 - 2,"Invoke-AtomicTest T1056.001 -TestNumbers 2"
     T1056.001 - 3,"Invoke-AtomicTest T1056.001 -TestNumbers 3"
     T1056.001 - 4,"Invoke-AtomicTest T1056.001 -TestNumbers 4"
     T1056.001 - 5,"Invoke-AtomicTest T1056.001 -TestNumbers 5"
     T1056.001 - 6,"Invoke-AtomicTest T1056.001 -TestNumbers 6"
     T1056.001 - 7,"Invoke-AtomicTest T1056.001 -TestNumbers 7"
     T1110.001 - 5,"Invoke-AtomicTest T1110.001 -TestNumbers 5"
     T1110.001 - 6,"Invoke-AtomicTest T1110.001 -TestNumbers 6"
     T1110.001 - 7,"Invoke-AtomicTest T1110.001 -TestNumbers 7"
     T1040 - 1,"Invoke-AtomicTest T1040 -TestNumbers 1"
     T1040 - 2,"Invoke-AtomicTest T1040 -TestNumbers 2"
     T1040 - 10,"Invoke-AtomicTest T1040 -TestNumbers 10"
     T1040 - 11,"Invoke-AtomicTest T1040 -TestNumbers 11"
     T1040 - 12,"Invoke-AtomicTest T1040 -TestNumbers 12"
     T1040 - 13,"Invoke-AtomicTest T1040 -TestNumbers 13"
     T1040 - 14,"Invoke-AtomicTest T1040 -TestNumbers 14"
     T1040 - 15,"Invoke-AtomicTest T1040 -TestNumbers 15"
     T1552 - 1,"Invoke-AtomicTest T1552 -TestNumbers 1"
     T1555.003 - 9,"Invoke-AtomicTest T1555.003 -TestNumbers 9"
     T1552.004 - 2,"Invoke-AtomicTest T1552.004 -TestNumbers 2"
     T1552.004 - 3,"Invoke-AtomicTest T1552.004 -TestNumbers 3"
     T1552.004 - 4,"Invoke-AtomicTest T1552.004 -TestNumbers 4"
     T1552.004 - 5,"Invoke-AtomicTest T1552.004 -TestNumbers 5"
     T1552.004 - 6,"Invoke-AtomicTest T1552.004 -TestNumbers 6"
     T1552.004 - 7,"Invoke-AtomicTest T1552.004 -TestNumbers 7"
     T1552.004 - 8,"Invoke-AtomicTest T1552.004 -TestNumbers 8"
     T1552.001 - 1,"Invoke-AtomicTest T1552.001 -TestNumbers 1"
     T1552.001 - 3,"Invoke-AtomicTest T1552.001 -TestNumbers 3"
     T1552.001 - 6,"Invoke-AtomicTest T1552.001 -TestNumbers 6"
     T1552.001 - 15,"Invoke-AtomicTest T1552.001 -TestNumbers 15"
     T1552.001 - 16,"Invoke-AtomicTest T1552.001 -TestNumbers 16"
     T1552.001 - 17,"Invoke-AtomicTest T1552.001 -TestNumbers 17"
     T1110.004 - 1,"Invoke-AtomicTest T1110.004 -TestNumbers 1"
     T1110.004 - 3,"Invoke-AtomicTest T1110.004 -TestNumbers 3"
     T1033 - 2,"Invoke-AtomicTest T1033 -TestNumbers 2"
     T1016.001 - 2,"Invoke-AtomicTest T1016.001 -TestNumbers 2"
     T1087.002 - 23,"Invoke-AtomicTest T1087.002 -TestNumbers 23"
     T1087.002 - 24,"Invoke-AtomicTest T1087.002 -TestNumbers 24"
     T1087.001 - 1,"Invoke-AtomicTest T1087.001 -TestNumbers 1"
     T1087.001 - 2,"Invoke-AtomicTest T1087.001 -TestNumbers 2"
     T1087.001 - 3,"Invoke-AtomicTest T1087.001 -TestNumbers 3"
     T1087.001 - 4,"Invoke-AtomicTest T1087.001 -TestNumbers 4"
     T1087.001 - 5,"Invoke-AtomicTest T1087.001 -TestNumbers 5"
     T1087.001 - 6,"Invoke-AtomicTest T1087.001 -TestNumbers 6"
     T1497.001 - 1,"Invoke-AtomicTest T1497.001 -TestNumbers 1"
     T1497.001 - 2,"Invoke-AtomicTest T1497.001 -TestNumbers 2"
     T1069.002 - 15,"Invoke-AtomicTest T1069.002 -TestNumbers 15"
     T1007 - 3,"Invoke-AtomicTest T1007 -TestNumbers 3"
     T1040 - 1,"Invoke-AtomicTest T1040 -TestNumbers 1"
     T1040 - 2,"Invoke-AtomicTest T1040 -TestNumbers 2"
     T1040 - 10,"Invoke-AtomicTest T1040 -TestNumbers 10"
     T1040 - 11,"Invoke-AtomicTest T1040 -TestNumbers 11"
     T1040 - 12,"Invoke-AtomicTest T1040 -TestNumbers 12"
     T1040 - 13,"Invoke-AtomicTest T1040 -TestNumbers 13"
     T1040 - 14,"Invoke-AtomicTest T1040 -TestNumbers 14"
     T1040 - 15,"Invoke-AtomicTest T1040 -TestNumbers 15"
     T1135 - 2,"Invoke-AtomicTest T1135 -TestNumbers 2"
     T1135 - 3,"Invoke-AtomicTest T1135 -TestNumbers 3"
     T1082 - 3,"Invoke-AtomicTest T1082 -TestNumbers 3"
     T1082 - 4,"Invoke-AtomicTest T1082 -TestNumbers 4"
     T1082 - 5,"Invoke-AtomicTest T1082 -TestNumbers 5"
     T1082 - 6,"Invoke-AtomicTest T1082 -TestNumbers 6"
     T1082 - 8,"Invoke-AtomicTest T1082 -TestNumbers 8"
     T1082 - 12,"Invoke-AtomicTest T1082 -TestNumbers 12"
     T1082 - 25,"Invoke-AtomicTest T1082 -TestNumbers 25"
     T1082 - 26,"Invoke-AtomicTest T1082 -TestNumbers 26"
     T1497.003 - 1,"Invoke-AtomicTest T1497.003 -TestNumbers 1"
     T1217 - 1,"Invoke-AtomicTest T1217 -TestNumbers 1"
     T1217 - 4,"Invoke-AtomicTest T1217 -TestNumbers 4"
     T1016 - 3,"Invoke-AtomicTest T1016 -TestNumbers 3"
     T1083 - 3,"Invoke-AtomicTest T1083 -TestNumbers 3"
     T1083 - 4,"Invoke-AtomicTest T1083 -TestNumbers 4"
     T1049 - 3,"Invoke-AtomicTest T1049 -TestNumbers 3"
     T1057 - 1,"Invoke-AtomicTest T1057 -TestNumbers 1"
     T1069.001 - 1,"Invoke-AtomicTest T1069.001 -TestNumbers 1"
     T1201 - 1,"Invoke-AtomicTest T1201 -TestNumbers 1"
     T1201 - 2,"Invoke-AtomicTest T1201 -TestNumbers 2"
     T1201 - 3,"Invoke-AtomicTest T1201 -TestNumbers 3"
     T1201 - 4,"Invoke-AtomicTest T1201 -TestNumbers 4"
     T1201 - 5,"Invoke-AtomicTest T1201 -TestNumbers 5"
     T1614.001 - 3,"Invoke-AtomicTest T1614.001 -TestNumbers 3"
     T1614.001 - 4,"Invoke-AtomicTest T1614.001 -TestNumbers 4"
     T1614.001 - 5,"Invoke-AtomicTest T1614.001 -TestNumbers 5"
     T1614.001 - 6,"Invoke-AtomicTest T1614.001 -TestNumbers 6"
     T1614 - 2,"Invoke-AtomicTest T1614 -TestNumbers 2"
     T1518.001 - 4,"Invoke-AtomicTest T1518.001 -TestNumbers 4"
     T1518.001 - 5,"Invoke-AtomicTest T1518.001 -TestNumbers 5"
     T1018 - 6,"Invoke-AtomicTest T1018 -TestNumbers 6"
     T1018 - 7,"Invoke-AtomicTest T1018 -TestNumbers 7"
     T1018 - 12,"Invoke-AtomicTest T1018 -TestNumbers 12"
     T1018 - 13,"Invoke-AtomicTest T1018 -TestNumbers 13"
     T1018 - 14,"Invoke-AtomicTest T1018 -TestNumbers 14"
     T1018 - 15,"Invoke-AtomicTest T1018 -TestNumbers 15"
     T1046 - 1,"Invoke-AtomicTest T1046 -TestNumbers 1"
     T1046 - 2,"Invoke-AtomicTest T1046 -TestNumbers 2"
     T1046 - 12,"Invoke-AtomicTest T1046 -TestNumbers 12"
     T1124 - 3,"Invoke-AtomicTest T1124 -TestNumbers 3"
     T1489 - 4,"Invoke-AtomicTest T1489 -TestNumbers 4"
     T1489 - 5,"Invoke-AtomicTest T1489 -TestNumbers 5"
     T1489 - 6,"Invoke-AtomicTest T1489 -TestNumbers 6"
     T1489 - 7,"Invoke-AtomicTest T1489 -TestNumbers 7"
     T1531 - 4,"Invoke-AtomicTest T1531 -TestNumbers 4"
     T1486 - 1,"Invoke-AtomicTest T1486 -TestNumbers 1"
     T1486 - 2,"Invoke-AtomicTest T1486 -TestNumbers 2"
     T1486 - 3,"Invoke-AtomicTest T1486 -TestNumbers 3"
     T1486 - 4,"Invoke-AtomicTest T1486 -TestNumbers 4"
     T1496 - 1,"Invoke-AtomicTest T1496 -TestNumbers 1"
     T1485 - 2,"Invoke-AtomicTest T1485 -TestNumbers 2"
     T1529 - 3,"Invoke-AtomicTest T1529 -TestNumbers 3"
     T1529 - 4,"Invoke-AtomicTest T1529 -TestNumbers 4"
     T1529 - 5,"Invoke-AtomicTest T1529 -TestNumbers 5"
     T1529 - 6,"Invoke-AtomicTest T1529 -TestNumbers 6"
     T1529 - 7,"Invoke-AtomicTest T1529 -TestNumbers 7"
     T1529 - 8,"Invoke-AtomicTest T1529 -TestNumbers 8"
     T1529 - 9,"Invoke-AtomicTest T1529 -TestNumbers 9"
     T1529 - 10,"Invoke-AtomicTest T1529 -TestNumbers 10"
     T1529 - 11,"Invoke-AtomicTest T1529 -TestNumbers 11"
     T1078.003 - 8,"Invoke-AtomicTest T1078.003 -TestNumbers 8"
     T1078.003 - 9,"Invoke-AtomicTest T1078.003 -TestNumbers 9"
     T1078.003 - 10,"Invoke-AtomicTest T1078.003 -TestNumbers 10"
     T1078.003 - 11,"Invoke-AtomicTest T1078.003 -TestNumbers 11"
     T1078.003 - 12,"Invoke-AtomicTest T1078.003 -TestNumbers 12"
     T1048.002 - 2,"Invoke-AtomicTest T1048.002 -TestNumbers 2"
     T1048.002 - 3,"Invoke-AtomicTest T1048.002 -TestNumbers 3"
     T1048.002 - 4,"Invoke-AtomicTest T1048.002 -TestNumbers 4"
     T1048 - 1,"Invoke-AtomicTest T1048 -TestNumbers 1"
     T1048 - 2,"Invoke-AtomicTest T1048 -TestNumbers 2"
     T1048 - 4,"Invoke-AtomicTest T1048 -TestNumbers 4"
     T1567.002 - 2,"Invoke-AtomicTest T1567.002 -TestNumbers 2"
     T1030 - 1,"Invoke-AtomicTest T1030 -TestNumbers 1"
     T1048.003 - 1,"Invoke-AtomicTest T1048.003 -TestNumbers 1"
     T1048.003 - 3,"Invoke-AtomicTest T1048.003 -TestNumbers 3"
     T1048.003 - 8,"Invoke-AtomicTest T1048.003 -TestNumbers 8"
     T1053.003 - 1,"Invoke-AtomicTest T1053.003 -TestNumbers 1"
     T1053.003 - 2,"Invoke-AtomicTest T1053.003 -TestNumbers 2"
     T1053.003 - 3,"Invoke-AtomicTest T1053.003 -TestNumbers 3"
     T1053.003 - 4,"Invoke-AtomicTest T1053.003 -TestNumbers 4"
     ''')
     LET CommandsToRun <= if(condition=RunAll, then='''Invoke-AtomicTest All -Confirm:$false''', else={ SELECT Command FROM CommandTable WHERE get(field=Flag)})

     LET RemoveLog <= if(condition=RemoveExecLog, then={ SELECT * FROM execve(argv=["pwsh", "-Command", "Remove-Item", ExecutionLogFile])})
     
     LET EnsureExecutionLog <= if(
           condition = EnsureExecLog,                  
           then      = {
               SELECT * FROM execve(
                   argv=[
                       'pwsh', '-exec', 'bypass',
                       '-Command',
                       '''if (-not (Test-Path -Path "''' + ExecutionLogFile +
                       '''")) { New-Item -Path "'''  + ExecutionLogFile +
                       '''" -ItemType File -Force | Out-Null }'''
                   ]
               )
           }
       )


      LET JustDoIt <= SELECT * FROM foreach(row=CommandsToRun, query={
            SELECT * FROM execve(argv=[   'pwsh', '-exec', 'bypass',
                                                '-Command', '''Import-Module "/usr/local/share/powershell/Modules/Invoke-AtomicRedTeam/Invoke-AtomicRedTeam.psd1" -Force; ''' + Command + ''' -GetPreReqs; ''' + Command + ''' -ExecutionLogPath ''' + ExecutionLogFile + ''';'''
                                            ])}

     )

     SELECT `Execution Time (UTC)`, `Execution Time (Local)`, '[' + Technique + '](https://github.com/redcanaryco/atomic-red-team/blob/master/atomics/' + Technique + '/' + Technique + '.md)' AS Technique, `Test Number`, `Test Name`, Hostname, Username, GUID FROM parse_csv(accessor="file", filename=ExecutionLogFile)

```
---

<img width="1790" height="812" alt="image" src="https://github.com/user-attachments/assets/5d5cc862-1e31-4a79-a86b-5cf438da6d86" />
<img width="1803" height="570" alt="image" src="https://github.com/user-attachments/assets/7546ab43-6a25-4fe1-91e3-cdb6ed626359" />
<img width="1828" height="842" alt="image" src="https://github.com/user-attachments/assets/e854413f-0dbe-4176-8d76-b9624b8676e5" />


How to Run Artifacts:

**Hunt Manager => New Hunt => Configure Hunt => Select Artifact => Configure Paremeters => launch**

<img width="1796" height="572" alt="image" src="https://github.com/user-attachments/assets/5d0e29ea-6412-4730-a03d-4749a7f776e5" />

<img width="1796" height="122" alt="image" src="https://github.com/user-attachments/assets/85763537-7192-41bc-bd45-137a149e955c" />

<img width="1618" height="795" alt="image" src="https://github.com/user-attachments/assets/b0a84066-2db1-4de0-8b4d-2d1b30a3526d" />

<img width="1803" height="845" alt="image" src="https://github.com/user-attachments/assets/991ccab7-3265-4b26-bf9f-e617e03e8f03" />

<img width="1618" height="804" alt="image" src="https://github.com/user-attachments/assets/adf158d7-4eb4-46fc-aa76-ca27c53e7322" />

<img width="1796" height="818" alt="image" src="https://github.com/user-attachments/assets/2ad3db16-0d01-47f2-90a4-68d4dc2c7825" />

<img width="1744" height="230" alt="image" src="https://github.com/user-attachments/assets/a5eb4be5-2cdb-4342-a772-cd38a44b7c9b" />

<img width="1797" height="837" alt="image" src="https://github.com/user-attachments/assets/893f11f9-0358-45ae-9590-67fc16d71245" />

---

## Phase 5:Wazuh Host Monitoring

 **Goal: Detect suspicious host behavior from the victim's machine.**
Steps:
- Install Wazuh Agent on Kali Linux.
- Set the manager IP to Ubuntu host.
- Start Wazuh agent service.
- View alerts in Wazuh dashboard or log files.

## 📸Wazuh Agent

![wazuh_agent](https://github.com/user-attachments/assets/df124747-423a-4902-aff7-e3fa9e5cb803)

---


## Phase 6: MITRE ATT&CK Events in Wazuh (via Kibana/Elastic)

1.**Open Kibana Web Interface**

  - Go to your browser and visit:
```bash
https://<your-wazuh-manager-ip>:5601
```
  - Log in with your Kibana/Wazuh dashboard credentials.

2. **Navigate to the Wazuh App**

  - In the left-hand menu, click on Wazuh .
  - It's redirected to the Wazuh Security Module dashboard.

3. **Select Your Agent**

  - Click on the tab or dropdown showing your agent's name, e.g.,
     **anik_kali (004)**
  - This filters events related to that specific agent.

4. **Click on the MITRE ATT&CK Tab**
  - At the top of the Wazuh module, click on MITRE ATT&CK.

5.**Switch to the Events Tab**

  - Within the MITRE ATT&CK section,see sub-tabs like:

     - Intelligence
     - Framework
     - Dashboard
     - Events ← Clicked this

6. **Analyze Events**
  - Now see all the events with:

      - MITRE IDs (e.g., T1078)
      - Tactics (e.g., Defense Evasion, Privilege Escalation)
      - Rule description
      - Rule ID and severity level

![wazuhdetected](https://github.com/user-attachments/assets/90c31c63-b3ed-4f4a-94d4-1f41e7dae36e)

![detection](https://github.com/user-attachments/assets/7e8f46e4-9c4c-4c0d-9dd4-56c196c7db6b)

![root](https://github.com/user-attachments/assets/3e6fc6bb-11dd-4a32-acab-8fc7cb4c5222)

![Root Detected](https://github.com/user-attachments/assets/53f05378-2c18-4d14-9027-07b014a61e88)

---

## Conclusion

This lab provides for you a complete hands-on simulation environment which emulates the Linux-based adversarial techniques as well as tests real-world detection capabilities by way of open-source tools. By leveraging Wazuh, Velociraptor, and Atomic Red Team, defenders gain a practical comprehension of how various attack vectors operate. Effective monitoring and investigation are enabled by these tools providing SIEM/XDR, forensics and endpoint visibility, and technique simulation.


This setup lets users do these things:

- For framework context, mimic precise methods such as T1059.004 to comprehend MITRE ATT&CK.
- To enable precise detection, visualize how endpoint actions generate security events plus how Wazuh rules can be customized.
- Suspicious behaviors can be identified through Velociraptor’s artifact collection as well as its live response.
- PowerShell simulates attacks on Linux a nontraditional yet increasingly critical target.
- Detection engineering principles as well as incident response workflows must be learned.


For cybersecurity students and also for researchers along with blue teamers, this structured educational lab is empowering. It gives to them a repeatable framework for safely improving detection and investigation skills in a controlled setting.

---




