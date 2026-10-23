---
title: "GPT-5.6 Sol finds decade-old critical flaws in PHP, the language behind 70% of the web"
date: "2026-10-23"
category: "News"
source: "OpenAI / The Rundown AI"
url: "https://openai.com/index/coordinated-disclosure-of-critical-vulnerabilities/"
status: "New"
tags: ["openai", "security", "php", "vulnerability", "gpt-5.6"]
---

## Summary

Two critical vulnerabilities in PHP — the language running roughly 70% of websites — were patched after researchers found them using OpenAI's GPT-5.6 Sol. Security firm Calif found the first bug on August 1; Google-owned Wiz independently caught both weeks later using Sol and Gemini 4 Argon. OpenAI spent two months working on patches with PHP maintainers and calls the find "one of the most impactful security issues in recent memory."

The bugs are over a decade old and let attackers hijack servers with no login required. The NSA issued an advisory urging anyone who can't patch immediately to pull systems offline. Finland, Germany, and the Netherlands took government websites offline, and Cloudflare reports early attacks already emerging.

Here's the uncomfortable part: the model that found these flaws is now generations behind the frontier. A "last-gen" model uncovered holes that a decade of human security review missed. OpenAI says more significant finds are still waiting to be disclosed.

## Key Points

- Two decade-old critical PHP flaws patched; exploitable with no authentication
- Found by Calif (Aug 1) using GPT-5.6 Sol; Wiz independently confirmed with Sol and Gemini 4 Argon
- NSA advisory: systems that can't patch should go offline; Finland, Germany, Netherlands pulled sites
- OpenAI worked patches with PHP maintainers for two months before disclosure
- OpenAI says these are only two of "several significant vulnerabilities" still being fixed

## Blog Angle

The irony writes itself: the same model family that made headlines for hacking Hugging Face just found bugs in 70% of the web. Write "AI found the holes humans missed for ten years" — the defensive side of AI security nobody is covering. Practical angle for readers: what this means for anyone running PHP sites, and why legacy code audits with LLMs should be a standard line item now.
