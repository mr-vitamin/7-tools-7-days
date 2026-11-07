# 7 Tools in 7 Days

I'm a cybersecurity student taking on a seven-day challenge to get hands-on experience with penetration testing tools, strengthen my Kali Linux and networking fundamentals, and build practical cybersecurity skills.

The scheduled challenge runs from October 26 through November 1, 2026. I focus on one tool per investigation and keep a record of commands, results, and questions. Testing and follow-up continued after the scheduled week; actual execution timestamps are retained in the private evidence log.

**Current status: All seven investigations completed within their defined scope.** Each write-up includes supporting evidence, findings, and testing boundaries. The [final project report](reports/final-report.md) brings the results and remediation priorities together.

## Seven-day schedule

| Day | Scheduled date | Tool | Focus |
| --- | --- | --- | --- |
| 1 | October 26 | Nmap | Network reconnaissance |
| 2 | October 27 | Wireshark | Packet analysis |
| 3 | October 28 | Gobuster | Web directory enumeration |
| 4 | October 29 | Burp Suite | Web application testing |
| 5 | October 30 | Metasploit | Service enumeration and FTP version validation |
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

The [investigations](investigations/) folder records each day's commands, evidence, findings, limitations, and lessons learned. The [templates](templates/) remain available for future investigations. My [Kali fundamentals log](docs/kali-fundamentals.md) is a separate study backlog.

The [final project report](reports/final-report.md) summarizes the seven tools, supported findings, remediation priorities, and coverage gaps. My [evidence notes](docs/evidence-handling.md) describe the separation between private originals and sanitized public copies.
