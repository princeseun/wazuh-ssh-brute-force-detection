# Wazuh SSH Brute-Force Detection & Automated IP Blocking

## Project Overview

This project demonstrates how to use Wazuh SIEM to detect repeated SSH login failures and automatically block suspicious IP addresses using Active Response.
The goal was to build a small security monitoring lab, simulate SSH brute-force attempts, detect the activity, and automatically respond by temporarily blocking the attacking IP address.

## Lab Architecture

|    Component        |       Role      |
|  Kali Linux         |    Wazuh Manager |
|  Ubuntu Server      |    Monitored endpoint (Wazuh Agent) |
|  Windows machine    |    SSH attack simulation |

## Tools & Technologies

- Wazuh SIEM 4.14.9
- Kali Linux
- Ubuntu Server
- SSH
- Linux firewall (iptables/nftables)
- Wazuh Active Response

## Detection Configuration

A custom Wazuh rule (ID: `100002`) was created to detect five matching failed SSH authentication events from the same source IP within 120 seconds.

```xml
<rule id="100002" level="10" frequency="5" timeframe="120">
  <if_matched_sid>5710</if_matched_sid>
  <same_srcip />
  <description>Repeated SSH login attempts using non-existent users.</description>
  <group>authentication_failed,ssh_bruteforce,</group>
</rule>
```

## Automated Response

Wazuh Active Response was configured to execute `firewall-drop` on the Ubuntu agent when rule `100002` is triggered.
The following configuration was added to the Wazuh manager's /var/ossec/etc/ossec.conf:

```
<command>
  <name>firewall-drop</name>
  <executable>firewall-drop</executable>
  <timeout_allowed>yes</timeout_allowed>
</command>

<active-response>
  <command>firewall-drop</command>
  <location>defined-agent</location>
  <agent_id>001</agent_id>
  <rules_id>100002</rules_id>
  <timeout>60</timeout>
</active-response>
```

## Testing & Results

SSH login attempts were simulated using a nonexistent username.

The test successfully demonstrated:

1. Detection of repeated SSH login failures.
2. Generation of a high-severity Wazuh alert.
3. Inspect /var/ossec/logs/active-responses.log on Ubuntu.
4. Identification of the attacking source IP.
5. Automatic firewall blocking through Active Response.
6. Automatic removal of the firewall block after 60 seconds.

## Evidence

<img width="760" height="417" alt="Image Oct 30, 2026, 06_25_50 AM" src="https://github.com/user-attachments/assets/2162ee65-b855-4f3d-a6cb-fd2f86f8535f" />

During the active response, the firewall displayed DROP rules for the attacking IP in the INPUT and FORWARD chains:

Chain INPUT (policy ACCEPT)
DROP  all  --  192.168.xx.xxx  0.0.0.0/0

Chain FORWARD (policy ACCEPT)
DROP  all  --  192.168.xx.xxx  0.0.0.0/0

- Wazuh detection alert.
- Active Response execution.
- Firewall DROP rules during the block.
- Firewall rules disappearing after the timeout.

## Key Learning Outcomes

- Configuring Wazuh monitoring.
- Creating custom detection rules.
- Monitoring SSH authentication events.
- Understanding brute-force attack patterns.
- Implementing automated incident response.
- Verifying firewall-based IP blocking.

## Conclusion

This project demonstrates a practical implementation of security monitoring and automated incident response using Wazuh.

It shows how a SIEM solution can detect suspicious authentication activity and trigger an automated firewall action to protect a monitored Linux endpoint.

**Status:** Successfully tested in a controlled home lab environment.
