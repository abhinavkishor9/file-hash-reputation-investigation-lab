# Investigation Timeline

## Timeline Purpose

This timeline correlates file metadata, reputation intelligence, local validation, execution evidence, and endpoint telemetry associated with the investigation of:

```text
C:\Windows\System32\notepad.exe
```

The timeline distinguishes between file-level observations and confirmed process activity.

## File Metadata Timeline

| Timestamp | Evidence | Observation |
|---|---|---|
| 09-09-2026 09:56:13 | Creation Time | File creation timestamp |
| 09-09-2026 09:56:13 | Last Write Time | File modification timestamp |
| 20-09-2026 12:22:01 | Last Access Time | First observed access timestamp |
| 21-09-2026 07:41:36 | Last Access Time | Later observed access timestamp |

The access timestamps were not interpreted as execution evidence.

## External Reputation Timeline

| Evidence | Observation |
|---|---|
| SHA256 | `468FFE129C395ABFB6B21A09EFDF261910A95FB98AA982EAD73CAA7B2B684577E` |
| VirusTotal Detection | `0 / 70` |
| Community Score | `0` |
| Known Filename | `NOTEPAD.EXE` |
| File Type | PE32+ executable, 64-bit |
| Last Analysis | 1 day before investigation capture |

The reputation result provided external context for the identified file.

## Endpoint Execution Timeline

| Timestamp | Source | Event | Assessment |
|---|---|---|---|
| 21-09-2026 07:28:14 | Sysmon | Event ID 1 | `notepad.exe` execution observed |
| 21-09-2026 07:41:24 | Sysmon | Event ID 1 | `notepad.exe` execution observed |

These two Sysmon Event ID 1 records provide direct evidence of process execution.

## Endpoint Network Timeline

Multiple Sysmon Event ID 3 events were observed during the investigation period, including activity around:

```text
07:31
07:32
07:34
07:36
07:38
07:40
07:43
07:45
```

The available output did not establish sufficient process-level attribution to associate these events with `notepad.exe`.

Therefore, the network events remain endpoint-level observations rather than confirmed process-specific activity.

## Wazuh Timeline

Wazuh telemetry included a Sysmon Event ID 1 record.

This demonstrated that Sysmon process telemetry was being ingested into Wazuh.

However, the available evidence did not establish a confirmed Wazuh event containing the investigated SHA256 hash or a specific malicious alert associated with `notepad.exe`.

## Investigation Activity Sequence

```text
09-09-2026
    │
    ├── File creation / write timestamp observed
    │
    ▼
20-09-2026
    │
    └── File access observed
    │
    ▼
21-09-2026 07:28:14
    │
    └── Sysmon Event ID 1
        notepad.exe execution observed
    │
    ▼
21-09-2026 07:31–07:45
    │
    └── Multiple Sysmon Event ID 3 events
        Process attribution not established
    │
    ▼
21-09-2026 07:41:24
    │
    └── Sysmon Event ID 1
        notepad.exe execution observed
    │
    ▼
Reputation / File Validation
    │
    ├── SHA256 calculated
    ├── VirusTotal: 0 / 70
    ├── Microsoft version information
    └── Valid Authenticode signature
    │
    ▼
Evidence Correlation
    │
    └── No confirmed malicious activity established
```

## Key Temporal Findings

### File History

The file's creation and last-write timestamps were recorded on 09-09-2026.

### Execution

Two Sysmon Event ID 1 records confirmed execution on 21-09-2026.

### Network Activity

Multiple network events occurred during the same broader period, but the available evidence did not establish that `notepad.exe` generated those connections.

### SIEM Visibility

Wazuh was receiving Sysmon telemetry, but a specific hash-based Wazuh result for the investigated file was not established.

## Final Timeline Assessment

The timeline confirms that the investigated file existed on the endpoint and was executed twice during the observed period.

External reputation and local file validation did not identify malicious indicators in the captured evidence.

Network activity was present on the endpoint, but process attribution was insufficient to connect it to `notepad.exe`.

The timeline therefore supports an evidence-based assessment without converting temporal correlation into unsupported causation.

## Investigation Principle

> A timeline should show what happened, when it happened, and what evidence connects the events — not simply place unrelated events next to each other.
