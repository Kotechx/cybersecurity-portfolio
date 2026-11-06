# RootMe — TryHackMe

- Completed: 2026-10-09 (revisited: 2026-11-05)
- Platform: TryHackMe
- Type: Beginner CTF (web → Linux)
- Skills: enumeration, file-upload filter bypass, reverse shell, SUID privilege escalation

## What this covers
A full foothold-to-root path on a Linux web host: discover a hidden upload form,
bypass a weak upload filter to get code execution, catch a reverse shell, then
escalate to root through a misconfigured SUID binary.

## Key concepts
- **Directory enumeration finds the real attack surface.** The linked site is
  rarely the whole app; brute-forcing paths exposed an unauthenticated upload
  panel that isn't meant to be public.
- **Extension blacklists are weak.** Blocking only `.php` doesn't stop alternate
  extensions the server still hands to the PHP interpreter. An allowlist plus
  content validation (magic bytes) is the correct control.
- **Reverse-shell callback IP.** In a VPN-based lab the payload must call back to
  the attacker's VPN tunnel interface, not a local/NAT address — only the tunnel
  address is routable from the target.
- **SUID interpreters = instant root.** A scripting-language interpreter left with
  the SUID bit can be told to run code as its owner. GTFOBins documents these.

## My approach
1. Port scan → only SSH and HTTP exposed, so the web app was the way in.
2. Directory brute-forcing revealed an upload panel and an uploads directory.
3. Uploaded a PHP reverse shell. The form rejected `.php`, so I tested how the
   filter decided what to block and used an alternate PHP-handled extension it
   missed.
4. Caught the shell as the web user and read the user flag.
5. Enumerated SUID binaries, spotted a scripting interpreter that shouldn't carry
   the SUID bit, and used it (per GTFOBins) to spawn a root shell and read the
   root flag.

Command shape (sanitized):

```
nmap -sV <TARGET>
gobuster dir -u http://<TARGET> -w <wordlist>
find / -perm -4000 -type f 2>/dev/null
```

*(No flags, the exact working extension, the SUID binary name, or target IPs —
those stay in my private notes.)*

## Where I got stuck (and what it taught me)
Two real troubleshooting lessons, both on the reverse shell:
- **Payload encoding.** The shell generator was set to URL-encode its output, so
  the uploaded file was full of `%XX` sequences instead of valid PHP — the server
  returned it as text and nothing executed. Switching encoding off fixed it.
- **Callback IP.** My first payload pointed at a NAT/local address the target
  couldn't route to, so the shell never connected back. The callback has to be the
  VPN tunnel interface.

These two cost me the most time and taught me to read what the payload generator
actually produces before trusting it.

## How a SOC would detect this
Every phase leaves a trace:
- **Upload →** an executable file type written into a web-served directory: alert
  on upload content/extension.
- **Reverse shell →** the web server process (e.g. `www-data`) opening an
  *outbound* connection to an unusual port — web servers shouldn't dial out.
- **Privesc recon →** a service account running `find ... -perm -4000`: no
  legitimate user enumerates SUID binaries.
- **Root shell →** a scripting interpreter spawning a shell as root: anomalous
  process execution.
- **Detection idea:** correlate the upload with an outbound connection from the
  web process, and alert on shells spawned by the web service account.

## What I'd learn next
Build the "web process spawns a shell" and "web process dials outbound"
detections in my SIEM home lab — the defensive side of this exact attack.
