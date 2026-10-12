# Metrics Log

Only real, measured values go here, each with a date. Zeros stay as zeros until something changes them.

## Environment

| Metric | Value | Date |
|---|---|---|
| Hosts built in the lab | 2 (splunk-siem, linux-target) | 2026-10-12 |
| Splunk version | 10.6.0.5 | 2026-10-06 |
| Splunk license | Trial, expires 2026-12-04, 500 MB/day | 2026-10-06 |
| Indexes | 3 (linux, windows, sysmon) | 2026-10-06 |
| Receiving port | 9997 listening | 2026-10-06 |
| splunk-siem VM | 4 GB RAM, 2 vCPU, 40 GB disk | 2026-10-06 |
| linux-target VM | 1 GB RAM, 1 vCPU, 15 GB disk, static IP 192.168.56.20 | 2026-10-12 |
| Universal Forwarder on linux-target | 10.6.0.5, runs as splunkfwd, starts on boot, sends /var/log/auth.log to index linux | 2026-10-12 |

## Memory baseline

| Metric | Value | Date |
|---|---|---|
| splunk-siem used / available (Splunk idle, no data ingested) | 1.1 GB / 2.7 GB | 2026-10-06 |
| linux-target used / available (1 GB RAM, before Universal Forwarder) | 320 Mi / 640 Mi | 2026-10-12 |
| linux-target used / available (Universal Forwarder installed, stopped) | 339 Mi / 621 Mi | 2026-10-12 |
| linux-target used / available (Universal Forwarder running, after reboot) | 361 Mi / 600 Mi | 2026-10-12 |

Measured at different moments, so the comparison is approximate.

## Ingestion baseline (normal VM activity, before any attack)

| Metric | Value | Date |
|---|---|---|
| auth.log events indexed in index=linux (host linux-target, sourcetype linux_secure) | 93 | 2026-10-12 |

These events are installation and admin activity, not attacks. Attack runs are counted from a recorded start time.

## Detection results

| Metric | Value |
|---|---|
| Attack scenarios documented | 0 |
| Detection rules written | 0 |
| ATT&CK techniques covered | 0 |
| Attack events generated | not measured yet |
| Attack events detected | not measured yet |

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
