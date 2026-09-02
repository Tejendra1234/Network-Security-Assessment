# Security Assessment Findings

## Assessment Summary

A controlled network security assessment was performed between a Kali Linux VM and a Windows VM.

The assessment focused on HTTP and FTP service accessibility, HTTP traffic analysis using Wireshark, and Windows Firewall controls for restricting access to TCP ports 80 and 21.

---

## Lab Configuration

| Component        | Details                                 |
| ---------------- | --------------------------------------- |
| Kali Linux       | `10.0.2.3`                              |
| Windows VM       | `10.0.2.5`                              |
| HTTP Service     | Apache — TCP/80                         |
| FTP Service      | FileZilla — TCP/21                      |
| Traffic Analysis | Wireshark                               |
| Firewall         | Windows Firewall with Advanced Security |

---

## Finding 1 — HTTP Service Accessible Before Firewall Control

### Observation

The Apache HTTP service running on the Windows VM was reachable from Kali Linux using the Windows VM's IP address.

```text
http://10.0.2.5
```

### Evidence

* Windows `netstat` showed TCP port 80 listening.
* Kali Linux successfully accessed the HTTP service.

### Assessment

The HTTP service was reachable from the Kali Linux system before the specific firewall blocking rule was enabled.

### Control Applied

A Windows Firewall inbound rule was created:

```text
Rule Name : Block HTTP TCP 80
Protocol  : TCP
Port      : 80
Action    : Block
```

### Result

After enabling the rule, the HTTP service was no longer accessible from Kali Linux.

### Conclusion

The Windows Firewall rule successfully restricted access to the HTTP service on TCP port 80.

---

## Finding 2 — FTP Service Accessible Before Firewall Control

### Observation

The FileZilla FTP service running on the Windows VM was reachable from Kali Linux.

The connection was initiated using:

```text
ftp 10.0.2.5
```

The connection reached the FileZilla server and displayed the server response and username prompt.

### Evidence

* Windows `netstat` showed TCP port 21 listening.
* Kali Linux successfully reached the FileZilla FTP service.

### Assessment

The FTP service was reachable from Kali Linux before the specific firewall blocking rule was enabled.

### Control Applied

A Windows Firewall inbound rule was created:

```text
Rule Name : Block FTP TCP 21
Protocol  : TCP
Port      : 21
Action    : Block
```

### Result

After enabling the rule, the FTP connection attempt from Kali Linux no longer reached the previous FTP server response.

### Conclusion

The Windows Firewall rule successfully restricted access to the FTP service on TCP port 21.

---

## Finding 3 — HTTP Traffic Observable with Wireshark

### Observation

HTTP communication between Kali Linux and the Windows VM was captured using Wireshark.

The following display filter was used:

```text
tcp.port == 80
```

The filtered traffic showed communication between:

```text
Source      : 10.0.2.3
Destination : 10.0.2.5
Destination Port : 80
```

### Analysis

The capture showed the TCP connection establishment sequence:

```text
SYN
 ↓
SYN/ACK
 ↓
ACK
```

HTTP traffic was then observed after the TCP connection was established.

Packet details were inspected to identify source and destination addresses, TCP ports, and HTTP protocol information.

### Conclusion

Wireshark provided packet-level visibility into the HTTP communication and allowed the TCP connection establishment process to be observed.

---

## Finding 4 — Asymmetric ICMP Behavior

### Observation

The baseline connectivity tests produced different results depending on the direction.

```text
Windows → Kali
Successful

Kali → Windows
100% packet loss
```

### Assessment

The reverse ICMP test did not receive responses from the Windows VM.

However, HTTP and FTP services were successfully reached from Kali Linux during baseline testing.

### Conclusion

The ICMP behavior was recorded as a baseline observation and was not treated as evidence that the TCP-based services were inaccessible.

---

# Overall Results

| Test                | Baseline   | Security Control           | Final Result               |
| ------------------- | ---------- | -------------------------- | -------------------------- |
| HTTP TCP/80         | Accessible | Block TCP/80               | Blocked                    |
| FTP TCP/21          | Accessible | Block TCP/21               | Blocked                    |
| HTTP traffic        | Observable | Wireshark analysis         | TCP/HTTP traffic analyzed  |
| TCP handshake       | Observable | Wireshark analysis         | SYN/SYN-ACK/ACK identified |
| Kali → Windows ICMP | 100% loss  | No additional modification | Recorded as baseline       |

---

# Security Impact

The experiments demonstrated that services exposed through specific TCP ports can be restricted using Windows Firewall inbound rules.

Blocking TCP port 80 prevented access to the HTTP service, while blocking TCP port 21 prevented access to the FTP service.

The Wireshark analysis demonstrated how network communication can be examined at packet level to identify addressing, TCP ports, connection establishment, and HTTP traffic.

---

# Limitations

This assessment was performed in a controlled virtual lab.

The testing did not include:

* Vulnerability exploitation
* Firewall bypass
* Credential attacks
* FTP authentication testing
* Full penetration testing
* Production network testing

The FTP test was limited to service connectivity. The username prompt was reached during the baseline test, but authentication was not performed.

Therefore, these findings should be interpreted as observations from a controlled network security and traffic analysis lab rather than a complete VAPT assessment.

---

# Final Assessment

The assessment successfully demonstrated:

1. Verification of HTTP and FTP service availability.
2. Packet-level analysis of HTTP traffic using Wireshark.
3. Identification of the TCP three-way handshake.
4. Creation of Windows Firewall inbound rules.
5. Successful restriction of HTTP traffic on TCP port 80.
6. Successful restriction of FTP traffic on TCP port 21.
7. Before-and-after testing to verify the effect of the firewall controls.

The results provide practical evidence of network traffic analysis and host-based firewall configuration in a controlled virtual environment.
