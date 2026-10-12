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

## Memory baseline (idle, before any agent or data)

| Metric | Value | Date |
|---|---|---|
| splunk-siem used (Splunk idle, no data ingested) | 1.1 GB | 2026-10-06 |
| splunk-siem available | 2.7 GB | 2026-10-06 |
| linux-target used (1 GB RAM, before Universal Forwarder) | 320 Mi | 2026-10-12 |
| linux-target available (same condition) | 640 Mi | 2026-10-12 |

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
