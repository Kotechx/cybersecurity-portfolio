# Home SOC Lab: Splunk SIEM Detection Engineering

SOC Analyst Portfolio · Splunk Enterprise · Sysmon · MITRE ATT&CK

Building an isolated home lab to practice core SOC analyst skills: log monitoring, alert triage and detection engineering. Splunk Enterprise is the SIEM; Windows and Linux endpoints send logs to it, I simulate attacks from Kali Linux, and I write my own detection rules mapped to MITRE ATT&CK.

**Status:** 1 of 4 hosts built · 0 attack scenarios · 0 detection rules · Lab period: October 2026 to present

_Numbers above are updated only from [metrics/metrics-log.md](metrics/metrics-log.md). Sections marked TODO are filled in as each piece is built and tested._

## Lab Architecture

| Host | Role | OS | IP (host-only) | Status |
|---|---|---|---|---|
| splunk-siem | SIEM (Splunk Enterprise) | Ubuntu Server 24.04.5 LTS | 192.168.56.10 | Built |
| linux-target | Monitored endpoint (SSH target), Universal Forwarder | Ubuntu Server 24.04 LTS | 192.168.56.20 (planned) | TODO |
| windows-target | Monitored endpoint, Sysmon + Universal Forwarder | Windows (evaluation) | 192.168.56.30 (planned) | TODO |
| kali | Attacker platform | Kali Linux | 192.168.56.50 (planned) | TODO |

Planned data flow:

`[Kali] -- attacks --> [Windows / Linux endpoint + Universal Forwarder] -- logs, port 9997 --> [Splunk] -- searches, alerts, dashboard`

TODO: diagram in `diagrams/`.

All hosts run in VirtualBox on a Windows 11 host. Lab traffic stays on an isolated **host-only** network. Each VM also has a NAT adapter, used for package installs. TODO: state whether the NAT adapter is disabled on target VMs during attack runs.

**Safety:** this is an isolated lab. Attacks are only ever run against machines I own, for training purposes.

## Tools Stack

| Tool | Version | Purpose |
|---|---|---|
| Splunk Enterprise | 10.6.0.5 (60-day trial, 500 MB/day) | SIEM: log collection, search, alerting |
| Ubuntu Server | 24.04.5 LTS | Splunk host (version listed as supported in Splunk's system requirements) |
| Oracle VirtualBox | 7.2.16 r174877 | Virtualization on a Windows 11 host |
| Splunk Universal Forwarder | TODO | Log shipping from endpoints |
| Sysmon | TODO | Windows endpoint telemetry |
| Kali Linux | TODO | Attack simulation |
| Atomic Red Team | TODO | ATT&CK technique simulation |

## Data Source

Splunk indexes: `linux`, `windows`, `sysmon`. Receiving port `9997` is open for Universal Forwarders. Separate indexes keep searches fast, let me cap storage per source and keep noisy Sysmon data apart from the others.

TODO: list of onboarded log sources once the endpoints are sending data.

## Attack Scenarios

One write-up per scenario in [scenarios/](scenarios/) (template: [scenarios/_template.md](scenarios/_template.md)).

| # | Scenario | MITRE ATT&CK | Detected? | Status |
|---|---|---|---|---|
| 1 | TODO | | | Planned |

## Detection Engineering

One file per rule in [detections/](detections/) (template: [detections/_template.md](detections/_template.md)).

| Rule | ATT&CK technique | Data source | Validated against | Status |
|---|---|---|---|---|
| TODO | | | | |

TODO: thresholds tuned from observed attack data, and false-positive / false-negative notes per rule.

## Key Findings

TODO: results after the attack simulations, with evidence.

### Detection gaps

TODO: attacks that produced no alert or a weak one, and why. Reported as they happened.

## Skills Demonstrated

TODO at the end of the project. Each skill links to the evidence in this repository (a rule, a screenshot, a metric). Skills without evidence are left out.

## Screenshots

TODO: dashboard, alerts and attack evidence in `screenshots/`.

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
