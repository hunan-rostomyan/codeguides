#### Equality Relations

JS has two equality relations: `==` and `===`. Let's call the double one _weak equality_ and the triple one _strict equality_. Strict equality is stronger than weak equality in the sense that:

> &forall;x: `x === y` &rarr; `x == b`.

JavaScript has two value types: primitive types (e.g. 5, "Obama") and reference types (e.g. add, {name: "Barack"}). These operators are defined only for some of those types.

* There is no universal equality relation between arbitrary values of a reference type. The programmer has the responsibility of defining the equality relation for the objects that she creates.

* As regards values of a primitive type, strict equality does nothing surprising: if the relata contain copies of the same primitive value (e.g. `x: 5`, `y: 5`), then they are said to be strictly equal. Weak equality is more permissive. It will evaluate to true if any of the following conditions are met, for relata `a` and `b`:
  1. `a === b`,
  2. `a' === b` where `a'` = `B(a)` and `B` is the type of `b`,
  3. `a === b'` where `b'` = `A(b)` and `A` is the type of `a`.


#### Questions

> Is `x == y` true for some two arrays `x` and `y`?
> Is `==` an [equivalence relation](https://en.wikipedia.org/wiki/Equivalence_relation#Definition)? What about `===`?