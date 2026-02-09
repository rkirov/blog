---
title: "Leaning on AI"
date: 2026-02-08T21:16:11-08:00
draft: false
---

# Leaning on AI

It's been five months since my last dedicated [Lean post](../lean4) and as usual I have started to lose steam on Lean projects. After the thrill of discovering the world of formalized mathematics started to wear off, I did not find motivation to push as hard as before. The SF Math with Lean work group kept me vaguely connected (at least in the one hour a week we meet [(see retro)](../lean_workgroup)) but other than that I wasn't putting in more than a few hours a week on math and Lean.

So when I tried to get back into working through Tao's companion chapter 6, I had a rough awakening - while I remembered roughly my way around Lean tactics, I seem to have forgotten most of mathlib theorem names. Grinding through chapter 6 felt even harder and more chore-ish than before.

Luckily for me, the times have changed...

## Claude Proof

It seems that AI tools reached a breaking point, where more and more engineers are using Claude Code daily for their coding tasks. As a manager I don't code as much at work, so I was slow to switch from the autocompletion use of AI (GitHub Copilot, which I used for last year's Advent of Code, for example) to the new fully agentic coding with AI.

But basically, I was faced with a hard choice - at my current pace, it would take me at least a year to complete Tao's companion text, and I have to commit to memory a large number of mathlib theorems - was it `lt_le_of_le` or `lt_of_le_lt`. Or I can take a plunge into using Claude Code for Lean more aggressively.

I decided to do the latter and the results were amazing.

## The Setup

Inspired by moltbot (which I briefly experimented with) I wanted to have a claude code instance that is continuously working - I got a VM from digital ocean, setup Claude code and all lean tooling, enabled Lean MCP server and let it grind. I even have an iOS terminal emulator to check on progress and discuss issues when stuck.

## Results

Claude Code 4.6 is fully able to autonomously complete all examples and exercises in the real analysis companion [see section 6.4 for example](https://github.com/rkirov/analysis/blob/main/analysis/Analysis/Section_6_4.lean). I used two concurrent agents and upgraded to Claude Max (which was enough for them).

In one weekend I (we?) managed to complete sections 6.2, 6.3, 6.4, while I kept my usual weekend routine - bike ride, board games, watched the Super Bowl. I was maybe 2 hours max in front of the computer, to do code reviews and read the agents' summaries. Without AI, this would have easily been 10x the time effort on my part, and given I know Lean basics, the educational value for me would have been questionable.

I also manually checked places where the agents spotted odd formulations in Lean, which turned out to be typos when I cross-referenced with the actual textbook. As it is the case with everything these days, there is nothing that I believe can't be done by AI on fundamental grounds, but some things are still more ergonomic for me to do. For example, if my agent opens an upstream request, I need to verify it is not sending Tao some nonsense and wasting his time. The agents do the work, we provide the value system.

I had to learn how to improve my prompt [see CLAUDE.md](https://github.com/rkirov/analysis/blob/main/CLAUDE.md) and periodically ask the agents for key insights. I do read the PRs they produce and incorporate suggestions if I don't like the style or believe there is an easier way to do something.

## Is there a problem?

A nagging thought started to bother me. Using AI is undoubtedly more efficient, but am I defeating the point of the whole exercise? Why am I working through Tao's Real Analysis companion? I want to learn formalized mathematics with Lean, and by analogy with computer programming, the only way to learn a language is to write a lot of code in it. I don't do math research so the only math within my access is undergraduate and graduate textbooks. But if AI is writing the Lean code, what am I learning?

I broke it down further. Not all knowledge is equal, and AI help has very different costs for each kind:

**Basic language familiarity** - key concepts, tactics. This I already got by manually working through chapters 2 to 5. It is an open question what I would recommend to a newcomer; following what I did doesn't seem efficient. I probably should have started using AI earlier, but it was also not as capable back then. My current bar: if a tactic appears in my codebase and I can't explain what it does and roughly when to reach for it, that's a gap worth closing.

**Knowing mathlib theorems and structures** - this doesn't feel useful to learn as a human at all. I know `a < b` and `b <= c` implies `a < c`, committing the mathlib theorem name to memory is not useful. This is pure search, and outsourcing search to AI is pure upside.

**Learning advanced proof engineering** - (again by analogy with executable software) design patterns, good proof style, etc. This is indeed my next goal, but Tao's companion is not the right exercise for it. Someone already decided what the lemmas are, how they're organized, what depends on what. I'm getting zero reps on the hardest part of proof engineering — decomposing a theorem into pieces that are individually provable and collectively sufficient. That requires a project where I'm making the structural decisions myself.

**Engaging with the underlying mathematical ideas** - this is the most important, non-negotiable thing. If Claude figures out the proof strategy and I read it afterwards, I'm getting a fraction of the value compared to working through it myself. That said, a lot of the proofs in this intro real analysis textbook have no novel proof strategy - they are just expand the definitions and follow your nose, and only formalization with Lean makes them tedious.

## The workflow

To make it more concrete, I am experimenting with the following protocol per chapter:

1. Read the textbook chapter.
1. Read the corresponding .lean file and spot novel Lean concepts (at this point this is rare).
1. Look through the examples and see if there are novel math ideas (often none, there is a reason those are not exercises). Ask Claude to complete all examples.
1. Read through the exercises and sketch out a proof in Lean code or as a comment in Lean.
1. Ask AI to finish the proofs of the exercises.
1. Review Claude's summary (which I asked it to always produce after each session in CLAUDE.md).
1. Review the generated code. Unfamiliar tactic? Look it up. Unexpected proof strategy? Stop and understand why it worked.
1. If there is a repeated pattern, encode it as a rule in CLAUDE.md.
1. If there is something I don't understand, ask clarifying questions.

## The CLAUDE.md as style guide

Step 8 deserves more attention, because it turned out to be surprisingly educational. The CLAUDE.md file is where I encode things like which tactics to prefer, when to ask me for help versus push through, what "done" looks like. Writing these instructions forces me to articulate my own standards. I can't tell Claude Code "write clean proofs" — I have to say what clean means. Prefer `exact` over `simp` when the term is short? Don't use `omega` to close goals that have mathematical content? Every rule I write is a proof engineering opinion I had to develop and make explicit.

And it's iterative. Claude does something I don't like, and instead of just fixing it, I ask whether this is a one-off or a pattern. If it's a pattern, it becomes a rule. Over time the CLAUDE.md becomes a document of my evolving Lean taste — a style guide for formalization that I authored through friction with the tool.

The irony is that this is one of the places where the AI workflow teaches me *more* about proof engineering than working alone would. Solo, my standards stay implicit. Having to communicate them to an agent forces clarity. And the resulting proofs end up more distinctly *mine* — shaped by my preferences, my aesthetic, my opinions about what good formalization looks like — than they would be if I were just following whatever path the tactics led me down.

## Looking forward

This all feels novel and foreign to the old world of actually writing the code yourself. But five months of losing steam taught me that the perfect workflow I never use is worse than the imperfect one that keeps me going. And maybe there's another thing I'm learning besides Lean — how to work with AI itself. That might end up being the most useful skill I take away from this whole project.