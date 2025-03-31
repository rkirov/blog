---
title: "Thoughts on Signals in the JavaScript Ecosystem"
date: 2025-03-30T08:05:09-07:00
draft: false
---

Around 5-8 years ago when working on Angular and TypeScript rules in Bazel, I got
very interested in exploring the core ideas of incremental computation, which seemed 
like the right way to abstractly (but still precisely) describe the similarities between
build systems and UI reactivity. As an aside, incremental dataflow is a third view into
the same concept, but I am least familiar with it.

I wrote the following series of [posts](https://rkirov.github.io/posts/incremental_computation/) 
to try to pull all the different threads together. It was useful for me to put something to paper,
 but at the end I failed in painting a picture as clear as I wanted.

To my surprise, after I left the JS framework world, fine-grained reactivity using signals became very popular. 
Currently, the following frameworks have implemented them - Angular, Ember, Knockout, Preact, Solid, Svelte, Vue and 
likely many more. There is even a ECMAScript proposal to add them to the core language [Stage 1](https://github.com/tc39/proposal-signals).

In a classic naming irony, the word "signal" likely originates from [Conal Elliott's functional reactive programming (FRP) work](http://conal.net/papers/icfp97/), but
signals are not FRP according the precise semantics of that work. What JS ecosystem calls signals is much more closely connected to the incremental or self-adjusting computation (SAC) from [Ucar, et.al. work](https://www.cs.cmu.edu/~rwh/students/acar.pdf) that I was learning about in years prior.
(Aside, if you already know FRP you can learn more about the similarities and differences with SAC in this Jane Street [blog post](https://blog.janestreet.com/breaking-down-frp/).)

Speaking of words, "incremental" and "self-adjusting" are not used in the JS ecosystem, but rather "reactivity", undoubtedly
due to React's popularity, which ironically is not [fully reactive](https://braythwayt.com/posterous/2012/11/01/programming-is-a-pop-culture.html).

It remains to be seen if JS signals just the latest fad ([programming is pop culture](https://braythwayt.com/posterous/2012/11/01/programming-is-a-pop-culture.html) after all) or truly a better approach to UI reactivity that would outlast the latest framework churn, but in this post I want to emphasize again the connection with SAC that transcends language of implementation and use-case.

## Differences between SAC implementations and Signals  

There are three notable implementations of self-adjusting computation:

1. original OCaml implemenation from the Ucar et.al. I haven't looked into that too much, other than reading the original papers.
1. haskell implementation with monads based on the [paper by Magnus Carlsson.](https://www.researchgate.net/publication/221241270_Monads_for_incremental_computing) I understood this enough to rewrite it in [TS.](https://github.com/rkirov/adapt-comp)
1. Jane Streets modern [Incremental.](https://github.com/janestreet/incremental) The connection with UIs is understood and utilized here first [blog.](https://blog.janestreet.com/introducing-incremental/)

Here is an example of SAC directly from Jane Street's Incremental [blog post](https://blog.janestreet.com/introducing-incremental/).
Don't worry if you can't read OCaml for now.

```ocaml
let width_v  = Var.create 3.
let depth_v  = Var.create 5.
let height_v = Var.create 4.

let width  = Var.watch width_v
let depth  = Var.watch depth_v
let height = Var.watch height_v

let base_area =
  Inc.map2 width depth ~f:( *. )
let volume =
  Inc.map2 base_area height ~f:( *. )

let base_area_obs = Inc.observe base_area
let volume_obs    = Inc.observe volume

let () =
  let v = Inc.Observer.value_exn in
  let display s =
    printf "%20s: base area: %F; volume: %F\n"
      s (v base_area_obs) (v volume_obs)
  in
  Inc.stabilize ();
  display "1st stabilize";
  Var.set height_v 10.;
  display "after set height";
  Inc.stabilize ();
  display "2nd stabilize"
```

producing the output:

```ocaml
1st stabilize: base area: 25.; volume: 125.
    after set height: base area: 25.; volume: 125.
       2nd stabilize: base area: 25.; volume: 250.
```

In JS this would be

```javascript
let width_v  = var(3);
let depth_v  = var(5);
let height_v = var(4);

let width  = width_v.watch();
let depth  = depth_v.watch();
let height = height_v.watch();

let base_area = Inc.map2(width, depth, (x, y) => x * y);
let volume = Inc.map2(base_area, height, (x, y) => x * y);

let base_area_obs = Inc.observe(base_area);
let volume_obs    = Inc.observe(volume);

function main() {
  let v = Inc.Observer.value_exn;
  let display = (s) =>
    console.log(`${s}: base area: ${v(base_area_obs)}; volume: ${v(volume_obs)}\n`);
  Inc.stabilize();
  display("1st stabilize");
  height_v.set(10);
  display("after set height");
  Inc.stabilize();
  display("2nd stabilize");
}
```

It still looks very far from signals. Conceptually, it is because SAC systems are built by four types of objects:

- Vars - think of them as inputs into the computation.
- Incrementals - the in-between values created with `watch` and transformed with `Inc.map2`. These were called `Modifiable` in previous SAC implementations.
- Observables - think of them as the outputs of the computation. These are called `Changable` in previous SAC implemenations.
- Inc - the incremental system reified outside any of the 3 objects above. This is called `Adaptive` is previous SAC implemenations.

However, the JS ecosystem deeply cares about simplicity so a system with two core primitives will almost always
be preferred to one with four. This is a cultural value of the ecosystem due to large number
of newcomers to programming entering through front-end technologies (I was one of those myself 15 years ago.)

First we remove the explicitly reified system `Inc`.
Instead of `Inc.stabalize`, the observable changes are recomputed on-demand exactly when an Observable is read. This is what is called `pull-based`. Note, that
in theory this misses one opportunity for some extra batching and optimization.

Alternative
approach would be to recompute even earlier when Var changes are made. That are so-called `push-based` systems, but they are even less efficient and harder to
build without what signal implementors call "glitches", i.e. failing to propagate changes in a topologically sorted order of dependencies.

Then we can collapse `Incrementals` and `Observables`, basically any intermediate value can be an output. We rename this final object `Signal`. Finally, we observe
that `Var` is just a special `Signal` that can be written to, but shares the reading interface. So we arrive at this simpler code while hypothetically perserving the same SAC implementation and semantics:

```javascript
let width  = signal(3);
let depth  = signal(5);
let height = signal(4);

let base_area = read(width, depth, (x, y) => x * y);
let volume = read(base_area, height, (x, y) => x * y);

function main() {
  let display = (s) =>
    console.log(`${s}: base area: ${base_area.get()}; volume: ${volume_obs.get()}\n`);
  display("initial");
  height.set(10);
  display("after set");
}
```

Note, I have built a toy signals library based on that interface [cont-signal](https://github.com/rkirov/cont-signal) if one wants to play with it.

In order to get fully look like JS signals, we have to finally replace the explicit `s.read(val => {...})` with `read(() => {...s.val()...})` and rename `read` with `computed`.
To the familiar with FP techniques this is akin to continuation passing style rewrites and points to a deeper connection with the continuation monad (ignore this comment if not.)

```javascript
let base_area = read(width, depth, (x, y) => x * y);
let volume = read(base_area, height, (x, y) => x * y);
```

gets turned into

```javascript
let base_area = computed(() => width.get() * depth.get());
let volume = computed(() => base_area.get() * height.get());
```

basically interleaving the signal reads and the computation description.

## JS Signals are SAC - So What?

One of my favorite quotes is by Goethe:

> Mathematicians are like Frenchmen: whatever you say to them they translate into their own language and forthwith it is something entirely different.

Undoubtedly, this might read like that to JS practitioners - signals don't seem that complex and connecting them to obscure CS research can feel useless.

But let's think more deeply why - so much software engineering energy is spent on writing and rewriting JS signals libraries, without 
spending some of it learning SAC as implemented in the functional world? There could be three different reasons:

1. the connection is unknown.
1. the energy required to explore the connection is higher than the practical benefits.
1. there is nothing to be gained by exploring this connection.

I hope to popularize the connection with this post and solve 1. On personal aesthetics grounds I would like to
believe there is something to be gained (i.e. we are not in option 3), but I can't directly point to what that is, so I could be just plain wrong.

Some possibilities include for one, borrowing SAC's usage of certain optimized data structures from 80s see [Dietz and Sleator](https://www.cs.cmu.edu/~sleator/papers/maintaining-order.pdf)
for dependency tracking.

Another possible direction is the connection with the continuation monad which allows one to precisely define "singals are kinda like promises".
Realizing something is a monad allows reuse of ideas like a unified do-notation. Turns out in JS that can be simulated by existing async/await notation [see example](https://github.com/rkirov/cont-signal/blob/main/burrido-wrapper.test.ts).

So if there are truly some insights to be gained (option 2), I have tried to do my part by reimplementing different SAC solutions in JS to reduce the friction from reading
foreign languages like OCaml and Haskell.

## Towards Reactivity a la Carte paper

I am big fan of the build systems paper [Build Systems a la Carte](https://www.microsoft.com/en-us/research/wp-content/uploads/2018/03/build-systems.pdf).
I keep envisioning a similar paper / blog post - Reactive UIs a la Carte. My own attempt to get to something like that with [incremental computation](https://rkirov.github.io/posts/incremental_computation/) has failed. I no longer believe the four different approaches I describe there `forward/reverse push/pull` computation are
the right ones. Even just `push/pull` reactive system, something everyone is convinced is inherently clear, is as not as well-defined as I would like to see it.

A number of folks working on reactive system in JS have also tried to write blogs describing the different approaches to reactivity:

- [What is Reactivity](https://www.pzuraq.com/blog/what-is-reactivity)
- [The taxonomy of reactive programming](https://vsavkin.com/the-taxonomy-of-reactive-programming-d40e2e23dee4)
- [Components of Reactivity](https://dev.to/ninjin/components-of-reactivity-4f0a)
- ... likely there are more, send me links if you know of some that belong to that list ...

Sadly they miss the mark (for what I like to see, they are excellent blog posts by themselves) because they connect too closely to the specifics of the DOM, UI programming or JavaScript. I would like to see an abstract (but still in code) toy model of reactivity, that can be modified to be push or pull, or any other category of reactive system. It would be a principled exploration of the design space by abstracting away the specifics, so that one can see what has been tried, what hasn't, where do current solutions land. The sheer amount of engineers working on JS frameworks make the ad-hoc exploration of the design space tractable, but it feels like a random walk. It is bound to reach all possible states, but could it have been so much faster by having a clear map of the land and by learning where have others been. (Maybe the answer is to get SPJ interested in UIs and write that paper himself :))

## On the value of abstraction in theory and practice

Finally, I wanted to self-reflect on what makes this subject so irresistible to me. I basically got nerd snipped into writing this post by my friend [Evan](https://neugierig.org/software/blog/) mentioning the latest JS reactivity news and me ending up thinking about it obsessively for hours until I wrote this.

A lot of it has to do with personally enjoying abstraction to a degree that would seem unreasonable for a software engineer. I have been on both sides of
that eternal debate - both as a proponent and opposer of more abstraction, but in this particular situation my sense is that something of value is to be gained by approaching reactivity design through a more cross-language and cross-usecase lenses. As I said earlier I don't know what that precisely is, and I could be deluding myself (won't be the first time.)

I will one day write a much longer post about abstraction in general, but something about reactivity approached in an abstract cross-domain way tickles my brain similarly to [Gödel, Escher, Bach](https://en.wikipedia.org/wiki/G%C3%B6del,_Escher,_Bach). Aside, I recently discovered MIT has videos of a [whole class](https://www.youtube.com/watch?v=lWZ2Bz0tS-s&ab_channel=jasonofthel33t) on it. As far as I can tell, that book did not make any logician, musician, artist or neuroscientist practically do their job better, and yet it is universally accepted as a masterpiece. So there must be something universally enjoyable in connecting seemingly disparate disciplines through their shared abstract patterns – a satisfaction that transcends immediate practical utility while long-term proving to be "unreasonably effective" (to borrow from [Wigner](https://www.maths.ed.ac.uk/~v1ranick/papers/wigner.pdf).)