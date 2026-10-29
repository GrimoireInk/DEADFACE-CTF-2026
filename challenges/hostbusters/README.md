
# Hostbusters — DEADFACE CTF 2026

## Category Overview

This directory contains technical investigations and challenge write-ups for the Hostbusters category of DEADFACE CTF 2026.

The challenges focus on Linux system enumeration, host investigation, service account permissions, and related security techniques.

Each completed challenge has its own directory containing a detailed README with the investigative methodology, terminal commands, evidence, findings, and recovered flag.

## Challenges

| Challenge | Points Available | Status | Write-up |
|---|---:|---|---|
| Foothold on Charon | 10 | ✅ Independently Solved | [View Write-up](foothold-on-charon/README.md) |

## Techniques Explored

### Linux Host Enumeration

[Foothold on Charon](foothold-on-charon/README.md) — Used `whoami`, `hostname`, `pwd`, and `id` to identify the current service account and host environment.

### Linux File and Directory Analysis

[Foothold on Charon](foothold-on-charon/README.md) — Examined directory listings and file permissions to locate and retrieve the initial foothold flag.

---

[← Return to Main Repository](../../README.md)
