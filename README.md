# 7 Tools in 7 Days

I'm a cybersecurity student taking on a seven-day challenge to get hands-on experience with penetration testing tools, strengthen my Kali Linux and networking fundamentals, and build practical cybersecurity skills.

From October 26 through November 1, 2026, I plan to focus on one tool each day and document what I try, what the results mean, and where I need more practice. This repository will be my record of that learning.

**Current status: Not Started.** The investigation notes and report are blank templates for the upcoming challenge.

## Seven-day schedule

| Day | Date | Tool | Focus |
| --- | --- | --- | --- |
| 1 | October 26 | Nmap | Network reconnaissance |
| 2 | October 27 | Wireshark | Packet analysis |
| 3 | October 28 | Gobuster | Web directory enumeration |
| 4 | October 29 | Burp Suite | Web application testing |
| 5 | October 30 | Metasploit | Exploitation in an authorized lab |
| 6 | October 31 | John the Ripper | Password auditing |
| 7 | November 1 | Nikto | Web server assessment and final report |

[Progress tracker](PROGRESS.md)

## My lab

My main computer runs Windows 11, and a separate physical Kali Linux PC will be my testing workstation. Both connect through a TP-Link TL-SG105 unmanaged switch. VMware is installed on Windows, where I plan to host deliberately vulnerable training machines.

Lab networking and target setup are still pending. The switch alone doesn't isolate the lab; I'll establish the testing boundaries before starting.

Testing will be limited to my own training systems or targets I have explicit permission to assess. Raw evidence will stay in a separate private shared workspace. Only reviewed, sanitized material will go here, without passwords, tokens, private network details, or sensitive packet captures.

## What I want to learn

- Use each tool and understand its options and output.
- Connect the exercises to Linux commands, networking, and troubleshooting.
- Explain observations with evidence and separate confirmed findings from tool alerts.
- Recognize gaps in my testing and suggest practical fixes for supported findings.
- Write clear investigation notes and a final report.

## Project notes

The [investigations](investigations/) folder has a record for each day, with space for commands, evidence, findings, limitations, and lessons learned. I'll track Linux concepts in my [Kali fundamentals log](docs/kali-fundamentals.md) and use the [templates](templates/) for consistent notes and reporting.

The [reports](reports/) folder is reserved for the final report. My [evidence notes](docs/evidence-handling.md) explain how I'll keep private originals separate from published copies.
