# Radamsa Cheat Sheet for Service Fuzzing

## Quick Commands

```bash
radamsa input.txt
radamsa -n 100 input.txt
radamsa -n 1000 seed.txt -o out-%n.txt
```

- -n → number of mutations
- -o → save outputs (%n = index)

## Seed Corpus Preparation

```bash
printf "USER anonymous\\r\\n" > corpus/user.txt
printf "PASS guest\\r\\n" > corpus/pass.txt
printf "MKD testdir\\r\\n" > corpus/mkd.txt
```

- Always use valid protocol inputs
- Include \r\n (critical for FTP)

## Generate Multiple Fuzz Cases

```bash
mkdir -p samples
radamsa -n 500 corpus/user.txt -o samples/user-%n.txt
radamsa -n 500 corpus/mkd.txt -o samples/mkd-%n.txt
```

#### Use when:
- you need reproducibility
- you want to isolate crash payloads

## Replay a Specific Case

```bash
python3 scripts/send_fuzz.py samples/mkd-271.txt
```

- Confirms if a payload consistently breaks the service
- Required before reporting a vulnerability

## Batch Send Loop

```bash
for f in samples/*.txt; do
  python3 scripts/send_fuzz.py "$f" >> logs/run.log 2>&1
done
```

- Automates testing
- Logs responses and failures

## Minimal Netcat Probe

```bash
radamsa corpus/user.txt | nc 192.168.56.20 21
```

- Fast but not reproducible
- Use only for quick checks

## What to Watch For

- Connection reset → parser failure
- No response → possible crash
- Timeout → hang / deadlock
- Partial response → parsing issue

## Practical Tips

- Start with valid protocol seeds before mutating
- Fuzz one command family at a time for cleaner triage
- Preserve payload indices for reproducibility
- Snapshot VM before campaigns
- Validate crash candidates with replay runs
- Log transport errors separately from process crashes
- Pair fuzzing logs with Event Viewer timestamps
- Use debugger attachment for exploitability assessment

## Common Pitfalls

- Missing \r\n → input ignored
- Sending too fast → false positives
- Not isolating target → unstable results
- Losing track of payload indices
- Mixing different command fuzzing in one run

## When to Use Radamsa

#### Use when:
- Testing simple protocols (FTP, HTTP basic)
- Quick fuzzing without setup
- Targeting legacy services

#### Avoid when:
- You need coverage-guided fuzzing
- Protocol is complex/stateful
- You need precise control over structure