# Incident Investigation: Malvertising → DLL Sideloading → Domain-Wide Compromise

Note: This write-up describes my own investigative process and findings from a guided SOC training exercise. Platform-specific task prompts and exact UI text are intentionally omitted; this is a narrative reconstruction in my own words, focused on methodology rather than reproducing the exercise content itself.

## Scenario Overview
I investigated a five-day intrusion that began with a single employee clicking a sponsored search result for a routine software update. The case spanned a workstation, a file server, and a domain controller, requiring me to correlate XDR process trees, SIEM process-creation events, and image-load telemetry to reconstruct the full chain from initial click to domain-wide compromise.

## Tools & Data Sources Used
- **XDR** — process trees, file-creation events
- **SIEM** — process-creation events (Sysmon EventID 1), image/module-load events (EventID 7), filtered and sorted by host, user, and time range

## Investigation Timeline

### 1. Tracing the malicious ad to its callback domain
Starting from a flagged workstation, I correlated SIEM and XDR timeline events around the user's initial browser activity to identify the external domain contacted immediately after the sponsored-ad click — a domain designed to look like a legitimate software update source.

### 2. Identifying the delivered artifact
Reviewing file-creation events in the user's Downloads folder shortly after the browser connection, I identified the file that was actually delivered: a disk image (.iso) disguised as a software updater.

### 3. Uncovering DLL sideloading via a mounted image
Walking the process ancestry from `explorer.exe`, I found that a shortcut file on the mounted image launched a signed Windows utility (`rundll32.exe`) — which, instead of behaving normally, loaded a DLL from a user-writable Temp directory rather than a standard system location. I confirmed this by cross-referencing process-creation events with image-load telemetry for the same process ID, since the module-load detail wasn't visible in the process tree alone.

### 4. Fingerprinting a polymorphic dropped executable
A generically-named executable ran from the Temp directory four separate times, each with a different SHA-256 hash — a sign the attacker was re-packing the payload to evade signature-based detection. I filtered SIEM process-creation events for that host and user, sorted them chronologically, and recorded the hash of the earliest execution to anchor the start of this activity.

### 5. Spotting a disguised payload by content-size mismatch
Among several files dropped to the same Temp directory, one carried a `.txt` extension but was over 280 KB — far larger than a plausible plain-text file, and inconsistent with the other legitimately-sized artifacts (a small log file, genuine installer binaries) in the same location. This extension/size mismatch was the indicator that the file was not actually being used as a text file.

### 6. Confirming a second, distinct payload variant
Returning to the same generically-named executable from step 4, I located its last execution in the sequence and recovered its hash, confirming it differed from the earliest variant — establishing that the attacker re-deployed the tool at least twice across the intrusion window.

### 7. Attributing hands-on-keyboard activity to an account
On the domain controller, several accounts had touched the host during the week, but one showed a tight, concentrated cluster of administrative tool execution — distinct from routine service-account activity — identifying the account actually driving that phase of the intrusion.

### 8. Isolating the one non-standard binary in the final burst
In the last cluster of process activity on the original workstation, I compared every process name against known Windows and business software, then checked the image path of each match in the corresponding SIEM process-create events. One process had an arbitrary, non-corporate name and ran directly from a non-standard location — the only binary in the burst that didn't correspond to any legitimate software.

## Key Finding
A single sponsored-ad click triggered a full attack chain: fake update page → disguised ISO delivery → DLL sideloading via a trusted Windows utility → a polymorphic payload re-deployed multiple times to evade detection → lateral administrative activity on the domain controller → a uniquely-named executable in the final burst. No single log source told the whole story — the investigation required pivoting between file-system artifacts, process-creation events, and image-load telemetry, and paying attention to inconsistencies (extension vs. file size, process name vs. execution path) rather than relying on names alone.

## What I Learned
This exercise taught me to treat a file's name and extension as a claim to verify, not a fact to trust — both the oversized `.txt` file and the "update" process whose real path didn't match a legitimate install location were only caught by checking the underlying evidence. I also practiced chronological reconstruction of repeated executions to distinguish an attacker's first foothold from later, evolved variants of the same tool, and learned to isolate a single anomalous process out of a noisy burst of otherwise-legitimate activity by systematically ruling out what it wasn't.
