---
title: "Local Lean Workgroup Retro"
date: 2026-02-01T14:04:19-08:00
draft: false
---

I started learning formalized mathematics with Lean last year [1](../lean1), [2](../lean2), [3](../lean3), [4](../lean4). As I progressed beyond basics, it dawned on me that I need to start interacting with the broader community as Lean is rapidly evolving and beyond basic getting started guides there aren't as many high-quality resources to learn solo. It could also be a great way to meet folks with similar interests and build more social connections. To that end I tried to start a local workgroup around Oct'25 [details](https://github.com/rkirov/analysis/blob/main/analysis/Analysis/Section_5_epilogue.lean). I have never done anything like this before, so this is a small retro of how it went so far (in Jan'26) and what's next.

## Vision

Without much experience of what is possible or advisable, my vision for this workgroup was for it to be something like "academia outside academia", i.e. take as many of the good parts of university classes, and adapt to the reality of doing this after work/school. So I set up a syllabus based on [Tao's Companion to Real Analysis](https://github.com/teorth/analysis) that I have been working through on my own.

A key ingredient is active learning, meaning everyone should be learning something from this class/workgroup and that happens best by writing Lean theorems and exercises.

I also wanted to emphasize in-person meetups at least weekly (along with ample online async communication) in order to build a sense of community.

## What worked well

- Interest - the most basic question was whether anybody would even be interested in this and show up. After close to 10-15 weekly meetings, I can say that there is generally a lot of interest and we always had at least 5-10 people show up. Some folks even commute from the further parts of the Bay Area to attend.
- Venue - I thought finding an appropriate place to host the meeting would be easy at first, thinking by analogy with pre-covid tech meetups I attended in various tech company offices. Actually, it turned out to be quite tricky initially, we didn't have many options that replied. Luckily, we got put in touch with [MOX](https://moxsf.com/) who were very enthusiastic about the meetup and have been hosting us since the first meeting. The location is very convenient, right next to a BART station in downtown SF.
- Personal accountability - as the year progressed my own motivation to work on Lean stuff has waned and the weekly meetup has acted as a good motivator to keep the basic connection going and not completely forget everything.
- Steady newcomer inflow - despite not doing much advertisement after the initial boost, we are still seeing newcomers every week. The Discord channel has 70 members, which suggests a wide pool of interested people in the area.

## What didn't work well

- Alignment with attendees' goals - since there is no general SF Lean meetup, we attract everyone: complete beginners wanting to try the Natural Number Game, Lean experts, software engineers interested in Lean as a programming language, AI researchers, and working mathematicians. Motivations vary just as much - some want active learning, some prefer the community aspect, some care more about foundations than formalization. This makes it hard to run a focused workgroup, but splitting into multiple meetups would leave each one too small.
- Lectures - I did a few intro lectures initially, and very quickly realized that I don't have the time to make them very polished and they go against my own vision for active learning.
- Retention - the Discord channel has 70 members but each session after the first had fewer than ten people show up. Most people attend once or twice before moving on. The constant influx of newcomers also means the group's effective level keeps resetting, making it hard to build momentum on more advanced material.
- Time spent outside the workgroup - it would have been more ideal if folks tried to work through exercises outside the meetup time and brought in questions they were stuck on. However, the majority (and sadly myself recently) only interact with Lean during the meetup.

## Where did we end up

Currently, we use the meetup as a shared Math in Lean coworking time. The majority of folks are newcomers who work through the Natural Number Game and first chapters of Tao's Real Analysis. At the beginning of the session, I create groups of folks working on a similar level of problems so they can discuss with each other. Experts help others, share advanced tips, tricks, and the occasional demo.

In terms of concrete progress: around 10 people finished the Natural Number Game through the workgroup, about 5 finished (or did major parts of) Chapter 2 of Tao's Real Analysis, and roughly 3 got through enough of Chapter 3 to feel ready to move on to Chapter 4. The funnel shape is expected, but it's encouraging that people are making real progress on non-trivial material.

## Going forward

I am going to continue to give this a try for another few months. For the reasons above, it is a bit of a challenge to reconcile my vision of active learning time for doing math in Lean with the attendees' schedules and vision. So I am officially dropping the syllabus and the Real Analysis focus (though not sure what other structured exercises I would recommend after NNG), but keeping the math focus (instead of general programming with Lean) and coworking aspect instead of giving talks.

I feel the natural pull is to turn this into an SF Lean meetup, where folks give weekly talks and demos. This workgroup shows there is enough interest and material, and it would solve a number of the challenges above. But I find that working together with people on the real experience of trying to learn and use Lean is more valuable to me than being in the passive audience of another talk. So for now, the coworking format stays.

## Advice for someone starting a similar group

- Active learning is even harder to pull off outside academia than inside it, but it's still worth trying. The alternative - passive talks - is easier to organize but produces less learning. Don't give up on the active format just because it's messy.
- Prepare for multiple async tracks. Give up on hard cohesion across the group, but still try to maximize the number of people working on the same problems at the same time. That's when the magic happens - when someone next to you is stuck on the same lemma and you figure it out together.
- Adjust as you go. Your initial vision will not survive contact with your actual attendees, and that's fine. The retro above is basically a story of continuous adjustment.

If you're in the Bay Area and interested, come join us - [details](https://discord.com/invite/bXNPhQyjfe) and [Discord](https://discord.com/invite/bXNPhQyjfe).

If you've had success running an active learning group outside of school - in any topic, not just math or Lean - I'd love to hear what worked for you. Reach out on Discord or leave a comment.
