---
title: "GoVibe: LDAP-Only Active Directory Enumeration and BloodHound Collection"
date: 2026-11-06
author: "James Pletcher (waffl3ss)"
tags: ["active directory", "ldap", "bloodhound", "enumeration", "tools"]
summary: "A Go rewrite of Vibe with pass-the-hash, Kerberos, channel binding support, and tiered BloodHound collection."
---

Active Directory enumeration is the first thing I do once I have any valid credential on an internal test. Users, groups, computers, SPNs, password policy, trusts, and the ACL relationships BloodHound turns into attack paths. There are plenty of tools for this, but I kept running into the same set of problems: Python dependency issues on whatever box I was working from, memory blowing up on large domains, and collectors that make far more noise than the job needs.

GoVibe is my Go rewrite of the Vibe enumeration tool, built around those problems. GoVibe works over proxy, with passwords, hashes, and kerberos tickets. It also upgrades the output to provide clean HTML and JSON files instead of just a database-like access. Additionally, GoVibe can provide bloodhound data from the same information that was queried from GoVibe in the first place, and no require additional connections and queries to LDAP. 

## What it collects

Over LDAP or LDAPS, GoVibe enumerates users, groups, computers, SPNs, password policies, and trusts. Output comes as JSON and HTML, similar to ldapdomaindump (but much cleaner HTML), and an interactive shell lets you query the collected data from a disk-backed cache. There's also an offline mode that loads cached data into the shell without touching LDAP again, which is useful for writing the report after you've lost access.

Authentication covers password, pass-the-hash with an NT or LM:NT hash, and Kerberos via a ccache. Traffic can go through a SOCKS5 proxy, including the Kerberos KDC traffic, so it works through a pivot.

## Scaling to large domains

Two design decisions make it work on big environments:

- **Streaming to disk.** Enumeration results stream to disk instead of being held in memory, so a domain with over 100,000 objects doesn't exhaust RAM on a small jump box.
- **LDAP range retrieval.** Active Directory only returns 1,500 values of a multi-valued attribute per request, so large group memberships silently truncate if a tool doesn't page through them. GoVibe handles range retrieval automatically, which means you don't miss the user who's member number 1,501 of a privileged group.

It also supports DCs that enforce LDAP channel binding, using the TLS server end point binding from RFC 5929. As more organizations turn on channel binding, collectors that don't support it just fail.

## Tiered collection

I wanted control over how much noise a collection makes, so GoVibe has three tiers:

1. **Lean**, with no BloodHound flags. Only the attributes GoVibe needs for its own output. Fastest and quietest.
2. **BloodHound**, with `-b` for BloodHound CE or `--bloodhound-legacy` for the older format. This adds security descriptors, delegation, SID history, LAPS, and RBCD attributes to the same queries. No extra searches, just more attributes per query.
3. **DCOnly**, with `--dconly`, which adds separate searches for OUs, GPOs, and containers. That's a larger LDAP footprint, so it's opt-in.

That tiering is useful for the report too. You can tell a client exactly what was queried and what detection opportunity it created.

## What I look at first

With the data collected, I go straight to a few things before opening BloodHound: the password policy and lockout threshold, because that sets the rules for any spraying; accounts with SPNs, for Kerberoasting; accounts with pre-authentication disabled, for AS-REP roasting; and descriptions, because admins still put passwords in user description fields. Then BloodHound for the paths. The HTML output is also handy to hand to the client's AD team, because it's readable without any tooling.

## Defensive takeaways

Everything GoVibe collects is readable by any authenticated user by default, which is the point worth making to clients. Restricting what low-privilege users can enumerate is hard in AD, so the more practical controls are reducing attack paths themselves: cleaning up excessive ACLs, removing unconstrained delegation, limiting Kerberoastable service accounts with strong passwords or gMSAs, and enforcing LDAP signing and channel binding. Monitoring for bursts of LDAP queries from unusual hosts is a solid detection opportunity.

Repo: [github.com/waffl3ss/GoVibe](https://github.com/waffl3ss/GoVibe)
