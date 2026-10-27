# SSH Brute-Force Detection and Automated Response with Wazuh

## 1. Project Overview

This project demonstrates the detection and automated mitigation of SSH brute-force activity using Wazuh SIEM and Active Response.

A controlled SSH attack simulation was performed from Kali Linux against a Raspberry Pi running Debian 13. Wazuh detected repeated authentication failures and automatically blocked the source IP address using an iptables firewall rule.

The block was automatically removed after approximately 60 seconds.

### Objectives

- Detect repeated failed SSH authentication attempts.
- Trigger Wazuh detection rule 5712.
- Automatically block the source IP using Active Response.
- Verify that the firewall rule is removed after the configured timeout.
- Preserve administrative access during testing.

## 2. Lab Environment

| Component | Operating System / Software | IP Address |
|---|---|---|
| SIEM Server | Ubuntu Server 24.04, Wazuh 4.14.9 | 192.168.178.25 |
| Protected Endpoint | Raspberry Pi, Debian 13, Wazuh Agent 4.14.9 | 192.168.178.162 |
| Test Machine | Kali Linux | 192.168.178.174 |
| Administrative Client | Windows PC | 192.168.178.124 |

The Wazuh server and Kali Linux were hosted in VirtualBox. Kali Linux used a bridged network adapter to obtain its own IP address on the local network.

All tests were performed within an authorized home laboratory.

## 3. Detection Rule

Wazuh rule 5712 was used to detect repeated SSH authentication attempts involving nonexistent users.

**Rule details:**

- Rule ID: 5712
- Severity level: 10
- Frequency: 8 matching events
- Timeframe: 120 seconds
- Source IP correlation: Enabled
- MITRE ATT&CK: T1110 – Brute Force

The rule identifies multiple failed SSH authentication attempts originating from the same source IP address.

## 4. Active Response Configuration

The following Active Response configuration was added to the Wazuh manager configuration file:

**File:** `/var/ossec/etc/ossec.conf`

```xml
<active-response>
  <disabled>no</disabled>
  <command>firewall-drop</command>
  <location>defined-agent</location>
  <agent_id>001</agent_id>
  <rules_id>5712</rules_id>
  <timeout>60</timeout>
</active-response>
```

The configuration instructs Wazuh to execute the `firewall-drop` command on the Raspberry Pi when rule 5712 is triggered.

The following administrative IP addresses were added to the manager's Active Response allowlist:

```xml
<white_list>192.168.178.124</white_list>
<white_list>192.168.178.25</white_list>
```

The configuration was validated and the Wazuh manager was restarted.

## 5. Attack Simulation

A controlled series of SSH authentication attempts was generated from Kali Linux against the Raspberry Pi using a nonexistent username.

Example command:

```bash
ssh -o PreferredAuthentications=none \
    -o PubkeyAuthentication=no \
    -o ConnectTimeout=5 \
    testwazuh@192.168.178.162
```

The attempts generated authentication failure events in the Raspberry Pi's SSH logs.

Example log entry:

```text
Invalid user testwazuh from 192.168.178.174
```

Wazuh correlated the events and triggered detection rule 5712.

## 6. Automated IP Blocking

At 16:40:07 on October 27, 2026, Wazuh Active Response executed the `firewall-drop` command.

The Raspberry Pi's firewall contained the following rule:

```text
Chain INPUT (policy ACCEPT)
num  target  prot  opt  source           destination
1    DROP    all   --   192.168.178.174  0.0.0.0/0
```

This rule dropped incoming IPv4 packets originating from the Kali Linux test machine.

The block was confirmed using:

```bash
sudo iptables -L INPUT -n --line-numbers
```

## 7. Automatic Unblocking

At 16:41:08, Wazuh executed the Active Response `delete` command.

The firewall was checked again:

```bash
sudo iptables -L INPUT -n --line-numbers
```

The previously inserted DROP rule was no longer present.

The Active Response log confirmed both operations:

```text
2026/10/27 16:40:07 firewall-drop: command "add"
2026/10/27 16:41:08 firewall-drop: command "delete"
```

These are abbreviated representations of the recorded JSON events.

**Observed blocking interval:** Approximately 61 seconds.

The configured timeout was 60 seconds.

## 8. Results

| Test | Result |
|---|---|
| SSH authentication failures logged | Passed |
| Wazuh rule 5712 triggered | Passed |
| Active Response executed | Passed |
| Kali source IP blocked | Passed |
| Firewall rule automatically removed | Passed |
| Administrative SSH client excluded from test | Passed |

The experiment demonstrated that Wazuh can detect SSH brute-force activity and execute an automated, temporary firewall response.

## 9. Security Considerations

- Administrative IP addresses were added to the Wazuh allowlist before enabling Active Response.
- The test machine used a separate IP address from the administrative client.
- The blocking timeout was limited to 60 seconds.
- The firewall configuration was backed up before testing.
- The experiment targeted the Raspberry Pi's host INPUT chain, not all Docker-forwarded traffic.
- Automated blocking can cause unintended service disruption if allowlists and detection rules are configured incorrectly.

## 10. Lessons Learned

This project provided practical experience in:

- Security monitoring with Wazuh SIEM.
- Linux SSH log analysis.
- Detection rule correlation.
- Automated incident response.
- Linux firewall management using iptables.
- VirtualBox networking and bridged adapters.
- Security testing and validation.
- Troubleshooting and operational risk management.

The main takeaway is that automated security responses should be tested carefully to ensure that they mitigate suspicious activity without disrupting legitimate administrative access.
