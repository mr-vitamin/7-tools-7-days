# Evidence handling
Raw originals belong in the private shared workspace, outside Git. Preserve them unchanged.

Use names such as day-01_nmap_2026-10-26_001_description.png. Record timestamps and timezone. Avoid credentials and target addresses in filenames.

Create separate sanitized copies. Inspect visible content, embedded text, metadata, and filenames for private addresses, usernames, credentials, tokens, and confidential information. Prefer sanitized text excerpts over captures. PCAPs are excluded from Git.

Publication staging means awaiting review, not approved. Copy only reviewed material into repository evidence folders. Inspect git status, staged filenames, and the complete staged diff before committing. Ignore rules do not protect already tracked files or detect all secrets.
