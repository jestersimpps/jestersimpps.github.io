---
layout: post
title: "Building vbcdr: an AIDE for Developers Who Vibe"
description: "Why I built a desktop development environment where terminals and browsers come first, and the code editor is intentionally secondary."
date: 2026-02-12
author: "Jester Simpps"
categories: [Development]
tags: [Electron, TypeScript, AI, Developer Tools, React, Open Source]
image: /images/vbcdr-cover.png
---

I got tired of fighting my IDE

Every day it's the same thing. open VS Code. open a terminal. open another terminal. open the browser. arrange windows. lose track of which terminal is running what. tab back and forth a hundred times between the code, the AI, and the preview

So i just built my own thing

![vbcdr editor view](/images/vbcdr-cover.png)

## The IDE is dead, long live the AIDE

AIDE stands for AI-Integrated Development Environment. traditional IDEs were built for a world where humans write every line of code. but that's not how i work anymore

My workflow now: open a terminal, tell Claude what i need, review what it writes, check the result in the browser. the code editor? i glance at it sometimes. maybe to understand a type or trace a bug

So vbcdr flips the traditional layout. terminals and browser previews **take the main stage**. Monaco editor is still there when you need it, but it's intentionally secondary

## Terminal-first everything

The terminal panel is the heart of it. xterm.js with WebGL rendering so it's buttery smooth. each project gets multiple terminal tabs - one for your AI agent, others for dev servers, database stuff, whatever you need

The AI terminal auto-creates when you switch projects. because in a vibe coding workflow the agent is **always running**

Built it with node-pty for native shell access. your actual shell, your actual environment, your actual PATH. not some sandboxed fake terminal

Terminal search with highlights, clear/restart buttons, and scroll-to-bottom. Shift+Enter inserts newlines in the LLM terminal input without submitting, which sounds tiny but makes a huge difference when you're writing multi-line prompts

You can drag files straight into the terminal for quick context. drop an image and it auto-attaches to the LLM via clipboard. no more copy-pasting file paths

```
┌──────────────────────────┬──────────────┐
│                          │   Git Graph  │
│  Browser or Editor       │              │
│                          ├──────────────┤
│                          │   LLM        │
│  Dev 1  │  Dev 2         │   Terminal   │
└──────────────────────────┴──────────────┘
```

## Built-in browser with project isolation

Every project gets its own integrated browser with per-project storage isolation. no more cookies leaking between projects. each one runs in its own Electron webview partition

i hooked into Chrome DevTools Protocol to get network monitoring and console capture without building any custom instrumentation. the browser panel has three devtools tabs: Console, Network, and Passwords

The network inspector shows expandable request details with headers, type, and accurate response sizes. console errors and network failures have a **one-click "send to LLM"** button that forwards them straight to your active AI terminal. see a 500 error? click once and your agent is already debugging it

The password manager detects login forms automatically. injects a script that watches for password fields using MutationObserver, catches form submissions, and offers to save credentials encrypted with Electron's native safeStorage API

Next time you visit that page it auto-fills

Device emulation switches between desktop, iPad, and mobile viewports with proper user agents. Google OAuth gets redirected to your system browser because Electron webviews and OAuth just don't mix

## 11 themes because why not

Ok so i might have gone overboard here. Dracula, Catppuccin, Nord, Tokyo Night, Gruvbox, One Dark, Solarized, and more. each with light and dark variants

I spend a lot of time staring at this thing. it should look good

## Multi-project state that actually works

This was the architectural decision i'm most proud of. every Zustand store uses a `Record<projectId, StateType>` pattern. switching projects instantly shows that project's terminals, browser tabs, file tree, editor state, and git history

**Nothing shared. Nothing leaks.** click a project tab and your entire context switches

Browser tabs persist to disk with a 500ms debounce so you don't thrash the filesystem. terminal processes stay alive in the background. switch back and everything is exactly where you left it

## Git graph that talks to your AI

The git panel renders a lane-based commit graph using SVG. each branch gets its own lane with a color. merge commits get larger nodes. bezier curves connect parent-child relationships across lanes

Hit "Commit" and it sends `/commit` to your active AI terminal. hit "New Feature", type a description, and it creates a properly formatted `feature/your-description-in-kebab-case` branch via your agent

The git tree isn't just visualization. it's a command interface that speaks through your AI agent

## Electron, Zustand, and a clean IPC layer

Electron 34 for cross-platform desktop. React 18 + Tailwind CSS 4 for the UI. electron-vite for builds. Zustand for state because Redux is overkill when your stores are this clean

The IPC layer is the backbone. six handler modules (projects, terminal, filesystem, git, browser, passwords) each manage their own domain. services like the PTY manager, file watcher, and git service do the actual work. clean separation between what the user sees and what the OS does

File watching uses chokidar with gitignore-aware filtering. file tree caches and debounces updates. Monaco gets live file sync so external changes from your AI agent writing files appear in real-time

## Steering beats typing

Building developer tools is addictive and humbling. you think you know what you want until you use it for a week and realize the layout needs to change, the terminal needs search, the browser needs password management, and the git graph needs to be interactive

Biggest insight: in an AI-first workflow the dev environment should be optimized for **steering**, not typing. large terminal real estate. easy browser access. git context at a glance. the code editor is just a reference viewer now

vbcdr isn't trying to replace VS Code. it's built for a different workflow entirely, one where you spend more time directing an AI than editing files yourself

That's what an AIDE is. not an IDE with AI bolted on. a fundamentally different tool for a fundamentally different way of building software

## What's next

Still on the roadmap:
- **Visual skills manager** - browse, enable/disable, and configure LLM agent skills and slash commands from a UI panel instead of managing config files
- **Password & browser favorites overhaul** - the current system needs a redesign for better UX and reliability
- Click files and folders in the project tree to send to your agent for easy context input
- Give the AI access to the webview browser so it can manipulate and test the preview
- Windows and Linux builds
- Many more ideas

Open source, MIT licensed, and very much a work in progress. first release is out for macOS Apple Silicon. feel free to contribute on [GitHub](https://github.com/jestersimpps/vbcdr-electron)
