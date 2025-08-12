---
title: "Learning Lean: Part 3"
date: 2025-06-27T22:30:30-07:00
draft: false
---

I am continuing to learn Lean (see [part 1](/posts/lean1) and [part 2](/posts/lean2)). I lost some steam around March-April, but in the last
two months I picked it up again. In a way it was a nice spaced repetition for relearning some of the basics. This summary was supposed to be my notes from finishing the rest of [Mathematics in Lean](https://leanprover-community.github.io/mathematics_in_lean) and finishing chapters 2 and 3 of exercises from [Terence Tao's Analysis textbook](https://github.com/teorth/analysis), but also I couldn't help also comment on the challenges and 
opportunties I see in doing formal mathematics with Lean.

## Thoughts on Mathematics in Lean

I enjoyed the first part of the book a lot more than the second. Naturally the second part goes deeper in mathematical topics
and the exercises start to have a very different feel to them. Especially around the topology and filter chapters, my own
understanding of the mathematics involved got shaky, and it was hard to learn/refresh a topic simultaneously with learning how to do it in Lean.
I think there is a missed opportunity to stay on each undergraduate topic longer, so that one can focus more on Lean techniques
over concepts they are more comfortable with.

There is an interesting point at which the complexity shifts from just manipulating Props and juggling theorems over
relatively simple objects like naturals, to working with a much bigger hierarchy of structures and understanding how Lean does polymorphism using 
type-classes (running into Lean's version of the diamond problem, etc.) Even after reading the book, I feel unprepared
for building up a mathematical subfield from scratch in Lean, while I am getting comfortable manipulating proofs. There are too few exercises to build more Lean expertise, so it mostly serves as a crash-course tour of mathlib to give you an idea where to start if you want to do more with Lean for a specific 
math field.

## Thoughts on Tao's Real Analysis Lean companion

After finishing Mathematics in Lean, I faced a big problem. From learning maybe 10+ programming languages
I know that one learns programming by writing lots of code. What the code does shouldn't matter much 
in the beginning, in fact it is often best to write code for the same computation problem you already
know well and have code for in another language. For example, something as simple as how to read 
and understand syntax and type errors the compiler spits out is something that no tutorial goes in depth on - they often assume code is written correctly at first.

But since this is my first foray into theorem proving with a computer, I have no other prior formal proofs 
to attempt to translate to Lean. Also, it looks like a lot of other folks are active researchers and after reading 
the basic tutorials they start to formalize their active research. I am not doing any research right now
so have no such area to do.

In what amounted to a lucky coincidence Terence Tao put out a companion to his Real Analysis book around 
the same time, which turns out was exactly what I needed. The first four chapters are axiomatic bootstrapping of mathematical foundations
through Peano Axioms, ZFC for sets, building up integers, rationals and reals. The Lean companion does a great job 
of building each of those from scratch, so one can incrementally learn how Lean enables classical math foundations, while
replacing them with the more ergonomic mathlib versions in the next chapters.

Working through the first four chapters has a great feeling of "bootstrapping modern mathematics from scratch", like 
those build your own Linux kernel or OS guides, except of course a lot more discovered not invented - we are working 
with what nature has given us, not some other lower level human engineering.

Writing a custom version of the foundational axiomatic natural, set theory, etc, is also a great educational 
technique, because it forces one to learn how to work with Sets and functions by doing, instead of just 
reading mathlib's source. Reminds me of those "you could have invented X" blog posts. Of course, just like those
it makes it clear that while you could bootstrap those from scratch, to achieve the same level of ergonomics
as mathlib is much harder, and it is best to swap for mathlib's version eventually.

The companion does a great job of proving enough scaffold so one doesn't get lost right away, but also plenty of
exercises to solidify Lean knowledge and learn some new tricks. There is also an active channel on [Lean's Zulip](https://leanprover.zulipchat.com/) where folks actively working on solving can discuss solutions and challenges.

Overall, it has been a great source for me and I plan to keep working through it. My solutions 
can be found [here](https://github.com/rkirov/analysis). Note for now, I am taking a bit of 
a brute-force approach (a proof is a proof) and plan to focus more on proper Lean proof style later.

## Accidental vs Inherent Complexity

There is no denying that formalizing mathematics is complex and more time consuming than pen and
paper proofs. As I struggle through the various exercises I am doing to learn, I keep asking myself how much of
 the complexity experienced is:

- accidental - attributable to poor decisions or missing features in Lean or Mathlib.
- inherent - it is truly what it takes to write a mathematics proof

My current feeling is that I am observing more inherent complexity, something like an 80/20 split.
At least based on my struggles so far, Lean is pedantic in a way that is aligned with 
the spirit of mathematical rigor that non-formal mathematics also aspires to, but is not 
forced to follow the same way a Lean proof is. For example, non-formal mathematics can just
claim "the proof is trivial". Lean will not let one do that and it might seem annoying at first,
but there is something to be gained by not allowing one to sweep random parts of the proof
 under the rug without any accountability.

Here are two examples:

- accidental complexity - I struggled a bit in chapter 3 dealing with the various ways we embed natural
  numbers in the ZFC set axiomatic systems. Notably, Lean makes one deal with literals separately 
  from Nats [see here for example.](https://github.com/rkirov/analysis/blob/main/analysis/Analysis/Section_3_1.lean#L887-L900)
  This is a purely Lean decision, and one can imagine formal systems that deal with
  natural number literals differently ([see agda](https://plfa.github.io/Naturals/#a-pragma) for example.)
- inherent complexity - the longest proof I had to write had to do with computing the cardinality 
  of the set of permutations [here.](https://github.com/rkirov/analysis/blob/main/analysis/Analysis/Section_3_6.lean#L2747-L3295)
  What made it long is making sure all indices align appropriately in the right `Fin n` sets.
  While surprisingly long and somewhat chore-ish, it all felt very aligned with what I believe
  is the right level of rigor for a math proof. The things Lean was asking me to prove were all
  things that I should have proved on pen-and-paper and if I would have skipped them it was more 
  of an indictment to my non-formal proof being sloppy. As Lean gets smarter higher order tactics
  the size of this proof could shrink as we offload parts to Lean internals.

That keeps me motivated to keep learning, as I see it as more of "the right way to do math" even 
if it is more complex and time consuming. For me personally, as someone who is outside academia, but still 
wants to interact with mathematics, the definition of valid proof as "something that is accepted by Lean"
is obviously superior to "something that is accepted by majority of other professional mathematicians",
because I do not have time or opportunity to interact with professional mathematicians.

## What is a proof

As an interesting anecdote, I recently went to a math meetup where we were working on pen-and-paper proofs 
of some basic algebra results. One of my partners in the exercise had the right idea, but struggled to 
formulate a proof, at least to what I consider the appropriate rigor of a proof. But oddly I found myself
unable to communicate my critique - "You have the right idea, but this is not a proof", they retorted with "It is a proof to me."
Trained mathematicians generally avoid this pitfall by spending years poring over proofs to soak up implicitly
what is acceptable and what is not and basically align (though not perfectly) on a single idea of acceptable rigor. But that leaves the door closed for a hobbyist, who doesn't have the time or access to a trained mathematician to be able to verify whether their hand-written proofs are of the right level of rigor (which is not written anywhere as a set of rules to follow.)

This episode was an unexpected reminder of the value of a single agreed upon definition of what is a valid proof - "whatever Lean accepts".
The high number of so called "cranks" that professional mathematicians have to deal with can also be seen as a practical
failure of the methods of mathematics - these are folks that are obviously drawn to mathematics, but unable to
interact with the mainline of mathematical thought they drift outside what is a mathematical proof, though often keep some
semblance of such. Having a shared mechanical definition of what is a proof, would be such a boon for 
hobbyist mathematicians, I am almost surprised it has taken us so long to get here.

## Math as one giant puzzle

Finally, working on formal proofs with Lean turns all of math into one giant puzzle. It has the quintessential 
components of a puzzle - a clearly defined goal, rules to apply, and a definition of done.

My interest in math has always been a reflection of my love for puzzles, and the new fast feedback that Lean provides
on which pieces "fit" and when the puzzle is done greatly enhances the puzzle experience and the dopamine boost I get 
from solving it. 

## Proving is computing in theory, but the pragmatics are all different

In a slight paraphrase of the old saying - in theory, proving and computing are the same (aka Curry-Howard), but in practice
they are very different. Somewhat amusingly (as I do enjoy the beginner's mindset), despite knowing programming
pretty well after 15 years as a professional software engineer, I find myself very out of familiar waters when 
proving with Lean - much more so than if I was trying to learn one more programming language.

One basic realization I had is that I don't know how to refactor Lean proofs well. The most basic ideas of refactoring—
isolating repeated code in variables and functions—don't work in proving, as just having more `let`s doesn't make
anything better without adding some theorems (Lean folks seem to call them API) that are intended to go
 together with those new definitions. So what is a single atomic unit of refactoring—a new variable/function definition—
  in Lean becomes a whole object plus appropriate API (almost like OOP ironically), and one has to carefully 
  find the right API that preferably captures the essence of that object, while simplifying by hiding some details.
Failing to provide the right API means that every proof now needs `unfold` and basically gets more complex, and
the refactoring fails to achieve any goal of a simpler, more readable proof.

AFAICT, it is early days for large-scale theorem proving and no one has written much on "good Lean style," "clean proofs,"
etc.—things we have in software engineering—so ideas are mostly disseminated by reading other people's proofs like
mathlib and conversations on Zulip. The analogy with computing is strong, so it is very clear the direction this
will take if it keeps growing.

The irony of the purest of pure mathematics benefiting from the years of lessons of large-scale software engineering
is not lost on me. According to Claude there are 500x more software developers in the US than mathematicians, so
it is exciting to bring all these established and battle tested tools for large-scale software development to 
accelerate and improve mathematics.

## Use of AI

My Lean journey would be greatly behind if I did it a few years ago before the advent of LLMs. I use GitHub Copilot 
(only useful for very repetitive parts of proofs) and Claude AI for close to 90% of my questions. Claude does 
hallucinate names of theorems in mathlib, but generally its intuition of what should be there is good, so I
can extract the theorem as an intermediate `have` and close it with `exact?`. The hardest last 10% of questions
are still best answered by the Lean experts on Zulip. AI keeps me from asking the trivial questions (which as a beginner
I don't even know how to separate) in that forum, so I don't DDOS it with too many questions. Overall, as a new domain
I see the benefit of AI even more than in a domain like programming where I am already an expert.

Someone has asked me if due to AI's success the whole point of learning Lean is useless, but from what I am 
seeing we are very far from AIs just spitting out correct proofs without some human nudging (despite all the IMO 
successes), and to do the nudges we do need to learn Lean.

## Some random gotchas

Finally, on a more practical note, working through chapters 2 and 3 of Tao's Lean companion to Analysis taught me 
some particular new Lean tricks. I am getting a feeling that my bag of tricks is large enough to not be completely stuck on
basic proofs any longer. The next level of learning is how to write idiomatic and clean proofs.

The tricks and gotchas I learned:
- be very clear when working within `Type` or `Prop`. Certain things are only allowed for `Prop`, like `obtain`.
- Every `Classical.choose` needs a matching `choose_spec` to work with. Using `generalized_proofs` allows one to pull
  the proofs in scope.
- If `rw` fails due to the `motive` error (often the thing you are trying to rewrite is used inside other types, etc), try 
  `simp`.
- always look over what's behind the ↑ (or turn them off with the right pp option). There are at least 4-5 different
  coercions that are represented with ↑, but require non-trivial theorems to transform appropriately. Sometimes the 
  right API theorems are not even there, one has to add them (and probably upstream).

## Reason to worry

The only thing that left a bad taste in my mouth in the last two months, is hitting performance problems with some
proofs. Using `set_option maxHeartbeats X` helped me get over it, but I didn't expect to hit such issues so early
in my Lean journey, on what are IMHO pretty basic proofs. Again using analogy with programming, I am writing what
feels like basic for-loops and if-statements programs, but already starting to need to learn about profiling and 
internals of the computation model in order to make progress. Feels too early in my journey to be becoming
performance minded and I wonder how much of it has to do with Lean's design vs inherent to theorem proving.

## Conclusion

I am having a lot of fun learning Lean and doing the exercises from Tao's book. It 
is the right next step for me to level up my Lean skills and I recommend it to
others in the same stage of the journey. If you are algebraicly minded mathematician
don't be offput by the Analysis title, at least the first four chapters are equally foundational, assuming you are not expecting to see category theory as foundations instead of set theory.

I am totally sold on formal proofs as the future of mathematics. Lean has the most 
momentum at the moment, including an active and welcoming community and I see no reason to try to learn other alterantives like Isabelle or Coq at the moment.

Formal mathematics, i.e. checking proofs with a computer, is especially beneficial to hobbyists like me. I hope one day these are prevalent enough so we drop "formal" and just call them proofs, like how we don't need to say "computer-aided computation" to differentiate computation we run on silicon instead of our limitted brains.

P.S. This post was too long. Thank you for reading if you got so far. I do need to blog more often in smaller chunks.