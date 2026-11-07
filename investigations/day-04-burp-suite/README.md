# Day 4: Burp Suite

Scheduled: 2026-10-29
Focus: HTTP request and response testing

Status: Completed

The web paths from Gobuster gave this investigation something specific to examine. My focus was how individual HTTP requests affected the response: a query-string change, HEAD instead of GET, a missing path, and the diagnostic page.

## Objectives

- Inspect the relationship between an HTTP request and its response.
- Compare a baseline with one change at a time.
- Confirm a content disclosure with a separate HTTP client.

## Environment and scope

The Burp Suite Community Edition 2026.9 exercise ran on my physical Kali workstation against Metasploitable 2 on HTTP/80 through the existing Ubuntu gateway. The target's home page and the saved route were checked before testing. My [lab setup](../../docs/lab-setup.md) describes the path and restrictions.

Burp's HTTP client sent five requests through its extension API in headless mode. Interception was disabled, and scope included only the target's HTTP URL. TARGET replaces the private address. The requests were unauthenticated.

## Commands and approach

| Request or setting | Purpose |
| --- | --- |
| GET / HTTP/1.1 | Establish the home-page baseline |
| GET /?lab=day4 HTTP/1.1 | Compare the response after adding a harmless query parameter |
| HEAD / HTTP/1.1 | Compare response headers with the GET response without retrieving its body |
| GET /seven-tools-control-7cfac1 HTTP/1.1 | Check a missing-path control |
| GET /phpinfo.php HTTP/1.1 | Inspect the diagnostic endpoint found during Day 3 |
| Host: TARGET; Connection: close | Keep each request directed at the lab host and close its connection |
| Python urllib requests from Kali | Check selected responses independently of Burp |

An earlier headless launch under WSL produced Java/Swing errors before any target test. The installed Kali copy initially required a startup-option correction: its value had to be attached with an equals sign. With that corrected, the HTTP client completed all five requests. UI errors were still logged, while the saved request/response pairs verified the HTTP-client function.

## Evidence and results

| Evidence | Observation | What it establishes |
| --- | --- | --- |
| [D4-01: Request comparisons](evidence/sanitized-results/01-request-comparisons.txt) | Baseline and query requests returned 200 with identical 891-character bodies; HEAD returned 200 with no body; the missing path returned 404 | The observed differences between these particular requests |
| [D4-02: Diagnostic response](evidence/sanitized-results/02-diagnostic-excerpt.txt) | /phpinfo.php returned 200 and PHP Version 5.2.4 | Diagnostic content was accessible without authentication |
| [D4-03: Independent requests](evidence/sanitized-results/03-independent-follow-ups.txt) | A separate client returned the same selected statuses and page titles | The endpoint behavior was reproduced outside Burp |

The excerpts preserve request methods, paths, status lines, selected headers, and relevant body strings. Addresses and execution timestamps are omitted or replaced; complete bodies remain private.

The home-page body stayed identical after adding lab=day4, so the result leaves parameter handling unresolved. HEAD returned the same success status with an empty body, consistent with its purpose.

The separate client also sent OPTIONS / and received 200. This was an additional HTTP observation in D4-03, separate from the five Burp requests. That response alone does not establish which methods the application permits.

## Findings

**Diagnostic disclosure - independently reproduced.** Burp retrieved the phpinfo() page without authentication. A separate client confirmed the same endpoint and title. This supports the Day 3 disclosure finding and adds a second tool's request/response evidence.

**Request handling - observed.** The query variation, HEAD request, and missing-path control produced distinguishable results. These comparisons establish the response behavior of the selected requests. Validation, injection, and authorization questions remain outside this request-comparison exercise.

## Limitations and remediation

The exercise covered five Burp HTTP-client requests and separate read-only checks. The saved exchanges support the HTTP comparisons despite the startup UI errors. GUI and Repeater operation, authenticated sessions, forms, injection, and HTTPS interception were outside the tested scope.

For a production application, I would remove or restrict the diagnostic page and reduce unnecessary server-version disclosure. The retest would request that endpoint without authentication and check the application's required routes. No remediation changes or retests were performed.

## Lessons learned

The unchanged response to the query-string variation was as useful as a changed response would have been: it defined what the comparison could support. HEAD and the missing-path control added context without changing several request properties at once.

The startup errors also needed to stay separate from the HTTP results. For me, the useful check is the saved exchange for the function being tested. Here, the five completed exchanges support the HTTP-client investigation.
