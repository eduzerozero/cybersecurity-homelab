Linux User Account & Group Monitoring – Wazuh SIEM

Overview

This project implements custom security detection rules in Wazuh SIEM to monitor Linux user account creation and group modification activities.

The objective is to identify potentially suspicious administrative changes on a monitored Linux endpoint using Auditd and custom Wazuh detection rules.

Both detection rules were successfully validated through simulated log events and live testing.

Lab Environment

Component

Technology

SIEM Platform

Wazuh 4.14.9

SIEM Server

Ubuntu Server 24.04 (VirtualBox VM)

Monitored Endpoint

Raspberry Pi (Linux)

Monitoring Agent

Wazuh Agent

Audit Logging

Linux Auditd

Detection Method

Custom Wazuh XML Rules

1. Auditd Configuration

Auditd was configured to monitor changes to critical Linux account and group configuration files.

sudo auditctl -w /etc/passwd -p wa -k user_account_changes
sudo auditctl -w /etc/group -p wa -k user_group_changes

The -p wa parameter monitors write operations and attribute changes.

These rules were applied at runtime. Persistent configuration across reboots has not yet been implemented as part of this test.

2. User Account Creation Detection

Custom Rule ID: 100103
Severity: Level 10

<group name="auditd,user_account_monitoring,">
  <rule id="100103" level="10">
    <if_sid>80700</if_sid>
    <field name="audit.key">^user_account_changes$</field>
    <field name="audit.command">^useradd$</field>
    <description>Linux user account creation detected via auditd</description>
    <group>account_creation,linux_security,</group>
  </rule>
</group>

Test Procedure

A temporary Linux account was created on the monitored Raspberry Pi:

sudo useradd -m soc-testuser2

Results

Wazuh generated two Level 10 alerts associated with the useradd process accessing and updating /etc/passwd.

The events included the openat and renameat system calls.

Result: PASS

3. Linux Group Modification Detection

Custom Rule ID: 100104
Severity: Level 10

<group name="auditd,linux_group_monitoring,">
  <rule id="100104" level="10">
    <if_sid>80700</if_sid>
    <field name="audit.key">^user_group_changes$</field>
    <field name="audit.command">^usermod$</field>
    <description>Linux user group modification detected via auditd</description>
    <group>group_modification,linux_security,</group>
  </rule>
</group>

Test Procedure

A temporary Linux account was created and added to the administrative sudo group:

sudo useradd -m soc-privtest
sudo usermod -aG sudo soc-privtest

The account was subsequently removed from the sudo group and added again to generate a new event for live validation.

Results

Wazuh successfully generated two Level 10 alerts associated with the usermod process accessing and updating /etc/group.

The alerts contained information including:

Process name: usermod

Audit key: user_group_changes

Original user ID: 1000

Effective user ID: 0

Detection rule: 100104

Result: PASS

4. Detection Pipeline

Linux User Management Command
            |
            v
     Linux Auditd
            |
            v
    /var/log/audit/audit.log
            |
            v
       Wazuh Agent
            |
            v
       Wazuh Manager
            |
            v
   Custom Detection Rules
            |
            v
     Level 10 Security Alert

5. Security Relevance

Monitoring account creation and group modifications helps security analysts investigate unauthorized administrative activity, suspicious account management, and potential privilege escalation.

These activities can be relevant to attacker persistence and privilege escalation techniques.

However, the current rules detect specific command activity against monitored system files. They do not independently prove that an account was successfully created or that sudo privileges were granted.

In particular, rule 100104 detects usermod activity associated with /etc/group and does not exclusively identify additions to the sudo group.

6. Limitations and Future Improvements

Make Auditd monitoring rules persistent across reboots.

Implement more specific detection of administrative group membership changes.

Correlate command execution with resulting account and group state.

Reduce duplicate alerts from multiple system calls.

Extend monitoring to additional account management tools and privileged configuration files.

7. Key Takeaways

This project provided hands-on experience with:

Linux security auditing

Auditd event analysis

Wazuh log decoding and rule evaluation

Custom XML detection rules

Live security event validation

SIEM alert investigation

Both custom detection rules were successfully tested in a controlled home lab environment.
