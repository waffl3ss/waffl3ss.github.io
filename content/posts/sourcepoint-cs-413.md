---
title: "Updating SourcePoint for Cobalt Strike 4.13: Suspicious Beats Malicious"
date: 2026-11-06
author: "James Pletcher (waffl3ss)"
tags: ["red team", "c2", "cobalt strike", "malleable c2", "evasion"]
summary: "Maintaining a fork of Tyl0us's polymorphic C2 profile generator, and why profile variety matters on red team engagements."
---

On a red team engagement, the command and control channel is the thing you can least afford to lose. Network defenders and EDR products have gotten good at spotting default and well-known C2 configurations, and a public Malleable C2 profile from GitHub is close to a signature at this point. If a hundred red teams use the same profile, it gets detected a hundred times.

SourcePoint, originally written by Tyl0us, is a polymorphic Malleable C2 profile generator for Cobalt Strike. It builds unique profiles on the fly, randomizing any options you leave blank. All credit for the original design goes to Tyl0us. My fork updates it for Cobalt Strike 4.13 and fixes the errors I hit along the way.

## The philosophy: suspicious, not malicious

The idea behind SourcePoint that I keep coming back to is moving C2 indicators from malicious to suspicious. A malicious indicator matches something known bad, and it gets blocked or alerted on automatically. A suspicious indicator might be a little odd, but proving it's C2 takes an analyst's time and research. Blending into normal traffic patterns and avoiding known signatures forces defenders into the slow path, which is exactly where a red team wants them, because it tests whether the SOC actually investigates.

## What a profile controls

Malleable C2 profiles shape nearly everything about a beacon:

- **HTTP traffic**: URIs for GET and POST, headers, how data is encoded and where it sits in the request, and data jitter so responses don't have a fixed size.
- **Timing**: sleep, jitter, and the initial check-in delay.
- **In-memory behavior**: how much memory is allocated for injected content, whether the PE header is copied, and options like the EAF bypass.
- **Infrastructure**: support for redirectors with X-Forwarded-For, CDN-fronted profiles with the cookie values they need, and team server host settings.
- **BeaconGate**, a newer Cobalt Strike feature for proxying API calls through a controlled path, configurable for comms, core, cleanup, or specific APIs.

SourcePoint exposes these as options and fills in randomized values for anything left blank, so each engagement gets a different fingerprint with minimal effort.

## Why maintain a fork

Cobalt Strike changes its profile language between versions. New options appear, old ones change behavior, and a profile that worked last year can fail validation or produce a beacon with unexpected behavior. Keeping the generator current is the difference between spinning up infrastructure in minutes and debugging profile errors on day one of an engagement. Always run the generated profile through `c2lint` before using it.

## Profiles are only part of it

A good profile doesn't save bad infrastructure. Domains need age and categorization, redirectors should only forward traffic that matches the profile and send everything else somewhere boring, and TLS certificates should look like they belong. The profile also has to agree with the redirector rules, which is where most of my own mistakes have happened: change a URI in the profile, forget the redirector, and the beacon never checks in. I keep the generated profile and the redirector config together per engagement so they stay in sync, and both go into the engagement records so the client can match our activity to their logs afterward.

## Defensive takeaways

Polymorphic profiles are a reminder that indicator matching alone won't catch a capable adversary. Defenders get more mileage from behavior: beaconing patterns visible in network metadata even with jitter, unusual processes making outbound web requests, newly registered or uncategorized domains, and memory scanning for injected code. Red team engagements are the place to test whether those detections work, which is the point of making the C2 hard to spot.

Original: [github.com/Tylous/SourcePoint](https://github.com/Tylous/SourcePoint)
Fork: [github.com/waffl3ss/SourcePoint](https://github.com/waffl3ss/SourcePoint)
