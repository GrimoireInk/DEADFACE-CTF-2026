# Aurora Patient Zero — DEADFACE CTF 2026

## Overview

Aurora Patient Zero is a series of mobile security challenges from DEADFACE CTF 2026 involving the investigation of Aurora Health's Patient Portal iOS application.

The challenge scenarios center on targeted phishing attacks against patients whose personal information may have been exposed through the application.

This directory documents my investigation of the application, including static analysis, information disclosure, forensic techniques, challenge solutions, and lessons learned.

## Challenge Progress

| Challenge | Points | Status | Write-up |
|---|---|---|---|
| Info Leaks | 25 | ✅ Solved | [View Write-up](info-leaks/README.md) |
| Down the Rabbit Hole | — | ✅ Solved | [View Write-up](down-the-rabbit-hole/README.md) |
| SQL Skillz | 50 | ✅ Solved | [View Write-up](sql-skillz/README.md) |
| Back to Basics | 15 | ✅ Solved | [View Write-up](back-to-basics/README.md) |

*Additional challenges will be added as they are investigated.*

## Investigation Environment

- **Operating System:** macOS
- **Application Platform:** iOS
- **Analysis Approach:** Static application analysis
- **Documentation:** Markdown, VS Code, and GitHub

## Key Technical Concepts

Throughout this challenge series, I am developing a deeper understanding of:

- iOS application packages (`.ipa`) and application bundles (`.app`)
- Apple property lists (`Info.plist`)
- Static application analysis
- Base64 encoding and JSON data structures
- Application data exposure and insecure storage
- Mobile application security and forensic investigation techniques

## Repository Organization

```text
aurora-patient-zero/
├── README.md
└── info-leaks/
    └── README.md
```

Each individual challenge write-up documents the challenge objective, technical concepts, investigation methodology, verified findings, and final solution.

## Security Disclaimer

All patient information referenced in these challenges is fictional and provided for the purposes of DEADFACE CTF 2026.

This documentation is intended for educational cybersecurity research and authorized CTF investigations.

Challenge flags may be hidden behind collapsible spoiler sections to allow readers to explore the solutions independently.

## Lessons Learned

This project reinforces the importance of systematic investigation, evidence-based analysis, and understanding how seemingly harmless application resources can expose sensitive information.

As additional challenges are completed, this section will be expanded to summarize new techniques, recurring vulnerabilities, and key takeaways.
