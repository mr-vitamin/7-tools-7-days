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

The Metasploit Framework 6.5.7 exercise ran on my physical Kali workstation against Metasploitable 2 through the restricted Ubuntu gateway. Testing was limited to service enumeration and banner validation on TCP/21 of the verified lab target. Exploitation was outside the defined scope. My [lab setup](../../docs/lab-setup.md) explains the network restrictions.

TARGET replaces the private address. The version module uses a connection initiated from Kali. It requires no reverse connection, account credentials, or target configuration change.

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

**FTP service identification - independently corroborated.** Both Metasploit and a separate connection received the vsFTPd 2.3.4 identification. This is stronger evidence of what the service advertises than a port-number label alone.

**FTP backdoor - unverified.** A matching version string does not prove that the installed executable contains a backdoor. No exploit trigger, command execution, or privilege change was tested. The earlier Day 1 hypothesis remains unresolved.

## Limitations and remediation

The service-enumeration and independent-validation objectives were fulfilled. Banner validation does not establish binary provenance, patch state, vulnerability presence, or exploitability. Exploit execution, authentication testing, and target file operations were outside this investigation; none was performed.

For a real service, I would verify the installed package and its source, assess the actual build, and replace unsupported or untrusted software. The earlier plaintext FTP observation also supports moving required file transfers to an encrypted alternative. A remediation retest would check the replacement's authentication, transport protection, and required operations. No remediation or remediation retest was performed.

## Lessons learned

The module determines which question a result can answer. Here, the auxiliary module identified the advertised service, and the direct greeting corroborated it.

For a future service check, I would keep the selected host and port, the module result, and the independent protocol response together. A vulnerability claim would need evidence beyond the advertised version.
