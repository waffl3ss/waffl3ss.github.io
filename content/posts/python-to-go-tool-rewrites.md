---
title: "Rewriting My Python Pentest Tools in Go: NameSpi, JobSpi, and KnockKnock"
date: 2026-11-06
author: "James Pletcher (waffl3ss)"
tags: ["golang", "python", "tool development", "osint", "methodology"]
summary: "What changed when my recon tools moved from Python scripts to Go binaries, and what I'd tell anyone thinking about doing the same."
---

My first public tools were Python. NameSpi built employee username lists from LinkedIn and Hunter.io. JobSpi pulled job titles and tenure for those employees. KnockKnock validated usernames against Microsoft 365 through OneDrive and Teams. They worked, people used them, and NameSpi in particular picked up a decent following.

Then I started learning Go, and those three tools became SpiSuite and GoKnock. The Python versions are archived now. This post is about why I did it and what changed for the tools and for how I use them on engagements. I utilized the rewriting of these tools as my method in learning Go.

## The deployment problem

The biggest reason was deployment. A Python tool on a pentest means the right Python version, a virtual environment or pipx, requirements that conflict with something else on the box, and the occasional package that needs a compiler. That's fine on my own Kali VM. It's a real problem on a jump box, a client-provided system, or a dropbox where I don't want to install anything.

A Go tool is one static binary. Cross-compiling for Windows, Linux, or macOS is an environment variable. Copy it over and run it. For offensive tools that need to run wherever the engagement puts me, that alone justified the rewrite. Not to mention I knew how these tools worked in and out, so it was a good way to learn Go.

## Combining tools that belonged together

NameSpi and JobSpi were separate because I wrote them at different times, not because they did different jobs. Both logged into LinkedIn, both looked up the same company, and running them back to back meant two logins and twice the requests. LinkedIn rate limits aggressively, so that duplication actually cost results.

SpiSuite merged them with a shared company lookup and a shared login session. One run, one login, both outputs. The lesson is that the rewrite is a chance to fix the architecture, not just translate the code line by line.

## Concurrency without the pain

KnockKnock's threading was a recurring headache. Several of its updates were some version of "threads fixed again." Go's goroutines and channels made concurrent enumeration much simpler to get right, and GoKnock's checks run in parallel without the fragility. Progress bars came along with it, which matters more than it sounds when you're validating thousands of usernames and want to know whether it's working.

## What didn't get easier

Rewriting doesn't fix external dependencies. Microsoft still changes how tenant names are discovered and what Teams tokens can access. LinkedIn still changes its responses and rate limits. Both tools needed updates after the rewrite for reasons that had nothing to do with the language. Anyone maintaining tools that scrape or query third-party services should expect that maintenance forever.

I also dropped some features along the way, like the legacy Skype output in KnockKnock, because Microsoft finally got rid of Skype. A rewrite is a good time to delete things.

## Releases and build-it-yourself

One decision I made with the Go versions was to stop shipping prebuilt binaries. If you can't build a Go project with `go build`, you probably shouldn't be running offensive tools against a client. It also means you know exactly what you're running, compiled from source you can read, which is a habit worth keeping for any tool that touches client information. Prebuilt offensive binaries floating around also get signatured quickly, so building your own has a practical benefit too.

## Advice if you're considering it

- Rewrite tools you actually use and maintain. Archive the rest honestly so people know where to go.
- Fix the design while you're in there. Merge duplicate work and remove dead features.
- Keep the command-line interface familiar so existing users can switch easily.
- Point the old repo at the new one in the README.

Repos: [SpiSuite](https://github.com/waffl3ss/SpiSuite), [GoKnock](https://github.com/waffl3ss/GoKnock), and the archived [NameSpi](https://github.com/waffl3ss/NameSpi), [JobSpi](https://github.com/waffl3ss/JobSpi), and [KnockKnock](https://github.com/waffl3ss/KnockKnock).
