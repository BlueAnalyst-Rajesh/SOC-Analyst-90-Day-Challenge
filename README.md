
# 🛡️ Day 01 — OSI Model & TCP/IP Stack in Security Monitoring

> **90-Day SOC Analyst Challenge | Day 01**

### 🎯 Objective

Understand the OSI Model and TCP/IP Model and learn how networking concepts are applied in real-world SOC monitoring and security investigations.

---

## 📚 1. Concepts Learned

### 🔹 OSI Model

The OSI Model consists of 7 layers:

| Layer | Name | SOC Relevance |
|---|---|---|
| 7 | Application | DNS, HTTP, HTTPS, SSH |
| 6 | Presentation | Data encoding / encryption |
| 5 | Session | Session management |
| 4 | Transport | TCP, UDP, Ports |
| 3 | Network | IP addressing and routing |
| 2 | Data Link | MAC address, ARP |
| 1 | Physical | Physical transmission |

### 🔹 TCP/IP Model

The TCP/IP model is the practical networking model used for Internet communication.

- **Application**
- **Transport**
- **Internet**
- **Network Access**

---

## 🔐 2. SOC Relevance

As a SOC Analyst, network information helps investigate suspicious communication between systems.

### Important Network Indicators

- **Source IP** → Who is communicating?
- **Destination IP** → Where is the communication going?
- **Source Port** → Originating service/application port
- **Destination Port** → Target service port
- **Protocol** → How is the communication happening?
- **DNS Query** → What domain is being requested?
- **Network Behavior** → Is the activity normal or unusual?

> ⚠️ **Important SOC Principle:**  
> An IP address, port number, and protocol alone are not enough to conclude that an activity is malicious. Additional context and correlation are required.

---

## 🧪 3. Practical Lab — Wireshark

### 🛠️ Tool Used

**Wireshark**

### 🔍 Activities Performed

- Captured live network traffic
- Identified TCP traffic
- Identified DNS traffic
- Analyzed source and destination information
- Analyzed TCP ports
- Observed TCP SYN packets
- Observed TCP three-way handshake

### 🔎 Wireshark Filters Used

```text
tcp
dns
ip.addr
tcp.flags.syn == 1 && tcp.flags.ack == 0
dns.flags.response == 0
````

---

## 🌐 4. TCP Packet Analysis

During the packet capture, TCP traffic was analyzed using Wireshark.

### Key Fields Observed

* Source IP
* Destination IP
* Source Port
* Destination Port
* TCP Flags
* SYN
* SYN-ACK
* ACK

### 🔄 TCP Three-Way Handshake

```text
Client  →  Server : SYN
Client  ←  Server : SYN-ACK
Client  →  Server : ACK
```

This process establishes a TCP connection between the two endpoints.

---

## 🌍 5. DNS Traffic Analysis

DNS traffic was analyzed using Wireshark.

### Key Observations

* **Protocol:** UDP
* **Destination Port:** 53
* DNS queries were visible in the packet details.

### Example Domains Observed

```text
web.whatsapp.com
c.pki.goog
settings-win.data.microsoft.com
```

### 🛡️ SOC Relevance

DNS traffic can provide useful investigation data.

A SOC Analyst can investigate:

* Suspicious domains
* Unusual DNS queries
* High-frequency DNS requests
* Newly observed domains
* Potential command-and-control activity

---

## 🌐 6. HTTP Traffic Analysis

HTTP traffic was analyzed using Wireshark.

### Important Fields

* Source IP
* Destination IP
* Source Port
* Destination Port
* HTTP Method
* Host
* URI
* User-Agent
* Response Status

**HTTP commonly uses TCP port 80.**

---

## 🚨 7. SOC Investigation Scenario

### Scenario

A SOC alert shows a network connection between a source IP and destination IP using TCP.

### Investigation Process

```text
Identify Source IP
        ↓
Identify Destination IP
        ↓
Check Ports & Protocol
        ↓
Check DNS History
        ↓
Check Destination Reputation
        ↓
Check Connection Frequency
        ↓
Identify Initiating Process
        ↓
Correlate Endpoint / SIEM Logs
        ↓
Determine Whether Activity Is Suspicious
```

### 🔑 Key Takeaway

A SOC Analyst should avoid making a conclusion based on a single indicator.

**Evidence + Context + Correlation = Better Investigation**

---

## 💬 8. Technical Interview Questions

### Q1. What is the difference between OSI and TCP/IP?

**Answer:**

OSI is a **7-layer conceptual/reference model**, while TCP/IP is a **practical networking model used for Internet communication**.

---

### Q2. What is Encapsulation?

**Answer:**

Encapsulation is the process where each networking layer adds its own control information, such as headers, to the data before transmission.

```text
Application Data
       ↓
TCP Header
       ↓
IP Header
       ↓
Ethernet Header
       ↓
Bits
```

The reverse process is called **Decapsulation**.

---

### Q3. Why is the Network Layer important for a SOC Analyst?

**Answer:**

The Network Layer provides logical IP addressing and routing.

Source and destination IP addresses help a SOC Analyst:

* Identify communicating systems
* Trace suspicious connections
* Correlate network activity
* Investigate potential attacks
* Use threat intelligence during investigations

---

## 🧠 9. Key SOC Takeaways

| Question          | Network Information          |
| ----------------- | ---------------------------- |
| **WHO?**          | Source IP                    |
| **WHERE?**        | Destination IP               |
| **HOW?**          | TCP/UDP + Port               |
| **WHAT?**         | Application Protocol         |
| **IS IT NORMAL?** | Context + Logs + Correlation |

### SOC Investigation Mindset

> **Don't just look at an IP. Understand the behavior behind the connection.**

---

## 📸 10. Practical Evidence

The following Wireshark evidence was collected:

* Network traffic capture
* TCP packet analysis
* DNS packet analysis
* TCP SYN / SYN-ACK / ACK handshake

### Evidence

Screenshots are available in the [`screenshots`](./screenshots/) folder.

---

## 📝 11. Reflection

This day helped me connect networking theory with real packet-level activity.

I learned how OSI and TCP/IP concepts appear in real network traffic and how a SOC Analyst can use IP addresses, ports, protocols, DNS traffic, and TCP behavior during security investigations.

The Wireshark exercises helped me understand the difference between simply observing network traffic and investigating it from a security perspective.

---

## ✅ 12. Day 01 Status

| Category            | Status      |
| ------------------- | ----------- |
| OSI Model           | ✅ Completed |
| TCP/IP Model        | ✅ Completed |
| Encapsulation       | ✅ Completed |
| TCP Analysis        | ✅ Completed |
| DNS Analysis        | ✅ Completed |
| Wireshark Practical | ✅ Completed |
| SOC Investigation   | ✅ Completed |
| Interview Questions | ✅ Completed |

### 🎯 Day 01 — Completed

**Tools:** Wireshark
**Focus:** Networking Fundamentals + Security Monitoring
**SOC Skills:** Packet Analysis, Network Investigation, Basic Traffic Analysis

```
