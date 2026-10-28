# Linux Sudo Execution Detection with Wazuh and Auditd

## Overview

This project demonstrates how to detect the execution of `sudo` on a Linux system using Linux Auditd and a custom Wazuh detection rule.

The goal is to collect command execution events from a Raspberry Pi, forward them to a centralized Wazuh SIEM server, and generate security alerts when monitored activity occurs.

**Detection status:** Successfully tested, including after a Raspberry Pi reboot.

## Lab Environment

| Component | Technology | Purpose |
|---|---|---|
| SIEM Server | Wazuh 4.14.9 on Ubuntu Server 24.04 | Centralized monitoring and detection |
| Endpoint | Raspberry Pi running Debian Linux | Monitored Linux system |
| Audit Framework | Linux Auditd | Command execution monitoring |
| Endpoint Agent | Wazuh Agent | Log collection and forwarding |
| Detection Engine | Wazuh Manager | Custom rule evaluation |

## Detection Architecture

```text
Raspberry Pi
    |
    | sudo command execution
    v
Linux Auditd
    |
    | /var/log/audit/audit.log
    v
Wazuh Agent
    |
    | Audit event forwarding
    v
Wazuh Manager
    |
    | Custom Rule 100100
    v
Security Alert (Level 5)
```

## 1. Configure Linux Auditd

Install and enable Auditd on the monitored Linux endpoint.

Create a persistent audit rule in:

`/etc/audit/rules.d/sudo_execution.rules`

```bash
-w /usr/bin/sudo -p x -k sudo_execution
```

This rule monitors execution of `/usr/bin/sudo` and assigns the audit key `sudo_execution`.

Load the persistent rules:

```bash
sudo augenrules --load
```

Verify the active rules:

```bash
sudo auditctl -l
```

Expected output:

```text
-w /usr/bin/sudo -p x -k sudo_execution
```

## 2. Configure Wazuh Agent Log Collection

On the Raspberry Pi, configure the Wazuh Agent to collect Auditd events.

File: `/var/ossec/etc/ossec.conf`

```xml
<localfile>
  <log_format>audit</log_format>
  <location>/var/log/audit/audit.log</location>
</localfile>
```

Restart the Wazuh Agent:

```bash
sudo systemctl restart wazuh-agent
```

Audit events are now forwarded to the Wazuh Manager.

## 3. Create the Custom Detection Rule

On the Wazuh Manager, edit:

`/var/ossec/etc/rules/local_rules.xml`

Add the following rule:

```xml
<group name="auditd,linux_command_execution,">
  <rule id="100100" level="5">
    <if_sid>80700</if_sid>
    <field name="audit.key">^sudo_execution$</field>
    <description>Custom detection: sudo execution monitored by auditd</description>
    <group>auditd,sudo_execution,</group>
  </rule>
</group>
```

### Rule Explanation

| Element | Purpose |
|---|---|
| `id="100100"` | Unique custom rule identifier |
| `level="5"` | Security alert severity |
| `<if_sid>80700</if_sid>` | Matches events processed by the Wazuh Auditd parent rule |
| `audit.key` | Matches the configured Auditd key |
| `description` | Describes the detected activity |

Validate the configuration:

```bash
sudo /var/ossec/bin/wazuh-analysisd -t
```

Restart the Wazuh Manager:

```bash
sudo systemctl restart wazuh-manager
```

## 4. Detection Testing

Execute the following command on the Raspberry Pi:

```bash
sudo whoami
```

Expected output:

```text
root
```

This triggers an Auditd event for the monitored `sudo` executable.

On the Wazuh Manager, inspect the generated alerts:

```bash
sudo grep '"id":"100100"' /var/ossec/logs/alerts/alerts.json | tail -1 | python3 -m json.tool
```

### Observed Detection Result

| Field | Observed value |
|---|---|
| Rule ID | `100100` |
| Rule Level | `5` |
| Agent Name | `raspberry-pi` |
| Audit Command | `sudo` |
| Command Argument | `whoami` |
| Audit Key | `sudo_execution` |
| Effective UID | `0` |
| Decoder | `auditd` |

The Wazuh Manager successfully generated an alert for the execution of `sudo whoami`.

## 5. Persistence Testing

To verify that the Auditd rule survives a system reboot, restart the Raspberry Pi:

```bash
sudo reboot
```

After reconnecting, check the active audit rules:

```bash
sudo auditctl -l
```

The `sudo_execution` rule remained active.

A subsequent `sudo whoami` execution generated another Wazuh alert with rule ID `100100`.

**Result: Persistence and end-to-end detection successfully verified.**

## 6. Security Considerations and Limitations

- The rule monitors execution of `/usr/bin/sudo`, not every Linux command.
- Legitimate administrative activity also generates alerts.
- Alert Level 5 does not automatically mean that malicious activity occurred.
- The current configuration provides detection and logging, not automatic prevention.
- Production deployments may require additional filtering, tuning, and monitoring to reduce alert noise.

## 7. Key Learnings

This project provided hands-on experience with:

- Linux Auditd and persistent audit rules
- Wazuh Agent log collection
- Centralized security event monitoring
- Custom Wazuh XML detection rules
- Rule chaining with `if_sid`
- Detection validation using `wazuh-logtest`
- Troubleshooting XML and duplicate rule IDs
- End-to-end SIEM testing and verification

## Conclusion

A custom Wazuh detection rule was successfully implemented to monitor `sudo` execution on a Raspberry Pi.

The detection was validated using real Auditd events, Wazuh alert logs, and a reboot persistence test.

This project demonstrates a practical approach to Linux endpoint monitoring and custom SIEM detection engineering in a home lab.
