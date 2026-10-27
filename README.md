# 7 Tools in 7 Days

I'm a cybersecurity student taking on a seven-day challenge to get hands-on experience with penetration testing tools, strengthen my Kali Linux and networking fundamentals, and build practical cybersecurity skills.

The challenge runs from October 26 through November 1, 2026. I focus on one tool each day and keep a record of my commands, results, and questions.

**Current status: Days 1 and 2 completed.** Nmap and Wireshark investigations are documented; Days 3–7 are Not Started.

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

## Lab infrastructure

My lab uses a dedicated physical Kali workstation and VMware Workstation on Windows, with Metasploitable 2 behind a restricted Ubuntu gateway. I've checked target connectivity, a blocked outbound probe, and behavior after reboot. My [lab setup notes](docs/lab-setup.md) cover the network design, troubleshooting, and testing limits.

Testing is limited to my own training systems or explicitly authorized targets. Raw evidence stays in a private shared workspace; only reviewed, sanitized material belongs here.

## What I want to learn

- Use each tool and understand its options and output.
- Connect the exercises to Linux commands, networking, and troubleshooting.
- Use evidence to explain findings and evaluate tool alerts.
- Recognize gaps in my testing and suggest practical fixes for supported findings.
- Write clear investigation notes and a final report.

## Project notes

The [investigations](investigations/) folder has a record for each day, with space for commands, evidence, findings, limitations, and lessons learned. I'll track Linux concepts in my [Kali fundamentals log](docs/kali-fundamentals.md) and use the [templates](templates/) for consistent notes and reporting.

The [reports](reports/) folder is reserved for the final report. My [evidence notes](docs/evidence-handling.md) explain how I'll keep private originals separate from published copies.
