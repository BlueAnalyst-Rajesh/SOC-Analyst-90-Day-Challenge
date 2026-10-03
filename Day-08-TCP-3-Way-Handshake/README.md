# 🚀 Day 08 – TCP 3-Way Handshake & Connection Analysis

## 🎯 Objective

Understand how a TCP connection is established and use Wireshark to determine:

- Who initiated the connection
- Whether the TCP handshake completed
- How client/server direction works
- How TCP flags help in SOC investigations
- Why a SYN packet alone does not prove a successful connection

---

## 📚 Core Concepts

### What is TCP?

**TCP (Transmission Control Protocol)** provides connection-oriented communication.

Before application data is normally exchanged, TCP establishes a connection using the **3-way handshake**.

### 🔄 TCP 3-Way Handshake

```text
Client                         Server
  |                              |
  | -------- SYN -------------> |
  |                              |
  | <------ SYN + ACK ---------- |
  |                              |
  | -------- ACK -------------> |
  |                              |
  |      Connection Established |
```

### Remember

```text
SYN
 ↓
SYN + ACK
 ↓
ACK
```

- **SYN** → Connection attempt is initiated
- **SYN + ACK** → Server acknowledges the SYN and responds with its own SYN
- **ACK** → Acknowledges the SYN/ACK and completes the handshake

---

## 🎯 Why Is This Important in SOC?

A SYN packet only shows that a TCP connection was **attempted**.

It does **not** prove that the connection was successfully established.

For example:

```text
SYN
 ↓
SYN + ACK
 ↓
ACK
```

indicates a completed 3-way handshake.

But:

```text
SYN
SYN
SYN
SYN
```

with no SYN/ACK can have multiple explanations, such as:

- Destination unavailable
- Firewall filtering
- Packet loss
- Routing problem
- Destination port not listening
- Network issue
- Connection retry/retransmission

Therefore:

> **SYN ≠ Successful Connection**

A SOC analyst should investigate the complete communication sequence before reaching a conclusion.

---

# 🧪 Practical Analysis

## Task 1 & 2 – Capture and Follow a TCP Handshake

### Wireshark Filter

```text
tcp.flags.syn == 1 && tcp.flags.ack == 0
```

### Initial SYN

```text
Frame Number   : 36
Source IP      : 2409:408d:79c:119d:7850:5c08:91c2:5972
Source Port    : 51299
Destination IP : 64:ff9b::399b:68e0
Destination Port: 443
Protocol       : TCP
TCP Flags      : 0x002 (SYN)
```

The SYN packet shows that the host initiated a TCP connection attempt to port **443**.

### Handshake Sequence

```text
Frame 36
51299 → 443
[SYN]

        ↓

Frame 37
443 → 51299
[SYN, ACK]

        ↓

Frame 38
51299 → 443
[ACK]
```

### Result

The SYN/ACK was found in Frame 37 and the final ACK was found in Frame 38.

Therefore, the **TCP 3-way handshake was completed** for this observed connection.

---

## Task 3 – Identify Connection Direction

Example traffic:

```text
10.65.57.237:55000
        ↓
142.250.x.x:443
[SYN]
```

The connection direction is:

```text
Client → Server
```

### Answers

| Item | Observation |
|---|---|
| Connection Initiator | `10.65.57.237` |
| Remote Server/Endpoint | `142.250.x.x` |
| Client Source Port | `55000` |
| Server Service Port | `443` |
| Handshake | `SYN → SYN/ACK → ACK` |
| Connection | Successfully established |

The conclusion is based on the observed TCP handshake sequence.

---

# 🚨 Task 4 – SOC Investigation

### Alert

```text
Source:
10.65.57.237

Destination:
185.220.101.45

Destination Port:
443

Observed:
SYN
SYN
SYN
SYN
SYN

No SYN/ACK observed
```

### Q1. What does the SYN tell you?

The SYN indicates that the host is attempting to initiate a TCP connection.

### Q2. Does this prove that the connection was successful?

**No.**

A successful TCP handshake requires evidence of the response and subsequent ACK:

```text
SYN
 ↓
SYN/ACK
 ↓
ACK
```

### Q3. What could explain repeated SYN packets?

Possible explanations include:

- Destination unavailable
- Firewall filtering
- Packet loss
- Routing problem
- Destination port not listening
- Connection retry/retransmission

### Q4. What additional evidence should be investigated?

```text
Source IP
Destination IP
Source Port
Destination Port
Packet Timestamp
TCP Flags
SYN / SYN-ACK / ACK
Retransmissions
Number of destinations
Number of ports
```

Additional context can also include:

- Destination IP reputation
- DHCP information when host identity is needed
- ARP information for local IP-to-MAC mapping
- Other related network traffic

### Q5. Would this immediately be classified as malicious?

**No.**

Repeated SYN packets alone are not enough to conclude malicious activity.

The analyst should investigate the destination, timing, connection pattern, responses, and additional evidence before reaching a conclusion.

---

# 🔎 Wireshark Filters Used

### Initial SYN

```text
tcp.flags.syn == 1 && tcp.flags.ack == 0
```

### SYN/ACK

```text
tcp.flags.syn == 1 && tcp.flags.ack == 1
```

### ACK packets

```text
tcp.flags.ack == 1
```

> Note: ACK appears in many TCP packets, so this filter alone does not identify only the final handshake ACK.

### Investigate a Host

```text
ip.addr == 10.65.57.237
```

### TCP Traffic for a Host

```text
ip.addr == 10.65.57.237 && tcp
```

### TCP Port 443

```text
tcp.port == 443
```

---

# 🛡️ SOC Relevance

TCP handshake analysis helps a SOC analyst investigate:

- Suspicious outbound connections
- Repeated connection attempts
- Failed TCP connections
- Port scanning behavior
- Unusual services
- Potential command-and-control indicators

### Investigation Mindset

```text
SYN observed
     ↓
Who initiated?
     ↓
Where?
     ↓
Which port?
     ↓
SYN/ACK received?
     ↓
Final ACK?
     ↓
Handshake completed?
     ↓
What happened afterward?
     ↓
Is there enough evidence
to call it suspicious?
```

The key principle is:

> **Unusual network behavior requires investigation; it is not automatically malicious.**

---

# 🔗 Connect Everything You've Learned

```text
Day 2 → Protocols & Ports
          ↓
Day 3 → Wireshark Filters
          ↓
Day 4 → IP & Traffic Analysis
          ↓
Day 5 → Subnetting & CIDR
          ↓
Day 6 → DHCP
          ↓
Day 7 → ARP & MAC Resolution
          ↓
Day 8 → TCP Connection & Handshake
```

### Simple Investigation Flow

```text
IP
 ↓
Port
 ↓
Wireshark
 ↓
TCP Flags
 ↓
Connection Behavior
 ↓
Evidence
 ↓
SOC Conclusion
```

Day 8 builds on the previous networking concepts by adding **TCP connection behavior** to the investigation process.

---

# 🧠 Interview Check

### 1. What is the TCP 3-way handshake?

```text
SYN → SYN/ACK → ACK
```

### 2. What does SYN mean?

A TCP connection attempt is being initiated.

### 3. What does SYN/ACK mean?

The server acknowledges the client's SYN and responds with its own SYN.

### 4. What does the final ACK do?

It acknowledges the SYN/ACK and completes the TCP 3-way handshake.

### 5. Does seeing a SYN automatically mean a successful connection?

**No.**

The response and subsequent handshake packets must be investigated.

### 6. Wireshark filter for initial SYN packets?

```text
tcp.flags.syn == 1 && tcp.flags.ack == 0
```

### 7. Difference between SYN and SYN/ACK?

```text
SYN
→ Initiates the connection

SYN + ACK
→ Acknowledges the client's SYN and responds with the server's SYN
```

---

# 🧠 Key Takeaways

```text
SYN       → Connection attempt
SYN/ACK   → Server response
ACK       → Completes handshake
```

### Remember

- TCP uses a 3-way handshake to establish a connection.
- The SYN sender is normally the connection initiator.
- Port numbers help identify the communication endpoints/services.
- A SYN alone does not prove a successful connection.
- Follow the complete TCP conversation before making a conclusion.
- Repeated SYN packets can have multiple legitimate or network-related explanations.
- SOC conclusions should be based on evidence and context.

---

# 📸 Evidence

👉 [View TCP 3-Way Handshake Analysis Screenshot](./screenshots/01-tcp-handshake-analysis.png)

The screenshot demonstrates the observed:

```text
SYN
↓
SYN/ACK
↓
ACK
```

sequence in Wireshark.

---

# 📊 Day 8 Status

**Status:** ✅ Completed

### Skills Practiced

- TCP fundamentals
- TCP 3-way handshake
- SYN / SYN-ACK / ACK analysis
- Client/server direction
- TCP port analysis
- Wireshark TCP filtering
- Connection success verification
- Evidence-based SOC investigation
