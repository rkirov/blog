+++
title = "Incremental Computation (part 3)"
date = 2020-05-19
images = []
tags = ['incremental computation']
categories = ['programming']
draft = false
+++

## Why TypeScript?

Maybe you wondering why did I use TypeScript for this post. As it happens I
have been working in the frond-end community in the last 8 years. My interest
in incremental computation started by observing the similarities between some
of the work I have done inside Angular's "change detection" algorithms, and
work I have done around integrating TypeScript's compiler in (Google's build
system)[https://bazel.build/].

## UI programming 

Why is incremental computation naturally appearing in UIs? As the user is
interacting with an UI they are providing new inputs. Usually, these inputs are
small compared with the initial input to the UI. On the other side the
producing the DOM is the quintessential "expensive" computation. If it redone
on each user input the UI will be unusable. So all UI frameworks attempt to
solve the incremental computation problem, struggling with the fact that JS has
no support for incremental computation. To make matters worse as we have seen
incrementality is easier in a functional language, but JS is not well suited to
that paradigm (which doesn't stop people from trying).

The following is a quick list of the popular JS frameworks and how much
incremental computation they support or not:

- Classic React - Incrementality is relegated to purely at the VDOM to DOM
  layer. This is similar to the classic on-line combinatorial algorithms. When
  the data structure is fixed a custom solution can be implemented, without the
  need for a general incremental computation framework. The user can treat the
  system as completely non-incremental. This is definitely a simpler conceptual
  model, but it is not the most efficient thing to do. The lack of
  incrementality outside VDOM to DOM was noticed as performance issue by the
  community. [React is not
  reactive](https://gist.github.com/sw-yx/9bf1fad03185613a4c19ef5352d90a09) is
  a great summary of the upside and downsides to moving to a fully reactive
  (read incremental) react.
- React Hooks - Moving from ad-hoc solutions like manually writing
  `shouldUpdate` to hooks like `useMemo`, `useEffect` which have better
  incrementality support.  However, as we have seen so far full incremental
  computation is more intricate than just memoizing. AFAIKT, there is no
  support for dynamic tracking of dependencies with the built-in hooks. The
  classic expression `c ? x : y` will have to be written as dependent on `[c,
  x, y]` in `useEffect`. 
- Angular - While React recomputes the DOM (VDOM to be more precise) directly,
  Angular's recomputation of the DOM can be seen as a reverse mode where the
  template is computed starting from its dynamic parts - the template
  expressions which pull all the needed data. This computation is also not
  incremental. Because Angular does ship with RxJS interested users are
  exploring adding more incrementality, because as we saw it can be used as
  building block for incrementality -
  [ngrx/component](https://ngrx.io/guide/component).  Separately, Angular
  supports diffing of arrays for `ng-for`, which can be seen as an ad-hoc
  incremental computation of type `T[] => Node[]`.
- Vue.js - Similar to Angular supports diffing for arrays and propagating
  diffs, but no other finer grained incrementality. 
- Lit HTML - Does not have any built-in incrementality, but because the
  computation is directly expressed in terms on the value inputs, recomputation
  is simpler.
- Cycle.js - the closest to a full solution for incrementity. The model data is
  carried through RxJS observables, which as we have seen naturally allows for
  incremental computation.
- Svelte - With the introduction of the `$:` reactivity marker Svelte is adding
  language level incrementality (called reactivity by the authors) to JS.
  Instead of embedded DSL fashion like cycle.js, which makes the incremental
  objects reified and visible to the user, in svelte they are generated allowing
  one to write in what seems like a incremental version of JavaScript. That
  said Svelte does not support dynamic incremental computation. This can
  be seen by the usual test of `$: a ? b : c` which will be updated on both
  `b` and `c` in Svelte.

As this very quick survey shows in the modern JS framework space, there is
general attempt to find the most ergonomic and practical way to expose some
sort of incremental computation on top of an imperative language - JavaScript.
And to be absolutely clear this is not a comparison of better vs worst of JS
framework. The more incremental solutions can turn out to be unergonomic or add
too high of an overhead (similar to parallel computation). This was merely an
attempt to put them on equal grounding against a common model of ideal (and
likely impractical) incrementity, so that we can understand the tradeoffs and
the landscape better.

If one moves further from native JS, an into transpiled-to-JS land, supporting
full incremental computation becomes much more manageable as [this blog by
Yaron
Minsky]([https://blog.janestreet.com/self-adjusting-dom-and-diffable-data/) and
[talk](https://www.youtube.com/watch?v=R3xX37RGJKE) show.

## Pragmatics

Given what you have seen so far, maybe you are excited to start using
incremental computation in your project. I have to absolutely honest, if it
wasn't very clear, there are too many unknowns around the pragmatics of
incremental computation to recommend it (at least in the JS space). This is
an non-exhaustive list of pragmatic questions around incremental computation:

- what is the runtime overhead of creating closures around each part of the
  computation. 
- how to not run in to issues around garbage collection and memory leaks around
  holding the incremental computation closures.
- using native data-structures and algorithms with differential solutions, vs
  pure incremental computation.
- in language like JS that has mutation, how does one prevent abuse of the 
  core invariants (all reads have to go through `read` method etc).
- is it better to support incremental computation as a DSL on top of an
  existing language vs writing in a simpler looking language that is natively
  incremental (transpiling to the code we have seen). 

In any case, these can lead to some great conversations, so if you think you
have an answer reach out.

## Conclusion

So what was this all about? To borrow a lovely analogy from Edward Kmett's talk
[Stop threading water - Learning to
learn](https://www.youtube.com/watch?v=j0XmixCsWjs) - research and development
(or academia and industry) are stuck in maze trying to reach each other.
Research has some lovely solutions, but doesn't understand well the real-world
problems, while development has excellent grasp on the real-world problems,
but lacks the principled thought-through solutions. Left on their own devices
each tries to traverse the maze of ideas to get through the other end.

But if they collaborate they can meet in the middle and shorten the search.
The picture that Kmett uses in his talk is something like this.

![The maze](/maze.png)

Looking around the space of UI programming for the web, I think we are getting
close to the meeting point of the research of incremental computation and the
industry need for efficiently updating UIs. 

My hope for this blog post is to shine some light on both loose ends so that
folks on both sides can be more efficient in connecting the problems and the
solutions.

## References

1. [Mokhov, Mitchell, Jones, Build Systems à la Carte, Proc. ACM Program, 2018.](https://www.microsoft.com/en-us/research/uploads/prod/2018/03/build-systems.pdf)
1. [Magnus Carlsson. Monads for Incremental Computing. ICFP '02 Proceedings of the seventh ACM SIGPLAN international conference on Functional programming](https://dl.acm.org/doi/abs/10.1145/581478.581482)
1. [U. Acar, G. Blelloch, and R. Harper. Adaptive functional programming. In Principles of Programming Languages (POPL02), Portland, Oregon, January 2002. ACM](https://www.cs.cmu.edu/~guyb/papers/popl02.pdf)
1. [Rust Salsa framework](https://crates.io/crates/salsa)
1. [SSA is Functional Programming](https://www.cs.princeton.edu/~appel/papers/ssafun.pdf)
1. https://www.jantar.org/papers/chakravarty03perspective.pdf
1. [Introducing Incremental](https://blog.janestreet.com/introducing-incremental/)
1. [Incremental computation in Swift](https://github.com/chriseidhof/incremental-simplified)
1. [Analysis and Caching of Dependencies](http://citeseerx.ist.psu.edu/viewdoc/download?doi=10.1.1.32.1230&rep=rep1&type=pdf)
1. [A Theory of Changes for Higher-Order Languages - Incrementalizing λ-Calculi by Static Differentiation](https://arxiv.org/abs/1312.0658) 
1. [Incremental Computation with Names](https://arxiv.org/pdf/1503.07792.pdf)
1. [Edward Kmett - "Stop Threading Water: Learning to Learn"](https://www.youtube.com/watch?v=j0XmixCsWjs)
1. [Incremental Computation via Function Caching](https://dl.acm.org/doi/pdf/10.1145/75277.75305?download=true)
1. [Typed incremental computation with names](https://arxiv.org/pdf/1808.07826.pdf)
