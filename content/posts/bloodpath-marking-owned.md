---
title: "bloodpath: Marking Owned Principals in BloodHound Straight From Cobalt Strike"
date: 2026-11-06
author: "James Pletcher (waffl3ss)"
tags: ["active directory", "bloodhound", "cobalt strike", "red team", "tools"]
summary: "A small Aggressor script that closes the gap between credentials you've captured and the attack paths you can actually use."
---

Some tools are frameworks. Others solve one annoying problem and stay small. bloodpath is the second kind, and it fixes something that slows down every Active Directory engagement I run through Cobalt Strike.

Credentials pile up when you extract them through beacons. With bloodhound data and path tracing, you can use bloodpath to mark users as owned, simplifying the attack path identification for an engagement.

## The problem

BloodHound's value isn't just showing every attack path in a domain. It's showing the paths available from where you are right now. To do that, it needs to know which users and computers you control, which BloodHound calls "owned" principals. Once you mark accounts as owned, you can query for the shortest paths from owned principals to Domain Admins or any other high-value target.

During an engagement, credentials pile up fast in Cobalt Strike's credentials tab: hashes from dumps, plaintext from Mimikatz, Kerberos tickets cracked offline. Marking each of those as owned in BloodHound by hand means switching windows, searching for the node, and clicking through the menu for every account. In practice, people skip it, and then they miss a path that was available the whole time because one account they already had was never marked.

## How bloodpath works

bloodpath is an Aggressor script, `bloodpath.cna`, paired with a small Python helper, `owned.py`. From the credentials tab in Cobalt Strike, you select accounts and send them to BloodHound. The script calls the helper, which connects to the BloodHound Neo4j database and sets the owned property on the matching user nodes. The script console shows each node it marks, so you have a record of what changed.

Setup is minimal: put both files in the Cobalt Strike directory, make the helper executable, and set the Neo4j credentials. If your team shares a BloodHound server, point the helper at it, and everyone sees the same owned set. That shared view is the real benefit on a multi-operator red team, because one operator's captured credential becomes everyone's starting point.

## Why it matters for methodology

Attack path analysis is only as good as the data you put in. Keeping owned principals current turns BloodHound from a map of the whole domain into a live answer to "what can I do next with what I have?" It also helps with reporting. When you write the attack narrative, the owned markers show which accounts were compromised and in what order, which makes the path from initial access to the objective easy to explain to the client.

## Owned paths on a real engagement

The workflow that makes this worth it looks like this. A dump on a workstation produces a handful of hashes. Instead of guessing which matters, I mark them all owned and run the shortest-path query from owned principals to high-value targets. Often the useful account is not the obvious one. It's a helpdesk user with local admin on a server where a privileged account has a session, or a service account in a group with an ACL nobody noticed. Seeing that immediately, instead of after an afternoon of manual digging, is the difference between moving laterally that day or the next.

## Limitations

bloodpath works with legacy BloodHound backed by Neo4j. BloodHound Community Edition manages owned principals through its own API rather than a property in Neo4j, so it needs a different approach. The helper also stores database credentials in a file, so treat the Cobalt Strike directory as sensitive.

## Defensive takeaways

The defensive lesson is the same one every BloodHound finding teaches: attack paths exist because of accumulated permissions and sessions. Reducing local admin rights, keeping privileged accounts off workstations, using tiered administration, and running BloodHound yourself to find and break the shortest paths to Tier 0 are what make a captured credential a dead end instead of a stepping stone.

Repo: [github.com/waffl3ss/bloodpath](https://github.com/waffl3ss/bloodpath)
