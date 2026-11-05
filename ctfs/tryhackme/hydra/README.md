# Hydra — TryHackMe

- Completed: 2026-11-04
- Platform: TryHackMe
- Type: Beginner (tooling)
- Skills: brute force (Hydra), HTTP POST form auth, SSH, Linux

## What this covers
Using Hydra to brute-force credentials against two different services: an HTTP
login form and SSH. The point is less "crack the password" and more learning how
Hydra talks to each service and why the syntax differs.

## Key concepts
- **HTTP forms are submitted by the field `name` attribute, not the placeholder.**
  The placeholder is display-only text and is never sent to the server, so Hydra's
  `http-post-form` body must use the real `name` values.
- **The failure string matters.** `http-post-form` needs a condition (`F=...`)
  that matches the exact message the app returns on a failed login. If it's wrong,
  Hydra reports false positives or misses the real hit.
- **SSH needs no failure string.** The SSH module relies on the protocol itself to
  tell success from failure, so the command is much simpler than the web form.
- **Thread count for SSH.** SSH drops connections under heavy parallelism, so a low
  thread count (`-t 4`) is more reliable than the default.
- **Wordlist:** rockyou.

## My approach
1. Found the login path at `/login`.
2. Built an `http-post-form` attack. First attempt failed — I had used the
   placeholder text as the field names instead of the `name` attribute.
3. Corrected the field names to match the form's real `name` values and got a valid
   credential; logged into the web app.
4. Re-ran Hydra against SSH with the `ssh` module, logged in over SSH, and read the
   flag file on the host.

Command shape (sanitized):

```
hydra -l <user> -P <wordlist> <TARGET> http-post-form "/login:username=^USER^&password=^PASS^:F=<failure-text>"
hydra -l <user> -P <wordlist> <TARGET> -t 4 ssh
```

*(No flags, found passwords, or target IPs — those stay in my private notes. I even
genericize the username, out of habit.)*

## Where I got stuck (and what it taught me)
Using the placeholder text instead of the `name` attribute. Fixing it forced me to
actually read how an HTML form builds a POST body — the server only ever sees
`name=value` pairs.

## How a SOC would detect this
This is the valuable half. Brute force is **loud** — every attempt is a log line.
- **Web form brute force →** web server access logs: a burst of POST requests to
  `/login` from one IP in seconds, almost all failures.
- **SSH brute force →** `/var/log/auth.log`: a flood of `Failed password for <user>
  from <ip>` entries from a single source.
- **Detection idea:** threshold on failed authentications per source IP per minute.
  This is exactly the failed-SSH-login pattern I detect in my own SIEM home lab —
  the attack side of a detection I already built.

## What I'd learn next
Tune the failed-login threshold in my SIEM so a slow brute force (spread out to stay
under the limit) still gets caught.
