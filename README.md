# Scanning-and-Enumeration-Lab
# 🔍 Scanning & Enumeration Detection Lab

## 🧠 Objective
Simulate common attacker reconnaissance techniques using Nmap, and detect scan activity through host and network-level logs. This lab is part of my learning path toward a SOC Level 1 role and builds practical Blue Team detection skills.

---

## 🖥️ Lab Setup

| Component | Details |
|----------|---------|
| Attacker VM | Xubuntu (Kali/Parrot also work) |
| Target VM   | Windows 10 (with Sysmon and Firewall Logging enabled) |
| Network     | Adapter1: NAT (for internet access), Adapter2: Internal Network (VM-to-VM communication) |
| Tools Used  | Nmap, Netcat, Wireshark, Windows Firewall, Sysmon |

---

## 🔧 Tasks Performed

### 1️⃣ SYN Scan (Stealth Scan)
- **Command**:  
  ```bash
  nmap -sS -Pn <target-ip>
  ![image](https://github.com/user-attachments/assets/378a88a4-b7ac-475d-a33c-95271a21b826)

  Detection:
1. Some dropped TCP attempts with src-ip, dst-ip, tcp flags (flag S (for SYN) logged in pfirewall.log.
2. nmap -sS -Pn scanned an open port on Windows VM as seen in the screen shot above, so Event ID: 5156 captured in Security log.
   with src-ip, dst-ip, src port, destination port: 3389.
