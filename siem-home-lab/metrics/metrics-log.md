# Metrics Log

Only real, measured values go here, each with a date. Zeros stay as zeros until something changes them.

## Environment

| Metric | Value | Date |
|---|---|---|
| Hosts built in the lab | 3 (splunk-siem, linux-target, kali-attacker) | 2026-10-14 |
| Splunk version | 10.6.0.5 | 2026-10-06 |
| Splunk license | Trial, expires 2026-12-04, 500 MB/day | 2026-10-06 |
| Indexes | 3 (linux, windows, sysmon) | 2026-10-06 |
| Receiving port | 9997 listening | 2026-10-06 |
| splunk-siem VM | 4 GB RAM, 2 vCPU, 40 GB disk | 2026-10-06 |
| linux-target VM | 1 GB RAM, 1 vCPU, 15 GB disk, static IP 192.168.56.20 | 2026-10-12 |
| Universal Forwarder on linux-target | 10.6.0.5, runs as splunkfwd, starts on boot, sends /var/log/auth.log to index linux | 2026-10-12 |
| kali-attacker VM | 2 GB RAM, 2 vCPU, static IP 192.168.56.50, Kali 2026.2, Hydra v9.7 and nmap preinstalled | 2026-10-14 |

## Memory baseline

| Metric | Value | Date |
|---|---|---|
| splunk-siem used / available (Splunk idle, no data ingested) | 1.1 GB / 2.7 GB | 2026-10-06 |
| linux-target used / available (1 GB RAM, before Universal Forwarder) | 320 Mi / 640 Mi | 2026-10-12 |
| linux-target used / available (Universal Forwarder installed, stopped) | 339 Mi / 621 Mi | 2026-10-12 |
| linux-target used / available (Universal Forwarder running, after reboot) | 361 Mi / 600 Mi | 2026-10-12 |
| kali-attacker used / available (2 GB RAM, idle, before any attack) | 675 Mi / 1.3 Gi | 2026-10-14 |

Measured at different moments, so the comparison is approximate.

## Ingestion baseline (normal VM activity, before any attack)

| Metric | Value | Date |
|---|---|---|
| auth.log events indexed in index=linux (host linux-target, sourcetype linux_secure) | 93 | 2026-10-12 |

These events are installation and admin activity, not attacks. Attack runs are counted from a recorded start time.

## Detection results

| Metric | Value |
|---|---|
| Attack scenarios documented | 1 |
| Detection rules written | 1 |
| ATT&CK techniques covered | 1 (T1110.001 - Password Guessing) |
| Attack events generated | 6 failed SSH logins + 2 successful logins (2 Hydra runs against linux-target, 2026-10-14) |
| Attack events detected | 6 of 6 failed logins captured by the SSH Brute Force detection rule |

## Daily ingestion (License page, "Volume used today")

| Date | MB used | Warning count | Notes |
|---|---|---|---|
| | | | |

## Snapshots (timeline)

| Snapshot | What it holds |
|---|---|
| ubuntu-base-ip-ok | splunk-siem: Ubuntu 24.04, static IP 192.168.56.10, SSH working |
| splunk-instalado | splunk-siem: Splunk installed, starts on boot |
| splunk-indices-9997 | splunk-siem: Splunk with 3 indexes and receiving port 9997 |
| linux-target-base | linux-target: Ubuntu 24.04, IP 192.168.56.20, 1 GB RAM, SSH ok, updated (2 packages deferred by phasing) |
| linux-target-uf | linux-target: Universal Forwarder installed, starts on boot, sends auth.log to index linux |
| kali-attacker-base | kali-attacker: Kali 2026.2, static IP 192.168.56.50, updated, Hydra/nmap confirmed |
