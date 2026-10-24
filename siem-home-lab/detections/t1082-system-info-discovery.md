# System Information Discovery (Atomic Red Team T1082-1)

- **ATT&CK technique:** T1082 (System Information Discovery)
- **Data source:** index `sysmon`, EventCode 1 (process creation), host `windows-target`, via Sysmon (SwiftOnSecurity config) and the Splunk Universal Forwarder
- **Hypothesis:** execution of `systeminfo.exe`, especially spawned from `cmd.exe` launched by `powershell.exe` rather than from a user double-clicking it, is consistent with scripted/automated reconnaissance rather than routine manual administration.

## SPL
```
earliest=-5m@m latest=now index=sysmon host=windows-target EventCode=1
| search Image="*systeminfo.exe" OR CommandLine="*systeminfo*"
| table _time, host, Image, CommandLine, ParentImage, ParentCommandLine
```

## Threshold and tuning
No count threshold is used here (unlike the SSH brute-force rule); this rule fires on any single execution of `systeminfo.exe` or any command line mentioning it. That is a deliberate choice for this first version: `systeminfo` is rare enough in normal workstation use that even one occurrence is worth a look, but it is also a legitimate admin and troubleshooting command, which is why false positives are expected and documented below.

## False positives / false negatives
- **False positive:** any legitimate administrator or support technician running `systeminfo` manually (e.g. for troubleshooting) would also trigger this rule. The current version does not distinguish automated execution from manual use; a stronger version would require specific parent-process patterns (e.g. `ParentImage=*powershell.exe* AND ParentImage of ParentImage != explorer.exe`) or correlate with other suspicious activity in the same session.
- **False negative:** an attacker gathering the same information through a different tool or API (e.g. WMI queries, PowerShell cmdlets like `Get-ComputerInfo`, or a compiled binary that doesn't invoke `systeminfo.exe`) would not be caught by this rule, since it is scoped narrowly to the `systeminfo` command itself.

## Validation
Tested against Atomic Red Team test T1082-1 (System Information Discovery), executed on windows-target via `Invoke-AtomicTest T1082-1` on 2026-10-22. The test ran `systeminfo` and a registry query (`reg query HKLM\SYSTEM\CurrentControlSet\Services\Disk\Enum`) chained with `&` inside `cmd.exe /c`, itself launched by `powershell.exe`. Sysmon captured the full process chain (EventCode 1) for all three processes, and the query above returned the `systeminfo.exe` and `cmd.exe` events with the expected `CommandLine` and `ParentImage` values. See [../scenarios/t1082-system-info-discovery.md](../scenarios/t1082-system-info-discovery.md) for the full write-up.

Saved as a Splunk scheduled alert ("T1082 - System Information Discovery"), running every 5 minutes, triggering when Number of Results > 0.
