# file-hash-reputation-investigation-lab

## Overview

A file hash is a cryptographic identifier calculated from a file's contents. In SOC/DFIR work, hashes are useful for determining whether the exact same file has already been observed in threat-intelligence sources.

For this lab, use SHA256 as the primary identifier and optionally collect MD5/SHA1 for compatibility with older tooling.

The investigation should follow this sequence:

File
  ↓
SHA256
  ↓
Reputation lookup
  ↓
Detection/vendor results
  ↓
Historical/contextual information
  ↓
Local endpoint telemetry
  ↓
Evidence-based assessment

A hash reputation result is supporting intelligence, not a complete verdict. A file with no detections is not automatically benign, and a single detection should not automatically be treated as confirmed malware.

VirusTotal currently supports searching for an existing file report by MD5, SHA1, or SHA256, so a hash lookup can be performed without re-uploading the file.

This lab focuses on investigating a potentially suspicious Windows executable using cryptographic hashes, file metadata, external reputation intelligence, digital signature validation, and local endpoint telemetry.

The investigation follows an evidence-driven approach:

```text
File Identification
        ↓
File Metadata
        ↓
SHA256 / SHA1 / MD5
        ↓
VirusTotal Reputation
        ↓
File Version & Digital Signature
        ↓
Sysmon Execution Evidence
        ↓
Endpoint Network Telemetry
        ↓
Wazuh Visibility Check
        ↓
Evidence Correlation
        ↓
Final Assessment
```

The objective is not to treat a hash reputation result as a standalone verdict. External intelligence is compared with local endpoint evidence to determine what can actually be confirmed.

# Lab Objectives

- Identify a Windows executable and establish its basic file properties, including path, size, and timestamps.
- Calculate and document SHA256, SHA1, and MD5 hashes to create reliable identifiers for the investigated file.
- Use the SHA256 hash to perform reputation-based investigation without uploading the executable itself.
- Review external reputation results, including vendor detections, known filenames, file type, tags, and community information.
- Validate the local file's version information and publisher details to determine whether they are consistent with the expected executable.
- Verify the Authenticode digital signature and understand how signature status contributes to file assessment.
- Determine whether the investigated executable was actually executed by examining Sysmon Event ID 1 process creation telemetry.
- Review Sysmon Event ID 3 network telemetry while avoiding unsupported attribution of network activity to the investigated process.
- Check Wazuh telemetry for visibility of the investigated file, hash, and related Sysmon events.
- Correlate file reputation, local validation, execution evidence, network telemetry, and SIEM visibility into a single investigation.
- Distinguish confirmed evidence from assumptions, missing telemetry, and unconfirmed relationships between events.
- Document the investigation in a way that supports an evidence-based assessment rather than relying on a single reputation result or indicator.


# Lab Scenario

A Windows endpoint has been selected for a file investigation after a system executable is identified for validation. The SOC analyst needs to determine whether the file is consistent with an expected Windows executable or whether there is evidence requiring further investigation.

The investigation begins with the file itself and gradually expands into external intelligence and endpoint telemetry.

The analyst is provided with the following file:

```text
C:\Windows\System32\notepad.exe
```

The investigation should establish the identity and characteristics of the file before making any assessment. This includes collecting metadata, calculating cryptographic hashes, reviewing file version information, and validating the digital signature.

The SHA256 hash is then used for an external reputation lookup. The analyst should review vendor detections, known filenames, file type, tags, and other available reputation information while treating the external result as supporting evidence rather than a final verdict.

After validating the file, the analyst investigates whether it was actually executed on the endpoint. Sysmon process creation telemetry is reviewed to identify execution events associated with `notepad.exe`.

Network telemetry is also examined to determine whether relevant connections occurred around the same period. The analyst must avoid assuming that network activity belongs to the investigated process unless the available telemetry provides sufficient process-level attribution.

Wazuh telemetry is reviewed as an additional source of endpoint evidence. Searches are performed using the filename, file path, and SHA256 hash where appropriate.

If specific telemetry or alerts cannot be confirmed, the limitation should be documented rather than treated as evidence that no activity occurred.

The final assessment should correlate:

- File metadata
- Cryptographic hashes
- External reputation
- File version information
- Digital signature status
- Process execution telemetry
- Network telemetry
- Wazuh visibility

The investigation should clearly separate confirmed observations from assumptions, missing evidence, and unconfirmed relationships.

The scenario is designed to demonstrate a practical SOC workflow where a file hash is only the starting point and the final assessment is based on multiple independent evidence sources.


## Lab Environment

| Component | Details |
|---|---|
| Operating System | Windows 10 Pro 22H2 |
| PowerShell | 7.6.6 |
| Sysmon | 4.91 |
| Wazuh Agent | 4.12.0 |
| Investigation Directory | `C:\FileHashReputationLab` |
| Evidence Directory | `C:\FileHashReputationLab\Evidence` |
| Sample Directory | `C:\FileHashReputationLab\Sample` |

## Investigated File

```text
File:
C:\Windows\System32\notepad.exe
```

### File Metadata

```text
File Size:       360448 bytes
Approx. Size:    352.00 KB
Creation Time:   09-09-2026 09:56:13
Last Write Time: 09-09-2026 09:56:13
```

The file was examined as a known Windows system executable and was not modified or uploaded during the investigation.

## Cryptographic Hashes

| Hash Type | Value |
|---|---|
| SHA256 | `468FFE129C395ABFB6B21A09EFDF261910A95FB98AA982EAD73CAA7B2B684577E` |
| SHA1 | `76CD26B59923157E09D2BC927BA8FB059F3155DC` |
| MD5 | `8A1D8175CCCA97054CDB25ACBB4CC07E` |

SHA256 was treated as the primary file identifier for reputation investigation.

## VirusTotal Findings

The SHA256 hash was checked using VirusTotal hash-based reputation lookup.

Observed results:

```text
Detection:        0 / 70
Community Score:  0
File Type:        PE32+ executable, 64-bit
File Size:        352.00 KB
Known Filename:   NOTEPAD.EXE
Tags:             peexe, 64bits
Last Analysis:    1 day ago
```

The captured VirusTotal result stated that no security vendors flagged the file as malicious.

This result indicates that no malicious detection was observed in the captured reputation data. It does not independently prove that a file is safe in every environment.

## Local File Validation

The local file metadata and version information were reviewed.

```text
File Version:
10.0.26100.8457 (WinBuild.160101.0800)

Product Name:
Microsoft® Windows® Operating System

Product Version:
10.0.26100.8457

Company:
Microsoft Corporation

Original Filename:
NOTEPAD.EXE.MUI
```

## Digital Signature Validation

Authenticode validation returned:

```text
Status:
Valid

Status Message:
Signature verified.
```

The valid Microsoft digital signature provides additional local evidence supporting the expected identity of the executable.

## Sysmon Execution Evidence

Sysmon Event ID 1 was searched for the investigated filename.

Observed execution events:

```text
21-09-2026 07:41:24
21-09-2026 07:28:14
```

This confirms that `notepad.exe` was observed in process creation telemetry twice during the investigation period.

The presence of a process creation event confirms execution activity, but execution alone does not indicate malicious behavior.

## Network Telemetry

Sysmon Event ID 3 telemetry was also reviewed.

Multiple network connection events were present during the investigation period. However, the available output did not provide sufficient process-level attribution to determine that these connections originated from `notepad.exe`.

Therefore, the network activity was not attributed to the investigated executable.

## Wazuh Visibility

Wazuh telemetry was checked as an additional endpoint evidence source.

A Sysmon Event ID 1 record was visible in Wazuh, confirming that Sysmon process telemetry was being ingested.

However, the provided evidence did not establish a specific Wazuh event containing the investigated SHA256 hash or a confirmed alert associated with `notepad.exe`.

Therefore, Wazuh visibility of the specific file was treated as unconfirmed rather than assumed.

## Evidence Summary

| Evidence | Observation | Interpretation |
|---|---|---|
| File Path | `C:\Windows\System32\notepad.exe` | Windows system location |
| SHA256 | `468FFE...684577E` | Unique file identifier |
| VirusTotal | `0 / 70` | No vendor detections observed |
| Community Score | `0` | No negative community score observed |
| File Version | `10.0.26100.8457` | Windows version information |
| Company | Microsoft Corporation | Consistent with expected system file |
| Digital Signature | Valid | Signature verification succeeded |
| Sysmon EID 1 | Two events | Execution confirmed |
| Sysmon EID 3 | Multiple events | Network activity observed, attribution not established |
| Wazuh | Sysmon telemetry visible | Specific file/hash visibility not confirmed |

## Evidence-Based Assessment

The collected evidence is consistent with an expected Windows system executable. The file had a valid digital signature, Microsoft version information, and a VirusTotal result of `0 / 70` detections.

Execution of the file was confirmed through Sysmon Event ID 1. However, the available network telemetry did not establish that the observed network activity belonged to `notepad.exe`.

No evidence in the captured results established malicious behavior.

The appropriate assessment is therefore based on the evidence available during the investigation rather than on the filename or reputation result alone.


## Limitations

- VirusTotal results represent the reputation state captured during the investigation.
- A `0 / 70` result does not guarantee that a file is universally safe.
- Wazuh evidence did not establish a confirmed hash-specific alert.
- Network events could not be confidently attributed to `notepad.exe`.
- File access timestamps were not treated as proof of execution or modification.
- External reputation was correlated with local evidence rather than used as the sole decision point.

