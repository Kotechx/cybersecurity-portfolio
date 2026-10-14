# SSH Brute Force Against linux-target (T1110.001)

## What I ran
From kali-attacker (192.168.56.50), against linux-target (192.168.56.20):

```
echo "linux-target" > ~/users.txt
echo -e "123456\nsenha123\nadmin\n<correct password>\nqwerty" > ~/passwords.txt
hydra -L ~/users.txt -P ~/passwords.txt -t 4 ssh://192.168.56.20
```

A 5-entry password list was used instead of a full wordlist (e.g. rockyou.txt) to keep the test fast, controlled, and low-volume against the Splunk trial license's 500 MB/day limit, with the correct password included so the run would prove a full attack lifecycle (failures followed by a success) rather than only failed attempts.

## What happened
I ran Hydra twice without noticing, having forgotten to record a start timestamp (`date -u`) before the first run. Reconstructing the timeline from `/var/log/auth.log` on linux-target showed two separate brute-force bursts, about 90 seconds apart:

| Run | Window (UTC) | Failed attempts | Successful login |
|---|---|---|---|
| 1 | 18:33:12-18:33:26 | 3 | 1 (port 33484) |
| 2 | 18:34:40-18:34:45 | 3 | 1 (port 36974) |

All failed and successful attempts were for user `linux-target` from source IP `192.168.56.50`, matching the Hydra output (`1 of 1 target successfully completed, 1 valid password found` per run).

Because both bursts fell inside the same 5-minute bucket used by the detection rule, [detections/ssh-brute-force-t1110.md](../detections/ssh-brute-force-t1110.md) returned a single aggregated result: `failed_attempts=6` for `src_ip=192.168.56.50`, `ssh_user=linux-target`. All 6 failed attempts across both runs were captured by the rule.

## Evidence
TODO: screenshot of the Splunk search results table (`_time`, `src_ip`, `ssh_user`, `failed_attempts`) in `../screenshots/`.

## Takeaway
- Forgetting to timestamp an attack run before executing it is recoverable: the raw log timestamps (`_time` in Splunk, down to the millisecond) let me reconstruct exactly when each burst happened, which attempts were failures versus a success, and which process ID (`sshd[NNNN]`) handled each connection.
- A small, curated password list (with the correct password known in advance) is enough to validate a detection rule end to end, without the time and ingestion cost of a full wordlist like rockyou.txt.
- A fixed 5-minute bucket can merge two distinct attack bursts into one result if they happen close together, which is useful here but is also exactly why the rule's false-negative case (attempts spread further apart, or fewer per window) is a real limitation worth stating rather than hiding.
