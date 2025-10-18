---
title: "Why formalize mathematics - more than catching errors"
date: 2025-10-16T22:32:49-07:00
draft: true
---

I read a good [post](https://lawrencecpaulson.github.io/2025/10/15/Proofs-trivial.html) by one of the authors of the Isabelle theorem prover, that got me thinking. The author, Lawrence Paulson, observed that most math proofs are trivial, but writing them (preferably with a proof assistant) is a worthwhile activity, for reasons similar to safety
checklists - "Not every obvious statement is true."

As I have been a bit obsessed with doing formalized mathematics this got me thinking about why am I excited to spend
many hours recently writing formalized proofs in Lean for exercises from Tao's [Real Analysis](https://github.com/rkirov/analysis) (along with this recent attempt to write a companion to [Riehl's Category Theory In Context](https://github.com/rkirov/category-theory-in-context-lean.)

At a very personal level, I simply enjoy math, programming and logical puzzles, and writing a Lean proof is basically
all three of those in one. Literally, in May when if I didn't spend more of my spare time on Lean, I was planning
to beat this [puzzle game](https://thinkygames.com/games/can-of-wormholes/.)

But this is a very narrow view, that is shaped by my
very specific past (like the fact that I studied math, but did not get into research, and that I spent last 15 years wokring as an engineer). Even after abstracting from my background, I still believe formalized math (currently with Lean, but down the road likely with a more powerful successor) is a very promising and exciting development for mathematics.

It might come to a surprise to you, but that is not solely due to the obvious benefit of finding potential errors in proofs as they are checked by a computer. While the benefits of more correctness is obvious and correctness is core to the mathematical project, I will discuss some other less than obvious benefits of shifting to formal math.

I have seen a similar story repeat in my engineering career when I worked on the team that brought TypeScript to Google's 
internal codebase.

## TypeScript story

TypeScript is an optional static type system for JavaScript (a dynamic language without static types). Very briefly,
when writing TypeScript, one is roughly writing JavaScript with some extra annotations - types. Those are checked by the TypeScript compiler, as part of the compilation / build process. If the types annotations are not consistent - say you
said the function `f` expects a `number` but somewhere you also wrote `f('foo')` an error will be thrown. If no such
errors are found, the type annotations are erased and JavaScript is emitted for further execution.

It seems all TypeScript is good for is catching errors and rejecting your programs. So when we went around Google pitching
different teams to migrate, the questions were always around how many bugs can this system find. What we soon realized is
that actually finding bugs in code that was tested and in production for many years is not that valuable. Some of it
is due to JavaScript being mainly used for UIs, so worst case scenario one often can just refresh the page (thankfully noone is writing mission critical software in JS, right?!?). And even more surprising the risk of fixing the "bug" that TS found was often higher, due to new errors that might arise.

If if correctness wasn't that big of a deal (especially over estalibshed codebases) what good is this type-checking. As I learned more about 

- TypeScript types power IDEs - turns out whatever the correctness checker needs to connect say the definition of `f` and 
all callsites to verify that the arguments match the parameter signatures, is the same thing a developer tooling needs to 
allow you to CTRL-click to definition, find all usages, refactor, etc nice
- TypeScript types are a design language - once you get familiar enough one can use them as an initial language to spec
design before the implementation. So a team of TS developers gets to share a mini-specification language for free, that they can use is early design before any runtime is even writen.
- Immediate feedback correctness - TypeScript generally integrates with your IDE, so while checking correctness of already writen and testing codebase is low value, having the correctness checks appear right underneath the code as you are writing it in the infamous "red squiglies" does offer big value, because of the immediacy. Caveats of course are, assuming
one is not doing TDD and writing tests first, and the project is not too large when type-checking starts to bog down and can't appear as you are typing.

## Analogy with Lean and Math

So by analogy, formalizing math with Lean has the following similar benefits outside correctness and assistance in writing proofs.

- powering math IDEs - instead of just writing Latex, and keeping all definition in ones head, with Lean one gets the
very basic IDE support every programmer expects, but no informal mathematician gets - click-through defintions, hover over statements, auto-generated docs (like https://leanprover-community.github.io/mathlib4_docs/), non-string searching, refactoring, etc. For example, the [Stacks Project](https://stacks.math.columbia.edu/) is a very useful attempt to get 
some of these benefits without formalization, but formalization basically guarantees that every part of mathemetics can
generate something like this.
- analysing meta-math trends - with fixed names of theorems and cross-project imports, one can generate more easily and reliably the meta-structures of mathematical proofs, which theorem use which other ones. 
- repository and version control - if a result is retracted currently there is no guarantee that other results that dependend on it would even know that happened and recursively retract their claims. Turning mathematical results into actual libraries with versions and dependency management will make the evolution of mathematical truth more streamlined.

Basically, the process of doing math will become more effienent and hopefully more pleasant. The price one has to pay is
proving a bunch of trivial proofs, which mathematicians often do anyways, but something like Lean will not allow anyone to
cheat and be too creative with what is "trivial". To Lean everything needs a proof, that said there are more and more powerful tactics being built, so "trivial" results do get often one line proofs, except one doesn't get to decide what 
is trivial, trivial is what leans currents set of tactics can close in one-line, the rest need more work.

Finally, note that a lot of the benefits I am describing above don't even need to have the full formal proofs, just formal statements is enough. This effort by Kevin Buzzard with support from Renaissance Philanthropy seems to confirm that just 
having statements is very valuable task by itself - https://www.renaissancephilanthropy.org/a-dataset-of-modern-formalized-theorem-statements. 