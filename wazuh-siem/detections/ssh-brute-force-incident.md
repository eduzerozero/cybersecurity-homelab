##Security Incident Report – SSH Brute-Force Detection

1. Incident Summary

On November 10, 2026, Wazuh SIEM detected multiple failed SSH authentication attempts targeting a Raspberry Pi in a controlled home lab environment.

The activity originated from a Windows workstation and was intentionally generated to validate a custom Wazuh detection rule.

2. Incident Details

Field

Value

Incident Type

Potential SSH Brute-Force

Detection Date

November 10, 2026

Detection Rule

100106

Severity

Level 10

Source IP

192.168.178.124

Destination IP

192.168.178.162

Target Host

raspberry-pi

Username

wazuh-test

Log Source

journald

Detection Status

Confirmed

Test Authorization

Controlled Home Lab

3. Detection and Analysis

The Wazuh agent collected SSH authentication events from systemd-journald and forwarded them to the Wazuh Manager.

A custom correlation rule identified repeated failed authentication attempts involving an invalid username from the same source IP address.

The rule was configured to correlate five matching events within a 60-second timeframe.

The Wazuh Dashboard displayed four alerts associated with Rule ID 100106 during the selected observation period.

Evidence

An example authentication failure recorded during the test:

Nov 10 09:27:47 zerozero sshd-session[213563]:
Failed password for invalid user wazuh-test
from 192.168.178.124 port 12850 ssh2

The event was decoded using the Wazuh SSH decoder and correlated with preceding authentication events.

4. Impact Assessment

The detected activity was generated intentionally for security testing.

No successful unauthorized login was identified in the reviewed evidence.

The alerts demonstrate that the SIEM can identify repeated SSH authentication failures associated with non-existent user accounts.

5. Recommended Mitigations

Disable password-based SSH authentication where appropriate and use SSH keys.

Restrict SSH access to trusted networks.

Implement rate limiting or automated blocking for repeated failed logins.

Monitor SSH authentication events continuously.

Review SSH security alerts and investigate suspicious source IP addresses.

6. Conclusion

The controlled test successfully validated the custom Wazuh SSH brute-force detection rule.

The project demonstrates practical experience with SIEM monitoring, log analysis, event correlation, security alert investigation, and incident documentation.
