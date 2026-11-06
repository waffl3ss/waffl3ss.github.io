---
title: "Rummage: An SMB Client Built for Looting Shares on Internal Pentests"
date: 2026-11-06
author: "James Pletcher (waffl3ss)"
tags: ["internal pentest", "smb", "file shares", "opsec", "tools"]
summary: "smbclient with a real interface: browse, preview, flag loot, and export it straight into the report."
---

File shares are where internal pentests are quietly won. Scripts with hardcoded passwords, unattend files, KeePass databases, backup configs, HR spreadsheets that should never have been on a department share. Automated spidering tools are great for a first pass, but at some point you have to actually look around, and doing that in `smbclient` means typing `cd` and `ls` hundreds of times and keeping track of what you found in a text file.

Rummage is the tool I wanted for that part of the job: a terminal UI SMB client that lets me browse shares like a file manager, preview files without downloading them, flag the interesting ones with notes, and export everything I flagged for the report. Went with a TUI interface with no font dependencies so that it can be run on any system you are using for the engagement.

## How it works

You point it at a target with an impacket-style string, like `CORP/jsmith:'pass'@10.0.0.5`, and it opens on a list of shares with tested read and write access. From there you browse directories, hit space to preview a file with syntax highlighting, queue downloads, and flag anything worth reporting.

Authentication covers what you actually have mid-engagement: passwords, pass-the-hash with an NT or LM:NT hash, Kerberos with a password, hash, or ccache, and guest. It also supports SOCKS4, 4a, and 5 proxies, with Kerberos KDC traffic going through the proxy too, which matters when you're working through a pivot. Stall detection handles flaky tunnels, and DFS referrals are followed.

There are also non-interactive commands, `shares`, `ls`, and `get`, for scripting and quick checks.

## Sessions and reporting

Every engagement eventually turns into a report, so Rummage keeps state. Session files save the target, user, domain, proxy, and your last location, so you can reopen exactly where you left off. Named sessions group multiple file servers under one engagement folder, and `rummage export` turns flagged items and notes into markdown for the findings write-up. That last part is what I care about most: the evidence trail is built while I work instead of reconstructed afterward.

## OPSEC: know what you put on the wire

The README includes a full OPSEC table, and I think every offensive tool should ship one. Testers should know which actions generate which events, especially on engagements where the blue team is watching or honeypots are deployed.

A few examples:

- **Connecting** creates a network logon (4624 type 3) on the target and NTLM or Kerberos events on the domain controller, and your session stays visible in `Get-SmbSession` while you're connected.
- **Listing shares** with access checks does a tree connect and a write-access probe on every share. Nothing is created, but the requested write access can trip a honey share monitored for write attempts. `--no-check` limits this to the share enumeration call.
- **Browsing a directory** never opens the files inside it. Names, sizes, and timestamps come back in the listing, so a honey file is only touched if you preview, download, or read its ACL.
- **Previewing** opens the file with read access and reads only the first 512KB by default.

Knowing that browsing doesn't open files is the difference between tripping a canary and not. It also makes the findings more useful for defenders, because I can tell them exactly which audit settings would have caught me.

## What defenders should take from it

Every finding Rummage produces is really an access control finding. Broad groups like Domain Users or Authenticated Users with read access to sensitive shares are the root cause most of the time. Regular access reviews, removing credentials from scripts in favor of managed identities or gMSAs, and file share auditing with SACLs on high-value folders all close the gap. Honey files and folders with list or read auditing are cheap and they work.

## Getting it

It's a single static Go binary with no runtime dependencies and no Font requirement. No prebuilt binaries, so build it yourself. If you cant build it yourself, then you probobly shouldnt be using it.

Repo: [github.com/waffl3ss/rummage](https://github.com/waffl3ss/rummage)
