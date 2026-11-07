# Day 6: John the Ripper

Scheduled: 2026-10-31
Focus: Password auditing

Status: Completed

A disposable hash made it possible to compare an unsuccessful John wordlist run with a successful one and verify the recovered value independently. The exercise stayed entirely offline and used no real account password or system hash.

## Objectives

- Understand how an offline wordlist test compares candidates with a stored hash.
- Use negative and positive controls to explain a recovery result.
- Verify the recovered value independently of John's summary.

## Environment and scope

The exercise used John the Ripper 1.9.0 in Ubuntu under WSL on my Windows host. Python's crypt module generated one disposable SHA-512 crypt hash with a fresh salt. The test value was created for this exercise and was never assigned to an account.

This exercise made no connection to the lab target. The hash, generated value, wordlists containing that value, and recovery data remain private. The names below identify local exercise files, with no hash or plaintext included in the repository.

## Commands and approach

| Command or step | Purpose |
| --- | --- |
| crypt.crypt(value, crypt.mksalt(crypt.METHOD_SHA512)) | Generate a salted disposable SHA-512 crypt hash |
| john --format=crypt --wordlist=negative.txt --session=negative test.hash | Test three candidates that excluded the generated value |
| john --show --format=crypt test.hash | Check the recovery summary after the negative run |
| john --format=crypt --wordlist=positive.txt --session=positive test.hash | Test four candidates, including the generated value |
| john --show --format=crypt test.hash | Check the recovery summary after the positive run |
| crypt.crypt(recovered, stored_hash) == stored_hash | Independently recompute and compare the hash |
| recovered == known_value | Compare recovery with the disposable value originally generated |

The commands explicitly selected the generic crypt format for this build. Separate session names kept the two runs identifiable. John stored recovery state for --show; that state and all sensitive inputs stayed outside the public repository.

The crypt module emitted a deprecation warning under Python 3.12. It remained available for this test. Repeating the generation step in a newer environment would require a supported compatible implementation.

## Evidence and results

| Evidence | Observation | What it establishes |
| --- | --- | --- |
| [D6-01: Negative control](evidence/sanitized-results/01-negative-control.txt) | One hash loaded; the candidate run completed with 0g | The three candidates did not recover this test value |
| [D6-02: Recovery run](evidence/sanitized-results/02-recovery.txt) | One hash loaded; the run completed with 1g | The second candidate list recovered one value |
| [D6-03: Independent verification](evidence/sanitized-results/03-independent-check.txt) | The hash comparison and known-value comparison returned True; --show changed from 0 cracked to 1 cracked | The recovered value matched both the stored hash and the original disposable value |

The recovered value is covered by an explicit text redaction. No hashes, salts, candidate lists, or actual execution timestamps are published. Remaining tool-output lines and verification summaries are retained.

## Findings

**Controlled offline recovery - independently verified.** John recovered the generated value when it was included in the candidate list. Recomputing the stored hash with that value produced a match.

**Wordlist coverage - demonstrated by the controls.** The negative run left one hash unresolved. That result described those candidates; it did not establish that the value was secure against other candidates or attack modes.

## Limitations and remediation

This was a one-hash learning exercise with deliberately constructed wordlists. It was not an audit of any user's password, a representative password-strength study, or a cracking-speed benchmark. The short runs and displayed rates cannot support a general time-to-crack estimate.

For a real application, I would protect stored hashes, use a maintained password-hashing scheme with an appropriate work factor and unique salts, and help users choose long, unique passwords. MFA and online rate limits address additional risks, while offline guesses against an already obtained hash remain a separate problem. No account or password-storage configuration was changed or retested.

## Lessons learned

Candidate coverage explains the result here. Including the known value in the second list changed the outcome from no recovery to one recovery.

For a future audit, I would retain an independent check alongside the tool's success counter. A demonstration with a known candidate supports only the stated exercise.
