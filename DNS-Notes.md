# 🌐 DNS (Domain Name System) Notes

**What is DNS?**
DNS is the "phonebook of the internet." It translates human-readable domain names (like google.com) into machine-readable IP addresses (like 142.250.190.46).

## 📊 How DNS Works (The Process)
```text
[Your Computer] 
      │
      ▼
1. [DNS Resolver] (Usually provided by your ISP)
      │
      ▼
2. [Root Server] (Directs to the TLD)
      │
      ▼
3. [TLD Server] (.com, .org, .in) (Directs to Authoritative)
      │
      ▼
4. [Authoritative Nameserver] (Holds the actual IP)
      │
      ▼
[Returns IP Address to your computer]


🏗️ Domain Hierarchy
TLD (Top Level Domain): The last part of the domain (e.g., .com, .org, .in).

SLD (Second Level Domain): The name right before the TLD (e.g., tryhackme in tryhackme.com).

Subdomain: A prefix created by the owner (e.g., admin.tryhackme.com, blog.tryhackme.com).

📜 Common DNS Record Types
A Record: Maps a domain to an IPv4 address.

AAAA Record: Maps a domain to an IPv6 address.

CNAME Record: Maps a domain to another domain (Alias).

MX Record: Directs email to the correct mail server.

TXT Record: Stores text data (often used for security like SPF/DKIM).

🛠️ Useful Tools & Commands
nslookup <domain> : Queries DNS to get IP address.

dig <domain> : Detailed DNS query (shows all records).

dig <domain> MX : Specifically looks for mail server records.
