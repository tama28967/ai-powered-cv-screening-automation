# Project Overview — AI-Powered Resume Screening & Recruitment Automation

**Project:** PRJ-0001 · **Source:** Engineering Briefcase (Opportunity OP-0002) · **Status as of:** 2026-08-04

## What this solution does

Automates the first stage of resume screening: applications arrive through an online form with a PDF resume, get validated and stored in Google Drive, have their content extracted, are evaluated by an AI model against your own criteria, get a score and recommendation, are checked for duplicates, and are recorded in Google Sheets — **your team reviews every result before any candidate is notified**, and no hiring decision is ever made by the system. *(Source: Engineering Briefcase §2–4, Accepted Proposal §6.)*

## What was agreed

Fixed price, scope as enumerated in the accepted Proposal — form intake, validation, Drive storage, extraction, AI evaluation, Sheets recording, duplicate detection, notifications, human review gate, configurable criteria, error handling, modular design. Full scope: Briefcase §6.

## Current status (evidence-traced — see the Testing & Validation Summary and Known Limitations documents for detail)

| Stage | Status |
|---|---|
| Design (Engineering Blueprint) | Complete |
| Implementation | Complete — 8 artifacts built |
| Testing | Complete (static/structural) — 5 defects found and fixed |
| Validation against your requirements | **Partially validated** — most of the design is confirmed sound; two items depend on information only you can provide |
| Target environment compatibility | **Not yet established** — no target environment has been identified |
| Deployment | **Not performed** |

**This project has not been deployed to a live system yet.** Everything described in this package is a thoroughly designed, built, and tested — but not yet running — candidate solution.

## What you need to know before anything further can happen

Three things, explained in full in `05-known-limitations-and-client-actions.md`. Nothing below has been guessed or assumed on your behalf.

## Document guide

| Document | For |
|---|---|
| `01-project-overview.md` (this document) | Everyone |
| `02-architecture-and-implementation.md` | Your technical team / future maintainer |
| `03-testing-and-validation-summary.md` | Technical + business |
| `04-deployment-status-and-operations.md` | Whoever will operate this once deployed |
| `05-known-limitations-and-client-actions.md` | You — what's needed from you next |
| `manifest.md` | Full audit list of everything in this package and where it comes from |
