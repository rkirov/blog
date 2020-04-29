+++
title = "Incremental Computation (Draft of part 1)"
date = 2020-04-26
images = []
tags = ['incremental computation']
categories = ['programming']
draft = false
+++

Incremental computation is a way of performing computations, with the
expectation of future changes in inputs. When those changes occur the new
output can be obtained efficiently, at minimum avoiding redoing the
whole computation.

Many programming environments deal with this problem - UI programming,
dataflow, build systems, etc. Despite its prevalence, I find it is rarely
viewer as a unified computational paradigm, as opposed to ad-hoc application of
caching. In comparison, other computational paradigms like concurrent or
distributed computating have better established nomenclature and techniques. 

The purpose of this text is to teach you what 'incremental computation' is
(likely you have seen it before, but didn't use that term) and establish common
terminology for the basic approaches to the problem. I spend the last few years
surveying various academic and industry work that relate to incremental
computation, resulting in diverse pool of prior work. I hope this text guides
future work in the theory or applications of incremental computation.

## Motivation

To introduce 'incremental computation' I will start with intentionally limited
environment that only contains functions and immutable primitives (numbers).
If I jump straight into a full featured programming language will obscure the
core ideas. In a more formal setting this would be the lambda calculus, but I 
will just use TypeScript and say away from higher-level constructs like arrays.
You can replace this with any run-of-the-mill programming language that has
closures.

Let's start with a very simple computation:
 
```typescript
function d(x: number, y: number) {
  return Math.sqrt(x * x + y * y);
}

const computation = d(1, 2);
```

Incremental computation, ultimately, is concerned with the question how to most
effectively re-compute `d(x, y)` when some of the inputs have changed from `1`
and `2` to something else.

Say the new computation I would like to know is `d(1, 3)`. The new inputs
do not fully match the old ones, so a simple caching will not work as writen.

However, instead of giving up on caching, I can break down the computation
down to the basic operations: addition, multiplication and square root.

There are four core operations performed - two multiplications, one addition
and one `Math.sqrt`. I call the basic building blocks of an computation -
operations, but note that this is not standard. To spell it out using temporary
variables:

```javascript
const op1 = 1 * 1;
const op2 = 2 * 2;
const op3 = op1 + op2;
const op4 = Math.sqrt(op3);
```

Compare now with the version with the new inputs.

```javascript
const op1 = 1 * 1;
const op2 = 3 * 3;
const op3 = op1 + op2;
const op4 = Math.sqrt(op3);
```

For this toy example, the best incremental computation will reuse or skip `1 *
1` and then redo the other three operations with the new inputs.

This can be easily expressed in an abstract directed graph form. The original
computation is:

![Computation graph](/incr1.png)

Each node represents a variable and all incoming edges together represent
the inputs to the function that computes the variable.

The recomputation for `d(1, 3)` after `d(1, 1)` will ideally only go through
the path in red.

![Recomputation graph 1](/incr3.png)

Meanwhile, the recomputation for `d(-1, -1)` after `d(1, 1)` will go start with
two red paths, but end in the middle (since the output does not change).

![Recomputation graph 2](/incr4.png)

The basic algorithm over the abstract computation graph is:

**Basic Algorithm of Incremental Computation**

1. when at least one of the inputs of an operation has changed, redo the
   operation to get new outputs.
1. repeat until there are no more changes.

This algorithm has no name as it is so simple. The complexity comes in how to
best build the computation graph, especially on top of general programming
languages which have no incremental primitives.

Also at this level you can already start to see the similarity with build
systems like 'make'. In a way build systems are the simplest instances of
incremental computation. 

The key questions around incremental computation at the abstract level of the 
computation graph will be:

- how is this computation graph built?
- how is the graph traversed. The Basic Algorithm in intentionally vague.
- can it change while the program is running?
- what if it has cycles?

The questions that I view secondary to the core and not going to explore here are:

- who does the recomputation  
- can it be parallelized across different processes

## Incremental Computation is not "just" Memoization

At this simple form incremental computation seems to be simply solved by
function memoization. Memoization is comparing the inputs of a function against
a cache of (inputs,output) pairs. If seen, the function body is skipped and
output used, otherwise the function body executes and the output is stored in
the cache.

Incremental computation usually is concerned with the recomputation against a 
single past result, which would make it most fitting to use memoization with
a cache size of one. I view basic caching considerations - like size of cache,
least-frequently-used vs other strategies, orthogonal to the core problems
of incremental computation.

As previously discussed, we have to first rewrite the function to separate
the basic operations.

```javascript
function d(x: number, y: number) {
  const op1 = x * x;
  const op2 = y * y;
  const op3 = op1 + op2;
  const op4 = Math.sqrt(op3);
  return op4;
}
```

Now, we can memoize each inner operation along with the whole function:

```javascript
const memoAdd = memo(add);
const memoSquare = memo(square);
const memoSqrt = memo(Math.sqrt);

function innerMemoD(x: number, y: number) {
  const op1 = memoSquare(x);
  const op2 = memoSquare(y);
  const op3 = memoAdd(op1, op2);
  const op4 = memoSqrt(op3);
  return op4;
}
const memoD = memo(innerMemoD);
```

However, notice that while each operation is memoized - the computation
proceeds exactly the same way as before through the

![Computation Graph](/incr5.png)

We made the cost of each operation (think edge in the graph) as small as
possible during the evaluation, but the flow is the same. 

We can produce the even more incremental which truly follows a different
computation path program by adding some 'check-if-changed-and-skip' calls:

```typescript
let lastOp1: null|number = null;
let lastOp2: null|number = null;
let lastOp3: null|number = null;
let lastOp4: null|number = null;

function skipCallsD(x: number, y: number) {
  const op1 = memoSquare(x);

  // why not 'if op === lastOp1 -> skip' here?
  const op2 = memoSquare(y);

  if (op1 === lastOp1 && op2 === lastOp2) return lastOp4;
  lastOp1 = op1; lastOp2 = op2;

  const op3 = memoAdd(op1, op2);

  if (op3 === lastOp3) return lastOp4;
  lastOp3 = op3;

  const op4 = memoSqrt(op3);
  lastOp4 = op4;

  return op4;
}
```

Notice that the structure of the checks matches the abstract computation graph.
We can't skip to the end if `op1` hasn't changed, because potentially `op2` can
be different.

This allows us to achieve the incremental flow depicted here for recomputing
`d(-1,-1)` after `d(1, 1)`

![Recomputation graph 2](/incr4.png)

Hopefully, at this point you are convinced that incremental computation problem
is not 'just' memoization as we had to add if-else statements too. Next I will
show that the program above can be made to look like just memoization, but will
require a bigger program transformation.

## Incremental Computation is "just" Memoization

We need to start with a bit of insight from the lambda calculus. Each variable
assignment `let x = <init>; <rest>` can be seen as `((x) => <rest>)(<init>)`,
giving us an opportunity to insert a memoization boundary on some new
functions that were not previously there. 

It will takes a few attempts to get to the correct form. Let's start naively
replacing assignments with functions.

```typescript
function wrongD(x: number, y: number) {
  return ((op1: number) => 
    ...
  )(memoSquare(x));
}
```

then add `op2`,

```typescript
function wrongD(x: number, y: number) {
  return ((op1: number) => 
    ((op2: number) =>
      ...
    )(memoSquare(y))
  )(memoSquare(x));
}
```

and eventually get to

```typescript
function wrongD(x: number, y: number) {
  return ((op1: number) => 
    ((op2: number) =>
      ((op3: number) =>
        ((op4: number) =>
          op4 
        )(memoSqrt(op3))
      )(memoAdd(op1, op2))
    )(memoSquare(y))
  )(memoSquare(x))
}
```

Now imagine that all four inner functions are also memoized. While
it will give us the correct recomputation of `d(-1,-1)` after `d(1,1)`
it will be wrong for `d(1, 3)` after `d(1, 1)`. This is because
as soon as `op1` is the same as before we will skip to the end, ignoring
any change to `op2`.

The correct rewriting for memoization is below, reflecting the fact that 
`op3` has two inputs over which we have to memoize together.

```typescript
function insideOutD(x: number, y: number) {
  return ((op1: number, op2: number) => 
    ((op3: number) =>
      ((op4: number) =>
        op4 
      )(memoSqrt(op3))
    )(memoAdd(op1, op2))
  )(memoSquare(x), memoSquare(y))
}
```

Finally, we can extract the functions to run them through the memoization operation:

```typescript
const fn1 = memo((op1: number, op2: number) => fn2(memoAdd(op1, op2)));
const fn2 = memo((op3: number) => fn3(memoSqrt(op3)));
const fn3 = memo((op4: number) => op4);

function superMemoDInner(x: number, y: number) {
  return fn1(memoSquare(x), memoSquare(y))
}
const superMemoD = memo(superMemoD, 'd');
```

While requiring a fancy program rewriting, to the point where the original
program is unrecognizable, this has a more aesthetically pleasing property that
one can begin to imagine a principled way of adding incrementality instead of
manually inserting if-else statements.

Also having so many memoization calls is certainly an overkill in any practical
problem -- we are memoizing an identity function at some point, which would be
faster to run -- but the point of pushing a simple example to the extreme is to
see all the opportunities for incrementality. In practice, the incrementality
solution is often manually, inserted into an non-incremental program. Exploring
maximal incrementality over tiny program, shows all the techniques and ways in
which incrementality can be inserted.

Also, notice that we still do not have incrementality for this flow: 

![Recomputation graph 1](/incr3.png)

While we can "skip to the end" of the computation, we cannot skip over the
purely `x` part of the computation graph, if we know `x` hasn't changed.

While it is likely achievable with more fancy program rewriting, we will get
there  by using a different technique - push-based incremental computation.

## Push vs Pull in Incremental Computation

The memoization solution above has a nice similarity with the original
computation, it followed roughly same path as the original computation. 

While very natural, there is an alternative approach. The new updates can themselves
trigger a recomputation. This is known as _push-based_ computation, because the
new values of inputs, push themselves into the computaiton.

The classical approach is knowns as _pull-based_, as to recompute `d(1,3)` we
have to pull the new inputs `1` and `3`. Similarly, `op3 = add(op1, op2)` is
seen as pulling `op1` and `op2` to compute `op3`.

Another terminology used is reactive vs interactive, but there is something called
'push-based reactivity' so I am going to steer clear of these to avoid confusion.

To summarize:
- classic computation (pull) - computation *pulls* the updated inputs to
  produce the desired outputs.
- push-based incremental computation - the updated inputs *push* themselves
  into the computation to produce the desired outputs.

In our original example, a push-based version will look something like this:

```typescript
const x = {
  set(x: number) {
    op1.update(x);
  }
};

const y = {
  set(y: number) {
    op2.update(y);
  }
};

const op1 = {
  last: null as number | null,
  update(x: number) {
    if (x !== this.last) {
      op3.updateArg1(x * x);
    }
  }
}

const op2 = {
  last: null as number | null,
  update(x: number) {
    if (x !== this.last) {
      op3.updateArg2(x * x);
    }
  }
}

const op3 = {
  arg1: null as number|null,
  arg2: null as number|null,

  updateArg1(x: number) {
    if (x !== this.arg1 && this.arg2 != null) op4.update(this.arg1 + this.arg2);
    this.arg1 = x;
  },

  updateArg2(y: number) {
    if (y !== this.arg2 && this.arg1 != null) op4.update(this.arg1 + this.arg2);
    this.arg2 = y;
  },
}

const op4 = {
  last: null as number | null,
  update(x: number) {
    if (x !== this.last) {
      const result = Math.sqrt(x);
      console.log('filal result:', result);
    }
  }
}

// computation
x.set(1);
y.set(1);

// re-computation
y.set(2);
```

This solution lets us achieve the desired recomputation flow.

![Recomputation graph 1](/incr3.png)

When is a push-based vs pull-based incremental solution better? To be honest I
am not sure yet, would like to hear what do you think.

## Connection with Reactive / Streaming frameworks

If you are familiar with reactive primitives like - streams, observables, etc,
you might have observed that the 'push-based' solution, can be much simpler
using them. For example, here is the same solution, using rxjs observables.
To be fully equivalent it will need to add caching along each observable,
so that it doesn't reemit when the new value equals the previous one.

```typescript
import {of, combineLatest, Subject} from 'rxjs';
import {map} from 'rxjs/operators';

const x = new Subject<number>();
const y = new Subject<number>();

function square(x: number) {
  return x * x;
}

function add(x: number, y: number) {
  return x + y;
}

const op1 = x.pipe(map(square));
const op2 = y.pipe(map(square));
const op3 = combineLatest(op1, op2).pipe(map(([x, y]) => add(x, y))); 
const op4 = op3.pipe(map(Math.sqrt));

op4.subscribe(console.log);

console.log('initial computation');
x.next(1);
y.next(1);

console.log('re-computation of with new y');
y.next(-1);

console.log('re-computation of with new x');
x.next(3);
```

The next challenge is to extend the toy computation model to a full blown
language that includes control flow and mutable object. We will begin with control flow.

## Incremental computation and control flow

TODO - expand the original example. Discuss connection with 'applicative vs
monadic' build systems.

## Adaptive computation 

Present https://github.com/rkirov/adapt-comp.

## Adding mutable objects and differential programming

TODO - present "delta calculus".

## UI programming 

TODO - incremental programming in JS frameworks.

## Conclusion

TODO

## References

1. Mokhov, Mitchell, Jones, Build Systems à la Carte, Proc. ACM Program, 2018.
1. Magnus Carlsson. Monads for Incremental Computing. ICFP '02 Proceedings of the seventh ACM SIGPLAN international conference on Functional programming
1. U. Acar, G. Blelloch, and R. Harper. Adaptive functional programming. In Principles of Programming Languages (POPL02), Portland, Oregon, January 2002. ACM
1. Conal Elliot, Functional Reactive Animation 
1. https://crates.io/crates/salsa
1. https://github.com/TimelyDataflow/differential-dataflow
