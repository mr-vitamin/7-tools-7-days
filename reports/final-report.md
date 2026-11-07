# 7 Tools in 7 Days: final project report

Scheduled challenge: October 26 through November 1, 2026
Status: Completed

This project connects seven investigations in my cybersecurity lab. My goal was to understand the tools well enough to explain their results, check important observations independently, and identify practical fixes. The completed investigations cover network reconnaissance, packet analysis, web paths, HTTP responses, service identification, password recovery, and web-server configuration.

The scheduled dates identify each tool's place in the challenge. Testing and follow-up continued beyond that week; actual collection timestamps and timezone details are recorded privately.

## Objectives

- Understand what each tool's output establishes.
- Check selected results with direct interactions or an independent method.
- Explain supported findings, meaningful testing boundaries, and remediation priorities.

## Environment and scope

The target was my Metasploitable 2 VM, identified as TARGET in public commands and excerpts. Physical Kali reached it through the restricted Ubuntu gateway. Gobuster and the Nikto scan used Ubuntu under WSL on the Windows host through its existing host-only connection. John's disposable hash exercise ran offline in WSL.

The [lab setup](../docs/lab-setup.md) explains these paths, the gateway restrictions, and the temporary key-authenticated SSH access from Windows to Kali. Target identity checks used the VM adapter/neighbor MAC match and lab home page for the Windows path, and the saved route and lab home page for Kali. The tool exercises used the established target connections without changing router settings or gateway routing rules.

Testing covered the owned training target and a disposable test hash. Household devices, external systems, real account hashes, denial-of-service tests, and target-initiated callbacks were outside scope.

## Commands and approach

| Investigation | Approach | Technical result |
| --- | --- | --- |
| [Day 1: Nmap](../investigations/day-01-nmap/) | Default and targeted scans, followed by direct service checks | Exposed TCP services; verified root listener, anonymous SMB directory operations, and anonymous FTP acceptance |
| [Day 2: Wireshark](../investigations/day-02-wireshark/) | Compare generated ICMP, TCP connection, SYN-scan, and FTP traffic | Packet-level explanations of the connections and readable FTP authentication |
| [Day 3: Gobuster](../investigations/day-03-gobuster/) | Enumerate a 28-entry list, then request selected paths and a missing-path control | Ten result lines; confirmed diagnostic pages and directory indexes; one timeout |
| [Day 4: Burp Suite](../investigations/day-04-burp-suite/) | Compare five HTTP-client requests and check selected responses independently | Query/method/control comparisons and reproduced diagnostic disclosure |
| [Day 5: Metasploit](../investigations/day-05-metasploit/) | Run the auxiliary FTP version module and read a separate greeting | Matching vsFTPd 2.3.4 identifications and a normal QUIT reply |
| [Day 6: John the Ripper](../investigations/day-06-john-the-ripper/) | Compare negative and positive candidate lists, then verify the recovery independently | One disposable value recovered; recomputed hash and known-value comparisons matched |
| [Day 7: Nikto](../investigations/day-07-nikto/) | Run a time-limited configuration/disclosure scan and inspect selected headers directly | Six reported items; independent header validation; scan terminated at its limit |

Recorded versions include Gobuster 3.6, Burp Community 2026.9, Metasploit 6.5.7, John 1.9.0, and Nikto 2.6.1.

## Evidence and results

The public evidence contains six cropped screenshots and thirteen text files, including the Gobuster wordlist. Each investigation explains its numbered evidence and any redactions. Full diagnostic bodies, authentication values, private addresses, hashes, recovered values, and packet captures remain private. The [evidence notes](../docs/evidence-handling.md) describe the handling practices.

The findings below group repeated observations of the same behavior together.

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

**Unauthenticated root execution - confirmed.** The TCP/1524 listener accepted harmless identity commands that returned root and uid=0. This demonstrated privileged execution through an existing listener. In a real environment, I would remove the listener, investigate its origin, and assess host integrity before trusting the system again.

**Anonymous file-service access and plaintext FTP - confirmed within the tested operations.** SMB allowed anonymous directory creation and deletion, with cleanup verified. FTP accepted anonymous authentication, and the packet evidence exposed readable authentication input. I would restrict anonymous access to a defined need, apply least privilege, and use encrypted transfer protocols.

**Public diagnostic pages and directory indexes - verified through HTTP requests.** The phpinfo() responses exposed runtime and configuration information, and /dav/ and /test/ displayed directory indexes. Burp reproduced the diagnostic response, and a separate client corroborated it. I would remove public diagnostic endpoints and disable unnecessary indexing. PHP's [phpinfo documentation](https://www.php.net/manual/en/function.phpinfo.php) describes the information the function can expose; Apache's [Options documentation](https://httpd.apache.org/docs/2.4/mod/core.html#options) provides indexing guidance for a supported replacement deployment.

**Advertised service versions - independently corroborated.** Metasploit and a separate FTP connection agreed on vsFTPd 2.3.4. Selected HTTP responses advertised PHP/5.2.4-2ubuntu5.10 and Apache/2.2.8. These are service identifications; installed package provenance, patch state, and exploitability need different evidence. The suspected FTP backdoor remains unverified.

**Missing suggested HTTP headers - independently observed.** Direct requests reproduced the five header omissions reported by Nikto. Their impact depends on the application and browser context. In particular, [HSTS is delivered over HTTPS](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/Strict-Transport-Security), so its absence on these plain-HTTP responses does not establish the configuration of an HTTPS service.

**Controlled offline recovery - independently verified.** John's negative list left one hash unresolved. The positive list included the generated value and recovered it; independent hash and known-value comparisons matched. For actual application storage, I would follow maintained [password-hashing guidance](https://cheatsheetseries.owasp.org/cheatsheets/Password_Storage_Cheat_Sheet.html), protect stored hashes, and use appropriate work factors and unique salts.

## Limitations and remediation

Each investigation met its stated objectives within a bounded scope. The meaningful coverage limits are:

- Nmap covered selected TCP services, with no comprehensive UDP assessment or exact OS identification. File operations beyond the demonstrated SMB directory changes were not comprehensively tested. SMB signing remained scanner-reported.
- Wireshark's evidence consists of notes and screenshots; the named packet captures were unavailable for reanalysis. The TCP/22 check established a connection without SSH user authentication.
- Gobuster used a limited wordlist and left /server-status unresolved after a timeout. Burp covered HTTP-client comparisons, with UI errors recorded; GUI/Repeater testing, authenticated web logic, injection, and HTTPS interception were outside scope. The independent OPTIONS request was separate from the five Burp requests.
- Metasploit validated the advertised FTP service version. Exploit execution and package integrity assessment were outside that investigation. John's one generated hash and constructed wordlists support the controlled recovery result. General password-strength and cracking-speed estimates were outside scope.
- Nikto reached its 180-second limit before exhausting the selected tests. Follow-ups checked the chosen HTTP responses, without validating HTTPS or demonstrating browser exploitation.

No remediation changes or remediation retests were performed. For a production system with these behaviors, I would prioritize the following work:

| Priority | Action and reason | Retest I would perform |
| --- | --- | --- |
| First | Remove the unauthenticated root listener and investigate host integrity because direct privileged execution was demonstrated | Verify the listener is gone, inspect unexpected services, and assess the rebuilt or remediated host from an approved client |
| Next | Restrict anonymous SMB operations and review anonymous FTP requirements; protect required file transfers with encryption | Check unauthenticated denials, approved-user permissions, and encrypted transport for required operations |
| Next | Remove diagnostic endpoints, disable unnecessary indexing, and restrict administration pages | Repeat the affected requests without authentication and check that required application functions still work |
| Planned hardening | Verify package provenance and support status, replace unsuitable software, and set application-appropriate security headers | Check actual packages and selected GET/HEAD responses; validate HTTPS before assessing HSTS |

## Lessons learned

The investigations build on a useful sequence: identify a service, inspect its behavior, and choose a follow-up that answers a specific question. Nmap's service list led to direct checks, while Wireshark connected the results with packet exchanges. Gobuster's redirects needed destination requests; Burp made the HTTP comparisons explicit.

The imperfect results were part of that reasoning. A rejected flag led back to the installed help, a timeout left a path unresolved, and startup errors had to be assessed separately from successful HTTP exchanges. Metasploit's greeting and John's negative/positive controls made independent checks straightforward. Nikto's time-limit termination explained where its coverage stopped.

For me, the practical habit to carry forward is to keep the command, visible result, and supported interpretation together. That makes a successful test easier to explain and leaves a clear question when the evidence stops short.

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
