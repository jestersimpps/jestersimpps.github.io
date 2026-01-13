---
layout: post
title: "What is llms.txt and Why Your Website Might Need One"
description: "A practical guide to the llms.txt specification - how to make your website more accessible to AI models and why it matters for discoverability."
date: 2025-01-13
author: "Jester Simpps"
categories: [AI]
tags: [AI, LLM, SEO, Web Development, Standards]
image: /images/llms-txt-guide.png
---

so you've probably heard about `llms.txt` if you've been following the AI space lately. think of it like `robots.txt` but for LLMs

## The Problem It Solves

LLMs have this fundamental issue: context windows. they can only process so much text at once. when an AI tries to understand your website it runs into a few problems:

- **HTML is messy** - nav, ads, scripts, and actual content all mixed together
- **websites are huge** - most sites have hundreds of pages
- **structure is all over the place** - every site does things differently

converting a complex website into something an LLM can actually use is hard. that's where `llms.txt` comes in

## What is llms.txt?

Jeremy Howard (co-founder of Answer.AI) proposed this back in September 2024. it's basically a markdown file at your website root that gives:

- a quick overview of your site
- what each section is about
- links to the important stuff with descriptions
- context that helps AI understand what you're offering

it's a curated index built specifically for machines

## The Format

it's just plain markdown. here's the basic structure:

```markdown
# Site Name

> Brief tagline or description

## About
A paragraph explaining what this site is and who it's for.

## Site Structure
- Homepage: / - What visitors find here
- Documentation: /docs - Technical guides and API references
- Blog: /blog - Articles and updates

## Key Pages
- [Getting Started](/docs/getting-started) - First steps for new users
- [API Reference](/docs/api) - Complete API documentation

## Contact
- Email: hello@example.com
- GitHub: github.com/example
```

## Real-World Example

here's what i use for this site:

```markdown
# Jo Vinkenroye - Web Application Developer

> Building ERP systems, SaaS platforms, and modern web applications

## About
Senior developer with 13+ years of experience specializing in
React, Next.js, blockchain development, and AI integration.

## Site Structure
- Homepage: / - Overview of skills, experience, and projects
- Experience: /experience - Detailed work history
- Blog: /blog - Technical articles and project write-ups

## Blog Posts
- Building a Tamagotchi on Garmin: /blog/garmigotchi
- Ad-Forge - AI-Powered Ad Generation: /blog/ad-forge
```

## Should You Add One?

ok here's the honest truth: no major AI company has officially said they use `llms.txt` when crawling. it's a proposed standard, not an adopted one

but adoption is growing. Anthropic, Cloudflare, Vercel, Cursor - they've all implemented it. Mintlify rolled it out across all their hosted docs in late 2024

**add one if:**
- you have docs or technical content
- you want to be early on something potentially important
- you're building for AI-native discovery
- it takes 10 minutes and costs nothing

**skip it if:**
- your site is mostly visual content
- you're waiting for official adoption

## Implementation in Next.js

if you're on Next.js with the App Router you can create a dynamic route:

```typescript
// app/llms.txt/route.ts
export async function GET() {
  const content = `# Your Site Name

> Your tagline here

## About
Your description...

## Key Pages
- Homepage: / - Main landing page
- Blog: /blog - Articles and guides
`;

  return new Response(content, {
    headers: {
      'Content-Type': 'text/plain; charset=utf-8',
    },
  });
}
```

for dynamic content like blog posts you can generate it programmatically:

```typescript
// app/llms.txt/route.ts
import { getAllPosts } from '@/lib/blog';

export async function GET() {
  const posts = getAllPosts();

  const blogSection = posts
    .map(post => `- ${post.title}: /blog/${post.slug}`)
    .join('\n');

  const content = `# My Site

## Blog Posts
${blogSection}
`;

  return new Response(content, {
    headers: { 'Content-Type': 'text/plain; charset=utf-8' },
  });
}
```

## Tools and Resources

a few tools can help you set this up:

- **[llms_txt2ctx](https://github.com/AnswerDotAI/llms-txt)** - CLI for parsing and generating context
- **vitepress-plugin-llms** - VitePress integration
- **docusaurus-plugin-llms** - Docusaurus integration
- **GitBook** - auto-generates for all hosted docs

## The Bigger Picture

whether or not `llms.txt` becomes a universal standard, the problem it solves isn't going away. AI models will keep needing structured access to web content

by implementing it now you're:
1. making your content more accessible to current AI tools
2. prepping for potential future adoption
3. thinking about content from an AI-first perspective

that last one might be the most valuable. as AI becomes a primary way people discover content, machine readability becomes just as important as human readability

## Conclusion

`llms.txt` is simple and low-effort but could pay off as AI-native discovery grows. takes minutes to implement and signals your site is ready for the AI-first web

check out the [official spec](https://llmstxt.org/) for more details, or look at how [Anthropic](https://docs.anthropic.com/llms.txt) and [Vercel](https://vercel.com/llms.txt) have done theirs
