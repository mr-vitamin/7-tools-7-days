# Day 5: Metasploit

Scheduled: 2026-10-30
Focus: Service enumeration and FTP version validation

Status: Completed

Metasploit's FTP version module and a separate FTP connection returned the same service identification. My focus was on configuring the module for one target and independently checking its advertised version.

## Objectives

- Configure an auxiliary module for one authorized target.
- Compare module output with an independent protocol response.
- Understand the difference between service identification and confirmed exploitability.

## Environment and scope

The Metasploit Framework 6.5.7 exercise ran on my physical Kali workstation against Metasploitable 2 through the restricted Ubuntu gateway. Testing covered service enumeration and banner validation on TCP/21 of the verified lab target. My [lab setup](../../docs/lab-setup.md) explains the network restrictions.

TARGET replaces the private address. The version module uses a connection initiated from Kali and reads the service greeting.

## Commands and approach

| Command | Purpose |
| --- | --- |
| version | Record the framework and console versions |
| use auxiliary/scanner/ftp/ftp_version | Select the FTP banner module |
| set RHOSTS TARGET; set RPORT 21 | Limit the module to the lab FTP service |
| set THREADS 1; set ConnectTimeout 5 | Use one worker and a bounded connection timeout |
| show options | Check the selected host, port, and worker count before execution |
| run | Collect the service banner |
| sessions -l | Check whether any session existed after the auxiliary module |
| TCP connect to TARGET:21; read greeting; send QUIT | Compare the module identification with the actual FTP greeting and close normally |

The final row describes a separate Python socket check. It read the FTP greeting and submitted QUIT without attempting authentication. The auxiliary module and direct check both used connections initiated toward the target.

## Evidence and results

| Evidence | Observation | What it establishes |
| --- | --- | --- |
| [D5-01: FTP version module](evidence/sanitized-results/01-ftp-version-module.txt) | The module reported FTP Banner: vsFTPd 2.3.4, completed one host, and listed no active sessions | Metasploit retrieved a banner from the selected service; no interactive session was opened |
| [D5-02: Direct FTP greeting](evidence/sanitized-results/02-direct-banner.txt) | The greeting was 220 (vsFTPd 2.3.4); QUIT received 221 Goodbye | The independently observed protocol greeting agreed with the module output |

These text excerpts replace the target address with TARGET and omit terminal formatting, local paths, and actual collection timestamps. The FTP replies, selected settings, and module result are retained.

## Findings

**FTP service identification - independently corroborated.** Both Metasploit and a separate connection received the vsFTPd 2.3.4 identification. The matching greetings corroborate the advertised service version.

**FTP backdoor - unverified.** A matching version string does not prove that the installed executable contains a backdoor. The FTP-backdoor hypothesis from the Nmap investigation remains unverified; service greetings alone cannot resolve it.

## Limitations and remediation

The investigation identified the advertised service and corroborated it independently. Binary provenance, patch state, authentication, and exploitability were outside this service-validation scope.

For a real service, I would verify the installed package and its source, assess the actual build, and replace unsupported or untrusted software. The Wireshark plaintext-FTP finding also supports an encrypted transfer alternative. A remediation retest would check authentication, transport protection, and required file operations; none was performed here.

## Lessons learned

The options listing made the selected host, port, and worker count explicit before the module ran. Its output then had a simple independent check: read the FTP greeting directly and close the connection with QUIT.

The empty session list was consistent with the auxiliary module's purpose. My takeaway is to choose a module around the question being asked. A successful service-enumeration run answers what the service advertises; a vulnerability finding would need different evidence.
