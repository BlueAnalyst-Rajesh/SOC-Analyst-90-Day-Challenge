
# 🔵 Day 03 — Wireshark Display Filters

## 🎯 Objective

Learn how to use Wireshark Display Filters to quickly identify and investigate specific network traffic.

The goal was to move from manually reading packets to filtering traffic efficiently like a SOC Analyst.

---

## 📚 Concepts Learned

- Wireshark Display Filters
- DNS traffic filtering
- TCP and UDP filtering
- Filtering by IP address
- Filtering by port
- Combining multiple filter conditions
- DNS query filtering
- DNS error filtering
- TCP SYN filtering
- IPv4 vs IPv6 filtering
- Identifying potential port scanning patterns

---

## 🔎 Important Display Filters

| Purpose | Wireshark Filter |
|---|---|
| DNS | `dns` |
| TCP | `tcp` |
| UDP | `udp` |
| HTTP | `tcp.port == 80` |
| HTTPS | `tcp.port == 443` |
| SSH | `tcp.port == 22` |
| SMB | `tcp.port == 445` |
| RDP | `tcp.port == 3389` |
| IPv4 address | `ip.addr == x.x.x.x` |
| Source IPv4 | `ip.src == x.x.x.x` |
| Destination IPv4 | `ip.dst == x.x.x.x` |
| IPv6 address | `ipv6.addr == x:x::x` |
| DNS Queries | `dns.flags.response == 0` |
| DNS Errors | `dns.flags.rcode != 0` |
| TCP SYN | `tcp.flags.syn == 1 && tcp.flags.ack == 0` |

---

# 🧪 Practical Analysis

## Task 1 — DNS Traffic

### Filter Used

```text
dns
````

### Result

**44 DNS packets** were observed in the capture.

The DNS traffic included:

* 24 queries
* 20 responses
* A records
* AAAA records
* HTTPS records
* CNAME responses

One DNS response returned `NXDOMAIN` / "No such name".

This was not automatically considered malicious because a single failed DNS lookup can occur during normal network activity.

---

## Task 2 — HTTPS Traffic

### Filter Used

```text
tcp.port == 443
```

### Observation

A TCP connection involving port **443** was observed.

Example:

```text
Source Port: 443
Destination Port: 49414
Protocol: TCP
```

The packet used IPv6 addressing.

Port 443 is commonly associated with HTTPS/TLS traffic.

---

## Task 3 — SMB Traffic

### Filter Used

```text
tcp.port == 445
```

### Result

No packets were displayed.

This means:

> No TCP/445 traffic was observed in this particular capture.

It does **not** prove that SMB was never used on the network.

---

## Task 4 — DNS Queries

### Filter Used

```text
dns.flags.response == 0
```

### Result

**34 packets displayed.**

This filter identifies DNS queries rather than DNS responses.

---

## Task 5 — DNS Errors

### Filter Used

```text
dns.flags.rcode != 0
```

### Result

**0 packets observed.**

No DNS responses with a non-zero response code were observed in this filtered capture.

---

## Task 6 — TCP SYN Packets

### Filter Used

```text
tcp.flags.syn == 1 && tcp.flags.ack == 0
```

### Result

**49 packets displayed.**

The filtered traffic contained TCP SYN packets, including connections targeting port 443.

A SYN packet represents an initial TCP connection attempt.

---

# 🧠 SOC Investigation Exercise

## Normal Web Browsing vs Port Scanning

### Scenario A

Multiple destination IP addresses were observed, with traffic mainly targeting:

```text
80
443
```

### Assessment

Based only on the observed traffic, this pattern is more consistent with normal web browsing.

Additional context would still be required to completely rule out malicious activity.

---

### Scenario B

A single destination IP was contacted across multiple ports such as:

```text
21
22
23
25
53
80
110
139
445
3389
```

### Assessment

This pattern is consistent with a potential port-scanning activity because one destination is being probed across many different ports.

A SOC analyst would investigate further instead of immediately declaring it malicious.

---

# 🔍 SOC Mindset

The key lesson from this exercise was:

```text
Observe
   ↓
Filter
   ↓
Identify Pattern
   ↓
Form Hypothesis
   ↓
Collect More Evidence
   ↓
Correlate
   ↓
Conclude
```

A single unusual packet does not automatically prove malicious activity.

Context, frequency, destination, ports, process information, authentication events and other telemetry should be correlated before making a final assessment.

---

# 📸 Practical Evidence

Wireshark screenshots:

* [DNS Query Filter](./screenshots/01-dns-query-filter.png)
* [TCP SYN Filter](./screenshots/02-tcp-syn-filter.png)

---

# 📝 Key Takeaways

* Learned how to use Wireshark Display Filters efficiently.
* Learned to isolate DNS queries and DNS errors.
* Learned to identify TCP SYN connection attempts.
* Learned the difference between IPv4 and IPv6 filtering.
* Learned how port patterns can indicate normal browsing or potential scanning.
* Learned not to classify traffic as malicious without sufficient evidence.
* Practiced evidence-based SOC investigation.

---

# 💭 Reflection

Day 3 helped me move from simply identifying network packets to filtering traffic and looking for behavioral patterns.

The main SOC lesson I learned is:

> Unusual traffic is an investigation starting point, not automatically proof of an attack.

---

# 📊 Day 03 Status

| Area                      | Status      |
| ------------------------- | ----------- |
| Wireshark Display Filters | ✅ Completed |
| DNS Filtering             | ✅ Completed |
| IP Filtering              | ✅ Completed |
| Port Filtering            | ✅ Completed |
| Combined Filters          | ✅ Completed |
| DNS Query Analysis        | ✅ Completed |
| DNS Error Analysis        | ✅ Completed |
| TCP SYN Analysis          | ✅ Completed |
| Port Scan Reasoning       | ✅ Completed |

### 🏆 Day 03 — COMPLETED
