---
layout: post
title: "Building SmallShop Part 1: Laying the Foundation"
description: "progress report on building an ai-powered alternative to shopify for small shop owners who just want to sell stuff online"
date: 2026-01-14
author: "Jester Simpps"
categories: [AI & Machine Learning]
tags: [AI, E-commerce, Next.js, Side Project, Automation, Vercel]
image: /images/smallshop-cover.png
canonical_url: https://jovweb.dev/blog/building-smallshop-progress-report
---

So i was helping a friend set up her online boutique on shopify. should have been simple right? haha

Menus within menus. diy themes. configurations everywhere. plugins here and there. before you even launch your small online boutique you've already spent like a week just setting it up

And then there's the cost. it's not just the base fee. it's the monthly subscription plus commission on every purchase plus cost per plugin. for a small boutique owner who just wants to sell jewelry or vintage clothes it's way overkill

## The Idea

I figured in the age of ai i could build something simpler. something for small shop owners who don't need much. no headache, no hassle. just let me create your shop and pay a low monthly subscription

If i could convert enough small shops i'd have monthly recurring revenue. simple business model :)

## The Pricing Pivot

My initial idea: €200 for design and setup, then €20/month. i thought €200 was already pretty low for custom web design, but doable with tools like claude code and vercel these days

My friend laughed. she told me none of her friends who own businesses would pay €200 upfront. not even close

Ok back to the drawing board

## Going Fully Automated

So i thought, if people won't pay for manual setup, what if there was no manual setup at all? what if i could build the website automagically, and do the dns for private domains automatically using cloudflare apis

That's the route i took

## The Architecture

I analyzed the general layout of most shops. they basically all have:

- landing page
- all products page
- collection page
- product by id page

Then i looked at what sections are common. a landing page needs a hero, call to action, features list, product grid, collections grid. based on these blocks i could build most pages using ai in a structured way

### 18 Themes and Styles

Different shops need different vibes. jewelry store wants something luxurious. clothing store might want something more vintage or boutique-y. so i built 18 themes and styles users can pick to make their store feel unique

### The Configuration System

Based on these parameters each shop gets:

- different pages
- different layouts or sections per page
- different variants per section
- global theming and styling

This gives enough flexibility to create unique shops from the start. and users can afterwards easily change their style, move sections around, pick a different variant. whatever they want

## The Onboarding Flow

During onboarding the entire shop gets generated using ai. two routes:

**Existing website**: user enters their old website url. i scrape it and extract all the product info, images, layout, design, styles and pages. this goes through an ai pipeline that decides which sections to use, which design style fits, and which pages to create

**No existing website**: user goes through a stepper to fill out info, upload images, logos, and add their initial products

## The AI Pipeline

Once onboarding starts the pipeline:

1. uses extracted info to generate sections
2. chooses which variants of sections to use
3. decides on the global design style
4. imports all products and collections
5. sends products through google's nano banana for ai image generation to create hero banners and better product images

## Current Status

I've got the smallshop landing page done. the idea is that the entire store gets generated automatically so the user only needs to manage their products. that's it

But ai can't always get everything perfect. so i also built a theme editor as a fallback. users can move sections, change the design, update the ai generated copywriting or imagery if they want to. it's an afterthought, not the main experience

![theme editor](/images/smallshop-theme-editor.png)

Still working on:

- the full onboarding flow
- the ai pipeline (this is the tricky part)

The ai pipeline takes time to fine-tune because ai doesn't have deterministic output. every tweak requires running through the cycle again and testing. it's slow but necessary

## Lessons Learned

After [globalpetsitter](https://www.globalpetsitter.com) i learned that making a good product is not enough. you first have to get traction

So with smallshop i'm trying something different:

- **undercut major platforms on pricing** - make it a no-brainer
- **offer a free ai-generated preview** - let them see their shop before paying anything
- **hook them with the result** - if the ai-generated website impresses them they're more likely to convert

The goal is to remove all friction. no upfront payment, no manual setup. just "here's what your shop could look like" and then a simple monthly subscription if they want to keep it

Still early days but the foundation is there. more updates as i make progress :)
