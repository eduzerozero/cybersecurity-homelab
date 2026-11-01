Critical File Monitoring – Custom Wazuh Detection Rules

Overview

This project demonstrates how to monitor critical Linux system files using Wazuh File Integrity Monitoring (FIM) and custom detection rules.

The objective is to detect unauthorized file modifications and generate higher-severity security alerts for sensitive system files such as /etc/passwd.

The project was implemented in a personal cybersecurity homelab using a Raspberry Pi as the monitored endpoint and a Wazuh SIEM server.

Lab Environment

Component

Technology

SIEM Platform

Wazuh 4.14.9

SIEM Server

Ubuntu Server 24.04 (VirtualBox)

Monitored Endpoint

Raspberry Pi (Linux)

Wazuh Agent

Agent 001 – raspberry-pi

Detection Method

File Integrity Monitoring (FIM)

Custom Detection Rule

100101

Alert Severity

Level 10

Objective

The objective of this detection is to:

Monitor modifications to critical Linux system files.

Detect changes to /etc/passwd in real time.

Generate higher-severity alerts using custom Wazuh rules.

Validate detection logic through controlled testing.

Improve security monitoring and incident detection capabilities.

1. Configure File Integrity Monitoring

The Wazuh agent configuration is located at:

/var/ossec/etc/ossec.conf

The following configuration enables real-time monitoring of /etc/passwd:

<directories check_all="yes" realtime="yes">/etc/passwd</directories>

After applying the configuration, the Wazuh agent confirmed that real-time monitoring was enabled:

wazuh-syscheckd: INFO: (6016):
Directory set for real time monitoring: '/etc/passwd'.

2. Create a Custom Detection Rule

Custom Wazuh detection rules are configured on the Wazuh manager in:

/var/ossec/etc/rules/local_rules.xml

The following rule was created:

<group name="syscheck,critical_file_monitoring,">
  <rule id="100101" level="10">
    <if_sid>550</if_sid>
    <match>File '/etc/passwd' modified</match>
    <description>Critical system file modified: /etc/passwd</description>
    <group>fim,critical_file_modification,</group>
  </rule>
</group>

Rule Explanation

Parameter

Description

id="100101"

Unique identifier for the custom detection rule

level="10"

Assigns a higher alert severity

if_sid>550

Requires the built-in Wazuh file modification rule

match

Matches the file modification message for /etc/passwd

description

Provides a descriptive security alert

group

Categorizes the detection

3. Validate the Configuration

The Wazuh rule configuration was validated using:

sudo /var/ossec/bin/wazuh-analysisd -t
echo $?

Result:

0

The Wazuh manager was restarted:

sudo systemctl restart wazuh-manager
sudo systemctl is-active wazuh-manager

Result:

active

This confirmed that the configuration passed syntax validation and the Wazuh manager started successfully.

4. Detection Testing

To avoid modifying the real /etc/passwd file, a separate test rule was created for a controlled test file:

/home/zerozero/fim-lab/fim-test.txt

The temporary test rule used the same parent rule and text-matching approach as the critical file detection.

<group name="syscheck,fim_testing,">
  <rule id="100102" level="10">
    <if_sid>550</if_sid>
    <match>/home/zerozero/fim-lab/fim-test.txt</match>
    <description>FIM test: Monitored file modified</description>
    <group>fim,custom_detection_test,</group>
  </rule>
</group>

A file modification was generated on the Raspberry Pi:

echo "Wazuh custom rule test $(date)" >> /home/zerozero/fim-lab/fim-test.txt

Test Results

Event

Rule ID

Alert Level

Result

Test file created

554

5

Detected

Test file modified

550

7

Detected

Custom detection triggered

100102

10

Successful

The Wazuh alert confirmed:

2026-11-01T19:34:07.202+0000
Rule: 100102
Level: 10

This demonstrates that the custom rule logic successfully increased the alert severity for a monitored file modification.

Testing limitation: The production rule 100101 was validated syntactically, and monitoring of /etc/passwd was confirmed as active. However, the production rule was not directly triggered because the real system file was intentionally left unchanged.

5. Security Relevance

The /etc/passwd file contains Linux account information. Unexpected modifications may indicate unauthorized account creation or other suspicious administrative activity.

Monitoring this file helps security analysts investigate potential persistence mechanisms, unauthorized configuration changes, and suspicious system activity.

Custom detection rules allow security teams to prioritize alerts for particularly sensitive files.

6. Lessons Learned

During this project, I gained practical experience with:

Configuring Wazuh File Integrity Monitoring.

Understanding built-in Wazuh detection rules.

Creating and validating custom XML detection rules.

Investigating rule matching and decoder behavior.

Troubleshooting detection rules that do not trigger.

Analyzing JSON security alerts.

Testing detection logic without modifying critical system files.

Conclusion

This project demonstrates how Wazuh can be used to monitor critical Linux files and extend built-in detection capabilities with custom security rules.

A controlled test successfully generated a custom Level 10 alert for a monitored file modification.

The resulting detection configuration provides a foundation for expanding critical file monitoring to additional sensitive Linux files.
