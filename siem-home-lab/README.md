# Home SOC Lab: Splunk SIEM Detection Engineering

SOC Analyst Portfolio · Splunk Enterprise · Sysmon · MITRE ATT\&CK

Building an isolated home lab to practice core SOC analyst skills: log monitoring, alert triage and detection engineering. Splunk Enterprise is the SIEM; Windows and Linux endpoints send logs to it, I simulate attacks from Kali Linux and locally via Atomic Red Team, and I write my own detection rules mapped to MITRE ATT\&CK.

**Status:** 4 of 4 hosts built · 2 attack scenarios documented · 2 detection rules written · Lab period: October 2026 to present

*Numbers above are updated only from* [*metrics/metrics-log.md*](metrics/metrics-log.md)*. Sections marked TODO are filled in as each piece is built and tested.*

## Lab Architecture

|Host|Role|OS|IP (host-only)|Status|
|-|-|-|-|-|
|splunk-siem|SIEM (Splunk Enterprise)|Ubuntu Server 24.04.5 LTS|192.168.56.10|Built|
|linux-target|Monitored endpoint (SSH target), Universal Forwarder|Ubuntu Server 24.04 LTS|192.168.56.20|Built|
|windows-target|Monitored endpoint, Sysmon + Universal Forwarder|Windows 10 Enterprise Evaluation|192.168.56.30|Built|
|kali-attacker|Attacker platform|Kali Linux 2026.2|192.168.56.50|Built|

Data flow:

`kali-attacker -- attacks --> linux-target / windows-target (+ Universal Forwarder) -- logs, port 9997 --> splunk-siem -- searches, alerts, dashboard`

!\[Lab architecture diagram](diagrams/architecture.svg)

All hosts run in VirtualBox on a Windows 11 host. Lab traffic stays on an isolated **host-only** network. Each VM also has a NAT adapter, used for package installs and tool downloads. TODO: state whether the NAT adapter is disabled on target VMs during attack runs.

**Safety:** this is an isolated lab. Attacks are only ever run against machines I own, for training purposes.

## Tools Stack

|Tool|Version|Purpose|
|-|-|-|
|Splunk Enterprise|10.6.0.5 (60-day trial, 500 MB/day)|SIEM: log collection, search, alerting|
|Splunk Universal Forwarder (Linux)|10.6.0.5|Log shipping from linux-target|
|Splunk Universal Forwarder (Windows)|10.6.1.1|Log shipping from windows-target (Security + Sysmon channels)|
|Ubuntu Server|24.04.5 LTS|splunk-siem and linux-target (version listed as supported in Splunk's system requirements)|
|Windows 10 Enterprise|Evaluation (90-day)|windows-target endpoint|
|Sysmon|v15.22, SwiftOnSecurity config (schema 4.91)|Windows endpoint telemetry (process creation, network, etc.)|
|Oracle VirtualBox|7.2.16 r174877|Virtualization on a Windows 11 host|
|Kali Linux|2026.2|Attack simulation platform|
|Hydra|v9.7|SSH brute-force simulation; attempted (unsuccessfully) for RDP|
|ncrack|(preinstalled on Kali)|Attempted (unsuccessfully) for RDP brute force|
|Invoke-AtomicRedTeam|Latest (master branch)|Local execution of MITRE ATT\&CK atomic tests on windows-target|

## Data Source

Splunk indexes: `linux`, `windows`, `sysmon`. Receiving port `9997` is open for Universal Forwarders.

* linux-target forwards `/var/log/auth.log` (sourcetype `linux\\\\\\\_secure`) to index `linux`
* windows-target forwards the **Security** event log to index `windows` (sourcetype `WinEventLog:Security`) and the **Sysmon operational log** to index `sysmon` (sourcetype `WinEventLog:Microsoft-Windows-Sysmon/Operational`)

## Attack Scenarios

One write-up per scenario in [scenarios/](scenarios/) (template: [scenarios/\_template.md](scenarios/_template.md)).

|#|Scenario|MITRE ATT\&CK|Detected?|Status|
|-|-|-|-|-|
|1|[SSH brute force against linux-target](scenarios/ssh-brute-force.md)|T1110.001|Yes (6/6 failed attempts detected)|Done|
|2|[System Information Discovery via Atomic Red Team](scenarios/t1082-system-info-discovery.md)|T1082|Yes (full process chain captured)|Done|
|3|TODO|||Planned|

## Detection Engineering

One file per rule in [detections/](detections/) (template: [detections/\_template.md](detections/_template.md)).

|Rule|ATT\&CK technique|Data source|Validated against|Status|
|-|-|-|-|-|
|[SSH Brute Force - Multiple Failed Logins](detections/ssh-brute-force-t1110.md)|T1110.001|index=linux, sourcetype=linux\_secure|Scenario 1 (Hydra, 6 failed logins)|Done, saved as scheduled Splunk alert|
|[T1082 - System Information Discovery](detections/t1082-system-info-discovery.md)|T1082|index=sysmon, EventCode=1|Scenario 2 (Atomic Red Team T1082-1)|Done, saved as scheduled Splunk alert|

## Key Findings

TODO: results after more attack simulations, with evidence.

### Detection gaps

* **RDP brute force tooling failure:** both ncrack and Hydra (Kali's standard brute-force tools) failed to complete a brute-force attempt against windows-target's RDP service, even after disabling Network Level Authentication (NLA). ncrack's handshake timed out with 0 completed attempts; Hydra's RDP module (explicitly marked "experimental" by its own authors) failed to establish the connection via `freerdp`. This is a tooling limitation, not a detection gap on the defensive side — it meant I could not generate a real RDP brute-force dataset with these tools, so I pivoted to a locally-executed Atomic Red Team test instead. TODO: revisit with a different tool (e.g. a custom script, or Metasploit's RDP scanner) if time allows.

## Skills Demonstrated

- **SIEM deployment and administration** — installed and configured Splunk Enterprise from scratch, including index design, boot-start persistence, and license management ([README](#lab-architecture), [metrics-log.md](metrics/metrics-log.md))
- **Log forwarding and agent management** — deployed and troubleshot Splunk Universal Forwarder on both Linux and Windows, resolving permission errors on each platform ([troubleshooting.md](troubleshooting.md))
- **Detection engineering** — wrote, tested, and tuned two SPL detection rules mapped to MITRE ATT&CK, with documented thresholds and false-positive analysis ([detections/](detections/))
- **Adversary simulation** — ran SSH brute-force attacks with Hydra and executed MITRE ATT&CK techniques with Atomic Red Team against my own isolated targets ([scenarios/](scenarios/))
- **Network segmentation** — designed an isolated host-only lab network separate from NAT internet access, documented in the [architecture diagram](diagrams/architecture.svg)
- **Technical documentation** — maintained a troubleshooting log, metrics log, and attack/detection write-ups throughout the project, with every number traceable to a real measurement
- **Version control** — managed the full project history in Git/GitHub, including incremental commits per milestone

## Screenshots

* [SSH brute force detected in Splunk](screenshots/ssh-brute-force-detection.png)
* [T1082 System Information Discovery detected in Splunk](screenshots/t1082-detection.png)



* [SOC Home Lab Overview dashboard](screenshots/dashboard-overview.png)

## What I Learned

- **Linux troubleshooting under pressure.** The Day 1 Ubuntu install left me with no user account and no network config. Working through GRUB recovery mode, manual netplan configuration, and a broken apt source taught me more about how Linux boots and configures itself than a clean install ever would have.
- **Permissions are rarely obvious, but always traceable.** Two separate access-denied problems (the Universal Forwarder on Linux needing the `adm` group to read `auth.log`, and the Windows Universal Forwarder needing `Event Log Readers` to read the Sysmon channel) taught me to read the actual error code instead of guessing, and to check group membership before anything else.
- **Time zones break SPL silently.** I lost time twice to the same mistake: writing absolute-time searches that Splunk interpreted in my local timezone instead of UTC, so the dashboard panels came back empty even though the data was there. Relative time (`earliest=-Nm`) avoided the problem entirely.
- **A failed attack is still a finding.** RDP brute force against my own Windows target failed with both ncrack and Hydra, even with NLA disabled. Instead of treating that as a dead end, I documented it as a detection/tooling gap and pivoted to Atomic Red Team, which ended up producing a cleaner, more realistic detection (T1082 via the full Sysmon process chain).
- **Detection engineering is iterative.** Writing the SSH brute-force rule meant reconstructing two separate Hydra runs from raw `auth.log` timestamps, picking a bucket span that would actually group the bursts together, and validating the threshold against real attack data instead of a number I assumed would work.
- **Infrastructure hygiene matters before day one of security work.** Keeping the Git repo and the VirtualBox VMs outside of OneDrive, separating a dedicated attacker VM from my personal Kali instance, and never committing installers or license files were small decisions that would have caused real problems later if skipped.

## Future Improvements

Planned only if time allows: Cowrie honeypot on an isolated VPS feeding the SIEM, Sigma rules converted to SPL, Python automation with threat-intel IP enrichment, revisiting RDP brute force with different tooling.

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

