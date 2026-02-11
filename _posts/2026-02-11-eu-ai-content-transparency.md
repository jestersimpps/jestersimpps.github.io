---
layout: post
title: "EU just made AI content marking mandatory"
description: "The EU's new Code of Practice requires machine-readable marking for all AI-generated content. Here's what it means for developers and why enforcement is the real challenge"
date: 2026-02-11
author: "Jester Simpps"
categories: [AI & Machine Learning]
tags: [AI Regulation, EU AI Act, Content Transparency, Machine Learning, Policy]
---

The EU dropped something big in December 2025

They published the first draft of their Code of Practice on AI content transparency and honestly i think this changes everything

## what is this

Part of the EU AI Act (Article 50) that says if you build AI systems that generate content, you need to mark that content in a machine-readable format

audio, images, video, text - basically everything AI can make these days

the marking needs to be:
- machine-readable (not just a visible watermark)
- detectable by systems automatically
- interoperable across platforms

## why this matters

We're drowning in AI content right now

deepfakes are getting scary good, AI text is everywhere, and honestly it's getting impossible to tell what's real

the EU is saying "ok we need to be able to identify this stuff systematically"

and the machine-readable part is key - we can't rely on humans to check every piece of content. we need automated systems that work at internet scale

## who needs to care

Two groups mainly:

**if you build AI models** - your outputs need proper marking built in

**if you use AI professionally** - you need to label deepfakes and AI text, especially for public interest stuff

## timeline

- now to jan 23: feedback period on draft 1
- mid-march: draft 2 expected
- june 2026: final version
- august 2, 2026: rules go live

so yeah, by august this isn't optional anymore

## technical stuff

They haven't specified exact standards yet but it needs to be:
- detectable by systems
- machine-readable (not just human labels)
- works across different platforms

probably means standardized metadata formats, digital watermarking, maybe blockchain-based provenance tracking

## bigger picture

This is the EU tackling the "provenance problem" at scale

think about what needs to happen:
- social platforms implementing detection systems
- AI providers building marking into their models
- CMS platforms tracking and displaying AI provenance
- journalists and fact-checkers getting better verification tools

pretty massive when you think about it

## what happens next

Commission is collecting feedback right now

this is when the technical details get hammered out - formats, standards, enforcement

if you're building with generative AI, now's the time to start thinking about implementation

## my take

I actually think this is smart

unlike regulations that try to restrict tech, this one focuses on transparency. it doesn't say you can't use AI - just be honest about when you do

the machine-readable aspect makes it practical. can't rely on humans for this at internet scale

but enforcement is the real question. how do you make global AI providers comply? what happens when bad actors strip the markings?

those challenges need addressing in the final version

overall though, step in the right direction. in a world where AI can generate anything, knowing what's real isn't just useful - it's necessary

## sources

- [Commission publishes first draft of Code of Practice on marking and labelling of AI-generated content](https://digital-strategy.ec.europa.eu/en/news/commission-publishes-first-draft-code-practice-marking-and-labelling-ai-generated-content)
- [First Draft Code of Practice on Transparency of AI-Generated Content](https://digital-strategy.ec.europa.eu/en/library/first-draft-code-practice-transparency-ai-generated-content)
- [The EU AI Act Newsletter #93: Transparency Code of Practice First Draft](https://artificialintelligenceact.substack.com/p/the-eu-ai-act-newsletter-93-transparency)
- [European Commission Publishes Draft Code of Practice on AI Labelling and Transparency](https://www.jonesday.com/en/insights/2026/01/european-commission-publishes-draft-code-of-practice-on-ai-labelling-and-transparency)
- [Transparency of AI-generated content: the EU's first draft Code of Practice](https://www.ashurst.com/en/insights/transparency-of-ai-generated-content-the-eu-first-draft-code-of-practice/)
