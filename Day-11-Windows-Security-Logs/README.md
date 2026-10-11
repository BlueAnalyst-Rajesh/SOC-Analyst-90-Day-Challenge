# Day 11 — Windows Security Logs & Network Investigation

## 🎯 Objective

Investigate Windows Security events and network traffic using PowerShell and Wireshark.

Goals:

- Analyze successful and failed logon events.
- Understand Windows Filtering Platform (WFP) events.
- Investigate DNS queries and responses.
- Correlate DNS resolution with TCP and TLS metadata.
- Practise evidence-based SOC alert triage.

## 🧰 Tools Used

- Windows PowerShell — Query and filter Windows event logs.
- Windows Event Viewer / Security Log — Inspect Windows security events.
- Wireshark — Analyze DNS, TCP, and TLS traffic.

## 🔗 Previous Knowledge Applied

This investigation builds on networking concepts covered in previous challenge days.

- TCP/IP and Ports: Interpreted TCP/443 and UDP/53 traffic.
- Wireshark Filters: Used the "dns" display filter.
- DNS Investigation: Examined query types and returned IP addresses.
- TCP Three-Way Handshake: Identified SYN, SYN/ACK, and ACK.
- Evidence-Based Analysis: Avoided declaring an activity malicious or safe based on a single indicator.

---

## 🔬 1. Windows Security Event Investigation

Event ID 4624 — Successful Logon

Command used:

Get-WinEvent -FilterHashtable @{
    LogName = 'Security'
    Id = 4624, 4625
} -MaxEvents 10 |
Select-Object TimeCreated, Id, ProviderName, Message

Observed results:

- The retrieved matching events were Event ID "4624".
- Provider: "Microsoft-Windows-Security-Auditing".
- One examined event contained:

Field| Observed value
Target account| "SYSTEM"
Logon type| "5"
Process| "C:\Windows\System32\services.exe"
Source IP| "-"

Analysis: Logon type 5 represents a service logon. The observed account and process are consistent with normal Windows service activity, but this event alone does not prove that the activity is safe.

Event ID 4625 — Failed Logon

Command used:

Get-WinEvent -FilterHashtable @{
    LogName = 'Security'
    Id = 4625
} -MaxEvents 10 |
Select-Object TimeCreated, Id, Message

Observed result: No events matching the query were returned.

Analysis: This query did not return matching failed-logon events. It does not prove that failed logons have never occurred or rule out brute-force activity.

---

## 🔬 2. Security Event ID Summary

Command used:

Get-WinEvent -LogName Security -MaxEvents 30 |
Group-Object Id |
Sort-Object Count -Descending |
Select-Object Name, Count

Observed results:

Event ID| Count in retrieved 30 events| Meaning
5158| 22| Windows Filtering Platform permitted a bind to a local port
5156| 8| Windows Filtering Platform permitted a network connection

Analysis: These counts represent only the 30 events retrieved by the command, not the entire Security log. A permitted connection or port bind is not automatically malicious or automatically safe.

---

## 🔬 3. Windows Filtering Platform — Event ID 5156

Examined permitted network connections and their application, IP addresses, ports, and protocols.

Example A — UDP/443

Field| Observed value
Application| "msedgewebview2.exe"
Destination IPv6| "2404:6800:4000:1025::54"
Destination port| "443"
Protocol| "17" (UDP)

Analysis: UDP/443 may be used by QUIC-based traffic. This event alone does not confirm QUIC or HTTP/3, nor does it establish whether the connection was benign.

Example B — DNS Traffic

Field| Observed value
Application| "chrome.exe"
Source IP| "10.202.17.237"
Destination DNS server| "10.202.17.55"
Destination port| "53"
Protocol| "17" (UDP)

Additional DNS-related events showed traffic to the same DNS server using different source ports.

Analysis: Event ID 5156 records a permitted network connection. It does not, by itself, reveal the DNS query name or prove that the connection was safe.

---

## 🔬 4. Wireshark — DNS Investigation

Display filter:

dns

Observed DNS Details

Field| Observed value
Client IP| "10.202.17.237"
DNS server| "10.202.17.55"
Transport| UDP
Queried domain| "fp.msedge.net"
Query types| HTTPS, AAAA, A
IPv4 address in the observed A response| "204.79.197.222"
DNS response status| No error reported

Analysis: DNS analysis identified the queried domain and its returned IPv4 address. A familiar domain or an error-free DNS response does not, by itself, prove that the resulting network activity is safe.

---

## 🔬 5. DNS → TCP → TLS Correlation

Investigated the observed packet sequence associated with "fp.msedge.net".

Connection Details

Observation| Evidence
DNS query name| "fp.msedge.net"
Observed A-record answer| "204.79.197.222"
TCP source| "10.202.17.237:49956"
TCP destination| "204.79.197.222:443"
TCP handshake| SYN → SYN/ACK → ACK
TLS metadata| TLS 1.3 Client Hello
SNI| "fp.msedge.net"

What the Evidence Shows

1. DNS: An A-record response returned "204.79.197.222" for the observed domain.
2. TCP: The SYN, SYN/ACK, and ACK sequence indicates that the TCP handshake completed.
3. TLS: A TLS Client Hello containing the SNI "fp.msedge.net" was observed.
4. Limitation: A subsequent Hello Retry Request was observed. The available evidence does not establish that the full TLS handshake or an application transaction completed successfully.

Important correlation note: The WFP observation involving IPv6 UDP/443 and the Wireshark observation involving IPv4 TCP/443 are different flows. They must not be presented as the same connection. Matching a local IP address alone also does not prove that a particular process generated a specific packet.

---

## 🚨 6. SOC Alert Triage Scenario

Scenario

A SOC alert reports that "chrome.exe" connected to an external IP on port "443". The TCP handshake completed, and the TLS Client Hello contained the SNI "fp.msedge.net". The scenario provides no malicious-reputation or endpoint-compromise evidence.

Investigation Approach

Step 1 — Verify the process

- Check the executable path.
- Verify the digital signature.
- Review the parent process and command-line arguments.
- Look for unexpected process behavior.

Step 2 — Validate network context

- Compare DNS results with the destination IP.
- Review the destination, protocol, port, and connection timing.
- Examine related endpoint and network alerts.
- Investigate any unexpected or inconsistent evidence.

Step 3 — Assess the evidence

- An external IP or port 443 alone does not establish malicious activity.
- A familiar domain or successful TCP handshake alone does not establish benign activity.
- Escalate if additional evidence indicates suspicious behavior.

Final Assessment

Verdict: No confirmed malicious activity based on the available scenario evidence.

The observed metadata may be consistent with ordinary browser activity. However, process verification and surrounding network context are still required.

This is a scenario-based assessment, not a claim that every verification step was completed during the practical exercise. Close the alert only when the investigation meets the organization's closure criteria.

---

## 🧠 7. Quick Revision

Topic| Remember
Event ID 4624| Successful logon
Event ID 4625| Failed logon
Logon type 5| Service logon
Event ID 5156| Network connection permitted
Event ID 5158| Local port bind permitted
UDP/53| Commonly used for DNS
TCP/443| Commonly used for HTTPS over TCP
TCP handshake| SYN → SYN/ACK → ACK
TLS Client Hello| TLS negotiation initiation; not proof of full handshake completion
Event or connection permitted| Does not automatically mean safe
No matching 4625 events returned| Does not prove failed logons never occurred

## 🎤 8. Interview Questions

Q1. What is the difference between Event IDs 4624 and 4625?

- "4624": A logon succeeded.
- "4625": A logon failed.

Q2. What does Windows Event ID 5156 indicate?

Windows Filtering Platform permitted a network connection. Additional context is required to assess whether the activity is benign or suspicious.

Q3. Why does a successful TCP handshake not prove that a connection is safe?

It confirms that the TCP connection was established. It does not establish the destination's trustworthiness or whether the application activity was malicious.

Q4. Why should a SOC analyst correlate DNS, TCP, and TLS evidence?

Correlation helps connect a domain lookup with subsequent network activity, provided the relevant timestamps, addresses, ports, protocols, and other evidence align.

---
## 📸 Evidence

1. [Windows 4624 — Successful Logon Analysis](./Screenshots/01-windows-4624-logon-analysis.png)
2. [Windows 4625 — Failed Logon Query Result](./Screenshots/02-windows-4625-query-result.png)
3. [Event 5156 — Network Analysis](./Screenshots/03-event-5156-network-analysis.png)
4. [Wireshark — DNS Query Analysis](./Screenshots/04-wireshark-dns-query-analysis.png)
5. [Wireshark — DNS Response Analysis](./Screenshots/05-wireshark-dns-response-analysis.png)
6. [DNS → TCP → TLS Correlation](./Screenshots/06-dns-tcp-tls-correlation.png)

---

## ✅ 10. Key Learnings & Summary

- Queried Windows Security events using PowerShell.
- Interpreted successful and failed logon event queries.
- Examined WFP events for permitted network connections and local port binds.
- Investigated DNS query types and responses in Wireshark.
- Correlated DNS resolution with TCP handshake and TLS Client Hello metadata.
- Practised SOC alert triage using evidence and explicit investigation limitations.

SOC Principle: Investigate the process, network context, and related evidence before deciding whether an alert is benign, suspicious, or malicious.
