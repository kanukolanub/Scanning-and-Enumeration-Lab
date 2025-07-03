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
 
### Detection
1. Some dropped TCP attempts with src-ip, dst-ip, tcp flags (flag S (for SYN) logged in pfirewall.log.
2. nmap -sS -Pn scanned an open port on Windows VM as shown in the screen shot, so Event ID: 5156 captured in Security log.
   with src-ip, dst-ip, src port, destination port: 3389.

### 2️⃣ TCP Connect Scan
Command:
nmap -sT <target-ip>

### Detection
1. Allow TCP attempt with initial SYN Packet with src-ip, dst-ip, dst port: 3389 logged in pfirewall.log.
2. The combination of Nmap's "open" report and the inbound ALLOW entries in both logs (pfirewall & Security logs) provides more than enough confirmation that the Windows firewall successfully allowed the full connection to Port 3389.
3. Event ID 5156 in security log indicates that
•The RDP service (or whatever service is on 3389) successfully bound to its port a while ago (likely at system startup or service start).
•	Your firewall is actively logging the actual network connections (5156) as they occur.
4. Nmap reports 3389/tcp open ms-wbt-server: This is the most crucial piece of evidence. Nmap only reports a TCP port as "open" after successfully completing the full TCP 3-way handshake (SYN, SYN-ACK, ACK). This proves that the outgoing SYN-ACK packet was sent by your Windows VM and received by Nmap. The connection literally could not have been established otherwise.

### 3️⃣ UDP Scan with Netcat Service
Command:
nmap -sU -p <port> <target-ip>
Setup:
Netcat UDP listener on Windows target:
nc -ul -p <port>

### Detection
POSITIVE SCENARIO:
1. When inbound Allow rule on windows firewall for specific UDP service such as Ncat 12345 port. Then pfirewall log shows Allow UDP entries.
2. And Security log in Event viewer reflects entry with Event ID 5156 with src-ip, dst-ip and destination port 12345.
3. Nmap report should show 12345/udp open.

NEGATIVE SCENARIO:
1. No Ncat service is running on windows VM.
2. Also outbound rule is set to capture windows response with an ICMP type 3 code 3 destination unreachable on windows firewall.
3. Nmap report should show 12345/udp closed.
4. pfirewall log should have DROP UDP entry and Security log should have Event IDs 5152.
