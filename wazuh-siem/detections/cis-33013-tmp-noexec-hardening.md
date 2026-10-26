CIS Benchmark Hardening – Securing /tmp with noexec

Overview

This project demonstrates Linux system hardening using Wazuh Security Configuration Assessment (SCA) and the CIS Debian Linux 13 Benchmark.

Environment

Endpoint: Raspberry Pi, Debian 13, ARM64

SIEM: Wazuh 4.14.9

Assessment: CIS Debian Linux 13 Benchmark

Check ID: 33013

Check: Ensure noexec option set on /tmp partition

1. Security Finding

Wazuh initially reported check 33013 as Failed because the /tmp filesystem was mounted without the noexec option.

Initial configuration:

/tmp tmpfs tmpfs rw,nosuid,nodev,nr_inodes=1048576

Without noexec, executable files stored in /tmp may be executed directly.

2. Investigation

The following commands were used to investigate the configuration:

findmnt -T /tmp -o TARGET,SOURCE,FSTYPE,OPTIONS
systemctl cat tmp.mount
sudo fuser -vm /tmp

The investigation confirmed that /tmp was mounted as tmpfs and managed by systemd.

3. Remediation

First, the security option was tested temporarily:

sudo mount -o remount,noexec /tmp

After successful testing, a persistent systemd override was created:

File: /etc/systemd/system/tmp.mount.d/override.conf

[Mount]
Options=mode=1777,strictatime,nosuid,nodev,noexec,size=50%%,nr_inodes=1m

The configuration was reloaded using:

sudo systemctl daemon-reload

A system reboot confirmed that the configuration persisted.

4. Validation

The final mount configuration was verified:

/tmp tmpfs tmpfs rw,nosuid,nodev,noexec,nr_inodes=1048576

Wazuh SCA check 33013 changed from Failed to Passed.

Additional service checks confirmed that the Docker containers and portfolio remained accessible during initial testing.

5. Troubleshooting

After rebooting, the Pi-hole container failed to start because the host's Nginx service occupied TCP port 80.

The conflict was identified using:

sudo ss -tulpn | grep ':80 '

The host Nginx service was stopped and its automatic startup disabled:

sudo systemctl stop nginx
sudo systemctl disable nginx

Pi-hole was then started successfully and reached a healthy state.

This issue was a port-binding conflict rather than an observed failure caused by the noexec setting.

6. Security Outcome

CIS check 33013: Failed → Passed

Persistent /tmp hardening configured

Configuration verified after reboot

Pi-hole restored to healthy operation

Port 80 startup conflict resolved

Lessons learned: Security configuration changes should be validated through both compliance scanning and operational testing. A passing security check alone does not guarantee that every application remains functional.
