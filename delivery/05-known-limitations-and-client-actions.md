# Known Limitations & Client Actions Required — PRJ-0001

This document exists so nothing about this project's current limits is hidden or downplayed. Three things are needed from you before the solution can be completed and deployed. Nothing below has been guessed, assumed, or filled in on your behalf.

## 1. Candidate evaluation criteria

**What's needed:** a description of what actually makes a candidate score well for you — specific skills, years of experience, keywords, or a weighted mix of factors. This was the one question your original job post asked applicants directly, and it's the one thing that shapes the entire screening layer.

**Why it matters:** the evaluation workflow is fully built to *read* your criteria from a configurable source at run time — but that source currently holds zero entries. Without this, the system has no basis to score anyone, by design; it will not guess on your behalf.

**What's already true regardless:** the mechanism itself (read criteria → build evaluation prompt → score → structured result) is built and tested. Only the content of "what to look for" is missing.

## 2. Resume format mix

**What's needed:** are the resumes you receive typically text-based PDFs, or do a meaningful share arrive as scanned images?

**Why it matters:** the built extraction step is designed for text-based PDFs. Scanned images require OCR (optical character recognition) — meaningfully different technology than what's currently built. Knowing your actual mix determines whether that's needed at all, and how much.

**What's already true regardless:** if your resumes are predominantly text-based (the common case), no further work is needed here.

## 3. n8n target environment

**What's needed:** do you already have an n8n instance we should deploy to? If yes, we'll need a way to connect to it. If no, hosting needs to be decided as a separate step before deployment.

**Why it matters:** deployment cannot proceed against an assumed or guessed environment — this platform's own governance requires actually confirming what the target supports before deploying anything to it, precisely to avoid deploying something that turns out to be incompatible with your actual setup.

**What we will NOT do:** ask you to send credentials through this document or any persistent project record. Once an instance is identified, access will be arranged through a secure channel appropriate at that time — never stored here.

## What resolving these unlocks

| Once you provide... | This becomes possible |
|---|---|
| Evaluation criteria | Full validation of AI scoring; criteria can be entered into the already-built configuration sheet |
| Resume format mix | Final confirmation of extraction coverage; OCR scoping if needed |
| n8n target environment | Real compatibility checking, then deployment |

## What does NOT require anything further from you

Everything described in `03-testing-and-validation-summary.md` as validated — the intake, storage, recording, duplicate detection, notification, human-review-gate, and error-handling design — is complete and does not wait on the three items above.
