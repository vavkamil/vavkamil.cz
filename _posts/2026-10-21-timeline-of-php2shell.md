---
layout: post
title: "Timeline of PHP2SHELL: What a shit show"
date: 2026-10-21 00:00:00 -0000
categories: ['Security research', 'Responsible disclosure', 'Ethical hacking']
tags: [RCE, PHP, security, exploit, CVE-2026-109618, CVE-2026-109619]
author: vavkamil
image: "/assets/img/posts/2026-10-21-timeline-of-php2shell.png"
---

PHP2Shell was supposed to be a coordinated security disclosure. Instead, we got a leaked embargo, an emergency release, an accidentally published RCE PoC, vendors racing to ship patches, and GitHub going down right in the middle of it all.

---

And that's just the last 24 hours of a story that goes all the way back to 2019. Here's how October 21, 2026, turned into an absolute shit show.

# What a shit show :)

- 2019 - Related PHP memory-handling bug reported
- 2020 - Bug fixed and closed
- 2024 - PHP rediscovers a related bug, considers RCE theoretical, won't fix

## 2026

- July 30 - Calif starts auditing PHP through OpenAI's Patch the Planet
August 1 - Calif discovers RCE paths using OpenAI's GPT-5.6 Sol
August 11 - Calif reports the RCE to PHP
August 11 - OpenAI's Codex Security independently reports the same root cause
August 14 - Calif provides a working RCE PoC
August 19 - Wiz independently discovers related PHP vulnerabilities using Gemini
August 24 - Wiz reports its findings to PHP
Patch release scheduled for October 22
October 20, 15:20 UTC - Hacker News user leaks embargoed CVE details
October 20, 18:26 UTC - watchTowr publicly confirms the embargo
October 20, 19:33 UTC - PHP announces an emergency security release forward by one day
October 20, 20:00 UTC - Calif mistakenly publishes a PHP2Shell PoC video on X
October 21, 03:02 UTC - Cloudflare activates PHP2Shell WAF protection by default
October 21, 07:41 UTC - Chainguard prematurely publishes container images containing the security fix
October 21, 07:43 UTC - Cloudflare publicly reports early exploitation attempts
October 21, 13:32 UTC - Wiz announces its PHP2Shell research on X and takes the spotlight
October 21, 13:34 UTC - PHP publishes GitHub security advisories
October 21, 13:36 UTC - PHP publishes its official security announcement
October 21, 14:00 UTC - Scheduled PHP security release
October 21, 14:06 UTC - PHP GitHub releases published for all four patched versions
EDIT:
October 21, 14:19 UTC - PHP Docker pipeline triggered
October 21, 14:31 UTC - PHP Docker pipeline finished
October 21, 14:52 UTC - GitHub goes down
October 21, 15:59 UTC - Patched images were being built
October 21, 16:30 UTC - First patched Docker images reportedly become available
October 21, 16:57 UTC - GitHub starting to recover
October 21, 16:57 UTC - GitHub mostly recovered
October 21, 17:55 UTC - All of Docker Hub's patched tags available
What a day ...
