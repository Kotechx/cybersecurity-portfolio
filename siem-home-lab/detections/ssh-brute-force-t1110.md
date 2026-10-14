# SSH Brute Force - Multiple Failed Logins

- **ATT&CK technique:** T1110 (Brute Force), specifically T1110.001 (Password Guessing)
- **Data source:** index `linux`, sourcetype `linux_secure`, host `linux-target` (`/var/log/auth.log` via Splunk Universal Forwarder)
- **Hypothesis:** an attacker trying many passwords against one SSH account from the same source in a short window produces repeated `Failed password` entries that a legitimate user would rarely generate.

## SPL
```
earliest=-5m@m latest=now index=linux host=linux-target sourcetype=linux_secure "Failed password"
| rex field=_raw "Failed password for (invalid user )?(?<ssh_user>\S+) from (?<src_ip>[\d\.]+) port (?<src_port>\d+)"
| bucket _time span=5m
| stats count as failed_attempts by _time, src_ip, ssh_user
| where failed_attempts >= 3
```

The fields (`ssh_user`, `src_ip`, `src_port`) are extracted at search time with `rex`, since no Splunk add-on for Linux log parsing is installed in this lab.

## Threshold and tuning
Threshold set at **3 failed attempts within a 5-minute window** (`bucket span=5m`). The 3-attempt threshold matches what a single Hydra run against this lab produced (3 failures per execution), so it catches that scenario without being so low that a single mistyped password would trigger it. The 5-minute window is short enough to catch an automated run, which in testing completed in a few seconds, while being wide enough to tolerate normal clock/indexing variance.

## False positives / false negatives
- **False positive:** a legitimate user who mistypes their password 3 times in a row within 5 minutes would also trigger this rule. The rule does not distinguish human error from automated guessing; it only measures volume and grouping by source IP and username.
- **False negative:** a "low and slow" attack that stays under 3 attempts per 5-minute window would not trigger this rule. This is a known limitation of fixed-threshold, fixed-window detection.

## Validation
Tested against two Hydra brute-force runs from kali-attacker (192.168.56.50) against linux-target (192.168.56.20), on 2026-10-14. Both runs landed in the same 5-minute bucket (18:30:00-18:35:00 UTC) because they were only about 90 seconds apart, producing a combined total of 6 failed attempts for user `linux-target` from `192.168.56.50`. The query returned exactly that one row (`failed_attempts=6`), confirming the rule fires for this attack pattern. See [../scenarios/ssh-brute-force.md](../scenarios/ssh-brute-force.md) for the full attack write-up.

Saved as a Splunk scheduled alert ("SSH Brute Force - Multiple Failed Logins"), running every 5 minutes, triggering when Number of Results > 0.
