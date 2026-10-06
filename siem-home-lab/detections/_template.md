# [Rule name]

- **ATT&CK technique:** [ID and name]
- **Data source:** [index / sourcetype / event IDs]
- **Hypothesis:** what malicious behavior this should catch

## SPL
```
index=... | ...
```

## Threshold and tuning
Why this threshold, and what data it was based on.

## False positives / false negatives
What legitimate activity could trigger it, and what it would miss.

## Validation
Which of my own attack runs it was tested against, and the result.
