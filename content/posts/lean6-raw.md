---
title: "Update on Real Analysis with Lean and Claude (Raw Pre-AI polish)"
date: 2026-03-08T23:35:37-07:00
draft: true
---


## Update on Real Analysis with Lean and Claude

*Disclaimer* - The core topic and key points were fully written by me (the human behind this), but Claude helped me flesh the article and polish it. As an experiment I am leaving the initial pre-AI edits draft [here](../lean6-raw). I both emphatise with readers tricked into reading low-signal but plausably looking one-shotted articles, but also benefit from the efficiency provided by Claude fleshing out my blog outlines quickly and effortlessly. So hope the raw outline dump convinces readers this is not a low-quality one-shot post. Feedback welcome, we are all trying to figure out how this brave new world should work!

Follow up to (leaning on claude article lean 5 insert link) upto chapter 8.3 in Tao's companion, speed is great, claude's capabilities are great (autoformalization is possible, but not my goal), stuck on nothing. Upstream many typos (put github link).

There have been many great recent essays on math, AI and formalization:

- https://arxiv.org/abs/2603.03684
- https://leodemoura.github.io/blog/2026/02/18/proof-assistants-in-the-age-of-ai.html
- https://www.theatlantic.com/technology/2026/02/ai-math-terrance-tao/686107/

but what I want to offer is a specific case study on working through two classic Real analysis problems form Chapter 8.2 with Claude and Lean.

## Case Study - Riemann's Divergence theorm

This has to do with the classical result by Riemann https://en.wikipedia.org/wiki/Riemann_series_theorem. 

## problem 8.2.5

The book outlines a sketch of a proof and the exercise is to complete the proof.

[include riemann-text.png]

The companion translates that to Lean as follows https://github.com/teorth/analysis/blob/main/analysis/Analysis/Section_8_2.lean#L457-L493 and again asks to complete it by removing the sorries.

Here is how I approached this:

phase 1 (rushed) - I work on these exercises after my daughter goes to bed 9pm to 11-12pm, so during the workweek I am often a bit more tired and less motivated. I figured this problem was hard, so I didn't read at all into the book scaffold, but still asked claude to grind it out (as I had it sitting idle). After a day or so (no idea how much clocktime it was), it spit out with zero intervention, that I checked in.

phase 2 (correction) - During the weekend, I came back to this as my goal is not to just get the lean solutions but actually understand the content. I read the book carefully and came up with a sketch of the proof for 9 sorries out of 10. The idea was something like (sadly I lost the log)

"1 and 2 - by previous exercise both a_plus and a_minus are not abs convegent. So they have to be infinite.
3 and 4 - if a_plus or a_minus is finite, the tail of one of them matches a, but they are not convergent while a is, which is a contradiction.
5 - this is obvious by construction of n', we never pick the same element twice.
6 and 7 - if the partical sums at some finite point stop crossing over L, that means the tail matches the tail of a_plus or a_minus. But those diverge, while there is a constant gap to L, a contradiction.
8 - direct consequence of 3 and 4. Since we pick infinitely many times from a_minus and a_plus by construction we have to pick all of them as the construction doesn't skip.
9 - We have shown n' is a bijection and we know `a` as series converges, so the sequence must go to zero.
10 - I have no idea actually."

Notably, this is not up to the standard of a modern mathematical proof, nonetheless it is a quick dump of my intuition and now I have Claude and Lean to work through the details.

phase 3 (collaboration) - I read the lean proof and tell Claude to align it with my human idea above where diverged. For Parts that are pure lean technicallities, I trust claude (look for something odd, I heard that claude might inject some tricky work around when fully stuck and left unattented for long). No such injections seens, no new tactics seen, plausable theorems from mathlib. I didn't dwell on minute style tactic points (theorem is too long to worry about 'simp' vs 'omega' etc). Claude restructures the proof somewhat.

pahse 4 (learn) - part 10 of the proof, I am stuck, I ask claude to explain what is happening in words, convince myself on paper conceptually, I read the proof after and agree it Makes sense. Wow this is tricky indeed, I ask to refactor a bit, and insert with better comments matching my understanding that I retell claude in similar intuitition sketch.

phase 5 (cleanup) - ask claude to clean up the whole thing and commit. The result is 300+ lines of lean - https://github.com/rkirov/analysis/blob/main/analysis/Analysis/Section_8_2.lean#L843-L1182. I guess to both the unintiated in Lean and the Lean expect it looks like slop (for different reasons). But through this process to me it is a formal proof that I roughly follow.

## followup problem 8.2.6 - divergence after rearrangement

Once I understood 8.2.5, I was in a good position to solve 8.2.6 (which previously I couldn't because I didn't grasp the solution to 8.2.5)

phase 1 - the human idea - I came up with the following construction, take positive elements until 1 is reached, then take one negative elements, then take positive elements until 2, and one more negative. Repeat to infinity.

phase 2 - claude lean writing - Gave that idea to claude to grind. It too took some time (around 30min to an hour). I was observerving its iteration. It is quite far from one-shotting, the constuction is messy to work with, but generally has the right idea in each iteration and hill climbs well. 

phase 3 - intervention. As luck would have it, I actually spotted the Claude stuck on a fundamentaly missing piece from my original idea. How would one prove that the one negative dip, doesn't pull back too far, so we break each bound N, but also oscillate back too much, and not converge to +inf. I came up with using the lim a_f_i -> 0 knowledge that we had from the previous problem. Not sure how productive is it to be watching claudes inner monologue to spot these intevention oppportunities (or how much longer I would have something to add as AI gets stronged. An area I wish AI improves in is knowing me better so I knows when to stop and ask and when to go. Factoring in times where I am not even at the screen, but it is still working.

phase 4 - cleanup and generalization to -inf, asking for refactoring with the -inf case in mind. Claude finishes that and we are done with the chapter, all in one 2-3 hour session. Another 300 line proof - https://github.com/rkirov/analysis/blob/main/analysis/Analysis/Section_8_2.lean#L1184-L1552. When I was writing these by hand, a 300 line proof would take me at least 10 times longer.

Note, I literally asked Claude if this is a correct retelling of our workflow and it agreed.

### Historical asside

I thought it will be curious to see how did Riemann actually prove it -https://www.maths.tcd.ie/pub/HistMath/People/Riemann/Trig/Trig.pdf (used claude for translation)

> The insight into the path to be taken in solving this problem came to him from the recognition that infinite series fall into two essentially different classes, depending on whether they remain convergent or not when all terms are made positive. In the former, the terms may be arbitrarily rearranged; the value of the latter, by contrast, depends on the ordering of the terms. Indeed, if in a series of the second class one denotes the positive terms in succession by a₁, a₂, a₃, …, and the negative ones by −b₁, −b₂, −b₃, …, it is clear that both Σa and Σb must be infinite; for if both were finite, the series would still converge after making all signs equal; but if one were infinite, the series would diverge. Now it is evident that the series can, by a suitable arrangement of its terms, be made to equal any arbitrarily given value C. For if one takes alternately positive terms of the series until their sum exceeds C, and then negative terms until their sum falls below C, the deviation from C will never be greater than the value of the term preceding the last change of sign. Since now both the quantities a and the quantities b ultimately become infinitely small as the index grows, the deviations from C will also, if one goes far enough in the series, become arbitrarily small — that is, the series will converge to C. Only to series of the first class are the laws of finite sums applicable; only they can truly be regarded as representing the totality of their terms — series of the second class cannot; a circumstance that was overlooked by the mathematicians of the previous century, mainly, it seems, because the series that proceed in ascending powers of a variable quantity belong, generally speaking (that is, with the exception of particular values of that quantity), to the first class.

Curiously, if one classifies the three levels of proof rigor - 1) my handwavy intuition proof sketch 2) proper paper math as taught in current math classes 3) formal Lean math, Riemann's proof is closest to 1), of course it was 150 year ago. While we adopted much more strict notion of proofs 2) to root out logical issues, I feel something got lost and with this new workflow my proof writing is suprisingly similar to Riemann's note of the past.

## Summary

I am surprisingly happy with this workflow and keep on charging ahead until I complete the Real Analysis exercises (likely another month or so.) If I used Claude less I likely would have learned more, but at the expense of 10x of my time, it doesn't feel like the right tradeoff right now. I am happy that I can still engage the material on an intuition level. After the iteration we did, I feel the resulting lean proof is *our* proof not just Claude's output. Admittedly I did much less work, but feel more like a collaborator than say a principal researcher whose name is on the paper because he proved the money (I did pay for the Claude $100/month subscription.)

As Terance Tao says in his interview https://www.theatlantic.com/technology/2026/02/ai-math-terrance-tao/686107/

> There’s a big crowd of people who really, really want AI success stories. And then there’s an equal and opposite crowd of people who want to dismiss all AI progress. And what we have is a very complicated and nuanced story in between.

Practically when using AI for writing Lean the two extremes are represented by:

- calling everything a proofslop - AI generated Lean is unreadable, not true, with some training and back-and-forth AI produced proofs are not too far from humans. Formalization is inherently more verbose and noisier because one has to deal with all details, e.g. sum_s in S f(s), really might depend on the mapping from N to s. Asking for the details is aligned with the goals and values of math.
- amazing producer of one-shotted breakthroughs - this is far from the truth, current AI tools still need a lot of steering still and using formal instead of informal proofs.

I believe I am finding a nice middle ground through my usage of Claude and Lean.

When I was in grad school I had a repeated feeling of being unable to see the forrest from the trees. I was grinding through definition and proofs with no intuition in sight, until I left academia althogether. In the new human / LLM / Lean collaborative math world, I feel I can stay more fully in the intuitive level, converse on that level with the LLM, which can then translate it to the formal Lean level (paying the high time cost of that conversion instead of using my human time) and bring back any missing insight to me. Finally, I know enough of Lean to spot check the formal level, but not required fully (especailly outside definitions). I can dream up math ideas, while claude can ground them in Lean, I don't see it as a bad developement.