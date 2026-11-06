---
title: "MFASweep in Go: Finding the Microsoft 365 Doors MFA Doesn't Cover"
date: 2026-11-06
author: "James Pletcher (waffl3ss)"
tags: ["microsoft 365", "entra id", "mfa", "conditional access", "tools"]
summary: "Why I ported dafthack's MFASweep to Go, and what single-factor gaps it finds after you have a valid password."
---

Getting a valid Microsoft 365 password on an engagement doesn't mean much if MFA stops you at the login page. But "MFA is enabled" and "MFA is enforced everywhere" are very different statements. Conditional access policies are written by people, and they often cover the web portal while leaving another service, client type, or protocol single-factor.

MFASweep was written by Beau Bullock (@dafthack) in PowerShell to test exactly that. It takes one set of credentials and tries them against a series of Microsoft services, reporting which ones let you in without a second factor. All credit for the original research and tool goes to him. What I did was port it to Go.

## Why port it

The original is PowerShell, which is great on a Windows box and awkward everywhere else. Most of my work happens from Linux, and I wanted a single cross-platform binary I could drop on any system without setting up PowerShell or dealing with module dependencies. Go compiles cleanly for Linux, macOS, and Windows from one codebase.

## What it checks

- **Microsoft Graph API**
- **Azure Service Management API**
- **Exchange Web Services** using basic authentication
- **The Microsoft 365 web portal** with six different device user agents: Windows, Linux, macOS, Android, iPhone, and Windows Phone
- **ActiveSync** using basic authentication
- **ADFS**, when the tenant is federated

The user agent checks are the most interesting part. A conditional access policy that requires MFA for Windows and macOS but forgets Linux or mobile platforms will let the same password through when the request claims to come from an uncovered device. Running all six shows that gap immediately.

## Use it carefully

Every check is a real authentication attempt against the target account. A full run is 10 attempts, or 11 with ADFS. If the password is wrong, you're burning attempts toward lockout on a real employee's account. That's why I only run it with credentials I've already validated, why individual checks can be run one at a time, and why the tool asks for confirmation before a full run unless you explicitly skip it. Check the lockout threshold with the client before running it, and document every attempt for the report.

## Reading the output

A success on one service doesn't automatically mean full access. Graph and Azure Service Management access can expose very different things depending on the user's roles, and a web portal login with a specific user agent may still be limited by session controls. I treat each single-factor result as a lead, confirm what it actually grants in a controlled way, and report what was demonstrated separately from what was merely exposed. A client deserves to know the difference between "this protocol accepted the password" and "we read the CFO's email."

## How it fits an engagement

The typical chain is: build a user list, validate usernames, spray carefully, then run MFASweep against any account that cracks. If something comes back single-factor, that service becomes the access path, and the finding moves from "a user had a weak password" to "a weak password gave access to email and data despite MFA."

## Defensive takeaways

- Write conditional access policies that require MFA for all cloud apps and all device platforms, then carve out exceptions deliberately, instead of listing what's covered.
- Block legacy authentication. Basic auth for EWS and ActiveSync has been retired by Microsoft for most tenants, but it's worth confirming rather than assuming.
- Use the What If tool in Entra ID to test policies against different client types.
- Move toward phishing-resistant MFA so a password alone is never enough.

Original: [github.com/dafthack/MFASweep](https://github.com/dafthack/MFASweep)
Go port: [github.com/waffl3ss/MFASweep](https://github.com/waffl3ss/MFASweep)
