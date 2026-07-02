---
title: "The Jacobian Challenge Retro"
date: 2026-06-22T15:14:33-07:00
draft: false
---

In April 2026, Kevin Buzzard posed the following challenge for Lean AI autoformalization: can an AI
system both define the [Jacobian variety](https://en.wikipedia.org/wiki/Jacobian_variety) in Lean and prove basic properties about it? ([zulip thread](https://leanprover.zulipchat.com/#narrow/channel/583336-Autoformalization/topic/Jacobian.20challenge/near/586351299), [github gist](https://gist.github.com/kbuzzard/778bc714030b3e974ab5f4038783d1a9)). The motivation behind this
challenge, as far as I can tell, was to point the autoformalization community at a specific problem chosen by Kevin, instead
of letting people choose freely from the vast number of known missing formalizations. Because few are incentivized to publish
their failed attempts, there were doubts about whether recent successes reflected general capability or just clever problem selection.

The Jacobian challenge was definitions-focused rather than theorem-focused. On the surface it just asks for the definition of a classically known object — its namesake lived in the 1800s — but implicitly it requires building a lot of non-trivial differential geometry that mathlib is missing, down to the definition of the genus of a Riemann surface. Kevin also attached a number of "universal" properties that the Jacobian should satisfy (universal in quotes, because I don't think they're rigorously universal in the category theory sense). It was also a test of whether the theorems listed are good enough to pin the formalization down, possibly preventing AI from submitting mathematically wrong definitions.

After about 22 working days of OSS collaboration using Claude Code, spread between April 19 and June 13, I produced a solution that was verified by [lean-eval](https://lean-lang.org/eval/problems/jacobian_challenge_diffgeo/) on June 11 (the last two days were final cleanup). Subsequent discussion on zulip gives some signal from the experts that it is correct — i.e. that misformalization hasn't happened — but since I don't know the math, I can't fully review it and vouch for that myself. In any case, this is a small retro, an experience report on that autoformalization.

To read more about the math see the [Verso exposition](https://rkirov.github.io/jacobian-claude/) (per-unit pages linked from [`units.html`](https://rkirov.github.io/jacobian-claude/units.html)). The [`formalization.yaml`](https://github.com/rkirov/jacobian-claude/blob/main/formalization.yaml) records the [mathlib-initiative](https://github.com/mathlib-initiative/formalization.yaml) self-reporting metadata for what was formalized.

## The facts

- the final solution is ~45k loc (`cloc`, excluding comments), after cleanup.
- I used Claude Code exclusively, always on the newest model available, on the $100/month subscription plan. Per the commit trailers, that meant Opus 4.7 at the start (late April), Opus 4.8 for most of the project, and a brief stint on Fable 5 during the window it was available. Fable 5 made good progress, though it's hard to say it wasn't just working through an easier stretch of the proof at that point.
- I saturated the subscription's weekly usage limit for four weeks running, so a rough estimate for the AI cost would be ~$100.
- I spent a lot of time monitoring the models and giving them project-management advice. That made it a poor experiment for *autoformalization*, but an excellent one for observing how AI fails on a project of this size. Either way, I don't know differential geometry, so I certainly couldn't have helped with the math itself. I asked the agent to summarize my inputs in a [human steering log](https://github.com/rkirov/jacobian-claude/blob/main/human_input.md).
- the agents looked at and borrowed parts from other OSS attempts at this challenge: degree/fibre well-definedness infrastructure ported from [Brsanch/jacobian-lean-challenge](https://github.com/Brsanch/jacobian-lean-challenge), the rectangle Green's theorem for the planar Stokes atom from [tangentstorm/JacobianChallenge](https://github.com/tangentstorm/JacobianChallenge), and a contour-deformation development (later superseded) plus an axiom inventory from [mrdouglasny/jacobian-challenge](https://github.com/mrdouglasny/jacobian-challenge).
- probably there is no path to upstreaming this into mathlib — partly the lack of generality (the differential geometry machinery was built for curves only), partly mathlib's policy on AI usage.

## What worked well

- I showed that off-the-shelf AI models and harnesses (Claude Code) are very capable at Lean formalization, even for definition-heavy tasks, given enough time and some basic project management help. I am confident that I overinfluenced the output, out of my own curiosity and desire to try things and poke around with the work. That said, I also feel that zero human involvement would have made the project slower, while hard to pinpoint exactly by how much.
- I learned a lot about Claude Code's newer features — auto-approval, /goal, /remote-control, subagents, and so on — but that space is evolving so fast it isn't worth writing up here; it'd be outdated in a few weeks.
- the key to successful autoformalization of a known result is reminding AI to follow the existing informal proof. It does try to be overly creative, in order to find possibly shorter, cleverer proofs, but that's how it got to dead ends that Lean rejected or ended up with useless proofs (ones with unsatisfiable hypotheses). My most common comment was "did you follow the literature".
- AI did not get stuck on details (it did work through some instance diamonds), nor did it try any crazy cheating exploit to close a goal. The only thing I caught it doing: early on, tasked with "remove the sorries", it turned them into uninstantiated type-classes — which of course solves nothing. When I pointed it out, it saved "avoid progress-theater" to memory and didn't repeat it.
- some slight parallelization was possible (2-3 subagents), which was the limit of my subscription token cost and VM memory usage.
- I spent almost no time reading the code, but it's of average quality as far as I can tell. I've only been writing Lean for a year, so I write only intermediate level Lean myself. At a glance I can't say the AI's code is worse than mine.


## What didn't work so well

- the final proof had a lot of garbage in it. I was focused on getting the thing working end-to-end, so I didn't ask it to clean up as it went. Once the proof passed lean-eval, I had it do a cleanup pass and we removed close to 20% of the code as dead code. There were a lot of warnings to work through too, but it did all cleanups in 2 days.
- while line-level Lean is fine, the AI is weak at the structural side: managing variables and namespaces, deciding where `set_option`s and comments belong, and small placement nits like putting a theorem before its first use site. It's also reluctant to do large refactors, tending to build a parallel structure alongside the old one rather than rework it in place — though I can't fully tell how much of that is an AI limitation versus just how painful Lean refactoring is.
- build times slowly got notably worse. The final solution takes 25 min on CI. I attempted to have AI improve build times for some time after the solution, with little initial success (but didn't give it a lot of time).
- the biggest source of wasted effort was the existence of multiple, vastly different proof paths for standard theorems in differential geometry. Again, I don't know this math, but from reading the AI's thought trace, a key result is proved in Forster and in Miranda (the two standard Riemann surfaces textbooks) using very different machinery. The agent would go down one path, hit a snag, pivot to the other, and leave a lot of half-finished machinery behind. It got worse with subagents: each starts with fresh context, so they flip-flopped a few times before settling on the final path.
- AI really starts to struggle to keep a consistent view of the project around the 10k loc mark, and it gets progressively worse toward 50k. Even with the 1M context window, things fill up, and I'd watch the agent grep around just to rediscover what was already in the project. We tried to help with .md plans and comments, but those added context pollution of their own and then drifted out of sync with the actual code — so I'm honestly unsure whether to recommend more of that or less. Since this is a general large-software problem, nothing specific to math or formalization, I'm optimistic it'll improve soon.
- I used the [lean-lsp MCP](https://github.com/oOo0oOo/lean-lsp-mcp) and the [lean4 skills](https://github.com/cameronfreer/lean4-skills), but can't tell whether they helped much or at all. Agents were haphazardly using MCP and sometimes switching to just shelling out to `lake build`.

### Which path to pick through the theory

The core challenge in formalizing a classical subject — auto or not — is which path to pick through it. The churn above is the cost of getting that choice wrong, but the choice is also where the interesting work lives. Coincidentally I'm also currently autoformalizing [Axler's Linear Algebra](https://github.com/rkirov/linear-algebra-done-right-lean/tree/main/LinearAlgebraDoneRightLean), and I'm excited about how one might innovate in teaching hundred-year-old subjects by finding new — more pedagogical, or more aesthetically pleasing — proof paths through what is nominally the same classical material. Without AI and formalization these explorations were too laborious and unglamorous (AI lowers the cost of experimentation), and too easy to get wrong (Lean guarantees you never lean on a true-but-unproven claim). While I picked the most standard path and told the agent to not risk and explore non-standard, it could be and interesting outcome of formalization that shorter (or better in some way) paths through known theories are found.

## Which direction to build

Generally, formalizing a large project (auto or not) goes: 1) read the literature and make a plan (a *blueprint*, in Lean lingo); 2) follow the plan to write the Lean code. Inevitably the informal plan misses something, and step 2 forces iteration on step 1. Nothing new here — it's the same tension present in all software projects, well documented in [*The Mythical Man-Month*](https://en.wikipedia.org/wiki/The_Mythical_Man-Month).

Within step 2, you can build in one of two directions:

1. **leaf-to-root** — upstream-to-downstream, forward linking (lots of names for the same thing): start from the foundations and build up toward the target theorem.
2. **root-to-leaf** — downstream-to-upstream, back linking: start from the target and work down toward the foundations.

Each direction has its own failure mode:

1. you might prove theorems that, while correct, aren't needed — either trivially true, or mathematically meaningful but not on the path to the goal.
2. you might prove theorems from hypotheses that turn out to be unprovable.

In my experience the second failure mode is the more costly one, so I recommend building leaf-to-root. Better to have mathematically correct proofs that just aren't needed than proofs that follow from a false premise. It also parallelizes better. The catch is that it requires enough planning up front to actually see where the leaves are, while 2) can work "just-in-time".

## Clean room rebuilds

Somewhat inspired by Gowers's discussion of "Kolmogorov complexity modulo experts" in the remarks on the AI solution to the [unit distance problem](https://cdn.openai.com/pdf/74c24085-19b0-4534-9c90-465b8e29ad73/unit-distance-remarks.pdf), I asked the agent to post-factum produce a short blueprint file. The goal is to map the now-known path to the solution, but removing all Lean details — just the math, references, and maybe some gotchas. This file is [clean_room_blueprint.md](https://github.com/rkirov/jacobian-claude/blob/main/docs/clean_room_blueprint.md). If interested, give it to your agent, instruct it not to use the internet, and see how long it takes to solve the challenge — now hopefully fully autonomously. In a sense this inverts Gowers's measure: he asks how short a hint sequence lets a human expert reconstruct a proof; here the hint is fixed and we measure how long an AI takes to rebuild the solution from it. I plan to test it with Fable 5 soon.

## Summary

Off-the-shelf AI tools like Claude Code are very capable at autoformalizing mathematics with no human mathematical guidance, but with significant project management. As a very rough estimate: about three weeks of a Claude Max subscription on current Opus 4.8 (1M context) buys roughly one [GTM](https://en.wikipedia.org/wiki/Graduate_Texts_in_Mathematics) textbook's worth of formal Lean content, or ~30k loc of Lean. My estimation is that the Jacobian challenge was about 1.5 GTM textbooks and ~45k loc in the end. If you're interested in math formalization but blocked by missing foundational definitions or theorems, I'd recommend looking into autoformalization. A lot of math missing from mathlib is in reach for AI tools with a bit of patience. If hobbyists like me on a personal subscription can quickly formalize whole textbooks, much more ambitious projects will be in scope very soon for  autoformalization with bigger resources - say Poicare's conjecture, FLT or classification of finite simple groups.