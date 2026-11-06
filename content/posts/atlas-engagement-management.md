---
title: "Atlas: One Place to Keep an Entire Pentest Engagement"
date: 2026-11-06
author: "James Pletcher (waffl3ss)"
tags: ["reporting", "engagement management", "methodology", "tools"]
summary: "Ingesting scan data, tracking hosts and findings, and getting it all into the report without copy and paste."
---

Every pentest generates a mess. Nmap XML in one folder, Nessus exports in another, Nuclei output, web probe results, a text file of credentials, a list of discovered usernames, and notes scattered across terminals. By reporting time, the hardest part isn't remembering what you found, it's reassembling it.

Atlas is the self-hosted web app I built to keep all of that in one place. Each engagement is its own project with hosts, services, findings, credentials, and discovered users, and everything can be exported in the formats our reporting workflow already uses.

Engagements often have notes in numerous areas and spread through different applications and tooling. Instead of having 20 files that you have to read through or grep data and write basic notes, this helps track assets and tool outputs in one location, with exports to numerous reporting platforms.

## Getting data in

The core idea is that you shouldn't type what a tool already told you. Atlas ingests Nmap XML, Nessus, Nuclei, BBOT, HTTPX, Lair JSON, and its own raw JSON format, and populates hosts and services automatically. Uploads can be tagged at import time, so you can mark a scan as "external," "DMZ," or "segment B" and filter by it later.

Hosts get color-coded status tracking, which turns into a simple way to see coverage at a glance: what has been looked at, what has something interesting, and what's been ruled out. Each host has a detail page with tabs for services, hostnames, credentials, web directories, and web probes.

One feature I use constantly is the port host list. From the Services page, one click downloads `hosts_port_445.txt` or whatever port you need, ready to feed straight into the next tool. That alone removes a lot of grep and awk from an engagement.

## Tracking what matters

- **Findings** carry severity from critical to informational, CVSS scores, affected hosts, CVE references, and full description, remediation, and evidence fields.
- **Credentials** are logged with the hosts and services they work on, with CSV export.
- **Discovered users** can be added one at a time, in bulk, or from a file, which keeps OSINT and enumeration output connected to the rest of the engagement.

Keeping credentials tied to where they work is a small design choice with a big payoff. When you write the attack narrative for the report, the chain of "found this here, reused it there" is already recorded.

## Getting data out

Atlas exports PlexTrac assets and findings, Lair, and Atlas raw JSON. The raw format exports an entire project and imports it into another Atlas instance, which makes handing an engagement off between consultants clean instead of a zip file of random folders. More export options to come.

## Running it

It runs from Docker or source with an embedded SQLite database, so there's no database server to stand up. Multi-user support assigns people to projects as owners or viewers, which matters because engagement data is client data and access should be scoped. An optional ngrok integration exists for remote access without port forwarding. If you use it, treat the tunnel like any other exposed service holding sensitive data: strong credentials, and turned off when you don't need it.

## Why it matters

Good reporting is part of the job, not paperwork after it. Findings are only useful to a client when the evidence is complete, the affected assets are accurate, and the remediation is specific. Most reporting errors I've seen come from data getting lost or mangled between tools and the report. Centralizing engagement data while the work happens is the fix, and it also makes peer review and QA easier because a second consultant can see the same picture.

It's also dark mode only, on purpose. There's a wiki in the repo with workflows and format references.

Repo: [github.com/waffl3ss/Atlas](https://github.com/waffl3ss/Atlas)
