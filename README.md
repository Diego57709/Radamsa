# Fuzzing Windows Services using Radamsa

## Professional Overview

This repository documents a practical vulnerability research workflow for fuzzing a legacy Windows network service with **Radamsa**. The target scenario is a vulnerable FTP daemon (FreeFloat FTP Server v1.00) running in an isolated VM, with command-level fuzzing focused on parser stability and crash discovery.

The objective is not to produce synthetic benchmark data, but to demonstrate a realistic analyst process:

- Build a seed corpus from valid protocol commands
- Mutate input at scale with Radamsa
- Deliver payloads over TCP with a lightweight Python harness
- Detect and reproduce abnormal behavior
- Classify findings and document limitations responsibly

## What Fuzzing Is (Practical Definition)

In this lab, fuzzing means repeatedly sending malformed FTP command variants to a live service and monitoring for behavior outside normal protocol error handling.

Success criteria include:

- Reproducible connection resets
- Service instability or process termination
- Missing or malformed server responses
- State transitions inconsistent with expected FTP behavior

## Why Radamsa for This Use Case

Radamsa is effective in early discovery phases because it quickly generates high-variance, semi-valid mutations that still interact with parser logic.

For Windows service testing, this gives strong value when:

- You need fast protocol fuzzing with minimal setup
- You are testing legacy software without source access
- You want immediate integration into shell and Python workflows

Compared to coverage-guided frameworks, Radamsa trades instrumentation depth for speed and operational simplicity.

## Lab Scenario

| Component | Configuration |
|----------|---------------|
| Target | FreeFloat FTP Server v1.00 |
| Target OS | Windows 11 VM |
| Attacker | WSL |
| Protocol | FTP (TCP/21) |
| Initial Attack Surface | `USER`, `MKD` |
| Delivery Method | Python socket harness (`scripts/send_fuzz.py`) |

## Workflow Overview

1. Prepare protocol-correct seeds with CRLF termination.
2. Generate indexed mutated payloads with Radamsa.
3. Send one payload per connection to the FTP service.
4. Record server responses and transport anomalies.
5. Replay interesting payloads to confirm reproducibility.
6. Classify behavior as expected rejection vs potential vulnerability.

## Current Findings Snapshot

Based on the current lab run and notes:

- Most malformed payloads were handled with expected FTP errors (`500`, `331`).
- A specific `MKD` mutation (`mkd-49.txt`) produced a reproducible abnormal condition.
- The client observed `Connection reset by peer`, indicating abrupt target-side failure handling.
- The behavior is currently classified as **potential vulnerability (crash-inducing input)** pending debugger-backed root-cause analysis.

## Repository Map

- `setup/installation.md`: Radamsa setup, corpus preparation, and screenshot-backed setup steps
- `fuzzing/fuzzing-process.md`: mutation strategy, Python harness workflow, and reproducible crash execution path
- `lab/results.md`: test observations, abnormal behavior analysis, reproducibility notes, and limitations
- `cheatsheet/radamsa-cheatsheet.md`: high-signal command reference for execution and triage
- `images/`: lab evidence (terminal output, execution artifacts, validation screenshots)

## Key Takeaways

- Mutational fuzzing remains highly effective against legacy protocol services.
- Reproducibility is more valuable than raw payload volume.
- Connection resets and silent drops are key triage indicators, not noise.
- Analyst-grade reporting requires separating observed facts from exploitability assumptions.

## Real-World Relevance

This methodology is directly applicable to:

- Internal assessments of unmanaged or legacy network services
- Pre-production parser hardening and regression checks
- Service decommissioning prioritization based on stability risk
- Building vulnerability research portfolios with evidence-driven findings

## Safety and Legal Scope

Run this workflow only in environments where you have explicit authorization.

- Use isolated VMs and non-production networks
- Snapshot targets before testing
- Treat crash-triggering payloads as sensitive security artifacts
