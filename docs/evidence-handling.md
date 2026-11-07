# My evidence notes

The project evidence consists of cropped screenshots, sanitized text excerpts, and a small enumeration wordlist. Each investigation links numbered evidence items and explains what the visible results establish.

## Public evidence

The six screenshots show Nmap and Wireshark results. Cropping and solid redactions remove personal prompts, network identifiers, and authentication values while retaining the relevant ports, flags, replies, and command results.

The thirteen text files include tool-output excerpts, HTTP observations, verification summaries, and the Gobuster wordlist. Their notes identify substitutions and omissions. TARGET replaces the private target address, and the recovered disposable value is explicitly redacted. Selected diagnostic strings support the findings without exposing complete configuration pages.

Evidence identifiers connect each report's observations with its files. Scheduled challenge dates remain in the reports; actual collection timestamps, timezones, and source provenance are recorded in the private evidence log. The Nmap and Wireshark reports also include their investigation dates.

## Private material

Raw screenshots, full outputs, lab notes, authentication values, generated hashes, recovered values, and available packet captures stay outside the public repository. Sanitized publication copies are kept separate from their source material.

The Wireshark investigation is supported by notes and screenshots. The packet captures named in those screenshots were unavailable for reanalysis; the report records that limitation.

## Publication review

The publication checks covered file contents, filenames, image metadata, redactions, evidence references, and relative Markdown links. They also checked the exact Git file list and diff for credentials, tokens, private addresses, internal paths, and unintended changes.

The repository's ignore rules exclude common sensitive and raw-evidence file types. The content review also checks material that could pass those filename rules, including text files and previously tracked files.
