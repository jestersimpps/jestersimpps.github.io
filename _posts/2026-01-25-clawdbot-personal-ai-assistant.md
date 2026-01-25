---
layout: post
title: "Clawdbot: the self-hosted AI assistant everyone's obsessing over"
description: "Deep dive into Clawdbot - the open-source personal AI that lives in your messaging apps, remembers everything, and is causing massive hype"
date: 2026-01-25
author: "Jester Simpps"
categories: [AI]
tags: [AI, Clawdbot, Personal Assistant, Open Source, Automation]
image: /images/clawdbot-cover.png
---

Ok so imagine this. You're texting your AI assistant on WhatsApp at 7 AM, asking it to check your emails and prep a morning briefing. Then you switch to Telegram on your laptop and just continue the same conversation. No context lost. No starting over

That's [Clawdbot](https://clawd.bot/) and it's causing chaos in the AI community right now

## What is Clawdbot?

So [Peter Steinberger](https://steipete.me/) (former founder of PSPDFKit) built this open-source personal AI that runs on your own devices. Unlike ChatGPT or Claude where you visit a website and start fresh every time, Clawdbot lives inside the messaging apps you already use

WhatsApp. Telegram. Discord. Slack. Signal. iMessage. Microsoft Teams

All connected to the same AI brain that remembers everything you've ever told it

The [GitHub repo](https://github.com/clawdbot/clawdbot) exploded to over 8,000 stars in weeks. People are calling 2026 "the year of personal agents" because of it :D

## Why people are losing their minds

Here's what makes it different from every other AI tool you've tried:

**Persistent Memory** - it doesn't forget what you told it yesterday. Mentioned you have a meeting on Friday? It remembers. Your preferences, your projects, your context - all stored locally

**Proactive Outreach** - most chatbots wait for you to type. This one reaches out. Morning briefings. Reminders. Alerts when something you care about happens

**Full Computer Access** - it can do anything you can do on your computer. Browse the web. Send emails. Manage your calendar. Control your smart home

**Self-Hosted** - your data stays on your machine. No corporate cloud storing your conversations

One [MacStories review](https://www.macstories.net/stories/clawdbot-showed-me-what-the-future-of-personal-ai-assistants-looks-like/) put it bluntly: "To say that Clawdbot has fundamentally altered my perspective of what it means to have an intelligent, personal AI assistant in 2026 would be an understatement"

That reviewer burned through 180 million tokens on the Anthropic API testing it. Insane

## Real-world use cases that sound insane

The community is building wild things. Here's what people are actually doing:

**Someone had it buy them a car.** [AJ Stuyvenberg wrote about](https://aaronstuyvenberg.com/posts/clawd-bought-a-car) outsourcing the painful aspects of car shopping to Clawdbot. Research, negotiations, scheduling test drives - all handled through text messages

**Automated morning briefs.** Users wake up to messages with email summaries, Slack highlights, calendar previews, and action items - saved to Obsidian with voice summaries via ElevenLabs

**Stock alerts and news monitoring.** Set up watchers for anything you can describe in a prompt. Weather warnings. Research updates. Important emails flagged before you even check

**Release workflow monitoring.** One developer has it watching GitHub Actions and npm publish - alerting him when builds complete

**Grocery autopilot.** A [community skill](https://github.com/Dicklesworthstone/agent_flywheel_clawdbot_skills_and_integrations) for Picnic pulls order history, infers preferred brands, maps recipes to cart, and completes orders automatically

## The technical setup

Clawdbot runs on a [Gateway architecture](https://docs.clawd.bot/start/getting-started) - basically a single long-running process that owns channel connections and the WebSocket control plane

Getting started is pretty straightforward:

```bash
npm install -g clawdbot@latest
clawdbot onboard --install-daemon
```

The onboarding wizard walks you through model setup (Anthropic, OpenAI, or local models), channel connections, and skill installation

For 24/7 availability, most users run it on a VPS. [Hetzner has servers for about $5/month](https://velvetshark.com/clawdbot-the-self-hosted-ai-that-siri-should-have-been) that handle it fine

Works on Mac, Windows (via WSL2), or Linux. Requires Node 22+

## The skills ecosystem

Skills are markdown files that teach Clawdbot how to use command-line tools. When you enable a skill, Clawdbot can intelligently use that tool across all your connected channels

There's a [public registry called ClawdHub](https://docs.clawd.bot/tools/skills) with 100+ community-built skills. Google Workspace. Meeting notes. Document processing. Perplexity search. GitHub sync

Users report having 40+ skills installed handling everything from home automation to autonomous coding loops

One user described running "Autonomous Claude Code loops from my phone. 'Fix tests' via Telegram. Runs the loop, sends progress every 5 iterations"

Pretty cool

## Browser automation

Clawdbot can [control a dedicated Chrome profile](https://docs.clawd.bot/tools/browser) that the agent manages. It's isolated from your personal browser and controlled through CDP (Chrome DevTools Protocol)

Fill out forms. Navigate sites. Use your logged-in sessions. Book appointments. Order food

You can even run the browser control server on a remote machine and point your Gateway at it for headless automation

## The "Lobster" workflow engine

[Lobster](https://github.com/clawdbot/clawdbot) is a Clawdbot-native workflow shell - a typed, local-first "macro engine" that turns skills and tools into composable pipelines with approval gates

Think of it like IFTTT but powered by AI and running entirely on your machine

## What the community is saying

The reactions have been intense

From [Hacker News discussions](https://news.ycombinator.com/item?id=46748880):

> "At this point I don't even know what to call @clawdbot. It is something new. After a few weeks in with it, this is the first time I have felt like I am living in the future since the launch of ChatGPT"

From Twitter:

> "Today was one of those days that I sort of ran to my computer after dropping off my toddler at daycare. Why? Because I got part-way through setting up @clawdbot last night and it's a portal to a new reality"

> "Me reading about @clawdbot: 'this looks complicated' me 30 mins later: controlling Gmail, Calendar, WordPress, Hetzner from Telegram like a boss"

[Medium posts](https://medium.com/@henrymascot/my-almost-agi-with-clawdbot-cd612366898b) are calling it "My Almost AGI" and describing multi-agent setups across multiple machines that SSH into each other for debugging

## The security concerns

Ok so not everyone is thrilled. And honestly, the concerns are valid

Running an AI agent with shell access on your machine is... spicy

The [official security documentation](https://docs.clawd.bot/gateway/security) acknowledges this upfront: "Clawdbot is both a product and an experiment: you're wiring frontier-model behavior into real messaging surfaces and real tools. There is no 'perfectly secure' setup"

Key risks to consider:

- **Tool blast radius** - could prompt injection turn into shell/file/network actions?
- **Session data privacy** - transcripts are stored on disk. Treat filesystem access as the trust boundary
- **Remote access** - anyone accessing your Telegram could potentially control your computer

Critics on [Michael Tsai's blog](https://mjtsai.com/blog/2026/01/22/clawdbot/) noted: "The lack of security is the big issue for me. I don't trust these companies to have that kind of access. You don't even need a bad actor, just an accidental, incorrect action from the LLM"

The project takes security seriously with [audit tools](https://docs.clawd.bot/gateway/security), token encryption, and sandboxing options. But this is still frontier territory

## How it compares to Claude Code

[Clawdbot and Claude Code](https://docs.clawd.bot/providers/anthropic) are complementary tools:

- **Claude Code** is focused on coding/development tasks in the terminal
- **Clawdbot** is a personal assistant platform that can access multiple services and messaging platforms

You can actually trigger Claude Code from within Clawdbot. Authenticate with an API key or reuse Claude Code CLI credentials

One user described it as "the best agentic system I've used since Claude Code itself"

## The Hacker News mystery

Interesting footnote: despite massive traction on Twitter and Reddit, Clawdbot has struggled to get visibility on [Hacker News](https://news.ycombinator.com/item?id=46662034)

One poster asked "Why does clawdbot not get any love in HN?" and speculated about possible reasons. Multiple submissions with few upvotes or comments

The project seems to have found its audience on social media rather than traditional tech forums. Maybe HN just isn't feeling it?

## Getting started today

Here's the fast path:

1. Install Node 22+ if you haven't
2. Run `npm install -g clawdbot@latest`
3. Run `clawdbot onboard --install-daemon`
4. Follow the wizard to connect your first channel (Telegram is easiest)
5. Start chatting

The [official docs](https://docs.clawd.bot/start/getting-started) are solid. There's also a [setup guide on addROM](https://addrom.com/clawdbot-your-personal-ai-assistant-for-windows-macos-and-linux/) and a [detailed VelvetShark review](https://velvetshark.com/clawdbot-the-self-hosted-ai-that-siri-should-have-been) with step-by-step instructions

[Vercel's AI Gateway](https://vercel.com/docs/ai-gateway/chat-platforms/clawd-bot) also has integration docs if you're using their platform

## Should you care?

Here's my take: Clawdbot represents something genuinely new

We've had chatbots. We've had AI assistants. We've had automation tools

But we haven't had a self-hosted, persistent-memory, multi-channel, proactive AI that can control your computer and reach out to you when something matters

Is it polished enterprise software? No. The [MacStories review](https://www.macstories.net/stories/clawdbot-showed-me-what-the-future-of-personal-ai-assistants-looks-like/) called it "a nerdy project, a tinkerer's laboratory"

Is it a glimpse of where personal AI is heading? Absolutely

The fact that someone used it to buy a car - handling research, negotiations, scheduling - and then wrote a [blog post](https://aaronstuyvenberg.com/posts/clawd-bought-a-car) saying it "sold me on the vision" tells you everything about where this is going

## Key takeaways

- **Clawdbot is an open-source personal AI that lives in your messaging apps** - WhatsApp, Telegram, Discord, Slack, iMessage, and more
- **It remembers everything** - persistent memory means no more starting from scratch
- **It's proactive** - morning briefs, alerts, reminders pushed to you
- **Full computer control** - browser automation, shell access, file management
- **Self-hosted** - your data stays on your machine
- **Active community** - 8,000+ GitHub stars, 100+ skills on ClawdHub
- **Security is a real concern** - wiring an LLM to your computer requires careful thought

The [GitHub repo](https://github.com/clawdbot/clawdbot) is open source under MIT license. The [official site](https://clawd.bot/) has demos and the [documentation](https://docs.clawd.bot/) is comprehensive

Whether you dive in now or wait for the ecosystem to mature, Clawdbot is worth watching. This is what personal AI looks like in 2026

---

**Sources:**
- [Clawdbot Official Site](https://clawd.bot/)
- [Clawdbot GitHub Repository](https://github.com/clawdbot/clawdbot)
- [Clawdbot Documentation](https://docs.clawd.bot/)
- [MacStories Review](https://www.macstories.net/stories/clawdbot-showed-me-what-the-future-of-personal-ai-assistants-looks-like/)
- [VelvetShark Review](https://velvetshark.com/clawdbot-the-self-hosted-ai-that-siri-should-have-been)
- ["Clawdbot Bought Me a Car" - AJ Stuyvenberg](https://aaronstuyvenberg.com/posts/clawd-bought-a-car)
- [Michael Tsai Blog](https://mjtsai.com/blog/2026/01/22/clawdbot/)
- [Medium: My Almost AGI with Clawdbot](https://medium.com/@henrymascot/my-almost-agi-with-clawdbot-cd612366898b)
- [addROM Setup Guide](https://addrom.com/clawdbot-your-personal-ai-assistant-for-windows-macos-and-linux/)
- [Vercel AI Gateway Docs](https://vercel.com/docs/ai-gateway/chat-platforms/clawd-bot)
- [Peter Steinberger](https://steipete.me/)
- [Clawdbot Shoutouts](https://clawd.bot/shoutouts)
- [Clawdbot Integrations](https://clawd.bot/integrations)
- [Skills Documentation](https://docs.clawd.bot/tools/skills)
- [Security Documentation](https://docs.clawd.bot/gateway/security)
- [iMessage Integration](https://docs.clawd.bot/channels/imessage)
- [Browser Automation](https://docs.clawd.bot/tools/browser)
- [Hacker News Discussion](https://news.ycombinator.com/item?id=46748880)
- [HN: "Why no love for Clawdbot?"](https://news.ycombinator.com/item?id=46662034)
- [Community Skills Collection](https://github.com/Dicklesworthstone/agent_flywheel_clawdbot_skills_and_integrations)
