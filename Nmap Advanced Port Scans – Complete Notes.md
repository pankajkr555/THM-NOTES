# Nmap Advanced Port Scans – Complete Notes

> **Topic:** Advanced Nmap Scanning Techniques  
> **Platform:** TryHackMe  
> **Target IP (Lab):** `10.49.187.181`  
> **Purpose:** Understand advanced TCP scans, firewall mapping, spoofing, fragmentation, Idle scans and Nmap troubleshooting.

---

## 1. Introduction

In basic Nmap scanning, we commonly use TCP SYN scans (`-sS`) and TCP Connect scans (`-sT`) to discover open ports and services.

However, sometimes normal scans are not enough.

For example:

- A firewall may block normal SYN packets.
- We may want to understand which ports a firewall allows.
- We may want to examine how a target responds to unusual TCP flags.
- We may need to understand how Nmap determines whether a port is open, closed or filtered.

Nmap provides several advanced scanning techniques for these situations.

This room covers:

1. TCP Null Scan
2. TCP FIN Scan
3. TCP Xmas Scan
4. TCP Maimon Scan
5. TCP ACK Scan
6. TCP Window Scan
7. Custom TCP Scan
8. Source IP Spoofing
9. MAC Address Spoofing
10. Decoy Scan
11. IP Fragmentation
12. Idle (Zombie) Scan
13. Additional Nmap options

---

# 2. Understanding TCP Flags

Before learning advanced scans, understand the TCP flags.

TCP flags are control bits in the TCP header. They help manage communication between two systems.

| TCP Flag | Full Form | Purpose |
|---|---|---|
| SYN | Synchronize | Starts a TCP connection |
| ACK | Acknowledgment | Acknowledges received data |
| FIN | Finish | Gracefully closes a connection |
| RST | Reset | Resets or rejects a connection |
| PSH | Push | Requests immediate delivery of buffered data |
| URG | Urgent | Indicates urgent data |
| ECE | ECN-Echo | Supports Explicit Congestion Notification |
| CWR | Congestion Window Reduced | Indicates congestion response |

### Normal TCP Three-Way Handshake

A normal TCP connection uses three steps:

1. SYN – Client requests a connection.
2. SYN-ACK – Server accepts the request.
3. ACK – Client acknowledges the response.

```text
Client                         Server
  |                              |
  | -------- SYN --------------> |
  |                              |
  | <------ SYN-ACK ------------ |
  |                              |
  | -------- ACK --------------> |
  |                              |
  |       Connection Established |
```

Nmap can manipulate TCP flags to perform different types of scans.

---

# 3. TCP Null Scan (`-sN`)

### What is a Null Scan?

A TCP Null Scan sends a TCP packet without setting any TCP flags.

In simple words, it sends a TCP packet with all TCP flags turned OFF.

### Command

```bash
sudo nmap -sN 10.49.187.181
```

### How does it work?

The target may respond differently depending on the port state.

| Port State | Expected Response |
|---|---|
| Open | Usually no response |
| Closed | RST response |
| Filtered | Usually no response |

Nmap generally interprets these responses as:

- RST received → Closed
- No response → Open or Filtered

**Important:** Null scans cannot reliably distinguish open ports from filtered ports because both may produce no response.

### When to use it?

Use a Null Scan when you suspect a stateless firewall that blocks packets containing the SYN flag but may allow unusual TCP packets.

**Remember:** Null = No TCP flags.

---

# 4. TCP FIN Scan (`-sF`)

### What is a FIN Scan?

A FIN Scan sends a TCP packet with only the FIN flag enabled.

FIN normally indicates that a TCP connection should be closed.

### Command

```bash
sudo nmap -sF 10.49.187.181
```

### How does it work?

| Port State | Expected Response |
|---|---|
| Open | Usually no response |
| Closed | RST response |
| Filtered | Usually no response |

### When to use it?

Use a FIN Scan when you suspect a stateless firewall that blocks normal SYN packets but may allow packets with the FIN flag.

**Remember:** FIN = Finish or close connection.

---

# 5. TCP Xmas Scan (`-sX`)

### What is an Xmas Scan?

An Xmas Scan sends a TCP packet with three flags enabled:

- FIN
- PSH
- URG

It is called an Xmas Scan because multiple TCP flags are switched ON, similar to lights on a Christmas tree.

### Command

```bash
sudo nmap -sX 10.49.187.181
```

### How does it work?

| Port State | Expected Response |
|---|---|
| Open | Usually no response |
| Closed | RST response |
| Filtered | Usually no response |

### When to use it?

Use it when you suspect a stateless firewall that only blocks packets containing the SYN flag.

**Remember:** Xmas = FIN + PSH + URG.

---

# 6. TCP Maimon Scan (`-sM`)

### What is a Maimon Scan?

A Maimon Scan sends a TCP packet with FIN and ACK flags enabled.

### Command

```bash
sudo nmap -sM 10.49.187.181
```

### How does it work?

The scan relies on how the target operating system responds to FIN/ACK packets.

The original behavior associated with older BSD-derived systems is:

| Port State | Expected Response |
|---|---|
| Open | No response |
| Closed | RST response |

However, most modern systems respond similarly for open and closed ports, making this scan less reliable.

### When to use it?

Use it when you suspect the target is running an older BSD-derived operating system.

**Remember:** Maimon = FIN + ACK.

---

# 7. TCP ACK Scan (`-sA`)

### What is an ACK Scan?

An ACK Scan sends TCP packets with only the ACK flag enabled.

Unlike SYN scans, an ACK Scan is not primarily designed to discover open services.

Its main purpose is to understand firewall filtering behavior.

### Command

```bash
sudo nmap -sA 10.49.187.181
```

### How does it work?

| Response | Nmap State |
|---|---|
| RST received | Unfiltered |
| No response | Filtered |
| Certain ICMP unreachable responses | Filtered |

### Important: What does Unfiltered mean?

Unfiltered does NOT mean that the port is open.

It means Nmap can reach the port without the probe being blocked by the filtering device.

For example:

```text
PORT    STATE
22/tcp  unfiltered
80/tcp  unfiltered
443/tcp filtered
```

This tells us about packet filtering, not whether services are running.

### When to use it?

Use an ACK Scan when you want to map which ports a firewall allows or blocks.

**Remember:** ACK Scan = Firewall mapping, not open-port discovery.

---

# 8. TCP Window Scan (`-sW`)

### What is a Window Scan?

A Window Scan is similar to an ACK Scan.

It sends TCP ACK packets and examines the TCP Window field in the returned RST packets.

### Command

```bash
sudo nmap -sW 10.49.187.181
```

### How does it work?

Some operating systems return different TCP Window values depending on whether the port is open or closed.

Nmap uses this difference to classify ports.

Possible results:

- Open
- Closed
- Filtered

**Important:** This behavior is operating-system dependent. Many modern systems do not provide a reliable distinction.

### When to use it?

Use a Window Scan when you want to experiment with firewall mapping or when the target's TCP implementation may provide useful window-size differences.

**Remember:** Window Scan = ACK scan + TCP Window field analysis.

---

# 9. Custom TCP Scan (`--scanflags`)

### What is a Custom Scan?

Nmap allows us to manually specify which TCP flags should be enabled in a packet.

This is useful when we want to test a specific combination of TCP flags.

### Syntax

```bash
sudo nmap --scanflags FLAGS TARGET_IP
```

### Example 1: RST Scan

```bash
sudo nmap --scanflags RST 10.49.187.181
```

This sends packets with only the RST flag enabled.

### Example 2: SYN + FIN Scan

```bash
sudo nmap --scanflags SYNFIN 10.49.187.181
```

This enables both SYN and FIN flags.

### Example 3: Multiple TCP Flags

```bash
sudo nmap --scanflags URGACKPSHRSTSYNFIN 10.49.187.181
```

This enables the listed TCP flags.

### Available TCP flags

```text
FIN
SYN
RST
PSH
ACK
URG
```

**Remember:** `--scanflags` allows us to customize TCP packet flags instead of using a predefined scan type.

---

# 10. Source IP Spoofing (`-S`)

### What is IP Spoofing?

IP Spoofing means sending packets with a forged source IP address.

Normally, a packet contains:

```text
Source IP      = Your IP
Destination IP = Target IP
```

With spoofing:

```text
Source IP      = Forged IP
Destination IP = Target IP
```

### Command

```bash
sudo nmap -S SPOOFED_IP TARGET_IP
```

### Example

```bash
sudo nmap -S 10.10.10.11 10.49.187.181
```

Here:

- `10.10.10.11` = Spoofed source IP
- `10.49.187.181` = Target IP

### Additional Options

You may need to specify the network interface:

```bash
sudo nmap -e tun0 -S 10.10.10.11 -Pn 10.49.187.181
```

Explanation:

| Option | Purpose |
|---|---|
| `-e tun0` | Selects the network interface |
| `-S IP` | Sets the source IP address |
| `-Pn` | Skips host discovery and treats the target as online |

Replace `tun0` with the correct interface on your system.

### Important Limitation

When you spoof your source IP, the target sends its response to the forged IP address.

Therefore, you may not receive the responses needed to determine scan results.

**Remember:** `-S` = Spoof Source IP.

---

# 11. MAC Address Spoofing (`--spoof-mac`)

### What is MAC Spoofing?

Every network interface has a MAC address used for communication on the local network.

MAC Spoofing means changing the MAC address that Nmap uses for scanning.

### Command

```bash
sudo nmap --spoof-mac SPOOFED_MAC TARGET_IP
```

### Example

```bash
sudo nmap --spoof-mac 00:11:22:33:44:55 192.168.1.10
```

### Common Options

Use a specific MAC address:

```bash
sudo nmap --spoof-mac 00:11:22:33:44:55 192.168.1.10
```

Generate a random MAC address:

```bash
sudo nmap --spoof-mac 0 192.168.1.10
```

Choose a vendor-based MAC address using a vendor name:

```bash
sudo nmap --spoof-mac Apple 192.168.1.10
```

### Important Limitation

MAC spoofing is generally useful only on the local Ethernet network segment.

MAC addresses are not routed across the internet. When traffic passes through routers, the Layer 2 frame is rebuilt for the next network hop.

**Remember:**

- IP address = Layer 3
- MAC address = Layer 2

---

# 12. Decoy Scan (`-D`)

### What is a Decoy Scan?

A Decoy Scan makes a target see scan traffic appearing to originate from multiple source IP addresses.

This makes it harder to identify the actual scanning source from the target's logs alone.

### Command

```bash
sudo nmap -D DECOY_IP,ME TARGET_IP
```

### Example

```bash
sudo nmap -sS -p 80 -D 10.10.20.21,10.10.20.28,ME 10.49.187.181
```

Here:

- `10.10.20.21` = First decoy IP
- `10.10.20.28` = Second decoy IP
- `ME` = Your actual source IP
- `10.49.187.181` = Target IP

### Random Decoy Addresses

```bash
sudo nmap -sS -p 80 -D RND,RND,ME 10.49.187.181
```

`RND` tells Nmap to generate a random decoy address.

### When to use it?

In an authorized lab, decoys help demonstrate how multiple apparent source addresses can complicate basic source attribution.

### Important Limitation

Decoy scans do not make a scan invisible.

Network monitoring, packet analysis, routing information and other evidence may still reveal the actual source.

**Remember:** `-D` = Decoys.

---

# 13. IP Fragmentation (`-f` and `-ff`)

### What is IP Fragmentation?

IP Fragmentation means dividing a large IP packet into smaller fragments before transmission.

The receiving system reassembles those fragments to reconstruct the original packet.

### Why is fragmentation used in Nmap?

Some simple packet-filtering systems inspect individual packets.

Fragmentation can make inspection more difficult for basic filtering systems because the complete packet is divided into smaller pieces.

Modern firewalls and IDS/IPS systems can often reassemble fragments, so fragmentation does not guarantee evasion.

### Fragment into 8-byte units (`-f`)

```bash
sudo nmap -sS -p 80 -f 10.49.187.181
```

`-f` requests small IP fragments, typically using 8-byte fragmentation units.

### Fragment into 16-byte units (`-ff`)

```bash
sudo nmap -sS -p 80 -ff 10.49.187.181
```

`-ff` requests fragmentation using 16-byte units.

### Custom Fragment Size (`--mtu`)

```bash
sudo nmap -sS --mtu 24 10.49.187.181
```

The MTU value supplied to this option must be a multiple of 8.

### IP Fragmentation Concepts

| Term | Meaning |
|---|---|
| IP ID | Identifies fragments belonging to the same original IP packet |
| Fragment Offset | Indicates where a fragment belongs in the original packet |
| MF Flag | More Fragments flag; indicates more fragments follow |
| Reassembly | Reconstructing the original IP packet |

**Remember:**

- `-f` = 8-byte fragmentation units
- `-ff` = 16-byte fragmentation units
- `--mtu` = Custom fragmentation size, multiple of 8

---

# 14. Idle Scan / Zombie Scan (`-sI`)

### What is an Idle Scan?

An Idle Scan is an advanced Nmap technique that uses another host as an intermediary.

That intermediary is called a Zombie.

The target sees scan probes that appear to originate from the Zombie rather than directly from the attacker.

### Basic Requirements

A suitable Zombie should:

1. Be reachable from the attacker.
2. Be relatively idle.
3. Have predictable IP ID behavior.
4. Respond in a way that allows Nmap to measure IP ID changes.

### Command

```bash
sudo nmap -sI ZOMBIE_IP TARGET_IP
```

### Example

```bash
sudo nmap -sI 10.10.5.5 10.49.187.181
```

Here:

- `10.10.5.5` = Zombie IP
- `10.49.187.181` = Target IP

The example Zombie IP is illustrative; use a suitable host in your authorized lab.

### How does it work?

An Idle Scan uses the Zombie's IP ID behavior to infer the target's port state.

The process:

1. Nmap checks the Zombie's current IP ID.
2. Nmap sends a probe to the target using the Zombie's apparent source IP.
3. The target responds to the Zombie.
4. Nmap checks the Zombie's IP ID again.
5. Nmap compares the change to infer the port state.

### Understanding IP ID Difference

In the classic Idle Scan model, a predictable Zombie IP ID counter may change as follows:

| IP ID Difference | Possible Interpretation |
|---|---|
| 2 | Target port is open |
| 1 | Target port is closed or filtered |

This simplified interpretation assumes the expected packet exchange and a suitable Zombie. Other traffic, network behavior and operating-system implementation can affect the result.

### When to use it?

Use an Idle Scan in an authorized environment when you need to demonstrate scanning through a suitable idle intermediary.

**Remember:** Idle Scan = Zombie host + IP ID analysis.

---

# 15. Additional Nmap Options

These options help customize scans and understand Nmap results.

## 15.1 Source Port (`--source-port`)

This option specifies the source port number used by Nmap.

### Command

```bash
sudo nmap --source-port 53 10.49.187.181
```

Here, Nmap uses source port `53`.

Why might this matter?

Some incorrectly configured firewall rules trust traffic based on its source port.

A source port can be selected for authorized firewall testing, but setting it does not automatically bypass a firewall.

---

## 15.2 Append Random Data (`--data-length`)

This option appends random data to the packets sent by Nmap.

### Command

```bash
sudo nmap --data-length 50 10.49.187.181
```

Here:

- `--data-length` = Option to append data
- `50` = Number of additional bytes

It can be useful when testing how network devices handle packets with different payload lengths.

---

## 15.3 Show Scan Reason (`--reason`)

This option explains why Nmap classified a host or port with a particular state.

### Command

```bash
sudo nmap -sS -F --reason 10.49.187.181
```

Example output:

```text
PORT    STATE   SERVICE   REASON
22/tcp  open    ssh       syn-ack ttl 64
80/tcp  closed  http      reset ttl 64
```

Explanation:

| Reason | Meaning |
|---|---|
| `syn-ack` | Target replied with SYN-ACK |
| `reset` | Target replied with RST |
| `no-response` | No response received |
| `arp-response` | Host responded to ARP discovery |

The exact reason depends on the scan type and observed response.

**Remember:** `--reason` = Why did Nmap reach this conclusion?

---

## 15.4 Fast Scan (`-F`)

The `-F` option scans fewer ports than Nmap's default scan.

### Command

```bash
sudo nmap -F 10.49.187.181
```

It scans the 100 most common ports by default.

Useful when you want a quicker initial scan.

---

## 15.5 Verbose Mode (`-v`)

Verbose mode displays more information about the scan while it is running.

### Command

```bash
sudo nmap -v 10.49.187.181
```

## 15.6 Very Verbose Mode (`-vv`)

Displays even more progress and scan information.

### Command

```bash
sudo nmap -vv 10.49.187.181
```

## 15.7 Debugging Mode (`-d`)

Displays additional technical details about Nmap's internal scanning process.

### Command

```bash
sudo nmap -d 10.49.187.181
```

## 15.8 More Detailed Debugging (`-dd`)

Provides a higher level of debugging information.

### Command

```bash
sudo nmap -dd 10.49.187.181
```

### Quick Comparison

| Option | Purpose |
|---|---|
| `-v` | Verbose output |
| `-vv` | Very verbose output |
| `-d` | Debugging information |
| `-dd` | More detailed debugging |
| `--reason` | Explains port/host state |
| `-F` | Scans 100 common ports |

---

# 16. Complete Nmap Command Cheat Sheet

This table is useful for quick revision.

| Technique | Command / Option | Purpose |
|---|---|---|
| Null Scan | `-sN` | Sends TCP packet with no flags |
| FIN Scan | `-sF` | Sends FIN flag |
| Xmas Scan | `-sX` | Sends FIN, PSH and URG |
| Maimon Scan | `-sM` | Sends FIN and ACK |
| ACK Scan | `-sA` | Maps firewall filtering |
| Window Scan | `-sW` | Examines TCP Window field |
| Custom Scan | `--scanflags` | Sets custom TCP flags |
| Source IP Spoofing | `-S IP` | Sets a forged source IP |
| MAC Spoofing | `--spoof-mac` | Sets a different MAC address |
| Decoy Scan | `-D` | Adds decoy source addresses |
| Idle Scan | `-sI` | Uses a Zombie host |
| Fragmentation | `-f` | Uses 8-byte fragmentation units |
| Fragmentation | `-ff` | Uses 16-byte fragmentation units |
| Custom MTU | `--mtu` | Sets fragmentation size |
| Source Port | `--source-port` | Specifies source port |
| Additional Data | `--data-length` | Appends random data |
| Scan Reason | `--reason` | Explains Nmap's conclusion |
| Fast Scan | `-F` | Scans 100 common ports |
| Verbose | `-v` | More scan information |
| Very Verbose | `-vv` | Even more information |
| Debug | `-d` | Debugging information |
| More Debug | `-dd` | More detailed debugging |

---

# 17. Which Scan Should You Use and When?

| Situation | Recommended Technique |
|---|---|
| Suspect a stateless firewall that blocks SYN packets | Null, FIN or Xmas |
| Suspect an older BSD-derived system | Maimon |
| Want to understand which ports a firewall allows | ACK Scan |
| Want to examine TCP Window behavior | Window Scan |
| Want to test a custom TCP flag combination | Custom Scan |
| Need to test source IP handling | IP Spoofing |
| Need to test MAC handling on a local network | MAC Spoofing |
| Want to demonstrate multiple apparent scan sources | Decoy Scan |
| Want to test packet fragmentation handling | `-f`, `-ff` |
| Have a suitable idle Zombie host | Idle Scan |
| Want to understand an unexpected scan result | `--reason` |

---

# 18. Important Concepts to Remember

### 1. Open vs Closed vs Filtered vs Unfiltered

| State | Meaning |
|---|---|
| Open | A service is listening on the port |
| Closed | The port is reachable but no service is listening |
| Filtered | Nmap cannot determine the port state because filtering prevents a clear response |
| Unfiltered | Nmap can reach the port, but cannot determine whether it is open or closed |

**Important:** An `unfiltered` port is not necessarily an open port.

### 2. Why do Null, FIN and Xmas scans sometimes work?

These scans send unusual TCP flag combinations instead of a normal SYN packet.

Some stateless firewall rules are configured to block SYN packets but may not block every other TCP packet type.

However, modern firewalls and operating systems may detect or block these scans.

### 3. What is the difference between ACK and SYN scans?

| SYN Scan | ACK Scan |
|---|---|
| Uses SYN flag | Uses ACK flag |
| Used to identify open ports | Used to map filtering behavior |
| SYN-ACK usually means open | RST means unfiltered |
| RST usually means closed | Does not directly identify open services |

### 4. IP Address vs MAC Address

| IP Address | MAC Address |
|---|---|
| Layer 3 | Layer 2 |
| Used for network routing | Used for local network delivery |
| Can travel across routers | Replaced at each routed network hop |

### 5. Firewall vs IDS vs IPS

| Technology | Main Purpose |
|---|---|
| Firewall | Allows or blocks network traffic |
| IDS | Detects suspicious activity and generates alerts |
| IPS | Detects and can actively block suspicious activity |

---

# 19. Practical Revision Commands

Use these commands only against your own systems or authorized lab targets.

Set the lab target:

```bash
TARGET=10.49.187.181
```

Basic advanced scan examples:

```bash
sudo nmap -sN $TARGET
sudo nmap -sF $TARGET
sudo nmap -sX $TARGET
sudo nmap -sM $TARGET
sudo nmap -sA $TARGET
sudo nmap -sW $TARGET
```

Custom TCP flags:

```bash
sudo nmap --scanflags RST $TARGET
sudo nmap --scanflags SYNFIN $TARGET
```

Firewall mapping:

```bash
sudo nmap -sA $TARGET
sudo nmap -sW $TARGET
```

Spoofing and decoy examples:

```bash
sudo nmap -S 10.10.10.11 $TARGET
sudo nmap --spoof-mac 0 $TARGET
sudo nmap -D RND,RND,ME $TARGET
```

Fragmentation:

```bash
sudo nmap -sS -f $TARGET
sudo nmap -sS -ff $TARGET
sudo nmap -sS --mtu 24 $TARGET
```

Idle Scan:

```bash
sudo nmap -sI ZOMBIE_IP $TARGET
```

Additional options:

```bash
sudo nmap --source-port 53 $TARGET
sudo nmap --data-length 50 $TARGET
sudo nmap -sS -F --reason $TARGET
sudo nmap -vv $TARGET
sudo nmap -dd $TARGET
```

---

# 20. Final Revision – One-Line Memory Tricks

- **Null Scan:** No TCP flags.
- **FIN Scan:** Only FIN flag.
- **Xmas Scan:** FIN + PSH + URG.
- **Maimon Scan:** FIN + ACK.
- **ACK Scan:** Firewall filtering analysis.
- **Window Scan:** ACK + Window field analysis.
- **Custom Scan:** Choose your own TCP flags.
- **IP Spoofing:** Change apparent source IP.
- **MAC Spoofing:** Change apparent MAC address.
- **Decoy Scan:** Multiple apparent source addresses.
- **Fragmentation:** Divide IP packets into smaller fragments.
- **Idle Scan:** Use a Zombie and analyze IP ID changes.
- **Source Port:** Choose the source port.
- **Data Length:** Add extra packet data.
- **Reason:** Understand why Nmap classified a port.
- **Verbose:** Get more information.
- **Debug:** Get technical details about Nmap's process.

---

## Conclusion

Advanced Nmap scanning is not just about discovering open ports. It also helps us understand how operating systems, TCP/IP protocols, firewalls and network monitoring systems behave.

A good penetration tester should understand:

1. How different TCP flags work.
2. How different scan types generate responses.
3. How Nmap interprets those responses.
4. How firewalls affect scan results.
5. How to troubleshoot unexpected results using `--reason`, verbose and debugging options.

**Important:** These techniques should only be used in authorized penetration testing environments, such as TryHackMe labs, your own network or systems where you have explicit permission to test.

**Next Room:** Nmap Post Port Scans – Continue the Nmap learning series.
