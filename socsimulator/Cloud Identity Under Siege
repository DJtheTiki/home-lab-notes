# Incident Investigation: Web Server Compromise → Lateral Movement → Data Exfiltration

> Note: This write-up describes my own investigative process and findings from a guided SOC training exercise. Platform-specific task prompts and exact UI text are intentionally omitted; this is a narrative reconstruction in my own words, focused on methodology rather than reproducing the exercise content itself.

## Scenario Overview
I investigated a full intrusion chain starting from a public-facing web server exploit, through command-and-control (C2) activity, credential harvesting against backup infrastructure, and ending in data exfiltration to an external cloud destination. The investigation relied on XDR process trees and SIEM event correlation across three different hosts.

## Tools & Data Sources Used
- **XDR** — process trees, execution ancestry, file hashes
- **SIEM** — process-creation events, network proxy logs, Sysmon events

## Investigation Timeline

### 1. Identifying the initial execution technique (LOLBin)
A vulnerable Confluence web server (CVE-2023-22527) gave the attacker code execution. Tracing the process chain, I distinguished between the legitimate web service process (which was simply the exploited entry point) and the actual "Living Off the Land Binary" the attacker repurposed: a signed Windows utility (`mshta.exe`) called with a remote `.hta` payload URL — a usage pattern that never occurs in legitimate operation, since this binary should only execute local files, not remote ones.

### 2. Confirming the C2 channel
Using XDR network events, I confirmed the outbound connection established immediately after the first-stage execution, identifying the external IP/port the compromised host called back to.

### 3. Mapping the in-memory PowerShell stage
I located the hidden PowerShell process spawned as part of the intrusion and recorded its SHA-256 hash from the XDR process node for later reference and threat-intel lookups.

### 4. Identifying the second-stage tool download
Reviewing SIEM proxy logs, I found an HTTP download event for an unrecognized binary and correlated it with a SIEM process-creation event to confirm the exact filename written to disk.

### 5. Recovering the payload hash from the correct source
This step taught an important distinction: the download event (proxy log) and the execution event (Sysmon process-creation log) are two separate records from two separate log sources — only the execution record carried the SHA-256 hash, since proxy logs typically capture traffic metadata, not file hashes. Cross-referencing the correct record was necessary to recover the hash.

### 6. Tracing credential harvesting on backup infrastructure
Pivoting to a second host (the backup server), I reviewed XDR timeline events tied to a service account and identified a PowerShell script specifically designed to extract stored credentials from the backup software — a tool matching a known, publicly documented credential-dumping pattern.

### 7. Identifying the exfiltration tool
On a third host (the file server), I located a process execution of a legitimate file-sync utility being abused to copy a sensitive file share to an external cloud storage destination, recording its SHA-256 hash from the XDR process node.

### 8. Scoping the exfiltration destination
Finally, I reviewed network connection events tied to that same process to identify the external IP address that received the exfiltrated data, confirming successful outbound transfer.

## Key Finding
The intrusion followed a classic chain: web-facing exploit → LOLBin-based initial execution → C2 beaconing → credential harvesting against backup infrastructure → lateral movement → data exfiltration via a legitimate cloud-sync tool. Each stage used a different host and a different "living off the land" technique, making single-host analysis insufficient — the full picture only emerged by correlating evidence across all three compromised systems.

## What I Learned
This exercise sharpened my ability to distinguish a legitimate-but-exploited process from the actual attacker tooling repurposing it, and taught me to verify *which* log source actually carries a given piece of evidence (e.g. a hash) rather than assuming it's present everywhere. I also practiced recognizing that attackers often rename or disguise malicious scripts to blend into normal administrative activity, reinforcing that context and behavior matter more than filenames alone.
