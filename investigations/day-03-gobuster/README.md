# Day 3: Gobuster

Scheduled: 2026-10-28
Focus: Web directory enumeration

Status: Completed

My goal was to connect Gobuster's path results with the content those paths returned. The follow-ups distinguish redirects, diagnostic pages, and directory indexes.

## Objectives

- Run a bounded directory enumeration against my authorized target.
- Compare discovered paths with a missing-path control.
- Check what a status code and response body actually establish.

## Environment and scope

The Gobuster 3.6 run used Ubuntu under WSL on my Windows host to assess the Metasploitable 2 web server on HTTP/80. This traffic used the existing VMnet1 host-only connection. The target's neighbor-table MAC matched its VM adapter, and its home page identified Metasploitable 2.

This was a different testing path from the physical Kali workstation described in my [lab setup](../../docs/lab-setup.md). No route or firewall rule changed. TARGET replaces the private target address below.

## Commands and approach

| Command | Purpose |
| --- | --- |
| gobuster dir -u http://TARGET/ -w words.txt -t 2 --delay 100ms --timeout 5s --no-color -o results.txt | Test a small list with two workers, a delay between each worker's requests, and a five-second request timeout |
| GET /seven-tools-control-7cfac1 | Establish the response to a deliberately missing path |
| GET /phpinfo.php; GET /dav/; GET /test/ | Inspect diagnostic content and the destinations of selected redirects |

The GET entries describe separate HTTP requests made with Python urllib. The run used a [28-entry list](evidence/sanitized-results/words.txt) containing common web paths and application names already visible on the lab home page. Redirect following was disabled so each response could be assessed separately.

The first command used an older -l option, which this build rejected. Its help showed that response lengths were already included by default. Removing that option resolved the command error. The scan finished with a timeout on /server-status. That path remains unresolved.

## Evidence and results

| Evidence | Observation | What it establishes |
| --- | --- | --- |
| [D3-01: Enumeration](evidence/sanitized-results/01-enumeration.txt) | Ten result lines: four 200 responses and six 301 redirects; one request timeout | Responses for the selected wordlist and the scan's coverage limit |
| [D3-02: HTTP follow-ups](evidence/sanitized-results/02-http-follow-ups.txt) | phpinfo() content, two directory indexes, application pages, and 404 controls | The content or redirect behavior behind selected paths |

These are sanitized text excerpts. TARGET replaces the address; full diagnostic bodies, network details, and actual collection timestamps remain private. The wordlist and visible result values are retained.

Both /phpinfo and /phpinfo.php returned diagnostic pages reporting PHP Version 5.2.4. Requests to /dav/ and /test/ returned 200 responses with Index of headings. The /dvwa/ request redirected again. A 200 response from /phpMyAdmin/ established that its page was reachable; authentication bypass was not tested.

## Findings

**Public diagnostic information - verified by follow-up requests.** The phpinfo() pages exposed runtime and configuration information without authentication. That information can help someone select further tests and learn details that a public application does not need to disclose.

**Directory indexes - verified by follow-up requests.** The /dav/ and /test/ responses displayed directory indexes. Index visibility can reveal filenames and application structure. These requests did not establish upload, write, or deletion permissions.

**Application paths - observed.** Several training applications were reachable or returned redirects. A discovered path alone does not establish a vulnerability in the application.

## Limitations and remediation

The list was deliberately small. Testing covered one host, HTTP/80, and the listed paths. It excluded recursive enumeration, additional virtual hosts, authentication testing, and comprehensive extension searches. The /server-status timeout does not establish whether that endpoint exists or is protected.

For a real service, I would remove public phpinfo() endpoints, disable unnecessary directory indexing, and restrict administrative interfaces to the users and networks that need them. I would retest the affected paths without authentication and check that required application pages still work. No remediation or remediation retest was performed.

## Lessons learned

A discovered path is a lead for the next check. I would read the destination response before drawing a conclusion from its redirect status.

In a future enumeration, I would keep the missing-path control beside the results and record timeouts separately. A timeout answers less than a completed response.
