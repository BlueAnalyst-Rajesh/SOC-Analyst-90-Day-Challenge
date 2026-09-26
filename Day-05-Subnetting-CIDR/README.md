# 🔵 Day 05 — Subnetting & CIDR for SOC Analysts

## 🎯 Objective

Understand how subnetting and CIDR help a SOC Analyst identify:

- Network boundaries
- Same-subnet communication
- Private vs Public IP addresses
- Internal vs External traffic
- Network context during security investigations

---

# 📚 Concepts Learned

## 1. IPv4 Address

An IPv4 address contains **32 bits** and is divided into four octets.

Example:

```text
10.240.36.237
````

Each octet ranges from:

```text
0 – 255
```

---

## 2. CIDR & Subnet Mask

CIDR specifies how many bits are used for the network portion.

| CIDR | Subnet Mask   |
| ---- | ------------- |
| /8   | 255.0.0.0     |
| /16  | 255.255.0.0   |
| /24  | 255.255.255.0 |

For `/24`:

```text
Network Bits : 24
Host Bits    : 8
Total IPs    : 256
```

Example:

```text
10.240.36.237/24
```

Network address:

```text
10.240.36.0/24
```

Traditional IPv4 range:

```text
Network   : 10.240.36.0
Usable    : 10.240.36.1 – 10.240.36.254
Broadcast : 10.240.36.255
```

---

# 🖥️ Practical 1 — Windows Network Configuration

Used `ipconfig` to identify the local IPv4 configuration.

### Observed Values

| Parameter       | Value           |
| --------------- | --------------- |
| IPv4 Address    | `10.240.36.237` |
| Subnet Mask     | `255.255.255.0` |
| CIDR            | `/24`           |
| Default Gateway | `10.240.36.109` |

### Network Calculation

```text
IP Address  : 10.240.36.237/24
Network     : 10.240.36.0/24
```

Therefore:

```text
Network Portion : 10.240.36
Host Portion    : 237
```

### SOC Relevance

Knowing the local subnet helps an analyst determine whether an IP address belongs to the internal network or is outside the local network.

📸 **Evidence:**
[View IP Configuration](./screenshots/01-ipconfig-network-details.png)

---

# 🌐 Practical 2 — Wireshark TCP/443 Analysis

Captured local network traffic using Wireshark.

### Display Filter

```text
ip.addr == 10.240.36.237 && tcp.port == 443
```

### Selected Packet — Frame 273

```text
Source IP      : 10.240.36.237
Destination IP : 4.150.223.101
Protocol       : TCP
Source Port    : 49303
Destination    : 443
TCP Flag       : SYN
```

Packet direction:

```text
10.240.36.237:49303
        ↓ SYN
4.150.223.101:443
```

The source `10.240.36.237` belongs to the private `10.0.0.0/8` range.

The destination `4.150.223.101` is a public IPv4 address.

Therefore:

```text
Private / Internal Host
        ↓
Public / External Host
```

TCP/443 is commonly associated with HTTPS/TLS traffic.

The selected packet showed:

```text
Flags: 0x002 (SYN)
```

A SYN is used to initiate a TCP connection.

The capture also showed SYN-ACK responses from the destination:

```text
4.150.223.101:443
        ↓ SYN-ACK
10.240.36.237
```

However, the selected conversation was marked **Incomplete**, so the evidence should not be described as a fully established connection.

📸 **Evidence:**
[View Wireshark Analysis](./screenshots/02-wireshark-subnet-analysis.png)

---

# 🔎 Wireshark Observations

Multiple TCP/443 packets were observed.

Examples:

```text
10.240.36.237:49303 → 4.150.223.101:443 [SYN]

10.240.36.237:61050 → 4.150.223.101:443 [SYN]

10.240.36.237:55493 → 4.150.223.101:443 [SYN]

10.240.36.237:62292 → 184.86.248.170:443 [SYN]
```

The destination also returned SYN-ACK packets for some connections.

This demonstrates the beginning of the TCP connection process:

```text
Client                  Server

  SYN  -------------------->

       <-------------------- SYN-ACK

  ACK  -------------------->
```

---

# 🧠 Same Subnet Analysis

For a `/24` network, the first three octets represent the network portion.

Example:

```text
192.168.10.25/24
192.168.10.200/24
```

Both belong to:

```text
192.168.10.0/24
```

Therefore, they are in the **same subnet**.

But:

```text
192.168.10.25/24
192.168.11.25/24
```

belong to:

```text
192.168.10.0/24
192.168.11.0/24
```

Therefore, they are in **different subnets**.

### Important Lesson

Always use the **given CIDR prefix** when determining whether two IP addresses belong to the same subnet.

---

# 🚨 SOC Scenario — SMB Traffic

Consider:

```text
Source      : 10.10.20.15
Destination : 10.10.20.80
Protocol    : TCP
Port        : 445
Network     : 10.10.20.0/24
```

Both systems belong to:

```text
10.10.20.0/24
```

Therefore, they are in the same subnet.

TCP/445 is commonly associated with:

**SMB — Server Message Block**

SMB is commonly used for file and network resource sharing.

### SOC Perspective

Same-subnet communication is not automatically malicious.

However, unusual SMB activity may require investigation because SMB can be involved in lateral movement.

A SOC Analyst should check:

* Source and destination host
* User/account involved
* Authentication events
* Process responsible for the connection
* Time of activity
* Other network connections
* Whether the activity is expected

---

# 🔍 SOC Investigation Mindset

Network information alone is not enough to determine whether activity is malicious.

For example:

```text
10.240.36.237 → 4.150.223.101:443
```

This tells us:

* Source is private/internal
* Destination is public/external
* Protocol is TCP
* Destination port is 443
* TCP SYN was observed

But it does **not** tell us whether the activity is malicious.

Additional evidence may include:

* Destination IP reputation
* DNS/domain information
* Process responsible for the connection
* Connection frequency
* Endpoint activity
* User and time context

### Core SOC Principle

```text
Evidence > Assumption
```

---

# 🛠️ Wireshark Filters Practiced

### Identify traffic involving the local IP

```text
ip.addr == 10.240.36.237
```

### Filter TCP traffic

```text
ip.addr == 10.240.36.237 && tcp
```

### Filter TCP/443 traffic

```text
ip.addr == 10.240.36.237 && tcp.port == 443
```

---

# 💼 Interview Preparation

### Q1. What is CIDR?

CIDR stands for **Classless Inter-Domain Routing**.

Example:

```text
10.240.36.237/24
```

`/24` means the first 24 bits represent the network portion.

---

### Q2. What is the subnet mask for /24?

```text
255.255.255.0
```

---

### Q3. What is the network address of 10.240.36.237/24?

```text
10.240.36.0
```

---

### Q4. Are 192.168.10.25/24 and 192.168.10.200/24 in the same subnet?

Yes.

Both belong to:

```text
192.168.10.0/24
```

---

### Q5. Are 192.168.10.25/24 and 192.168.11.25/24 in the same subnet?

No.

They belong to different `/24` networks.

---

### Q6. Does being on the same subnet mean the communication is safe?

No.

Same-subnet communication can be legitimate or suspicious depending on the context and evidence.

---

### Q7. What is TCP/445 commonly used for?

TCP/445 is commonly used for **SMB**.

---

### Q8. Does TCP/443 prove that traffic is safe?

No.

TCP/443 is commonly associated with HTTPS/TLS, but the port number alone cannot determine whether the activity is benign or malicious.

---

# 🛡️ SOC Relevance

Subnetting helps a SOC Analyst understand network relationships during an investigation.

It can help determine:

* Whether two systems are in the same network
* Whether traffic is internal or external
* Whether an IP is private or public
* Whether communication crosses a network boundary
* Whether unusual internal communication requires investigation

Important:

```text
Same Subnet ≠ Safe
Private IP ≠ Safe
Port 443 ≠ Safe
```

The analyst must combine network information with additional evidence before reaching a conclusion.

---

# 📝 Key Takeaways

* IPv4 addresses contain 32 bits.
* `/24` = `255.255.255.0`.
* `/24` contains 24 network bits and 8 host bits.
* `10.240.36.237/24` belongs to `10.240.36.0/24`.
* `10.0.0.0/8` is a private IPv4 range.
* TCP/443 is commonly used for HTTPS/TLS.
* TCP/445 is commonly used for SMB.
* SYN is used to initiate a TCP connection.
* Same subnet does not automatically mean safe.
* IP address + port alone are not enough to determine malicious activity.
* SOC investigation requires evidence and context.
---

# ✅ Day 05 Status

| Area                  | Status |
| --------------------- | ------ |
| IPv4 Basics           | ✅      |
| Subnet Mask           | ✅      |
| CIDR                  | ✅      |
| Network Address       | ✅      |
| Same Subnet Analysis  | ✅      |
| Private/Public IP     | ✅      |
| Wireshark Practical   | ✅      |
| SOC Application       | ✅      |
| Interview Preparation | ✅      |

### Self-Assessment

Main improvement areas:

* Clearly distinguish network address from host address.
* Use the given CIDR prefix when comparing subnets.
* Avoid making security conclusions from only an IP address or port.
* Support SOC conclusions with evidence and context.

---

## 🧠 Quick Revision

```text
IPv4
→ 32 bits

/8
→ 255.0.0.0

/16
→ 255.255.0.0

/24
→ 255.255.255.0

10.0.0.0/8
→ Private range

10.240.36.237/24
→ 10.240.36.0/24

TCP 443
→ Commonly HTTPS/TLS

TCP 445
→ SMB

SYN
→ Starts TCP connection

SOC
→ Evidence > Assumption
```
