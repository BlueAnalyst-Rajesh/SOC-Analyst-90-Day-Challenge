# Day 11 — Windows Security Logs & Network Analysis

## 🎯 Objective
Investigate Windows logon events and network activity using PowerShell and Wireshark. Practise evidence-based SOC analysis by examining Windows Security logs, DNS resolution, TCP connections, and TLS metadata.
## 🧰 Tools Used
 * Windows PowerShell
 * Windows Security Logs
 * Windows Filtering Platform (WFP)
 * Wireshark
## 1. Windows Security Log Analysis
Important Event IDs
| Event ID | Meaning |
|---|---|
| 4624 | Successful logon |
| 4625 | Failed logon |
| 5156 | Network connection permitted |
| 5158 | Local port bind permitted |
Practical 1 — Find Successful and Failed Logons
Command:
```
Get-WinEvent -FilterHashtable @{ LogName = 'Security' Id = 4624, 4625 } -MaxEvents 10 | Select-Object TimeCreated, Id, ProviderName, Message
```
Observed result:
All 10 returned events had Event ID 4624.
One inspected event showed the account SYSTEM.
 * Logon Type: 5
 * Process: C:\Windows\System32\services.exe
 * Source IP: -
> Analysis: These details are consistent with a Windows service logon. This event alone does not prove that the system is completely safe.
> 
Practical 2 — Check Failed Logons
Command:
```
Get-WinEvent -FilterHashtable @{ LogName = 'Security' Id = 4625 } -MaxEvents 10 | Select-Object TimeCreated, Id, Message
```
Observed result: No events matched the query.
> Analysis: No matching Event ID 4625 events were returned by this query. This does not prove that failed logons have never occurred or rule out brute-force activity outside the available logs or time range.
> 
Logon Types — Quick Reference
| Logon Type | Meaning |
|---|---|
| 2 | Interactive logon |
| 3 | Network logon |
| 5 | Service logon |
| 10 | Remote Interactive logon, commonly RDP |

## 2. Security Event Summary & Network Analysis
Practical 3 — Group Recent Security Events
Command:
```
Get-WinEvent -LogName Security -MaxEvents 30 | Group-Object Id | Sort-Object Count -Descending | Select-Object Name, Count
```
Observed result:
| Event ID | Count |
|---|---|
| 5158 | 22 |
| 5156 | 8 |
> Analysis: These counts represent only the 30 most recent retrieved Security events, not the total number of events on the system.
> 
Practical 4 — Inspect Event ID 5156
Event ID 5156 records a network connection permitted by Windows Filtering Platform.
Observed network details:
| Field | Observation |
|---|---|
| Application | msedgewebview2.exe |
| Source IPv6 | 2409:408d:1d8d:ee18:bda4:be7b:4c38:c566 |
| Source Port | 53229 |
| Destination IPv6 | 2404:6800:4000:1025::54 |
| Destination Port | 443 |
| Protocol | UDP (17) |
A separate DNS-related observation showed:
 * Application: chrome.exe
 * Client IP: 10.202.17.237
 * DNS server: 10.202.17.55
 * Destination port: 53/UDP
> Analysis: Event ID 5156 indicates that a connection was permitted. It does not, by itself, establish whether the traffic was benign or malicious. The observations above represent different flows and should not automatically be treated as one connection.
> 
## 3. Wireshark DNS Investigation
Practical 5 — Analyse DNS Queries and Responses
Display filter:
dns

Observed details:
| Field | Observation |
|---|---|
| Client IP | 10.202.17.237 |
| DNS Server | 10.202.17.55 |
| Domain | fp.msedge.net |
| Query Types | HTTPS, AAAA, A |
| Final IPv4 Answer | 204.79.197.222 |
| DNS Response | No error observed in the inspected response |
Frames 10–12 contained DNS responses. Frame 12 contained the A-record response, including a CNAME chain before the final IPv4 answer.
> Analysis: DNS investigation helps identify requested domains and returned IP addresses. A successful DNS response alone does not establish whether subsequent network activity is safe or malicious.
> 

## 4. DNS → TCP → TLS Correlation
Practical 6 — Trace the Network Activity
Observed sequence:
> Domain: fp.msedge.net \rightarrow DNS Response: 204.79.197.222 \rightarrow TCP Connection to Port 443 \rightarrow SYN \rightarrow SYN/ACK \rightarrow ACK \rightarrow TLS 1.3 Client Hello \rightarrow SNI: fp.msedge.net
> 
Connection details:
| Field | Observation |
|---|---|
| Source | 10.202.17.237:49956 |
| Destination | 204.79.197.222:443 |
| TCP Handshake | Completed |
| TLS Client Hello | Frame 19 |
| TLS SNI | fp.msedge.net |
| Additional Observation | Hello Retry Request reported in Frame 24 |
> Analysis:
>  * SYN \rightarrow SYN/ACK \rightarrow ACK confirms that the TCP handshake completed.
>  * The TLS Client Hello and SNI were observed.
>  * The available evidence does not confirm that the full TLS handshake or application transaction completed.
>  * The Windows Filtering Platform observation involving an IPv6 destination and UDP/443 is a different flow from this IPv4 TCP/443 connection.
> Important SOC lesson: Correlate events carefully. Similar domains, applications, or local IP addresses do not automatically prove that separate observations belong to the same connection.
> 
## 5. SOC Scenario & Assessment
Scenario
chrome.exe connects to an external IP over port 443. The TCP handshake completes, and the TLS SNI is fp.msedge.net. No malicious reputation or endpoint-compromise evidence is available in the scenario.
Investigation Checklist
 * [ ] Verify the executable path and digital signature.
 * [ ] Review the parent process and command line.
 * [ ] Compare the DNS answer, destination IP, and TLS SNI.
 * [ ] Check timestamps and surrounding network activity.
 * [ ] Review related endpoint alerts and other suspicious indicators.
 * [ ] Escalate if additional evidence suggests malicious activity.
Final Assessment
No confirmed malicious activity based on the available evidence.
The observed activity may be consistent with ordinary browser traffic, but that does not establish that it is definitely safe. Validate the process and surrounding context before closing a real SOC alert.
## 🔁 Quick Revision Notes
| Topic | Remember |
|---|---|
| Event 4624 | Successful logon |
| Event 4625 | Failed logon |
| Logon Type 5 | Service logon |
| Event 5156 | Network connection permitted |
| Event 5158 | Local port bind permitted |
| Wireshark DNS filter | dns |
| TCP handshake | SYN \rightarrow SYN/ACK \rightarrow ACK |
| TLS SNI | Indicates the hostname requested in the TLS Client Hello |
| No 4625 results | Does not prove that failed logons never occurred |
| One indicator | Is not enough to conclude safe or malicious |
Common Mistakes to Avoid
 * Do not treat every external connection as malicious.
 * Do not assume permitted traffic is safe just because Event ID 5156 exists.
 * Do not conclude that HTTPS succeeded merely because TCP connected.
 * Do not claim that separate network flows are the same connection without matching evidence.
 * Do not label a system compromised without sufficient supporting evidence.
## Self-Check
Before reviewing an alert, ask yourself:
 * What exactly does the event or packet prove?
 * Which details are still unverified?
 * Do the timestamps, process, IP address, port, and domain correlate?
 * Is there enough evidence to close the alert, or should it be escalated?
## 🎤 Interview Practice
Q1. What is the difference between Event IDs 4624 and 4625?
4624 records a successful logon, while 4625 records a failed logon.
Q2. What does Event ID 5156 indicate?
Windows Filtering Platform permitted a network connection. It does not automatically mean the traffic is safe.
Q3. Does a completed TCP handshake mean HTTPS succeeded?
No. It confirms the TCP connection was established. Further TLS and application-level evidence is needed.
Q4. How would you investigate a suspicious browser connection?
Verify the executable, signature, parent process, command line, destination IP, DNS activity, TLS metadata, timestamps, and related security alerts.
## 🛠️ Skills Practised
 * Filtering Windows Security logs using PowerShell
 * Interpreting logon Event IDs and logon types
 * Inspecting Windows Filtering Platform events
 * Analysing DNS queries and responses in Wireshark
 * Identifying TCP handshake behaviour
 * Examining TLS Client Hello and SNI metadata
 * Correlating evidence without making unsupported conclusions
 * Applying evidence-based SOC alert assessment

## 📸 Evidence

1. [Windows 4624 — Successful Logon Analysis](./Screenshots/01-windows-4624-logon-analysis.png)
2. [Windows 4625 — Failed Logon Query Result](./Screenshots/02-windows-4625-query-result.png)
3. [Event 5156 — Network Analysis](./Screenshots/03-event-5156-network-analysis.png)
4. [Wireshark — DNS Query Analysis](./Screenshots/04-wireshark-dns-query-analysis.png)
5. [Wireshark — DNS Response Analysis](./Screenshots/05-wireshark-dns-response-analysis.png)
6. [DNS → TCP → TLS Correlation](./Screenshots/06-dns-tcp-tls-correlation.png)
                  
## ✅ Conclusion
Day 11 provided practical experience in Windows Security log analysis, PowerShell event filtering, WFP network-event inspection, and Wireshark DNS/TCP/TLS investigation.
Key takeaway: A SOC analyst must distinguish observed facts from assumptions, correlate evidence carefully, document investigation limits, and make conclusions supported by the available data.
