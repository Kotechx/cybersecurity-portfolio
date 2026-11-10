# Blue — TryHackMe

- Completed: 2026-11-09
- Platform: TryHackMe
- Type: Beginner CTF (Windows / SMB)
- Skills: service enumeration, vulnerability validation, remote exploitation, credential dumping, offline hash cracking

## What this covers
A Windows Server box left running an outdated SMB service. The machine is
vulnerable to a well-known 2017 SMBv1 remote code execution flaw (the same class
of bug behind the WannaCry outbreak). The goal is to go from an unauthenticated
network position to full administrative control of the host, then recover and
crack a local account's password.

## Key concepts
- **Legacy services are liabilities.** SMBv1 has no place in a modern network;
  its mere presence, plus disabled SMB signing, signals a poorly hardened,
  legacy-era system before any exploit is attempted.
- **Validate before you exploit.** A vulnerability scanner script can confirm the
  specific flaw is present, so exploitation is a verified step rather than a guess.
- **Kernel-level exploits skip privilege escalation.** This flaw executes in the
  context of the operating system itself, so the initial foothold already holds
  the highest privilege level — no separate escalation step is needed.
- **A hash is as weak as the password behind it.** Dumping a local account's
  password hash is only half the story; a short, common password falls to a
  wordlist in seconds once the hash is in hand.

## My approach
1. Enumerated the host with a full port/service scan and confirmed an exposed,
   outdated SMB service with weak configuration.
2. Ran the scanner's SMB vulnerability scripts to validate the specific flaw was
   present before committing to an exploit.
3. Used the matching exploit module to gain remote code execution, landing
   directly at the system's highest privilege level.
4. Upgraded the basic shell to a fuller post-exploitation session to enable
   credential access.
5. Dumped the local account hashes and cracked a user's hash offline against a
   common wordlist.

*(No flags, credentials, IPs, or exact commands — those stay in my private notes.)*

## What this reinforced
This was a clean end-to-end example of exploiting a legacy, unpatched service.
The habit worth keeping is confirming a vulnerability with a scanner before
exploiting it, and recognizing that a basic shell is only a starting point — the
real value comes from upgrading it into credential access and post-exploitation.

## How a SOC would detect this
- **Source:** network sensor (IDS/NSM) + Windows host logs (Sysmon).
- **Pattern:** any SMBv1 traffic on the network, a signature match for the known
  SMB exploit, an internal server opening an outbound connection to an external
  host on an uncommon port, and process-level reads against the credential store.
- **Detection ideas:**
  - Alert on **any SMBv1 traffic** — it shouldn't exist in a modern environment.
  - Alert on **credential-dumping behavior**: reads against the credential
    process or the SAM hive by anything that isn't the OS (Sysmon Event ID 10,
    Windows Event 4663).
  - Alert on **anomalous egress**: an internal server initiating outbound
    connections on high, uncommon ports.

## What I'd learn next
Stand this attack up against my SIEM home lab and actually write and tune the
detection rules above against real logs, instead of only theorizing them.
Capturing the exploitation traffic to study the SMB packet signature firsthand
is the natural follow-up.
