# Linux-Atomic-Wazuh-Simulation-Lab
Complete End-to-End Lab: Simulating Linux MITRE ATT&amp;CK with Atomic Red Team, Velociraptor &amp; Wazuh

## Project Objective
This open-source security lab demonstrates how to simulate MITRE ATT&CK techniques on Linux and detect them using real-world blue team tooling. The goal is to help learners understand attacker behavior and validate detection using modern EDR and DFIR frameworks.
You will:
  - Simulate attacks with Atomic Red Team (e.g., T1059.004)
  - Detect adversarial activity with Wazuh SIEM/EDR
  - Hunt and collect artifacts using Velociraptor
  - Learn the theory, architecture, and configuration details

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

## kali Network Config
<img width="712" height="594" alt="image" src="https://github.com/user-attachments/assets/585d6356-2ad3-402b-a0de-1e1ed46aca11" />

## Ubuntu Network Config







