# 🏴‍☠️ Nmap Master Cheat Sheet

Nmap (Network Mapper) is a free and open-source utility for network discovery and security auditing. This cheat sheet covers everything from basic host discovery to advanced stealth scanning techniques used by real-world penetration testers.

---

## 1️⃣ Host Discovery (Kaun Online Hai?)
Before scanning ports, we need to find which devices are alive on the network.

| Command | Explanation (Kaam Kya Karta Hai) | Use Case (Kab Use Karein) |
| :--- | :--- | :--- |
| `nmap -sn <Target>` | **Ping Scan:** Finds live hosts without scanning their ports. | Fast network mapping. Less noisy. |
| `nmap -sL <Target>` | **List Scan:** Only lists the IPs that will be scanned. | To verify target range before an actual scan. |
| `nmap -Pn <Target>` | **No Ping:** Treats all hosts as online. | Used when the target blocks ICMP (Ping) requests. |
| `nmap -PS<port> <Target>` | **TCP SYN Ping:** Pings specific TCP ports (default 80). | Bypasses firewalls that block ICMP but allow TCP. |

*Example Target Format:* `192.168.1.1-10` (Range), `192.168.1.0/24` (Subnet), `target.com` (Domain).

---

## 2️⃣ Port Scanning (Kaunse Darwaze Khule Hain?)
This is the core of Nmap. Discovering open TCP/UDP ports.

| Command | Explanation | Use Case / Hacker Perspective |
| :--- | :--- | :--- |
| `nmap -sS <Target>` | **SYN Scan (Stealth):** Only sends the first SYN packet. Never completes handshake. | **Hacker's Favourite.** It's fast and stealthy. Won't appear in target's application logs. (Requires `sudo`). |
| `nmap -sT <Target>` | **TCP Connect Scan:** Completes the full 3-way handshake. | Used when you don't have root privileges. Noisy (gets logged). |
| `nmap -sU <Target>` | **UDP Scan:** Scans UDP ports (DNS, SNMP, DHCP). | Slower than TCP. Used to find UDP services. |
| `nmap -p- <Target>` | **All Ports:** Scans all 65,535 ports. | Thorough scan. Takes a long time. |
| `nmap -p 80,443 <Target>` | **Specific Ports:** Scans only the defined ports. | Fast and targeted. |
| `nmap -F <Target>` | **Fast Mode:** Scans only the top 100 common ports. | Quick recon when time is limited. |

---

## 3️⃣ Service & OS Detection (Kya Chal Raha Hai?)
Figuring out the exact software and OS running on the target.

| Command | Explanation | Use Case |
| :--- | :--- | :--- |
| `nmap -sV <Target>` | **Version Detection:** Interacts with the port to find the exact software version (e.g., Apache 2.4.41). | Crucial for finding known CVEs/Exploits for that version. |
| `nmap -O <Target>` | **OS Detection:** Guesses the target's Operating System using TCP/IP stack fingerprinting. | To know if the target is Windows, Linux, or a Router. |
| `nmap -A <Target>` | **Aggressive Scan:** Runs `-sV`, `-O`, default scripts (`-sC`), and traceroute all at once. | The "All-in-One" command. Very noisy but gives maximum info. |

---

## 4️⃣ Timing & Evasion (Speed vs. Stealth)
Controlling how fast Nmap runs. Fast scans get caught; slow scans take forever.

| Command | Explanation | Use Case |
| :--- | :--- | :--- |
| `nmap -T4 <Target>` | **Aggressive Timing:** Very fast. | The **standard** for most CTF and lab environments. |
| `nmap -T0 <Target>` | **Paranoid Timing:** Extremely slow (waits 5 mins between ports). | Used to evade strict Intrusion Detection Systems (IDS). |
| `nmap -T5 <Target>` | **Insane Timing:** Fastest possible. | Can cause packet loss or crash the target. |
| `--max-rate 10` | Limits Nmap to 10 packets per second. | Throttles bandwidth. Good for stealth. |

---

## 5️⃣ Output & Formatting (Result Save Karna)
Saving scan results for reporting or later analysis.

| Command | Explanation | Use Case |
| :--- | :--- | :--- |
| `nmap -oN scan.txt` | **Normal Output:** Saves the standard screen output to a text file. | For reading later. |
| `nmap -oX scan.xml` | **XML Output:** Saves in XML format. | Used by tools like Metasploit, Burp Suite, or reporting tools. |
| `nmap -oG scan.gnmap` | **Grep-able Output:** Format that is easy to search using `grep`. | Great for extracting specific IPs/Ports via scripts. |
| `nmap -oA scan_all` | **All Formats:** Saves in Normal, XML, and Grep-able formats simultaneously. | **Best Practice:** Always use this for professional engagements. |
| `nmap -v` or `-vv` | **Verbose:** Shows live scan progress on screen. | Useful for debugging slow scans. |

---

## 🚀 Real-World Hacker Combinations (The "One-Liners")

These are the combinations pentesters use in actual engagements.

### 1. The "Quick Recon" (Fast & Effective)
`nmap -sS -sV -T4 --top-ports 100 <Target>`
*Explained: Stealth scan the top 100 ports, find versions, and do it fast.*

### 2. The "Full Pentest" (Deep Dive)
`nmap -p- -sV -sC -A -T4 -oA full_scan <Target>`
*Explained: Scan all 65,535 ports, run default vulnerability scripts, detect OS/Version, use aggressive timing, and save all formats.*

### 3. The "Stealth Scan" (Bypass IDS)
`sudo nmap -sS -T1 -f -D RND:10 -p 22,80,443 <Target>`
*Explained: Slow timing (T1), fragment packets (-f), and use 10 random decoy IPs (-D RND:10) to mask your source IP.*

### 4. The "Ping Block Bypass" (Firewall Evasion)
`nmap -Pn -sS -T4 -p 80,443,22 <Target>`
*Explained: Target blocks Ping (-Pn), but we scan specific ports anyway with stealth and speed.*

---
*Cheat sheet created by [Your Name] - Keep Hacking!*
