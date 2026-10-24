# System Information Discovery via Atomic Red Team (T1082)

## What I ran

On windows-target, after an initial attempt to simulate a network-based attack (RDP brute force) failed due to tooling limitations (see Takeaway below), I pivoted to a locally-executed technique using the Atomic Red Team framework:

```powershell
Invoke-AtomicTest T1082-1
```

This runs atomic test T1082-1 ("System Information Discovery"), which executes:
```
systeminfo
reg query HKLM\SYSTEM\CurrentControlSet\Services\Disk\Enum
```
both chained inside a single `cmd.exe /c` call, launched from a PowerShell session.

## What happened

The test completed with exit code 0 at 2026-10-22, in the window 16:50:12-16:50:31 UTC. In Splunk, the [T1082 detection rule](../detections/t1082-system-info-discovery.md) found the full process chain captured by Sysmon:

```
powershell.exe
  -> cmd.exe   ("cmd.exe" /c systeminfo & reg query HKLM\SYSTEM\CurrentControlSet\Services\Disk\Enum)
       -> systeminfo.exe
```

All three processes were logged as Sysmon EventCode 1 (process creation) with full command lines and parent-child relationships intact, confirming that Sysmon's process-creation logging captures this kind of reconnaissance chain even when the final command (`systeminfo`) is wrapped inside intermediate shells.

## Evidence
TODO: screenshot of the Splunk search results table (`_time`, `host`, `Image`, `CommandLine`, `ParentImage`, `ParentCommandLine`) in `../screenshots/`.

## Takeaway

- **Pivoting from a failed attack is normal and worth documenting.** I first attempted an RDP brute-force attack against windows-target using ncrack and then Hydra. Both failed at the protocol level: ncrack's RDP module timed out during the handshake (even after disabling NLA), and Hydra's RDP module (which it labels as "experimental") failed to establish the connection at all (`freerdp: The connection failed to establish`). Rather than keep fighting known tooling limitations, I switched to Atomic Red Team, which tests detection coverage directly without depending on a specific network protocol implementation.
- **Windows Defender actively interferes with Atomic Red Team's own installation**, flagging and quarantining several atomic test binaries (for techniques like T1055 and T1218.003) as malicious before I ever ran an attack. I had to add a Defender exclusion for the `C:\AtomicRedTeam` folder to get a clean install. This is itself a small piece of evidence that the built-in antivirus has baseline signature coverage against known offensive-security tooling.
- **A single `systeminfo` command is weak evidence on its own** (any admin might run it), but the full parent-process chain (PowerShell spawning cmd.exe spawning systeminfo.exe) makes the pattern more suspicious than the command alone, which is why the detection rule's documented false-positive rate should be taken seriously rather than treated as a solved problem.
