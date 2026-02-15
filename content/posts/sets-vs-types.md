---
title: "From Sets in Math to Types in Lean: Subtype, Fin, Set, Finset, and Fintype"
date: 2026-02-14T20:54:04-08:00
draft: false
---

## Preface: Why All Mathematicians Should Learn Lean

LLMs can generate plausible-sounding proofs at unprecedented speed and scale. Some are correct, many are not, and LLMs themselves cannot reliably tell the difference. Humans can — but they don't scale. Formal proof assistants like Lean do: they verify correctness mechanically and can serve as a backstop to the torrent of AI-generated mathematics.

This puts mathematicians at a fork. Either accept formal proofs as the verification layer, or reject all LLM-generated mathematics outright. The first path seems far more productive — but it requires mathematicians to read and write formalized math, at least well enough to verify that formal statements faithfully capture informal ones.

I've been working through a Lean companion to Tao's *Analysis I*, and the experience has surfaced many small friction points — places where the gap between blackboard math and Lean's type theory causes real confusion. This post is one in a series meant to address those friction points with practical guidance.

Today's topic: translating set-theoretic language into Lean's type system. This is not a theoretical comparison of set theory vs type theory as foundations. It's a practical guide for working mathematicians who take a piece of math written in set-theoretic notation and try to formalize it in Lean — and immediately hit confusing errors.

## Basics

I assume you are familiar with the basic language of sets (even if not its formalized version as ZFC). Generally, there are primitive objects and sets, which are unordered collections of objects or sets (skipping a lot of details here, but this is the rough practical version a working mathematician uses). One notable property is that an object can be a member of many sets — e.g. 0 is a member of `ℕ`, `ℤ`, `{0}`, `{0,1,2}`, `{x | x ∈ ℤ, x > -1}`, etc.

In type theory, we similarly have objects and types, where objects belong to types and types themselves belong to higher-level types called *universes*, but notably each object belongs to a unique type. It is an inherent property of the object — at any point the type system knows its unique type. In Lean we write the type after `:`. Mathlib defines the basic number types one expects for doing math: `ℕ`, `ℤ`, `ℚ`, `ℝ`, `ℂ`.

Right off the bat this poses some awkwardness for mathematicians. Take the simple number 0 — what is its unique type? `ℕ` or `ℤ` or `ℚ`? Lean deals with this in a way familiar to Haskell programmers, but perhaps confusing to mathematicians at first. There is no object called just `0`. There is a numeric *literal* `0` (which is not a priori a mathematical object), which through the typeclass system any type can "claim" for itself, defining how to turn the literal into an object of that type.

If there is a specific type annotation like `(0 : ℚ)`, or one can be inferred from context, the zero literal is turned into the specific zero object of the type needed. To avoid confusion for newcomers, Lean has a default so that `#eval 0` results in `0 : ℕ`.

## Explicit coercions

Now that we have distinct objects like `(0 : ℕ)` and `(0 : ℚ)`, how do we capture the mathematical intuition that they are basically the same thing? Lean makes it easy to convert between them via the natural injection. Instead of naming this injection, you can write `(0 : ℕ) : ℚ` and Lean's typeclass mechanism finds and applies the right injection automatically. As a reminder that coercion is really a function — not the same `0` belonging to different sets — Lean renders `↑0`, the up arrow indicating an injection was applied.

Make sure to always hover over `0` or `↑` in the Lean infoview. It will show you which type `0` belongs to and the specific injection used.

## Subtypes

In math, you might say "let `n` be an *even* natural number" and move on. Implicitly in set theory terms, you mean `n` belongs to the set of even numbers `{0, 2, 4, ...}`. However, in type theory this is a problem: is `n` in `ℕ` (with some extra info) or in `{0, 2, 4, ...}`? An object can belong to only a single unambiguous type.

Lean resolves this by pairing each value with a proof that it satisfies some property. This is called a **subtype** and the syntax is `{n : ℕ // Even n}`, and an element is a pair — the value `x.val : ℕ` and a proof `x.prop : Even x.val`.

A key example is `Fin n`, the type of natural numbers less than `n`. It is defined as `{i : ℕ // i < n}` — a subtype of `ℕ`. When you write `i : Fin 5`, `i.val` is the underlying natural number and `i.prop` is a proof that `i.val < 5`.

Subtypes are distinct from their parent type. `(0 : ℕ)` and `(⟨0, by decide⟩ : Fin 5)` are different objects with different types. Lean provides coercions — if `i : Fin 5`, you can use `i` where a `ℕ` is expected and Lean inserts `↑i` — but this can still surprise newcomers when type mismatches appear.

Finally, note that subtypes are merely a convenience. One can always just work with the unpacked raw value plus the proposition - a function can ask for `(x : ℕ) (hxeven : Even x)`.

Now we can clarify the two coercion arrows you'll encounter:

- `↑x` coerces a **value** — e.g. `↑i` sends `i : Fin 5` to `ℕ`
- `↥` coerces something **into a type** — you'll see this later when we discuss `Set`, where `↥S` turns a set into its corresponding subtype

For now, `↑` is the one you'll see most: it injects an element into a larger type.

### Subtypes of subtypes

In set theory, you can take a subset of a subset without thinking twice: "let `S` be the even numbers, and let `T` be the even numbers less than 10." Both `S` and `T` are just sets of natural numbers. An element of `T` is an element of `S` is a natural number — no wrapping or unwrapping needed.

In Lean, subtypes nest. If `S = {n : ℕ // Even n}` and you want the elements less than 10, you get `{x : S // x.val < 10}`, which is `{x : {n : ℕ // Even n} // x.val < 10}`. An element of this type is a pair whose first component is itself a pair: `⟨⟨4, by decide⟩, by decide⟩`. To get back to the underlying natural number, you write `x.val.val`.

This compounds. Each layer of restriction adds another `.val` to unwrap and another proof to carry. Lean's coercions help — `↑x` will often get you back to `ℕ` — but in the infoview you'll see types like `↥↥S` and expressions like `↑↑x`, which can be disorienting.

The practical advice: avoid nesting subtypes when you can. Instead of building `{x : {n : ℕ // Even n} // x.val < 10}`, prefer a single subtype with a combined predicate: `{n : ℕ // Even n ∧ n < 10}`. This keeps elements flat — one `.val`, one `.prop` — and avoids the coercion chains. In set theory this distinction doesn't exist; in Lean it determines how painful your proofs are.

## Sets

With subtypes in hand, we can understand Lean's `Set`. Lean supports familiar set notation — you can write `({0, 1, 2} : Set ℕ)` or `{x : ℕ | x > 0}` and get an object of type `Set ℕ`. The literal `{0, 1, 2}` is polymorphic — it can produce a `Set ℚ` or `Set ℤ` depending on context. You can use `∈`, `∪`, `∩`, `⊆` and other set operations as expected.

When you need to treat a set as a type — to define a function on its elements, or quantify over them — Lean coerces `S` to the subtype `↥S`, which as we saw is `{x : T // x ∈ S}`. So you can write `(f : ↥S → ℝ)` to define a function on the elements of `S`.

One important thing to note: **sets are always relative to a type.** There is no bare `Set` in Lean, only `Set T` for some type `T`. A useful mental model: in Lean, "sets are subsets" — you always need an ambient type to even talk about them.

## Finsets and Fintypes

Under the hood, `Set T` is defined as `T → Prop` — just a membership predicate. You can state that `x ∈ S` and prove it, but you can't iterate over a `Set`, sum over it, or take its cardinality. So how do you do computation with sets?

Lean provides two separate mechanisms, and the distinction between them is one of the most confusing aspects for newcomers.

### Finset

`Finset T` is a *concrete, finite subset* of a type `T`. Unlike `Set`, a `Finset` carries actual data — it knows exactly which elements it contains, and membership is decidable (you can compute whether `x ∈ S`). This is what you need for:

- summing: `∑ x ∈ S, f x` (finite sums — no convergence proof needed)
- counting: `S.card` (returns a plain `ℕ`, not a cardinal)
- filtering: `S.filter (fun x => x > 3)` (requires the predicate to be decidable)

A subtle but important point: `∑ x ∈ S, f x` and `∑ x : S, f x` mean different things. With `∈`, the variable `x` has the ambient type (`x : T`) and the sum just restricts which elements are included. With `:`, Lean coerces `S` into a subtype `↥S`, so `x : ↥S` — a pair of a value and a membership proof. If `f : T → ℝ`, then `∑ x ∈ S, f x` works directly, but `∑ x : S, f x` fails because `x` has the wrong type. You'd need `f ↑x` to coerce back. In practice, `∈` is almost always what you want when summing over a `Finset`.

You can build a `Finset` with a literal like `({1, 3, 5} : Finset ℕ)`, or from `Finset.range n` (the numbers 0 to n-1), or by filtering another `Finset`.

Like `Set`, a `Finset` always lives relative to an ambient type — `Finset ℕ`, `Finset (Fin n)`, etc. And like `Set`, you can convert a `Finset` to a subtype with `↥S`. But unlike `Set`, you can actually compute with it.

### Fintype

`Fintype T` is a *typeclass* asserting that the type `T` has finitely many elements. It's not a subset — it's a property of the whole type. When `[Fintype T]` is in scope, `Finset.univ : Finset T` gives you a `Finset` containing every element of `T`.

This also lets you write `∑ x, f x` (sum over the whole type) without specifying a set — Lean knows there are finitely many `x` to sum over. Common `Fintype` instances include `Fin n`, `Bool`, and any product or subtype of finite types.

### The choice: Finset vs Fintype

When a textbook says "let `S` be a finite set," you have two encodings:

- `(S : Finset U)` — `S` is a finite subset of some ambient type `U`
- `[Fintype S]` — `S` *is* the type, and it's finite

Each has tradeoffs — we'll compare them in the example below.

### Converting between Set and Finset

You can convert a `Set` to a `Finset` with `.toFinset`, provided the predicate is decidable and the type is finite. For example, `{x : Fin n | x > 3}.toFinset` produces a `Finset (Fin n)`. Going the other way, every `Finset` can be viewed as a `Set` via coercion — if `S : Finset ℕ`, you can use `S` where a `Set ℕ` is expected.

## A full example

Here's a textbook exercise from Tao's *Analysis I* (Exercise 7.1.6):

> Let S be a finite set which is the disjoint union of E₁, ..., Eₙ. Then ∑(x ∈ S) f(x) = ∑(i=1..n) ∑(x ∈ Eᵢ) f(x).

One sentence. Perfectly clear to any math student. But formalizing it in Lean required choosing between the encodings we've discussed. Here's what I landed on:

```lean
theorem sum_union_disjoint {n : ℕ} {S : Type*} [Fintype S]
    (E : Fin n → Finset S)
    (disj : ∀ i j : Fin n, i ≠ j → Disjoint (E i) (E j))
    (cover : ∀ s : S, ∃ i, s ∈ E i)
    (f : S → ℝ) :
    ∑ s, f s = ∑ i, ∑ s ∈ E i, f s
```

Notice the choices and their consequences:

- **`S` is a type with `[Fintype S]`**, not a `Finset U`. This gives us the clean LHS: `∑ s, f s` — sum over every element of the type. With `S : Finset U` instead, the LHS would be `∑ s ∈ S, f s` and `f` would need to be `f : U → ℝ` — defined on the entire ambient type, not just `S`.
- **The index type is `Fin n`**, not `(i : ℕ)` with a hypothesis `i < n`. The textbook's "E₁, ..., Eₙ" becomes `E : Fin n → Finset S` — the bound is baked into the type rather than carried as a separate proof. And since `Fin n` has a `Fintype` instance, `∑ i` on the RHS just works.
- **`E i : Finset S`** — each piece of the partition is a `Finset`, so we can sum over it.
- **The partition is stated as hypotheses** — `disj` (pairwise disjoint) and `cover` (every element belongs to some `E i`). In the textbook, "S is the disjoint union" says this in one phrase. In Lean, since `S` is a standalone type, the relationship must be spelled out.
- **`f : S → ℝ` is a total function** — defined on all of `S`, not on a subtype. This avoids coercion pain. We restrict the summation range on the RHS with `∑ s ∈ E i`, not the function domain.

## Conclusion

Formalizing even a straightforward set-theoretic statement in Lean requires real choices. Many come down to whether properties are packaged into a subtype, stated explicitly as hypotheses, or carried implicitly via typeclasses. Usually any encoding works in terms of what you can prove — but the readability of the resulting proof can differ significantly.
