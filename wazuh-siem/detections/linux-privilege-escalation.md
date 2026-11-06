Linux Privilege Escalation Detection with Wazuh

Overview

This project implements a custom Wazuh SIEM detection rule to monitor potentially suspicious privilege escalation activity on a Linux system.

The detection identifies executions of usermod that attempt to add a user to the sudo group.

Lab Environment

SIEM: Wazuh 4.14.9

Manager: Ubuntu 24.04 VM

Monitored Endpoint: Raspberry Pi

Log Source: Linux Auditd

Detection Rule: 100105

Alert Severity: Level 12

Detection Logic

The following command is monitored:

sudo usermod -aG sudo <username>

Adding an account to the sudo group can grant administrative privileges. Such changes should be monitored because they may indicate unauthorized privilege escalation.

Auditd Configuration

An Auditd execution watch was configured for the usermod binary:

sudo auditctl -w /usr/sbin/usermod -p x -k usermod_execution

Custom Wazuh Rule

The detection was implemented in:

/var/ossec/etc/rules/local_rules.xml

<group name="auditd,privilege_escalation,">
  <rule id="100105" level="12">
    <if_sid>80700</if_sid>
    <field name="audit.type">^SYSCALL$</field>
    <field name="audit.key">^usermod_execution$</field>
    <field name="audit.command">^usermod$</field>
    <field name="audit.execve.a0">^usermod$</field>
    <field name="audit.execve.a1">^-aG$</field>
    <field name="audit.execve.a2">^sudo$</field>
    <description>Potential privilege escalation: usermod command targeting sudo group</description>
    <group>privilege_escalation,linux_security,</group>
  </rule>
</group>

Testing and Validation

A temporary test account named soc-test105 was created in the lab.

The following command was executed:

sudo usermod -aG sudo soc-test105

Wazuh successfully generated a live alert with:

Rule ID: 100105

Severity: 12

Agent: raspberry-pi

Command: usermod

Target Group: sudo

Log Source: /var/log/audit/audit.log

Troubleshooting

The initial rule matched standalone EXECVE events during testing but did not generate live alerts.

Investigation revealed that Wazuh processed the relevant Auditd records as a grouped SYSCALL event containing additional EXECVE fields.

The rule was updated to match audit.type=SYSCALL while inspecting the decoded command arguments.

After the modification, the rule passed wazuh-logtest and successfully generated a live alert.

Limitations

This rule detects a specific usermod -aG sudo command pattern. It does not cover every possible method of granting administrative privileges.

The alert confirms command execution, not necessarily successful completion of the account modification.

Cleanup

The temporary test account was removed after validation:

sudo userdel -r soc-test105

The account's removal and absence from the sudo group were verified.

Skills Demonstrated

Linux security monitoring

Auditd configuration and event analysis

Wazuh custom rule development

SIEM alert validation

Privilege escalation detection

Troubleshooting and incident investigation
