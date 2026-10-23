# My evidence notes

I'll keep raw screenshots, packet captures, logs, and lab notes in the private shared workspace. Published evidence will be a separate, sanitized copy, so the originals stay unchanged.

My naming pattern will be something like day-01_nmap_2026-10-26_001_description.png. Each investigation will include evidence IDs, collection times and timezone, and a short explanation of what the files show.

Before publishing, I'll review the content, filenames, and metadata for credentials, tokens, usernames, private addresses, and other confidential details. Packet captures will stay private; sanitized text excerpts will usually be enough to explain an observation.

The publication-staging folder is a place for copies awaiting review. I'll move selected files into the repository only after checking them, then review the staged filenames and full Git diff before committing. The ignore rules help exclude sensitive file types, but they can't catch every secret or protect files already tracked by Git.
