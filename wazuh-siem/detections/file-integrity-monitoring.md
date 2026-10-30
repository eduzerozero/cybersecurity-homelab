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

##File Integrity Monitoring (FIM) with Wazuh

#Overview

This project demonstrates real-time File Integrity Monitoring (FIM) using Wazuh and Linux Audit.

The objective is to detect unauthorized file modifications and deletions and identify the user and process responsible for these changes.

#Lab Environment

Component

Technology

SIEM

Wazuh 4.14.9

SIEM Server

Ubuntu Server 24.04

Monitored Endpoint

Raspberry Pi

Wazuh Agent

Agent ID 001

Audit Framework

Linux auditd

Monitoring Directory

/home/zerozero/fim-lab

#1. FIM Configuration

File Integrity Monitoring was enabled in the Wazuh agent configuration:

File: /var/ossec/etc/ossec.conf

<directories check_all="yes" realtime="yes">/home/zerozero/fim-lab</directories>

This configuration enabled real-time detection of file changes in the monitored directory.

#2. File Modification Detection

A test file was created and modified to generate a FIM event.

echo "FIM Test - Original Content" > /home/zerozero/fim-lab/test.txt
echo "FIM Test - Modified Content" > /home/zerozero/fim-lab/test.txt

Wazuh successfully detected the file modification.

Detection Results:

Rule ID: 550

Severity Level: 7

Description: Integrity checksum changed.

Event: modified

Monitoring Mode: realtime

Wazuh identified changes to file attributes and cryptographic hashes.

#3. File Deletion Detection

The monitored file was deleted using:

rm /home/zerozero/fim-lab/test.txt

Wazuh generated a file deletion alert.

Detection Results:

Rule ID: 553

Severity Level: 7

Description: File deleted.

Event: deleted

The event was mapped to MITRE ATT&CK techniques:

T1070.004 – File Deletion

T1485 – Data Destruction

#4. Who-data Monitoring

To identify the responsible user and process, Who-data monitoring was enabled.

Updated configuration:

<directories check_all="yes" whodata="yes">/home/zerozero/fim-lab</directories>

After restarting the Wazuh agent, the following audit rule was automatically registered:

-w /home/zerozero/fim-lab -p wa -k wazuh_fim

This allows Wazuh to collect additional audit information about file operations.

#5. Who-data Detection Test

A new test file was created and deleted:

echo "Who-data Test" > /home/zerozero/fim-lab/whodata-test.txt
rm /home/zerozero/fim-lab/whodata-test.txt

Wazuh successfully identified the file deletion and attributed it to the responsible user and process.

Detection Results:

Attribute

Result

Rule ID

553

Severity

Level 7

Event

deleted

Monitoring Mode

whodata

User

zerozero

User ID

1000

Process

/usr/bin/rm

Process ID

75471

Parent Process

/usr/bin/bash

Agent

raspberry-pi

The collected audit data included:

{
  "user": {
    "id": "1000",
    "name": "zerozero"
  },
  "process": {
    "id": "75471",
    "name": "/usr/bin/rm",
    "parent_name": "/usr/bin/bash"
  }
}

#6. Detection Evidence

File Deletion Alert



Who-data Process Attribution



#7. Security Relevance

File Integrity Monitoring is useful for detecting potentially unauthorized changes to sensitive files.

Who-data monitoring provides additional forensic context by identifying:

The user responsible for a file operation

The process that performed the operation

The parent process

The affected file

This information supports incident investigation, system monitoring, and security auditing.

Note: These alerts were generated through controlled tests in an isolated homelab environment. They demonstrate detection capabilities and do not indicate an actual security compromise.

#8. Conclusion

The implementation successfully demonstrated:

Real-time file modification detection

File deletion detection

Linux auditd integration

User and process attribution through Who-data

Security event analysis in Wazuh

This project demonstrates practical experience with SIEM monitoring, Linux auditing, endpoint security, and incident investigation.
