---
layout: post
title: "Using Clawdbot to Write Jekyll Posts from Your Phone"
description: "How I set up an AI assistant to create blog posts via Signal messages, bringing mobile-first writing to a static site workflow."
date: 2026-01-26 14:19:00 +0100
published: true
categories: technology
tags: ai jekyll automation
lang: en
---

*This post was fully written by Claude Opus via Signal as a test of the Clawdbot publishing workflow.*

One of the friction points with running a Jekyll blog is that publishing requires access to your development machine. You need to create a markdown file, add the right front matter, commit, and push. It's not something you can easily do from your phone while waiting for coffee.

Today I solved that problem using [Clawdbot](https://github.com/clawdbot/clawdbot), an open-source AI assistant that connects to messaging platforms like Signal, Telegram, and WhatsApp.

## The Setup

The workflow is simple:

1. **Send a Signal message** to my Clawdbot instance with the post idea
2. **Clawdbot formats it** as a proper Jekyll post with front matter
3. **Clawdbot commits and pushes** to GitHub

The assistant has access to my blog's repository structure, so it knows the correct format for posts, the categories I typically use, and the front matter fields I need.

## Why This Works

Static sites are great for many reasons—speed, security, simplicity—but they traditionally sacrifice the convenience of mobile publishing that platforms like Substack or Medium offer. This setup brings back that convenience without giving up control over my content.

The AI handles the tedious parts:
- Generating the filename in `YYYY-MM-DD-title.md` format
- Adding consistent front matter
- Formatting the markdown properly
- Committing and pushing to GitHub

I just focus on the ideas.

## The Technical Details

Clawdbot runs as a local gateway on my Mac, connecting to Signal via `signal-cli`. When I message it, the request goes to Claude (Anthropic's AI), which has access to tools that can read and write files in a sandboxed workspace.

To give it context about my blog, I cloned my repository into its workspace. Now it can see my existing posts and match the style and structure—and push changes directly.

## Trying It Yourself

If you run a Jekyll blog and want to experiment with this workflow:

1. Install Clawdbot and connect it to your messaging platform of choice
2. Clone your blog repo into the assistant's workspace
3. Start sending post ideas

The barrier between "having a thought" and "publishing it" just got a lot smaller.

---

*This post was drafted via Signal message and published by Clawdbot.*
