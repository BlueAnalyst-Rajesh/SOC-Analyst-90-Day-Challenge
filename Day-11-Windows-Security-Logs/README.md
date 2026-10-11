# Day 11 — Windows Security Logs & Network Investigation

## 🎯 Objective

Investigate Windows Security events and network activity using PowerShell and Wireshark. Understand successful and failed logon events, Windows Filtering Platform (WFP) events, DNS resolution, and TCP/TLS connection behavior.

## 🧰 Tools Used

- Windows PowerShell — Windows event log querying and filtering
- Windows Event Log — Security event investigation
- Wireshark — DNS, TCP, and TLS packet analysis

## 🔗 Previous Knowledge Applied

This investigation builds on networking and packet-analysis concepts learned in previous SOC challenge days.

- TCP/IP and Ports: Used knowledge of TCP/443, UDP/53, and network endpoints to interpret connection details.
- Wireshark Display Filters: Applied the "dns" filter to inspect DNS queries and responses.
- DNS Investigation: Connected DNS query types and returned IP addresses with subsequent network observations.
- TCP Three-Way Handshake: Identified SYN, SYN/ACK, and ACK packets to verify TCP connection establishment.
- Evidence-Based Analysis: Applied the principle that a single event, IP address, port, or domain is insufficient to confirm malicious activity.

These concepts helped connect Windows event-log investigation with packet-level network analysis while keeping conclusions limited to the available evidence.


## 1. Windows Security Event Investigation

Event ID 4624 — Successful Logon

Used PowerShell to retrieve recent successful and failed logon events.

Get-WinEvent -FilterHashtable @{
    LogName = 'Security'
    Id = 4624, 4625
} -MaxEvents 10 |
Select-Object TimeCreated, Id, ProviderName, Message

Observed results:

- The retrieved matching events were Event ID "4624".
- The events were recorded by "Microsoft-Windows-Security-Auditing".
- One examined event showed:
  - Target account: "SYSTEM"
  - Logon type: "5"
  - Process: "C:\Windows\System32\services.exe"
  - Source IP: "-"

Analysis: Logon type 5 represents a service logon. The observed account and process are consistent with normal Windows service activity, but the event alone is not sufficient to prove that the activity is completely safe.

Event ID 4625 — Failed Logon

Get-WinEvent -FilterHashtable @{
    LogName = 'Security'
    Id = 4625
} -MaxEvents 10 |
Select-Object TimeCreated, Id, Message

Observed result: No events matching the query were returned.

Analysis: This means the query did not find matching failed-logon events in the available results. It does not prove that failed logons have never occurred or rule out brute-force activity.

## 2. Security Event ID Summary

Used PowerShell to group the most recent 30 Security log events by event ID.

Get-WinEvent -LogName Security -MaxEvents 30 |
Group-Object Id |
Sort-Object Count -Descending |
Select-Object Name, Count

Observed results:

Event ID| Count in retrieved 30 events
5158| 22
5156| 8

- 5158: Windows Filtering Platform permitted a bind to a local port.
- 5156: Windows Filtering Platform permitted a network connection.

Analysis: These counts apply only to the 30 events retrieved by the command, not to the entire Security log. Permitted network activity is not automatically malicious or automatically safe.

## 3. Windows Filtering Platform — Event ID 5156

Examined permitted network connections and their associated application, source, destination, ports, and protocol.

Observed network activity

Example A — UDP/443

- Application: "msedgewebview2.exe"
- Destination IPv6: "2404:6800:4000:1025::54"
- Destination port: "443"
- Protocol: "17" (UDP)

UDP/443 may be used by QUIC-based traffic, but this event alone does not confirm that QUIC or HTTP/3 was used, nor does it establish whether the connection was benign.

Example B — DNS traffic

- Application: "chrome.exe"
- Source IP: "10.202.17.237"
- Destination DNS server: "10.202.17.55"
- Destination port: "53"
- Protocol: "17" (UDP)

Additional DNS-related network events showed traffic to the same DNS server using different source ports.

Analysis: Event ID 5156 records a permitted network connection. It does not, by itself, reveal the DNS query name or establish the safety of the connection.

## 4. Wireshark — DNS Investigation

Applied the display filter:

dns

Observed details:

- Client IP: "10.202.17.237"
- DNS server: "10.202.17.55"
- Transport: UDP
- DNS query name: "fp.msedge.net"
- Query types observed: HTTPS, AAAA, and A
- The A-record response included the IPv4 address "204.79.197.222".
- The observed DNS response reported no error.

Analysis: DNS analysis identified the queried domain and its returned IPv4 address. A domain's apparent association with a known service is useful context, but domain reputation or familiarity alone does not prove that a connection is safe.

## 5. DNS → TCP → TLS Correlation

Investigated the packet sequence associated with "fp.msedge.net".

Observation| Evidence
DNS resolution| "fp.msedge.net" resolved to "204.79.197.222" in the observed A-record response
TCP source| "10.202.17.237:49956"
TCP destination| "204.79.197.222:443"
TCP handshake| SYN → SYN/ACK → ACK
TLS metadata| TLS 1.3 Client Hello
Server Name Indication| "fp.msedge.net"

The TCP handshake indicates that the TCP connection was established. The TLS Client Hello and SNI provide evidence of a TLS connection attempt for the observed server name.

A subsequent Hello Retry Request was observed. The available packet evidence does not establish that the full TLS handshake or an application transaction completed successfully.

Important correlation note: The WFP event involving IPv6 UDP/443 and the Wireshark flow involving IPv4 TCP/443 are different observed flows. They should not be represented as the same connection. Likewise, matching a local IP address alone is insufficient to prove that a particular process generated a specific packet.

## 6. SOC Alert Triage Scenario

Scenario: "chrome.exe" connects to an external IP on port 443. The TCP handshake completes, and the TLS SNI is "fp.msedge.net". No malicious reputation or endpoint-compromise evidence is available in the scenario.

Investigation steps

1. Verify the process
   
   - Check the executable path.
   - Verify the digital signature.
   - Review the parent process and command-line arguments.
   - Look for unexpected process behavior.

2. Validate network context
   
   - Compare DNS results with the destination IP.
   - Review the destination, protocol, port, and connection timing.
   - Examine related endpoint and network alerts.
   - Investigate any unexpected or inconsistent evidence.

3. Make an evidence-based assessment
   
   - Do not classify a connection as malicious based only on an external IP or port 443.
   - Do not classify it as safe based only on a familiar domain or successful TCP handshake.
   - Escalate if additional evidence indicates suspicious behavior.

Final assessment

Based on the scenario's available evidence, malicious activity is not confirmed. The observed metadata may be consistent with ordinary browser activity, but process verification and surrounding context are still required.

Closure should follow the organization's investigation and alert-closure criteria.

## 7. Key Learnings

- Interpreted Windows Security Event IDs "4624" and "4625".
- Learned that logon type "5" represents a service logon.
- Used PowerShell to query and group Windows Security events.
- Interpreted WFP Event IDs "5156" and "5158".
- Examined DNS queries and A, AAAA, and HTTPS query types in Wireshark.
- Correlated DNS resolution with TCP handshake and TLS Client Hello metadata.
- Distinguished a permitted network connection from proof of benign activity.
- Practised evidence-based alert triage without making unsupported conclusions.

## 📸 Evidence

1. "Windows 4624 — Successful Logon Analysis" (./screenshots/01-windows-4624-logon-analysis.png)
2. "Windows 4625 — Failed Logon Query Result" (./screenshots/02-windows-4625-query-result.png)
3. "Windows Filtering Platform — Event ID 5156 Network Analysis" (./screenshots/03-event-5156-network-analysis.png)
4. "Wireshark — DNS Query Analysis" (./screenshots/04-wireshark-dns-query-analysis.png)
5. "Wireshark — DNS Response Analysis" (./screenshots/05-wireshark-dns-response-analysis.png)
6. "DNS → TCP Handshake → TLS Client Hello / SNI Correlation" (./screenshots/06-dns-tcp-tls-correlation.png)

Replace these suggested paths with your actual screenshot filenames. Remove any link for which you do not have matching evidence.

## ✅ Day 11 Summary

Day 11 focused on Windows Security log investigation and network traffic analysis using PowerShell and Wireshark. The practical work demonstrated how to examine logon events, interpret permitted network connections, investigate DNS responses, and follow TCP/TLS metadata while keeping conclusions limited to the evidence available.

SOC principle: Investigate the process, network context, and related evidence before deciding whether an alert is benign, suspicious, or malicious.
