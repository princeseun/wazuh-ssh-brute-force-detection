# SECURITY INCIDENT INVESTIGATION REPORT

**Incident Title:** SSH Authentication Failures and Privileged Access Investigation  
**Incident Date:** 22 October 2026  
**Affected System:** Ubuntu Server (`labagent`)  
**Affected System IP:** `192.168.xx.ubuntu`  
**Monitoring System:** Wazuh 4.14.9
**Incident Status:** Investigation documented; authorization of source activity not confirmed.

## 1. Incident Summary

On 22 October 2026, the Wazuh monitoring system detected repeated SSH authentication failures originating from `192.168.xx.attacker` and targeting the Ubuntu server `labagent` (`192.168.xx.ubuntu`).

The activity triggered custom Wazuh rule `100002`, configured to identify five matching failed SSH authentication events from the same source IP within 120 seconds.

Further investigation revealed an earlier successful SSH login from the same source IP using the account `ubuntu_prins`. The session involved root-level access and system configuration activity consistent with Wazuh repository setup.

The investigation confirmed the authentication failures, successful login, privileged activity and automated firewall response. However, it did not establish whether the source activity was authorized or malicious.

## 2. Who? — Parties Involved

**Source IP:** `192.168.xx.attacker`
**Target system:** Ubuntu server (`labagent`)
**Target IP:** `192.168.xx.ubuntu`
**Account targeted during failed attempts:** `ololade`
**Account used during successful login:** `ubuntu_prins`
**Monitoring system:** Kali Linux running Wazuh Manager.
**Wazuh agent:** `labagent`, Agent ID `001`.

The identity of the person operating the source machine has not been established.

## 3. What? — What Happened?

Repeated SSH authentication failures were recorded against invalid usernames, including `lab-agent`, `labagent`, `unbuntu_prins` and `ololade`.

Wazuh generated a level 10 alert under custom rule `100002` after detecting repeated failures involving `ololade`.

Investigation also identified a successful SSH login from the same source IP, followed by a root shell and system configuration activity.

## 4. When? — Incident Timeline

All times below are based on the Ubuntu server's journal unless otherwise indicated.

| Date and time | Event |
|---|---|
| Oct 22,  08:55 | Failed SSH attempts using invalid usernames from `192.168.xx.attacker`. |
| Oct 22,  13:26 | Failed attempts using invalid username `unbuntu_prins`. |
| Oct 22,  13:27:10 | Successful SSH login as `ubuntu_prins` from `192.168.xx.attacker`. |
| Oct 22,  13:27:26 | Root shell opened using `sudo -i`. |
| Oct 22,  13:46:23 | `apt-transport-https` installation recorded. |
| Oct 22,  14:01:29 | Successful SSH session closed. |
| Oct 22,  14:03:33 | Wazuh signing-key file creation timestamp. |
| Oct 22,  14:03:59 | Wazuh repository configuration file creation timestamp. |
| Oct 22,  15:29–15:30 | Repeated failed SSH attempts involving `ololade`. |
| Oct 22,  15:30 | Wazuh rule `100002` alert recorded. |

**Timestamp note:** The Wazuh alert displayed `10:30:14 -0500`, while the Ubuntu journal recorded the corresponding activity around 15:30. The difference reflects the displayed timezone offsets and should be normalized in the final evidence package.

## 5. Where? — Location of the Activity

The activity targeted the Ubuntu server:

- Hostname: `labagent`
- IP address: `192.168.xx.ubuntu`
- Service involved: SSH
- SSH service logs: systemd journal
- Monitoring platform: Wazuh Manager on Kali Linux.

The source of the recorded SSH activity was `192.168.xx.attacker`.

## 6. Why? — Possible Reason

The exact reason for the activity could not be conclusively established.

The successful SSH session, root-shell activity, package installation, and subsequent Wazuh repository configuration timestamps are consistent with system administration or Wazuh installation activity.

However, the investigation did not establish whether the activity was authorized. Therefore, it would be premature to classify it as a confirmed malicious intrusion.

## 7. How? — Detection, Investigation and Response

**Detection**

Wazuh used custom rule `100002`:

i.   Rule level: 10
ii.  Frequency: 5 matching events
iii. Timeframe: 120 seconds
iv.  Detection condition: repeated SSH authentication failures from the same source IP.

**Investigation**

The following sources were examined:

1. Wazuh alert records.
2. Ubuntu SSH authentication journal.
3. Sudo activity logs.
4. Package installation logs.
5. APT repository configuration.
6. Wazuh signing-key metadata.
7. File ownership, permissions, and timestamps.

**Response**

Wazuh active response was configured to execute `firewall-drop` when rule `100002` triggered.

Testing confirmed that the source IP was temporarily blocked through firewall DROP rules and that the rules were automatically removed after 60 seconds.

## 8. Impact Assessment

The investigation confirmed:

1. Repeated failed SSH authentication attempts.
2. A successful SSH session from the same source IP.
3. Root-level access during that session.
4. Package installation and repository configuration activity.

The investigation did not establish:

- Unauthorized access.
- Data theft or exposure.
- Malware installation.
- System compromise.
- Service disruption attributable to the activity.

The overall business or operational impact therefore remains undetermined.

## 9. Response and Recommendations

**Response already implemented:**

- Custom Wazuh detection rule `100002`.
- Automated firewall blocking through `firewall-drop`.
- Temporary 60-second block duration.
- Verification of firewall rule creation and removal.

**Recommended improvements:**

1. Confirm whether `192.168.1.45` is an authorized administration machine.
2. Review the `ubuntu_prins` account's access and privileges.
3. Use SSH public-key authentication and disable password authentication where operationally appropriate.
4. Review privileged account activity and restrict root access.
5. Retain and regularly review authentication and system logs.
6. Consider tuning the active-response timeout based on operational requirements.

## 10. Conclusion

The investigation demonstrated that Wazuh successfully detected repeated SSH authentication failures and initiated an automated firewall response.

Log analysis also identified a successful SSH login and privileged system activity from the same source IP earlier that day.

Although the evidence establishes successful privileged access and subsequent configuration changes, it does not establish malicious intent or unauthorized access.

The incident should therefore remain classified as an investigated SSH security event pending confirmation of whether the source activity was authorized.

## 11. Evidence and Appendices

The following evidence supports the investigation:

- **Evidence A:** Wazuh custom detection rule `100002`.
- **Evidence B:** Wazuh alert JSON record.
- **Evidence C:** Ubuntu SSH journal entries showing failed and successful authentication.
- **Evidence D:** Sudo journal records showing root-shell and privileged commands.
- **Evidence E:** DPKG package installation records.
- **Evidence F:** Wazuh APT repository configuration (`wazuh.list`).
- **Evidence G:** Wazuh signing-key details (`wazuh.gpg`).
- **Evidence H:** File metadata showing ownership, permissions and timestamps.
- **Evidence I:** Active-response logs and firewall DROP-rule verification.

**Report prepared for:** Security Incident Investigation and Reporting Lab  
**Prepared by:** Security Analyst (PriNs)  
**Report date:** 3 November 2026
