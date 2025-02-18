---
title: "Learning Lean: Part 1"
date: 2025-02-11T22:26:55-08:00
draft: false
---

## Motivation

I've been captivated by the recent movement to popularize mathematics formalization through the Lean theorem prover, and this year I'm diving deeper into learning it.

For those unfamiliar with this revolution, I highly recommend watching [Kevin Buzzard's talks on YouTube](https://www.youtube.com/watch?v=SEID4XYFN7o&ab_channel=InternationalMathematicalUnion) for an overview of why formal mathematics is generating such excitement in the mathematical community.

The immediate benefits of formalization are well-documented: it helps catch errors in proofs and reduces the need for trust between collaborators since every step is mechanically verified. However, I believe there's another compelling advantage that's less frequently discussed: formalization enables a better separation of concerns in mathematical writing.

When proofs are formally verified by tools like Lean, mathematicians can focus their written exposition on what truly matters to human understanding: the intuition, motivation, and creative journey that led to the proof. Instead of devoting pages to mechanical verification of technical details, we can concentrate on conveying the key insights that make the proof work.

This shift in mathematical writing style becomes even more significant in the era of Large Language Models (LLMs). Having more written text about mathematical intuition and proof strategies, rather than just formal proof details, will be invaluable for training AI systems to understand how mathematicians actually think and approach problems.

I envision future mathematicians collaborating with AI in a two-tiered approach: engaging with conversational AI to explore high-level proof strategies while using Lean and specialized coding assistants to construct the formal proofs themselves. The resulting papers will focus on telling the story of the proof—highlighting the key insights, explaining where the "spark of genius" occurred, and distinguishing these moments from the parts that were "just following the symbols." The complete formal proofs will live in GitHub repositories, serving as detailed technical appendices to the main narrative.

## Audience

I am trying to follow the ethos of "learning in public," but these are still more of my personal learning notes and not (yet) targeting a very broad audience. Also, I am optimizing for getting posts out quickly over a polished presentation.

As such, this post will be most useful for people with a similar background as mine. A quick summary of where I am as I begin this journey:

- formally trained as a mathematician up to the PhD level. I successfully passed my qual exams and defended my PhD. I am most familiar with algebra, number theory, and combinatorics subfields.
- failed to become a research mathematician after one year of postdoc. In the last 15 years, I have worked as a software engineer, mostly in web front-end.
- a deep interest in programming languages and worked briefly with compilers and PL tools - specifically in JavaScript and TypeScript and toyed around with 20+ languages.
- not studied PL theory, type theory, logic, or mathematical foundations (though have read bits and pieces.)

One curious consequence of formalization is that most mathematicians do not care about foundations and the fuzziness of informal mathematics lets them get away with it. But when working on the formalization of mathematics, one has to deal a bit more seriously with foundations basically from day one. So my lack of formal CS Type Theory/Mathematics Foundations knowledge is slowing down my learning of Lean somewhat, but then again a good way to learn more about those.

Math and programming do meet eventually and become the "same thing" (Curry-Howard isomorphism), but ironically the level at which they meet goes through less popular areas each. For math, it is axiomatic foundations and for programming it is FP and dependent type theory. So despite spending 20+ years combined on math and programming, I still only have roughly ~70% of the prerequisites needed to even begin understanding formalization of mathematics.

An analogy I was toying with in my head is if I want to engage with a classic foreign language book like [I_Ching](https://en.wikipedia.org/wiki/I_Ching) one needs to know:

- classical Chinese language
- historical cultural context
- structure of the text and how divination works

Each one is a deep and independent body of knowledge, but one has to put them together to be productive in understanding the text.

## Intro

First, I finished the Natural Number Game and the Set Game [online](https://adam.math.hhu.de/). They do a good job of letting you see just enough of Lean to get a sense of what you are getting into, while having enough challenges (and hints) to keep you engaged.

## Learning the language

I tried to learn the Lean language through [text](https://Leanprover.github.io/theorem_proving_in_Lean4) before getting into the mathematics.

As a programming language nerd, dependent type languages are quite exotic and unlike any other language, even ones with fancier type systems like TypeScript. The rest of this post will mostly go over the things that tripped me on the PL side.

## The three-levels of objects

This took me a while to grok this (big thanks to the Lean Zulip chat participants that answered all my questions with great patience), but unlike the classic values vs types duality, Lean (and other dependently typed languages) have a three-layer type hierarchy.

- terms - variables, expressions, etc. in the programming language
- types - types of terms
- universes - the types for the types

Since everything needs to have a type, one has to answer what is the type of a universe? Turns out it is just a higher universe. So the picture of `what the type of what` (shown as `:` below) is quite fancy (thanks to Kyle Miller for showing me this layout).

```
rfl       3       Fin        ...
  :       :        :
3 = 3    Nat     n => Type   ...
  :       :        :
Prop :   Type  :  Type 1   : ... 
```

This is how I call the objects in each (this is just my understanding, not a formal definition)

```
proof                   value             type ctor            ...
  :                       :                   :
proposition             type         type of the type ctor     ...
  :                       :                   :
propositions univ : basic types univ  :  higher types univ   : ... 
```

## Syntactic separation of the three kinds of objects

When I used to teach TypeScript classes, I had a lot of students struggle with separating the type and term (we called it value) concepts. They were used to languages like Java or C++ with simpler type systems, where there aren't many purely type constructs (to first approximation, the object returned from the constructor for C is assigned type C and that's it). In TypeScript, this is no longer true and there is a whole slew of fancy type-only concepts like conditional types, mapped types, etc. One easy trick we taught them is you can syntactically see whether you are working in the "type world" or not. Basically, `let x : <type world> = <value world>`. In TS these have different namespaces, operators (like `keysof`) etc, but one can always know syntactically what's allowed where.

With dependent types, this easy trick gets much harder. The `type world` has access to the variables of the `value world` and vice versa, the types themselves can appear on the other side because they have universes as types. When reading and writing Lean I kept losing track of which objects can be written where, because in theory there is no limitation.

In theory, the homoiconicity of the types and terms language is nice once one builds up the mental familiarity (and especially if one is building tools for the language). For example, in Lean a type function call is the same as a term function call, while in most mainstream programming languages generics use a completely different function call syntax - `T<S>`. But for beginners, homoiconicity is disorienting. I guess, it's how new LISP learners feel like.

## Optional names vs optional types

My other big confusion came from the fact that variable names can be added to type signatures (which is needed for dependent types, but allowed even if only classical types are used). For example, in `def f : Nat -> Nat := fun x => x + 3`
we can expand (vacuously) the definition in two different ways:
- `def f : (x: Nat) -> Nat := fun x => x + 3` 
- `def f : Nat -> Nat := fun (x: Nat) => x + 3` 

The first adding the optional arg name in the type in case we need it for dependent types, and the second adding type to the argument, which is optional because of type inference.

So depending on the syntactic position `a` could stand for `(a: \alpha)` or `(aa: a)` which can be very confusing to the novice when `a` is some generic parameter.

## Numeric Literals and Type Classes

My first interaction with Lean was broken for a bit due to numeric literals being objects in type classes. Reminded me of my first experience with Haskell close to 20 years ago, which involved typing `let x = 1` and getting an inscrutable error. Much later, I learned it has to do with the same design decision.

While it is very defensible and beautiful to be able to use a general solution like type classes for literals, to a newcomer it feels inverted - literals feel simple, while type classes are fancy. For example, I started experimenting with numbers after reading chapter 2, while type classes are explained only in chapter 10. So I got some literal type class errors, before technically I knew anything about type classes. 

My past experience with Haskell helped, but I wonder how many newcomers (say mathematicians that have written a bit of numeric python) get lost around this.

## Observation on Curry-Howard

As the saying goes - in theory there is no difference between theory and practice, while in practice there is. Similarly, by Curry-Howard programs and proofs are the same thing, and Lean puts them as close as possible on equal footing. But in practice, they are still separated into two big universes `Prop` and `Type` (from what I gather it has to do with impredicative). In practice, "theorem proving programs" also end up looking quite different from computation programs and the language has many constructs to improve the experience of writing proofs like tactics, implicit arguments, etc, that are not needed when writing computational programs.

In theory, something like implicit arguments and the `variable` command should be universally useful for any programming language, but in practice, those feel like they will add too much magic for what they solve in regular programs, while in theorem proving they seem necessary to improve the developer experience.

## My remaining questions

- when are two things definitionally equal vs I have to write a proof. Basically, I just try `rfl` and if it doesn't work get to proving, but I wonder if experts have an intuitive understanding of what should work before trying.
- Prop vs Decidable - I vaguely understand the distinction, but can use more examples. Also would like to know does this relate to the `noncomputable` property.
- #eval vs #reduce - what's the difference? Feels like two different evaluation models.
- relative power of Lean proof-based simplifications (using `simp`) compared to CAS like Mathematica and SymPy that also have `simplify` but as a black box procedure.

These will likely become clearer as I continue to experiment with Lean.

## Final words

As stated in the beginning [chapter 8](https://Leanprover.github.io/theorem_proving_in_Lean4/structures_and_records.html)

>  We have, moreover, noted that it is a remarkable fact that it is possible to construct a substantial edifice of mathematics based on nothing more than the type universes, dependent arrow types, and inductive types; everything else follows from those. 

After finishing the text, I understand what is meant and share the awe of how remarkable this is. What seems like a small (but quite unorthodox) functional programming language allows one to build all of known mathematics on top.

As a quick review of the [text](https://Leanprover.github.io/theorem_proving_in_Lean4/) - it covers a lot, has a good pace, goes deep when needed, with good examples and exercises are good (but can use more of those). My main critique is that sometimes it starts using concepts or syntax that was not introduced earlier (the curse of knowledge for the authors). 

I have gotten through the basic oddities of working with a dependently typed language and ready to move to doing [Mathematics in Lean](https://Leanprover-community.github.io/mathematics_in_Lean/mathematics_in_Lean.pdf) next.

Big thanks to the Lean Zulip community who has been very helpful and welcoming through this. I paired my learning with asking Claude many questions. It did well for the simpler ones, but only the experts on Zulip could help me with the harder ones and could connect my questions with bigger conversations, past blog posts, or papers.