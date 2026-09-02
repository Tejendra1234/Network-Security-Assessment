# Network Security Assessment and Traffic Analysis Using Wireshark and Windows Firewall

## Overview

This project demonstrates a controlled network security assessment and traffic analysis lab using Kali Linux and a Windows VM.

The lab was used to establish connectivity between the two systems, verify network services, capture and analyze HTTP traffic with Wireshark, and evaluate Windows Firewall controls by blocking HTTP and FTP traffic.

The project focuses on understanding network communication, TCP/IP traffic, service ports, packet analysis, and firewall-based traffic control in a controlled virtual environment.

---

## Objectives

* Establish communication between Kali Linux and a Windows VM.
* Identify the IP addresses of the lab systems.
* Verify HTTP and FTP services running on the Windows system.
* Capture and analyze HTTP traffic using Wireshark.
* Identify the TCP three-way handshake.
* Analyze source and destination IP addresses and TCP ports.
* Create Windows Firewall inbound rules to block HTTP traffic on TCP port 80.
* Create Windows Firewall inbound rules to block FTP traffic on TCP port 21.
* Perform before-and-after connectivity testing to evaluate the firewall rules.

---

## Lab Architecture

```text
                 NAT Network
                     |
          +----------+----------+
          |                     |
          |                     |
     Kali Linux              Windows VM
      10.0.2.3                10.0.2.5
          |                     |
          |                     |
     Wireshark              Windows Firewall
          |                     |
          |              +------+------+
          |              |             |
          |          Apache HTTP    FileZilla
          |            TCP/80        TCP/21
          |              |             |
          +--------------+-------------+
                 Test Traffic
```

### Lab Systems

| System     | IP Address | Purpose                                     |
| ---------- | ---------- | ------------------------------------------- |
| Kali Linux | `10.0.2.3` | Traffic generation and Wireshark analysis   |
| Windows VM | `10.0.2.5` | Target system hosting HTTP and FTP services |

### Services Tested

| Service       | Protocol | Port | Test                               |
| ------------- | -------- | ---: | ---------------------------------- |
| Apache HTTP   | TCP      |   80 | Connectivity and firewall blocking |
| FileZilla FTP | TCP      |   21 | Connectivity and firewall blocking |

---

## Tools Used

* Kali Linux
* Windows VM
* Wireshark
* Windows Firewall with Advanced Security
* Apache HTTP Server
* FileZilla FTP Server

---

# Methodology

## 1. Network Baseline

The first stage established the network configuration of both virtual machines.

The Windows VM was verified with `ipconfig`, and the Kali Linux system was checked using `ip a`.

The configured addresses used during the assessment were:

```text
Kali Linux  : 10.0.2.3
Windows VM  : 10.0.2.5
```

Connectivity was then tested between the systems.

Windows was able to ping the Kali Linux system successfully.

The reverse ICMP test from Kali Linux to Windows did not receive responses and showed 100% packet loss. This behavior was recorded as part of the baseline rather than treating it as a successful connectivity test.

Despite the ICMP behavior, TCP-based application services were reachable between the systems.

---

## 2. Service Verification

The Windows system was checked for the services used in the assessment.

The following services were verified as listening:

```text
TCP/80  - Apache HTTP Server
TCP/21  - FileZilla FTP Server
```

From Kali Linux, the HTTP service hosted on the Windows VM was accessed using:

```text
http://10.0.2.5
```

The Apache/XAMPP page was successfully displayed.

The FTP service was also tested from Kali Linux using:

```text
ftp 10.0.2.5
```

The connection reached the FileZilla FTP service and displayed its server response and username prompt.

The FTP authentication process was not tested because credentials were not configured as part of this lab.

---

# 3. HTTP Traffic Analysis with Wireshark

Wireshark was used to capture network traffic generated during HTTP communication between Kali Linux and the Windows VM.

A display filter was applied to isolate TCP port 80 traffic:

```text
tcp.port == 80
```

The filtered traffic showed communication between:

```text
Source      : 10.0.2.3
Destination : 10.0.2.5
Destination Port : 80
```

### TCP Three-Way Handshake

The captured traffic showed the TCP connection establishment sequence:

```text
SYN
 ↓
SYN/ACK
 ↓
ACK
```

This demonstrated the TCP three-way handshake occurring before HTTP communication.

### HTTP Traffic

After the TCP connection was established, HTTP traffic was observed in the capture.

The packet details were inspected to identify:

* Source IP address
* Destination IP address
* Source TCP port
* Destination TCP port
* HTTP protocol information

The packet details showed HTTP communication involving TCP port 80.

---

# 4. Windows Firewall — Blocking HTTP

The next experiment evaluated Windows Firewall as a traffic control mechanism.

Windows Firewall with Advanced Security was opened and an inbound port rule was created.

The rule was configured to:

```text
Protocol : TCP
Port     : 80
Action   : Block
```

The rule was named:

```text
Block HTTP TCP 80
```

After the rule was enabled, Kali Linux attempted to access:

```text
http://10.0.2.5
```

The HTTP page was no longer accessible.

### Result

```text
Before firewall rule : HTTP accessible
After firewall rule  : HTTP blocked
```

This demonstrated that the Windows Firewall inbound rule was effective in restricting access to the HTTP service on TCP port 80.

---

# 5. Windows Firewall — Blocking FTP

A second firewall experiment was performed against the FileZilla FTP service.

An inbound port rule was created with:

```text
Protocol : TCP
Port     : 21
Action   : Block
```

The rule was named:

```text
Block FTP TCP 21
```

Before applying the rule, Kali Linux was able to connect to the FileZilla FTP service.

After enabling the firewall rule, a new FTP connection attempt from Kali Linux no longer reached the previous FTP server response.

### Result

```text
Before firewall rule : FTP accessible
After firewall rule  : FTP blocked
```

This demonstrated that Windows Firewall could restrict access to the FTP service by blocking TCP port 21.

---

# Key Findings

| Test                    | Before Firewall  | After Firewall | Result                        |
| ----------------------- | ---------------- | -------------- | ----------------------------- |
| HTTP TCP/80             | Accessible       | Blocked        | Firewall rule effective       |
| FTP TCP/21              | Accessible       | Blocked        | Firewall rule effective       |
| HTTP packet capture     | Observable       | —              | Wireshark captured traffic    |
| TCP three-way handshake | Observable       | —              | SYN/SYN-ACK/ACK identified    |
| Kali → Windows ICMP     | 100% packet loss | —              | Recorded as baseline behavior |

---

# Security Observations

### 1. Network traffic can be observed at packet level

Wireshark provided visibility into network communication, including source and destination addresses, TCP ports, TCP connection establishment, and HTTP traffic.

### 2. Service ports are important security control points

The HTTP and FTP services were associated with TCP ports 80 and 21 respectively. These ports were used to create specific firewall controls.

### 3. Firewall rules can restrict service access

The HTTP and FTP experiments demonstrated that Windows Firewall inbound rules can be used to block access to specific TCP ports.

### 4. Before-and-after testing provides evidence of control effectiveness

Testing service accessibility before and after applying firewall rules provided a direct way to evaluate whether the configured controls produced the expected result.

---

# Limitations

This project was conducted in a controlled virtual lab environment.

The assessment was limited to:

* Network connectivity verification
* HTTP traffic capture and analysis
* TCP handshake observation
* HTTP service blocking
* FTP service blocking

The project did not include:

* Vulnerability exploitation
* Firewall bypass techniques
* Credential attacks
* FTP authentication testing
* Full penetration testing
* Production network testing

Therefore, this project should be considered a **network security and traffic analysis lab**, not a complete penetration test or VAPT assessment.

---

# Skills Demonstrated

* Basic TCP/IP networking
* IP addressing
* TCP and UDP concepts
* Network service and port identification
* Wireshark packet capture
* Wireshark traffic filtering
* TCP three-way handshake analysis
* HTTP traffic analysis
* Windows Firewall configuration
* Inbound firewall rule creation
* Before-and-after security testing
* Security evidence documentation

---

# Project Evidence

## Architecture

* Windows IP configuration
* Kali Linux IP configuration

## Baseline

* Windows-to-Kali connectivity
* Kali-to-Windows ICMP behavior
* HTTP and FTP service verification
* HTTP service access from Kali
* FTP service connection from Kali

## Wireshark

* HTTP traffic capture
* TCP port 80 filtering
* TCP three-way handshake
* HTTP packet details

## Firewall

### HTTP

* HTTP TCP/80 blocking rule
* HTTP access blocked after firewall configuration

### FTP

* FTP TCP/21 blocking rule
* FTP access blocked after firewall configuration

---

# Conclusion

This project provided hands-on experience with network traffic analysis and host-based firewall controls in a controlled Kali Linux and Windows environment.

Wireshark was used to observe and analyze HTTP communication and the TCP connection establishment process. Windows Firewall was then used to restrict access to the HTTP and FTP services through inbound TCP port rules.

The before-and-after testing demonstrated how firewall controls can change service accessibility and provided practical evidence of the configured security controls.
