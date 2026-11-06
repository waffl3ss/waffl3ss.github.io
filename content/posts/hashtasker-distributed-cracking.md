---
title: "HashTasker: Distributed Hashcat for a Pentest Team"
date: 2026-11-06
author: "James Pletcher (waffl3ss)"
tags: ["password cracking", "hashcat", "infrastructure", "tools"]
summary: "Splitting hashcat jobs across every cracking node we own, with a web UI the whole team can use."
---

Password cracking is one of the highest-value things a pentest team does with hardware. NTDS dumps, Kerberoast tickets, NetNTLMv2 from Responder, WPA handshakes from wireless work: they all end up on a GPU eventually. The problem on a team is never the cracking itself. It's coordination. Who is using which rig, whose job is hogging it, where the wordlists live, and how long until your hashes come back.

HashTasker is how I solved that for my team. It's a distributed hash cracking platform in Go: one server binary with a web UI and API, and lightweight workers on every cracking node. Each job's wordlist keyspace is split across all online workers, so a job finishes as fast as the whole fleet can run it instead of waiting on one box.

Originally, having multiple cracking rigs you could run one job on a single rig. now its split between the two rigs, allowing quicker speeds at cracking passwords. Aditionally, other tools like hashtopolis, didnt have LDAP authentication, and also only runs a single job at a time. While that is fine for individuals, when you work in a pentesting team running 15 engagements simultaneously, theres no time to wait for another job to finish before starting your own. HashTasker allows multiple jobs to run at the same time, with an additional queue system built in for when VRAM is maxxed out. 

## How the work gets split

When a consultant submits a job, the server divides the wordlist into chunks and hands one to each worker. There are two distribution modes:

- **push**, the default, where the server splits the wordlist and workers download their chunk over HTTP. It works anywhere with no shared storage.
- **nfs**, where workers read wordlists from a shared mount and the server just tells each one which range to run using hashcat's `--skip` and `--limit`. With huge wordlists, job creation becomes nearly instant because nothing has to be copied.

Multiple jobs run concurrently across the fleet. Jobs support multiple rulesets, custom wordlist uploads, `--username` mode for dumps that include account names, and per-job potfile control so one client's results never leak into another's.

## Built for a team, not a single operator

The features that matter most are the boring operational ones:

- **LDAP and Active Directory login** with group-based admin mapping, so access follows the team's existing accounts.
- **Live progress per worker**, showing speed, ETA, and whether a node is running, queued, overloaded, or failed.
- **Webhook notifications** when jobs are created, start, finish, fail, or get cancelled, so nobody sits watching a progress bar.
- **Exports** of cracked or all hashes to CSV, and DPAT-compatible potfiles for NTLM jobs, which feeds directly into password audit reporting.
- **Self-maintenance**: jobs older than 45 days are purged and orphaned files cleaned up hourly. Client hashes are sensitive data, and retention should be automatic, not something someone remembers to do.

## Analytics that improve the next engagement

The admin analytics show crack rates by wordlist and hash mode, runtimes, and peak usage. NTDS dumps are excluded by default, because one domain dump with tens of thousands of hashes would drown out everything else. Over time this tells the team which wordlists and rules actually earn their GPU time against real client passwords, so the default attack plan gets better with every engagement.

## Why it matters for clients

Cracking results are some of the most persuasive evidence in a report. Showing a client that a large share of their domain passwords fell in hours to common wordlists and rules makes the case for longer minimum lengths, banned password lists, and phasing out NTLM far better than any policy recommendation alone. A DPAT report built from HashTasker's potfile export makes that analysis quick to produce.

## Deployment notes

The worker uses only the Go standard library and cross-compiles cleanly. The server needs CGO for SQLite. The server also refuses to start if the session key, database password, or admin password are left at their example values, which is a small thing, but default credentials on internal tooling are exactly what we exploit on everyone else's networks.

Repo: [github.com/waffl3ss/HashTasker](https://github.com/waffl3ss/HashTasker)
