---
layout: post
title: "Lessons from Building Global Pet Sitter"
description: "what i learned co-founding a pet sitting platform. the good, the bad, and everything in between"
date: 2026-01-13
author: "Jo Vinkenroye"
categories: [Career]
tags: [Startup, Entrepreneurship, Side Project, Lessons Learned, Convex, Next.js]
image: /images/global-pet-sitter-hero.jpeg
---

i co-founded [Global Pet Sitter](https://www.globalpetsitter.com) with my friend Geert. two developers, over 10 years of housesitting experience between us, trying to build something transparent and community-driven. it started as a simple idea and honestly turned into one of the most educational experiences of my life

## why we built it

we'd been housesitters ourselves for years. used platforms like TrustedHousesitters and Nomador. and honestly, we got frustrated

the problems kept piling up:

- **fees getting out of control** - TrustedHousesitters introduced a $12 booking fee per sit on top of annual memberships. felt like nickel-and-diming the community
- **customer support that doesn't respond** - emails go unanswered, phone support removed, just chatbots. when something goes wrong during a sit, you're on your own
- **review systems that feel broken** - false reviews can't be removed even with proof. no accountability when hosts or sitters behave badly
- **no real vetting** - background checks are voluntary and paid by the sitter. platform takes minimal action even when theft or pet neglect is reported

we kept reading the same complaints over and over on Trustpilot and Sitejabber. TrustedHousesitters has a 2.7/5 rating on Sitejabber with over 1000 reviews. people describing theft, property damage, last-minute cancellations, and a platform that "investigated but refused to take action"

something that started as community-focused became over-commercialised. and we thought, maybe we could do better

the mission was clear: build something transparent where pet owners and sitters could connect without all the friction. sounds simple right? haha

## the tech stack

we went with Next.js 16 (App Router + Turbopack), Convex for the real-time backend, and Clerk for auth. Tailwind for styling because life's too short to write CSS from scratch

why Convex? real-time updates out of the box. when a sitter applies or a message comes in, you see it instantly. no websocket setup, no polling, no headaches. the schema is defined in TypeScript, queries are type-safe, and deployments just work

```typescript
// this is how clean Convex queries are
export const getApplicationsByAssignment = query({
  args: { assignmentId: v.id("sitAssignments") },
  handler: async (ctx, args) => {
    const applications = await ctx.db
      .query("applications")
      .withIndex("by_sit_assignment", (q) =>
        q.eq("sitAssignmentId", args.assignmentId)
      )
      .filter((q) => q.eq(q.field("deletedAt"), undefined))
      .collect();
    return applications;
  },
});
```

## trust is everything

when you're asking someone to take care of their pet, you're asking for a lot of trust. pets are family. people don't just hand over their dog to a stranger on the internet

we spent way more time on verification, reviews, and safety features than we initially planned. privacy controls became a big thing. and honestly we should have started there. building trust isn't a feature you add later, it's the foundation

### blind reviews

one thing we built that i'm pretty happy with: blind reviews. both parties submit their review independently, and neither can see the other's review until both have submitted (or 14 days pass). prevents retaliation reviews and keeps things honest

```typescript
const isReviewVisible = (review: { visibleAt?: number }): boolean => {
  if (!review.visibleAt) return false;
  return review.visibleAt <= Date.now();
};
```

simple logic but it changes the whole dynamic

### importing reviews from other platforms

one problem new platforms face: experienced sitters already have reviews on other sites. they don't want to start from zero. so we built a review import system

the flow works like this:

1. user links their profile on another platform (TrustedHousesitters, Nomador, etc.)
2. we verify they own the profile by having them add a unique code to their bio
3. once verified, they can upload screenshots of their reviews
4. GPT-4 Vision extracts the review data automatically

```typescript
export const extractReviewFromScreenshot = action({
  args: { imageUrl: v.string() },
  handler: async (_, args) => {
    const response = await openai.chat.completions.create({
      model: VISION_MODEL,
      messages: [{
        role: "user",
        content: [
          { type: "text", text: "Extract the review information:" },
          { type: "image_url", image_url: { url: args.imageUrl } },
        ],
      }],
      response_format: { type: "json_object" },
    });
    // returns: reviewerName, reviewText, overallRating, reviewDate
  },
});
```

the verification part was interesting. we generate a unique code like `gps-verify-abc123`, user adds it to their profile bio, and we fetch the page to check if the code exists

```typescript
const codeFound = html.includes(profile.verificationCode);
if (codeFound) {
  await updateProfileStatus({ status: "verified" });
}
```

simple but effective. prevents people from claiming reviews that aren't theirs

## two-sided marketplaces are hard

you need pet owners AND pet sitters. but pet owners won't join without sitters, and sitters won't join without pet owners. classic chicken and egg

we ended up focusing on building features that actually help both sides - map search so you can find sits near you, dual accounts so you can be both a sitter and an owner. the little things that make the experience smoother

### the map search problem

map search sounds easy until you actually build it. we use LocationIQ for geocoding (way cheaper than Google) and had to build a backfill system for existing users

```typescript
export const backfillSitterCoordinates = action({
  args: {
    batchSize: v.optional(v.number()),
    delayMs: v.optional(v.number()), // rate limit protection
  },
  handler: async (ctx, args) => {
    const sitters = await ctx.runQuery(
      internal.geocoding.getSittersNeedingGeocoding,
      { limit: batchSize }
    );
    // process in batches to avoid rate limits
    for (const sitter of sitters) {
      // geocode and update...
      await new Promise(resolve => setTimeout(resolve, delayMs));
    }
  },
});
```

we also use supercluster for clustering markers when you zoom out. otherwise the map becomes unusable with too many pins

## listening changes everything

one of the biggest lessons: listen to your users. we recently changed how we talk about the platform entirely based on feedback. the way we described Global Pet Sitter wasn't resonating with people, so we rewrote it

every complaint is a feature request in disguise. every confused user is showing you where your product falls short. pretty cool when you start seeing it that way

## soft deletes everywhere

early on we learned: never actually delete data. everything uses soft deletes with a `deletedAt` field. when something goes wrong (and it will), you can recover. when users ask "what happened to my listing?", you can actually tell them

```typescript
.filter((q) => q.eq(q.field("deletedAt"), undefined))
```

you see this pattern everywhere in our codebase. slightly annoying to remember but saves you so many headaches

## what we've shipped

looking back at what we built:

- **map search** - find sits in your area visually with marker clustering
- **dual accounts** - be a sitter and an owner with the same profile
- **privacy controls** - you decide what info to share
- **real-time messaging** - no refresh needed, messages just appear
- **blind reviews** - honest feedback without fear of retaliation
- **review importing** - bring your reputation from other platforms using AI extraction
- **early adopter rewards** - because the first users took a chance on us

and we're working on an iPhone app. it's coming :)

## the tech wasn't the hard part

i spent so much time worrying about the tech stack, the architecture, the performance. turns out the hard part was getting people to care about your product

the best code in the world doesn't matter if nobody uses it. i wish i learned this earlier

that said, picking the right tools (Convex, Clerk, Next.js) made the dev experience smooth enough that we could focus on the product instead of fighting infrastructure

## would i do it again

absolutely. even with all the late nights and moments of doubt. building something from scratch with someone you trust teaches you things you can't learn any other way

maybe that's the real lesson. the product might succeed or fail, but you always come out knowing more than when you started
