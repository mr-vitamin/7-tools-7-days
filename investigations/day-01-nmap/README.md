# Day 1: Nmap

Scheduled: 2026-10-26
Focus: Network reconnaissance

Status: Completed
Investigation dates: October 26–27, 2026

I started with Nmap reconnaissance, then checked selected services directly. My Day 1 follow-up work continued on October 27 before the Wireshark exercise.

## Objectives

- Compare basic, TCP connect, SYN, and targeted service scans.
- Turn scan results into questions I could test.
- Separate a service identification from a confirmed security finding.

## Environment and scope

I used my physical Kali Linux workstation to assess my Metasploitable 2 VM through the restricted Ubuntu gateway. Testing was limited to that authorized lab target. The [lab setup](../../docs/lab-setup.md) describes the network and its limits.

The commands below replace the real address with TARGET. They are sanitized records of the commands used, not additional tests.

## Commands and approach

| Command | Why I used it |
| --- | --- |
| nmap TARGET | Establish a baseline using the default TCP port selection |
| nmap -sT TARGET | Compare a TCP connect scan with the baseline |
| nmap -sP TARGET | Check host discovery using the legacy option; the current equivalent is -sn |
| nmap -sS TARGET | Compare SYN scanning with TCP connect scanning |
| nmap -sS -A TARGET | Gather service versions, default script output, OS clues, and traceroute |
| nmap -sT -p 6200 TARGET | Check one port relevant to the suspected FTP backdoor |
| nmap -sV -p 21 --script ftp-anon,ftp-syst TARGET | Inspect FTP authentication and configuration |
| nmap -sV -p 21 --script ftp-anon --script-args ftp-anon.maxlist=20 TARGET | Request a bounded anonymous directory listing |
| nmap -sT -p 139,445 --script smb-enum-shares,smb-security-mode TARGET | Enumerate SMB shares and security settings |
| nc -vz -w 3 TARGET 1524 | Check reachability without opening an interactive session |
| nc -v TARGET 1524 | Interact with the exposed shell; whoami and id checked its identity |
| nc -v TARGET 21; ftp TARGET 21 | Examine FTP replies manually, then use a client for data connections |
| smbclient -L TARGET; smbclient -N //TARGET/tmp | Discover shares, then test anonymous access to the tmp share |

I initially combined two TCP scan methods, which Nmap rejected. I also confused Netcat's connectivity-only mode with an interactive connection. Reading the manuals and simplifying the commands helped resolve both problems.

## Evidence and results

| Evidence | Observation | What it establishes |
| --- | --- | --- |
| [D1-01: Default scan](evidence/screenshots/01-default-scan.png) | 23 open TCP ports; 977 closed ports in the default 1,000-port scan | A broad exposed service surface within the tested selection, not a scan of all ports |
| [D1-02: Shell identity](evidence/screenshots/02-root-shell-verification.png) | Remote whoami returned root; id returned uid=0 | Root-level command execution through the existing unauthenticated listener on TCP/1524 |
| [D1-03: SMB operations](evidence/screenshots/03-smb-directory-cleanup.png) | A test directory appeared in the share; later removal was followed by an object-not-found response | Anonymous directory creation and deletion, with cleanup |

D1-01 and D1-02 were collected October 26; D1-03 was collected October 27. These are cropped screenshot excerpts. Redactions remove personal prompts and network identifiers; no result text was replaced. The root account name in D1-02 is part of the privilege evidence.

The advanced scan identified vsftpd 2.3.4, OpenSSH 4.7p1, Apache 2.2.8, and Samba 3.0.20, among other services. It showed a two-hop path and did not provide an exact OS match. These are tool identifications, not independent confirmation of every version or vulnerability.

## Findings

**Exposed root shell — manually verified.** I connected to TCP/1524 and ran harmless identity commands. This demonstrated actual remote root command execution, not just a scanner warning. I used an already exposed shell; I did not deploy an exploit payload or demonstrate privilege escalation.

**Anonymous SMB directory changes — manually verified.** Nmap reported anonymous access, and smbclient allowed listing the tmp share. I created a test directory with md, listed it, then removed it with rmdir after returning to its parent. A later attempt to enter it returned NT_STATUS_OBJECT_NAME_NOT_FOUND. File upload, modification, download, and deletion were not separately verified.

**Anonymous FTP login and empty listings — manually verified.** Both NSE and direct FTP interaction showed successful anonymous authentication. The FTP client returned successful listing-transfer responses without displaying entries. Upload, deletion, and download permissions remain untested.

**FTP backdoor — not confirmed.** TCP/6200 was closed when checked. The reported FTP version prompted research, but I did not demonstrate the backdoor trigger or execute an exploit against it. A matching version and a closed listener do not settle whether the compromised code is present.

**SMB signing — scanner-reported.** NSE reported signing disabled. I did not perform a relay or interception attack.

## Limitations and remediation

This was a focused learning exercise, not a complete penetration test. The initial scans covered the default TCP selection, with later targeted checks; there was no comprehensive UDP assessment.

On a production system, I would remove the unauthenticated shell, restrict SMB to authenticated users with necessary permissions, review anonymous FTP access, and use encrypted file-transfer protocols. Reported legacy services would need further assessment and supported replacements or updates. No production remediation or remediation retest was performed.

## Lessons learned

Open ports are starting points, not automatic vulnerabilities. Service names can be inferred from port numbers, and version detection does not prove a specific weakness. Directly testing one operation gives more precise evidence than treating reported READ/WRITE access as unlimited control.

FTP also helped me separate connectivity, authentication, and authorization: a successful login did not establish a data connection or prove file permissions. My strongest takeaway was to state exactly what each test demonstrated and what remained unknown.
