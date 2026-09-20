
# Day 02 — Core Network Protocols & Port Numbers

## 🎯 Objective

Learn the most important network protocols and port numbers used in SOC operations, and understand how network traffic can be investigated from a SOC analyst perspective.

---

## 📚 Protocols & Ports Learned

| Protocol | Port | Transport | Main Purpose | SOC Relevance |
|---|---:|---|---|---|
| DNS | 53 | UDP/TCP | Domain name resolution | C2, DNS tunneling, suspicious queries |
| HTTP | 80 | TCP | Web communication | Web attacks, suspicious requests |
| HTTPS | 443 | TCP | Encrypted web communication | TLS traffic analysis |
| DHCP | 67/68 | UDP | Dynamic IP configuration | Rogue DHCP detection |
| SSH | 22 | TCP | Secure remote administration | Brute-force / unauthorized access |
| SMB | 445 | TCP | File and printer sharing | Lateral movement |
| RDP | 3389 | TCP/UDP | Remote Desktop access | Brute-force / unauthorized remote access |

> **Important:** A port number alone does not prove malicious activity. SOC analysts need context, authentication logs, endpoint telemetry, reputation, frequency, and other correlated evidence.

---

# 🔎 1. DNS — Port 53

### What is DNS?

DNS (Domain Name System) translates domain names into IP addresses.

Example:

```text
google.com → IP address
````

DNS commonly uses:

```text
UDP 53
```

TCP 53 can also be used in certain situations.

### SOC Perspective

Attackers may abuse DNS for:

* Command and Control (C2)
* DNS tunneling
* Suspicious domain communication
* Data exfiltration

A single unusual DNS query does **not** automatically mean compromise.

Useful investigation points:

```text
Source IP
Destination DNS server
Requested domain
Query frequency
Domain reputation
Subdomain patterns
Entropy / randomness
Host activity
```

---

# 🌐 2. HTTP — Port 80

HTTP is commonly used for web communication.

Common HTTP methods:

```text
GET
POST
PUT
DELETE
```

Common response codes:

```text
200 OK
301/302 Redirect
400 Bad Request
401 Unauthorized
403 Forbidden
404 Not Found
500 Server Error
```

### SOC Perspective

A response code alone does not prove whether an attack succeeded.

For example:

```text
GET /cmd.exe → 404
```

only tells us that the server returned a 404 response.

We need additional evidence to determine whether the request was malicious or successful.

---

# 🔐 3. HTTPS — Port 443

HTTPS provides encrypted web communication using TLS.

Basic model:

```text
HTTP
 ↓
TLS encryption
 ↓
TCP
 ↓
IP
```

Modern web traffic may also use HTTP/3 over QUIC/UDP 443, but traditional HTTPS analysis commonly involves TCP/443.

### SOC Perspective

SOC analysts may investigate:

* Destination IP
* TLS information
* Domain
* Connection frequency
* Certificate information
* Endpoint process
* Destination reputation

Encrypted traffic does not automatically mean malicious traffic.

---

# 📡 4. DHCP — Ports 67/68

DHCP automatically provides network configuration to clients.

Basic DORA process:

```text
Discover
   ↓
Offer
   ↓
Request
   ↓
ACK
```

Ports:

```text
DHCP Server → UDP 67
DHCP Client → UDP 68
```

### SOC Perspective

A rogue DHCP server may provide unauthorized network configuration.

Potential investigation indicators:

```text
Unexpected DHCP server
Unexpected DHCP Offer
Unexpected gateway
Unknown MAC address
Unexpected DNS server
```

---

# 🗂️ 5. SMB — TCP 445

SMB stands for:

**Server Message Block**

It is commonly used for:

* File sharing
* Printer sharing
* Windows network resources

### SOC Perspective

SMB is important for detecting potential lateral movement.

Example attack pattern:

```text
Compromised PC
     ↓
Credential abuse
     ↓
TCP 445
     ↓
Another internal system
     ↓
File/resource access
```

Useful investigation points:

```text
Source host
Destination host
User/account
Authentication events
Process responsible for connection
Number of internal systems contacted
EDR telemetry
```

> **Correction:** SMB uses **TCP 445**, not port 444.

---

# 🖥️ 6. SSH — TCP 22

SSH (Secure Shell) is commonly used for secure remote administration of Linux/Unix systems.

### SOC Perspective

Investigate:

```text
Who connected?
From where?
Was authentication successful?
How many failed attempts occurred?
What account was used?
What happened after authentication?
```

SSH access should normally follow the organization's authorization and access-control policies.

A pattern such as:

```text
Many failed attempts
        ↓
Successful authentication
        ↓
Post-login activity
```

is worth investigating, but the pattern alone does not prove compromise.

---

# 🖥️ 7. RDP — TCP/UDP 3389

RDP (Remote Desktop Protocol) is used for remote access to Windows systems.

Common port:

```text
TCP 3389
```

UDP 3389 may also be involved in modern RDP connections.

### SOC Perspective

RDP is important during investigations involving:

* Brute-force attempts
* Unauthorized remote access
* Compromised credentials
* Lateral movement

Important Windows authentication events include:

```text
4624 → Successful logon
4625 → Failed logon
```

For RDP investigation, **Logon Type 10 (RemoteInteractive)** can be particularly relevant.

---

# 🧪 Practical Wireshark Analysis

## 1. DNS Traffic Analysis

Wireshark filter used:

```text
dns
````

Observed DNS traffic:

```text
Source IP        : 10.104.49.237
Destination IP   : 10.104.49.158
Protocol         : UDP
Source Port      : 51464
Destination Port : 53
Query            : web.whatsapp.com
```

### SOC Interpretation

The captured traffic shows a DNS query from a client to a DNS server using UDP port 53.

From a SOC perspective, DNS traffic can be investigated for:

* Suspicious domains
* Unusual query frequency
* DNS tunneling indicators
* Command and Control (C2) activity
* Domain reputation

A single DNS query is not enough to determine whether the activity is malicious.

---

## 2. TCP Port 443 Traffic Analysis

Wireshark filter used:

```text
tcp.port == 443
```

Observed TCP packet:

```text
Source IP        : 104.18.39.21
Destination IP   : 10.180.76.237
Protocol         : TCP
Source Port      : 443
Destination Port : 52409
TCP Flags        : ACK
Sequence Number  : 25
Acknowledgment   : 29
TCP Payload      : 0 bytes
```

### SOC Interpretation

The packet shows TCP traffic using destination/source port 443, which is commonly associated with HTTPS web communication.

The packet itself shows TCP port 443 activity. Further TLS details and endpoint context would be required for deeper investigation.

A SOC analyst can investigate:

* Source and destination IP
* Connection frequency
* TLS information, when available
* Destination/domain reputation
* Endpoint process responsible for the connection
* Related network activity

Port 443 traffic alone does not prove that the activity is malicious or benign.

---

# 🚨 SOC Investigation Scenario

### Alert

```text
Source IP      : 10.10.20.15
Destination IP : 10.10.20.50
Destination    : TCP 3389
Failed logins  : 23
Time           : 4 minutes
Successful     : 1 login
```

### Q1. What service is involved?

**RDP — Remote Desktop Protocol**

Port:

```text
3389
```

### Q2. Why is this interesting?

Multiple failed authentication attempts followed by a successful login can indicate suspicious authentication activity and requires investigation.

### Q3. Does this prove compromise?

**No.**

Network/authentication evidence alone does not prove that the system was compromised.

### Q4. What would you investigate next?

```text
Source IP reputation
Username
Windows Event ID 4625
Windows Event ID 4624
Logon Type
Successful login time
Endpoint process activity
Process creation
Other network connections
User activity after login
EDR alerts
Destination host activity
```

---

# 🎯 Interview Questions

## 1. Why is DNS important in SOC operations?

DNS converts domain names into IP addresses.

SOC analysts monitor DNS because attackers can abuse DNS for:

* Command and Control
* DNS tunneling
* Suspicious domain communication

However, an unusual DNS query alone is not enough to confirm malicious activity.

---

## 2. Difference between HTTP and HTTPS?

| HTTP                         | HTTPS                       |
| ---------------------------- | --------------------------- |
| Port 80                      | Port 443                    |
| No TLS encryption            | Uses TLS                    |
| Data is not protected by TLS | Data is protected by TLS    |
| Common web communication     | Encrypted web communication |

**Interview answer:**

> HTTP commonly uses port 80 and does not provide TLS encryption. HTTPS commonly uses port 443 and uses TLS to protect communication.

---

## 3. Why is TCP 445 important for SOC analysts?

TCP 445 is commonly used by SMB for Windows file and resource sharing.

Attackers may abuse SMB during lateral movement or unauthorized access.

A connection to TCP 445 does not automatically mean an attack. The SOC analyst should correlate it with authentication logs, endpoint activity, user context, and other evidence.

---

# 🧠 SOC Mental Model

When I see a network connection, I should ask:

```text
WHO?
 ↓
Source IP / Host / User

WHAT?
 ↓
Protocol / Port / Activity

WHERE?
 ↓
Destination IP / Host / Domain

WHEN?
 ↓
Timestamp / Frequency

WHY?
 ↓
Is this expected?

WHAT HAPPENED AFTER?
 ↓
Authentication
Process activity
Network connections
Endpoint activity
```

---

# 📸 Practical Evidence

Wireshark screenshots:

* [DNS Traffic Analysis](./screenshots/01-dns-analysis.png)
* [HTTPS/TCP Traffic Analysis](./screenshots/02-https-tcp-analysis.png)

---

# 📝 Key Takeaways

### I learned:

* Important SOC network protocols and ports
* DNS and its SOC relevance
* HTTP vs HTTPS
* DHCP DORA process
* SMB and lateral movement
* SSH authentication investigation
* RDP authentication investigation
* Windows Event IDs 4624 and 4625
* Wireshark port-based traffic analysis
* Why port numbers alone cannot prove malicious activity

### Most Important Lesson

> **Do not identify an attack from a port number alone. Identify the service, collect evidence, correlate events, and then determine whether the activity is suspicious.**

---

# 💭 Reflection

Day 2 improved my understanding of how common network protocols appear during SOC investigations.

I learned that knowing port numbers is not enough. A SOC analyst must connect:

```text
Network Traffic
      +
Authentication Logs
      +
Endpoint Activity
      +
Threat Intelligence
      +
User Context
      ↓
Better Investigation
```

---

# ✅ Day 02 Status

**Completed**

### Self-Assessment

| Area                       | Status |
| -------------------------- | ------ |
| Protocol & Port Knowledge  | ✅      |
| DNS Analysis               | ✅      |
| HTTP/HTTPS                 | ✅      |
| DHCP                       | ✅      |
| SMB                        | ✅      |
| SSH                        | ✅      |
| RDP                        | ✅      |
| Wireshark Analysis         | ✅      |
| SOC Investigation Thinking | ✅      |
| Evidence-Based Analysis    | ✅      |


