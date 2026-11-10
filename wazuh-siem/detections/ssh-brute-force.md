##SSH Brute-Force Detection with Wazuh SIEM

#Overview

This project demonstrates how to detect potential SSH brute-force attacks using Wazuh SIEM.

A custom detection rule was developed to identify multiple failed SSH login attempts originating from the same IP address within a short period.

The detection was successfully validated using Wazuh Logtest and real SSH authentication events.

#Lab Environment

SIEM: Wazuh 4.14.9

Wazuh Manager: Ubuntu Server

Monitored Endpoint: Raspberry Pi

Wazuh Agent ID: 001

Log Source: systemd-journald

Detection Rule: 100106

#Detection Rule

The following custom Wazuh rule detects repeated failed SSH login attempts involving non-existent users.

<group name="sshd,brute_force,">
  <rule id="100106" level="10" frequency="5" timeframe="60">
    <if_matched_sid>5710</if_matched_sid>
    <same_srcip />
    <description>Possible SSH brute-force attack: multiple failed logins from the same IP</description>
    <group>authentication_failed,brute_force,linux_security,</group>
  </rule>
</group>

#Rule Explanation

Rule ID 100106: Custom SSH brute-force detection

Level 10: High-severity security alert

Frequency 5: Correlates five matching events

Timeframe 60: Events must occur within 60 seconds

Rule 5710: Detects SSH login attempts involving non-existent users

same_srcip: Correlates events originating from the same source IP address

#Testing Procedure

Verified that the Raspberry Pi was connected to the Wazuh Manager.

Confirmed that SSH authentication events were collected through journald.

Tested the custom detection rule using wazuh-logtest.

Generated controlled failed SSH login attempts from a Windows computer.

Monitored Wazuh security alerts in alerts.json.

Results

The custom detection rule successfully generated Level 10 security alerts during live testing.

Observed results:

Rule ID: 100106

Severity: 10

Source IP: 192.168.178.124

Target: Raspberry Pi

Test username: wazuh-test

Log source: journald

Two Level 10 alerts were observed during the live test.

#Limitations

The current rule focuses on authentication attempts involving non-existent SSH users. It does not cover every type of SSH brute-force attack.

Additional detection rules could be implemented to monitor repeated failed authentication attempts against existing user accounts.

#Conclusion

This project demonstrates practical experience in:

Linux security monitoring

SSH authentication log analysis

Wazuh custom rule development

Event correlation

SIEM alert investigation

Security detection testing

The detection was successfully tested in a controlled home lab environment.
