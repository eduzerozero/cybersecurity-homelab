##SSH Brute-Force Detection – Wazuh SIEM

#Objective

Detect repeated failed SSH authentication attempts and investigate the resulting Wazuh security alerts.

#Lab Environment

SIEM: Wazuh 4.14.9

SIEM Server: Ubuntu Server 24.04 LTS

Monitored Endpoint: Raspberry Pi running Debian 13

Wazuh Agent: 4.14.9

Test Client: Windows PowerShell

#Test Scenario

Multiple failed SSH authentication attempts were generated against the Raspberry Pi using a non-existent test account.

The test was performed in an authorized homelab environment.

#Detection Rule

The Wazuh SSH brute-force detection rule was examined before testing.

Parameter

Value

Rule ID

5712

Severity

Level 10

Frequency

8

Timeframe

120 seconds

Source IP correlation

Enabled

Referenced rule

5710

#Detection Results

Wazuh successfully identified repeated SSH authentication failures and generated a brute-force detection alert.

#Detected alert:

Rule ID: 5712

Severity: Level 10

Agent: raspberry-pi

Timestamp: October 22, 2026, 16:37:49

Additional observed alerts included:

Rule 5710 – SSH login attempt using a non-existent user

Rule 5503 – PAM authentication failure

Rule 2502 – Multiple incorrect password attempts

#Evidence

The following screenshot shows the security events detected by Wazuh during the controlled test.



#Analysis

Wazuh correlated repeated SSH authentication failures originating from the test activity.

The SIEM generated a Level 10 alert when the configured detection conditions were met.

This demonstrates how event correlation can help distinguish repeated suspicious authentication activity from isolated login failures.

#Conclusion

The test successfully validated SSH brute-force detection using Wazuh SIEM.

Key learning outcomes:

Understanding SSH authentication logs

Analyzing Wazuh detection rules

Investigating security alerts

Understanding event correlation

Documenting security detection scenarios
