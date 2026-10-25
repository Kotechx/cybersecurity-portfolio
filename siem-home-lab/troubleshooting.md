# Troubleshooting Log

Problems hit while building the lab, how I diagnosed them, and what fixed them. "Cause" is only stated as confirmed when I verified it; otherwise it is marked as a hypothesis.

## 1. Ubuntu installer stalled at "copying metadata from /cdrom"

- **Symptom:** installer sat on its last step for 30+ minutes while the VM still showed CPU activity
- **Diagnosis:** opened the installer shell and read `/var/log/installer/subiquity-server-debug.log`; no install errors, only network events
- **Cause:** not confirmed. Hypothesis: the installer's final steps did not complete, which would also explain problems 2 to 4
- **Outcome:** the installer eventually reached the reboot prompt (with a `failed unmounting cdrom.mount` warning) and the installed system booted
- **Evidence:** TODO screenshot

## 2. No user account and wrong hostname after first boot

- **Symptom:** login prompt showed `localhost`, and no username I knew was accepted
- **Diagnosis:** booted GRUB recovery mode to a root shell, `ls /home` returned nothing
- **Cause:** confirmed that no user existed. Why the installer's profile step was not applied is not confirmed (same hypothesis as problem 1)
- **Fix:** from the recovery shell: `adduser`, `usermod -aG sudo <user>`, `echo splunk-siem > /etc/hostname`, reboot
- **Verification:** `hostnamectl` showed `splunk-siem`

## 3. No network on either adapter

- **Symptom:** `ip -4 a` showed only the loopback interface, and boot waited about 2 minutes on `systemd-networkd-wait-online`
- **Diagnosis:** `ip a` showed both interfaces DOWN, `networkctl` showed both `off`/`unmanaged`, and `/etc/netplan/` was empty
- **Cause:** confirmed that no netplan configuration had been written
- **Fix:** created `/etc/netplan/01-lab.yaml` (NAT adapter on DHCP, host-only adapter on static 192.168.56.10/24 with no gateway), `chmod 600`, `netplan apply`
- **Why no gateway on the host-only adapter:** two default routes could make the system pick the wrong one and lose internet access
- **Verification:** `ip route` showed a single default route via NAT; ping to an IP and to a hostname both worked

## 4. apt tried to install packages from the install CD

- **Symptom:** `apt install openssh-server` failed with `File not found - /cdrom/pool/main/...`
- **Diagnosis:** `grep -rn cdrom /etc/apt/sources.list` found the `file:///cdrom` source, and `/etc/apt/sources.list.d/` held only `ubuntu.sources.curtin.orig`
- **Cause:** confirmed that the CD source was still configured and the online repositories file was missing
- **Fix:** restored the online repositories from the `.curtin.orig` backup as `ubuntu.sources` and commented out the CD line in `sources.list`
- **Verification:** `apt update` fetched from archive.ubuntu.com and security.ubuntu.com with no `cdrom` errors

## 5. OpenSSH reported inactive after install

- **Symptom:** `systemctl status ssh` showed the service inactive, but SSH from Windows worked
- **Diagnosis:** `systemctl status ssh.socket` showed the socket enabled, listening on port 22 (IPv4 and IPv6), with `Triggers: ssh.service`
- **Cause:** consistent with socket activation on Ubuntu 24.04: `ssh.socket` listens on port 22 and starts `ssh.service` on the first incoming connection. I did not capture the service state before the first connection, so this is the most likely explanation, not a proven one
- **Fix:** none needed. SSH was working as designed
- **Lesson:** "inactive" does not always mean "down". Check the socket unit before restarting a service
## Splunk bucket integrity warning (windows index)

During dashboard testing, a search against index=windows returned: 
"Failed to read ... Rawdata may be corrupt" for one bucket. 
Results still returned complete (5325 events), so likely an isolated 
bucket issue with no data loss so far. Monitoring for recurrence; 
if search results come back incomplete in the future, this bucket 
is the first suspect.
