# Metrics Log

Only real, measured values go here, each with a date. Zeros stay as zeros until something changes them.

## Environment

| Metric | Value | Date |
|---|---|---|
| Hosts built in the lab | 1 (splunk-siem) | 2026-10-06 |
| Splunk version | 10.6.0.5 | 2026-10-06 |
| Splunk license | Trial, expires 2026-12-04, 500 MB/day | 2026-10-06 |
| Indexes | 3 (linux, windows, sysmon) | 2026-10-06 |
| Receiving port | 9997 listening | 2026-10-06 |
| splunk-siem VM | 4 GB RAM, 2 vCPU, 40 GB disk | 2026-10-06 |

## Memory baseline (Splunk idle, no data ingested)

| Metric | Value | Date |
|---|---|---|
| used | 1.1 GB | 2026-10-06 |
| available | 2.7 GB | 2026-10-06 |

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
| ubuntu-base-ip-ok | Ubuntu 24.04, static IP 192.168.56.10, SSH working |
| splunk-instalado | Splunk installed, starts on boot |
| splunk-indices-9997 | Splunk with 3 indexes and receiving port 9997 |
