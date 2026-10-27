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

TARGET replaces the real lab address in the commands I used.

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
| [D1-01: Default scan](evidence/screenshots/01-default-scan.png) | 23 open TCP ports; 977 closed ports in the default 1,000-port scan | The exposed services within the default TCP selection |
| [D1-02: Shell identity](evidence/screenshots/02-root-shell-verification.png) | Remote whoami returned root; id returned uid=0 | Root-level command execution through the existing unauthenticated listener on TCP/1524 |
| [D1-03: SMB operations](evidence/screenshots/03-smb-directory-cleanup.png) | A test directory appeared in the share; later removal was followed by an object-not-found response | Anonymous directory creation and deletion, with cleanup |

D1-01 and D1-02 were collected October 26; D1-03 was collected October 27. These are cropped screenshot excerpts. Redactions cover personal prompts and network identifiers. The visible result text is unchanged. The root account name in D1-02 is part of the privilege evidence.

The advanced scan identified vsftpd 2.3.4, OpenSSH 4.7p1, Apache 2.2.8, and Samba 3.0.20, among other services. It showed a two-hop path. The service versions are recorded as Nmap identifications.

## Findings

**Exposed root shell — manually verified.** I connected to TCP/1524 and ran harmless identity commands. The commands confirmed remote execution with root privileges through the existing listener.

**Anonymous SMB directory changes — manually verified.** Nmap reported anonymous access, and smbclient allowed listing the tmp share. I created a test directory with md, listed it, then removed it with rmdir after returning to its parent. A later attempt to enter it returned NT_STATUS_OBJECT_NAME_NOT_FOUND.

**Anonymous FTP login and empty listings — manually verified.** Both NSE and direct FTP interaction showed successful anonymous authentication. The FTP client returned successful listing-transfer responses without displaying entries.

**FTP backdoor — unverified hypothesis.** The reported FTP version prompted research into a possible backdoor. My targeted scan found TCP/6200 closed at the time of testing.

**SMB signing — scanner-reported.** NSE reported signing disabled.

## Limitations and remediation

The initial scans covered the default TCP selection, followed by targeted checks. Coverage excluded a comprehensive UDP assessment. Nmap returned no exact OS match, and the reported service versions need independent validation.


The root-shell test used an existing listener; it involved no deployed exploit payload or demonstrated privilege escalation. The suspected FTP backdoor remains unverified because I did not test its trigger. SMB file upload, modification, download, and deletion permissions were not separately tested. FTP upload, deletion, and download permissions also remain untested. SMB signing was assessed through NSE output, with no relay or interception test.
On a production system, I would remove the unauthenticated shell, restrict SMB to authenticated users with necessary permissions, review anonymous FTP access, and use encrypted file-transfer protocols. Reported legacy services would need further assessment and supported replacements or updates. No production remediation or remediation retest was performed.

## Lessons learned

I learned to use open ports and service versions to choose the next test. A port-based service label needs checking, and each reported permission needs an operation that verifies it. Creating and removing a directory gave me specific evidence of anonymous SMB access.

FTP made the steps easier to understand: establish the connection, authenticate, open the data connection, and test an operation. I now record the result of each step and the questions it leaves open.
