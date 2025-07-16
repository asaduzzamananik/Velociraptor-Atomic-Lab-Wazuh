# Linux-Atomic-Wazuh-Simulation-Lab
Complete End-to-End Lab: Simulating Linux MITRE ATT&amp;CK with Atomic Red Team, Velociraptor &amp; Wazuh

## Project Objective
This open-source security lab demonstrates how to simulate MITRE ATT&CK techniques on Linux and detect them using real-world blue team tooling. The goal is to help learners understand attacker behavior and validate detection using modern EDR and DFIR frameworks.

Key Goals:
  - Understand and simulate MITRE ATT&CK behavior using Atomic Red Team (e.g., T1059.004)
  - Detect adversarial activity with Wazuh SIEM/EDR
  - Hunt and collect artifacts using Velociraptor
  - Provide a fully documented and beginner-friendly guide with tooling explanations

This project helps bridge the gap between offensive simulation and defensive monitoring through hands-on practice

 ## Network Architecture
 [ Ubuntu Server (10.0.2.15) ]
  ├─ Wazuh Manager (SIEM/EDR)
  └─ Velociraptor Server (Forensics)

[ Kali Linux (10.0.2.17) ]
  ├─ Wazuh Agent
  ├─ Velociraptor Client
  └─ Atomic Red Team Testbed

## Virtual Machines (VirtualBox)
| **VM NAME**  | **Network Adapter** | **Purpose** | **Tools Used** |
|---------------|-------------|---------------|---------------|
| **Ubuntu**    | **Adapter 1: NAT Network**  | Wazuh Manager + Velociraptor  | Detection, logging, Wazuh , Velociraptor) |             
| **Kali Linux** | **Adapter 1: NAT Network**   | Test Agent (Attacker) | Simulate techniques using Atomic Red Team |


**Network Settings for All VMs**

Select Adapter 1:

Check Enable Network Adapter
Attached to: NAT Network
Adapter Type: Intel PRO/1000 MT Desktop (default is fine)
Cable Connected: Checked
Select Adapter 2:

Repeat the same for Kali Linux And Ubuntu

## Ubuntu Network Config
<img width="717" height="569" alt="image" src="https://github.com/user-attachments/assets/aaec7cc4-c0ec-461a-8659-5b108470ca00" />

## kali Network Config
<img width="712" height="594" alt="image" src="https://github.com/user-attachments/assets/585d6356-2ad3-402b-a0de-1e1ed46aca11" />

## Tool Overview: Why Each Tool Was Used

| **Tool**  | **What It Is** | **Why It Was Used** | 
|---------------|-------------|---------------|
| **Wazuh** | Open-source XDR/SIEM platform  | To detect attacker behavior via log analysis and provide alerting/visuals |
| **Velociraptor** | DFIR (Digital Forensics and Incident Response) Framework | To collect and query forensic data on endpoints |
| **Atomic Red Team** | Library of scripted adversary techniques from MITRE ATT&CK | To simulate real-world attack behaviors in a controlled lab environment |
| **PowerShell** | Cross-platform version of Windows PowerShell | Required to execute Atomic Red Team techniques written in PowerShell |

## Lab Setup 

## Ubuntu Server Phase (Wazuh + Velociraptor)

### Phase 1: Install Wazuh Manager

This step sets up the main SIEM/XDR system for alerting.

```bash
curl -sO https://packages.wazuh.com/4.7/wazuh-install.sh
sudo bash ./wazuh-install.sh -a
```

**⚠️ Note: In this lab, Wazuh was installed using this community repo for ease: https://github.com/samiul008ghub/soc_setup**
Dashboard: https://10.0.2.15:5601

<img width="1809" height="884" alt="image" src="https://github.com/user-attachments/assets/28c80240-f939-4088-9ab6-eaf4f6c353b3" />

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

Dashboard: https://10.0.2.15:8000

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





## Prepare Kali Linux Attack Host

1. Install PowerShell on Linux
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

3. Installing Invoke AtomicRedTeam
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

