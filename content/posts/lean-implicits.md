---
title: "From Explicit to Implicit in Lean"
date: 2026-04-12T09:25:28-07:00
draft: true
---

Note: AI was used to edit this post. As a proof-of-human thought and input, I am also publishing the [original
draft](../lean-implicits-raw) which was written fully before asking AI to edit the post with me.

This post is aimed at Lean language beginners that are interested in writing proofs in Lean, but
still feel lost when reading Lean code.

A very simplified mental model of Lean is that at the core there are two systems:

1) a very fancy functional programming language. The objects here are `Nat`, `List Nat`, functions of those, etc

```lean
def n: Nat := 3
def l: List Nat := [1, 2, 3]
def f (n : Nat) := n + 1
```

The type system is fancy in the way that types have their own types (and so on recursively). But for the purposes
of this post we will just talk about the first layer `Type` (which is shorthand for `Type 0`)

```lean
#check Nat  --  Nat : Type
#check List Nat -- List Nat : Type
#check Nat -> Nat -- Nat → Nat : Type
```

Finally, parametric polymorphism is very pleasingly uniform with the rest of the system, meaning
we can write generic parameters using exactly the same syntax for declaration or application
(no <> or [] like Java or TypeScript.) So far this should be very familiar to people with exposure
to typed FP.

```lean
def append (a: Type) (x : a) (arr: List a) := arr ++ [x]
```

1) a system to state and manipulate logical propositions about the objects of 1). By convention these are
named `h*` which stands for hypothesis.

```lean
def n := 3
def hnpos : n > 2 := by decide
```

Again this is very uniform - system 2) uses the same keywords for defining and declaring types as 1).
Notably, the type of the proposition is allowed to depend on variables. In the example above, `>`
is a fancy way of writing `GT.gt n 2`. On the surface it is a generic type like `List a`. Note that `n`
is a variable of type `Nat`, unlike `a` which was of type `Type`. Allowing such dependency is why
we say Lean is based on `dependent types`.

Finally, if the logical proposition is the type, the object
of that type is a proof of the proposition, in this case `by decide`. Don't worry if you don't understand what it does, there
are many ways to prove that 3 is bigger than 2. If the proposition was not true like `n < 2` we can still state it,
but will have a very hard time proving, i.e. filling out the other side of `def h: n < 2 := sorry`.

Any statement from first order logic can be written in Lean, `n > 2`, `n = n`, `∀ n: Nat, ∃ m: Nat, m * 2 = n`, etc.
Again since all types have their own type, it is natural to ask what's the type of those statements - it is `Prop`.

```lean
#check n > 2 -- n > 2 : Prop
#check n = n -- n = n : Prop
#check ∀ n: Nat, ∃ m: Nat, m * 2 = n -- ∀ (n : Nat), ∃ m, m * 2 = n : Prop
```

Props can be manipulated using the usual logical ways - creating conjunctions, disjunctions, implications, iff, etc.

```lean
def and_comm (p q: Prop) (h : p ∧ q) : q ∧ p := ⟨h.2, h.1⟩
```

You might see the ones above stated with `theorem` not `def`, which is more idiomatic and ergonomic, but the differences are minor.
The keyword `example` is like an anonymous `theorem` and `lemma` is a synonym for `theorem`.

So objects of `Type` and `Prop` are the core ingredients of any Lean proof. Usually, one passes some number of objects, props about
those and tries to prove some new prop.

```lean
example (a b c n : Nat) (ha : a > 1) (hb : b > 1) (hc : c > 1) (hn : n > 2) : a ^ n + b ^ n ≠ c ^ n := by sorry
```

## Ergonomics

I hope the above exposition convinces you that there is a minimal core to Lean. I like to think of it
as the explicit model. It is nice and simple but sadly has bad ergonomics once you start doing non-trivial
proofs. The argument lists become long and unwieldy, so a lot of the complexity in Lean comes from adding
different systems of implicits, inference, etc, to make it more ergonomic to write proofs in Lean.

The situation reminds me of this quote from [Wadler's monad paper](https://homepages.inf.ed.ac.uk/wadler/papers/marktoberdorf/baastad.pdf)

> Pure functional languages have this advantage: all flow of data is made explicit.
> And this disadvantage: sometimes it is painfully explicit.

In the rest of the post I will go over some of the niceties Lean overlays on this simple model
of types and props. I will use the following very contrived example. Suspend disbelief and
imagine a much more complex proof as we will be doing various refactoring to make it "simpler"
while it already is quite simple:

```lean
lemma t1 (n : Nat) (hn: n > 0) : n + 1 = 1 + n := by sorry
lemma t2 (n : Nat) (hn: n > 0) : n + n = 2 * n := by sorry
lemma t3 (n : Nat) (hn: n + 1 = 1 + n) : n + 2 = 2 + n := by sorry

theorem t4 (n : Nat) (hn : n > 0) : True := by
  have h1 := t1 n hn
  have h2 := t2 n hn
  have h3 := t3 n h1
  trivial
```

### Basic inference

One can ask Lean to infer an expression by replacing it with `_`. Depending on the context it might
succeed or fail. In the example below `n` can be inferred from the `h*`.

```lean
theorem t4 (n : Nat) (hn : n > 0) : True := by
  have h1 := t1 _ hn
  have h2 := t2 _ hn
  have h3 := t3 _ h1
  trivial
```

### Inferred Arguments declaration

Instead of asking every user of the theorem `t1` to write `_` the author of the theorem can flip the script
and declare the argument as always inferable, since they know that `n` is present in the dependent type of `hn`.

To do that we use `{}` curly brackets.

```lean
lemma t1 {n : Nat} (hn: n > 0) : n + 1 = 1 + n := by sorry
lemma t2 {n : Nat} (hn: n > 0) : n + n = 2 * n := by sorry
lemma t3 {n : Nat} (hn: n + 1 = 1 + n) : n + 2 = 2 + n := by sorry

theorem t4 (n : Nat) (hn : n > 0) : True := by
  have h1 := t1 hn
  have h2 := t2 hn
  have h3 := t3 h1
  trivial
```

If the inference gets in the way one can always explicitly overwrite it with `t1 (n := ...) ...`.

### Variable keyword

The next step for ergonomics is noticing that `{n : Nat}` repeats in every `t*` theorem. Lean
allows you to declare a variable for all declarations in a namespace using the `variable` keyword.

```lean
variable {n: Nat}

lemma t1 (hn: n > 0) : n + 1 = 1 + n := by sorry
lemma t2 (hn: n > 0) : n + n = 2 * n := by sorry
lemma t3 (hn: n + 1 = 1 + n) : n + 2 = 2 + n := by sorry

theorem t4 (n : Nat) (hn : n > 0) : True := by
  have h1 := t1 hn
  have h2 := t2 hn
  have h3 := t3 h1
  trivial
```

Note, `t4` can keep an explicit `n` argument or not, there is no clash with the `variable` declaration.

### Structure packing

One might notice that we keep talking about the natural numbers bigger than zero, i.e. the positive
natural numbers. Naturally one might want to create a new type to define those. This should be
a very natural refactoring to software people.

Lean has the `structure` keyword for that, and we can just combine the underlying `carrier` object
in Nat with the prop that it is bigger than zero.

```lean
structure PNat where
  val : ℕ
  hpos : val > 0

variable (n : PNat)

lemma t1 : n.val + 1 = 1 + n.val := by sorry
lemma t2 : n.val + n.val = 2 * n.val := by sorry
lemma t3 (hn: n.val + 1 = 1 + n.val) : n.val + 2 = 2 + n.val := by sorry

theorem t4 (n : PNat): True := by
  have h1 := t1 n
  have h2 := t2 n
  have h3 := t3 n h1
  trivial
```

Note, that we had to make our variable explicit `(n : PNat)` instead of implicit `{n : PNat}` because we trimmed the
argument list so much there is nothing to infer it from.

### Operator overloading

You might have noticed that the structure refactoring forced us to add `.val` everywhere, making it hardly look
any cleaner. As it turns out Lean has a form of operator overloading that allows us to define `+` and `*` on `PNat` which
does the `.val` under the hood.

```lean
structure PNat where
  val : ℕ
  hpos : val > 0

instance PNatAdd : Add PNat where
  add a b := ⟨a.val + b.val, add_pos a.hpos b.hpos⟩

instance PNatMul : Mul PNat where
  mul a b := ⟨a.val * b.val, mul_pos a.hpos b.hpos⟩

def one : PNat := ⟨1, by norm_num⟩
def two : PNat := ⟨2, by norm_num⟩

variable (n : PNat)

lemma t1 : n + one = one + n := by sorry
lemma t2 : n + n = two * n := by sorry
lemma t3 (hn: n + one = one + n) : n + two = two + n := by sorry

theorem t4 (n : PNat): True := by
  have h1 := t1 n
  have h2 := t2 n
  have h3 := t3 n h1
  trivial
```

So the actual proof got noticeably simpler, but we had to add more and more infrastructure, including proofs that
adding or multiplying two positive numbers gives us back a positive number (turns out those were already in Lean).
Finally, we had to move away from the literals `1` and `2` and create custom `PNat` vars `one` and `two`. Lean has
overloading mechanisms to allow us to keep the literals, but that would take us too far.

### Custom type classes

Finally, you might have noticed the operator overloading of `+` and `*` was done through the type class system, using
the `instance` keyword. As it turns out that system is extremely flexible and open to customization.

At its core it is another type of "implicit" resolution, where a piece of Lean data - type or prop is attached
to some other one. One mental picture is of a big global hashmap. Then whenever we need that extra data, Lean can
run a query to fetch what is needed. One can view it as a functional type of dependency injection (if coming from
a more OOP world) and very similar to Haskell's type classes (if coming from FP, but note there are some differences).

Let's use a custom type class to attach implicitly the proof that `n` is positive to a given `n`. First we extend
the type class system by defining a new type class with the `class` keyword. The key to use in the search is a
given `a : Nat` and the returned data is `pos : a > 0`.

```lean
class IsPos (a: Nat) where
  pos : a > 0
```

Now every time we need a positive proof for some `n` we pull it with `[IsPos n]` and every next function
invocation that needs it also gets it inferred from the one present in the local context.
Of course, we can use `[IsPos n]` with the variable keyword.

```lean
variable (n: Nat) [IsPos n]

lemma t1 : n + 1 = 1 + n := by sorry
lemma t2 : n + n = 2 * n := by sorry
lemma t3 (hn: n + 1 = 1 + n) : n + 2 = 2 + n := by sorry

theorem t4 (n : Nat) [IsPos n]: True := by
  have h1 := t1 n
  have h2 := t2 n
  have h3 := t3 n h1
  trivial
```

Finally, while the `[IsPos n]` instance stays anonymous and it is implicitly passed between function calls, we
can always give it a name in a local variable with `have hnpos : IsPos n := by infer_instance` and now
we can treat `hnpos` as a structure and get `hnpos.pos` for the actual `Prop`.

Also, if we want to explicitly put a proof that n is positive in the type system, we can do so like this.
Note the instance can be named before the `:` — otherwise it gets an auto-generated name like `instIsPosA`.
The name doesn't need to be used most of the time, but it is useful for debugging.

```lean
def a := 3
instance aIsPos : IsPos a where
  pos := by decide

def h := t4 a
```

When typeclass resolution fails, the error messages can be cryptic. You can ask Lean to show
you exactly what it searched for and where it got stuck by enabling the synthesis trace. This is
invaluable for debugging "failed to synthesize instance" errors.

```lean
set_option trace.Meta.synthInstance true in
def h := t4 a
```

In summary, the type class system is another way of implicitly passing data around, but it has more
of a global flavor - the `IsPos a` instance can be pulled out of the global hashmap with `[IsPos a]`
whenever one has access to `a`.

## Learn more

Hopefully, this contrived example has given you a more concrete understanding of how the various
language features exist to improve the ergonomics of writing complicated proofs, by showing you how they
are used on a very minimal example (doesn't even need Mathlib). I encourage you to paste these
examples in the [Lean playground](https://live.lean-lang.org/) and try to extend or break them.

You can learn a lot more about implicits, variable declarations and type classes
by reading the [official guide](https://leanprover.github.io/theorem_proving_in_lean4/) and the
[language reference](https://lean-lang.org/doc/reference/latest/).
