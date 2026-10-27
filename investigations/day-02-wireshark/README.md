# Day 2: Wireshark

Scheduled: 2026-10-27
Focus: Packet analysis

Status: Completed
Investigation date: October 27, 2026

I used Wireshark to connect the results I saw in Nmap with the packets behind them, then examined how traditional FTP exposes authentication traffic.

## Objectives

- Capture and filter traffic involving my authorized target.
- Recognize ICMP exchanges, TCP connection setup, and SYN-scan behavior.
- Examine readable FTP authentication and understand the limits of capture visibility.

## Environment and scope

I captured on eth0 on my physical Kali Linux workstation while generating traffic to my Metasploitable 2 VM through the Ubuntu gateway. The [lab setup](../../docs/lab-setup.md) describes this path. I did not perform ARP poisoning, capture another household device's session, or conduct a man-in-the-middle attack.

TARGET below replaces the real target address. Raw captures and authentication values remain private.

## Commands and approach

| Command or filter | Purpose |
| --- | --- |
| ping TARGET | Generate ICMP Echo Requests and Replies |
| nc -vz -w 3 TARGET 22 | Make a short TCP connection to the SSH port |
| nmap -sS -p 21,22,80,445 TARGET | Generate a controlled four-port SYN scan |
| ftp TARGET 21 | Generate failed test authentication and a successful anonymous FTP login |
| ip.addr == TARGET | Display packets involving the target without deleting other captured traffic |
| tcp.stream eq 7 | Isolate the observed TCP conversation; stream numbers belong to that capture |

I inspected packet lists and expanded Ethernet, IPv4, ICMP, and TCP details. I compared packet direction, flags, sequence information, source ports, and application responses.

## Evidence and results

| Evidence | Observation | Interpretation |
| --- | --- | --- |
| [D2-01: TCP connection](evidence/screenshots/01-tcp-handshake.png) | SYN, SYN-ACK, ACK, followed by SSH identification and connection-closing packets | TCP established before the application exchange; connecting to port 22 did not authenticate an SSH user |
| [D2-02: SYN scan](evidence/screenshots/02-syn-scan.png) | Probes to 445, 22, 80, and 21 received SYN-ACK replies followed by scanner resets | Packet behavior supports Nmap's four open-port results without a normal completed handshake for those probes |
| [D2-03: FTP authentication](evidence/screenshots/03-ftp-authentication.png) | Readable USER/PASS requests, a rejected login, and a later 230 success response | The FTP control channel exposed authentication input; the anonymous login was accepted |

All three screenshots were collected October 27. Copies are cropped to the relevant packet lists. Source/destination addresses and authentication values are covered with solid redactions; visible packet flags, ports, response codes, and ordering are unchanged. No raw packet bytes or capture files are included.

In the ICMP exercise, ping reported 17 requests and 17 replies with 0% loss. Wireshark displayed 34 matching packets from 464 captured packets after applying the target filter. One reply showed about 7.24 ms response time.

The SSH-port capture used an ephemeral client port and contained FIN and RST packets during closure. Those flags alone do not establish malicious activity.

The four-port scan's probe order differed from the command's port order. It reused one source port across different destination ports, producing distinct endpoint combinations. A separate unanswered ICMP request did not negate the TCP responses.

## Security concepts and findings

**Plaintext FTP authentication — observed.** I could read the submitted authentication values in the capture. The first test login was rejected; anonymous authentication later succeeded. Acceptance did not prove that an email address supplied in the password field was verified. No real credential values are reproduced here.

**TCP behavior — observed.** A full handshake differed from the SYN/SYN-ACK/reset pattern of the scan. TCP conversations are distinguished by source and destination addresses and ports, so the same source port can appear in different conversations.

**Visibility and encryption — learning takeaways.** Capture privileges do not make unrelated switched traffic visible. SSH/SFTP and FTPS protect communications differently from traditional FTP, and encrypted traffic can still expose metadata. I did not demonstrate SSH decryption or an HTTPS interception test.

## Limitations and remediation

The packet analysis concerned my own generated lab traffic. Captures can contain sensitive data, so only selected redacted screenshots are proposed for publication. The saved PCAP files named in the screenshots were not present in the reviewed workspace; these results are supported by notes and screenshots rather than a fresh analysis of raw captures.

For a real service, I would replace plaintext FTP with an appropriate encrypted alternative and limit anonymous access to an explicit need. No server changes or remediation retest were performed. This exercise was packet analysis and authentication testing, not an exploitation exercise.

## Lessons learned

A successful TCP connection is not an authenticated session. An RST is not automatically an attack, and a single unanswered packet does not explain an entire exchange. Comparing packet sequences with the command that generated them made the scan results easier to understand.

The most useful discovery was seeing FTP authentication as actual readable packets. It turned the difference between encrypted and unencrypted protocols into something I could verify directly.
