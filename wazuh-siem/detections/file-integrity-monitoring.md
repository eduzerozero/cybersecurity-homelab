# File Integrity Monitoring – Wazuh SIEM

## Objective

Configure and validate File Integrity Monitoring (FIM) using Wazuh to detect file modifications on a monitored Linux endpoint.

## Lab Environment

| Component | Technology |
|---|---|
| SIEM | Wazuh 4.14.9 |
| SIEM Server | Ubuntu Server 24.04 LTS |
| Monitored Endpoint | Raspberry Pi – Debian 13 |
| Agent | Wazuh Agent 4.14.9 |
| Monitoring Mode | Real-time |

## Configuration

A dedicated test directory was created on the Raspberry Pi:

`/home/zerozero/fim-lab`

The following configuration was added to the Wazuh agent's existing `<syscheck>` section:

```xml
<directories check_all="yes" realtime="yes">/home/zerozero/fim-lab</directories>
```

The Wazuh agent was restarted and the logs confirmed that real-time file monitoring had started.

## Test Scenario

A test file was created inside the monitored directory.

The file was modified using:

```bash
echo "Security test: File modified" >> ~/fim-lab/test.txt
```

The objective was to verify whether Wazuh would detect the change and generate a security alert.

## Detection Results

Wazuh successfully detected the file modification.

| Field | Result |
|---|---|
| Rule ID | 550 |
| Severity | Level 7 |
| Description | Integrity checksum changed |
| Agent | raspberry-pi |
| Event | modified |
| Monitoring Mode | realtime |
| MITRE ATT&CK | T1565.001 |

### File Integrity Analysis

The Wazuh alert recorded changes to the following attributes:

| Attribute | Before | After |
|---|---|---|
| File size | 21 bytes | 41 bytes |
| Modification time | 02:24:50 | 02:39:36 |
| MD5 | Original hash | Changed hash |
| SHA-1 | Original hash | Changed hash |
| SHA-256 | Original hash | Changed hash |

The changed hash values confirmed that the monitored file content had been modified.

## Evidence

The following screenshots show the Wazuh FIM alert and its detailed attributes.

![Wazuh FIM Alert Details](../screenshots/fim-alert-details.png)

![Wazuh FIM Hash Comparison](../screenshots/fim-hash-comparison.png)

## Security Analysis

File Integrity Monitoring can help identify unauthorized modifications to important system files.

In this controlled test, the modification was performed intentionally. The resulting Wazuh alert demonstrates the ability to detect changes and provide useful forensic information.

The event was mapped to MITRE ATT&CK technique T1565.001 – Stored Data Manipulation. This mapping describes a possible security-relevant behavior, not proof of malicious activity.

## Conclusion

The File Integrity Monitoring test was successful.

Key learning outcomes:

- Configuring Wazuh FIM
- Monitoring Linux directories in real time
- Detecting file modifications
- Analyzing file metadata and cryptographic hashes
- Investigating Wazuh security alerts
- Understanding MITRE ATT&CK mappings
