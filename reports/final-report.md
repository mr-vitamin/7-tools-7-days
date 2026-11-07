# 7 Tools in 7 Days: final project report

Scheduled challenge: October 26 through November 1, 2026
Status: Completed

My project brings together seven investigations of network discovery, packet behavior, web responses, service identification, and offline password recovery. All seven investigations fulfilled their defined objectives. Each report explains what the evidence establishes and where testing stopped.

The dates in the schedule are planned challenge slots. Testing and follow-up continued after that week. Actual collection timestamps, timezones, and evidence provenance are recorded in the private evidence log.

## Objectives

- Use the tools against my authorized lab and understand their output.
- Follow selected results with a separate check.
- Explain supported findings, practical fixes, and the questions still open.

## Environment and scope

The target was my Metasploitable 2 VM, identified as TARGET in public commands and excerpts. My physical Kali workstation reached it through the restricted Ubuntu gateway. Gobuster and the original Nikto run used Ubuntu under WSL on the Windows host through its existing host-only connection. John used a disposable local hash in WSL and made no target connection.

The [lab setup](../docs/lab-setup.md) documents the network, gateway restrictions, and direct host-only path. Target identity was checked using the VM adapter/neighbor MAC match and the lab home page for the Windows path, and the saved route and lab home page for Kali. No router, firewall, SSH, or routing settings were changed during the remaining exercises.

Tests covered one owned training target and one disposable test hash. Household devices, external systems, real password hashes, denial-of-service tests, and target-initiated callbacks were excluded. The lab is intentionally vulnerable; these results describe its tested state rather than a production deployment.

## Commands and approach

| Investigation | Approach and technical outcome | Completion boundary |
| --- | --- | --- |
| [Day 1: Nmap](../investigations/day-01-nmap/) | Default and targeted reconnaissance, followed by direct shell, SMB and FTP checks | Completed; selected TCP services, with no comprehensive UDP assessment |
| [Day 2: Wireshark](../investigations/day-02-wireshark/) | ICMP, TCP handshake and SYN-scan comparisons; readable FTP authentication | Completed; original PCAPs unavailable for reanalysis |
| [Day 3: Gobuster](../investigations/day-03-gobuster/) | 28-entry list, ten result lines, missing-path control and selected HTTP follow-ups | Completed; /server-status timed out and enumeration was limited |
| [Day 4: Burp Suite](../investigations/day-04-burp-suite/) | Five HTTP-client requests comparing query, method, missing-path and diagnostic responses | Completed; headless HTTP-client comparisons; GUI, Repeater, and injection testing outside scope |
| [Day 5: Metasploit](../investigations/day-05-metasploit/) | Auxiliary FTP version module, followed by independent greeting and QUIT | Completed; service enumeration and independent validation; exploitation outside scope |
| [Day 6: John the Ripper](../investigations/day-06-john-the-ripper/) | Negative and positive candidate lists, --show summaries and independent hash comparison | Completed; one generated test hash, with no real-account strength conclusion |
| [Day 7: Nikto](../investigations/day-07-nikto/) | Bounded configuration/disclosure scan and independent response-header follow-ups | Completed bounded assessment; scan stopped before full selected-test coverage |

The remaining tools were Gobuster 3.6, Burp Community 2026.9, Metasploit 6.5.7, John 1.9.0, and Nikto 2.6.1. The original Nmap and Wireshark reports do not record tool versions. Their reports and evidence are preserved unchanged.

## Evidence and results

Full outputs and sensitive inputs remain private. The public evidence consists of the six existing cropped screenshots and thirteen new text files, including the Gobuster wordlist. Text excerpts state what was redacted or omitted. Diagnostic bodies, credentials, private addresses, packet captures, hashes, and recovered values are excluded.

The following findings summarize the supported results. Repeated observations of one endpoint are grouped together rather than counted as separate vulnerabilities.

| Finding | Supporting evidence | What was established |
| --- | --- | --- |
| Unauthenticated root command execution on TCP/1524 | D1-02 | The existing listener accepted commands; whoami and id returned root identity |
| Anonymous SMB directory changes | D1-03 | A test directory could be created and removed anonymously, with cleanup checked |
| Plaintext FTP authentication | D2-03 | Submitted authentication values were readable in the captured control-channel traffic |
| Anonymous FTP acceptance | Day 1 notes and D2-03 success response | Anonymous authentication was accepted; detailed file-operation permissions remain untested |
| Public PHP diagnostic pages | D3-02, D4-02, D4-03 | Unauthenticated requests retrieved phpinfo() content |
| Directory indexes | D3-02 | /dav/ and /test/ displayed Index of headings |
| Advertised FTP version | D5-01, D5-02 | Metasploit and a separate greeting agreed on vsFTPd 2.3.4; exploitability was not established |
| HTTP version disclosure and missing suggested headers | D7-01, D7-02 | Selected HTTP responses advertised versions and omitted the five reported headers |
| Disposable offline recovery | D6-01 through D6-03 | Candidate inclusion changed the result; the recovered value matched independently |

## Findings

**Exposed root listener - confirmed in Day 1.** This was the clearest demonstrated impact: unauthenticated access allowed root-level commands through an existing listener. It did not require a deployed payload or show privilege escalation from a lower-privilege account. In a real environment, I would remove the listener, investigate why it existed, and assess the host's integrity before trusting it again.

**Anonymous file-service access - supported within tested operations.** The SMB test demonstrated anonymous directory creation and deletion. FTP accepted anonymous authentication, and Wireshark showed readable authentication input. I would limit anonymous access to an explicit requirement, apply least privilege, and use an encrypted transfer protocol. File upload, modification, download and deletion permissions were not comprehensively tested across either service.

**Diagnostic and directory exposure - confirmed through HTTP responses.** The PHP pages revealed runtime/configuration information, while directory indexes exposed navigable directory structure. PHP's [phpinfo documentation](https://www.php.net/manual/en/function.phpinfo.php) describes the diagnostic information that function can emit. I would remove public diagnostic endpoints and disable unnecessary indexes, using the supported server's [directory Options configuration](https://httpd.apache.org/docs/2.4/mod/core.html#options) when planning a replacement deployment. The modern documentation is guidance for that replacement, not proof of the old lab server's configuration.

**Version and header observations - confirmed at the response level.** Version strings guide further investigation but cannot establish binary provenance or patch state. The five missing headers were reproduced on selected HTTP responses. Their application impact was not tested. In particular, [HSTS is an HTTPS-delivered policy](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/Strict-Transport-Security); the HTTP-only scan did not validate an HTTPS service or demonstrate an HSTS-related attack.

**Password recovery - a controlled exercise.** The recovered disposable value was deliberately included in the positive list. The negative result says only that its three candidates failed. This supports a lesson about offline guessing and verification, with no finding about a real user's password. For actual application storage, I would follow maintained [password-hashing guidance](https://cheatsheetseries.owasp.org/cheatsheets/Password_Storage_Cheat_Sheet.html), protect the hashes, and choose suitable work factors and unique salts.

**Unresolved hypotheses.** The suspected FTP backdoor was not triggered or tested. SMB signing remained scanner-reported from Day 1. No relay, interception, authenticated web authorization, or browser exploit test confirmed additional impact. The Day 5 version module did not resolve these questions.

## Limitations and remediation

The work was a series of bounded learning investigations. It was not an exhaustive penetration test. The main boundaries are the unavailable original Wireshark captures, Gobuster's limited wordlist and one timeout, Burp's HTTP-client scope, Metasploit's banner-validation scope, the single disposable John test hash, and Nikto's time-limit termination. GUI and Repeater testing and Metasploit exploitation were outside the completed exercises.

Testing did not cover all ports, protocols, virtual hosts, authenticated web paths, application logic, or HTTPS. Root and file-service results from Days 1-2 are historical observations supported by those unchanged investigations; they were not repeated during the remaining investigations. Recommended fixes were not applied, so there are no remediation-pass claims.

For a production system with these confirmed behaviors, I would prioritize the work as follows:

| Priority | Action and reason | Retest I would perform |
| --- | --- | --- |
| First | Remove the unauthenticated root listener and investigate host integrity because direct privileged execution was demonstrated | Verify the listener is gone, inspect unexpected services, and assess the rebuilt or remediated host from an approved client |
| Next | Restrict anonymous SMB operations and review anonymous FTP requirements; protect required file transfers with encryption | Check unauthenticated denials, approved-user permissions, and encrypted transport for required operations |
| Next | Remove diagnostic endpoints, disable unnecessary indexing, and restrict administration pages | Repeat the affected requests without authentication and check that required application functions still work |
| Planned hardening | Verify package provenance and support status, replace unsuitable software, and set application-appropriate security headers | Check actual packages and selected GET/HEAD responses; validate HTTPS before assessing HSTS |

These are proposed actions for a real service. No remediation changes were made to the training target or lab network.

## Lessons learned

The most useful pattern across the project was to turn each output into a specific follow-up question. Nmap identified services to inspect. Wireshark showed the exchanges behind those connections. Gobuster found paths, Burp made the response differences explicit, and Nikto provided configuration observations to validate.

Metasploit reinforced the boundary between identifying a service and exploiting it. John made the same principle concrete in a different setting: a success counter became a supported result when the recovered value passed an independent check.

For a future investigation, I would define any broader web or exploitation scope separately, preserve the original packet captures, and include remediation retests. The completed series supports the selected observations without extending them into untested claims.

## Evidence index

| Evidence ID | Reviewed file | Main use |
| --- | --- | --- |
| D1-01 | [Default scan](../investigations/day-01-nmap/evidence/screenshots/01-default-scan.png) | TCP reconnaissance |
| D1-02 | [Shell identity](../investigations/day-01-nmap/evidence/screenshots/02-root-shell-verification.png) | Root execution |
| D1-03 | [SMB operations](../investigations/day-01-nmap/evidence/screenshots/03-smb-directory-cleanup.png) | Anonymous directory changes |
| D2-01 | [TCP connection](../investigations/day-02-wireshark/evidence/screenshots/01-tcp-handshake.png) | Handshake behavior |
| D2-02 | [SYN scan](../investigations/day-02-wireshark/evidence/screenshots/02-syn-scan.png) | Probe/response pattern |
| D2-03 | [FTP authentication](../investigations/day-02-wireshark/evidence/screenshots/03-ftp-authentication.png) | Plaintext input and login response |
| D3-01 | [Enumeration](../investigations/day-03-gobuster/evidence/sanitized-results/01-enumeration.txt) | Paths and timeout |
| D3-02 | [HTTP follow-ups](../investigations/day-03-gobuster/evidence/sanitized-results/02-http-follow-ups.txt) | Diagnostic pages and indexes |
| D4-01 | [Request comparisons](../investigations/day-04-burp-suite/evidence/sanitized-results/01-request-comparisons.txt) | Request/response differences |
| D4-02 | [Diagnostic response](../investigations/day-04-burp-suite/evidence/sanitized-results/02-diagnostic-excerpt.txt) | PHP disclosure |
| D4-03 | [Independent requests](../investigations/day-04-burp-suite/evidence/sanitized-results/03-independent-follow-ups.txt) | Separate HTTP check |
| D5-01 | [Version module](../investigations/day-05-metasploit/evidence/sanitized-results/01-ftp-version-module.txt) | Metasploit result |
| D5-02 | [Direct banner](../investigations/day-05-metasploit/evidence/sanitized-results/02-direct-banner.txt) | Independent FTP greeting |
| D6-01 | [Negative control](../investigations/day-06-john-the-ripper/evidence/sanitized-results/01-negative-control.txt) | Unsuccessful candidates |
| D6-02 | [Recovery](../investigations/day-06-john-the-ripper/evidence/sanitized-results/02-recovery.txt) | Successful candidate run |
| D6-03 | [Independent check](../investigations/day-06-john-the-ripper/evidence/sanitized-results/03-independent-check.txt) | Hash/value comparison |
| D7-01 | [Bounded scan](../investigations/day-07-nikto/evidence/sanitized-results/01-bounded-scan.txt) | Alerts and termination |
| D7-02 | [Header validation](../investigations/day-07-nikto/evidence/sanitized-results/02-header-validation.txt) | Independent response headers |

Actual collection times and timezone details are retained in the private evidence log. The original screenshot-based investigations retain their existing date statements.
