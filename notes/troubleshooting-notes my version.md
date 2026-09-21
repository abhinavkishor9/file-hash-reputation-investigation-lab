# Troubleshooting Notes

## 1. Lab Directory Creation

### Problem

The investigation required separate locations for evidence and sample files.

### Resolution

Created the required directories:

```powershell
$LabPath = "C:\FileHashReputationLab"
$EvidencePath = "$LabPath\Evidence"
$SamplePath = "$LabPath\Sample"

New-Item -ItemType Directory -Path $EvidencePath -Force
New-Item -ItemType Directory -Path $SamplePath -Force
```

### Validation

```powershell
Test-Path $LabPath
Test-Path $EvidencePath
Test-Path $SamplePath
```

All required paths returned `True`.

---

## 2. File Path Validation

### Problem

Before hashing the file, the investigation needed to confirm that the target existed.

### Resolution

```powershell
$PEFile = "C:\Windows\System32\notepad.exe"

Test-Path $PEFile
```

The target file was available.

### Lesson

Always validate the target path before collecting hashes or metadata.

---

## 3. Hash Calculation

### Problem

Multiple hash algorithms were required for comparison and documentation.

### Resolution

PowerShell hash calculation commands were used for SHA256, SHA1, and MD5.

Observed values:

```text
SHA256:
468FFE129C395ABFB6B21A09EFDF261910A95FB98AA982EAD73CAA7B2B684577E

SHA1:
76CD26B59923157E09D2BC927BA8FB059F3155DC

MD5:
8A1D8175CCCA97054CDB25ACBB4CC07E
```

### Lesson

SHA256 should normally be the primary modern file identifier, while SHA1 and MD5 can still be useful for legacy references and correlation.

---

## 4. Evidence File Creation

### Problem

Investigation results needed to remain separate from the console output.

### Resolution

Evidence was stored under:

```text
C:\FileHashReputationLab\Evidence
```

Files included:

```text
01-file-metadata.txt
02-sha256.txt
03-sha1.txt
04-md5.txt
05-hash-record.txt
06-virustotal-reputation.txt
```

### Lesson

Separating evidence artifacts improves investigation reproducibility and makes later review easier.

---

## 5. VirusTotal Lookup

### Problem

The investigation required external reputation information without uploading the file.

### Resolution

The SHA256 hash was used for a hash-only VirusTotal lookup.

Observed result:

```text
Detection: 0 / 70
Community Score: 0
```

### Lesson

Hash-only lookup is preferable when investigating potentially sensitive organizational files because it avoids uploading the actual file.

Organizational policy should still be considered before using external services.

---

## 6. Interpreting a 0 / 70 Result

### Problem

A zero-detection result could be incorrectly interpreted as proof that the file is completely safe.

### Resolution

The result was treated as reputation evidence only.

The investigation continued with:

- File metadata
- Version information
- Digital signature
- Sysmon process creation
- Network telemetry
- Wazuh visibility

### Lesson

Reputation is one evidence source, not the entire investigation.

---

## 7. Authenticode Validation

### Problem

The file's publisher and signature status needed to be validated locally.

### Resolution

Authenticode validation returned:

```text
Status:
Valid

Status Message:
Signature verified.
```

### Lesson

Digital signature validation provides useful supporting evidence about file identity and integrity.

---

## 8. Sysmon Event ID 1 Search

### Problem

The investigation needed to determine whether the file had actually executed.

### Resolution

Sysmon Event ID 1 was filtered using the filename:

```powershell
Get-WinEvent -FilterHashtable @{
    LogName = "Microsoft-Windows-Sysmon/Operational"
    Id = 1
} -MaxEvents 500 |
Where-Object {
    $_.Message -match [regex]::Escape((Split-Path $PEFile -Leaf))
} |
Select-Object TimeCreated, Message
```

Observed events:

```text
21-09-2026 07:41:24
21-09-2026 07:28:14
```

### Lesson

The process creation events provide direct evidence that the executable was executed.

---

## 9. Sysmon Event ID 3 Investigation

### Problem

Network activity needed to be reviewed.

### Resolution

Sysmon Event ID 3 was queried:

```powershell
Get-WinEvent -FilterHashtable @{
    LogName = "Microsoft-Windows-Sysmon/Operational"
    Id = 3
} -MaxEvents 200 |
Select-Object TimeCreated, Message
```

Multiple network events were observed.

### Limitation

The available output did not provide sufficient process-level information to attribute the network activity specifically to `notepad.exe`.

### Lesson

Endpoint network activity should not be attributed to a process unless the telemetry supports that attribution.

---

## 10. Wazuh Search Limitations

### Problem

Wazuh showed Sysmon telemetry, but the available evidence did not establish a hash-specific result for the investigated file.

### Resolution

Potential searches included:

```text
468FFE129C395ABFB6B21A09EFDF261910A95FB98AA982EAD73CAA7B2B684577E

notepad.exe

C:\Windows\System32\notepad.exe
```

For archived Wazuh data, searches could also include:

```text
_index:wazuh-archives-* AND "468FFE129C395ABFB6B21A09EFDF261910A95FB98AA982EAD73CAA7B2B684577E"

_index:wazuh-archives-* AND "notepad.exe"
```

### Lesson

A missing SIEM result should be recorded as a visibility limitation rather than interpreted as proof that no activity occurred.

---

## 11. Timestamp Interpretation

### Problem

The file contained multiple timestamps, including access-related timestamps.

### Resolution

The investigation distinguished:

```text
Creation Time
Last Write Time
Last Access Time
Process Creation Time
```

The Last Access Time was not treated as proof of execution.

Sysmon Event ID 1 was used as the execution evidence.

### Lesson

Different timestamps represent different types of activity and should not be treated as interchangeable.

---

## 12. Reputation and Local Evidence Conflict

### Problem

External reputation and local endpoint evidence may sometimes disagree.

### Resolution

The investigation used a correlation approach:

```text
External Reputation
        +
File Metadata
        +
Digital Signature
        +
Execution Evidence
        +
Network Evidence
        +
SIEM / EDR Evidence
```

### Lesson

When evidence conflicts, document the conflict and identify what remains confirmed, plausible, or unknown.

---

## 13. Avoiding Unsupported Attribution

### Problem

Multiple Sysmon network events existed around the same period as `notepad.exe` execution.

### Incorrect Conclusion

```text
notepad.exe generated the network connections.
```

### Correct Approach

```text
Network activity was observed on the endpoint, but
process-level attribution to notepad.exe was not established.
```

### Lesson

Temporal proximity does not automatically establish causation or process attribution.

---

