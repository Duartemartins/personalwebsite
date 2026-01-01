---
layout: post
title: "Interessant3 #153 | Claude Code, Offline AI, European Heatwaves"
description: "Three Interesting Things for W/C 2025-08-10"
date: 2025-08-17 21:20:10 +0000
published: true
categories: newsletter
lang: en
image: /assets/images/substack/171220301-social.webp
---

1. **Claude Code is All You Need**

-  Conor Dwyer treats Claude Code as a **Swiss Army knife** for projects, server ops, admin, and even creative writing.
-  _“Vibe coding”_ - letting AI spin up apps from prompts - works surprisingly well under the right constraints, as shown by a SmartSplit clone that runs cleanly in 900 lines of PHP but collapses into bloat when left to over-engineer in Node.js.
-  The “autonomous startup” experiment illustrates both potential and absurdity: Claude autonomously built and deployed a SaaS-style web app, but with shaky business logic and eventual policy blockages.
-  For real-world use, Claude excelled at tasks like migrating a legacy PHP/MySQL app, auto-documenting codebases, handling obscure dependency errors, renaming and merging bank statements, and even helping draft this very article.
-  Yet, the human role remains central: steering, clarifying, forgiving mistakes. Claude is less “independent coder” and more “hyper-diligent junior dev” that thrives on abundant context.
-  [Claude Code is All You Need – dwyer.co.za](https://dwyer.co.za/static/claude-code-is-all-you-need.html)

1. **Manish – I Want Everything Local: Building My Offline AI Workspace**

-  Begins from a simple but radical demand: _no cloud, no remote execution_. The aim - LLMs, code execution, and browsing, all on-device.
-  The stack blends Ollama (local LLMs), Apple’s new **Container** tool for VM-level isolation, Playwright for headless browsing, and a lightweight orchestration layer called coderunner.
-  Privacy is the philosophy: code runs inside isolated containers, results are mapped to shared volumes, and the host system is never touched - eliminating the risk of sensitive data leakage to third-party APIs.
-  The UI story reveals the messy reality of “offline-first” AI: failed attempts at a Mac-native app, a fallback to Electron/Next.js, and workarounds for missing features in open-source tools.
-  Tool-calling support and interoperability remain patchy - some models are advertised as having tool support but don’t actually implement it yet. Workarounds rely on exposing everything via the Model Context Protocol (MCP).
-  Tested use-cases include local video editing, chart generation from CSVs, browser-based research, image manipulation, and GitHub tool installs - all AI-orchestrated but fully offline.
-  Limitations: Apple-only for now, fragile build processes, and bot-detection throttling the headless browser. Yet the project signals a deeper shift: reclaiming compute and privacy from the cloud giants.
-  [Building My Offline AI Workspace – Instavm.io](https://instavm.io/blog/building-my-offline-ai-workspace)

1. **Adaptation to Heatwaves in Europe Outpaces Climate Change**

-  Long-term mortality data suggests Europeans are adapting to heatwaves faster than climate change is worsening them.
-  From 2000–2022, Europe’s heat tolerance increased by about +1 °C every ~18 years, challenging conventional “static” projections of future deaths.
-  Economic growth - especially investments in infrastructure like air conditioning - proved central to this resilience.
-  Eastern Europe showed the sharpest rise in cooling energy demand, underscoring regional disparities in adaptation speed.
-  This doesn’t erase climate risks but does complicate linear catastrophe narratives, highlighting human capacity for adaptation.
-  [Adaptation to Heatwaves in Europe Outpaces Climate Change – Human Progress](https://humanprogress.org/outpacing-climate-change-adaptation-to-heatwaves-in-europe/?__readwiseLocation=)