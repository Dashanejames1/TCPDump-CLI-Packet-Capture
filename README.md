# 🔍 # TCPDump-CLI-Packet-Capture

**Author:** Dashane James  
**Lab Environment:** VMware Workstation | Kali Linux | Metasploitable 2 | TCPDump  
**Purpose:** Capture and analyze network traffic from the command line, for environments where a GUI tool like Wireshark isn't available.  
**Status:** 🔵 Completed

---

## 📋 Overview

Wireshark is a great tool, but it requires a graphical interface. In real-world environments (headless servers, remote sessions over SSH, systems with no desktop), you often don't have that option. TCPDump fills that gap: it captures packets from the command line with no GUI required.

This repository covers TCPDump in three progressively deeper tasks: filtering live traffic to a specific host, saving a capture to a `.pcap` file that can be opened in Wireshark later, and using the `-A` flag to read an active Telnet session in plaintext ASCII directly from the terminal. The Telnet capture in particular reinforces a finding from the Wireshark lab: unencrypted protocols expose session data to anyone with basic capture tools, no decryption needed.

---

## 🧪 Lab Environment

| Component | Details |
|---|---|
| Hypervisor | VMware Workstation (Host-Only Network) |
| Attacker Machine | Kali Linux 2026.1, `192.168.79.129` |
| Target Machine | Metasploitable 2, `192.168.79.130` |
| Network Type | Host-Only (isolated, no internet exposure) |
| Host OS | Windows 11, ASUS Vivobook 14 |

> ⚠️ **Note:** All activity was performed in a controlled, isolated lab environment against a deliberately vulnerable machine. No unauthorized access to live networks was performed.

---

## 🛠️ Tools Used

- **TCPDump**: Command-line packet capture tool used to filter, capture, and save network traffic without a graphical interface
- **Wireshark**: Used to open and analyze `.pcap` files saved by TCPDump, demonstrating that the two tools work together as a capture-then-analyze workflow
- **Telnet**: Used to generate a live plaintext session against Metasploitable, demonstrating unencrypted credential and session data exposure

---

## 🔬 Tasks / Assessments Performed

### 1. Start a TCPDump capture filtered to traffic from Metasploitable

Rather than capturing everything on the interface (which produces a lot of noise), I used a host filter to limit the capture to only traffic involving the target.

```bash
sudo tcpdump -i eth0 host 192.168.79.130
```

<img width="323" height="256" alt="Screenshot 2026-07-23 235433" src="https://github.com/user-attachments/assets/8f50ceac-a1e5-4448-a283-9a7a6651fd1e" />

**Finding:**

With no deliberate traffic generated, the capture showed Metasploitable's own background network activity, ARP queries from the target trying to locate other devices on the subnet (looking for a DHCP server at .254). This is the same kind of background noise encountered in the Wireshark lab, the value here is in recognizing it and knowing it isn't meaningful.

TCPDump is most useful in environments where a GUI isn't available: headless servers, SSH sessions, or automated capture scripts in a SOC environment. It's also well-suited for working alongside Wireshark, use TCPDump to capture traffic in environments where Wireshark can't run, then open the saved file in Wireshark for deeper analysis.

---

### 2. Save a capture to a `.pcap` file and open it in Wireshark

Instead of printing the capture output to the screen, I saved it directly to a file using the `-w` flag. The `.pcap` format is the standard for packet capture files and is compatible with Wireshark and most other analysis tools.

```bash
# Capture and write to file
sudo tcpdump -i eth0 -w capture.pcap

# Open the saved file in Wireshark
wireshark ~/capture.pcap
```

<img width="326" height="101" alt="Screenshot 2026-07-24 192949" src="https://github.com/user-attachments/assets/0f45cb1c-5e8c-4e3c-a67a-a9de09bf235b" />

<img width="856" height="401" alt="Screenshot 2026-07-24 203700" src="https://github.com/user-attachments/assets/5fde7b17-6025-4ee0-a16d-0f8aa1e99dbd" />

**Finding:**

When writing to a file, TCPDump only prints the interface name and snapshot length, no packet data appears on screen until you stop the capture with Ctrl+C. After stopping, it reports how many packets were captured and how many were dropped (if any). The actual data is in the file.

Opening the saved `.pcap` in Wireshark confirmed full compatibility: all packets captured by TCPDump were readable with Wireshark's full analysis capabilities. This two-tool workflow is practical in real environments, TCPDump handles capture on systems where Wireshark can't run, and Wireshark handles the analysis offline.

---

### 3. Capture Telnet traffic on port 23 using the `-A` flag

The `-A` flag tells TCPDump to display packet contents in readable ASCII rather than just headers. For encrypted protocols, this would show you scrambled data. For Telnet, which sends everything unencrypted, it shows you the live session content directly in the terminal.

I opened a Telnet session from Kali to Metasploitable in a separate terminal while TCPDump was running in this one.

```bash
sudo tcpdump -i eth0 port 23 -A -n
```

<img width="648" height="281" alt="Screenshot 2026-07-25 110643" src="https://github.com/user-attachments/assets/47c48edb-967a-4b66-9443-918c3e6a2478" />

<img width="650" height="265" alt="Screenshot 2026-07-25 110911" src="https://github.com/user-attachments/assets/e5c84c32-68ce-4192-9162-487809b1e2a1" />

**Finding:**

The live session data was visible directly in the TCPDump output, including the shell prompt `msf@metasploitable` appearing in readable ASCII. No GUI, no stream reconstruction, no decryption needed. The `-A` flag was enough to read the Telnet session in real time.

This confirms the same thing the Wireshark lab showed with HTTP, unencrypted protocols expose their full session content to anyone with a packet capture tool on the network segment. The difference here is that TCPDump demonstrates it works without any graphical interface at all. An attacker with SSH access to a machine on the same network as a Telnet server can do this from the command line with a single command.

This is why Telnet should be replaced with SSH in any environment. SSH encrypts session data end-to-end; Telnet does not encrypt anything.

---

## 📊 Key Findings Summary

| Port/Service | Tool Used | Risk Level | Notes |
|---|---|---|---|
| 23/tcp Telnet | TCPDump (`-A`) | 🔴 Critical | Live session data captured in plaintext, including the shell prompt `msf@metasploitable`: directly from the terminal, no GUI or stream reconstruction required |
| Background ARP traffic | TCPDump | 🟢 Low | Metasploitable's own ARP/DHCP housekeeping traffic, not a vulnerability, but background noise an analyst needs to distinguish from actual findings |

**Risk Levels:** 🔴 Critical | 🟠 High | 🟡 Medium | 🟢 Low

---

## 🗺️ MITRE ATT&CK Mapping

| Action Performed | ATT&CK Tactic | Technique ID | Technique Name |
|---|---|---|---|
| Filtered live packet capture on target host | Reconnaissance | T1040 | Network Sniffing |
| Captured Telnet session data in plaintext via `-A` flag | Credential Access | T1040 | Network Sniffing |

---

## 🛡️ Defensive Recommendations

Based on findings, the following remediations would be recommended in a real environment:

1. **Telnet transmits all data in plaintext (Critical)**: Disable Telnet entirely and replace with SSH. SSH encrypts session data end-to-end; Telnet exposes everything, including credentials and active shell sessions, to any host on the same network capable of capturing traffic.
2. **Unrestricted packet capture possible on unsegmented networks**: Sensitive traffic should flow over encrypted channels (SSH, TLS) so that packet capture alone cannot reveal credentials or session contents. Network segmentation reduces the scope of what any single capture point can see.
3. **Background host traffic can mask real threats**: Establishing a baseline of expected network chatter (ARP, DHCP) makes it easier to identify genuinely anomalous traffic during incident response.

---

## 🔑 Technical Notes

```bash
# Task 1: Filtered capture, only show traffic to/from Metasploitable
sudo tcpdump -i eth0 host 192.168.79.130

# Task 2: Save capture to a .pcap file for later analysis in Wireshark
sudo tcpdump -i eth0 -w capture.pcap

# Open a saved .pcap file directly in Wireshark
wireshark ~/capture.pcap

# Task 3: Capture Telnet traffic and display contents in readable ASCII
sudo tcpdump -i eth0 port 23 -A -n

# TCPDump flag reference:
# -i    interface to capture on (e.g. eth0)
# -w    write captured packets to a file
# -A    display packet contents in readable ASCII
# -n    don't resolve hostnames or port names (avoids DNS hangs)
# host  filter, only capture traffic to/from a specific IP address
# port  filter, only capture traffic on a specific port number

# Workaround: the -A flag must have a dash before it as its own argument.
# "port 23 A -n" (missing the leading dash on A) will not work correctly.
# The correct syntax is "port 23 -A -n".

# Note: -w writes packets to the file in binary .pcap format, no output
# appears on screen while capturing. Stop with Ctrl+C to close the file,
# then open it in Wireshark: wireshark ~/capture.pcap
```

---

## 📌 About This Project

This repository demonstrates practical, tool-agnostic packet analysis skills. Real-world environments don't always provide a GUI, and analysts need to be comfortable working from the command line alone.

The plaintext Telnet session captured here reinforces the same finding as the HTTP credentials exposed in the Wireshark lab: unencrypted protocols have no secrets from anyone on the same network with a capture tool. TCPDump makes that point without needing Wireshark at all.

**Related repositories (in order):**

- `Nmap-Host-Discovery-and-Lab-Baseline`: Identified port 23 (Telnet) as open; this lab captures what was moving through that open port
- `Nmap-Scan-Types-SYN-vs-TCP-vs-UDP`: Confirmed the same port list through different scan types; Telnet (port 23) was flagged as high risk in that summary
- `Service-Version-Detection-and-OS-Fingerprinting`: Identified Linux telnetd on port 23 via version detection, confirmed by the session data captured here
- `Wireshark-Capture-and-Analyze-Traffic`: The GUI-based counterpart to this lab; Follow TCP Stream on port 80 revealed HTTP credentials the same way -A revealed Telnet session data here
- `Nmap-Scripting-Engine`: Used NSE scripts against the same Metasploitable target, including the FTP backdoor that port 21 (also identified as open) enabled

---

## 👤 Author

**Dashane James**  
Senior Field Service Technician → Cybersecurity Analyst  
📍 Yonkers, NY  
🎓 B.S. Information Technology, SUNY Canton  
🏆 CompTIA Security+  
🔗 [GitHub](https://github.com/Dashanejames1) | [Zero Trust Cyber Security Brand](https://www.instagram.com/zerotrust_cybersecurity)

---

*This repository is part of an active portfolio demonstrating hands-on cybersecurity skills. All lab work performed in isolated environments for educational purposes.*
