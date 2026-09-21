# 🔵 Day 04 — Subnetting, CIDR & Private/Public IPs

## 🎯 Objective

Understand IPv4 addressing, private/public IP ranges, basic CIDR notation, and how a SOC Analyst can classify internal and external network communication.

The main goal was to answer:

> Is this IP internal/private or external/public?

I also connected IP classification with my existing Wireshark skills from Day 3.

---

## 📚 Concepts Learned

- IPv4 addresses
- Private IPv4 addresses
- Public IPv4 addresses
- RFC 1918
- CIDR notation
- `/24` network
- Internal vs external communication
- Wireshark IP classification
- TCP/443 analysis
- TCP SYN analysis
- Evidence-based SOC investigation

---

# 🔐 Private IPv4 Address Ranges

The three RFC 1918 private IPv4 ranges are:

| Private Range | CIDR | Example |
|---|---|---|
| 10.0.0.0 – 10.255.255.255 | `10.0.0.0/8` | `10.20.5.10` |
| 172.16.0.0 – 172.31.255.255 | `172.16.0.0/12` | `172.20.10.5` |
| 192.168.0.0 – 192.168.255.255 | `192.168.0.0/16` | `192.168.1.20` |

### Important Note

Not every `172.x.x.x` address is private.

Only:

```text
172.16.0.0 → 172.31.255.255
````

is part of the RFC 1918 private range.

For example:

```text
172.20.5.10 → Private
172.31.5.10 → Private
172.40.5.10 → Not RFC 1918 private
```

---

# 📘 Basic CIDR Understanding

Example:

```text
192.168.1.0/24
```

The `/24` means that the first 24 bits represent the network portion.

For a basic `/24` network:

```text
Network Address: 192.168.1.0
Usable Host Range: 192.168.1.1 – 192.168.1.254
Broadcast Address: 192.168.1.255
```

---

# 🛠️ Practical Wireshark Analysis

## Task 1 — Private IP + DNS Analysis

### Wireshark Filter

```text
dns
```

### Observed Packet

```text
Source IP:        10.240.36.237
Destination IP:   10.240.36.109
Protocol:         UDP
Source Port:      62292
Destination Port: 53
```

The packet was identified as a:

```text
DNS Query
```

### IP Classification

```text
10.240.36.237 → Private
10.240.36.109 → Private
```

Both addresses belong to:

```text
10.0.0.0/8
```

which is an RFC 1918 private IPv4 range.

### Traffic Interpretation

The packet shows:

```text
Internal Host
10.240.36.237:62292
        ↓
      UDP/53
        ↓
DNS Server
10.240.36.109:53
```

The use of UDP destination port `53` is consistent with DNS query traffic.

### SOC Lesson

The IP address and port should be interpreted together with:

* Protocol
* Direction
* Packet type
* Network context

A single IP address should not be assigned a role based only on an assumption or a separate lookup.

### Evidence

📸 [Task 1 — Private IP & DNS Analysis](./screenshots/01-private-ip-dns-analysis.png)

---

# 🌐 Task 2 — External HTTPS Traffic

### Wireshark Filter

```text
tcp.port == 443
```

### Observed Packet

```text
Source IP:        10.240.36.237
Destination IP:   40.126.17.131
Source Port:      63581
Destination Port: 443
Protocol:         TCP
```

### IP Classification

```text
10.240.36.237 → Private/Internal
40.126.17.131 → Public/External
```

Therefore, the traffic represents:

```text
Internal Machine
10.240.36.237
      ↓
TCP 63581 → 443
      ↓
External/Public IP
40.126.17.131
```

### Interpretation

TCP port `443` is commonly associated with HTTPS/TLS traffic.

The source port `63581` is an ephemeral client port, while destination port `443` is commonly used as the server-side HTTPS port.

### SOC Perspective

An internal machine communicating with a public IP over TCP/443 is not automatically malicious.

It could represent:

* Normal web activity
* A legitimate application
* An update service
* Other authorized software communication

Therefore, additional evidence is required before making a security conclusion.

### Evidence

📸 [Task 2 — External HTTPS Traffic](./screenshots/02-external-https-traffic.png)

---

# 🚨 Task 3 — TCP SYN + Public IP Analysis

### Wireshark Filter

```text
tcp.flags.syn == 1 && tcp.flags.ack == 0
```

### Observed Packet

```text
Source IP:        10.240.36.237
Destination IP:   40.126.17.131
Source Port:      63581
Destination Port: 443
Protocol:         TCP
TCP Flag:         SYN
```

### IP Classification

```text
10.240.36.237 → Private
40.126.17.131 → Public
```

### TCP Analysis

The packet contains:

```text
SYN
```

with:

```text
Source Port:      63581
Destination Port: 443
```

A SYN packet with no ACK set represents an initial TCP connection attempt.

### Conclusion

> The packet represents an initial TCP connection attempt from a private/internal host to a public/external IP on TCP port 443.

TCP/443 is commonly associated with HTTPS/TLS, but the port alone does not prove that the activity is safe or malicious.

### Evidence

📸 [Task 3 — TCP SYN & Public IP Analysis](./screenshots/03-syn-public-ip-analysis.png)

---

# 🚨 SOC Investigation Scenario

## Alert: Suspicious Outbound Connection

```text
Source IP:        10.10.15.25
Destination IP:   185.220.101.45
Source Port:      52144
Destination Port: 443
Protocol:         TCP
Time:             02:17 AM
```

### Initial Classification

```text
10.10.15.25
↓
Private/Internal

185.220.101.45
↓
Public/External

TCP/443
↓
Commonly associated with HTTPS/TLS
```

### Does this automatically mean malware?

No.

The IP addresses, port, and protocol provide useful context, but they are not enough to determine whether the activity is malicious.

---

# 🔎 Additional Evidence to Investigate

Before making a conclusion, I would investigate:

### 1. Destination IP Reputation

Check:

* VirusTotal
* AbuseIPDB
* Other trusted threat intelligence sources

### 2. DNS Activity

Investigate:

* Which domain resolved to the destination IP?
* Was the domain expected?
* Is the domain suspicious?

### 3. Process Information

Identify which process created the connection:

* Browser
* PowerShell
* CMD
* Known application
* Unknown executable

### 4. Connection Frequency

Check whether the connection was:

* A single connection
* Repeated connections
* Periodic connections
* A possible beaconing pattern

### 5. Endpoint Activity

Look for:

* Process creation
* PowerShell activity
* CMD activity
* New files
* Other suspicious behavior

### 6. User and Time Context

The alert occurred at:

```text
02:17 AM
```

I would check:

* Was the user active?
* Was there a scheduled task?
* Was an update running?
* Was there an automated backup or service?

### 7. Other Network Connections

Check whether the same endpoint communicated with:

* Other external IPs
* Other suspicious domains
* Multiple unusual destinations

---

# 🧠 Alert-Specific Investigation

Different alerts require different evidence.

### Example — Brute Force Alert

```text
Authentication Logs
        ↓
Failed Logins
        ↓
Successful Login
        ↓
Username
        ↓
Logon Type
```

### Example — Suspicious Outbound Connection

```text
DNS
 ↓
Destination Reputation
 ↓
Process
 ↓
Connection Frequency
 ↓
Endpoint Activity
 ↓
User / Time Context
```

### SOC Principle

> Investigation evidence should match the type of alert being investigated.

---

# 🧪 Wireshark Filters Practiced

| Purpose                 | Filter                                     |
| ----------------------- | ------------------------------------------ |
| DNS traffic             | `dns`                                      |
| Traffic involving an IP | `ip.addr == x.x.x.x`                       |
| Source IP               | `ip.src == x.x.x.x`                        |
| Destination IP          | `ip.dst == x.x.x.x`                        |
| TCP/443 traffic         | `tcp.port == 443`                          |
| Initial TCP SYN         | `tcp.flags.syn == 1 && tcp.flags.ack == 0` |

---

# 🎯 Interview Check

## Q1. What are the three RFC 1918 private IPv4 ranges?

```text
10.0.0.0/8
172.16.0.0/12
192.168.0.0/16
```

## Q2. Is 172.40.5.10 private?

No.

The private `172.x.x.x` range is:

```text
172.16.0.0 → 172.31.255.255
```

Therefore:

```text
172.40.5.10
```

is not an RFC 1918 private address.

## Q3. What does this mean?

```text
10.0.0.15 → 8.8.8.8:443
```

`10.0.0.15` is a private/internal IP.

`8.8.8.8` is a public IP.

TCP port `443` is commonly associated with HTTPS/TLS.

However, the IP addresses and port alone cannot prove whether the activity is safe or malicious.

Additional evidence such as DNS, process information, destination reputation, timing, connection frequency, and endpoint logs should be investigated.

---

# 🔗 SOC Investigation Workflow

```text
New Alert
    ↓
Identify Source
    ↓
Private or Public?
    ↓
Identify Destination
    ↓
Private or Public?
    ↓
Protocol + Port
    ↓
Traffic Direction
    ↓
Collect Additional Evidence
    ↓
Form Hypothesis
    ↓
Validate with Logs / Network Data
    ↓
Determine Alert Disposition
```

---

# 💡 Key Takeaways

* `10.0.0.0/8` is a private IPv4 range.
* `172.16.0.0/12` is a private IPv4 range.
* `192.168.0.0/16` is a private IPv4 range.
* Not every `172.x.x.x` address is private.
* Private IP does not automatically mean safe.
* Public IP does not automatically mean malicious.
* TCP/443 does not automatically mean safe.
* One packet does not prove compromise.
* IP + Port + Protocol alone are not enough.
* Traffic direction provides useful investigation context.
* Wireshark can help validate network observations.
* Investigation evidence should match the alert type.
* SOC analysts should avoid assumptions and validate findings using multiple sources.

---

# 📝 Reflection

Today I learned how to classify IPv4 addresses as private or public and connect this knowledge with Wireshark network analysis.

I also learned that an IP address should not be identified or assigned a role based only on assumptions. Packet-level evidence such as IP addresses, ports, protocols, direction, and packet type should be considered together.

The main SOC lesson I learned today was:

> **Observe → Collect Evidence → Validate → Form a Hypothesis**

---

# 📊 Day 04 Status

| Topic                        | Status      |
| ---------------------------- | ----------- |
| IPv4 Basics                  | ✅ Completed |
| RFC 1918 Private IP Ranges   | ✅ Completed |
| Public vs Private IPs        | ✅ Completed |
| Basic CIDR `/24`             | ✅ Completed |
| Internal vs External Traffic | ✅ Completed |
| Wireshark IP Classification  | ✅ Completed |
| TCP/443 Investigation        | ✅ Completed |
| TCP SYN Analysis             | ✅ Completed |
| SOC Investigation Scenario   | ✅ Completed |
| Evidence-Based Reasoning     | ✅ Completed |

## 🏆 Day 04 — COMPLETED

### Main Skill Developed

> **Evidence-based network investigation**

```text
IP
 ↓
Private / Public
 ↓
Port
 ↓
Protocol
 ↓
Direction
 ↓
Context
 ↓
Additional Evidence
 ↓
Conclusion
```
