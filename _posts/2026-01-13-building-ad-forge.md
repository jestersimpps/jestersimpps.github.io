---
layout: post
title: "i built a tool to generate video ads with ai"
description: "ad forge collapses the entire video ad creation workflow into one ai-powered pipeline. describe your concept, get a finished video"
date: 2026-01-13
author: "Jester Simpps"
categories: [AI]
tags: [AI, Video Generation, Next.js, Fal.ai, Automation]
image: /images/ad-forge-cover.jpg
canonical_url: https://jovweb.dev/blog/building-ad-forge
---

so i run globalpetsitter.com. connecting pet owners with sitters around the world. like any startup i need promo content, specifically video ads for social media. the problem is making even a simple 30 second ad takes forever

## the problem

every time i wanted to make a video ad the process was something like:

1. write a script or storyboard
2. find or create images for each scene
3. generate or record voiceover
4. convert images to video clips with motion
5. sync audio with video
6. edit everything together in capcut

each step needs different tools, different logins, tons of context switching. for a 6 scene ad we're talking hours. and if i didn't like the result? start over

## ad forge

i built ad forge to collapse all of this into one pipeline. describe your ad in plain text, let ai do the heavy lifting

here's what the output looks like:

<video src="/images/ad-forge-demo.mp4" controls playsinline style="width: 100%; border-radius: 12px; margin: 2rem 0;" poster="/images/ad-forge-cover.jpg"></video>

### how it works

7 stages:

**1. sketch** - describe your ad concept, target audience, tone, duration. for the globalpetsitter ad i wrote something like "woman leaving for travel, worried about her pet, then feeling relieved knowing they're cared for"

**2. scenes** - gemini breaks your sketch into individual scenes with descriptions, settings, mood, suggested durations. it structures the narrative arc automatically

**3. style** - ai generates a style guide. color palette, lighting, visual mood, character descriptions, location details. keeps everything visually consistent

**4. images** - fal.ai generates an image for each scene. the system uses reference images from previous scenes and character portraits to maintain consistency. this was the hardest part to get right

**5. videos** - each image becomes a video clip with camera movement (pan, zoom, dolly, etc). fal.ai's image-to-video is pretty good at this

**6. audio** - for scenes with dialogue it generates voiceover with tts. you can assign different voices to different characters

**7. merge** - combines video and audio, optional lip-sync for talking characters. ffmpeg handles this

### tech stack

- next.js 16 with react 19 for the ui
- google gemini for script generation and scene breakdown
- fal.ai for image generation and image-to-video
- openai for some text generation
- ffmpeg webassembly for video processing in browser

### some design decisions

**campaign persistence** - everything saves to localstorage automatically. you can close the browser and pick up later

**reference images** - this was crucial. when generating scene 3 you can reference the location image, character portraits, previous scenes. the ai uses these as style anchors

**stage based workflow** - each stage produces output you can review. don't like the scenes? regenerate before moving on. gives you control without overwhelming options

## results

the globalpetsitter ad that would've taken me a full day now takes about 30 minutes of active work (plus generation time). more importantly i can iterate fast. try different tones, swap scenes, regenerate individual images without starting over

## what's next

ad forge is still rough. i want to add:

- background music selection
- more camera movement options
- direct export to social formats (9:16 for tiktok/reels, 16:9 for youtube)
- templates for common ad formats

for now it's solving my problem: making video ads for globalpetsitter without the time sink. sometimes that's enough :)
