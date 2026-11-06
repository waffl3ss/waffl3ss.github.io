---
title: "SpiSuite: Building Target User Lists From LinkedIn and Hunter.io"
date: 2026-11-06
author: "James Pletcher (waffl3ss)"
tags: ["osint", "username enumeration", "password spraying", "social engineering", "tools"]
summary: "Combining NameSpi and JobSpi into one Go tool that turns company employee data into ready-to-use username lists."
---

Password spraying and phishing both start with the same input: a list of people who work at the target. Getting that list right matters. Too few names and you miss the one account with Summer2026! as its password. Too many bad names and you waste attempts and risk lockouts. SpiSuite is the tool I use to build that list, and it's a Go combination of two older Python tools of mine, NameSpi and JobSpi.

## Where the names come from

- **LinkedIn.** SpiSuite pulls employees from the company's LinkedIn page through your own LinkedIn account. Results depend on your account's network, so a well-connected account sees more employees. It can look up the company ID from the company name and give you a list of candidates to choose from.
- **Hunter.io.** Pulls names and emails based on your subscription level. The free tier is limited to 10 results, but Hunter's most useful output is the organization's email format, which tells you how to turn names into addresses.
- **Statistically likely names.** An optional list of roughly 250,000 statistically likely names for when you want broad coverage. It's a very long list, so use it intentionally.

## Cleaning and mangling

Raw scraped names are messy. SpiSuite deduplicates, strips accents, normalizes capitalization, and then mangles names into whatever format the target uses. There are 10 built-in mangle modes covering the common conventions, like first.last, first initial plus last name, and last name plus first initial. Combined with the email format from Hunter.io, that produces a list you can feed directly into username validation and then spraying.

## Job information

The JobSpi side adds context for each employee: current job title, role, time at the company, location, and full name. That's useful in two ways. For phishing, it lets you target people by role, like new hires who haven't been through awareness training or IT staff who'll have privileged access. For spraying, it helps prioritize, because a service desk account is a more interesting target than a summer intern.

## Practical limits

LinkedIn rate limits heavily, especially on the experience enumeration that powers the job data, so you'll only get job details back for a subset of users. If you run enumeration repeatedly in a short window, results drop off sharply. The system you run it from should have previously logged into LinkedIn through a browser, or you'll hit a security check. I don't recommend putting your LinkedIn password in the YAML config; leave it blank and SpiSuite prompts for it.

Debug mode exists for when LinkedIn inevitably changes something.

## Handling the data responsibly

An employee list with job titles and locations is personal data, even when every piece of it came from a public profile. I keep it inside the engagement folder, include only what the report needs, and delete it with the rest of the engagement data at the end. Rules of engagement matter here too: some clients exclude executives or specific roles from phishing or spraying, and the job data makes it easy to filter those people out before anything is sent.

## How it fits the methodology

SpiSuite is the first step in a chain: build the list, validate which usernames are real with something like GoKnock, then spray carefully against the organization's lockout policy if spraying is in scope and authorized. Each step narrows the list and reduces risk to the client's accounts.

## Defensive takeaways

You can't stop employees from having LinkedIn profiles, so assume attackers have your employee list and email format. The controls that matter are downstream: phishing-resistant MFA everywhere, smart lockout and password spray detection, banned password lists that cover seasons and company names, and awareness training aimed at the roles attackers actually target.

Repo: [github.com/waffl3ss/SpiSuite](https://github.com/waffl3ss/SpiSuite)
