# Day 10 — DNS Investigation & Suspicious DNS Behavior

## 🎯 Objective

Understand how DNS works and learn how a SOC Analyst can investigate DNS traffic for suspicious activity.

---

## 🌐 What is DNS?

**DNS (Domain Name System)** translates human-readable domain names into IP addresses.

Example:

```text
wac-ring.msedge.net
        ↓
DNS
        ↓
2603:1063:2000:1::254
```

This allows systems to communicate with the correct destination.

---

## 🔑 Important DNS Concepts

| Record / Concept | Meaning |
|---|---|
| A | Resolves a domain to an IPv4 address |
| AAAA | Resolves a domain to an IPv6 address |
| DNS Query | Client requests information from DNS server |
| DNS Response | DNS server provides the answer |
| NXDOMAIN | Requested domain does not exist according to the DNS response |
| Port 53 | Common DNS port |

### Useful Wireshark Filters

```text
dns
```

Display DNS traffic.

```text
dns.flags.response == 0
```

Display DNS queries.

```text
dns.flags.response == 1
```

Display DNS responses.

```text
dns.flags.rcode != 0
```

Find DNS responses with a non-zero response code.

---

# 🧪 Practical Investigation

## Task 1 — DNS Query Analysis

Used the following filter:

```text
dns.flags.response == 0
```

Observed DNS queries from:

```text
Source      : 10.240.36.237
DNS Server  : 10.240.36.109
Port        : 53
Domain      : wac-ring.msedge.net
```

Example query:

```text
Frame 16
10.240.36.237 → 10.240.36.109
UDP 52121 → 53
Query Type: HTTPS
Domain: wac-ring.msedge.net
```
## 🔬 DNS Query Types Observed

For `wac-ring.msedge.net`, multiple DNS query types were observed:

| Frame | Query Type | Purpose |
|---|---|---|
| 16 | HTTPS | HTTPS DNS record query |
| 17 | AAAA | IPv6 address lookup |
| 18 | A | IPv4 address lookup |

The AAAA response in Frame 19 returned:

`2603:1063:2000:1::254`

### 📸 Evidence

👉 [View DNS Query Analysis Screenshot](./screenshots/01-dns-query-analysis.png)

---

# 🔎 Task 2 — DNS Response Analysis

Observed the corresponding DNS response:

```text
Frame 19
10.240.36.109 → 10.240.36.237
UDP 53 → 57944
```

Query:

```text
wac-ring.msedge.net
```

Query Type:

```text
AAAA
```

Response:

```text
No error
```

Resolved IPv6 address:

```text
2603:1063:2000:1::254
```

### 📸 Evidence

👉 [View DNS Response Analysis Screenshot](./screenshots/02-dns-response-analysis.png)

---

# 🔗 Task 3 — DNS to TCP Connection Analysis

After DNS resolution, the resolved IPv6 address was contacted over TCP port 443.

Observed:

```text
Client:
2409:408d:1d92:2bfb:25fd:6874:ac11:4087

Server:
2603:1063:2000:1::254
```

### TCP 3-Way Handshake

```text
Frame 22
50293 → 443 [SYN]

        ↓

Frame 23
443 → 50293 [SYN, ACK]

        ↓

Frame 24
50293 → 443 [ACK]
```

The TCP handshake completed successfully.

A TLS Client Hello was also observed with:

```text
SNI: wac-ring.msedge.net
```

### 🔗 Correlation

The investigation connected the DNS activity with subsequent network traffic:

```text
wac-ring.msedge.net
        ↓
DNS Response
        ↓
2603:1063:2000:1::254
        ↓
TCP/443
        ↓
SYN → SYN/ACK → ACK
        ↓
TLS Client Hello
        ↓
SNI: wac-ring.msedge.net
```

### 📸 Evidence

👉 [View DNS-to-TCP Analysis Screenshot](./screenshots/03-dns-to-tcp-analysis.png)

---

# 🚨 SOC Investigation Scenario

### Alert

```text
ALERT: Suspicious DNS Activity

Source Host:
10.65.57.237

DNS Query:
xj29akd91.example-domain.com

DNS Response:
185.220.101.45

Destination Port:
443
```

### Investigation Approach

I would investigate the activity step by step instead of immediately declaring it malicious.

```text
DNS Query
    ↓
Identify Domain
    ↓
Check DNS Response
    ↓
Identify Resolved IP
    ↓
Check TCP Connection
    ↓
Check Connection Timing
    ↓
Check IP / Domain Reputation
    ↓
Identify Endpoint Process
    ↓
Determine Whether Activity Is Legitimate or Malicious
```

### Important Evidence to Collect

- Domain reputation
- IP reputation
- DNS history
- Connection timing and frequency
- TCP handshake
- Endpoint process
- Command line activity
- Related network traffic
- DHCP / ARP information for device context
- Relevant endpoint or security logs

### 🧠 Important SOC Rule

```text
Suspicious ≠ Malicious ≠ Confirmed Compromise
```

A random-looking or unusual domain is an **indicator requiring investigation**, not automatic proof of malware.

---

# 🔐 SOC Relevance

DNS analysis helps a SOC Analyst identify:

- Suspicious domain lookups
- Failed or unusual DNS requests
- Possible command-and-control activity
- DNS tunneling indicators
- Repeated automated connections
- Domain-to-IP relationships
- Network communication following DNS resolution

A useful investigation chain is:

```text
Domain
  ↓
DNS Query
  ↓
DNS Response
  ↓
Resolved IP
  ↓
TCP Connection
  ↓
TLS / Application Traffic
  ↓
Endpoint Investigation
```

---

# 🎯 Interview Check

### 1. What is DNS?

DNS translates human-readable domain names into IP addresses so systems can communicate.

### 2. What is the difference between A and AAAA?

```text
A     → IPv4
AAAA  → IPv6
```

### 3. What does this filter do?

```text
dns.flags.response == 0
```

It displays DNS queries.

### 4. What is NXDOMAIN?

NXDOMAIN indicates that the requested domain does not exist according to the DNS response.

### 5. Does a suspicious domain automatically mean the system is compromised?

No.

It is an indicator that requires further investigation using domain/IP reputation, DNS history, connection timing, endpoint activity, processes, and related network traffic.

---

# 🔗 Connect Everything You've Learned

```text
Day 4 → Private/Public IP
       ↓
Day 5 → Subnet & Network Analysis
       ↓
Day 6 → DHCP gives IP + Gateway + DNS
       ↓
Day 7 → ARP finds MAC address
       ↓
Day 8 → TCP 3-Way Handshake
       ↓
Day 9 → TCP Connection Behavior
       ↓
Day 10 → DNS Investigation
```

### 🧠 SOC Investigation Flow

```text
DNS
 ↓
Identify Domain
 ↓
Resolve IP
 ↓
Check Connection
 ↓
Analyze TCP/TLS
 ↓
Investigate Endpoint
 ↓
Determine Risk
```

---

# 📌 Key Takeaways

- DNS translates domain names into IP addresses.
- `A` records provide IPv4 addresses.
- `AAAA` records provide IPv6 addresses.
- DNS queries and responses can be analyzed in Wireshark.
- DNS resolution can be followed by TCP/443 communication.
- A successful TCP handshake does not automatically mean the traffic is malicious.
- Suspicious DNS activity requires additional evidence.
- SOC Analysts should avoid making conclusions from a single indicator.

---

## 📸 Evidence

### DNS Query

👉 [View DNS Query Analysis](./screenshots/01-dns-query-analysis.png)

### DNS Response

👉 [View DNS Response Analysis](./screenshots/02-dns-response-analysis.png)

### DNS → TCP Connection

👉 [View DNS-to-TCP Analysis](./screenshots/03-dns-to-tcp-analysis.png)

---

## 🛠️ Tools Used

- Wireshark
- DNS
- TCP/IP
- IPv6
- Network Traffic Analysis

---

## 🎓 Skills Practiced

```text
DNS Traffic Analysis
DNS Query/Response Analysis
A & AAAA Record Analysis
IPv6 Analysis
TCP 3-Way Handshake Analysis
DNS-to-TCP Correlation
Suspicious DNS Investigation
SOC Evidence-Based Reasoning
