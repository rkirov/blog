---
title: "Learning Lean: Part 2"
date: 2025-03-02T07:14:48-08:00
draft: false
---

I am continuing to learn Lean (see [part 1](/posts/lean1)) by going through
[Mathematics in Lean](https://leanprover-community.github.io/mathematics_in_lean). These are my notes as I just finished chapters one through five.

## Mathematics in Lean

The online book is well-paced and well-written to connect with what an average student of mathematics already knows. The focus is on learning how to do basic mathematical proofs in Lean, and the underlying language is taught as needed for those goals, in comparison to Theorem Proving in Lean which is more of a language guide. For example, the fact that naturals
are constructed as an inductive type with zero and succ constructors is shown much after one is shown how to work with them. This is aligned with how a lot of mathematics is done before one discusses foundations in a typical mathematics curriculum, which might differ from a CS one.

I especially enjoyed doing the multiple exercises in each section. They are really well interspersed within the text and well-balanced to keep one at the edge of their current skill. The "gamification" really works for me, and it turns learning Lean into just another puzzle similar to
the [puzzle games](/posts/puzzles2024) I enjoy. It makes me very bullish on using Lean as an educational tool.

## Beginner gotchas

Despite a lot of work on good errors, every programming language has those areas where a slight typo sends you in a completely different path in the compiler, and the error you get is inscrutable. A lot of beginners drop off
due to those, but if they get over that hump, they can pattern match on the less-than-perfect error and it stops being a problem for intermediate or advanced users.

Here is my beginner errors list (in case it is useful for someone else):
- number literals are type-class polymorphic and sometimes a bit of type hinting is needed - `(2 : \N)`
- starting to use tactics without `by`
- using `|` instead of `\|` for `divides`
- wrong imports - syntax highlighting shows the whole block as red, instead of just the one wrong import

## Simple made hard

The common complain about formalizing mathematics is that it makes the simple things like `I have this equation ... + b = ..., move b over to get ... = ... - b` hard because one has to find the right formal way to prove it. I see the concern, but also not as worried - at least for simple undergraduate math so far. In the short few weeks I have been playing with Lean, I have gotten noticeably better at these manipulations and there are a lot of tools to help. For the example above one can:

- learn the somewhat standard naming schema of mathlib - something like `add_eq_sub`. I haven't internalized it yet.
- use VSCode symbol auto-completions
- navigate to the mathlib or built-in file with theorems for the underlying type and look around
- abstract the desired manipulation to `example (a b c : \R) (h: a + b = c) : a = c - b := ...` and use tactics like `exact?`, `apply?` or `hint` for Lean to complete.
- ask [loogle](http://loogle.lean-lang.org/) - like google for Lean types
- get in-editor LLM autocompletion from Github copilot 
- ask AI like ChatGPT or Claude
- ask an expert in the Lean Zulip chat

With all these options, I found that I can unstuck myself faster and faster.

## Simple made simpler

As a counterpoint to the above, formalization does offer some great benefits even for simple intro math proofs:

- "gamification" mentioned above - doing pen-paper proofs especially if learning how to write proofs means one has to either 1) verify the proof themselves 2) give it to someone else to verify. Option 1) is error-prone and easy to fool oneself and 2) is slow. I learned non-trivial amount of algorithms by doing coding challenges like[leetcode](https://leetcode.com/) and [advent of code](https://adventofcode.com/) and with Lean this can be extended to proof-based math.
- a digital brain extension - having all the hypotheses visible at a glance, updating the goal, hovering over definitions, and all other UI niceties, actually make simple mechanical proofs even easier to write. For example, the theorems for function image and preimage on sets from [chapter 4](https://leanprover-community.github.io/mathematics_in_lean/C04_Sets_and_Functions.html#functions) were something I have done on pen-and-paper, and doing them with Lean is strictly better for me. These are basically mechanical transformations without much insight and doing mechanical things are strictly better with computers.
- standardization - countless math books begin with a chapter on notation (while still each being slightly different) and basic theorems to be used later (with custom naming schema like 1.1). It is hard to argue that there is much value in all that replication. While Lean is extensible and in theory everyone can build their own mathlib or custom tactics, it is less likely because it is paradoxically harder to do when it is required to be fully checked.

## Modeling Formal Mathematics

It is somewhat surprising to me that Lean does so well in modeling mathematics with type theory at its core instead of set theory. Moreover, it is a type theory without subtypes and using type classes for all needed ad-hoc polymorphism. Without getting into a full OOP vs FP flame war it is interesting to observe the how well the type classes approach works out for the pragmatics of actual mathematical proof theories.

The choice of type theory means that some very reasonable mathematical statements like `ℕ ⊆ ℤ` are not even meaningful in Lean (which is kind of amusing), but so far I haven't found this to be a problem - most theorems are proved in a fixed type or appropriate type-class is picked automatically.

Instead of subtypes, Lean has the `Coe` type class and AFAIKT it serves basically the same purpose - a natural number can be passed to a function that needs integers without an explicit cast needed.

Finally, my biggest surprise was how Lean defines division and subtraction over bigger domains than what one expects. Seeing that `3 - 4 = 0` and `1 / 0 = 0` are provably true statements in Lean might raise some eyebrows. I guess the alternative is to either only define the function over more limited domains (which now have to add extra type coercions) or let them only work when a proof is passed. Neither seems great, so this is an interesting design choice that seems to work reasonably well. It can feel a bit annoying that certain identities need extra hypotheses to work, but it might be the least bad choice.

In the computational math world this feels like known bad choice (not a security expert, but I think silent integer overflows cause a lot of security bugs). But in the theorem proving world, where statements are already verified and the extra behavior , it doesn't seem like a terrible choice. Sure x => x - y is not injective over the naturals, but one has to deal with it in all proofs.

## Super-tactics

While I am starting to grok the basic tactics around inductive types - `intro`, `cases`, `induction`, the specialized super tactics - `aesop`, `linarith`, `ring`, `simp`, `decide` are still a bit of mystery to me.

It feels that understanding better when and how to use them (at some point I expect performance to become a concern too) is what the the practice of math formalization is about, and there is no shortcut other than doing a lot of formalization.

## Working through open problems

One passing thought - all examples of proofs in MIL are for well-known positive results. So far I have done maybe close to hundred universal proofs (true for x,y,z in some type) and one counter-example. But when working on a real math problem, one often alternates between attempting to prove the general statement, and trying to find a counter-example that would reject the hypothesis directly.

It would be instructive for me to see such a workflow in Lean (maybe it comes later in MIL, I haven't finished it). Something like - iterative transformation of the hypothesis goal, until one spots a goal that is more clearly unprovable. Then taking that and turning it into a counter-example to show falsehood of the original universal statement, ideally keeping the original transformation.

There are a number books of counter-examples like [Counterexamples in Topology](https://en.wikipedia.org/wiki/Counterexamples_in_Topology) and I wonder if these are well-suited for formalization too.

## Next

I am finishing the remaining chapters in MIL next.