# 7 Tools in 7 Days

I'm a cybersecurity student interested in how security tools work, what their results mean, and how to check those results. This completed project brings together seven hands-on investigations in my authorized lab.

The investigations connect network reconnaissance and packet analysis with web enumeration, HTTP testing, service validation, password recovery, and web-server assessment. Each report includes commands, supporting evidence, troubleshooting, findings, and lessons learned.

## Seven completed investigations

The scheduled challenge ran from October 26 through November 1, 2026. Testing and follow-up continued beyond that week. These are scheduled dates; actual collection timestamps and timezone details are recorded privately.

| Day | Scheduled date | Investigation | Focus |
| --- | --- | --- | --- |
| 1 | October 26 | [Nmap](investigations/day-01-nmap/) | Network reconnaissance |
| 2 | October 27 | [Wireshark](investigations/day-02-wireshark/) | Packet analysis |
| 3 | October 28 | [Gobuster](investigations/day-03-gobuster/) | Web directory enumeration |
| 4 | October 29 | [Burp Suite](investigations/day-04-burp-suite/) | HTTP request and response testing |
| 5 | October 30 | [Metasploit](investigations/day-05-metasploit/) | Service enumeration and FTP version validation |
| 6 | October 31 | [John the Ripper](investigations/day-06-john-the-ripper/) | Controlled password recovery |
| 7 | November 1 | [Nikto](investigations/day-07-nikto/) | Bounded web-server assessment |

The [final project report](reports/final-report.md) connects the results, explains their impact, and prioritizes remediation. The [progress tracker](PROGRESS.md) lists all seven completed investigations.

## What the project demonstrates

- Turning open ports and service identifications into specific follow-up tests.
- Reading packet sequences and recognizing plaintext authentication traffic.
- Checking discovered web paths and comparing HTTP responses.
- Corroborating a tool result with a separate protocol or hash check.
- Explaining the difference between a confirmed finding and an untested possibility.

The supported findings include unauthenticated root command execution through an existing listener, anonymous SMB directory changes, readable FTP authentication, public diagnostic pages, and directory indexes. The daily reports explain the evidence behind each result and the limits of the tests.

## Lab and evidence

My lab combines a physical Kali workstation, a Windows VMware host, Metasploitable 2, and a restricted Ubuntu gateway. Some exercises used Ubuntu under WSL on the Windows host. The [lab setup](docs/lab-setup.md) explains the testing paths, temporary SSH access to Kali, and isolation checks.

Testing stayed within the owned lab and a disposable local hash exercise. The [evidence notes](docs/evidence-handling.md) describe the published screenshots and text excerpts. Full outputs, private addresses, authentication values, hashes, and packet captures remain outside this repository.

## Reusable materials

The [templates](templates/) are blank starting points for other investigations. They are separate from the completed reports above.
