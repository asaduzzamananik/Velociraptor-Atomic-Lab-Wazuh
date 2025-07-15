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

## Phase 1: Wazuh Server on Ubuntu








