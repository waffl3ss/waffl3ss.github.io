---
title: "WARP: Consolidating Wireless Assessments Into One Binary"
date: 2026-11-06
author: "James Pletcher (waffl3ss)"
tags: ["wireless", "wifi", "evil twin", "WPA", "tools"]
summary: "PMKID capture, handshakes, enterprise evil twin, WPS, and rogue AP hunting in a single Go daemon, with scope enforced in code."
---

Wireless assessments have always been a pile of tools held together with notes. airodump-ng for discovery, aireplay-ng for deauth, hcxtools to convert captures, eaphammer for enterprise evil twin, something else for WPS, and a spreadsheet to keep track of which networks are actually in scope. Each tool has its own dependencies, and on site you're usually running from a laptop or a small device in a closet where installing Python packages is the last thing you want to deal with.

WARP, the Wireless Assessment and Rogue Platform, is my answer: a single Go daemon with thin clients that covers capture, enterprise attacks, and site survey, built for authorized assessments under a signed statement of work.

## What it captures

WARP doesn't crack. It writes hashcat-ready files for a separate rig, which keeps the field device small and the GPUs where they belong.

- **PMKID** captures need no client at all. The access point gives up the PMKID in the first message of the handshake, which makes it the fastest attack on a WPA-Personal network.
- **Four-way handshakes** for WPA, WPA2, and WPA3-Personal when a client joins, which is what targeted deauthentication is for.
- **WPS Pixie Dust**, recovering the PIN offline from a single exchange.
- **Enterprise evil twin**, including cloning a real network's RADIUS certificate so the rogue presents something that matches, to capture credentials from clients that don't validate the server properly.
- **Hidden network decloaking** to recover the network name.

The handshake parser is strict about what it accepts, and only takes real EAPOL-Key frames, so bad captures don't waste cracking time.

## Scope enforced in code

The part I care most about is the scope model. You start an engagement from the client's list of in-scope network names, and every attack action goes through a scope gate in the daemon. The terminal dashboard, browser interface, and scripting interface all hit the same check, so there's no back door around it. Listening and hunting work on anything, because passive observation is how you find rogue access points, but active attacks only go to in-scope networks. Adding a network to scope is recorded in an audit log.

In wireless testing, the neighbor's network is always in range. Enforcing scope in the tool instead of in the tester's memory is the right way to do it.

## Rogue AP hunting

Finding rogue access points is often more valuable to a client than cracking their PSK. WARP lets you mark suspicious BSSIDs, run rogue classification, and hunt a specific access point by signal. A rogue that broadcasts the corporate network name from the parking lot is a finding worth reporting on its own.

## Built for the field

It's a static binary with no CGO, so it runs on amd64 or arm64 without matching a glibc version on whatever hardware you deployed. Root is only needed to reconfigure adapters; read-only commands run unprivileged. There's a terminal dashboard for on-site work and a dependency-free browser interface served on loopback by default, and everything is scriptable with JSON output.

## A note on how it was built

I started WARP myself, but the nl80211, EAP, and WPS internals were taking a very long time, so I built it out with Claude Code doing the heavy lifting on implementation. The design, the scope model, and the hardware validation are mine. I say so in the README because it's true, and because I think it's where offensive tooling is going.

## Defensive takeaways

Use WPA3 or long random PSKs, disable WPS, require clients to validate the RADIUS server certificate on enterprise networks, and run wireless intrusion detection that can spot rogue access points broadcasting your SSIDs.

Repo: [github.com/waffl3ss/warp](https://github.com/waffl3ss/warp)
