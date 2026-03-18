# Lab Results and Analysis

## Test Context

- Target: FreeFloat FTP Server (Version 1.00) running on lab environment
- Command family tested: `MKD`, `USER`
- Input generation: Radamsa mutational fuzzing from valid command seeds
- Harness: Python TCP sender with per-payload execution and console output

## Execution Summary

- Total payloads sent: ~200+ (per command set)
- `USER` mutations: generated and tested
- `MKD` mutations: generated and tested
- Target port: TCP/21
- Observation source: client-side script output (responses and connection behavior)

## Observed Behavior

### Normal Behavior

Most mutated inputs resulted in expected FTP error responses:

- `500 command not understood` → invalid syntax after mutation  
- `331 Password required` → partially valid `USER` commands  

This indicates that the server correctly handles many malformed inputs at the protocol level.

---

### Abnormal Behavior

A specific payload (`mkd-49.txt`) triggered a connection reset:

```text
[!] CRASH with mkd-49.txt -> [Errno 104] Connection reset by peer
```

Observed characteristics:
- The connection was abruptly closed by the server
- No valid FTP response was returned
- Behavior differs from standard protocol error handling

## Reproducibility

The crash was tested multiple times using the same payload:

```bash
python3 scripts/send_fuzz.py samples/mkd-49.txt
```

Result:
- The connection reset occurred consistently
- Confirms deterministic abnormal behavior for this input
- 
## Example Crash Pattern

- Command: `MKD`
- Payload type: mutated directory name with non-standard byte sequences
- Effect: immediate connection reset
- Reproducibility: consistent across multiple executions

This suggests that the issue is triggered during command parsing rather than during later processing stages.

## Why This Likely Happens

Based on observed behavior (without internal instrumentation), possible causes include:

- Improper handling of malformed or non-ASCII input
- Lack of input validation on command arguments
- Unsafe string operations in command parsing

The abrupt connection reset suggests the server encountered an unhandled condition rather than gracefully rejecting the input.

## What It Means for Vulnerability Discovery

A reproducible connection reset is a strong indicator of a bug

It may correspond to:
- A denial-of-service (DoS) condition
- A deeper memory handling issue (requires further analysis)

However:
- No debugger or memory analysis was used
- No direct evidence of memory corruption or code execution

Therefore, this should be classified as:

```bash
Potential vulnerability (crash-inducing input), pending deeper analysis
```

## Limitations of This Fuzzing Approach

- No server-side instrumentation (no debugger, no logs)
- Limited to single-command fuzzing (no session/state fuzzing)
- No protocol-aware mutation (purely mutational fuzzing)
- Root cause cannot be confirmed without deeper analysis