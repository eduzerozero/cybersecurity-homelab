# Windows Brute-Force Detection

## Overview

This detection scenario focuses on identifying repeated failed Windows authentication attempts using Wazuh SIEM.

A Windows 11 endpoint was integrated into the homelab and monitored through the Wazuh Agent.

The objective was to develop, configure, and validate custom detection rules for failed authentication attempts and potential brute-force attacks.

## Lab Environment

| Component | Technology |
|---|---|
| SIEM Platform | Wazuh 4.14.10 |
| SIEM Server | Ubuntu 24.04 LTS |
| Monitored Endpoint | Windows 11 |
| Windows Agent | Wazuh Agent 002 |
| Log Source | Windows Security Event Log |
| Authentication Event | Event ID 4625 |

## Detection Rules

### Rule 100200 – Failed Windows Authentication

- Severity: Level 8
- Trigger: Windows Security Event ID 4625
- Purpose: Detect individual failed Windows authentication attempts

### Rule 100201 – Potential Brute-Force Attack

- Severity: Level 10
- Threshold: 5 failed authentication attempts
- Timeframe: 60 seconds
- Correlation: Same target username
- MITRE ATT&CK: T1110 – Brute Force

The correlation uses the Windows event field `win.eventdata.targetUserName` to group authentication failures by target account.

## Test Procedure

A dedicated local Windows test account named `wazuh-test` was created.

Failed authentication attempts were generated in a controlled lab environment using PowerShell and deliberately incorrect credentials.

The following steps were performed:

1. Confirmed that failed authentication attempts generated Windows Security Event ID 4625.
2. Verified that Wazuh Rule 100200 detected individual authentication failures.
3. Generated five failed authentication attempts within 60 seconds.
4. Confirmed that Wazuh Rule 100201 generated a Level 10 alert.
5. Improved Rule 100201 using `same_field` to correlate events by target username.
6. Added MITRE ATT&CK technique T1110.
7. Repeated the positive detection test successfully.

## Results

| Validation | Result |
|---|---|
| Windows Event ID 4625 generated | Passed |
| Wazuh Rule 100200 triggered | Passed |
| Initial Rule 100201 triggered | Passed |
| Updated Rule 100201 triggered | Passed |
| MITRE ATT&CK T1110 present | Passed |
| Different-account correlation test | Pending |

The updated rule generated a Level 10 alert on November 21, 2026, at 13:42:22 UTC.

The alert identified the target account `wazuh-test` and included the MITRE ATT&CK technique T1110.

## Limitations

The positive test confirmed that repeated authentication failures for the same account trigger the updated rule.

A negative test using multiple different target accounts has not yet been performed. Therefore, the effectiveness of the account-based correlation in preventing false positives remains to be validated.

The test events originated locally from the monitored Windows endpoint. They do not represent an external network attack.

## Future Improvements

- Validate correlation behavior across different Windows accounts.
- Investigate false-positive scenarios.
- Extend Windows authentication monitoring.
- Evaluate additional MITRE ATT&CK mappings.
- Explore alert enrichment and incident investigation workflows.

## Disclaimer

All tests were performed in an authorized personal cybersecurity homelab for educational purposes.
