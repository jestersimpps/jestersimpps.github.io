---
layout: post
title: "Garmigotchi: A Tamagotchi That Lives Off Your Health Data"
description: "i built a virtual pet watchface for garmin that reacts to your real health metrics. neglect your health and your creature dies"
date: 2025-01-13
author: "Jester Simpps"
categories: [Development]
tags: [Garmin, Wearables, Gamification, MonkeyC, Side Project]
image: /images/garmigotchi-cover.png
---

what if your smartwatch had a tiny creature living inside it. one that thrives when you're healthy and dies when you neglect yourself

that's garmigotchi. a tamagotchi-style virtual pet for garmin watches that feeds on your real health data

## the concept

your garmin already tracks everything. body battery, stress levels, sleep quality, steps, heart rate. but most people ignore these metrics until something goes wrong

garmigotchi turns that data into something you actually care about. a small creature that visibly reacts to how you're treating your body

high stress for too long? your pet gets anxious. didn't sleep well? it looks exhausted. hit your step goal with energy to spare? it's ecstatic

and if you consistently neglect your health? it dies. you'll have to wait until next month for a new one

## how it works

the creature lives in a small circular area on your instinct 3 watchface. every minute it checks your health metrics and updates its mood

### the mood system

i mapped garmin's metrics to a mood grid based on body battery and stress:

![Garmigotchi mood system](/images/garmigotchi-moods.png)

each mood has its own sprite animation and personality. an ecstatic creature bounces around energetically. a sleepy one barely moves

### neglect points

here's where it gets interesting. the creature tracks "neglect points" that accumulate when you're not taking care of yourself:

- body battery drops to critical? +1 point per hour
- terrible sleep score? +2 points
- high stress for 30+ minutes? +1 point
- didn't hit your step goal? +0.5 to +1 point

good days heal your creature. a solid night's sleep combined with hitting your step goal removes 2 neglect points

but let it accumulate to 16 points and your creature dies. becomes a ghost until the next month when a new egg hatches

### evolution

the creature evolves through 5 stages each month:

1. **Egg** (days 1-3)
2. **Blob** (days 3-14)
3. **Pup** (days 14-21)
4. **Mutt** (days 21-31)
5. **Elder** (day 31+)

keep it alive long enough and you'll see the full evolution. kill it early and you're stuck with a ghost

## the tech

building for garmin watches is... different. they use monkeyc, a proprietary language that compiles to run on extremely resource-constrained hardware

every sprite, every animation frame, every string. it all counts against tight memory limits. i ended up with 110+ pre-rendered sprites covering all mood and evolution combinations

the mood calculation runs every minute, pulling data from:
- body battery sensor
- stress levels
- heart rate
- step count vs goal
- previous night's sleep score

state persists locally on the watch. your creature's neglect points, evolution stage, and death status survive restarts

## why i built this

i've had a garmin for years. the health data is genuinely useful but i'd only check it reactively. after feeling tired or stressed

garmigotchi flips that. now i glance at my watch and immediately see how i'm doing. the little creature provides instant feedback without needing to dig through menus

it's also surprisingly motivating. seeing my pet look sick because i've been stressed all day actually makes me want to take a break. hitting my step goal feels more rewarding when my creature celebrates with me :)

## what's next

the watchface is built for all instinct watches. i'm working on adding more creature variants

it's currently in beta. if you want to keep a virtual pet alive with your health data, check it out at [garmigotchi.vercel.app](https://garmigotchi.vercel.app)
