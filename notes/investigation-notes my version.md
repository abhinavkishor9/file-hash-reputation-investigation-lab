# Investigation Notes

## 1. File Identification

Investigated file:

```text
C:\Windows\System32\notepad.exe
```

The file was located under the Windows System32 directory.

Basic metadata was collected before performing reputation analysis.

```text
Size:            360448 bytes
Approx. Size:    352.00 KB
Creation Time:   09-09-2026 09:56:13
Last Write Time: 09-09-2026 09:56:13
```

The last access timestamp was observed separately and was not interpreted as modification evidence.

## 2. Cryptographic Hash Calculation

Three cryptographic hashes were calculated.

```text
SHA256:
468FFE129C395ABFB6B21A09EFDF261910A95FB98AA982EAD73CAA7B2B684577E

SHA1:
76CD26B59923157E09D2BC927BA8FB059F3155DC

MD5:
8A1D8175CCCA97054CDB25ACBB4CC07E
```

SHA256 was used as the primary identifier because it provides a strong and commonly used method for identifying an exact file.

## 3. Reputation Investigation

The SHA256 value was submitted to VirusTotal as a hash lookup.

Observed result:

```text
Detection:
0 / 70

Community Score:
0

File Type:
PE32+ executable, 64-bit

Size:
352.00 KB

Known Filename:
NOTEPAD.EXE

Tags:
peexe
64bits
```

No security vendors flagged the file as malicious in the captured result.

This was treated as reputation evidence rather than a final determination.

## 4. Filename and Metadata Comparison

Known filenames associated with the reputation result included:

```text
NOTEPAD.EXE
notepad.exe
```

The local filename was:

```text
notepad.exe
```

The local file metadata was also reviewed to determine whether its version information was consistent with the expected Windows executable.

## 5. File Version Information

The local version information showed:

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

These values were consistent with the expected identity of a Windows system executable.

## 6. Digital Signature Validation

Authenticode validation returned:

```text
Status:
Valid

Status Message:
Signature verified.
```

The signature result provided additional local evidence supporting the file's expected publisher and integrity.

A valid signature was not treated as proof that the executable could never be abused. It was considered one piece of the overall evidence set.

## 7. Process Execution Investigation

Sysmon Event ID 1 was searched for `notepad.exe`.

Two process creation events were identified:

```text
21-09-2026 07:41:24
21-09-2026 07:28:14
```

This confirms that the investigated executable was executed during the observed period.

The result answers an important investigation question:

```text
Was the file executed?
Yes.
```

However:

```text
Was the execution malicious?
Not established by the available evidence.
```

## 8. Network Telemetry Investigation

Sysmon Event ID 3 was reviewed for endpoint network activity.

Multiple network events were present during the investigation period.

However, the available event output did not establish sufficient process-level attribution to associate those connections specifically with `notepad.exe`.

Therefore:

```text
Network activity observed:
Yes

Network activity attributed to notepad.exe:
Not established
```

This distinction prevents unrelated endpoint network activity from being incorrectly associated with the investigated process.

## 9. Wazuh Investigation

Wazuh telemetry was reviewed to determine whether the endpoint telemetry was available through the SIEM.

A Sysmon Event ID 1 record was visible in Wazuh.

The following searches were considered for the specific investigation:

```text
468FFE129C395ABFB6B21A09EFDF261910A95FB98AA982EAD73CAA7B2B684577E

notepad.exe

C:\Windows\System32\notepad.exe
```

The available evidence did not establish a confirmed Wazuh event containing the investigated SHA256 hash.

Therefore, no hash-specific Wazuh alert was claimed.

## 10. Evidence Correlation

| Evidence Source | Result | Confidence |
|---|---|---|
| File Path | Windows System32 location | High |
| SHA256 | Unique hash calculated | High |
| VirusTotal | 0 / 70 detections | High |
| File Metadata | Windows executable metadata | High |
| Version Information | Microsoft Windows information | High |
| Authenticode | Valid signature | High |
| Sysmon EID 1 | Two executions observed | High |
| Sysmon EID 3 | Network events observed | Medium |
| Network Attribution | Not established for notepad.exe | High |
| Wazuh | Sysmon telemetry visible | Medium |
| Hash-specific Wazuh Alert | Not established | High |

## 11. Evidence-Based Assessment

The combined evidence did not show malicious activity associated with the investigated file.

The strongest supporting observations were:

- The file was located in the expected Windows System32 directory.
- The file contained Microsoft Windows version information.
- Authenticode validation was successful.
- VirusTotal reported `0 / 70` detections.
- Sysmon confirmed execution.
- No provided evidence established malicious behavior.
- Network activity could not be confidently attributed to the process.
- A specific Wazuh hash-based alert was not established.

The investigation therefore supports an assessment that the observed evidence is consistent with an expected Windows system executable, while recognizing that reputation and signature evidence should not be treated as absolute proof of safety.

