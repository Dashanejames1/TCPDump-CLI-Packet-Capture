# 🔍 TCPDump-CLI-Packet-Capture

**Author:** Dashane James  
**Lab Environment:** [e.g. VMware Workstation | Kali Linux | Metasploitable 2|TCPDump]  
**Purpose:** [Master TCPDump for headless server environments where Wireshark is unavailable.]  
**Status:** 🟢 Active / 🟡 In Progress / 🔵 Completed

---

## 📋 Overview

[Write 2-3 sentences describing what this project is, what you did, and why. Example: "This repository documents a series of ______ assessments performed in an isolated VMware home lab. The goal was to build hands-on proficiency with ______ aligned with CySA+ exam objectives."] 

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

- **[TCPDump]** — [What it does]
- **[Tool 2]** — [What it does]
- **[Tool 3]** — [What it does]
- **[Tool 4]** — [What it does]

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
Port/Service	Tool Used	Risk Level	Notes
[e.g. 21/tcp FTP]	[e.g. Nmap]	🔴 Critical	[Notes]
[e.g. 23/tcp Telnet]	[e.g. TCPDump]	🔴 Critical	[Transmitted session data, in plaintext. Captured directly via 'tcpdump -A']
[e.g. 80/tcp HTTP]	[e.g. Nikto]	🟠 High	[Notes]
[e.g. 3306/tcp MySQL]	[e.g. OpenVAS]	🟠 High	[Notes]
[e.g. 22/tcp SSH]	[e.g. Nmap]	🟡 Medium	[Notes]
Risk Levels: 🔴 Critical | 🟠 High | 🟡 Medium | 🟢 Low

🗺️ MITRE ATT&CK Mapping
Action Performed	ATT&CK Tactic	Technique ID	Technique Name
[e.g. Port scanning]	[e.g. Reconnaissance]	[e.g. T1595]	[e.g. Active Scanning]
[Action]	[Tactic]	[ID]	[Technique]
[Action]	[Tactic]	[ID]	[Technique]
[Action]	[Tactic]	[ID]	[Technique]
🛡️ Defensive Recommendations
Based on findings, the following remediations would be recommended in a real environment:

[Finding 1] — [Recommendation]
[Finding 2] — [Recommendation]
[Finding 3] — [Recommendation]
[Finding 4] — [Recommendation]
[Finding 5] — [Recommendation]
📚 CySA+ Exam Relevance
This lab directly maps to the following CompTIA CySA+ (CS0-003) exam domains:

Domain	Coverage
Security Operations (33%)	[What this lab covers in this domain]
Vulnerability Management (30%)	[What this lab covers in this domain]
Incident Response (20%)	[What this lab covers in this domain]
Reporting & Communication (17%)	[What this lab covers in this domain]
🔑 Technical Notes
[Any important notes about your lab setup, workarounds, or lessons learned. Example:

"Always add the -n flag to Nmap scans in this VMware environment to prevent DNS resolution hangs."]

-sP and -sT contradict eachother (Can't be used together because -sP means just do a ping/host-discovery sweep, skip ports entirely but -sT tells Nmap to do a full TCP connect port scan. These commands contradict eachother.)

# Any important commands or workarounds
-w capture.pcap - means -w(write file to:) and capture.pcap is the file the packets captured are being sent to that are later analyzed by Wireshark.

wireshark ~/capture.pcap - opens the file in wireshark directly in one step.

Flags used in TCPDump: 
-i(interface)
-w(write)
-r(read)
-n(No DNS)
-A(ASCII output)
📌 About This Project
[1-2 sentences about how this fits into your overall portfolio and career goals.]

Related repositories:

[Repo Name] — [Brief description]
[Repo Name] — [Brief description]
[Repo Name] — Coming soon
👤 Author
Dashane James
Senior Field Service Technician → Cybersecurity Analyst
📍 Yonkers, NY
🎓 B.S. Information Technology — SUNY Canton
🏆 CompTIA Security+ | CySA+ (In Progress)
🔗 GitHub | Zero Trust Cyber Security Brand

This repository is part of an active portfolio demonstrating hands-on cybersecurity skills. All lab work performed in isolated environments for educational purposes.

