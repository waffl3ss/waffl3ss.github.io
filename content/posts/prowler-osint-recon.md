---
title: "Prowler: External Recon and Document Metadata in One Go Binary"
date: 2026-11-06
author: "James Pletcher (waffl3ss)"
tags: ["osint", "reconnaissance", "external pentest", "metadata", "tools"]
summary: "Subdomains, zone transfers, SMTP user checks, web discovery, and harvesting usernames from public document metadata."
---

The start of every external test and social engineering engagement is the same: figure out what the organization actually has exposed and who works there. That usually means running five or six recon tools, then hunting through search engines for public documents and running exiftool against everything you download to pull usernames and software versions. It's valuable work and it's tedious every single time.

Prowler is a single-binary OSINT toolkit in Go that runs that whole process as a pipeline, built for authorized assessments. Searching for and extracting metadata from files has also been a thorn in the side of pentests for a while, with different tools being created and abandoned over the years. This method provides a straight forward interactive browser mode to get the files and the information.

## Recon phases

The `recon` command runs five phases, each of which can be skipped:

1. **Domain identification.** Certificate transparency logs from crt.sh, DNS enumeration, subdomain brute forcing with SecLists support, WHOIS and RDAP lookups, and netblock discovery. Certificate transparency is consistently one of the best sources, because organizations issue certificates for internal-sounding hosts that never show up anywhere else.
2. **Zone transfers.** AXFR checks against every discovered nameserver. It's rare to find one open now, but when you do, it hands over the entire zone, so it's always worth the few seconds.
3. **SMTP enumeration.** VRFY, EXPN, and RCPT TO testing, with catch-all detection so you don't treat "accepts everything" as "every user exists."
4. **Web app discovery.** HTTP and HTTPS probing with page titles and technology fingerprinting, deduplicated by final URL so redirects don't inflate the results.
5. **Cloud keywords.** Generates keyword lists from the domain and subdomain patterns to feed cloud_enum for storage bucket and cloud resource discovery.

## Document metadata

Phase six, the `scan` command, is the part I built Prowler for. Organizations publish PDFs, Word documents, spreadsheets, and slide decks constantly, and those files carry metadata: author names, usernames, the software and version used to create them, and creation dates.

Prowler searches Google, Bing, and DuckDuckGo for documents on the target domain using a stealth browser, downloads them concurrently with MD5 hashing to deduplicate, and extracts metadata natively from PDF and Office XML formats, with optional exiftool for richer output. It then pulls out usernames and software versions and filters out noise.

Search engines don't love automated queries, so there's a headed mode that pauses for you to solve a CAPTCHA, adjustable delays between queries, per-engine toggles, and a mode that skips search entirely and processes a list of URLs you already have.

## What it gets you

Usernames from metadata often reveal the internal naming convention, like first initial plus last name, which turns a list of employee names into a list of likely account names for password spraying. Software versions show what's on employee workstations, which helps when crafting payloads for a phishing engagement. Old documents sometimes include internal hostnames or file paths in their metadata too.

## Staying passive where it matters

Most of what Prowler does is passive or close to it: certificate transparency, public search engines, and public documents never touch the target's infrastructure. The SMTP, zone transfer, brute force, and web probing phases do touch it, which is why every phase can be skipped. On engagements where the rules of engagement limit active testing, or where OSINT is scoped separately from the external test, I run only the passive phases and note exactly which phases ran in the report. Knowing which steps generate traffic toward the client is part of staying inside scope.

## Defensive takeaways

Strip metadata from documents before publishing them; most document workflows and many web platforms can do this automatically. Review certificate transparency for your own domains to see what you're exposing. Disable VRFY and EXPN on mail servers, make sure zone transfers are restricted, and keep an inventory of internet-facing web applications, because the forgotten ones are where external tests find their footholds.

Repo: [github.com/waffl3ss/Prowler](https://github.com/waffl3ss/Prowler)
