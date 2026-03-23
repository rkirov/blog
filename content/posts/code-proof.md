---
title: "Code Proven to Work - The Math Way"
date: 2026-03-22T17:25:24-07:00
draft: false
---

This post is aimed at a general programmer and no prior knowledge of math or CS is assumed.

I got nerd-sniped to write this after reading Simon's excellent post [Code proven to work](https://simonwillison.net/2025/Dec/18/code-proven-to-work/).
As someone who works in the software industry that is adapting to using AI tools, the post was very timely and very well put. I agree with everything said, but I just can't resist being that guy - bringing up the "you keep using that word" meme.

As you might have already guessed the word is "proof", and using it in the context of manual and automated testing is mildly off-putting. When I see "proof" I think first of mathematical proof - like the proof of the infinitude of primes.

You might think that mathematical proof is not applicable to software engineering, but that is not quite true. As it turns out it has been known and possible for a long time, but the tools and techniques are mostly unknown outside academia and high-stakes coding like security.

## Example in Lean

Following the `show-don't-tell` principle I will directly show you an example in a language that can write both code and proofs about the code - [Lean](https://lean-lang.org/).

Here is quick sort written in Lean:

```lean
partial def qsort : List Int -> List Int 
  | [] => []
  | x :: xs =>
    let lesser := xs.filter (fun y => y < x)
    let greater_eq := xs.filter (fun y => y >= x)
    qsort lesser ++ [x] ++ qsort greater_eq
```

[Try it in the Lean playground](https://live.lean-lang.org/#codez=A4QwTgLgliA2AEATApgM3gRwM4HtLwC54AZKLCeASQDsKBaAPhLIpooCh54AfeAbQC68ALxNBnHvAAehIlKwiGErrGQVVWLMjCFh0rADpUUWBG3wAFKgCu1eAE9FD+AB5pASmXxVFAOZhkEDMwAH1kDF19IxNgyxs7R1FnBj0pTy4ubDx1ZE1zAGp8/ikhQsxcfH9A4LCMIA)

Here is the TS version of the exact same function logic:

```ts
function qsort(xs: number[]): number[] {
  if (xs.length === 0) return [];
  const [x, ...rest] = xs;
  const lesser = rest.filter(y => y < x);
  const greaterEq = rest.filter(y => y >= x);
  return [...qsort(lesser), x, ...qsort(greaterEq)];
}
```

But if you have written a functional language before you should be able to spot what the algorithm does directly in Lean (and presumably you remember the gist of the algorithm).

If you wrote this in a regular language the next thing you would do is to add tests. Unless you are using a fancier framework like property testing, you will pass some lists like [], [1,2,3], [3,2,1] and check that each comes back sorted.

But this is Lean, a language that has proofs as first class objects and we can directly prove properties of this function without writing a single test. Note this function is typed and type-checks but all of this goes beyond types.

## Proving termination

First, we will prove that this function doesn't go into an infinite loop, always a possibility with recursion. Informally, you will likely argue like this - the initial call starts with a list of `n` elements, since we peel off one, both `lesser` and `greaterEq` would be elements of at most `n - 1`. Finally the empty array terminates, so by induction the function terminates for all lists.

As it turns out this proof can be written in code too. Terminating functions are so important in Lean that they are required by default and the `partial` keyword turns off the termination checker. So we remove it and provide Lean with the proof as follows:

```lean
def qsort : List Int -> List Int 
  | [] => []
  | x :: xs =>
    let lesser := xs.filter (fun y => y < x)
    let greater_eq := xs.filter (fun y => y >= x)
    qsort lesser ++ [x] ++ qsort greater_eq
termination_by x => x.length
decreasing_by
  all_goals
    simp
    calc (List.filter _ xs.attach).length
        ≤ xs.attach.length := List.length_filter_le _ _
      _ = xs.length := List.length_attach
      _ < xs.length + 1 := Nat.lt_succ_of_le (Nat.le_refl _)

```

[Try it in the Lean playground](https://live.lean-lang.org/#codez=CYUwZgBAjgzg9gJwC4QFwQDIEsYoJIB2KAtAHyY75EQBQEEAPhANoC6EAvOW3YxAB5p0/GJ1K96AGxAppMGCARoOAmADowWSUkUQAFGACuBCAE8xZiAB4BASgkRpKAOYIQAQx0IA+iCjLVDS0vfSMTcy5LUhV+e3p6WERZEHldAGo0ln52DOh4ZAhXDy9fKBovAFssAk8sOAJvACNzQUj+NWkCZyQACxpQAGM3dxhq5ybTXndJSW9nOGmYB1GKgAcHAemB/WxcIO1db0DPJHcBntsOkC7eh3iIQBMiY6RT86ubnoDdpHfunu9NAcfNIIEdvHdQZxAp0/l9KL9et4Tmc+vd6EcbCIEZ9MgBGAIAOU8HSQ3hghgGA28cDA3hBeiJP2k3jcYEkoPs5R6IEQIAqeSS3gGcGMKD0IjQFFwEEISFs+ncktl8tQDj0iQKIkuwtFEEVMXUOuoitQKmaEHyCEmXJ5bn5GtJ+R0wH0EvQ3xlRBVtHugAAifRYCAAK0lADFqvoHapLjDerYADQQQNPEOAJMJI07o2pnDIk48M0kszmUCHTRBzZbTEA)

The extra lines are `termination_by` which defines a function to turn the input into a natural number that we can do induction on, in our case naturally we pick the length. And `decreasing_by` is the proof that if the input is of length `n` the subsequent recursive calls are of arrays of smaller length. I won't go into how proofs are written and checked in Lean, but roughly we put together two other known proofs:

- `Nat.lt_succ_of_le` - a mouthful way of saying `n < n + 1`. Yes, even basic facts like this are theorems with names that we need to invoke like functions.
- `List.length_filter_le` - the fact that a filter of a list produces a list of length less than or equal to the original's length.

The rest is some massaging of Lean technicalities. Yes, you should also be getting a feel for why this is not widely adopted in industry - it's a lot to ask of programmers to write.

## Proving correctness

Now that we proved that function terminates, we want to show that the output is indeed sorted. There are two parts to that:

1) the count of every possible number should be the same in the input and the output
1) for any two indices i, j of the output, if i <= j, then o[i] <= o[j].

In Lean we write those statements as:

```lean
theorem qsort_count (xs : List Int) (a : Int) :
    (qsort xs).count a = xs.count a

theorem qsort_sorted (xs : List Int) : 
    ∀ (i j : Fin (qsort xs).length), i ≤ j → (qsort xs).get i ≤ (qsort xs).get j
```

[Try it in the Lean playground](https://live.lean-lang.org/#codez=C4Cwpg9gTmC2AEBHAztYB9AxhArgO2HgAoAPZeALngBkBLZQgSQIEpiBDS+Z4NigKHhDiKNPDIsAdNnyFOAXnHJpuAvHb9+oSDASioGVAbAATYmS50G3Vl0HDAAETFa8AFZcAYrTwijhCZIANmB4AOagLAA08C6AJkRu8IBJhL5iAaFghHEpBkpS6YSuQA)

Once we know `Fin n` is `[0, 1, ..., n - 1]` (`range` in Python) this should be somewhat readable to all. Conceptually the proofs are not hard:

1) We show that every element has one of three fates: it is either the pivot x, it ends up in `lesser`, or in `greater_eq`. Nothing gets added or removed. We use induction for the inner calls.
1) By case analysis depending on whether `o[i]` and `o[j]` are in `lesser`, `x`, or `greater_eq`. Use induction if both are in `lesser` or in `greater_eq`.

That said the actual proofs as written in Lean are quite verbose because of keeping track of all the details, see appendix.

To reiterate, after coming up with these two properties and proving them, in the same language (literally in the same .lean file) we don't need to write a single test or come up with any test case, or wonder whether we forgot some edge cases to test. The implementation is proven to terminate and produce a correct result for *all* possible inputs (that is an infinite collection.)

## Where does that leave us

I wanted to show that it is possible to *prove* your code is correct, in the truly mathematical sense. Lean is just the language for doing that that is gaining the most popularity, but there are other possibilities.

Also, it should be clear why you haven't seen this in any industry software project - the ergonomics are not great. Even if one swallows the cost of a new language, a whole new library ecosystem, just so one can prove things about their code, the proofs themselves are very long and laborious to write. Programmers don't want to become mathematicians.

However, as I am writing this there is an AI revolution underway and any argument based on human ergonomics is suspect. The ergonomics of AI-written code are very different. In fact, I didn't write the proofs in the appendix either, Claude did, but I know enough Lean to verify the statements it proved are the ones I needed.

So could it be that in the AI-writing-code future, programming languages like Lean will have a bigger role?

I invite you to take a bit of time to learn Lean: [Functional Programming in Lean](https://lean-lang.org/functional_programming_in_lean/). And next time you are vibe coding something, maybe try asking Claude to write it in Lean and prove the correctness. The results might surprise you - including the token usage as Claude wrestles with Lean (at least it is not your time.)

## Appendix: full proof

Written by Claude. I know enough Lean to read it line by line, but would struggle to write it from scratch. [Try it in the Lean playground](https://live.lean-lang.org/#codez=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).

```lean
import Mathlib.Data.List.Pairwise

def qsort : List Int -> List Int
  | [] => []
  | x :: xs =>
    let lesser := xs.filter (fun y => y < x)
    let greater_eq := xs.filter (fun y => y >= x)
    qsort lesser ++ [x] ++ qsort greater_eq
termination_by x => x.length
decreasing_by
  all_goals simp_wf
  all_goals exact Nat.lt_succ_of_le (Nat.le_trans (List.length_filter_le _ _) (Nat.le_of_eq List.length_attach))

-- Correctness part 1: qsort preserves element counts
theorem count_filter_partition (tail : List Int) (p : Int → Bool) (a : Int) :
    (tail.filter p).count a + (tail.filter (fun y => !p y)).count a = tail.count a := by
  induction tail with
  | nil => simp
  | cons x xs ih =>
    simp only [List.filter, List.count_cons]
    split <;> simp_all [List.count_cons] <;> split <;> omega

theorem qsort_count (xs : List Int) (a : Int) :
    (qsort xs).count a = xs.count a := by
  match xs with
  | [] => simp [qsort.eq_def]
  | pivot :: tail =>
    rw [qsort.eq_def]; simp only
    rw [List.count_append, List.count_append,
        qsort_count (tail.filter (fun y => decide (y < pivot))) a,
        qsort_count (tail.filter (fun y => decide (y ≥ pivot))) a]
    simp only [List.count_cons, List.count_nil]
    have h := count_filter_partition tail (fun y => decide (y < pivot)) a
    have : (fun y => !decide (y < pivot)) = (fun y => decide (y ≥ pivot)) := by
      ext y; cases hy : decide (y < pivot) <;> simp_all [Int.lt_iff_add_one_le]
    rw [this] at h
    omega
termination_by xs.length
decreasing_by all_goals simp_wf; exact Nat.lt_succ_of_le (List.length_filter_le _ _)

-- Helper: membership version (derived from count, used in sortedness proof)
theorem qsort_mem (a : Int) (xs : List Int) : a ∈ qsort xs ↔ a ∈ xs := by
  match xs with
  | [] => simp [qsort.eq_def]
  | pivot :: tail =>
    rw [qsort.eq_def]
    simp only [List.mem_append, List.mem_cons, List.not_mem_nil, or_false]
    rw [qsort_mem a (tail.filter (fun y => decide (y < pivot))),
        qsort_mem a (tail.filter (fun y => decide (y ≥ pivot)))]
    simp only [List.mem_filter]
    constructor
    · rintro ((⟨h, _⟩ | rfl) | ⟨h, _⟩)
      all_goals simp_all
    · rintro (rfl | h)
      · left; right; rfl
      · by_cases hlt : a < pivot
        · left; left; exact ⟨h, by simp [hlt]⟩
        · right; exact ⟨h, by simp [Int.not_lt.mp hlt]⟩
termination_by xs.length
decreasing_by all_goals simp_wf; exact Nat.lt_succ_of_le (List.length_filter_le _ _)

-- Correctness part 2: qsort produces a sorted list
theorem qsort_pairwise (xs : List Int) : List.Pairwise (· ≤ ·) (qsort xs) := by
  match xs with
  | [] => simp [qsort.eq_def]
  | pivot :: tail =>
    rw [qsort.eq_def]; simp only
    rw [List.pairwise_append]
    refine ⟨?_, qsort_pairwise _, ?_⟩
    · rw [List.pairwise_append]
      refine ⟨qsort_pairwise _, List.pairwise_singleton _ _, ?_⟩
      intro a ha b hb
      simp only [List.mem_singleton] at hb; subst hb
      have := (qsort_mem a _).mp ha
      rw [List.mem_filter] at this; simp at this
      exact Int.le_of_lt this.2
    · intro a ha b hb
      have hb' := (qsort_mem b _).mp hb
      rw [List.mem_filter] at hb'; simp at hb'
      rcases List.mem_append.mp ha with ha' | ha'
      · have ha'' := (qsort_mem a _).mp ha'
        rw [List.mem_filter] at ha''; simp at ha''
        exact Int.le_trans (Int.le_of_lt ha''.2) hb'.2
      · simp only [List.mem_singleton] at ha'; subst ha'
        exact hb'.2
termination_by xs.length
decreasing_by all_goals simp_wf; exact Nat.lt_succ_of_le (List.length_filter_le _ _)

theorem qsort_sorted (xs : List Int) :
    ∀ (i j : Fin (qsort xs).length), i ≤ j → (qsort xs).get i ≤ (qsort xs).get j := by
  intro ⟨i, hi⟩ ⟨j, hj⟩ hij
  simp only [List.get_eq_getElem]
  rcases Nat.eq_or_lt_of_le hij with rfl | hlt
  · exact Int.le_refl _
  · exact List.pairwise_iff_getElem.mp (qsort_pairwise xs) i j hi hj hlt
```
