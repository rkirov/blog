+++
title = "Incremental Computation (Draft)"
date = 2020-04-26
images = []
tags = []
categories = []
draft = true
+++

Incremental computation is a computational paradigm that naturally appears in a
many programming environments - UI programming, dataflow, build systems, etc.
Despite its prevalence, its fundamentals, nomenclature and practical trade-offs
are not as well-shared across applications as other paradigms like concurrent
computations or distributed computations. The purpose of this text is to teach
you what it is and what are the core approaches to incremental computation.  I
spend the last few years surveying various academic and industry work that
relate to incremental computation, trying to understand them under common
terminology. I hope this illuminates the current implementations and
facilitates the sharing of knowledge.

## Motivation

To introduce 'incremental computation' I will start with intentionally limited
environment that only contains functions and immutable primitives (numbers).
If I jump straight into a full featured programming language will obscure the
core ideas. In a more formal setting this would be the lambda calculus, but we
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

Say the new computation we would like to know is `d(1, 3)`. The new inputs
do not fully match the old ones, so a simple caching will not work as writen.

However, instead of giving up on caching, we can break down the computation
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
1` and then redo the other three operations with the new inputs. This can be
easily expressed in an abstract directed graph form:

```
x -> op1
y -> op2
op1 -> op3
op2 -> op3
op3 -> op4
```

The basic algorithm over the abstract computation graph is:

Basic Algorithm of Incremental Computation

1. when at least one of the inputs of an operation has changed, redo the
   computation to get new outputs.
1. repeat this or all nodes.

This algorithm has no name as it is so simple. The complexity comes in how to
best build the computaiton graph, especially on top of general programming
languages which have no incremental primitives.

Also at this level you can already start to see the similarity with build
systems like 'make'. In a way build systems are the simplest instances of
incremental computation. 

The key questions around incremental computation at the abstract level of the 
computation graph will be:

- how is this computation graph built?
- how is the graph traversed. We were intentionally vagues in the Basic Algorithm.
- can it change while the program is running?
- what if it has cycles?

The questions that I view secondary to the core and not going to explore here are:

- who does the recomputation  
- can it be parallelized across different processes

# Incremental Computation is not "just" Memoization

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
const memoAdd = memo(add, 'add');
const memoSquare = memo(square, 'square');
const memoSqrt = memo(Math.sqrt, 'sqrt');

function innerMemoD(x: number, y: number) {
  const op1 = memoSquare(x);
  const op2 = memoSquare(y);
  const op3 = memoAdd(op1, op2);
  const op4 = memoSqrt(op3);
  return op4;
}
const memoD = memo(innerMemoD, 'd');
```

However, notice that while each operation is memoized - the flow through the
function is always the same `op1 -> op2 -> op3 -> op4`, even if there some
sharing like computing `d(1, 3)` or `d(-1, 2)` after `d(1, 2)`. 

We can produce the 'even-more-incremental' program by adding some 
'check-if-changed-and-skip' calls:

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
We can't skip to the end if `op1` hasn't changed, because potentially `op2`.

Hopefully, at this point you are convinced that incremental computation problem
is not 'just' memoization. Next I will show that it can be made to be just
memoization, but will require a bigger program transformation.

# Incremental Computation is "just" Memoization

We need to start with a bit of insight from the lambda calculus. Each variable
assignment `let x = <init>; <rest>` can be seen as `((x) => <rest>)(<init>)`,
giving us an opportunity to insert a memoization boundary on some new
functions that were not previously there. 

The rewriten function now looks like this:

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

Notice that we didn't use the similar looking rewriting, which would be more inline 
with mechanically replacing each variable assignment operation.

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

It would have resulted in the correct original computation, but wrong incremental
computation. Turning this rewriting to an actual automated process would be an
interesting project.

Finally, we can extract the functions to run them throught memoization operation:

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
program is unrecognizable, this has a more aesteically pleasing property that
one can begin to imagine a principled way of adding incrementality instead of
manually inserting if-else statements.

Also having so many memoization calls is certainly an overkill in any practical
problem -- we are memoizing an identity function at some point, which would be
faster to run -- but the point of pushing a simple example to the extreme is to
see all the opportunities for incrementality. In practice, the incrementality
solution is often manually, inserted into an non-incremental program. Explroing
maximal incrementality over tiny program, shows all the techniques and ways in
which incrementality can be inserted.

But first we will need to introduce a key dimension of incremental computation, and 
show another solution which is in a way the complete opposite.

# Push vs Pull in Incremental Computation

The memoization solution above there was a nice similarity re-computation, followed
exactly the same path as the original computation. Another way to say this is 
that the re-computation was started and in the process it *pulled* the
updated inputs.

While very natural, there is an alternative approach. The new updates can themselves
trigger a recomputation. 

Another terminology used is reactive vs interactive, but there is something called
'push-based reactivity' so I am coing to steer clear of these to avoid confusion.

To sumarize:
- classic computation (pull) - computatation *pulls* the updated inputs to
  produce the desired outputs.
- push-based incremental computation - the updated inputs *push* themselves
  into the computation to produce the desired outputs.

In our original example, a push-bassed version will look something like this:

```typescript
const x = {
  set(x: number) {
    p1.update(x);
  }
};

const y = {
  set(y: number) {
    p2.update(y);
  }
};

const p1 = {
  last: null as number | null,
  update(x: number) {
    if (x !== this.last) {
      p3.updateArg1(x * x);
    }
  }
}

const p2 = {
  last: null as number | null,
  update(x: number) {
    if (x !== this.last) {
      p3.updateArg2(x * x);
    }
  }
}

const p3 = {
  arg1: null as number|null,
  arg2: null as number|null,

  updateArg1(x: number) {
    this.arg1 = x;
    if (this.arg2 != null) p4.update(this.arg1 + this.arg2);
  },

  updateArg2(x: number) {
    this.arg2 = x;
    if (this.arg1 != null) p4.update(this.arg1 + this.arg2);
  },
}

const p4 = {
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

Notice that in terms of operations this approach is just as efficient as the
pull-based one - one operation is reused and three others reevalutated. 

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
x.next(2);
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
