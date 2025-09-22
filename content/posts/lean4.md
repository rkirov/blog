---
title: "Learning Lean: Part 4"
date: 2025-09-20T07:54:08-07:00
draft: false
---

It's been 3 months since my previous post about learning Lean [part 3](/posts/lean3), so it's time to write another one. I have mostly continued to work through Tao's Analysis book through his excellent [companion](https://github.com/teorth/analysis) - which means all proofs from the paper textbook are fully formalized in Lean, while examples and exercises are stated in Lean but left with a `sorry` for me to fill in. Altogether the companions has close to 2000 sorries to fill-in.

Since my previous post, I fully finished chapters 4 and 5, which amounted to constructing the integers, the rationals, and finally the reals (using equivalence classes of Cauchy sequences). I continue to enjoy the approach of building the concept from scratch (even though it is in mathlib), proving the main theorems, showing an equivalence with the mathlib construction, and in the next chapter switching to mathlib for the said concept and repeating the journey for a new one. It is a bit surprising how many theorems one needs to prove to get from the axiomatic foundations of sets and naturals (which were chapters 2 and 3 of the book) to having a fully rigorous formal construction of the reals and all the "trivial" properties that one expects to be true (like $x^n * x^m = x^{n + m}$). That said, it is very satisfying to see that everything is formally proved and logically follows from the constructions. I see it like the math version of building your OS kernel, except doing with a computer what mathematicians first did in the late 19th century.

## Progress

At this point my Lean skills have improved to the point that I can express basic mathematical ideas - meaning writing basic undergraduate proofs involving inequalities, integers, rationals or reals, induction, sets, etc. - and mostly debug my own issues and unblock myself.

Unlike chapters 2 and 3 where the proofs were almost trivial when done on paper, in chapters 4 and 5, a lot more of my time was spent actually figuring out the actual mathematical content. The hardest proof was showing that roots exist for real numbers, for which I had to open Rudin for reference.

The second most time-consuming issue was finding the right mathlib theorems to use. Learning to use [loogle](https://loogle.lean-lang.org/) well is becoming a very invaluable skill (sadly, for some reason `exact?`, `apply?`, and `rw?` have gotten less useful to me). For some of the harder problems in the chapter 5 epilogue, there were even missing theorems in mathlib that I plan to upstream at some point.

## What's next

While I do plan to continue to work through the companion, and I do find it enjoyable like playing a logical puzzle game, I am starting to think that the next step for me is to start paying more attention to proof-writing style. Similar to code, writing readable and maintainable proofs is a skill that needs to be intentionally built after the basic "make the computer do what you want" is achieved.

Having gone from my first 10-line proofs to some 500-line proofs feels very similar to my software writing path some 20 years ago, when I started noticing that I can make the computer do what I want, but unless I follow good style and practices, the code starts to collapse under its own weight. Oddly, proofs can't really have bugs, but one visible impact I am already observing when writing long, not well-structured proofs is noticeable type-checking slowdowns.

To make things harder, as a language Lean is insanely extensible (think Lisp, but with a fancy type system that macros can interact with), so there are many ways to complete a proof - from an extremely verbose, long and detailed series of `rw`, to a one-liner magic high-level tactic like `aesop` or `grind`.

I can spend time reading high-quality Lean proofs like mathlib, but that sounds much less fun than getting a proof through the checker, so a better plan is to start contributing Lean code to active mathematics projects where I can get code reviews and learn through code reviews from other more experienced contributors.

## Proof engineering

The movement to formalize mathematics is implicitly turning the process of doing mathematics into software engineering. While obviously related to software, writing computer proofs is substantially different enough to deserve its own term, and I like the term "proof engineering" (I first read it in a Dan Abramov bsky post).

This means a mathematician, even in the most abstract of fields, when working on formalization has to deal with practical software-like concerns like proof style, structuring abstractions, naming conventions, etc. Notably, this is very different from dealing with theoretical computer science (like type theory). Of course, a mathematician working on formalization does need to work with fancier type theories than any software engineer, but outside working on mathematical foundations and logic, they need to know a bit about the specific flavor of types implemented in Lean (far from a type theorist), but be actively practicing proof engineers.

As far as I can tell, this is an area that is still developing as formalized mathematics is growing orders of magnitude over previous heights, and Lean itself is somewhat new as a language. My prediction is that proof engineering versions of engineering classics like the books of Martin Fowler, Michael Feathers, or John Ousterhout will eventually start to materialize as more and more of mathematics starts to be done on a computer and proofs do present similar yet different challenges.

Note, I just looked up [proof engineering](https://proofengineering.org/) and there seems to be some interesting prior art, but notably it lost steam around 2020 and has no mention of Lean.

## Living the abstraction dream

Speaking of notable differences from writing software that actually executes, since proofs don't run (unless you count type-checking, which of course when sophisticated enough is another type of computation), when writing proofs one gets to experience the proverbial software engineering dream - abstractions that can't leak. Because proofs don't run, they have no side effects; you don't need to worry about whether the proof accesses disk, network, runs inefficient algorithms, etc. Dan Abramov made the same observation [recently](https://bsky.app/profile/danabra.mov/post/3lzcdmxqdb22w).

In fact, if you can't write the proof yet, you can just put `sorry` and continue any other proof that uses the statement, as if the proof was there. In software this would be equivalent to writing function signatures with types, but body of just throwing, and successfully running the whole program without having an implementation. What?!?

The only thing you need to worry about is the binary statement - was it actually provable or not. I did get bitten a few times by assuming a proof that wasn't correct, but it wasn't anything I couldn't fix with a few extra hypotheses.

Assuming a proof actually exists, which particular one gets written at the end doesn't matter to the rest of the code using it. What's even cooler is this is not just a meta-property, but actually expressible in Lean itself and called [`proof irrelevance`](https://github.com/leanprover/lean4/blob/9fc18b8ab462cb9100d37a23814ebbac330e8577/src/Init/Core.lean#L842-L843) (though it cannot be proved itself and it is magically built-in, it seems).

## Using Claude

I am continuing to use Copilot and Claude to help me with proofs when applicable. The outcomes are mixed, but AI is improving rapidly, so I am willing to put up with the slight annoyance when it's wrong.

When I use AIs when coding, it is a tireless intern/junior developer that I need to manage and guide when it veers off course. While when using AI in an area where I am a novice myself, it is a very different experience. Claude's answers feel like a drunken retired professor that has seen it all, but also is getting it all mixed up. It can over- or under-simplify things, or be just plain wrong.

It can send me on a very distracting direction, most annoyingly inventing mathlib theorems that don't exist, but it does often get me unstuck by mentioning some concepts I didn't even know existed and wasn't sure how to even search for.