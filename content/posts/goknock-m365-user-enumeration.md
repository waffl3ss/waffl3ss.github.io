---
title: "GoKnock: Validating Microsoft 365 Usernames Through OneDrive and Teams"
date: 2026-11-06
author: "James Pletcher (waffl3ss)"
tags: ["microsoft 365", "username enumeration", "teams", "onedrive", "tools"]
summary: "Confirming which users exist in a tenant without sending a single authentication attempt."
---

Once you have a list of likely usernames for an organization, the next question is which ones are real. The obvious way to find out is to try logging in, but authentication attempts generate sign-in logs and count toward lockout. On an external test or red team, burning attempts on accounts that don't exist is wasteful and noisy.

GoKnock validates usernames against Microsoft 365 using OneDrive and Microsoft Teams, without authenticating as the target user at all. It's the Go version of my earlier Python tool, KnockKnock.

Utilizing tools to create a list of usernames, such as SpiSuite, is useful, but when you get a list of 50,000 potential users, conducting password spraying against something like M365 causes smart lockouts and detections, as well as potentially causin accounts to actually get locked out. Narrowing down a list of potential users from 50,000 potential users to a list of 200 verified and valid users prevents those issues on an engagement, as well as speeding up the process.

## OneDrive enumeration

Every licensed user with OneDrive gets a personal site at a predictable URL built from the tenant name and the username, with the dots and @ in the address replaced by underscores. Requesting that URL returns a different response for users who exist than for users who don't. No credentials are sent to the target account, so nothing shows up in that user's sign-in logs.

The catch is that you need the tenant's `.onmicrosoft.com` name, which isn't always the same as the email domain. Microsoft keeps changing how that can be discovered, and GoKnock now uses a headless browser to find it, or you can supply it manually. Also note that a user only has a OneDrive URL after their OneDrive has been provisioned, so a miss doesn't always mean the user doesn't exist.

## Teams enumeration

Teams has a user search that external users can query when federation with outside organizations is allowed, which is the default in many tenants. With a valid Teams token from any Microsoft account, GoKnock searches for each target address and records which ones resolve to real users.

With the right token, Teams also returns presence information: availability, device type, and the out-of-office message. Out-of-office messages are a surprisingly good source of intel. They name coworkers, managers, and phone numbers, and they tell you who is away, which is useful context for social engineering pretexts.

Getting a working token is the fiddly part, because Microsoft keeps changing what each token can access. The interactive option, `-t proxy`, opens a browser, lets you sign in to your own account, and captures the right token automatically.

## Workflow

When both modules run, users confirmed by OneDrive are removed before the Teams check so nobody gets checked twice. The output is a clean, validated list ready for the next step, which is usually a careful password spray timed against the lockout threshold.

## Reading the results honestly

Validated usernames are a means, not a finding by themselves, and I write them up that way. The report records how many names went in, how many were confirmed by each method, and which method confirmed them, because that tells the client exactly which setting exposed the information. If Teams confirmed most of the list, the fix is external access. If OneDrive did, the conversation is about accepting that the behavior is inherent to the service and compensating elsewhere. It also keeps me honest about the difference between "these accounts exist" and "these accounts are vulnerable," which are very different statements in a report.

## Defensive takeaways

This is the kind of finding clients push back on because "it's just usernames." The point is that it removes guesswork for an attacker and makes spraying quieter and more effective. The fixes:

- Restrict Teams external access to an allow list of partner domains instead of open federation.
- Review whether external users can see presence information.
- Assume valid usernames are known, and invest in phishing-resistant MFA and password spray detection.
- Be thoughtful about what goes in out-of-office messages.

Repo: [github.com/waffl3ss/GoKnock](https://github.com/waffl3ss/GoKnock)
