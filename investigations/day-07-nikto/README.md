# Day 7: Nikto

Scheduled: 2026-11-01
Focus: Bounded web-server assessment

Status: Completed

My final investigation brought the web-server observations together with a three-minute Nikto scan. Direct HTTP requests checked selected alerts, including the context behind a missing HSTS header on an HTTP service.

## Objectives

- Run a bounded web-server assessment against the lab target.
- Validate selected alerts using independent response evidence.
- Bring the seven investigations together in a [final report](../../reports/final-report.md).

## Environment and scope

The Nikto 2.6.1 run used Ubuntu under WSL on my Windows host against Metasploitable 2 on HTTP/80 through the existing host-only connection. The target's VM adapter identity and home page were checked before testing. Independent HTTP follow-ups used my physical Kali workstation through the Ubuntu gateway.

The [lab setup](../../docs/lab-setup.md) describes the gateway restrictions and direct host-only path. The scan and follow-ups used those existing connections. TARGET replaces the private address below.

## Commands and approach

| Command or step | Purpose |
| --- | --- |
| perl nikto/program/nikto.pl -Version | Record the installed scanner version |
| perl nikto/program/nikto.pl -h http://TARGET/ -Tuning 123b -Plugins 'headers;tests' -maxtime 180s -Pause 0.1 -timeout 5 -nocheck -nolookup -nointeractive -Format txt -output report.txt | Limit the run to one host, selected configuration/disclosure/identification tests, selected plugins, and a three-minute ceiling |
| GET /; HEAD /; GET /phpinfo.php | Independently inspect selected response headers from Kali |

The HTTP follow-ups used Python urllib. The scan categories were interesting files, misconfiguration/default files, information disclosure, and software identification. The pause spaced requests; response times also affected the rate.

The initial launch required the XML::Writer dependency. Nikto also rejected combining a full URL with a separate -port option. Installing the dependency in the testing environment and using the URL alone resolved those setup issues. The assessment then ran until its configured time limit.

## Evidence and results

| Evidence | Observation | What it establishes |
| --- | --- | --- |
| [D7-01: Bounded scan](evidence/sanitized-results/01-bounded-scan.txt) | Six reported items; termination at the 180-second limit with one error | The alerts produced before the scan stopped, with incomplete coverage |
| [D7-02: Header validation](evidence/sanitized-results/02-header-validation.txt) | Independent GET and HEAD responses lacked the five flagged headers and included the PHP version header | The response-header observations were reproducible on the selected HTTP endpoints |

These are sanitized text excerpts. Target identifiers and actual test timestamps are removed; alert text, response-header values, and termination details are retained. Exact test times and the scanner source revision are recorded privately.

Nikto reported PHP version disclosure and missing Referrer-Policy, Strict-Transport-Security, Content-Security-Policy, X-Content-Type-Options, and Permissions-Policy headers. The direct root GET, root HEAD, and diagnostic GET responses reproduced those observations.

## Findings

**Version disclosure - independently verified at the response level.** The selected responses advertised PHP/5.2.4-2ubuntu5.10 and Apache/2.2.8. These headers reveal software identifications. They do not independently establish the installed package's provenance or patch state.

**Missing suggested browser-protection headers - independently observed.** The five headers were absent from the tested responses. CSP, MIME-type protections, referrer policy, and feature permissions need policies suited to the application. Their absence alone does not demonstrate script execution, data leakage, or a working browser attack.

**HSTS alert - limited by HTTP scope.** The scan used plain HTTP. HSTS is meaningful when delivered over HTTPS; the tested responses do not establish whether an HTTPS service exists or how it is configured.

## Limitations and remediation

Nikto stopped at its configured time limit before exhausting the selected tests. The follow-ups validated the reported headers on the chosen HTTP responses. Authenticated areas, HTTPS configuration, browser exploitation, and denial-of-service testing were outside this assessment.

For a real deployment, I would remove diagnostic pages, reduce unnecessary version disclosure, and assess the installed packages for supported replacements. Suitable browser-protection headers would need application compatibility checks. HTTPS and certificate validation would come before assessing HSTS over HTTPS. No server configuration changes or remediation retests were performed.

## Lessons learned

The scan reported one error when it reached the time limit. That termination belongs beside the six reported items because it explains the coverage. The independent requests answered specific alert questions without repeating the whole scan.

The HSTS alert was a reminder to read the protocol context. My next question for a separate HTTPS assessment would be whether a trusted TLS service exists and what policy it returns. The HTTP responses here support the narrower header observations.
