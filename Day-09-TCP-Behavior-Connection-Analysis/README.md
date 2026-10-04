# 🚀 Day 09 – TCP Behavior & Connection Analysis

## 🎯 Objective

Understand how a SOC analyst can use TCP flags and packet behavior to determine what is happening to a TCP connection.

Focus areas:

- SYN
- ACK
- FIN
- RST
- TCP connection patterns
- Wireshark TCP flag analysis
- Evidence-based investigation

---

## 🧠 TCP Flags

| Flag | Meaning | SOC Relevance |
|---|---|---|
| SYN | Start a TCP connection | Connection attempt |
| ACK | Acknowledge TCP information | Confirms received information |
| FIN | Graceful termination | Connection closing |
| RST | Immediate reset | Connection reset/aborted |

### Key Principle

> A TCP flag is evidence, not a verdict.

A single RST or SYN does not automatically indicate malicious activity.

---

# 🔗 Connection With Previous Days

```text
Day 3 → Wireshark Filters
        ↓
Day 4 → IP Analysis
        ↓
Day 6 → DHCP / IP Assignment
        ↓
Day 7 → ARP / MAC Resolution
        ↓
Day 8 → TCP 3-Way Handshake
        ↓
Day 9 → TCP Behavior & Connection Analysis
```

### Investigation Flow

```text
IP
 ↓
Port
 ↓
TCP Flags
 ↓
Packet Sequence
 ↓
Traffic Pattern
 ↓
Context
 ↓
SOC Conclusion
```

---

# 🔥 FIN vs RST

## FIN

FIN is used for **graceful TCP connection termination**.

Example:

```text
FIN
 ↓
ACK
 ↓
FIN
 ↓
ACK
```

It generally means the endpoint has finished sending data and the connection is being closed normally.

## RST

RST immediately resets/aborts a TCP connection.

Example:

```text
SYN
 ↓
RST
```

This indicates that the connection attempt was reset.

Possible reasons include:

- Port not listening
- Firewall behavior
- Application rejection
- Server/device issue
- Network issue
- Security control blocking traffic

### Important SOC Principle

> RST does not automatically mean an attack.

---

# 🧪 Practical Analysis

## Task 1 — TCP RST Analysis

### Wireshark Filter

```text
tcp.flags.reset == 1
```

### Observed Packet

```text
Frame Number   : 561
Source IP      : 2409:408d:79c:119d:a05a:4aa7:c812:1b6d
Destination IP : 2600:140f:400::b854:e949
Source Port    : 62129
Destination Port: 443
TCP Flags      : 0x014 (RST, ACK)
```

### Observation

The packet contains the **RST + ACK** flags, indicating that the TCP connection was reset while acknowledging TCP information.

The RST alone is not enough to determine malicious activity.

---

## Task 2 — TCP FIN Analysis

### Wireshark Filter

```text
tcp.flags.fin == 1
```

### Observed Packet

```text
Frame Number   : 557
Source IP      : 2409:408d:79c:119d:a05a:4aa7:c812:1b6d
Destination IP : 64:ff9b::14b8:af0b
Source Port    : 62130
Destination Port: 443
TCP Flags      : 0x011 (FIN, ACK)
```

### Observation

The packet contains **FIN + ACK**, indicating that TCP connection termination has begun gracefully.

---

# 🔎 Task 3 — Compare SYN and RST

### Initial SYN Filter

```text
tcp.flags.syn == 1 && tcp.flags.ack == 0
```

The observed connection followed:

```text
Frame 11
[SYN]
   ↓
Frame 12
[SYN, ACK]
   ↓
Frame 13
[ACK]
```

### Result

The standard TCP 3-way handshake completed successfully.

```text
SYN
 ↓
SYN/ACK
 ↓
ACK
 ↓
Connection Established
```

This connects directly with the TCP handshake analysis from Day 8.

---

# 🕵️ Task 4 — RST Investigation

### Scenario

```text
10.65.57.237:51000
        ↓
10.65.57.50:445
        SYN

10.65.57.50:445
        ↓
10.65.57.237:51000
        RST
```

### Q1. What happened?

The TCP connection attempt was reset/rejected.

### Q2. Does the RST prove that an attack occurred?

**No.**

### Q3. Possible explanations

- Port not listening
- Firewall behavior
- Application rejected the connection
- Server/device issue
- Network issue

### Q4. Additional evidence to investigate

```text
Why was the connection reset?
        ↓
Was it repeated?
        ↓
How many destinations?
        ↓
How many ports?
        ↓
Was the traffic expected?
        ↓
What device owns the source IP?
        ↓
What happened before and after?
```

Relevant supporting evidence can include:

- IP addresses
- Ports
- TCP flags
- Packet timing
- DHCP information
- ARP information
- Related network traffic

---

# 🚨 Task 5 — SOC Investigation Scenario

### Alert

```text
Source IP      : 10.65.57.237
Destination IP : 10.65.57.50
Destination    : TCP/445
```

Observed:

```text
10.65.57.237 → 10.65.57.50:445
[SYN]

10.65.57.50 → 10.65.57.237
[RST]
```

This behavior occurs **150 times against 150 different internal IP addresses**.

### Q1. What immediately stands out?

One internal host is attempting TCP/445 connections to **150 different internal IP addresses**, with the connections being reset.

### Q2. Is this automatically malicious?

**No.**

The behavior is suspicious and requires investigation, but it could also be legitimate security or vulnerability scanning.

### Q3. What could this behavior represent?

Possible explanations include:

- Network/service discovery
- Port scanning
- Authorized vulnerability scanning
- Malware or worm propagation

### Q4. What evidence should be collected next?

```text
Source asset identity
        ↓
Device owner
        ↓
Running process
        ↓
Security/IT tools
        ↓
Authorization for scanning
        ↓
Endpoint logs
        ↓
Authentication activity
        ↓
Previous similar activity
```

### Q5. Escalation reasoning

The activity should be investigated and handled according to the SOC's incident-response procedure.

A Tier-1 analyst should first validate whether the source is an authorized scanner or legitimate administrative/security infrastructure.

If the activity is unauthorized or additional evidence indicates compromise, it should be escalated according to the defined SOC procedure.

> Do not automatically block an endpoint solely because of the TCP/445 pattern without appropriate authorization and evidence.

---

# 🎤 Interview Check

### Q1. Difference between FIN and RST?

**FIN** is used for graceful TCP termination.

**RST** immediately resets/aborts the connection.

### Q2. Does RST automatically mean malicious activity?

**No.**

### Q3. What does this suggest?

```text
SYN
 ↓
SYN/ACK
 ↓
ACK
 ↓
FIN
```

A TCP connection was established and connection termination began gracefully.

### Q4. What does this suggest?

```text
SYN
 ↓
RST
```

The connection attempt was reset.

### Q5. Why is this more suspicious?

```text
1 SYN → 1 destination
```

is a single connection attempt.

Whereas:

```text
150 SYNs
 ↓
150 different internal IPs
 ↓
TCP/445
```

shows a broader connection pattern and is consistent with network/service discovery or scanning.

However, legitimate vulnerability scanning can produce similar behavior, so additional evidence is required.

---

# 🛡️ SOC Mindset

```text
Single Packet
     ↓
Observation

Repeated Packets
     ↓
Behavior

Many Destinations
     ↓
Pattern

Asset + Timing + Process + Logs
     ↓
Context

Evidence + Context
     ↓
Investigation Conclusion
```

### Key Principle

> **A packet is evidence, not a verdict.**

---

# 🔗 What I Learned

By connecting Day 8 and Day 9:

```text
SYN
 ↓
SYN/ACK
 ↓
ACK
 ↓
Connection Established
 ↓
FIN / RST
 ↓
Connection Behavior
```

I learned that TCP flags can help identify **how a connection starts, behaves, and terminates**.

I also learned that SOC analysis should focus on **patterns and context rather than interpreting a single packet in isolation**.

---

# 📸 Evidence

👉 [View TCP RST Analysis Screenshot](./screenshots/01-tcp-rst-analysis.png)

👉 [View TCP FIN Analysis Screenshot](./screenshots/02-tcp-fin-analysis.png)

---

# 🧠 Key Takeaways

```text
SYN  → Start connection
ACK  → Acknowledge
FIN  → Graceful termination
RST  → Immediate reset
```

Important SOC rules:

- RST does not automatically mean malicious activity.
- FIN generally indicates graceful termination.
- TCP behavior should be analyzed as a sequence.
- Multiple destinations and repeated connections can reveal patterns.
- Suspicious patterns still require context.
- Asset identity, process information, timing and logs can help validate the activity.
- Always use evidence before reaching a SOC conclusion.

---

# 📊 Day 9 Status

**Status:** ✅ Completed

### Skills Practiced

- TCP flag analysis
- RST analysis
- FIN analysis
- TCP connection pattern analysis
- Wireshark TCP filtering
- Network scanning pattern recognition
- Evidence-based SOC investigation
- Tier-1 SOC reasoning
