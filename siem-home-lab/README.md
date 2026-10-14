# Home SOC Lab: Splunk SIEM Detection Engineering

SOC Analyst Portfolio · Splunk Enterprise · Sysmon · MITRE ATT&CK

Building an isolated home lab to practice core SOC analyst skills: log monitoring, alert triage and detection engineering. Splunk Enterprise is the SIEM; Windows and Linux endpoints send logs to it, I simulate attacks from Kali Linux, and I write my own detection rules mapped to MITRE ATT&CK.

**Status:** 3 of 4 hosts built · 1 attack scenario documented · 1 detection rule written · Lab period: October 2026 to present

_Numbers above are updated only from [metrics/metrics-log.md](metrics/metrics-log.md). Sections marked TODO are filled in as each piece is built and tested._

## Lab Architecture

| Host | Role | OS | IP (host-only) | Status |
|---|---|---|---|---|
| splunk-siem | SIEM (Splunk Enterprise) | Ubuntu Server 24.04.5 LTS | 192.168.56.10 | Built |
| linux-target | Monitored endpoint (SSH target), Universal Forwarder | Ubuntu Server 24.04 LTS | 192.168.56.20 | Built |
| kali-attacker | Attacker platform | Kali Linux 2026.2 | 192.168.56.50 | Built |
| windows-target | Monitored endpoint, Sysmon + Universal Forwarder | Windows (evaluation) | 192.168.56.30 (planned) | TODO |

Data flow:

`[kali-attacker] -- attacks --> [linux-target / windows-target + Universal Forwarder] -- logs, port 9997 --> [splunk-siem] -- searches, alerts, dashboard`

TODO: diagram in `diagrams/`.

All hosts run in VirtualBox on a Windows 11 host. Lab traffic stays on an isolated **host-only** network. Each VM also has a NAT adapter, used for package installs and, on kali-attacker, for downloading tools. TODO: state whether the NAT adapter is disabled on target VMs during attack runs.

**Safety:** this is an isolated lab. Attacks are only ever run against machines I own, for training purposes.

## Tools Stack

| Tool | Version | Purpose |
|---|---|---|
| Splunk Enterprise | 10.6.0.5 (60-day trial, 500 MB/day) | SIEM: log collection, search, alerting |
| Splunk Universal Forwarder | 10.6.0.5 | Log shipping from linux-target to splunk-siem |
| Ubuntu Server | 24.04.5 LTS | splunk-siem and linux-target (version listed as supported in Splunk's system requirements) |
| Oracle VirtualBox | 7.2.16 r174877 | Virtualization on a Windows 11 host |
| Kali Linux | 2026.2 | Attack simulation platform |
| Hydra | v9.7 | SSH brute-force simulation |
| Sysmon | TODO | Windows endpoint telemetry |
| Atomic Red Team | TODO | ATT&CK technique simulation |

## Data Source

Splunk indexes: `linux`, `windows`, `sysmon`. Receiving port `9997` is open for Universal Forwarders. Separate indexes keep searches fast, let me cap storage per source and keep noisy Sysmon data apart from the others.

linux-target forwards `/var/log/auth.log` (sourcetype `linux_secure`) to index `linux`. TODO: Windows log sources, once windows-target is built.

## Attack Scenarios

One write-up per scenario in [scenarios/](scenarios/) (template: [scenarios/_template.md](scenarios/_template.md)).

| # | Scenario | MITRE ATT&CK | Detected? | Status |
|---|---|---|---|---|
| 1 | [SSH brute force against linux-target](scenarios/ssh-brute-force.md) | T1110.001 | Yes (6/6 failed attempts detected) | Done |
| 2 | TODO | | | Planned |

## Detection Engineering

One file per rule in [detections/](detections/) (template: [detections/_template.md](detections/_template.md)).

| Rule | ATT&CK technique | Data source | Validated against | Status |
|---|---|---|---|---|
| [SSH Brute Force - Multiple Failed Logins](detections/ssh-brute-force-t1110.md) | T1110.001 | index=linux, sourcetype=linux_secure | Scenario 1 (Hydra, 6 failed logins) | Done, saved as scheduled Splunk alert |

## Key Findings

TODO: results after more attack simulations, with evidence.

### Detection gaps

TODO: attacks that produced no alert or a weak one, and why. Reported as they happened.

## Skills Demonstrated

TODO at the end of the project. Each skill links to the evidence in this repository (a rule, a screenshot, a metric). Skills without evidence are left out.

## Screenshots

- [SSH brute force detected in Splunk](screenshots/ssh-brute-force-detection.png)

TODO: dashboard and more attack evidence in `screenshots/`.

## What I Learned

TODO

## Future Improvements

Planned only if time allows: Cowrie honeypot on an isolated VPS feeding the SIEM, Sigma rules converted to SPL, Python automation with threat-intel IP enrichment.

## Repository Structure

```
├── README.md
├── troubleshooting.md   # problems hit and how I diagnosed them
├── diagrams/            # architecture diagrams
├── screenshots/         # dashboard, alerts, attack evidence
├── scenarios/           # one write-up per attack scenario
├── detections/          # one SPL rule per file
├── configs/             # sanitized config snippets (no secrets)
└── metrics/             # metrics-log.md
```
