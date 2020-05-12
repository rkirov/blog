+++
title = "Incremental Computation (Draft of part 2)"
date = 2020-05-10
images = []
tags = ['incremental computation']
categories = ['programming']
draft = false
+++

## Incremental computation (part 2)

We have been talking about general computation, but so far our language was
very limited. We only used function calls, numbers, and simple variable
binding.

We will slowly add more language primitives and see how to still have 
incrementality of the computation. First is conditional statements.

## Conditional statements

Let's add a single conditional statement first.

```typescript
function cond(b: boolean, x: number, y: number) {
  return b ? x * x : y * y; 
}
```

What does it mean to make `cond` incremental? Say
we compute the result first for some values of
`b`, `x`, `y`.

```typescript
// computation
cond(true, 1, 2);
````

It would be ideal to skip any work if the recomputation looks like `cond(true,
1, 3)`. The value of `y` is inconsequential while the first argument is `true`.
However, if `b` is `false` the situation is flipped and any change of `x`
should be inconsequential to the computation.

Let's go over the techniques we learned about in part 1. First it is clear that
naive memoization will not work. When going from `cond(true, 1, 1)`  to
`cond(true, 1, 3)` memoization will redo the whole work, while we want to do
nothing.

Then we can try to build the computation graph. Breaking down the computation
into basic operations we get:

```typescript
const op1 = x * y;
const op2 = y * y;
const op3 = b ? op1 : op2;
```

![Computation graph](/dynamic1.png)

However, this still comes short as changing `y` means that even if `b` is
always `true`, the square of `y` will be computed. This is unavoidable
using the techniques we have seen so far.

In order to achieve better incrementality in presence of conditionals, we will
have to move to a dynamic model of computations.

## Dynamic vs Static computation graph

Then depending on the last value of `b` the computation is represented by one
of the two following options.

![cond is true graph](/dynamic-true.png)

![cond is false graph](/dynamic-false.png)

The incremental algorithm should switch between the two options after reading
`b`.

In this example computing `b` is trivial (it is an input), but in general
there could be another sub-computation to get to it.

To fully support incremental computation and conditional statements one needs
to have support for a dynamically adjusting computation graph. Otherwise, the
static graph techniques will over-approximate, but still provide some value.
They will basically treat `op1 ? op2 : op3` as a basic function `f(op1, op2,
op3)` that always needs all three inputs.

One has to be careful in using the word dynamic as here we are talking about
adjusting the computation graph, not the graph creation itself. As we have
seen in part 1, the computation graph implementations we used where build
dynamically. We still consider them static incremental computation as they
cannot adjust during re-computations.

Unsurprisingly, the same observations were made in the build systems space in
the paper `Build systems a la carte`. In their work they used a model for build
systems written in Haskell. Our separation between static and dynamic
computation graph is equivalent to what they call - applicative vs monadic
build systems. Most popular build systems like make are applicative (static).

## Adaptive (self-adjusting) computation

While turning the example above into an incremental function is relatively
easy using if-else statements (try it), a general framework that supports
dynamic incremental computation is too involved to go through in this post.

Luckily, this has been well studied in the past and there existing
implementations. The work of [Acar - Self-adjusting
computation](https://www.cs.cmu.edu/~rwh/theses/acar.pdf) support dynamic
incremental computation in OCaml. It was slightly modified and improved by
Carlsson in [Monads for Incremental
Computing](https://dl.acm.org/doi/abs/10.1145/581478.581482).  His
implementation is in Haskell making heavy use of monads and do-notation.  I
have translated his implementation to TypeScript -
[adapt-comp](https://github.com/rkirov/adapt-comp). It required loosing some of
the heavy monadic type guarantees from the Haskell implementation, but
it caries the same basic algorithms and datatypes.

At this point I use the names 'self-adjusting', 'adaptive' and 'incremental'
computation interchangeably when talking about the concepts. The library
implementing the particular algorithms for incremental computation is called
'adapt-comp' in reference to the original work by Acar, et.al.

The distance function example from part 1 looks like this in adaptive
computation:

```typescript
const x = comp(pure(1));
const y = comp(pure(1));

const op1 = comp(read(x, x => pure(x * x)));
const op2 = comp(read(y, x => pure(x * x)));
const op3 = comp(read(op1, x => 
                 read(op2, y =>
                   pure(x + y))));
const op4 = comp(read(op3, x => pure(Math.sqrt(x)))); 

// computation
console.log(op4.get());

// re-computation
write(y, 3);
console.log(op4.get());
```

The basic block of `adapt-comp` are the `comp` constructor which builds a
single build block of the computation. The only argument to `comp` describes
how is the operation built. It either a basic value, which has to be wrapped
like this - `pure(<some value>)`, or a result of reading another variable using
the `read` function. Once a variable is read with `read`, we pass a callback
that describes what to do with the value read. The only catch is that at the
end of the callback we still have to return another `pure` or `read`.

(Aside: if one is familiar with monads they recognize the structure here, which
comes from the Haskell linage of this code. If not don't worry, this exposition
is self-contained.)

This is a push-based direct incremental computation using the terminology from
part 1. When one calls `write(y, 3)`, all the computations that had `read(y,
...)` have their corresponding callback retriggered, then recursively their
dependents are retriggered. At any point the last and new values are compared
and if the value hasn't changed the retriggering is stopped.

So far, this doesn't seems to carry its weight as we had simpler
implementations doing this in part 1. However, 'adapt-comp' supports a dynamic
incremental computation too. Our original dynamic example looks like this in 
'adapt-comp':

```typescript
const bVar = comp(pure(true));
const xVar = comp(pure(1));
const yVar = comp(pure(1));

const cond = comp(read(bVar, b => b ? 
                                  read(xVar, x => pure(x * x)) :
                                  read(yVar, y => pure(y * y))));
```

One convenience is that we no longer had to extract `op1` and `op2` into
separate definitions. There are implicitly objects behind the two `read` calls
that play similar role.

But the real benefit is that the dynamic reads during computing `cond` are
recorded on each computation and recomputation. Since `y` is not a static
dependency of `cond`, but rather something that was read (or not) as we
went along. Thus a recomputation is not triggered when y changes.

```
write(y, 3);  // does not even recompute y * y
console.log(op4.get());  // immediate return of the previous value
```

## Observable example

As seen in part 1, the observable pattern and in particular RxJS implementation
allows for easy implementation of push-based incremental computation (and
more).

To strengthen this point this is how our example will look like in RxJS.

```typescript
import {of, switchMap, Subject} from 'rxjs';
import {map} from 'rxjs/operators';

const b = new Subject<boolean>();
const x = new Subject<number>();
const y = new Subject<number>();

const op1 = x.pipe(map(square));
const op2 = y.pipe(map(square));
const op3 = b.pipe(switchMap(b => b ? op1 : op2));

op3.subscribe(console.log);

// initial computation
b.next(true);
x.next(1);
y.next(1);

// re-computation of with new y
y.next(3);  // no log occurs

// re-computation of with new x
x.next(3);  // log 9 occurs
```

(Aside for readers familiar with monads: you might remember that build systems
article called systems supporting dynamic computation graphs -- monadic. It is
not a coincidence that we had to use the switchMap operator here, which indeed
has the monadic interface).

If feels natural to do for and while loops next, but actually more primitive 
concept is recursion.

## Recursion

We have been focusing on a single function so far. Extending the framework to
multiple distinct functions is straight forward. Graphically, a function call
was treated as a single node, but if it is an incremental computation we can 
embedded its own graph into the node to form one bigger computation graph.

The question of multiple calls into the same function, especially recursively
might give us a pause. Say we want to make the following recursive function
incremental:

```typescript
function f(x: number, y: number) {
  if (x <= 0) {
    return y; 
  }
  return f(x - 1, x + y);
}
```

The intermediate step is to write it in the form of simple operations and conditionals:

```typescript
function f(x: number, y: number) {
  const nextX = x - 1;
  const nextY = x + y;
  const lessZ = x <= 0;
  return lessZ ? y : f(nextX, nextY);
}

f(10, 0);  // returns 55;
```

Now we can use the `adapt-comp` library to directly translate:

```typescript
// description
function f(x: Modifiable<number>, y: Modifiable<number>): Mod<number> {
  const nextX = comp(read(x, x => pure(x - 1)));
  const nextY = comp(read(x, x => read(y, y => pure(x + y))));
  const lessZ = comp(read(x, x => pure(x <= 0)));
  return comp(read(lessZ, b => b ? read(y) : read(f(nextX, nextY)))); 
}
const x = comp(pure(1));
const y = comp(pure(1));

// computation
const res = f(x, y);
res.get();  // returns 55;

// re-computation
write(y, 100);
res.get();  // returns 155;
```

As we see there is nothing special about recursion, because we can dynamically
create new computation variables (they have the `Modifable<T>` interface). The
function boundaries are almost irrelevant at for the re-computation. The only
thing that matters is which computations are created `cond` and what reads have
happened during the computation.

## Looping and Mutable variables

Finally, we can get to `for` and `while` loops. Unfortunately, we cannot
directly support `for` and `while` loops in the `adapt-comp` world. Calling
`comp` in a loop by itself is fine, but we cannot interleave the termination
control flow from the `read` callbacks back to original loop.

However, we are in luck as all `for` and `while` loops can be rewritten through
recursion. In fact the recursion example above could have been written as the
following imperative program:

```typescript
function f(x: number, y: number) {
  let acc = y;
  while (x <= 0) {
    acc += x; 
  }
  return acc;
}
```

So we already have an incremental solution for it above.

## Adding mutable objects and differential programming

TODO - present "delta calculus".

## Online combinatorial algorithms

TODO - go over some of Tarjans' papers.

## UI programming 

TODO - incremental programming in JS frameworks.

## Other implemenations

TODO - rust salsa and timely dataflow.

## Conclusion

TODO

## References

1. Mokhov, Mitchell, Jones, Build Systems à la Carte, Proc. ACM Program, 2018.
1. Magnus Carlsson. Monads for Incremental Computing. ICFP '02 Proceedings of the seventh ACM SIGPLAN international conference on Functional programming
1. U. Acar, G. Blelloch, and R. Harper. Adaptive functional programming. In Principles of Programming Languages (POPL02), Portland, Oregon, January 2002. ACM
1. Conal Elliot, Functional Reactive Animation 
1. https://crates.io/crates/salsa
1. https://github.com/TimelyDataflow/differential-dataflow
1. https://www.cs.princeton.edu/~appel/papers/ssafun.pdf
1. https://www.jantar.org/papers/chakravarty03perspective.pdf
1. https://blog.janestreet.com/introducing-incremental/
