# 🔍 TCPDump-CLI-Packet-Capture

**Author:** Dashane James  
**Lab Environment:** [e.g. VMware Workstation | Kali Linux | Metasploitable 2|TCPDump]  
**Purpose:** [Master TCPDump for headless server environments where Wireshark is unavailable.]  
**Status:** 🟢 Active / 🟡 In Progress / 🔵 Completed

---

## 📋 Overview

[This repository documents a series of packet capture exercises performed using TCPDump in an isolated VMware home lab against Metasploitable 2. The goal was to build hands-on proficiency with command-line packet capture — including filtering by host, saving captures to file for later analysis in Wireshark, and reading raw plaintext session data — for environments where a GUI tool like Wireshark isn't available.] 

T
---

## 🧪 Lab Environment
| Component | Details |
|---|---|
| Hypervisor | VMware Workstation (Host-Only Network) |
| Attacker Machine | Kali Linux 2026.1 — `192.168.79.129` |
| Target Machine | [Metasploitable] — `192.168.79.130` |
| Network Type | Host-Only (isolated, no internet exposure) |
| Host OS | Windows 11 — ASUS Vivobook 14 |

> ⚠️ **Note:** All activity was performed in a controlled, isolated lab environment against deliberately vulnerable machines. No unauthorized access to live networks was performed.

---

## 🛠️ Tools Used

- **TCPDump** — Command-line packet capture tool used to filter, capture, and save network traffic without a graphical interface
- **Wireshark** — Used to open and analyze `.pcap` files saved by TCPDump, demonstrating compatibility between the two tools
- **Telnet** — Used to generate a live plaintext session against Metasploitable to demonstrate unencrypted credential/session exposure
---

## 📁 Repository Structure

```
[Repo-Name]/
├── README.md
├── [folder-1]/
│   ├── [file1.txt]        # Description of file
│   └── [file2.md]         # Description of file
├── [folder-2]/
│   ├── [file3.md]         # Description of file
│   └── [file4.txt]        # Description of file
└── reports/
    └── [final-report.md]  # Full assessment report
```

---

## 🔬 Tasks / Assessments Performed

### 1. [Start a TCPDump capture on eth0, filtered to only show traffic to/from Metasploitable.]

For this task, I opened command prompt in Kali and started a tcp capture using a manual host filter to only show traffic to/from metasploitable. The purpose was to demonstrate using TCPDump to replicate the kind of traffic capture traffic from the target host as we did in the previous tasks using Wireshark. This shows that TCPDump has anlot of the same capabilities as wireshark without the GUI as a command line-only packet capture tool.
# Command used
[sudo tcpdump -i eth0 host 192.168.79.130]

Output

The output displays live traffic which seems to be Metasploitable's background system chatter. I didn't initiate any traffic here so all that is displayed is the "background noise". Metasploitable is asking "Who has this IP" and getting an answer with a MAC address. This is Metasploitable trying to find another device (.254) likely a DHCP server.

<img width="323" height="256" alt="Screenshot 2026-07-23 235433" src="https://github.com/user-attachments/assets/8f50ceac-a1e5-4448-a283-9a7a6651fd1e" />

Findings: 
I found that TCPDump is most useful in an environment that had a server with no graphical interface, or when you need something fast and lightweight via' SSH.This is also useful in real SOC/Server environments for quick, scriptable, GUI-less packet captures. It is a great tool to use alongside Wireshark with TCPDumps lightweight capture with Wireshark's much richer analysis GUI. Also I noticed both Wireshark and TCPDump displays some type of background chatter which an analyst would have to sort through and distinguish the difference from more critical data.


### 2. [Save a capture to a .pcap file, then open it in wireshark.]
[For this task instead of just running a tcp capture and printing it to the screen as usual, this time I saved the output to a file. The reason for doing this was so that I can then open the file in wireshark for further reconaissance.]

# Command used
[sudo tcpdump -i eth0 -w capture.pcap]

## Output

The output produced just states that tcpdump is listening on eth0 and shows the snapshot length but doesn't say anything else until the command is forced to stop (Ctrl+C). Once it stops it shows the amount of packets captured and packets dropped, if any. No other output is shown as the actual data is saved to the indicated pcap file. I then opened the pcap file in wireshark and confirmed that TCPDump's saved output is fully compatible with Wireshark for deeper analysis. 

<img width="326" height="101" alt="Screenshot 2026-07-24 192949" src="https://github.com/user-attachments/assets/0f45cb1c-5e8c-4e3c-a67a-a9de09bf235b" />

<img width="856" height="401" alt="Screenshot 2026-07-24 203700" src="https://github.com/user-attachments/assets/5fde7b17-6025-4ee0-a16d-0f8aa1e99dbd" />


Finding: [What did you discover?]
I found that the main reason for using TCPDump instead of Wireshark is that it can be used in situations where a server may have a "headless" remote configuration with no GUI available. This makes it much more lightweight/portable for those environments, while Wireshark still offers much deeper analysis capabilities. However, TCPDump and Wireshark do work well together because TCPDump can capture traffic in environments where Wireshark can't run and that capture can later be opened in Wireshark for further analysis.

### 3. [Capture port 23 telnet traffic using -A flag. Note the Output.]
[I ran a command to capture telnet traffic using TCPDump with -A flag to display packet contents in ASCII. This allows me to read the raw session data in plaintext while logging into metasploitable via telnet from another terminal. ]

# Command used
[sudo tcpdump -i eth0 port 23 A -n]

## Output
The output shows the live Telnet session between Kali and Metasploitable on port 23. Because the -A flag displays packet contents in readable ASCII, fragments of the actual session. Real data such as the shell prompt(msf@metsasploitable) were visibly directly in the TCPDump output, confirming data was transmitted in clear text.

<img width="648" height="281" alt="Screenshot 2026-07-25 110643" src="https://github.com/user-attachments/assets/47c48edb-967a-4b66-9443-918c3e6a2478" />

<img width="650" height="265" alt="Screenshot 2026-07-25 110911" src="https://github.com/user-attachments/assets/e5c84c32-68ce-4192-9162-487809b1e2a1" />


Finding: [I found that using TCPDump with the -A flag while telnetting into Metasploitable, I captured the login session in plaintext directly in the terminal. Unlike the HTTP example used in previous tasks, no GUI or "Follow Stream" feature is needed to read the text.This confirms that Telnet sends all data completely unencrypted and that even basic packet capture tools are enough to intercept data in a Telnet session. Which proves why Telnet should be disabled and replaced with SSH, which encrypts session data end-to-end.]


📊 Key Findings Summary

| Port/Service | Tool Used | Risk Level | Notes |
|---|---|---|---|
| 23/tcp Telnet | TCPDump | 🔴 Critical | Transmits session data in plaintext — captured directly via `tcpdump -A`, no GUI or stream reconstruction needed |
| N/A (background traffic) | TCPDump | 🟢 Low | Metasploitable's own ARP/DHCP housekeeping traffic — not a vulnerability, but background noise an analyst must learn to distinguish from real activity |

**Risk Levels:** 🔴 Critical | 🟠 High | 🟡 Medium | 🟢 Low

🗺️ MITRE ATT&CK Mapping

| Action Performed | ATT&CK Tactic | Technique ID | Technique Name |
|---|---|---|---|
| Filtered live packet capture on target host | Reconnaissance | T1595 | Active Scanning |
| Captured Telnet session in plaintext via TCPDump | Credential Access | T1040 | Network Sniffing |
| Saved capture to `.pcap` for offline analysis | Collection | T1005 | Data from Local System |


🛡️ Defensive Recommendations
Based on findings, the following remediations would be recommended in a real environment:

1. **Telnet transmits data in plaintext (Critical)** — Disable Telnet entirely and replace with SSH, which encrypts session data end-to-end.
2. **Unrestricted network sniffing possible** — Segment sensitive traffic onto encrypted channels (SSH, TLS) so that packet capture alone cannot reveal credentials or session data.
3. **Background system traffic can mask real threats** — Establish a baseline of normal host chatter (ARP, DHCP) so anomalous traffic is easier to identify during incident response.


🔑 Technical Notes
[Any important notes about your lab setup, workarounds, or lessons learned. Example:


"Always add the -n flag to Nmap scans in this VMware environment to prevent DNS resolution hangs."]

-sP and -sT contradict each other (Can't be used together because -sP means just do a ping/host-discovery sweep, skip ports entirely but -sT tells Nmap to do a full TCP connect port scan. These commands contradict each other.)

# Any important commands or workarounds


# Task 1: Filtered capture — only show traffic to/from Metasploitable
sudo tcpdump -i eth0 host 192.168.79.130

# Task 2: Save a capture to a .pcap file for later analysis in Wireshark
sudo tcpdump -i eth0 -w capture.pcap

# Open a saved .pcap file directly in Wireshark from the terminal
wireshark ~/capture.pcap

# Task 3: Capture Telnet traffic and display contents in readable ASCII
sudo tcpdump -i eth0 port 23 -A -n

# Flags used in TCPDump:
# -i   interface to capture on (e.g. eth0)
# -w   write captured packets to a file
# -r   read packets from a saved file
# -n   don't resolve hostnames/IPs (avoids DNS hangs)
# -A   display packet contents in ASCII (readable plaintext)
# host <IP>   capture filter — only show traffic to/from a specific host
# port <#>    capture filter — only show traffic on a specific port

# Workaround: -w and -A require a space before the flag — e.g. "port 23 A -n"
# without a dash in front of A will not work as intended; always confirm
# each flag is written as its own separate "-X" argument.

# -w capture.pcap - means -w(write file to:) and capture.pcap is the file the packets captured are being sent to that are later analyzed by Wireshark.

# wireshark ~/capture.pcap - opens the file in wireshark directly in one step.

📌 About This Project
[This repository is part of my broader cybersecurity portfolio, focused on building practical, tool-agnostic packet analysis skills — since real-world environments often don't provide a GUI, and analysts need to be comfortable working from the command line alone.]

Related repositories:

- `Nmap-Host-Discovery-and-Lab-Baseline` — Established a baseline of the lab network using ping sweeps and host discovery, documenting the target environment before deeper scanning began
- `Nmap-Scan-Types-SYN-vs-TCP-vs-UDP` — Compared SYN, TCP connect, and UDP scan types against the target, examining differences in speed, stealth, and reliability
- `Wireshark-Capture-and-Analyze-Traffic` — Captured and analyzed the same kind of traffic using Wireshark's GUI, including Follow TCP Stream and display filtering
- `Nmap-Scripting-Engine` — Used Nmap's scripting engine to detect and, in one case, exploit the vsFTPd backdoor also referenced in this repo's Telnet findings
- `Service-Version-Detection-and-OS-Fingerprinting` — Identified exact software versions on Metasploitable and researched CVEs tied to them

👤 Author
Dashane James
Senior Field Service Technician → Cybersecurity Analyst
📍 Yonkers, NY
🎓 B.S. Information Technology — SUNY Canton
🏆 CompTIA Security+ | CySA+ (In Progress)
🔗 GitHub | Zero Trust Cyber Security Brand

This repository is part of an active portfolio demonstrating hands-on cybersecurity skills. All lab work performed in isolated environments for educational purposes.

