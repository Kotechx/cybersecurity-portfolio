# Corridor — TryHackMe

- Completed: 2026-08-12 (revisited: 2026-11-05)
- Platform: TryHackMe
- Type: Beginner CTF (web)
- Skills: web enumeration, hash identification, IDOR / broken access control

## What this covers
A web app that presents a row of "doors," where each door is reached through a
URL identified by an MD5 hash. The intended set of doors is limited, but the
identifiers are predictable, which makes it possible to reach a resource that
isn't meant to be linked.

## Key concepts
- **MD5 is an unsalted, deterministic hash.** The same input always produces the
  same output, so a hash of a small or common value can be reversed with a lookup
  service instead of real computation.
- **Predictable identifiers = IDOR.** When a resource is addressed by a value an
  attacker can guess or generate, access control that trusts the identifier
  (instead of checking permissions) can be bypassed.
- **Out-of-range values.** The visible doors mapped to a sequence of numbers, so
  forging the identifier for a value *outside* that sequence exposed a hidden
  resource.

## My approach
1. Opened the app and mapped the doors; each one was addressed by an MD5 hash.
2. Reversed a couple of the hashes with a lookup service and saw they decoded to
   sequential numbers — a clear pattern.
3. Counted the available doors and reasoned that a value outside that range might
   point somewhere not linked in the UI.
4. Generated the hash for that out-of-range value and used it to reach the hidden
   resource.

*(No flags, exact identifiers, or target details — those stay in my private notes.)*

## What this reinforced
No new concept for me here — this was a warm-up that reused MD5, hash lookups, and
predictable-ID manipulation I already knew. The useful takeaway was naming the
bug class precisely: broken access control through a guessable identifier.

## How a SOC would detect this
- **Source:** web server access logs.
- **Pattern:** a single IP requesting many different resource paths in quick
  succession = enumeration / resource brute-forcing.
- **Detection idea:** alert when one source requests an unusual number of distinct
  resource IDs in a short window, or requests an ID outside the expected range.

## What I'd learn next
Write this detection as a query in my SIEM home lab against real web logs.
