##Wazuh SIEM – Cybersecurity Homelab

#Project Overview

This project documents the deployment and configuration of a Wazuh-based Security Information and Event Management (SIEM) environment.

The objective is to gain hands-on experience in security monitoring, log analysis, detection engineering, file integrity monitoring, and incident investigation.

The homelab uses a dedicated Ubuntu virtual machine as the Wazuh server and a Raspberry Pi as a monitored Linux endpoint.

#Lab Architecture

Component

Technology

Purpose

Virtualization

Oracle VirtualBox

Hosts the SIEM server

SIEM Server

Ubuntu 24.04 LTS

Runs the Wazuh infrastructure

SIEM Platform

Wazuh 4.14.10

Security monitoring and detection

Data Storage

Wazuh Indexer

Stores and indexes security events

Visualization

Wazuh Dashboard

Alert analysis and investigation

Monitored Endpoint

Raspberry Pi

Linux endpoint monitored by Wazuh

Audit Logging

Linux auditd

Records security-relevant system activity

File Integrity Monitoring

Wazuh FIM

Detects monitored file modifications

Architecture:

Raspberry Pi → Wazuh Agent → Wazuh Manager → Filebeat → Wazuh Indexer → Wazuh Dashboard

#Detection Engineering

Six custom Wazuh detection rules were developed and tested.

Rule ID

Detection

Severity

Validation

100100

Sudo Command Execution

5

Live test

100101

Critical File Modification

10

Live test

100103

Linux User Account Creation

10

Live test

100104

Linux Group Modification

10

Live test

100105

Potential Privilege Escalation

12

Live test

100106

SSH Brute-Force Detection

10

Wazuh Logtest

The custom rules are available in config/wazuh/wazuh-custom-rules.xml.

#Security Testing

Sudo Execution Monitoring

Monitored privileged command execution using Linux auditd.

Validated that executing a command through sudo generates a custom Wazuh alert.

#User Account Monitoring

Tested the detection of Linux user creation and group modifications using useradd and usermod.

#Privilege Escalation Detection

Validated a custom rule that detects attempts to add a Linux user to the sudo group using usermod -aG sudo.

#File Integrity Monitoring

Configured monitoring of critical Linux files, including /etc/passwd.

Successfully detected changes to the monitored file through Wazuh FIM.

#SSH Brute-Force Detection

Created a correlation rule for repeated failed SSH authentication attempts originating from the same IP address.

Validated the rule using wazuh-logtest with simulated SSH authentication failures.

This confirms rule evaluation in the test environment; a live end-to-end test of this rule on the new server remains a separate validation step.

#Detection Documentation

Additional technical documentation is available in the detections/ directory:

sudo-execution.md

critical-file-monitoring.md

file-integrity-monitoring.md

linux-user-account-monitoring.md

linux-privilege-escalation.md

ssh-authentication-failure.md

ssh-brute-force-detection.md

ssh-brute-force-incident.md

ssh-bruteforce-active-response.md

The directory also contains notes on security hardening and vulnerability remediation.

#Project Structure

wazuh-siem/
├── README.md
├── config/
│   ├── auditd/
│   │   ├── sudo_execution.rules
│   │   └── wazuh-homelab.rules
│   └── wazuh/
│       └── wazuh-custom-rules.xml
├── detections/
│   └── Security testing and detection documentation
└── screenshots/
    └── Wazuh alerts and investigation evidence

#Key Learning Outcomes

Deploying and configuring a Wazuh SIEM environment

Connecting and monitoring Linux endpoints

Working with Linux auditd and system logs

Creating and validating custom detection rules

Investigating security alerts

Implementing file integrity monitoring

Understanding SSH authentication monitoring and event correlation

Troubleshooting Wazuh configuration and XML rule errors

#Future Improvements

Perform an end-to-end SSH brute-force detection test

Add additional Linux and Windows agents

Expand MITRE ATT&CK mappings

Evaluate Wazuh Active Response in a controlled lab

Develop additional detection and incident investigation scenarios

#Disclaimer

This project is intended for educational purposes and authorized security testing within a personal homelab environment.









